---
title: Configuring a build
---

# Configuring a build

Everything about a build lives in one text file, `.config` by default. `mimiconf.py`
reads it, lets you change it, and builds from it. Use `-c path` to work on a different
file, which is handy when you keep one config per project.

## The menu

```
python mimiconf.py
```

Without a `.config` it starts at the board picker: maker, then category, then board. The
panel on the right (or at the bottom, in a narrow terminal) describes whatever is
highlighted: a board's chip, console pins and test status, or an option's help text,
default, what it depends on and roughly how much flash it costs.

The home screen has your board, the presets, one entry per group of options with a count
of what's switched on, the flash map, the output folder and **Build now**.

| Key | What it does |
|---|---|
| Up, Down, PgUp, PgDn, Home, End | move |
| Enter or Right | open a menu, toggle an option, edit a value |
| Space | toggle an option, or step to the next choice |
| Esc, Left or Backspace | go back |
| `/` | search options by name or help text |
| `?` | show the full help for the highlighted item |
| `P` | presets |
| `L` | flash map |
| `B` | save and build |
| `S` | save |
| `Q` | quit (asks whether to save) |

Options that can't be used right now are greyed out, and the panel says what they
need. For example, "Updates must confirm" is greyed out with a single slot, because there
is nothing to roll back to.

## The command line

```
python mimiconf.py boards                  # every board, by maker and category
python mimiconf.py defconfig <board>       # start a .config with that board's defaults
python mimiconf.py preset product          # apply a preset (no name: list them)
python mimiconf.py set SHELL_DIAG=n BOOT_VERIFY=once
python mimiconf.py show                    # every option with its current value, and warnings
python mimiconf.py layout                  # the flash map, without building
python mimiconf.py build                   # build; -o DIR to put it somewhere else
python mimiconf.py toolchains              # which compilers were found
```

Option names are the IDs in the [option reference](options.html). Booleans take `y` or
`n`.

## Presets

A preset resets every option to its default and then applies a handful of values. It's
a starting point; change anything afterwards.

| Preset | What you get |
|---|---|
| `minimal` | One slot, no console, no OS interface, the image hash checked on its first boot only. The smallest useful build. |
| `hobby` | Boot menu, command line with diagnostics, uploads over the console, A/B slots without trial boots, Zephyr and raw firmware accepted, no signatures. |
| `product` | A/B plus a golden recovery image, updates must confirm or roll back, watchdog on trial boots, crash-loop guard, anti-rollback, the OS interface, no diagnostics. |
| `secure` | Only images signed with your keys boot, the hash is checked every boot, no raw or RAM images, no diagnostics, and the bootloader wipes its RAM before handing over. You still have to set `KEYS`. |

Small chips turn some features off by default so the build fits (a 64 KB STM32 has no
room for the console and signatures together, for example). A preset leaves those off
and tells you which ones it skipped.

## The .config file

It looks like a kernel `.config`:

```
# MimiBoot-Mu configuration (python mimiconf.py)
MU_BOARD="nucleo-f411re"
MU_LAYOUT="ab"
# MU_AB_TRIAL is not set
MU_BOOT_VERIFY="every-boot"
MU_SIGNATURES=y
MU_KEYS="keys/mimiboot.pub"
MU_BREAK_IN_MS=300
```

You can edit it by hand. Unknown or inapplicable options are ignored when it's loaded.

## Where builds go

By default, `~/MimiBoot/<board>/`. Set **Output folder** (`OUTPUT_DIR`) to any path you
like, with or without quotes. `build -o DIR` overrides it for one build.

## Exporting the source

Switch on **Also export the configured source tree** (`EXPORT_SOURCE`) and every build
also writes `source/` next to the binaries. It contains exactly the files this
configuration compiles, its generated config header and linker script, and a Makefile:

```
cd ~/MimiBoot/nucleo-f411re/source
make                  # or: make CROSS=/path/to/arm-none-eabi-
```

That's useful for auditing what went into a build, or for building it somewhere without
Python. RP boards get the CMake project and the exact build command in `BUILD.txt`
instead of a Makefile.

## Signing keys

```
python tools/mbtool.py keygen --out keys/mimiboot
```

writes `keys/mimiboot.key` (the secret key: keep it out of git) and `keys/mimiboot.pub`.
Set `KEYS` to the `.pub` file and the public key is compiled into the bootloader. The file
can hold several keys, one per line.
