---
layout: post
title: "Episode 05 - Expensive Mistakes"
author: dodger-one
tags:
  - rp2040
  - rp2350
  - dmg
  - hardware
  - fail
---

# Expensive Mistakes

This episode will be short in terms of content, but very, very long in terms of actual time spent...

I will cover several different stages of the development process so there is only one "Mistakes" episode.

Maybe two. :rofl:

## PAM8302

By mistake, I purchased the famous PAM8302 amplifier. At first, I thought it would be a great option for the audio subsystem of the PicoDMG.

Until I realized that the emulator only supports the MAX98357A for audio output.

The MAX98357A decodes digital audio, while the PAM8302 only amplifies an analog signal.

So I put the PAM8302 aside and ordered a MAX98357A instead. :lol:

<div align="center">
  <img src="{{ site.baseurl }}/media/episode05/IMG20260317174203.webp" width="400" alt="PAM8302">
</div>

## Audio jack

Still on the audio path, I purchased an audio jack that seemed perfect for fitting inside the original Game Boy shell.

<div align="center">
  <img src="{{ site.baseurl }}/media/episode05/IMG_20260419_212138.webp" width="400" alt="Audio Jack">
</div>

When it arrived, and after a meticulous inspection, I realized it didn't include the physical switching mechanism used by the original Game Boy audio jack.

That means if I wire it on the same bus as the speaker, I will end up with audio coming out of both the speaker and the headphones at the same time... xD

I also put this aside together with the PAM8302 for future projects. :lol:

## Cracking screens

Because shitz happenz... :-(

<div align="center">
  <img src="{{ site.baseurl }}/media/episode05/IMG20260406220959.webp" width="400" alt="TFT cracked">
</div>

During my early RP2350 tests I noticed a strange line across the TFT.

After a closer inspection, I discovered the problem.

I had cracked the display. :-(

## Switch from RP2040 to RP2350 (Pico 2)

This wasn't really a mistake.

But it generated an enormous amount of wasted time.

<div align="center">
  <img src="{{ site.baseurl }}/media/episode05/IMG20260401183102.webp" width="400" alt="RP2350">
</div>

### Pico 2 debugging

First debugging attempt using the embedded UART port: FAIL.

<div align="center">
  <img src="{{ site.baseurl }}/media/episode05/IMG20260404112726.webp" width="400" alt="RP2350">
</div>

Second debugging attempt using the default GPIO UART port: FAIL.

<div align="center">
  <img src="{{ site.baseurl }}/media/episode05/IMG20260404222522.webp" width="400" alt="RP2350">
</div>

At this point I was starting to suspect the problem might not be the hardware...

### Pico 2 screen "crap"

This is the holy grail of mistakes in this project.

In fact, it was such a disaster that I ended up writing a separate technical postmortem about it.

Along with the RP2040 → RP2350 migration, I also decided to move from the original C/Pico-SDK version of the emulator based on the excellent [YouMakeTech Pico-GB](https://github.com/YouMakeTech/Pico-GB) project to the newer Rust-based RP2350 implementation from [Altaflux](https://github.com/Altaflux/gb-rp2350).

Since it was already running on the RP2350 and the codebase looked more modern than the original C implementation, it seemed like a logical step forward.

The first challenge wasn't getting the emulator running.

It was getting *anything* on the display.

I was only getting a completely white screen, like the one shown earlier in the cracked screen section.

The first thing I did was rewire everything.

<div align="center">
  <img src="{{ site.baseurl }}/media/episode05/IMG20260413221730.webp" width="400" alt="re-wire">
</div>

Just to eliminate any possible wiring issues.

Then I spent several days running tests (see the technical document for details), and eventually I managed to get the display smoke test working.

<div align="center">
  <img src="{{ site.baseurl }}/media/episode05/IMG20260417175912.webp" width="400" alt="RP2350 display smoke test">
</div>

<div align="center">
  <img src="{{ site.baseurl }}/media/episode05/IMG20260417180135.webp" width="400" alt="RP2350 display smoke test">
</div>

After applying those settings to the emulator, I finally got the ROM selection menu on screen.

<div align="center">
  <img src="{{ site.baseurl }}/media/episode05/IMG20260417181245.webp" width="400" alt="rom menu">
</div>

And finally, the greatest Game Boy game ever made booted.

<div align="center">
  <img src="{{ site.baseurl }}/media/episode05/IMG20260417182910.webp" width="400" alt="Tetris booting">
</div>

But then came the biggest WTF moment of the entire process.

What happened to the music?

I added additional debug information to the CDC console and quickly discovered the problem.

The emulator was running *extremely* slowly.

I was getting around **10 FPS**.

<iframe
  title="RP2350 + Rust Version Running at 10 FPS"
  width="560"
  height="560"
  src="https://gnulinux.tube/videos/embed/5ozVUFm9HpvbR136yRPart"
  frameborder="0"
  allowfullscreen>
</iframe>

Can you imagine trying to play a Game Boy game at 10 FPS?

Just to be sure, I checked the original Game Boy frame rate again.

Around **60 FPS**.

I didn't give up immediately.

I spent almost a full week testing different optimizations, trying alternative display configurations, reverting changes, profiling code, reading documentation and generally chasing ghosts.

Eventually I accepted reality.

The Rust version simply wasn't performing well enough for my use case.

I rolled back to the original C/Pico-SDK implementation and immediately got around **30 FPS** on the RP2040 without any kind of optimization, which was already significantly better than the ~10 FPS I was getting from the Rust version on the RP2350.

Almost two weeks spent on this adventure. :rofl:

In the end, I learned an important lesson.

Just because a codebase is newer doesn't mean it is better suited for your project.

After nearly two weeks of debugging, testing and frustration, I decided to go back to the C/Pico-SDK version and continue moving forward.

Sometimes the fastest way forward is admitting that a detour was a mistake.

## Related Technical Documentation

- [Rust RP2350 Version: Technical Postmortem]({{ site.baseurl }}/tech/rust_version_rp2350_issues.html)
