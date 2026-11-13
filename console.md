---
title: Console and boot config
---

# Console and boot config

## Getting to the console

The console is USB serial on RP boards and a UART elsewhere (115200 8N1 by default; the
pins are in the board's description and in [Boards](boards.html)). Normal boots are
silent. The console comes up when:

- the board's button is held at reset,
- any key arrives on the UART within the break-in window right after reset (300 ms by
  default, `BREAK_IN_MS`; 0 turns it off and boots don't wait at all),
- the running app asked for the menu,
- or there is nothing bootable.

On boards with only native USB (Feathers, XIAOs, the Nano 33 BLE, MKR boards) the UART
console needs a USB-to-serial adapter on the console pins. Nordic's DKs, the micro:bit and
the Arduino Zero have a debugger with a virtual COM port already wired to them.

## The boot menu

```
  MimiBoot  ...   nucleo-f411re, power-on/pin

> 1  A/B (newest good image)      ab  myapp   1.2.0   GOOD
  2  golden recovery image        g   (empty)

   e  edit args    c  command line    r  reboot

   booting os in 10 s, any key stops the countdown
```

| Key | What it does |
|---|---|
| Up, Down, Enter, or 1 to 8 | boot an entry |
| `e` | edit the selected entry's arguments, then boot it (this boot only) |
| `c` | the command line (if built in) |
| `r` | reboot |
| `u` | the chip's ROM loader (RP and STM32 only) |

The countdown is `MENU_SECS` (10 s by default) unless the boot config sets a timeout.

## The command line

| Command | What it does |
|---|---|
| `ls` | slots and their state |
| `entries` | the boot config's entries (`*` marks the default) |
| `info <slot>` | image header and log entry |
| `verify [slot]` | SHA-256 and signature check, all slots if none given |
| `boot <entry or slot> [args...]` | boot it now (a new A/B image boots as a trial) |
| `boot <slot> force` | boot an image with a good hash and signature, whatever its state |
| `once <entry> [args...]` | boot it on the next reset only, then reboot |
| `confirm <slot>`, `reject <slot>` | mark it good (and preferred) or bad |
| `retry <slot> [n]` | give an image n more tries |
| `recv <slot>` | receive an image from `mbtool install` |
| `ramboot [args...]` | receive a RAM image and start it; flash untouched |
| `erase <slot>` | wipe a slot (asks first) |
| `cfg show`, `cfg recv`, `cfg reset` | the boot config |
| `default <entry>` | save the default entry |
| `request <mode>` | queue restart / menu / config / flash / bootsel and reboot |
| `capabilities` | what this build supports |
| `menu`, `reboot`, `bootsel`, `exit` | back to the menu, reboot, ROM loader, boot normally |

With the diagnostic commands built in there are also `keys` (signature policy and the
trusted keys), `log` (the raw boot log state), `layout` (the flash map), `md <off> [len]`
(dump memory or flash) and `sha <off> <len>` (hash a flash range). Turn these off for
products: `md` reads anything the bootloader can see.

The command line can be left out of the build entirely (`SHELL`). The menu then has no
`c` key, and uploads aren't possible.

## Flash mode

A running app can ask for flash mode (`MB_REQ_FLASH`). The bootloader then skips the
countdown, opens the command line, prints `MB FLASH READY` and waits for `mbtool`.
After 5 minutes without input it boots normally.

## The boot config file

The config is a small text file stored in two flash sectors. Saving writes the other
copy and then marks it current, so a power cut never leaves you without one. Push one
with `mbtool cfg push boot.cfg`, look at it with `cfg show`. It needs `CFG_FILE` in the
build (off by default on bare-metal chips, where it costs two erase sectors).

Two syntaxes are accepted. The plain one:

```
timeout 5                    # menu countdown in seconds (0: no menu unless asked)
default os                   # entry name or number
entry os
    title My OS
    slot ab                  # the A/B pair, with the update and rollback policy
    args console=ttyS0 quiet
entry recovery
    slot g
```

And a GRUB-style one (`GRUB` option):

```
set default=os
set timeout=5
menuentry os "My OS" {
    slot ab
    args console=ttyS0 quiet
}
menuentry recovery "Golden recovery" {
    slot g
}
```

Up to 8 entries and 3840 bytes of text. A line that doesn't parse is skipped and
reported. Security settings can't be changed from the config: keys and "refuse unsigned
images" are compiled in.

Without a config file (or with nothing usable in it) the built-in entries apply: the A/B
pair, the golden image, and one entry per extra slot.
