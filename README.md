# ZMK config for beekeeb Toucan2 Keyboard

[The beekeeb Toucan2 Keyboard](https://beekeeb.com/introducing-toucan2/) is a wireless split 42-key column‑stagger keyboard that a display and a trackpad, with an aggressive stagger on the pinky columns.

# Keymap

Images below are generated from [config/toucan.keymap](config/toucan.keymap) with
[keymap-drawer](https://github.com/caksoylar/keymap-drawer) — see [Regenerating the keymap images](#regenerating-the-keymap-images).

Legend per key: center = tap, bottom = hold, top = double-tap (`2✕`).
The outermost column on each half is `&trans` on every layer except ALT, where the top outer key on each
half is `&bootloader`.

| # | Layer | How to reach it |
|---|-------|-----------------|
| 0 | BASE  | default |
| 1 | NAV   | hold left inner thumb |
| 2 | SYM   | hold right inner thumb |
| 3 | GAME  | double-tap `D` |
| 4 | GNUM  | hold left inner thumb while on GAME, or touch the trackpad |
| 5 | RES1  | double-tap `C` (reserved, unmapped) |
| 6 | RES2  | double-tap `H` (reserved, unmapped) |
| 7 | RES3  | double-tap `,` (reserved, unmapped) |
| 8 | ALT   | hold both inner thumbs (NAV + SYM tri-layer) |

Every key on the reserved layers, and the right half of GNUM/ALT, is `&to 0` — a single tap gets you back to BASE
(except the two ALT reset keys described below).

## BASE

Colemak-DH. The bottom row is tap-dance mod-taps: hold `Z X C D` / `H , . /` for GUI / CTRL / ALT / SHIFT,
double-tap them for Bluetooth profile switching (`BT ←`, `BT →`, `BT CLR`), ZMK Studio unlock, or a layer jump.
Outer-column letters double-tap to copy/paste (`⌃C`, `⇧⌃C`, `⌃V`, `⇧⌃V`).

![BASE layer](keymap-drawer/img/base.svg)

## NAV

Arrows, paging, editing keys on the left; number pad and media transport on the right.
The trackpad scrolls instead of moving the cursor while this layer is held.

![NAV layer](keymap-drawer/img/nav.svg)

## SYM

Brackets and media on the left, symbols on the right. The trackpad also scrolls on this layer.

![SYM layer](keymap-drawer/img/sym.svg)

## ALT

Tri-layer: held automatically when NAV and SYM are both held. F1–F14, plus the firmware reset keys.

Reset keys live here because reaching ALT needs only the two thumb keys — no tap-dance — so they stay
reachable even if the home-row behaviors misbehave:

| Key | Binding | Effect |
|-----|---------|--------|
| Top outer, left half | `&bootloader` | Left half reboots into the UF2 bootloader and mounts as a USB drive — copy the `.uf2` in to flash. No physical reset button needed. |
| Top outer, right half | `&bootloader` | Same for the right half. |
| Innermost, bottom row, left half | `&sys_reset` | Soft-reboots the left half into the existing firmware. |
| Innermost, bottom row, right half | `&sys_reset` | Soft-reboots the right half. |

ZMK reset behaviors are source-specific: each key resets the half it physically sits on, which is why
there is one per side. (Combos are the exception — a combo-triggered reset always hits the central half —
so keep these as plain keys.)

Note that ZMK Studio cannot replace a flash for every change: it edits layer bindings live, but behavior
properties such as a tap-dance `tapping-term-ms` are compile-time devicetree values and still need new firmware.

![ALT layer](keymap-drawer/img/alt.svg)

## GAME

QWERTY with the mods moved off the home row and onto the pinky column, so WASD-style play works
without hold-taps firing. `F` double-taps to `T` and triple-taps to `tt`; `X` double-taps to `Z`
and triple-taps to `zz`.

![GAME layer](keymap-drawer/img/game.svg)

## GNUM

Numbers and paging for the GAME layer. Also activated for as long as a finger rests on the trackpad
(the `is_touching_processor` node in [boards/shields/toucan/toucan.dtsi](boards/shields/toucan/toucan.dtsi)).

![GNUM layer](keymap-drawer/img/gnum.svg)

## Reserved layers

RES1/RES2/RES3 are empty placeholders, reachable by double-tapping `C`, `H` and `,`.

![Reserved layers](keymap-drawer/img/reserved.svg)

## Regenerating the keymap images

```sh
pip install keymap-drawer

# re-parse the keymap, then hand-merge the tap-dance legends into keymap-drawer/toucan.yaml
keymap parse -z config/toucan.keymap

# redraw
for layer in BASE NAV SYM ALT GAME GNUM; do
    keymap draw keymap-drawer/toucan.yaml -s "$layer" -o "keymap-drawer/img/$(echo "$layer" | tr 'A-Z' 'a-z').svg"
done
keymap draw keymap-drawer/toucan.yaml -s RES1 RES2 RES3 -o keymap-drawer/img/reserved.svg
```

`keymap parse` renders tap-dances as raw `&tdN` references; [keymap-drawer/toucan.yaml](keymap-drawer/toucan.yaml)
spells them out as tap/hold/double-tap legends, so edit that file rather than overwriting it.

# Customizations

- **Keymap**: [config/toucan.keymap](config/toucan.keymap)
- **General configs**: [boards/shields/toucan/toucan_left.conf](boards/shields/toucan/toucan_left.conf) and [boards/shields/toucan/toucan_right.conf](boards/shields/toucan/toucan_right.conf)
- **Swipe shortcuts**: the `swipe_button_mapper` node in [boards/shields/toucan/toucan.dtsi](boards/shields/toucan/toucan.dtsi)
- **Invert scroll / trackpad settings**: the `tps43_trackpad` node in [boards/shields/toucan/toucan_right.overlay](boards/shields/toucan/toucan_right.overlay)

# License

The code in this repo is available under the MIT license.

The included shield nice_view_gem is modified from https://github.com/M165437/nice-view-gem licensed under the MIT License.

The linked trackpad module is based on https://github.com/geeksville/zmk_driver_azoteq

ZMK code snippets are taken from the ZMK documentation under the MIT license.

The embedded font QuinqueFive is designed by GGBotNet, licensed under under the SIL Open Font License, Version 1.1.
