---
title: Zephyr and FreeRTOS
---

# Zephyr and FreeRTOS

MimiBoot-Mu can boot what Zephyr and FreeRTOS projects already produce, on STM32, nRF
and SAM D. Both paths have run in QEMU on an STM32F405 build. Neither has run on a real
board yet, and no full Zephyr application has been built against the generated overlay.

## Zephyr

Zephyr can build apps for MCUboot, and MimiBoot-Mu reads the MCUboot image format.

1. Turn on **Zephyr / MCUboot images** (`IMG_MCUBOOT=y`). If your app won't call
   MimiBoot's confirm, also turn off **Updates must confirm** (`AB_TRIAL=n`), otherwise
   a new image is rolled back after a few boots.
2. Build MimiBoot-Mu. The output folder has `zephyr-partitions.overlay`, which puts
   Zephyr's `boot_partition` and `slot0_partition` where the bootloader and slot A
   actually are.
3. Build the app for MCUboot with that overlay:

   ```
   west build -b <board> app -- -DCONFIG_BOOTLOADER_MCUBOOT=y \
       -DDTC_OVERLAY_FILE=path/to/zephyr-partitions.overlay
   ```

   Zephyr then links at slot 0 + 0x200 (`CONFIG_ROM_START_OFFSET`). On SAM D51 use
   `-DCONFIG_ROM_START_OFFSET=0x400`, and `--header-size 0x400` when signing: its vector
   table has to be 1 KB aligned. `layout.txt` gives the right value for your board.

   The overlay replaces the board's whole partition table. If something in the board's
   files points at another partition label (`storage_partition` for settings, say), add
   that partition back outside MimiBoot's boot log and config area. The comment at the
   top of the overlay gives that range.

4. Sign it. For signature checks use an Ed25519 key:

   ```
   imgtool keygen -k key.pem -t ed25519
   imgtool sign --header-size 0x200 --pad-header --align 8 --version 1.0.0 \
       --slot-size <slot size from layout.txt> --key key.pem zephyr.bin app.signed.bin
   ```

   MimiBoot needs the public half as one hex line in the `KEYS` file:

   ```
   python -c "from cryptography.hazmat.primitives import serialization as s;\
   k=s.load_pem_private_key(open('key.pem','rb').read(),None);\
   print(k.public_key().public_bytes(s.Encoding.Raw,s.PublicFormat.Raw).hex())" > zephyr.pub
   ```

5. Flash `app.signed.bin` at slot A's address.

What gets checked: the image header, the SHA-256 TLV over the header, image and
protected TLVs, the Ed25519 TLV (imgtool's default mode, which signs the digest), and the
security counter TLV for anti-rollback. Encrypted images and "pure" Ed25519 signatures
are refused. RSA and ECDSA signatures aren't checked, so such an image counts as unsigned
and only boots while signatures are optional.

Not supported: MCUboot's swap trailer. Zephyr's `boot_write_img_confirmed()` and mcumgr
image uploads talk to that trailer, which MimiBoot-Mu doesn't read. Turn trials off, or
confirm through MimiBoot's app library.

## FreeRTOS, bare metal, Arduino builds

Two ways:

- **Raw firmware** (`IMG_RAW=y`): link the firmware at a slot's address and flash the
  plain `.bin` there. The bootloader checks that the initial stack pointer and reset
  address look sane, and starts it. There's no integrity check.
- **Wrapped**: link at slot + 0x1000 and run
  `python tools/mbtool.py pack fw.elf --target <chip>`. You get the SHA-256 check,
  optional signatures, versions and the handoff.

Either way the firmware starts with:

- VTOR pointing at its vector table and the stack pointer loaded from it,
- interrupts disabled and cleared in the NVIC, SysTick stopped,
- the console UART and the pins the bootloader used back in their reset state,
- the reset clock: HSI/MSI on STM32; on nRF the internal oscillator, with the crystal
  stopped and TIMER0 reset; OSC8M divided by 8 on SAM D21; the 48 MHz DFLL on SAM D51,
- interrupts enabled (PRIMASK clear).

That's what STM32Cube's `HAL_Init()`, the nrfx and ASF `SystemInit()` functions and
FreeRTOS's Cortex-M ports expect. FreeRTOS reads the initial stack pointer through VTOR
when it starts the scheduler, so VTOR has to be right, and it is.

If trial boots use the watchdog (`TRIAL_WATCHDOG=y`), the STM32 and nRF watchdogs keep
running in your firmware and can't be stopped, so it has to refresh them. That option is
off by default on these chips. On SAM D the watchdog keeps running too, but the firmware
can switch it off.
