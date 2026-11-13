---
title: AVR (Lite core)
---

# AVR (Lite core)

On AVR, MimiBoot-Mu uses a separate small core. It speaks STK500v1, the protocol of
Arduino's own bootloader, so the Arduino IDE and `avrdude -c arduino` upload to it
unchanged. The STK500 code was written from Atmel's AVR061 application note.

Supported: ATmega328P (Uno, Nano, Pro Mini), ATmega168, ATmega644P, ATmega1284P and
ATmega2560. The 328P build has run in the simavr emulator with real avrdude uploads;
the others have only been compiled.

## Size

The default build is about 1.6 KB and fits the 328P's 2 KB boot section. Switching off
the app check and the extras gets it under 1 KB. With the flash-write service and staged
updates it's about 2 KB and needs the 4 KB section. The build picks the smallest boot
section that fits and tells you which one.

## Options

| Option | What it does |
|---|---|
| Serial upload (STK500) | Arduino IDE / avrdude uploads |
| EEPROM read/write | `avrdude -U eeprom:...` through the bootloader |
| When to listen for an upload | after an external reset only (the IDE pulses DTR), or after every reset |
| Upload wait | 250 ms to 8 s |
| Check the app | CRC-32 of the app, recorded at upload and checked every boot |
| No app: stay in the bootloader | wait for an upload instead of jumping into nothing |
| Apps can reboot into the bootloader | `mu_enter_bootloader()` |
| Boot handoff | reset cause and MimiBoot flags passed in registers |
| Trial boots | a new upload must call `mu_confirm()`, or it stops being started |
| Flash-write service | lets apps write their own flash (`mu_spm()`) |
| Staged updates | the app stores a new image, the bootloader installs it |
| LED | blink while listening |

The [option reference](options.html#avr-bootloader-lite-core) has defaults and costs.

## Fuses

The bootloader needs its boot section size (BOOTSZ) and BOOTRST set in the fuses. That's
a one-time write with an ISP programmer (a USBasp, another Arduino as ISP, an AVRISP).
The build prints the value, for example:

```
boot section 2048 bytes: BOOTSZ=0b01, BOOTRST programmed -> hfuse 0xDA
```

and writes a `boards.txt` entry with the matching `upload.maximum_size`, so the Arduino
IDE knows how much room your sketch has.

A stock Uno only reserves 512 bytes, so the default build won't fit without changing
the fuses.

## Using it from a sketch

Ordinary sketches need nothing. For the extras, include `mu_avr.h` (from `client/`,
with `lite/` on the include path too) and set `MU_BOOT_SIZE` to the boot section size:

```c
#define MU_BOOT_SIZE 2048UL
#include "mu_avr.h"

void setup() {
    mu_handoff_t h = mu_handoff();      /* read this first thing */
    if (h.by_mimiboot && (h.flags & MU_HF_TRIAL))
        mu_confirm();                   /* this build works: no more trial boots */
}

/* later: */
mu_enter_bootloader();                  /* reboot and wait for an upload */
mu_reboot();                            /* plain reboot */
```

`mu_handoff()` returns whether MimiBoot started the sketch, the reset cause bits
(`mcusr`) and MimiBoot's flags.

## Writing flash from the app

AVR only allows flash writes from the boot section, so with the flash-write service on,
the bootloader exposes a small jump table at the end of flash:

```c
if (mu_api_present())
    mu_spm(MU_SPM_WRITE, addr, page_buffer);   /* one SPM_PAGESIZE page of the app area */
```

Not available on parts with more than 128 KB of flash (the ATmega2560) yet.

## Staged updates

With staged updates on, a running sketch can update itself without a second chip:

1. Write the new image from `MU_STAGE_BASE` onward (the top half of the app area), page by
   page with `mu_spm()`.
2. Call `mu_stage_commit(length, mu_crc32_flash(MU_STAGE_BASE, length))`.

The sketch reboots. The bootloader checks the CRC, copies the new image over the old one
and records it. If the power goes out halfway, it picks up where it stopped.

## Things to know

- If you flash a sketch by ISP without erasing the EEPROM, clear MimiBoot's app record
  (the top 16 bytes of EEPROM), or the CRC check will refuse the new sketch. An upload
  through the bootloader rewrites the record.
- The Mega 2560 build speaks STK500v1 (`avrdude -c arduino`), not the stk500v2 protocol of
  the Mega's stock bootloader.
- The default "reset pin only" entry has not run in the emulator; the emulator tests use
  "every reset".
