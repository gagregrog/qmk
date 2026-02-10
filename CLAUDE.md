# QMK Project — Claude Context

## Overview

This directory (`~/qmk/`) contains two repos for QMK keyboard firmware:

- **`qmk_firmware/`** — Fork of [qmk/qmk_firmware](https://github.com/qmk/qmk_firmware) (remote: `gagregrog/qmk_firmware`, branch: `gagregrog`). Contains keyboard hardware definitions. Upstream is `qmk/qmk_firmware`.
- **`qmk_userspace/`** — User overlay repo ([gagregrog/qmk_userspace](https://github.com/gagregrog/qmk_userspace)). Contains all keymaps, shared user code, and build targets.

The primary typing layout is **Colemak-DH**, with QWERTY available as a toggleable alternative via `TG_BASE`.

QMK config points here:

```
user.qmk_home=~/qmk/qmk_firmware
user.overlay_dir=~/qmk/qmk_userspace
```

## Keyboards

All keyboards are fully migrated to data-driven `keyboard.json`.

| Keyboard                                  | MCU        | Bootloader | Pointing              | LEDs                | Split               | Notes                              |
| ----------------------------------------- | ---------- | ---------- | --------------------- | ------------------- | ------------------- | ---------------------------------- |
| `bastardkb/charybdis/3x5_3_h`             | RP2040     | uf2        | PMW3389 trackball     | RGB Matrix (WS2812) | Yes (vendor serial) | Primary daily driver               |
| `bastardkb/dilemma/3x5_3_h`               | RP2040     | uf2        | Cirque trackpad (SPI) | RGBLIGHT (2 LEDs)   | Yes                 | Has OLED (SSD1306 I2C)             |
| `handwired/dactyl_manuform/4x6/pro_micro` | ATmega32u4 | caterina   | None                  | RGBLIGHT            | Yes                 | Parent `4x6/info.json` is upstream |
| `handwired/dactyl_manuform/4x6/elite_c`   | ATmega32u4 | atmel-dfu  | None                  | RGBLIGHT            | Yes                 | Same parent as pro_micro           |
| `handwired/sim_pad/1x3`                   | RP2040     | uf2        | Cirque trackpad (SPI) | None                | No                  | 3 direct-pin keys, OLED            |
| `handwired/twokey`                        | ATmega32u4 | caterina   | None                  | None                | No                  | 2x2 sparse matrix (2 keys)         |

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

## Home Row Mods ("Timeless" Configuration)

Based on pgetreuer's port of urob's "ZMK Timeless Home Row Mods" to QMK. All HRM tap-hold settings are centralized in the userspace `config.h` and apply to all keyboards. Notes on the implementtation can be found here: https://www.reddit.com/r/ErgoMechKeyboards/s/TKTljFqDg4

### pgetreuer's notes

You cannot fetch from reddit, so below is a summary of the notes that pgetreuer prepared on this configuration:

```
Overview
A common struggle in this sub is configuring home row mods, aka HRMs, since [home row mods are hard to use](https://getreuer.info/posts/keyboards/faqs/index.html#home-row-mods-are-hard-to-use). Beyond merely adjusting the tapping term, QMK has [a cornucopia of tap-hold options](https://docs.qmk.fm/tap_hold), however, it is a lot to sift through to actually put together a good configuration.

urob's Timeless Home Row Mods is an excellent solution. urob [describes his configuration](https://github.com/urob/zmk-config?tab=readme-ov-file#timeless-homerow-mods) in detail and incrementally, describing how each option helps solve a problem. It is well explained and worth a read, though in ZMK terms. Here I'll describe Timeless Home Row Mods done analogously in QMK. The following described tap-hold options work equally for layer-tap keys, though our focus is on the home row mods use case.

TL;DR: The complete configuration
In your config.h, add:

#define TAPPING_TERM 250
#define PERMISSIVE_HOLD
#define FLOW_TAP_TERM 150
#define CHORDAL_HOLD
#define SPECULATIVE_HOLD
Large tapping term ⇒ "timelessness" ⌚
In its most basic description, a mod-tap key acts as the "mod" when held longer than the TAPPING_TERM and otherwise as another function, such as a letter key, when held for less than that. Like urob says, it is challenging to type with such consistent timing to use mod-tap keys based on this rule alone. This motivates a "timeless" configuration where how long keys are held does not matter. This is done by setting the TAPPING_TERM to a generous value, like 250 ms, or even larger:

#define TAPPING_TERM 250
Despite the name, the intention isn't that the behavior is literally timer-free; rather, this makes it timer-insensitive. HRMs are "timeless" in the sense that the behavior is mainly determined by what else you press, and insensitive to how long you press.

Responsive HRMs despite the large tapping term: Permissive Hold + Flow Tap
By default, a large tapping term introduces two problems: (1) you must hold the mod-taps for a sluggishly long time to invoke the modifier, and (2) there's a noticeable input lag during normal typing. We'll address these like urob does through QMK options:

#define PERMISSIVE_HOLD
#define FLOW_TAP_TERM 150
For the first problem, use [Permissive Hold](https://docs.qmk.fm/tap_hold#permissive-hold) (analogous to ZMK "balanced" flavor). Then, supposing A is a mod-tap key, a "nested" press like "A ↓, B ↓, B ↑, A ↑" results in A being settled as held, even if the whole sequence is typed within the tapping term. This enables quick use of mod-taps, independent of how long the tapping term is!

For the second problem, use [Flow Tap](https://docs.qmk.fm/tap_hold#flow-tap) (analogous to ZMK's require-prior-idle-ms). With this option, when a mod-tap is pressed within a FLOW_TAP_TERM timeout of the preceding key, the tapping behavior is immediately triggered. Effectively, it disables HRMs during fast typing, eliminating input lag. A timeout of 150 ms for FLOW_TAP_TERM is a good starting point, though you may want to tune it higher or lower. urob's suggestion, according to your "relaxed" typing speed:

Opposite hands rule: Chordal Hold
The above alone is already a pretty nice configuration, however, you may find that Permissive Hold sometimes falsely triggers modifiers in [rolls](https://getreuer.info/posts/keyboards/glossary/index.html#roll), where one hand rapidly presses adjacent keys. This is solved using [Chordal Hold](https://docs.qmk.fm/tap_hold#chordal-hold) (analogous to ZMK's positional hold-tap with hold-trigger-on-release), to implement an "opposite hands rule":

#define CHORDAL_HOLD
With Chordal Hold + Permissive Hold, for the nested press to trigger the hold behavior within the tapping term, the mod-tap key must be chorded with a key on the opposite hand. Same-hand chords are immediately settled as tapped. This solves the rolling issue.

Edit: As an exception to the "opposite hands" rule, Chordal Hold allows combining multiple same-side modifiers within the tapping term. For instance if A and B are two left-hand-side mod-tap keys and C is a right-hand side key, then an input of "A ↓, B ↓, C ↓, C ↑" results in Aand B settled as held. This is useful for multi-mod hotkeys like Ctrl+Shift+V. The exact rules are subtle, but the essence of it is Chordal Hold balances between guarding against accidental mod fires in rolls while yet supporting multiple same-side mods—see [this PR comment](https://github.com/qmk/qmk_firmware/pull/24560#discussion_r1894655238) for gory details. (Thank you to u/ldebritto for raising this point.)

HRMs and mouse use 🖱️
While Chordal Hold's opposite hands rule is helpful during normal typing, it gets in the way of performing hotkey chords one handed, say, while the other hand is using the mouse.

Even with Chordal Hold, it is yet possible to perform one-handed hotkey chords by pressing the mod-tap key, waiting out the tapping term (still holding the mod-tap key), and then finally tapping the other key. urob notes that because of this use case, one might not want to set the TAPPING_TERM to "infinity." Rather, you want it to be high enough to prevent accidental mod triggers during typing, yet low enough to practically wait out as an escape hatch for one-handed hotkeys.

Another mouse-related issue is that modifier + mouse inputs like Shift+clicking or Ctrl+scrolling require waiting out the tapping term, which feels laggy and annoying. To solve that, use [Speculative Hold](https://docs.qmk.fm/tap_hold#speculative-hold) (analogous to ZMK's hold-while-undecided).

#define SPECULATIVE_HOLD
When a mod-tap key is pressed, the modifier is applied immediately. Supposing the mod-tap key is latter settled as tapped, the modifier is cancelled before sending the tapping key.

Advanced configuration and fine tuning
Here is how to troubleshoot and tune:

Noticeable delay when tapping HRMs: Increase FLOW_TAP_TERM.

False negatives (same-hand): Reduce TAPPING_TERM (or disable Chordal Hold)

False negatives (cross-hand): Reduce FLOW_TAP_TERM

False positives (same-hand): Increase TAPPING_TERM

False positives (cross-hand): Increase FLOW_TAP_TERM

In the above, "false positives" mean triggering modifiers accidentally, while a "false negatives" mean failing to trigger modifiers when they are desired:

Additionally, all options are per-key configurable (see TAPPING_TERM_PER_KEY, PERMISSIVE_HOLD_PER_KEY, get_speculative_hold()), or even more finely, are per-chord configurable. Personally, I find it helpful to:

Set shorter timeouts on my Shift HRMs, through TAPPING_TERM_PER_KEY + the get_tapping_term() callback. I use HRMs for shifting, rather than a thumb shift key like urob does.

Enable Flow Tap only on my pinky HRMs, through get_flow_tap_term().

For Chordal Hold, use the [get_chordal_hold()](https://docs.qmk.fm/tap_hold#per-chord-customization) callback to define a few exceptions to the "opposite hands" rule.

Hopefully something in this guide has been helpful to you. Enjoy your HRMs!

QMK documentation references:

[Tapping term](https://docs.qmk.fm/tap_hold#tapping-term)

[Permissive Hold](https://docs.qmk.fm/tap_hold#permissive-hold)

[Flow Tap](https://docs.qmk.fm/tap_hold#flow-tap)

[Chordal Hold](https://docs.qmk.fm/tap_hold#chordal-hold)

[Speculative Hold](https://docs.qmk.fm/tap_hold#speculative-hold)

Related links:

[my keymap](https://github.com/getreuer/qmk-keymap) – an example implementation

[Home row mods are hard to use](https://getreuer.info/posts/keyboards/faqs/index.html#home-row-mods-are-hard-to-use)

[A guide to home row mods](https://precondition.github.io/home-row-mods)

[Layout Buffet – Home-row mods](https://blog.zsa.io/layout-buffet-home-row-mods/)
```

### Core settings (userspace `config.h`)

| Define                   | Value | Purpose                                                           |
| ------------------------ | ----- | ----------------------------------------------------------------- |
| `TAPPING_TERM`           | 250   | Large term makes behavior timer-insensitive                       |
| `TAPPING_TERM_PER_KEY`   | flag  | Enables `get_tapping_term()` callback for per-key tapping terms   |
| `QUICK_TAP_TERM_PER_KEY` | flag  | Enables `get_quick_tap_term()` callback for per-key quick tap     |
| `PERMISSIVE_HOLD`        | flag  | Quick mod activation via nested press (A↓ B↓ B↑ A↑ = hold)        |
| `FLOW_TAP_TERM`          | 150   | Disables HRMs during fast typing to eliminate input lag           |
| `CHORDAL_HOLD`           | flag  | Opposite hands rule prevents false triggers in rolls              |
| `SPECULATIVE_HOLD`       | flag  | Immediate modifier application for responsive mouse/hotkey combos |

### HRM order: CAGS (Ctrl, Alt, Gui, Shift)

Defined in `keymaps/common/home_row_mods.h`. Shift is on the index fingers (T left, N right in Colemak-DH).

### Per-key shift tuning (`gagregrog.c`)

Shift mod-taps (`LSFT_T`/`RSFT_T`) get special per-key overrides. pgetreuer himself recommends these in his guide and uses them in [his keymap](https://github.com/getreuer/qmk-keymap) (he uses `TAPPING_TERM - 45` with a base of 240ms).

**`get_tapping_term()` — shift gets `TAPPING_TERM - 50` (200ms)**

Why: In a rolling pattern (Shift↓ Letter↓ Shift↑ Letter↑), QMK's state machine processes the shift release as a "first tap" if it happens within the tapping term — *before* Chordal Hold or Permissive Hold can evaluate (those trigger on the *other* key's release, not the mod-tap's own release). The keyboards previously used `TAPPING_TERM 175` at the keyboard level, and the user's natural shift hold duration (~180ms) exceeded that, resolving as a hold. When the global term was raised to 250ms for the timeless config, rolling shifts started resolving as taps. A shorter per-key tapping term for shift restores the timing safety net.

Tuning: If shift still misfires as tap, reduce toward 175ms (`TAPPING_TERM - 75`). If false shift triggers occur during fast typing, increase toward 215ms (`TAPPING_TERM - 35`).

**`get_quick_tap_term()` — shift returns 0 (disabled)**

Why: `QUICK_TAP_TERM` (defaults to `TAPPING_TERM`) controls retap-as-repeat — if a mod-tap is tapped and pressed again within this window, the second press immediately registers the tap keycode (bypassing hold evaluation). Since T and N are extremely common letters in English, frequently retapping the shift key within the quick-tap window when shift is actually intended is a real risk. Setting it to 0 for shift ensures every press goes through full tap-hold evaluation.

Trade-off: Holding T or N for OS key repeat no longer works (every hold resolves as shift after the tapping term). Double-tap the letter instead if repeat is needed.

### Chordal Hold (`gagregrog.c`)

- `chordal_hold_layout` arrays defined per layout (`LAYOUT_split_3x5_3_h` with 30 entries, `LAYOUT_split_4x6_6` with 52 entries)
- Thumb keys marked `'*'` (exempt from opposite-hands rule)
- `get_chordal_hold()` callback adds exceptions for `RSFT_T(KC_N)` + `KC_QUOT`/`KC_SLASH`/`KC_SCLN` (same-hand shift for `"`, `?`, `:`)

### Flow Tap (`gagregrog.c`)

- `get_flow_tap_term()` callback disables Flow Tap for shift mod-taps (returns 0 for any `LSFT_T`/`RSFT_T` key)
- This prevents fast typing from resolving shift as a tap before Chordal Hold can evaluate (e.g., typing `?` quickly)
- All other mod-taps use the default Flow Tap behavior

### Important: keyboard-level tapping settings

Keyboard-level `tapping.term` or `TAPPING_TERM` overrides the userspace value. The charybdis and dilemma previously had `TAPPING_TERM 175` at the keyboard level — these were removed so the userspace 250ms applies universally. Only `tapping.toggle` (for `TT()` keys) remains at the keyboard level.

### QMK tap-hold state machine notes

Understanding why the per-key shift tapping term matters requires knowing how QMK's `process_tapping()` evaluates events while a mod-tap is in the "pressed/undecided" state (within the tapping term):

1. **Own release** → always resolves as "first tap" (TAP), regardless of Chordal Hold or Permissive Hold
2. **Other key release + Chordal Hold says not a chord** (same hand) → TAP
3. **Other key release + Permissive Hold** (opposite hand, nested press) → HOLD
4. **Tapping term expires** (no events or next event is past the term) → HOLD

The critical insight is that rule #1 takes priority: if the mod-tap key itself is released before any other key is released, it's *always* a tap. Chordal Hold and Permissive Hold only evaluate on the *other* key's release (rules #2/#3). This means the tapping term (rule #4) is the only protection against the rolling pattern (Shift↓ Letter↓ Shift↑ Letter↑) resolving as a tap.

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

## VS Code / clangd Setup

A multi-root workspace (`qmk.code-workspace`) provides three roots: `qmk` (parent), `qmk_firmware`, `qmk_userspace`.

### IntelliSense via compile_commands.json

QMK's build system generates a compilation database that gives clangd the exact include paths, defines, and flags for every source file:

```bash
# Generate for a specific keyboard (also compiles)
qmk compile --compiledb -kb bastardkb/charybdis/3x5_3_h -km gagregrog
```

This creates `qmk_firmware/compile_commands.json`. Clangd finds it automatically for firmware files. The userspace `.clangd` has `CompilationDatabase: ../qmk_firmware` to point clangd at the same database.

**Regenerate when:** You add/remove source files, change `rules.mk` includes, or need IntelliSense for a different keyboard's conditional code. After regenerating, restart clangd (`Cmd+Shift+P` > "clangd: Restart Language Server").

**Per-keyboard caveat:** The database reflects one keyboard's build. Code inside conditionals for other keyboards (e.g., `#if defined(RGBLIGHT_ENABLE)` when built for charybdis which uses RGB Matrix) won't resolve. Regenerate with the relevant keyboard if needed.

### Workspace settings

- `git.detectSubmodules: false` — prevents VS Code from tracking qmk_firmware submodules (lib/chibios, lib/lufa, etc.)
- `files.exclude` — hides build artifacts (`.build/`, `*.hex`, `*.bin`, `*.uf2`) and child dirs from parent root
- `files.associations` — maps QMK JSON files to JSONC, `.h`/`.c` to C
- `clangd.arguments: ["--header-insertion=never"]` — prevents clangd from auto-inserting headers

### .clangd files

Both repos have `.clangd` configs that strip embedded-specific compiler flags clang doesn't understand (`-mmcu`, `-mcpu`, etc.) and suppress false-positive diagnostics (ASM constraints, unknown attributes). The userspace `.clangd` mirrors the firmware's settings.

## Dotfiles Integration

- **`~/.dotfiles/.scripts/setup-qmk.sh`** — Clones repos to `~/qmk/`, sets up remotes, runs `qmk setup` and `qmk doctor`
- **`~/.dotfiles/.zshconfig/qmk.sh`** — Exports `QMK_HOME` and `QMK_USERSPACE` env vars, adds toolchain to PATH
