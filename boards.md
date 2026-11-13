---
title: Boards
---

# Boards

Every board MimiBoot-Mu knows about, grouped the way the board picker groups them. "Tested" says how far each one has actually been run; see [Test status](status.html) for the details.

To use a board that isn't listed, see [Adding boards and chips](porting.html).

## STMicroelectronics

STM32 Cortex-M MCUs: F0/F1/F3 (Cortex-M0/M3/M4), F2/F4/F7 (M3/M4/M7), L4, G0, G4. Bare-metal full core, UART console, the chip's ROM loader as the fallback.

### Nucleo boards

| Board | Chip | Console | Tested |
|---|---|---|---|
| `nucleo-f103rb`<br>Nucleo-F103RB (ST-LINK VCP = USART2) | stm32f103rb | USART2, TX A2 RX A3 | built |
| `nucleo-f030r8`<br>Nucleo-F030R8 (Cortex-M0, no VTOR: apps keep RAM[0:0xC0] free) | stm32f030r8 | USART2, TX A2 RX A3 | built |
| `nucleo-f072rb`<br>Nucleo-F072RB (Cortex-M0, no VTOR: apps keep RAM[0:0xC0] free) | stm32f072rb | USART2, TX A2 RX A3 | built |
| `nucleo-f303re`<br>Nucleo-F303RE | stm32f303re | USART2, TX A2 RX A3 | built |
| `nucleo-f401re`<br>Nucleo-F401RE | stm32f401re | USART2, TX A2 RX A3 | built |
| `nucleo-f411re`<br>Nucleo-F411RE | stm32f411re | USART2, TX A2 RX A3 | built |
| `nucleo-f446re`<br>Nucleo-F446RE | stm32f446re | USART2, TX A2 RX A3 | built |
| `nucleo-f746zg`<br>Nucleo-F746ZG (ST-LINK VCP = USART3 PD8/PD9) | stm32f746zg | USART3, TX D8 RX D9 | built |
| `nucleo-l432kc`<br>Nucleo-L432KC (VCP = USART2 PA2/PA15) | stm32l432kc | USART2, TX A2 RX A15 | built |
| `nucleo-l476rg`<br>Nucleo-L476RG | stm32l476rg | USART2, TX A2 RX A3 | built |
| `nucleo-g031k8`<br>Nucleo-G031K8 | stm32g031k8 | USART2, TX A2 RX A3 | built |
| `nucleo-g071rb`<br>Nucleo-G071RB | stm32g071rb | USART2, TX A2 RX A3 | built |
| `nucleo-g0b1re`<br>Nucleo-G0B1RE | stm32g0b1re | USART2, TX A2 RX A3 | built |
| `nucleo-g431rb`<br>Nucleo-G431RB (VCP = LPUART1 PA2/PA3) | stm32g431rb | LPUART1, TX A2 RX A3 | built |
| `nucleo-g474re`<br>Nucleo-G474RE (VCP = LPUART1 PA2/PA3) | stm32g474re | LPUART1, TX A2 RX A3 | built |

### Blue Pill / Black Pill

| Board | Chip | Console | Tested |
|---|---|---|---|
| `bluepill`<br>Blue Pill (STM32F103C8, USART1 on PA9/PA10) | stm32f103c8 | USART1, TX A9 RX A10 | built |
| `bluepill-128k`<br>Blue Pill with 128 KB (STM32F103CB) | stm32f103cb | USART1, TX A9 RX A10 | built |
| `blackpill-f401cc`<br>WeAct Black Pill F401CC (USART1 on PA9/PA10) | stm32f401cc | USART1, TX A9 RX A10 | built |
| `blackpill-f411ce`<br>WeAct Black Pill F411CE (USART1 on PA9/PA10) | stm32f411ce | USART1, TX A9 RX A10 | built |

### Discovery kits

| Board | Chip | Console | Tested |
|---|---|---|---|
| `stm32vldiscovery`<br>STM32VLDISCOVERY / QEMU stm32vldiscovery (STM32F100RB) | stm32f100rb | USART1, TX A9 RX A10 | emulator |
| `stm32f4-discovery`<br>STM32F4DISCOVERY (F407VG; console USART2 PA2/PA3 - wire a USB-UART) | stm32f407vg | USART2, TX A2 RX A3 | built |

### Custom boards (bare chip)

| Board | Chip | Console | Tested |
|---|---|---|---|
| `stm32f103rc-generic`<br>STM32F103RC board (256 KB; console USART1 PA9/PA10) | stm32f103rc | USART1, TX A9 RX A10 | built |
| `stm32f103re-generic`<br>STM32F103RE board (512 KB; console USART1 PA9/PA10) | stm32f103re | USART1, TX A9 RX A10 | built |

### QEMU test boards

| Board | Chip | Console | Tested |
|---|---|---|---|
| `netduinoplus2`<br>QEMU netduinoplus2 (STM32F405RG) - emulator test board | stm32f405rg | USART1, TX A9 RX A10 | emulator |
| `netduino2`<br>QEMU netduino2 (STM32F205RG) | stm32f205rg | USART1, TX A9 RX A10 | built |

## Nordic Semiconductor

nRF52 (Cortex-M4F, Bluetooth LE) and nRF51 (Cortex-M0). Bare-metal full core, UART console. Replaces Nordic's MBR at 0x0: Zephyr / nrfx / FreeRTOS firmware, no SoftDevice.

### Nordic development kits

| Board | Chip | Console | Tested |
|---|---|---|---|
| `nrf52840dk`<br>Nordic nRF52840 DK (console = J-Link VCOM, P0.06/P0.08) | nrf52840 | UART, TX P0.06 RX P0.08 | built |
| `nrf52dk`<br>Nordic nRF52 DK (nRF52832; console = J-Link VCOM, P0.06/P0.08) | nrf52832 | UART, TX P0.06 RX P0.08 | built |
| `nrf52833dk`<br>Nordic nRF52833 DK (console = J-Link VCOM, P0.06/P0.08) | nrf52833 | UART, TX P0.06 RX P0.08 | built |
| `nrf52dk-nrf52810`<br>nRF52 DK built for nRF52810 (Zephyr's nrf52dk/nrf52810; UARTE console P0.06/P0.08) | nrf52810 | UART, TX P0.06 RX P0.08 | built |
| `nrf52840dk-nrf52811`<br>nRF52840 DK built for nRF52811 (Zephyr's nrf52840dk/nrf52811; UARTE console P0.06/P0.08) | nrf52811 | UART, TX P0.06 RX P0.08 | built |
| `nrf52833dk-nrf52820`<br>nRF52833 DK built for nRF52820 (Zephyr's nrf52833dk/nrf52820; UARTE console P0.06/P0.08) | nrf52820 | UART, TX P0.06 RX P0.08 | built |

### Maker boards (Adafruit, Seeed, Arduino)

| Board | Chip | Console | Tested |
|---|---|---|---|
| `feather-nrf52840`<br>Adafruit Feather nRF52840 (native USB only: USB-UART on TX P0.25 / RX P0.24) | nrf52840 | UART, TX P0.25 RX P0.24 | built |
| `xiao-nrf52840`<br>Seeed XIAO nRF52840 (native USB only: USB-UART on TX P1.11 / RX P1.12) | nrf52840 | UART, TX P1.11 RX P1.12 | built |
| `nano33ble`<br>Arduino Nano 33 BLE (native USB only: USB-UART on TX P1.03 / RX P1.10) | nrf52840 | UART, TX P1.03 RX P1.10 | built |

### BBC micro:bit

| Board | Chip | Console | Tested |
|---|---|---|---|
| `microbit-v2`<br>BBC micro:bit v2 (nRF52833; console = interface chip VCOM, P0.06/P1.08; button A) | nrf52833 | UART, TX P0.06 RX P1.08 | built |
| `microbit`<br>BBC micro:bit v1 / QEMU microbit (nRF51822; no VTOR: vector relay, apps keep RAM[0:8] free) | nrf51822 | UART, TX P0.24 RX P0.25 | emulator |

## Microchip / Atmel

Classic AVR (Arduino Uno, Nano, Mega...) with the small Lite core, and SAM D21 / D51 (Arduino Zero, Adafruit M0/M4, Seeed XIAO) with the full core.

### AVR: Arduino and classic ATmega

| Board | Chip | Console | Tested |
|---|---|---|---|
| `uno`<br>Arduino Uno / Nano / Pro Mini 5V (ATmega328P, 16 MHz) | atmega328p | UART0 at 115200 baud, 16 MHz | emulator |
| `pro-mini-3v3`<br>Arduino Pro Mini 3.3V (ATmega328P, 8 MHz) | atmega328p | UART0 at 57600 baud, 8 MHz | built |
| `diecimila`<br>Arduino Diecimila / Nano (ATmega168, 16 MHz) | atmega168 | UART0 at 115200 baud, 16 MHz | built |
| `sanguino-644p`<br>Sanguino / ATmega644P (16 MHz) | atmega644p | UART0 at 115200 baud, 16 MHz | built |
| `mighty-1284p`<br>MightyCore ATmega1284P (16 MHz) | atmega1284p | UART0 at 115200 baud, 16 MHz | built |
| `mega2560`<br>Arduino Mega 2560 (16 MHz; avrdude -c arduino, not the stk500v2 wiring bootloader) | atmega2560 | UART0 at 115200 baud, 16 MHz | built |

### SAM D21 (Cortex-M0+)

| Board | Chip | Console | Tested |
|---|---|---|---|
| `arduino-zero`<br>Arduino Zero (console = EDBG virtual COM, SERCOM5 PB22/PB23) | samd21g18a | SERCOM5 UART, TX PB22 RX PB23 | built |
| `mkrzero`<br>Arduino MKR Zero / MKR boards (Serial1 = SERCOM5 PB22/PB23: wire a USB-UART) | samd21g18a | SERCOM5 UART, TX PB22 RX PB23 | built |
| `feather-m0`<br>Adafruit Feather M0 (Serial1 = SERCOM0 PA10/PA11: wire a USB-UART) | samd21g18a | SERCOM0 UART, TX PA10 RX PA11 | built |
| `trinket-m0`<br>Adafruit Trinket M0 (SAMD21E18; UART = SERCOM0 PA06/PA07: wire a USB-UART) | samd21e18a | SERCOM0 UART, TX PA06 RX PA07 | built |
| `xiao-samd21`<br>Seeed XIAO SAMD21 (UART = SERCOM4 PB08/PB09: wire a USB-UART) | samd21g18a | SERCOM4 UART, TX PB08 RX PB09 | built |

### SAM D51 (Cortex-M4F)

| Board | Chip | Console | Tested |
|---|---|---|---|
| `feather-m4`<br>Adafruit Feather M4 Express (SAMD51J19; Serial1 = SERCOM5 PB16/PB17: wire a USB-UART) | samd51j19a | SERCOM5 UART, TX PB16 RX PB17 | built |
| `samd51j20a`<br>SAM D51J20A board with the Feather M4 console pins (SERCOM5 PB16/PB17) | samd51j20a | SERCOM5 UART, TX PB16 RX PB17 | built |
| `wio-terminal`<br>Seeed Wio Terminal (SAMD51P19; Pi-header UART = SERCOM2 PB26/PB27) | samd51p19a | SERCOM2 UART, TX PB26 RX PB27 | built |

## Raspberry Pi

RP2040 and RP2350 (Arm and RISC-V) with the original RP ports on the Pico SDK: USB console, UF2, fixed flash presets.

### Pico boards

| Board | Chip | Console | Tested |
|---|---|---|---|
| `pico`<br>Raspberry Pi Pico (RP2040; RAM-loaded images only) | rp2040 | USB (CDC serial) | hardware |
| `pico2`<br>Raspberry Pi Pico 2 (RP2350, Arm) | rp2350-arm | USB (CDC serial) | built |
| `pico2-riscv`<br>Raspberry Pi Pico 2 (RP2350, Hazard3 RISC-V) | rp2350-riscv | USB (CDC serial) | built |

### Other RP2350 boards

| Board | Chip | Console | Tested |
|---|---|---|---|
| `rp2350-lcd`<br>Waveshare RP2350-LCD-1.28 (16 MB, Arm) | rp2350-arm | USB (CDC serial) | hardware |

Status values: `hardware` ran on a real board; `emulator` booted in an emulator (QEMU / simavr); `built` compiles and fits; never run; `host` logic host-tested only.
