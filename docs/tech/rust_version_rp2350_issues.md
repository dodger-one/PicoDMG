# Rust RP2350 Version: Technical Postmortem

<!-- vim-markdown-toc GFM -->

* [Description](#description)
* [Initial decision](#initial-decision)
* [Early bring-up work](#early-bring-up-work)
* [Console and observability problems](#console-and-observability-problems)
* [Display bring-up: the biggest practical problem](#display-bring-up-the-biggest-practical-problem)
    * [What was observed](#what-was-observed)
    * [What was tried](#what-was-tried)
    * [What the results looked like](#what-the-results-looked-like)
* [Performance measurements](#performance-measurements)
    * [Technical interpretation](#technical-interpretation)
* [Main debugging strategy that was used](#main-debugging-strategy-that-was-used)
* [Why I abandoned the Rust path](#why-i-abandoned-the-rust-path)
    * [Practical reasons](#practical-reasons)
    * [Technical reason in one sentence](#technical-reason-in-one-sentence)

<!-- vim-markdown-toc -->

## Description

This document summarizes what happened during the attempt to move the project from the [YouMakeTech RP2040 C/Pico-SDK](https://github.com/YouMakeTech/Pico-GB) version of the emulator to the [Altaflux Rust-based RP2350](https://github.com/Altaflux/gb-rp2350) version, what was tried technically, what worked, what did not, and why that path was abandoned.

This is not a language-level judgment against Rust. The problem was practical: the Rust RP2350 version did not converge to a usable emulator fast enough, and the working C codebase later proved to be a better foundation for the actual hardware goals.

## Initial decision

The original plan was:

- stop evolving the old RP2040 code as the main target
- move to RP2350 / Pico 2 hardware for more headroom
- use an existing Peanut-GB-based Rust RP2350 emulator as the new base

The assumption was reasonable:

- RP2350 is more capable than RP2040
- there was already a Rust RP2350 emulator candidate
- in theory this should have provided a cleaner long-term platform

## Early bring-up work

The first phase was not about emulator accuracy. It was basic hardware bring-up and board adaptation.

Main goals in that phase:

- adapt DMG-LCD-06 button matrix support
- define and refine RP2350 pinout
- wire TFT, SD, audio, and later NFC expectations
- get visibility through LED/UART/USB CDC
- make the Rust firmware build cleanly

## Console and observability problems

One of the early frictions was that debugging visibility was weak.

Symptoms seen during this stage:

- no obvious response from the onboard LED in emulator mode
- no reliable UART output on `GP0/GP1`
- CDC/USB debug becoming the practical fallback

This matters because it slowed every other debugging step. Before display or emulator issues could be trusted, the board needed a known-good way to show status.

Observed sequence:

1. UART was tried first and did not provide reliable output.
2. Dedicated LED smoke testing proved the board itself was alive.
3. USB CDC became the only way for runtime diagnostics.

This was not the final reason for abandoning the Rust branch, but it increased debugging cost significantly.

## Display bring-up: the biggest practical problem

The largest technical problem on the Rust RP2350 path was the TFT/display path.

### What was observed

Multiple repeated failure patterns appeared:

- white screen with no image
- wrong colors / pastel / reversed-looking output
- rolling or unstable display behavior
- low frame rate even when the screen was not rendering correctly
- cases where performance improved but the display still remained white

These symptoms came up across several tests with both small and larger SPI TFTs.

### What was tried

- move to the "standard" SPI pinout matching RP2040 expectations
- port working TFT test logic from the RP2040 branch into Rust RP2350 test programs
- compare behavior between native Rust TFT tests and tests migrated from the RP2040 implementation
- investigate color order, inversion, and panel-specific configuration
- try exact-display / alternate display-path variants
- repeatedly rollback and retest single display changes atomically
- isolate known-working SPI paths and compare FPS against white-screen cases

### What the results looked like

The branch produced several characteristic states:

- `white screen + ~30 FPS`
- `white screen + ~18 FPS`
- `white screen + ~15 FPS`
- `working image + ~10 FPS`
- `working image + ~7.6 FPS`

Those numbers are critical because they explain the final decision.

The problem was not only "the panel is white".
The problem was that even when the display path could be made to show something, the emulator was still far from usable.

## Performance measurements

Very bad early state:

```text
FPS: 0.09 | frames=3 elapsed_ms=32651
FPS: 0.09 | frames=3 elapsed_ms=32787 | last_frame_ms=32708
```

Still unusable after improvement:

```text
FPS: 7.59 | frames=8 elapsed_ms=1054 | last_frame_ms=131
```

With some changes disabled / isolated:

```text
FPS: 18.11 | frames=19 elapsed_ms=1049 | last_frame_ms=55
FPS: 17.96 | frames=18 elapsed_ms=1002 | last_frame_ms=56
```

Best case during atomic rollback-style testing:

```text
FPS: 29.82 | frames=30 elapsed_ms=1006 | last_frame_ms=30
```

Known-working hardware SPI path, but still far too slow:

```text
FPS: 10.07 | frames=11 elapsed_ms=1092 | last_frame_ms=98
```

### Technical interpretation

These numbers exposed two facts:

1. The bottleneck was not one simple configuration error.

- If it had been one wrong pin or one wrong display mode, fixing it would have produced a step-function result.
- Instead, different combinations traded between:
  - no image
  - wrong image
  - better FPS but no display
  - visible display but unusable FPS

2. Even the best observed runtime behavior was not close to acceptable gameplay.

- A Game Boy target needs roughly `59.7 FPS` for full-speed behavior.
- The Rust RP2350 branch spent most of its time in the `7-30 FPS` range.
- Even the best `~30 FPS` state was only about half of the target.

## Main debugging strategy that was used

I drove the testing in a disciplined way after the initial chaos:

- rollback to a more stable point
- perform one change at a time
- keep a separate rollback directory/branch to preserve baselines

It revealed an important technical conclusion:

- there was no single isolated change that moved the Rust RP2350 emulator from broken to usable
- instead, the branch had multiple interacting problems in display, runtime integration, and overall performance

## Why I abandoned the Rust path

I did not abandon the Rust RP2350 version because of ideology. I give up because It was unusable.

### Practical reasons

1. The old RP2040 C firmware was performing much better out of the box.
2. The Rust RP2350 branch required heavy bring-up work before it even reached comparable integration quality.
3. The Rust RP2350 branch remained too slow for acceptable gameplay.
4. Display behavior remained unreliable during much of the investigation.
5. Every additional fix had a high validation cost because basic hardware feedback was already expensive.

### Technical reason in one sentence

The Rust RP2350 emulator was abandoned because it never converged to both:

- correct display behavior
- acceptable emulation speed

at the same time.
