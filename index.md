---
title: Overview
---

# MimiBoot-Mu

MimiBoot-Mu is a bootloader for small microcontrollers: RP2040 and RP2350, STM32,
Nordic nRF51 and nRF52, Microchip SAM D21 and D51, and classic AVRs such as the
ATmega328P on an Arduino Uno.

It is configured like a Linux kernel. A menu (`python mimiconf.py`) lets you pick a
board and switch features on or off, and the build contains only what you picked.
The configuration is a small text file, so you can change it and rebuild whenever
your needs change.

Source: [github.com/MilkmanAbi/MimiBoot-Mu](https://github.com/MilkmanAbi/MimiBoot-Mu)

## Why it exists

Most bootloaders are one fixed design for one chip family: Optiboot on AVR, the UF2
bootloaders on SAMD and nRF52, MCUboot for Zephyr, the ROM loaders chips come with.
They're good at what they do. If you need something they don't do, or want to remove
something to save space, you end up patching them.

MimiBoot-Mu puts every feature behind a build option, and the same options exist on
every chip that can support them. A build on an Uno can be under 1 KB and do nothing
but start the app. A build on an STM32 or nRF52 can do signed A/B updates with rollback,
a boot menu, uploads over a serial console and requests from the running OS, in about
34 KB. Anything in between is a matter of flipping options.

## Two cores

The **full core** runs on the 32-bit chips. It handles slots, image checks, signatures,
the console, the boot config and the OS interface. On RP2040/RP2350 it builds with the
Pico SDK; on STM32, nRF and SAM D it is bare metal with no vendor SDK.

The **lite core** runs on AVR. It speaks STK500 like Arduino's own bootloader, so the
Arduino IDE and avrdude keep working, and adds an app check, trial boots, a flash-write
service and staged updates.

## Where to start

- [Getting started](getting-started.html): toolchains, the first build, flashing it.
- [Configuring a build](configuring.html): the menu, the command line, presets.
- [Features](features.html): what each part does, in plain words.
- [Option reference](options.html): every option, its default and its cost in flash.
- [Boards](boards.html): what's supported and how well tested it is.
- [Test status](status.html): read this before flashing anything you can't easily recover.

## Status

Earlier builds of the RP2040/RP2350 ports have run on real boards. The STM32, nRF,
SAM D and AVR code has run in emulators (QEMU, simavr) or against models, or has only
been compiled. Nothing on those families has run on silicon yet. The
[test status](status.html) page says exactly what ran where.
