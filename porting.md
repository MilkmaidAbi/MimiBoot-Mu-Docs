---
title: Adding boards and chips
---

# Adding boards and chips

Boards, chips and chip families are data in `config/targets.toml`. For a chip that's
already supported, a new board is a few lines and no C.

## A new board

Copy a board with the same chip and change the pins. The fields depend on the vendor.

STM32:

```toml
[board.my-f411-board]
title = "My F411 board (console USART1 on PA9/PA10)"
category = "Custom boards (bare chip)"
chip = "stm32f411ce"
uart = "usart1"          # one of the family's uarts
tx = "A9"
rx = "A10"
af = 7                   # alternate function number (not on F1)
led = "C13"              # optional
button = "A0"            # optional; add button_active_high = true if pressed reads 1
status = "built"
```

nRF:

```toml
[board.my-nrf-board]
title = "My nRF52840 board"
category = "Maker boards (Adafruit, Seeed, Arduino)"
chip = "nrf52840"
tx = "P0.06"
rx = "P0.08"
led = "P0.13"            # LEDs and buttons are active low unless *_active_high = true
status = "built"
```

SAM D:

```toml
[board.my-samd-board]
title = "My SAMD21 board"
category = "SAM D21 (Cortex-M0+)"
chip = "samd21g18a"
sercom = 0
tx = "PA10"
rx = "PA11"
mux = "C"                # the PORT function letter for that SERCOM on those pins
rxpo = 3                 # the SERCOM's RXPO / TXPO values (D21 and D51 encode TXPO differently)
txpo = 1
status = "built"
```

Zephyr's board files (`boards/<vendor>/<board>/*.dts` and `*-pinctrl.dtsi`) are a good
source for the console pins, the SERCOM pads and the `rxpo`/`txpo` values.

AVR:

```toml
[board.my-avr-board]
title = "My 8 MHz ATmega328P board"
category = "AVR: Arduino and classic ATmega"
chip = "atmega328p"
f_cpu = 8000000
baud = 57600
led = "B5"
status = "built"
```

`category` decides where the board shows up in the picker. Run
`python tests/test_muconfigurator.py` afterwards: it checks that every board resolves, and with
`--build-all` that it builds.

## A new chip in a known family

Add a `[chip]` entry with its memory:

```toml
[chip.stm32f412rg]
family = "stm32f4"
flash = "1M"
ram = "256K"
sectors = "4x16K,1x64K,7x128K"   # the erase sectors, in order
boot = "48K"                     # default size of the bootloader region
```

`sectors` has to match the reference manual exactly: the flash layout is planned on
these boundaries so that erasing one region can't touch another. On nRF and SAM D21 the
layout works in 4 KB sectors and the driver erases the smaller physical pages inside
them; SAM D51 uses its 8 KB blocks.

Small chips can switch features off by default so the build fits:

```toml
defaults = { SIGNATURES = false, CONSOLE = false }
```

Every chip needs at least one board, or the build tests won't compile it.

## A new family or vendor

A `[family]` entry holds the register addresses and constants the vendor code reads:
the flash controller, the reset-cause register, the clock, the GPIO block, the UARTs.
The existing STM32 families show what's needed.

New vendor code goes in `port/<vendor>/` and implements the functions in
`port/cortexm/mcu.h`:

| Function | What it does |
|---|---|
| `mcu_flash_erase`, `mcu_flash_program` | erase and write flash; read back and refuse to write a unit twice |
| `mcu_reset_flags_take` | why the chip reset |
| `mcu_init`, `mcu_restore` | set up clocks (and a timer) and put them back before an image starts |
| `mcu_time_ms` | milliseconds, if the chip doesn't use SysTick |
| `mcu_uart_init`, `_deinit`, `_getc`, `_putc` | the console |
| `mcu_wdt_start`, `_feed`, `_disarm` | the watchdog |
| `mcu_led`, `mcu_button`, `mcu_pins_release` | LED, button, and their pins back to reset state |

Everything else (startup code, the vector table, starting images, the console, the app
library) is shared in `port/cortexm/cm_port.c`. The generator side is in
`tools/mu/gen.py` (`VENDOR_MACROS`) and `tools/mu/build.py` (`VENDOR_SOURCES`).
