---
title: Before you flash
---

# Before you flash

MimiBoot-Mu sits at the start of flash, so a bootloader that misbehaves can leave a board
that doesn't start anything. Every chip it supports has a way back that doesn't depend on
the bootloader: a debug probe, a ROM loader or an ISP programmer. Have that way back
ready before you flash, especially on boards nobody has tried yet.

How far each board has been tested is on the [feature matrix](matrix.html). In short:
earlier builds ran on real RP2040 and RP2350 boards; everything else has run in an
emulator, against a register model, or has only been compiled.

## STM32: keep an ST-Link on SWD

<div class="warn-box">
<strong>No STM32 flash write has ever run on a real chip.</strong> QEMU has no STM32
flash controller, so the erase and program drivers (F0/F1/F3, F2/F4/F7, L4/G0/G4) are
written from the reference manuals and have never executed. Uploads, the boot log,
confirming an image and the rollback floors all write flash. The boot path itself
(reading flash, checking images, starting the app) has run in QEMU on the F405 and F100.
</div>

Until your board has been tried:

- **Wire SWD.** A Nucleo or Discovery board has an ST-Link on board. A Blue Pill or Black
  Pill needs an external one (an ST-Link V2 or a clone works) on SWDIO, SWCLK, GND and,
  if you can, NRST.
- **Check that you can connect under reset.** In STM32CubeProgrammer pick *Mode: Under
  reset* (with NRST wired) or *Hot plug*, connect, and try a full chip erase once before
  you need it. If the bootloader turns off the debug pins or crashes early, connecting
  under reset still works because the chip is held before any code runs.
- **Know where BOOT0 is.** On most families, holding BOOT0 high at reset starts the
  chip's own ROM loader (UART, and USB DFU on chips that have it), whatever the flash
  holds. On a Blue Pill that's the BOOT0 jumper; on a Nucleo it's a pin on the morpho
  header. **STM32G0 is different:** from the factory it takes BOOT0 from an option bit
  (`nBOOT_SEL` set), not the pin, and the pin doubles as SWCLK. On a G0, SWD is your
  way back.
- **Leave readout protection alone.** `mbtool provision` prints the option-byte commands
  for write protection and RDP; it never runs them. RDP level 1 can be undone (going back
  to level 0 erases the whole chip). **RDP level 2 can't be undone, ever**, and it also
  closes SWD for good.
- **Write protection (WRP)** on the bootloader's sectors stops anything writing there,
  including a debugger, until you clear the option bytes. Clear them in CubeProgrammer if
  you need to reflash the bootloader.

If something goes wrong: connect under reset, mass-erase, flash a known-good image (or
your old firmware) and report what happened. A board stuck in the MimiBoot menu is fine:
the menu is the bootloader working, and the console can upload a fixed image.

## nRF51 and nRF52

nRF51 builds ran in QEMU, including uploads written through the real flash driver. On
nRF52 the UARTE console, the crystal start, APPROTECT, the watchdog, GPIO port 1 and VTOR
have never run.

- **Use an SWD probe**: the J-Link on a Nordic DK, or any CMSIS-DAP probe with pyOCD or
  OpenOCD. `nrfjprog --recover` (or `pyocd erase --chip`) erases the whole chip and
  reopens a locked debug port.
- **Save the factory contents first** if you want them back. MimiBoot replaces whatever
  is at address 0: Nordic's MBR and SoftDevice, or Adafruit's UF2 bootloader.
- **`DEBUG_LOCK` closes the debug port** until a full erase. Leave it off while you're
  trying a board.
- **`BOOT_LOCK` protects the bootloader's flash until reset**, also from a debugger on
  nRF52805/810/811/832. A reset (or `--recover`) gets you past it.

## SAM D21 and D51

The flash driver ran against a model of each NVM controller; clocks, SERCOM, PORT, the
watchdog and USB never ran on a chip.

- **Use an SWD probe**: an Atmel-ICE, a J-Link, or a CMSIS-DAP probe with OpenOCD.
- **Clear BOOTPROT first.** Most boards ship with it set to protect their UF2 or SAM-BA
  bootloader (OpenOCD: `at91samd bootloader 0`, then reset).
- **Save the factory bootloader** if you want it back. Once it's gone the board's
  double-tap-reset UF2 drive is gone too, until you flash it again by SWD.
- **The Security Bit** closes the debug port. Only a chip erase clears it. `mbtool
  provision` prints the commands; it never runs them.

## RP2040 and RP2350

Earlier builds ran on real Pico and RP2350 boards; this configurable version has not been
flashed yet.

- **BOOTSEL always works.** Hold BOOTSEL while plugging in and the boot ROM's UF2 drive
  appears, whatever the flash holds. The bootloader can also be asked to go there
  (`MB_REQ_BOOTSEL`, or `bootsel` on the console).
- **Never burn OTP on the RP2350.** Secure boot, debug disable and the boot key slots live
  in one-time-programmable fuses. Nothing in MimiBoot writes OTP, and nothing should:
  a wrong fuse can't be undone.

## AVR (Arduino Uno, Nano, Mega and friends)

The Uno build ran in simavr with the real avrdude. The other AVRs only compile, and the
large-flash paths (RAMPZ/ELPM on the 1284P and 2560) never ran.

- **Keep an ISP programmer at hand**: a USBasp, an AVRISP, or a second Arduino running
  ArduinoISP. ISP can always rewrite flash and fuses, whatever the bootloader does.
- **Write down your fuses** (`avrdude -U hfuse:r:-:h`) before you change them. The
  bootloader needs BOOTSZ and BOOTRST set; the build prints the value.
- **Leave the lock bits alone** while trying a board. A chip erase over ISP clears them.
- The default `reset-pin` entry and the 8 MHz Pro Mini build haven't run anywhere.

## For every board

- Start with the **hobby** preset or the board's defaults: no signatures required, no
  locks, the console on. Turn on `REQUIRE_SIGNATURE`, `BOOT_LOCK`, `DEBUG_LOCK` and
  `CONSOLE_AUTH` once the basics work on your board.
- If you require signatures, check that one signed image boots before you lose your way
  back. A bootloader that refuses everything still opens its menu, but it won't start an
  unsigned recovery image.
- Keep the build folder (`~/MimiBoot/<board>/`): `layout.txt`, the config header and the
  exact `.config` say what is on the chip.
- **Tell us how it went.** A board that boots, or doesn't, is the most useful report there
  is: open an issue on the [repository](https://github.com/MilkmanAbi/MimiBoot-Mu) with the
  board, the `.config` and what the console printed.
