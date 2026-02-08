# QMK Keyboards

Custom [QMK](https://qmk.fm/) keyboard firmware for gagregrog's keyboards.

## Directory Structure

```
~/qmk/
  qmk_firmware/   # Fork of qmk/qmk_firmware — keyboard hardware definitions
  qmk_userspace/  # Keymaps, shared user code, and build config
```

**qmk_firmware** is a fork of the upstream QMK repo. It contains `keyboard.json` hardware definitions for custom and modified keyboards. Keymaps are NOT stored here (except default keymaps shipped with the keyboard).

**qmk_userspace** is a standalone repo that overlays the firmware. It contains all personal keymaps (`keyboards/*/keymaps/gagregrog/`), shared code (`users/gagregrog/`), and build target definitions (`qmk.json`).

## Keyboards

| Keyboard | Type | Features |
|---|---|---|
| [Charybdis Nano](https://bastardkb.com/charybdis-nano/) | 3x5+3, split | PMW3389 trackball, RGB Matrix |
| [Dilemma v2](https://bastardkb.com/dilemma/) | 3x5+3, split | Cirque trackpad, OLED, RGBLIGHT |
| Dactyl Manuform 4x6 | 4x6+6, split | RGBLIGHT (pro_micro & elite_c variants) |
| Sim Pad | 1x3, single | Cirque trackpad, OLED |
| Two Key | 2 keys, single | Tap dances |

## Building

Requires [QMK CLI](https://docs.qmk.fm/#/newbs_getting_started).

```bash
# Build all targets
qmk userspace-compile

# Build one keyboard
qmk compile -kb bastardkb/charybdis/3x5_3_h -km gagregrog

# Flash
qmk flash -kb bastardkb/charybdis/3x5_3_h -km gagregrog
```

## Setup

```bash
# Run the setup script (clones repos, configures QMK)
~/.dotfiles/.scripts/setup-qmk.sh
```

Or manually:

```bash
git clone git@github.com:gagregrog/qmk_firmware.git ~/qmk/qmk_firmware
git clone git@github.com:gagregrog/qmk_userspace.git ~/qmk/qmk_userspace
qmk config user.qmk_home="$HOME/qmk/qmk_firmware"
qmk config user.overlay_dir="$HOME/qmk/qmk_userspace"
qmk setup
```
