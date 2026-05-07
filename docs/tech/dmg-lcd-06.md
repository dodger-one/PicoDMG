---
layout: default
title: DMG-LCD-06 Tech Note
---

# DMG-LCD-06 Button Matrix Technical Note

## Scope

This document explains how the original Nintendo Game Boy `DMG-LCD-06` board handles the buttons, which ribbon pins are used by this project, how they are connected to the RP2350, and where the compatibility logic lives in the source code.

This note is only about **button input**.
The original LCD glass, contrast circuit, audio path, and other functions of the `DMG-LCD-06` board are outside the scope of this document.

## Background

The original DMG does **not** wire one GPIO per button.
Instead, it uses a **2-row by 4-column button matrix**.

The matrix lines exposed by the `DMG-LCD-06` ribbon are:

- `P10`
- `P11`
- `P12`
- `P13`
- `P14`
- `P15`

In the original hardware:

- `P14` and `P15` are the **row select** lines
- `P10..P13` are the **column read** lines

The CPU selects one row at a time and then reads back which columns are pulled low by pressed buttons.

## How the DMG Button Matrix Works

### Matrix layout

The project follows the original Game Boy joypad matrix behavior:

- `P14` low selects the **D-pad row**
- `P15` low selects the **button row**

When `P14` is low:

- `P10` = Right
- `P11` = Left
- `P12` = Up
- `P13` = Down

When `P15` is low:

- `P10` = A
- `P11` = B
- `P12` = Select
- `P13` = Start

### Active-low behavior

All lines are handled as **active-low**.

That means:

- idle / not pressed = logic high
- pressed = logic low

This is why the firmware configures the read lines with pull-ups and then looks for a low state during scanning.

## Ribbon Pinout Used By This Project

Only the ribbon pins needed for the button matrix are used here.

Project wiring by ribbon position:

| Ribbon pin | DMG signal | Role        |
| ---------- | ---------- | ----------- |
| `4`        | `P11`      | Column read |
| `5`        | `P14`      | Row select  |
| `6`        | `P13`      | Column read |
| `7`        | `P12`      | Column read |
| `8`        | `P10`      | Column read |
| `9`        | `P15`      | Row select  |

## RP2350 Pin Assignment Used By The Project

On the RP2350 build used by this project, the `DMG-LCD-06` matrix is connected like this:

| RP2350 GPIO | DMG signal | Function    |
| ----------: | ---------- | ----------- |
|       `GP2` | `P11`      | Column read |
|       `GP3` | `P14`      | Row select  |
|       `GP4` | `P13`      | Column read |
|       `GP5` | `P12`      | Column read |
|       `GP6` | `P10`      | Column read |
|       `GP7` | `P15`      | Row select  |

This mapping is intentionally arranged to match the physical ribbon order used during bring-up, not the numeric order of `P10..P15`.

References:

- `src/firmware/main.c:89`

## Button Mapping Summary

Once the matrix is scanned, the project resolves the buttons like this:

| Selected row | Read line | Button |
| ------------ | --------- | ------ |
| `P14 = low`  | `P10`     | Right  |
| `P14 = low`  | `P11`     | Left   |
| `P14 = low`  | `P12`     | Up     |
| `P14 = low`  | `P13`     | Down   |
| `P15 = low`  | `P10`     | A      |
| `P15 = low`  | `P11`     | B      |
| `P15 = low`  | `P12`     | Select |
| `P15 = low`  | `P13`     | Start  |

Reference:

- `src/firmware/main.c:294`

## Firmware Scanning Logic

The firmware compatibility layer lives in `src/firmware/main.c`.

### 1. Compile-time enable switch

DMG matrix mode is disabled by default and must be enabled at build time.

CMake option:

- `RP2040_GB_USE_DMG_BUTTON_MATRIX=ON`

This option is converted into the compile-time define:

- `USE_DMG_BUTTON_MATRIX=1`

References:

- `src/CMakeLists.txt:11`
- `src/CMakeLists.txt:149`

### 2. GPIO definitions

When matrix mode is enabled, the project replaces the normal direct button GPIO mapping with the DMG matrix mapping.

Reference:

- `src/firmware/main.c:89`

### 3. GPIO initialization

The initialization code configures the matrix as follows:

- `P10..P13` as inputs
- pull-ups enabled on `P10..P13`
- `P14` and `P15` as outputs
- idle state sets both `P14` and `P15` high

This creates the standard idle matrix state with no row selected.

References:

- `src/firmware/main.c:328`
- `src/firmware/main.c:345`

### 4. Runtime scan sequence

The scan function is `read_button_states(...)`.

It performs two passes:

#### Pass 1: D-pad

- set `P14 = 0`
- set `P15 = 1`
- wait `5 us`
- read `P10..P13`

#### Pass 2: A/B/Select/Start

- set `P14 = 1`
- set `P15 = 0`
- wait `5 us`
- read `P10..P13`

Then it restores the idle state:

- `P14 = 1`
- `P15 = 1`

References:

- `src/firmware/main.c:290`
- `src/firmware/main.c:298`
- `src/firmware/main.c:306`
- `src/firmware/main.c:314`

## Why This Compatibility Layer Exists

The upstream emulator code expects either:

- direct per-button GPIO wiring
- or another platform-specific input scheme

The original `DMG-LCD-06` board does not present the buttons that way.
It presents a shared matrix that must be scanned.

This project adds a hardware adaptation layer so the original DMG board can still be used for input while the emulator runs on RP2040/RP2350 hardware.

In practical terms, this layer makes the original button board behave like a normal emulator joypad API.

## Source Code Reference Map

These are the most relevant source locations for DMG-LCD-06 button compatibility:

### Build-time option

- `src/CMakeLists.txt:11`
- `src/CMakeLists.txt:149`

### GPIO mapping

- `src/firmware/main.c:89`

### Matrix scan implementation

- `src/firmware/main.c:290`

### Matrix GPIO initialization

- `src/firmware/main.c:328`

### ROM selector input path using matrix scan

- `src/firmware/main.c:388`

### Main gameplay input path using matrix scan

- `src/firmware/main.c:1522`

## Build Example

To enable `DMG-LCD-06` matrix input in a build:

```bash
cmake -S . -B build_pico2 -G Ninja \
  -DPICO_BOARD=pico2 \
  -DRP2040_GB_USE_DMG_BUTTON_MATRIX=ON
cmake --build build_pico2 -j --target RP2040_GB
```

## Practical Wiring Notes

- Only connect the `P10..P15` lines if you are using the `DMG-LCD-06` only as a button board.
- Do not expect the firmware to work if the row and column lines are swapped.
- Do not remove the pull-up behavior in firmware unless the external circuit has been redesigned.
- The scan logic assumes the original Game Boy active-low button matrix behavior.

## External Reference

Original schematics reference used by the project:

- DMG schematics: `https://github.com/Gekkio/gb-schematics/tree/main/DMG-LCD-06`
