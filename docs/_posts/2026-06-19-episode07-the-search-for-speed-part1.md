---
layout: post
title: "Episode 07 - The Search for Speed (Part 1)"
author: dodger-one
tags:
  - rp2350
  - st7789
  - performance
  - emulator
  - technical
---

## The Search for Speed (Part 1)

Or maybe "Need for Speed"?  
After all the holy crap seen on Episode 06, when the new Pico 2 Finally Booted, the next problem became obvious immediately:

> It worked, but it was nowhere near the performance target.

The rough starting point was around the low-`30 FPS` range on the first ST7789-oriented attempts. The end goal was very clear from the beginning: make the emulator run at the original Game Boy cadence, which in practice meant getting stable full-speed gameplay at about `59.75 FPS`.

This Episode is the a technical summary of that phase.  
Sorry guys, no great pic's (or not a lot of them at least)...

## Step 1: Treat ST7789 As a Different Performance Problem

The original project lineage was built around the ILI9225 path, but the ST7789 build changed the constraints completely:

- different SPI transfer format
- different panel size and orientation
- different render area
- much more pixel traffic once the image was scaled to fill the DMG window properly

The final ST7789 driver now uses a `320x240` logical space and renders the Game Boy image into a `266x240` area, centered and shifted to fit the enclosure layout:

- [src/firmware/mk_st7789.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/mk_st7789.c#L28-L44)

That was visually the right decision, but it also meant the frontend had to scale every `160x144` frame into a much larger output region. From that point on, the LCD path was no longer a small detail. It was one of the main performance battlegrounds.

## Step 2: Increase SPI Throughput for the ST7789 Path

The ST7789 path also needed a different SPI configuration than the old ILI9225 one.

The split is visible in the LCD init path:

- ST7789: `50 MHz`, `8-bit`
- ILI9225: `30 MHz`, `16-bit`

Reference:

- [src/firmware/main.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/main.c#L1388-L1398)

That change mattered because the ST7789 transport is fundamentally a byte-stream path. Once the panel was working reliably, the next constraint was simple: the more bytes that could be pushed per second, the less time core 0 had to stall behind the display.

This was one of the first clearly measurable wins.

### FPS impact:

- Around +5~7FPS win. Emulation around 36FPS

## Step 3: Keep LCD Work Off the Main Emulation Core

The biggest structural improvement was not a magic compiler flag. It was architectural:

- run emulation on core 0
- run LCD submission on core 1
- communicate through a bounded queue of scanline jobs

That design is still present in the current source:

- LCD queue depth and line buffers:
  - [src/firmware/main.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/main.c#L197-L200)
- launching the LCD worker core:
  - [src/firmware/main.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/main.c#L854-L868)
- core 1 command loop:
  - [src/firmware/main.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/main.c#L870-L919)
- line submission from the emulator frontend:
  - [src/firmware/main.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/main.c#L925-L945)

This was one of the most important turning points in the speed search.

Before that split, LCD output and emulation time were too entangled. Once the panel transport was pushed onto the second core, the project became much easier to reason about:

- emulation cost could be measured separately
- LCD queue contention became visible
- display bottlenecks stopped looking like "mystery slowdowns"

It did not solve performance by itself, but it made the rest of the work possible.

### FPS impact:

- Around +10~12FPS win. Emulation around 48FPS

## Step 4: Stop Sending Tiny Pieces of Work to the Panel

The next major improvement was inside the ST7789 driver itself.

Instead of treating every line update as an isolated SPI transaction, the driver now:

1. scales one Game Boy line from `160` pixels to `266`
2. duplicates rows vertically to reach the `240`-pixel output height
3. accumulates several adjacent rows into a small band
4. queues that band to DMA

Relevant code:

- line scaling:
  - [src/firmware/mk_st7789.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/mk_st7789.c#L218-L226)
- DMA queue types and buffers:
  - [src/firmware/mk_st7789.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/mk_st7789.c#L92-L123)
- DMA queue management:
  - [src/firmware/mk_st7789.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/mk_st7789.c#L290-L485)
- banded write path in `mk_ili9225_write_pixels()`:
  - [src/firmware/mk_st7789.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/mk_st7789.c#L602-L670)

This was a real performance optimization, not a cosmetic refactor.

The goal was to reduce command overhead and let the SPI/DMA pipeline move larger contiguous chunks instead of constantly bouncing between tiny writes. Once that banded path was in place, the ST7789 backend started behaving much more like a transport engine and much less like a debug port.

### FPS impact:

- Around +4~5FPS win. Emulation around 52FPS

## Step 5: Add Real Instrumentation

For a while, performance tuning was still too qualitative:

- "this scene feels faster"
- "this version seems smoother"
- "Kirby still dips here"

That was not enough anymore.

The project needed to print useful timing data every second, and that is exactly what happened. The log evolved from a simple FPS counter into a set of performance buckets that exposed where time was really going.

The current tree still contains the simpler human-readable FPS report:

- [src/firmware/main.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/main.c#L1576-L1592)

During the speed-search phase, the exported development thread shows the more detailed `PERF:` logs that broke frame cost into buckets such as:

- `emu_us`
- `lcd_submit_us`
- `lcd_core_us`
- `audio_cb_us`
- `i2s_us`

That changed the process completely. Instead of just asking "is it faster?", the question became:

- is core 0 spending time inside emulation?
- is it blocked while submitting LCD work?
- is core 1 saturated by the panel pipeline?
- is audio stealing a meaningful amount of budget?

Once those numbers were visible, several bad ideas could be rejected quickly.

### FPS impact:

- direct gain: `0` :rofl:
- practical gain: huge, because from this point on I stopped guessing
- this is the moment when performance work turned into profiling instead of folklore

## Step 6: Use Interlace Carefully, Understand Frame Skip Correctly

Two existing knobs from Peanut-GB kept showing up during testing:

- `interlace`
- `frame_skip`

The important thing is that they are not equivalent.

Peanut-GB's `frame_skip` is a real draw-suppression mechanism:

- line rendering is skipped on alternate frames:
  - [src/inc/peanut_gb.h](https://github.com/dodger-one/PicoDMG/blob/master/src/inc/peanut_gb.h#L1440-L1449)
- the skip state toggles at VBlank:
  - [src/inc/peanut_gb.h](https://github.com/dodger-one/PicoDMG/blob/master/src/inc/peanut_gb.h#L3414-L3420)

And in this frontend, enabling `frame_skip` also disables audio output work:

- [src/firmware/main.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/main.c#L1505-L1509)

That is why `frame_skip` could produce huge FPS numbers during experiments: it was not "slightly lighter rendering". It was effectively a fast-forward mode.

`interlace`, on the other hand, was a much more defensible practical compromise for this hardware. The relevant Peanut-GB logic is here:

- [src/inc/peanut_gb.h](https://github.com/dodger-one/PicoDMG/blob/master/src/inc/peanut_gb.h#L1451-L1465)
- [src/inc/peanut_gb.h](https://github.com/dodger-one/PicoDMG/blob/master/src/inc/peanut_gb.h#L3422-L3430)

And the frontend still exposes it at boot/runtime:

- default flag:
  - [src/firmware/main.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/main.c#L115-L117)
- boot initialization:
  - [src/firmware/main.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/main.c#L1463-L1469)
- serial toggle:
  - [src/firmware/main.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/main.c#L1631-L1637)

That distinction mattered a lot during testing:

- `frame_skip` could make benchmark numbers look spectacular while cheating in obvious ways
- `interlace` was a more honest tradeoff that reduced visible work without turning the emulator into fast-forward

### FPS impact:

- Interlace: 59.75 FPS 100% of the time, but with heavy visual artifacts. Not worth the tradeoff.
- Frame Skip: +100~200FPS -> fast forward mode
- Nothing useful but experience

## Step 7: Reject the Optimizations That Only Looked Good on Paper

Not every experiment survived.

This is probably the most important part of the story.

During this phase there were several attempts to improve heavy scenes with increasingly aggressive heuristics:

- alternate display dropping
- adaptive display skipping
- queue-depth tuning
- other "cheap tricks" around presentation timing

The exported thread shows the eventual conclusion very clearly: those heuristics changed some numbers, but they did not converge toward a clean solution. They sometimes improved calm scenes, but they did not reliably fix the hard workloads.

That realization was valuable because it prevented the project from turning into a pile of unstable hacks.

The real gains came from:

- transport improvements
- better pipeline structure
- honest instrumentation
- understanding what the emulator core and frontend were actually doing

Not from inventing endless new skip modes.

## Step 8: Push the RP2350 Until It Complains

By this point, most of the obvious bottlenecks had already been eliminated. The LCD path had been redesigned, DMA batching was in place, SPI throughput had been improved, and the emulator could finally be profiled instead of guessed about.
Yet a small gap remained.
Depending on the scene, the emulator was still missing roughly `5-8 FPS` from the target.

At that point there was only one remaining question:

> Was the RP2350 simply running out of CPU budget?

There was an easy way to find out. The current code still reflects that phase:

* [src/firmware/main.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/main.c#L121-L128)
* [src/firmware/main.c](https://github.com/dodger-one/PicoDMG/blob/master/src/firmware/main.c#L1304-L1317)

The emulator raises the voltage rail, programs the PLL, and reports the resulting system clock during boot.

What followed was not particularly scientific.

The clock was increased.  
Then increased again.  
And again.  
And again.

Until eventually one of two things would happen:

* the emulator would reach full speed
* the board would stop booting

Fortunately, the emulator won.

### FPS impact:

* Final gain: approximately +5~8 FPS
* Result: stable `59.75 FPS`

The interesting part is not that overclocking worked.

The interesting part is that it only worked because most of the other bottlenecks had already been removed. Earlier in the project, additional clock speed would simply have made the system hit the same limitations slightly faster.

At this stage, however, the remaining bottleneck was finally what everyone expected from the beginning:

CPU performance.

## The Result

By the end of that ST7789-on-RP2350 phase, the project had gone from a barely convincing first port to a build capable of maintaining approximately `59.75 FPS` essentially all the time in the target configuration.

For the first time, the PicoDMG stopped feeling like a promising prototype and started feeling like a real Game Boy.

That result did not come from one miracle patch.

It came from a stack of cumulative changes:
- overclock the RP2350 aggressively enough
- configure the ST7789 path for higher SPI throughput
- keep LCD work on core 1
- queue scanlines instead of blocking immediately
- batch panel writes into DMA-backed bands
- measure the cost of the emulator, LCD submission, LCD core, and audio separately
- use interlace as a controlled tradeoff instead of abusing frame skip

In other words: the path to full speed was not "make one thing faster".

It was:

> Turn the emulator frontend into a system that could actually be profiled, reasoned about, and improved.

## What Comes Next

This was only the first phase of the search for speed.

Once the obvious ST7789 transport wins were in place, the remaining slow scenes became much more interesting, because they were no longer explained by a single easy bottleneck. That is where the work started shifting from broad structural fixes into deeper profiling and harder tradeoffs.

That will be the subject of Part 2.
