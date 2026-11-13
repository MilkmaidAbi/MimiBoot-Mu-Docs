---
title: Features
---

# Features

This page explains what the full core does, area by area. Every feature here is a build
option; the [option reference](options.html) has the exact names, defaults and costs.
The AVR core is described on its [own page](avr.html).

## Slots

Flash after the bootloader is split into slots. Each slot holds one image.

- **single**: one slot. A new image replaces the old one. There is no fallback, so this
  is the setup for when you just want the bootloader to start your firmware.
- **ab**: two slots, A and B. An update goes into the slot that isn't running. If the new
  image turns out to be bad, the old one is still there.
- **ab+golden**: A and B plus a golden recovery image. The golden slot can only be
  replaced by a newer version, and it boots when nothing else will.

With A/B + golden you can add up to three extra slots for things you boot on purpose
from the menu or the boot config, like a test build or a second OS.

On bare-metal chips the slot sizes come from the chip's flash and sector map. Every
region starts and ends on an erase boundary, so erasing one slot can't touch another.
`python mimiconf.py layout` prints the map. RP boards use fixed layouts for 2, 4 and
16 MB of flash.

Next to the slots there is a small boot log (two flash sectors, written alternately).
It records each image's state: new, on trial, good or bad, how many trial boots are
left, and which A/B slot is preferred.

## Updates, trials and rollback

A new image in an A/B slot can be handled in two ways.

With **Updates must confirm** on, the image gets a few trial boots (3 by default). The
app has to call `mb_client_confirm()` once it has decided it works. If it doesn't, the
bootloader marks it bad after the last try and goes back to the previous good image.
With **Watchdog during trial boots** on, a hang also counts as a failed try. Note that on
STM32 and nRF the watchdog can't be stopped once it's started, so the app has to keep
refreshing it.

With it off, a new image is trusted as soon as its checks pass. This is the default on
bare-metal chips, because most firmware there (stock Zephyr, FreeRTOS, Arduino) knows
nothing about MimiBoot and would never confirm itself.

The **crash-loop guard** watches a confirmed image: after several watchdog resets in a
row it boots the other good image, or golden, instead.

## Image checks

Every image has a header the bootloader can check cheaply. The payload behind it can be
checked three ways (**Check the image hash**):

- **every-boot**: SHA-256 of the whole payload before every start. Catches flash
  corruption. Costs roughly 10 to 100 ms per 100 KB, depending on the chip.
- **once**: the first boot of an image hashes it and records that in the boot log. Later
  boots only check the header. Faster, but corruption that happens later goes unnoticed.
- **never**: header checks only. Images uploaded through MimiBoot are still hashed
  before they're accepted; images written with a debugger are never checked.

## Signatures

With **Ed25519 signatures** on, signed images are verified against the public keys
compiled into the bootloader. By default unsigned images still boot. Turn on **Refuse
unsigned images** and only images signed by one of your keys will start. That setting
requires the hash on every boot, since a valid signature on the header says nothing
about a payload nobody hashed.

**Anti-rollback**: every image carries a security version. Once an image with version N
is confirmed, A/B images below N are refused.

Keys are compiled in. Nothing in flash, the config file or the console can add a key or
switch the check off.

## Image formats

- **MimiBoot images**, made with `mbtool pack`: a 4 KB header (name, version, SHA-256,
  optional signature, security version, where the handoff block goes) and then the
  firmware unchanged.
- **Zephyr / MCUboot images**, signed with imgtool or `west sign`. The SHA-256 TLV and
  an Ed25519 TLV are checked. See [Zephyr and FreeRTOS](zephyr-freertos.html).
- **Raw firmware**: a plain `.bin` with its vector table at the start of a slot. Useful
  for FreeRTOS or Arduino builds you don't want to wrap. There's nothing to check beyond
  a sane stack pointer and reset address.
- **RAM images**: copied to RAM, checked there and started there. The RP2040 port needs
  these; elsewhere they're handy for trying a build without flashing it (`mbtool ramboot`).

## The console

On RP boards the console is USB serial. On STM32, nRF and SAM D it is a UART at 115200
baud by default. It is reached when:

- the board's button is held at reset,
- a key arrives on the UART within the break-in window after reset (300 ms by default),
- the running app asks for it,
- or nothing is bootable.

It starts with a boot menu that counts down and then boots the default entry. Behind the
menu there is a command line for listing slots, booting a particular slot once,
confirming or rejecting images, uploads and the boot config. A set of diagnostic
commands (memory dump, hashing a flash range, the raw boot log) can be left out on its
own, and the command line can be left out entirely, leaving just the menu.

Details: [Console and boot config](console.html).

## Boot config file

A small text file in flash, with GRUB-style entries: a name, a title, which slot,
arguments for the image. It's kept in two flash sectors and written atomically, so a
power cut while saving leaves the old one intact. Without it the bootloader uses
built-in entries (A/B, golden, one per extra slot).

## Talking to the image

The **handoff** is a small block the bootloader writes into the started image's RAM:
which slot and entry it was started from, the argument string, why the chip reset, and
the bootloader's version. It's checksummed and tied to the image, so stale data from an
earlier boot is never mistaken for a fresh one.

The **OS control service** works the other way: the running image can leave a request
for the next boot, like "open the menu", "boot entry X once with these arguments", "boot
the other slot once" or "go to flash mode for an upload".

Both are used through the app library: [App library and handoff](client.html).

## Leaving nothing behind

With **Wipe the bootloader's RAM before handing over** on, the last thing the bootloader
does is zero its own RAM (its data, the copy of the boot log, the config text, its
stack) using only CPU registers, then jump. The app starts with nothing of the bootloader
in RAM. The handoff block lives in the app's own RAM and survives. This is available on
STM32, nRF and SAM D.

Before every jump the bootloader also puts back what it touched: interrupts off and
cleared, SysTick stopped, the UART and pins back to reset state, the clock back to the
reset clock.

## Board hardware

The LED and the button can each be left alone completely. The console baud rate is an
option. On chips without a ROM loader (nRF, SAM D) the menu simply stays up when there's
nothing to boot.
