---
title: Test status
---

# Test status

Read this before flashing a board you can't easily recover. It says what has actually
run, and where. The same information, with more detail, is in
[docs/HARDWARE-STATUS.md](https://github.com/MilkmanAbi/MimiBoot-Mu/blob/main/docs/HARDWARE-STATUS.md)
in the repository.

| Label | Meaning |
|---|---|
| hardware | ran on a real board |
| emulator | the real build ran in QEMU or simavr |
| model | the real driver code ran on a PC against a model of the hardware |
| built | compiles and fits; never ran |

## By family

| Family | Status | What ran |
|---|---|---|
| RP2040, RP2350 | hardware (earlier builds) | Earlier versions of these ports ran on a Pico and a Waveshare RP2350-LCD. Builds made through `mimiconf.py`, with options switched off, have not been flashed yet. |
| STM32F405, STM32F100 | emulator | In QEMU: signed images boot, corrupted and wrongly signed ones are refused, raw and Zephyr images boot, the break-in opens the menu, the app library finds its slot and reads the handoff. |
| other STM32 | built | |
| nRF51 (micro:bit) | emulator | In QEMU: the console, booting with interrupts forwarded to the app (no VTOR on this chip), an upload over the UART written to flash and booted, the RAM wipe, the menu-only console. QEMU emulates this chip's flash controller, so those uploads really wrote flash. |
| nRF52 | built | Shares the flash, UART and timer code with nRF51. The UARTE console used on the nRF52810/811/820, the crystal start, debug unlocking, the watchdog and VTOR handling have not run. |
| SAM D21, D51 | built + model | The flash driver ran against a model of each flash controller. Clocks, the SERCOM UART, pins and the watchdog have not run. |
| ATmega328P | emulator | In simavr with real avrdude: uploads and verify, the CRC check refusing a damaged app, trial boots, the app asking for the bootloader, a staged update. |
| ATmega168, 644P, 1284P, 2560 | built | |

## Known gaps

- No STM32 flash write has ever run: QEMU doesn't emulate the STM32 flash controller.
  Every erase is checked blank and every write is read back, so a driver problem shows
  up as an error rather than as corrupted flash, but test on a board you can recover
  with an ST-LINK first.
- Uploads over a real UART haven't run on any board yet (only in QEMU on the nRF51).
- The SAM D clocks are the internal oscillators, accurate to about 2%. Fine for 115200
  baud on the bench, not checked over temperature.
- MCUboot's swap trailer (Zephyr's `boot_write_img_confirmed()`, mcumgr uploads) isn't
  supported.
- AVR fuse values are computed from the datasheets and haven't been written to a chip.
  The flash-write service isn't available on the ATmega2560. The ATmega32u4 isn't
  supported.
- The menu (TUI) has been tested on Linux only, not with windows-curses.

## Running the tests

```
python tests/run.py                       # core logic on the host (312 checks)
python tests/test_profiles.py             # the core built with features switched off
python tests/test_mimiconf.py --build-all # configurator, and every board compiles
python tests/qemu/test_qemu.py            # STM32 and nRF51 in QEMU
python tests/test_sim_flash.py            # SAM D flash driver against a model
python tests/avr/test_avr.py              # AVR in simavr with avrdude (Linux or WSL)
python3 tests/tui_smoke.py                # the menu in a pseudo-terminal (Linux or WSL)
```

The QEMU tests need `qemu-system-arm` (on Windows they use the one in WSL). The Zephyr
cases need imgtool (`pip install imgtool`). The AVR tests need simavr's library and
avrdude on Linux or WSL. `tests/test_profiles.py` and `tests/test_sim_flash.py` need a
host C compiler.
