# QMK Project — Claude Context

## Overview

This directory (`~/qmk/`) contains two repos for QMK keyboard firmware:

- **`qmk_firmware/`** — Fork of [qmk/qmk_firmware](https://github.com/qmk/qmk_firmware) (remote: `gagregrog/qmk_firmware`, branch: `gagregrog`). Contains keyboard hardware definitions. Upstream is `qmk/qmk_firmware`.
- **`qmk_userspace/`** — User overlay repo ([gagregrog/qmk_userspace](https://github.com/gagregrog/qmk_userspace)). Contains all keymaps, shared user code, and build targets.

QMK config points here:
```
user.qmk_home=~/qmk/qmk_firmware
user.overlay_dir=~/qmk/qmk_userspace
```

## Keyboards

All keyboards are fully migrated to data-driven `keyboard.json`.

| Keyboard | MCU | Bootloader | Pointing | LEDs | Split | Notes |
|---|---|---|---|---|---|---|
| `bastardkb/charybdis/3x5_3_h` | RP2040 | uf2 | PMW3389 trackball | RGB Matrix (WS2812) | Yes (vendor serial) | Primary daily driver |
| `bastardkb/dilemma/3x5_3_h` | RP2040 | uf2 | Cirque trackpad (SPI) | RGBLIGHT (2 LEDs) | Yes | Has OLED (SSD1306 I2C) |
| `handwired/dactyl_manuform/4x6/pro_micro` | ATmega32u4 | caterina | None | RGBLIGHT | Yes | Parent `4x6/info.json` is upstream |
| `handwired/dactyl_manuform/4x6/elite_c` | ATmega32u4 | atmel-dfu | None | RGBLIGHT | Yes | Same parent as pro_micro |
| `handwired/sim_pad/1x3` | RP2040 | uf2 | Cirque trackpad (SPI) | None | No | 3 direct-pin keys, OLED |
| `handwired/twokey` | ATmega32u4 | caterina | None | None | No | 2x2 sparse matrix (2 keys) |

## Build Targets (qmk_userspace/qmk.json)

```json
{
    "build_targets": [
        ["bastardkb/charybdis/3x5_3_h", "gagregrog"],
        ["bastardkb/dilemma/3x5_3_h", "gagregrog"],
        ["handwired/dactyl_manuform/4x6/pro_micro", "gagregrog"],
        ["handwired/dactyl_manuform/4x6/elite_c", "gagregrog"],
        ["handwired/sim_pad/1x3", "default"],
        ["handwired/twokey", "gagregrog"]
    ]
}
```

The sim_pad uses the `default` keymap from the firmware repo. All others use `gagregrog` keymaps from the userspace.

## Data-Driven Migration Rules

### Where settings belong

**`keyboard.json`** (leaf keyboard dir) — single source of truth for:
- features, matrix_pins, bootloader, processor, USB, tapping, diode_direction
- split config, rgb_matrix.driver, ws2812.driver, split.serial.driver
- usb.shared_endpoint, rgblight config

**`rules.mk`** — only for settings with no keyboard.json mapping:
- `POINTING_DEVICE_DRIVER`, `OLED_DRIVER`, `*_SUPPORTED` flags
- `OPT_DEFS`, `PICO_INTRINSICS_ENABLED`, `DEBUG_ENABLE`
- Custom flags like `USE_DEFAULT_TD_ACTIONS`

**`config.h`** — only for settings with NO mapping in `data/mappings/info_config.hjson`:
- SPI pins, I2C pins, `OLED_DISPLAY_*`, `CRC8_*`
- `RP2040_BOOTLOADER_*`, `CIRQUE_PINNACLE_*`
- Custom defines like `KB_DILEMMA`

**Critical rules:**
- Always check `data/mappings/info_config.hjson` before deciding where a setting belongs
- Parent keyboard dirs use `info.json`; leaf keyboard dirs use `keyboard.json`
- NEVER put `keyboard.json` in a parent dir when child keyboard dirs exist

## Userspace Structure

```
qmk_userspace/
  qmk.json                          # Build targets
  keyboards/                         # Per-keyboard keymap overrides
    bastardkb/charybdis/3x5_3_h/keymaps/gagregrog/
    bastardkb/dilemma/3x5_3_h/keymaps/gagregrog/
    handwired/dactyl_manuform/4x6/keymaps/gagregrog/
    handwired/twokey/keymaps/gagregrog/    # Also has keymaps/alexa/
  users/gagregrog/                   # Shared code across all keyboards
    gagregrog.c / .h                 # Entry point
    config.h / rules.mk             # User-level config
    combos/                          # Combo definitions
    keymaps/                         # Shared layout definitions
      common/keycodes.h              # Custom keycodes
      common/layers.h                # Layer definitions
      common/home_row_mods.h         # HRM setup
      common/unicode.h               # Unicode sequences
      layouts/layout_core.h          # Core keys (RGB conditional)
      layouts/layout_3x5_3.h         # Charybdis/Dilemma layout
      layouts/layout_4x6_6.h         # Dactyl 4x6 layout
    led/                             # RGB/LED utilities
    mouse_turbo_click/               # Mouse turbo click feature
    oled/                            # OLED display code + font
    overrides/                       # Key overrides
    secrets/                         # Secrets (gitignored)
    tap_dance/                       # Tap dance utilities + defaults
    utils/                           # Keycode utilities
```

## Keycode Conventions

RGB keycodes differ by feature:
- **RGB Matrix** (charybdis): `RM_TOGG`, `RM_NEXT`, etc.
- **RGBLIGHT** (dilemma, dactyl): `UG_TOGG`, `UG_NEXT`, `UG_VALU`, `UG_VALD`, etc.
- `layout_core.h` uses `#if defined(RGB_MATRIX_ENABLE)` / `#elif defined(RGBLIGHT_ENABLE)` conditionals

Mouse keycodes: `MS_BTN1`, `MS_BTN2` (not the old `KC_BTN*`)

## Building & Flashing

```bash
# Build all targets
qmk userspace-compile

# Build a single target
qmk compile -kb bastardkb/charybdis/3x5_3_h -km gagregrog

# Flash (RP2040 — put in bootloader mode first)
qmk flash -kb bastardkb/charybdis/3x5_3_h -km gagregrog

# Clean build cache (needed after deleting generated files)
qmk clean
```

Compiled artifacts (.uf2/.hex) are output to the userspace root.

## Key Lessons

- `keyboard.json` with matrix coordinates in layouts causes QMK to auto-generate the LAYOUT macro — delete any manual `.h` defining it
- If `<keyboard>.c` only contains `#include "<keyboard>.h"`, delete it (no-op)
- `qmk clean` is needed after deleting generated files to clear build cache
- Dactyl `chordal_hold_layout` needs 52 args (not 48) for `LAYOUT_split_4x6_6`
- The dactyl `4x6/info.json` is upstream-maintained — don't modify it

## Dotfiles Integration

- **`~/.dotfiles/.scripts/setup-qmk.sh`** — Clones repos to `~/qmk/`, sets up remotes, runs `qmk setup` and `qmk doctor`
- **`~/.dotfiles/.zshconfig/qmk.sh`** — Exports `QMK_HOME` and `QMK_USERSPACE` env vars, adds toolchain to PATH
