---
title: Getting started
---

# Getting started

## What you need

- Python 3.11 or newer.
- A copy of the repository: `git clone https://github.com/MilkmanAbi/MimiBoot-Mu`.
- The compiler for your chip:

| Chip | Toolchain |
|---|---|
| STM32, nRF, SAM D | [Arm GNU Toolchain](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads) (`arm-none-eabi-gcc`) |
| AVR | avr-gcc. The one the Arduino IDE or arduino-cli installs is found automatically. |
| RP2040 | the Pico SDK (set `PICO_SDK_PATH`), plus the Arm toolchain |
| RP2350 | the Pico SDK, picotool and a RISC-V toolchain (the build makes an Arm + RISC-V image) |

`python mimiconf.py toolchains` shows what it found. If something lives in an odd place,
point it there with `MU_ARM_GCC`, `MU_AVR_GCC`, `PICO_SDK_PATH`, `PICOTOOL` or
`RISCV_TOOLCHAIN`.

On Windows the menu needs curses: `pip install windows-curses`. Everything also works
from the command line without it.

## Your first build

```
cd MimiBoot-Mu
python mimiconf.py
```

The first screen lists chip makers. Pick one, then a category, then your board. You land
on the home screen, where you can change options or just choose **Build now**. The
result goes to `~/MimiBoot/<board>/`.

The same without the menu:

```
python mimiconf.py defconfig nucleo-f411re
python mimiconf.py build
```

`python mimiconf.py boards` lists every board name.

## What the build gives you

STM32, nRF and SAM D:

| File | What it is |
|---|---|
| `mimiboot.bin`, `.hex`, `.elf` | the bootloader |
| `layout.txt`, `layout.json` | the flash map: where each slot is and where to link apps |
| `app-slot-a.ld` ... | linker `MEMORY` snippets for apps in each slot |
| `zephyr-partitions.overlay` | a devicetree overlay for Zephyr apps |
| `client.txt` | how to compile the app library into your firmware |
| `build-info.json` | sizes and the options that went in |

AVR: `mimiboot-lite.hex`, the fuse value for the boot section, and a `boards.txt` entry
with the right `upload.maximum_size` for the Arduino IDE.

RP2040/RP2350: `.uf2`, `.bin` and `.elf` files, as the Pico SDK makes them.

## Flashing the bootloader

Read [Before you flash](before-you-flash.html) first. On STM32 especially: no flash write
has run on a real chip yet, so keep an ST-Link on SWD and check you can connect under
reset before you start.

STM32, nRF and SAM D: write `mimiboot.bin` to the start of flash with an SWD probe
(ST-LINK, J-Link, a DK's on-board debugger, a CMSIS-DAP probe, OpenOCD, pyOCD). Some
things to know:

- On nRF and SAM D this replaces the factory bootloader at address 0 (Nordic's MBR and
  SoftDevice, Adafruit's UF2 bootloader, SAM-BA). Save a copy first if you want it back.
- SAM D boards usually ship with the BOOTPROT fuse set. Clear it first, for example with
  OpenOCD's `at91samd bootloader 0`.
- nRF52 firmware built for Nordic's SoftDevice (the nRF5 SDK, the Adafruit and Seeed
  Arduino cores) won't run without the MBR. Zephyr, nrfx and FreeRTOS firmware will.

AVR: the bootloader needs its boot section and BOOTRST set in the fuses, once, with an ISP
programmer. The build prints the fuse value. See [AVR](avr.html).

RP2040/RP2350: hold BOOTSEL while plugging in and copy the `.uf2` over, or use picotool.

## Then put an app on it

[Images and flashing](images.html) explains how to link an app for a slot, wrap it into
an image and install it, either with the debugger or over the bootloader's console.
