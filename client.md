---
title: App library and handoff
---

# App library and handoff

Your firmware doesn't need anything from MimiBoot-Mu to be booted. The app library is
for when it wants to know how it was started, confirm an update, ask for something on
the next boot, or install an update itself.

This page covers the full core (RP, STM32, nRF, SAM D). AVR apps use a small header
instead; see [AVR](avr.html).

## Adding it to your build

STM32, nRF and SAM D: every build writes `client.txt` listing the source files and the
compiler flags. In short, compile these with your app:

```
core/mb_hash.c core/mb_ed25519.c core/mb_image.c core/mb_bcl.c core/mb_config.c
core/mb_control.c core/mb_grub.c core/mb_recv.c core/mb_formats.c client/mb_client.c
port/cortexm/cm_port.c port/<stm32|nrf|samd>/<vendor>_flash.c
```

with `-DMB_CM_CLIENT -DMB_CONFIG_HEADER="path/to/mimiboot_config.h"` and the repository
root and `include/` on the include path. Use the `mimiboot_config.h` from the same build
as the bootloader on the board: it carries the flash layout. On nRF and SAM D also add
`-fno-delete-null-pointer-checks` (flash starts at address 0).

RP2040/RP2350 with the Pico SDK:

```cmake
include(path/to/MimiBoot-Mu/mimiboot_client.cmake)
target_link_libraries(my_app mimiboot_client)
```

## A minimal app

```c
#include "client/mb_client.h"

extern const mb_port_t mb_mcu_port;      /* RP: mb_rp2040_port / mb_rp2350_port */
MB_HANDOFF_BUFFER;                       /* optional: lets the bootloader pass the handoff */

int main(void) {
    mb_client_init(&mb_mcu_port);
    mb_client_set_handoff(&mb_handoff);  /* call this before anything clears RAM it uses */

    mb_client_status_t st;
    mb_client_status(&st);
    /* ... start up, check that things work ... */
    if (st.state == MB_ST_TRIAL)
        mb_client_confirm();             /* "this build is good": no rollback */
    ...
}
```

`MB_HANDOFF_BUFFER` puts a buffer in the section `.uninitialized_data.mb_handoff`. Make
sure your linker script places `.uninitialized_data*` in a `NOLOAD` section that the
startup code doesn't zero. `mbtool pack` finds the buffer in the ELF and records its
address in the image header, and the bootloader fills it in right before starting the
image.

## What the handoff contains

| Field | Meaning |
|---|---|
| `slot`, `slot_name` | the slot this image was started from |
| `entry`, `entry_name` | the boot config entry, if it was started through one |
| `kind` | how it was chosen: normal, trial, once, fallback, golden, from the menu, a non-default entry |
| `args` | the argument string (up to 95 characters) |
| `reset_cause` | power-on/pin, watchdog, or a requested reboot |
| `boot_count` | the boot log's counter |
| `loader_version` | the bootloader's version |

It is checksummed and carries the ID of the image it was written for. The library clears
it once read, so data left in RAM from an earlier boot is never taken for a new one.
`mb_client_args()` always returns a string: from the handoff if there is one, otherwise
from the boot config in flash.

## Status and confirming

`mb_client_status(&st)` fills in:

| Field | Meaning |
|---|---|
| `slot` | the slot we're running from (`MB_SLOT_NONE` if MimiBoot didn't start us) |
| `state` | `MB_ST_TRIAL` means the bootloader is waiting for a confirm |
| `tries_left` | trial boots left before rollback |
| `active` | the preferred A/B slot |
| `version`, `security_version`, `min_security` | versions and the anti-rollback floor |
| `name`, `entry` | the image's name and the entry we were booted as |

`mb_client_confirm()` marks the running image good, makes it the preferred A/B slot and
raises the anti-rollback floor to its security version. Call it once your app has
decided that it works. `mb_client_reject(slot)` marks another slot bad.

## Requests for the next boot

```c
mb_client_request(MB_REQ_MENU, 0);              /* stop in the boot menu */
mb_client_request(MB_REQ_BOOT_ONCE, slot);      /* boot this slot once */
mb_client_request(MB_REQ_FLASH, 0);             /* console up, waiting for mbtool */
mb_client_request(MB_REQ_BOOTSEL, 0);           /* the chip's ROM loader (RP, STM32) */
mb_client_request_entry("debug", "loglevel=7"); /* boot a config entry once, with args */
```

The request is written to the boot log; reset the chip afterwards and the bootloader acts
on it once. `mb_client_reboot_menu()` and `mb_client_reboot_config()` queue a request
and reboot in one call. `mb_client_capabilities()` says which requests this bootloader
build understands.

## Installing an update from the app

```c
int other = mb_client_other_slot();
mb_client_install_begin(other, &hdr);        /* the 256-byte image header */
mb_client_install_write(offset, data, len);  /* as many times as needed */
mb_client_install_end();                     /* hash check, then the header is written */
```

The header goes in last, after the payload has hashed correctly, so a half-written image
is never seen as one. On the next reset the bootloader boots it, as a trial if trials
are on. Writing your own slot or the golden slot is refused.

Flash on bare-metal chips is written in small units that can only be written once per
erase (2 to 64 bytes depending on the chip). Pass chunks that start on a 256-byte
boundary and you're fine everywhere.

`mb_client_recv(slot, io, getc, puts, max_frame)` runs the same upload protocol as the
bootloader's console over any byte stream you give it (a UART, a socket, a USB endpoint),
so `mbtool ota` can push an update to a running app.

## Boot config from the app

`mb_client_get_config()`, `mb_client_set_config()` and `mb_client_set_default()` read
and replace the boot config text, or just change the default entry.
