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

Three kinds of signature, each a build option, any mix of them:

- **Ed25519**: small keys, small signatures, quick to check. The usual choice.
- **ECDSA P-256**: for keys that already exist as P-256, and for Zephyr images signed
  with imgtool's `ecdsa-p256`.
- **LMS** (RFC 8554, NIST SP 800-208): built on SHA-256 alone, so a quantum computer
  doesn't break it. Signatures are bigger (1 to 3 KB) and keys are stateful: every
  signature uses up one of the key's one-time keys, so `mbtool` saves the key file before
  it hands out a signature. An LMS-only bootloader is about 4 KB smaller than an Ed25519
  one.

Signed images are checked against the public keys compiled into the bootloader. By
default unsigned images still boot. Turn on **Refuse unsigned images** and only images
signed by one of your keys will start. That setting requires the hash on every boot,
since a valid signature on the header says nothing about a payload nobody hashed.

A signing key that leaks can be **revoked** in the field with a statement signed by a
separate revocation key, without rebuilding the bootloader. Details for all of this are
on the [Security](security.html) page.

**Anti-rollback**: every image carries a security version. Once an image with version N
is confirmed, A/B images below N are refused.

Keys are compiled in. Nothing in flash, the config file or the console can add a key or
switch the check off.

## Encrypted images and measured boot

**Encrypted images** travel encrypted and are decrypted into the slot as they're
written, using MCUboot's scheme (AES-CTR with an X25519 key envelope), so images made by
imgtool work too. The image in flash is plaintext; encryption protects it on the way.

With **Boot record**, an image that asks for it gets a record of this boot: which checks
ran, which key signed it, the floors in force, and a measurement (a hash over the
bootloader and the image header) that a server can recompute. It's a report, not
attestation: nothing in it is signed by the device.

## Locks

**Lock the bootloader's flash** write-protects the bootloader (and on nRF the golden
slot) right before the jump, until the next reset, so a compromised app can't rewrite
it. **Keep the debug port closed** does the same for SWD on newer nRF52 silicon.
**A locked console** answers questions but refuses anything that changes the board until
someone answers a challenge with an unlock key. Leave the locks off while you're trying a
new board; see [Before you flash](before-you-flash.html).

## Updating the bootloader itself

With **Two-stage**, MimiBoot is split in two: a small stage 0 that never changes, and two
copies of stage 1, the bootloader proper. A new stage 1 is uploaded like any image and
stage 0 starts the best valid copy, so a power cut during a bootloader update can't brick
the board.

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

The console can also speak **mcumgr (SMP)**, the protocol Zephyr's update tools use, so they
can list, upload and confirm images over the serial port. (It has been tested with the
`smp` Python library, not yet with the mcumgr command-line tool.)
On boards whose only connection is USB (SAM D), the console can run over **native USB**
as a serial port.

Details: [Console and boot config](console.html).

## Boot config file

A small text file in flash, with GRUB-style entries: a name, a title, which slot,
arguments for the image. It's kept in two flash sectors and written atomically, so a
power cut while saving leaves the old one intact. **Signed boot config** makes the
bootloader ignore a config that isn't signed with your keys. Without it the bootloader uses
built-in entries (A/B, golden, one per extra slot).

## Talking to the image

The **handoff** is a small block the bootloader writes into the started image's RAM:
which slot and entry it was started from, the argument string, why the chip reset, and
the bootloader's version. It's checksummed and tied to the image, so stale data from an
earlier boot is never mistaken for a fresh one.

The **OS control service** works the other way: the running image can leave a request
for the next boot, like "open the menu", "boot entry X once with these arguments", "boot
the other slot once" or "go to flash mode for an upload".

With **OS hooks** on, this becomes a two-way conversation with the running OS:

- The handoff also says how the previous run ended (a reboot and why, an update, a
  power-off, a panic with its message and address, a watchdog reset), what the
  bootloader was built with, and a few random bytes.
- Requests go through a small mailbox in RAM, so asking for the menu or another entry
  writes no flash.
- The OS can reboot, power off (nRF), roll back to a confirmed image, change the default
  entry, or report a panic, which counts as a crash: after several in a row the
  previous good image boots.
- `mbctl` gives your shell all of it, the way `bootctl` and `grub-reboot` do on Linux.

Both are used through the app library: [App library and handoff](client.html) and
[OS integration](os-integration.html).

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
