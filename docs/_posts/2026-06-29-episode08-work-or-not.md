---
layout: post
title: "Episode 08 - Everything Worked, Until It Didn't"
author: dodger-one
tags:
  - rp2350
  - st7789
  - performance
  - emulator
  - technical
---

## Everything Worked, Until It Didn't

Because sometimes the inception comes from yourself ;-)

## Acto 1: the idea

As with many bad ideas, it started with a simple question:

> What if the PicoDMG could identify physical cartridges using NFC tags?

A few minutes later, an AliExpress order had already been placed. 🤣🤣🤣

After some searching, I found what looked like the perfect NFC reader: small enough to fit inside the DMG shell and, more importantly, with the antenna located on the opposite side of the PCB. That detail was critical, because my plan was to place the reader as close as physically possible to the cartridge slot.

The idea was simple:

- attach an NFC tag to each physical cartridge
- scan the cartridge when inserted
- automatically boot the corresponding ROM

In other words: make the emulator behave as if it were actually reading the cartridge.

So, after the usual cycle of thinking, wiring, coding... and repeating that process many more times, I finally got the first proof of concept working:

<iframe title="Episode 08: First NFC tests" width="560" height="315" src="https://gnulinux.tube/videos/embed/gaTRFKJJ3j8ykEArWXw7En" allow="fullscreen" sandbox="allow-same-origin allow-scripts allow-popups allow-forms" style="border: 0px;"></iframe>

For anyone interested in the technical details, the complete implementation can be found in the [NFC workflow](tech/nfc.md) document.

At this point, everything looked extremely promising.

## Acto 2: The moment I Almost Cried

After months of soldering, debugging, and rewriting code, there comes a point where you stop believing the project will ever actually work.

This was one of those moments.

I had finally reached the point where everything needed to come together:

the RP2350
the display
the audio
the NFC reader
and, most importantly, the real AA battery power system

I installed the three AA batteries, closed the shell as best as I could, and pressed the power switch.

<iframe title="Episode 08: The moment I Almost Cried" width="560" height="315" src="https://gnulinux.tube/videos/embed/6PTsMYhrQd47uzR1y3a2QL" allow="fullscreen" sandbox="allow-same-origin allow-scripts allow-popups allow-forms" style="border: 0px;"></iframe>

I think the audio says everything.

That moment of absolute happiness, after months of soldering, debugging, and writing code, is difficult to describe.

For the first time, the PicoDMG stopped feeling like a collection of parts and started feeling like a real console.

## Acto 3: Reality Strikes Back

Well... apparently not everything was going THAT well. 🤣

### Act 3.1: the switch

As it turned out, I had not aligned the original power switch mechanism correctly with the plastic slider of the DMG shell.

As a result, the console was physically incapable of reaching the ON position.

Which, admittedly, is not the ideal behavior for a portable console.

<iframe title="Episode 08: First Fail" width="560" height="315" src="https://gnulinux.tube/videos/embed/cNH2hB2j4wDZvArHdtYCXx" allow="fullscreen" sandbox="allow-same-origin allow-scripts allow-popups allow-forms" style="border: 0px;"></iframe>

### Act 3.2: the microsd

> But how exactly is this supposed to work if I forgot to insert the microSD card with the ROMs?

After several minutes of confusion, I eventually discovered the root cause:

I had forgotten to insert the microSD card containing the ROMs.

In my defense, expecting an emulator to emulate games without providing any games was, in retrospect, somewhat optimistic.

<iframe title="Episode 08: Second Fail" width="560" height="315" src="https://gnulinux.tube/videos/embed/gzXFZbL2LgJtDjWatuPGii" allow="fullscreen" sandbox="allow-same-origin allow-scripts allow-popups allow-forms" style="border: 0px;"></iframe>

### Act 3.3: The Epic NFC fail

At this point, everything should have been working.

Except it wasn't.

The NFC reader simply could not read the tag.

After some investigation, the culprit became obvious:

The antenna was too far away from the NFC tag.

Apparently, electromagnetic fields have very strong opinions about distance.

<iframe title="Episode 08: Third and Epic Fail" width="560" height="315" src="https://gnulinux.tube/videos/embed/gXEFMmLZZ28jVTPNKiv1SX" allow="fullscreen" sandbox="allow-same-origin allow-scripts allow-popups allow-forms" style="border: 0px;"></iframe>

## Acto 4: The Great Internal Reorganization

So, the problem was not the software.

The problem was not the NFC reader itself.

The problem was where the hell I had decided to place the NFC reader inside the console.

And what did that realization mean?

Well...

* desolder the NFC board completely
* redo part of the wiring
* relocate several components
* take out the Dremel
* cut part of the original Game Boy RF shielding
* and somehow find a layout that would actually work

At this point, the project had entered what I like to call the "Dremel phase".

Which, in my experience, is usually a sign that things have gone horribly wrong.

### Stage 1: Before NFC
<div align="center">
  <img src="{{ site.baseurl }}/media/episode08/IMG20260426221239.webp" width="400" alt="Before NFC">
</div>

### Stage 2: NFC installed
<div align="center">
  <img src="{{ site.baseurl }}/media/episode08/IMG20260427221432.webp" width="400" alt="NFC Installed">
</div>
<div align="center">
  <img src="{{ site.baseurl }}/media/episode08/IMG20260428085515.webp" width="400" alt="NFC Installed Back">
</div>

🤯🤯🤯🤯🤯🤯🤯🤯🤯🤯🤯🤯🤯 Cabling madness 🤯🤯🤯🤯🤯
<div align="center">
  <img src="{{ site.baseurl }}/media/episode08/IMG20260428222346.webp" width="400" alt="Cabling madness">
</div>

### Stage 3: Enters the Dremel

<div align="center">
  <img src="{{ site.baseurl }}/media/episode08/IMG20260430221008.webp" width="400" alt="Final position">
</div>

<div align="center">
  <img src="{{ site.baseurl }}/media/episode08/IMG20260501171240.webp" width="400" alt="Dremel art">
</div>

## Act 5: Victory!

After relocating the NFC reader and rebuilding a significant part of the console's internal layout, the system finally behaved exactly as originally intended.

I could now take a physical cartridge, insert it into the console, and watch the correct game boot automatically.

No menus.

No ROM selection screens.

Just insert the cartridge and play.

Which, honestly, still feels a little bit like magic.

And perhaps that was the whole point of the project from the beginning:

To build something that, for a brief moment, makes your brain forget that there is a microcontroller, an emulator, and several months of debugging hidden inside.

And simply makes you think:

<iframe title="Episode 08: Victory!" width="560" height="315" src="https://gnulinux.tube/videos/embed/hRLWc2L5eHm1FtNTyggQHo" allow="fullscreen" sandbox="allow-same-origin allow-scripts allow-popups allow-forms" style="border: 0px;"></iframe>

> "Yep. That's a Game Boy."
<hr>
