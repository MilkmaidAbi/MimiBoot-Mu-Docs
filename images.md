---
title: Images and flashing
---

# Images and flashing

## Where to link your app

On STM32, nRF and SAM D, each build writes `layout.txt`. It lists every slot and the
address to link your firmware at, which depends on the image format:

| Format | Link the firmware at |
|---|---|
| MimiBoot image (`mbtool pack`) | slot address + 0x1000 |
| Zephyr / MCUboot image | slot address + 0x200 (SAM D51: + 0x400) |
| raw `.bin` | the slot address itself |

The build also writes `app-slot-a.ld`, `app-slot-b.ld` and so on: a linker `MEMORY`
block for a MimiBoot image in that slot. Use it in place of the `MEMORY` block in your
own linker script.

On Cortex-M0 chips without VTOR (nRF51, STM32F0) apps have to leave the first few bytes
of RAM alone: 8 bytes on nRF51, 192 on STM32F0. The generated `app-slot-*.ld` already
starts RAM after them.

On RP boards the slots are at fixed addresses; `python tools/mbtool.py layout` prints them.

## Making an image

```
python tools/mbtool.py pack app.elf --target stm32f411re --name myapp --version 1.2.0 -o app.mimg
```

`pack` reads the ELF, works out whether it runs from flash or RAM, finds the handoff
buffer if the app declares one, and writes a 4 KB header in front of the firmware.
`--target` is the chip name from `layout.txt` (or `rp2040` / `rp2350`); the bootloader
refuses images built for another chip.

Useful options:

| Option | What it does |
|---|---|
| `--key keys/mimiboot.key` | sign it |
| `--secver N` | security version, for anti-rollback |
| `--golden` | mark it as a golden (recovery) image |
| `--trial-ms N` | watchdog timeout for its trial boots |

Other commands:

```
python tools/mbtool.py info app.mimg --pub keys/mimiboot.pub    # show and check an image
python tools/mbtool.py sign app.mimg --key keys/mimiboot.key    # sign an existing image
python tools/mbtool.py keygen --out keys/mimiboot               # new Ed25519 key pair
```

Zephyr and FreeRTOS firmware can also be booted without `mbtool`: see
[Zephyr and FreeRTOS](zephyr-freertos.html).

## Installing with a debugger

Write the image (or the raw `.bin`) to the slot address from `layout.txt`. On the next
reset the bootloader finds it, checks it and boots it.

## Installing over the console

The bootloader can receive an image over its console and write it to a slot itself.
This needs the console, the command line and **Image upload** switched on.

1. Open the bootloader's menu: hold the board's button at reset, or reset the board
   while pressing a key in a serial terminal. On RP boards, use the BOOT button or have
   the running app ask for the menu.
2. Close the terminal so the port is free.
3. Run:

```
python tools/mbtool.py install app.mimg --slot a --port COM5
```

The upload goes in CRC-checked frames. The image is hashed (and its signature checked)
before its header is written, so a broken upload never looks like an image. A new A/B
image boots as a trial on the next reset if trials are on.

`mbtool` talks at 115200 baud. If you changed the bootloader's baud rate, uploads won't
work with it yet. On RP boards `--port` can be left out; elsewhere it's needed.

Other console commands from the host:

```
python tools/mbtool.py ramboot app_ram.mimg -- arg1 arg2   # send a RAM image and start it
python tools/mbtool.py cfg push boot.cfg                   # replace the boot config
python tools/mbtool.py cmd "ls" "verify"                   # run console commands
```

## Installing from the running app

An app linked with the app library can write an update into the other slot itself
(`mb_client_install_*`) or receive one over any byte stream (`mb_client_recv`). See
[App library and handoff](client.html). `mbtool ota` and `mbtool deploy` drive that
from the host, for an OS that offers a `boot recv` / `boot flash` command.

## RP2040 and RP2350 extras

```
python tools/mbtool.py uf2 app.mimg --slot a -o app_a.uf2   # a UF2 that lands in slot a
python tools/mbtool.py flash app.mimg --slot a               # picotool load at the slot address
```

The RP2040 port can only start RAM images, so link RP2040 apps for SRAM
(`pico_set_binary_type(... no_flash)`).
