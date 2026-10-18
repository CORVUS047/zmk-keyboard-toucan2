# ZMK config for beekeeb Toucan2 Keyboard

[The beekeeb Toucan2 Keyboard](https://beekeeb.com/introducing-toucan2/) is a wireless split column‑stagger
keyboard with a display and a trackpad, and an aggressive stagger on the pinky columns.

**This build is 36 keys**: three rows of five columns per half, plus three thumb keys per half. The shield's
physical layout in [boards/shields/toucan/toucan.dtsi](boards/shields/toucan/toucan.dtsi) declares 42
positions and the matrix scans six columns per half, but the outermost column on each half has no switch
fitted — which is why columns 0 and 11 are `&trans` on every layer, and why the generated images below
draw six phantom keys down the outer edges. Don't bind anything there; it is unreachable.

# Keymap

Images below are generated from [config/toucan.keymap](config/toucan.keymap) with
[keymap-drawer](https://github.com/caksoylar/keymap-drawer) — see [Regenerating the keymap images](#regenerating-the-keymap-images).

Legend per key: center = tap, bottom = hold, top = triple-tap (`3✕`) on BASE, double-tap (`2✕`) on GAME.
A double-tap on BASE just types the letter twice.
The outermost column on each half is `&trans` on every layer — those positions have no switch.

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

Colemak-DH. The bottom row holds mods — `Z X C D` / `H , . /` give GUI / CTRL / ALT / SHIFT — and
**triple-tap** for Bluetooth profile switching (`BT ←`, `BT →`, `BT CLR`) or a layer jump.
Outer-column letters triple-tap to copy/paste (`⌃C`, `⇧⌃C`, `⌃V`, `⇧⌃V`). Double-tapping any of these keys
types the letter twice, so `add`, `book` and `jazz` still work.

These keys are a hold-tap (`hm0`–`hm7`) wrapping a tap-dance (`td0`–`td7`), not a mod-tap nested *inside* a
tap-dance. Order matters: a hold is not an "interrupting keypress", so a tap-dance on the outside cannot
resolve until its full `tapping-term-ms` elapses, which stacked ~400ms of latency onto every modifier. With
the hold-tap outermost the modifier resolves on its own term, and the tap-dance only ever counts taps.
`require-prior-idle-ms = <150>` stops fast letter rolls from firing a modifier; `quick-tap-ms = <200>`
keeps the second and third taps of a tap-dance from being read as holds.

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
| Outer thumb, left half (`TAB` on BASE) | `&bootloader` | Left half reboots into the UF2 bootloader and mounts as a USB drive — copy the `.uf2` in to flash. No physical reset button needed. |
| Outer thumb, right half (`BSPC` on BASE) | `&bootloader` | Same for the right half. |
| Innermost, bottom row, left half | `&sys_reset` | Soft-reboots the left half into the existing firmware. |
| Innermost, bottom row, right half | `&sys_reset` | Soft-reboots the right half. |

Both inner thumbs are already held down to reach ALT, so press the outer thumb key with your index finger.

ZMK reset behaviors are source-specific: each key resets the half it physically sits on, which is why
there is one per side. (Combos are the exception — a combo-triggered reset always hits the central half —
so keep these as plain keys.)

ZMK Studio is compiled into the left half (`CONFIG_ZMK_STUDIO=y` plus the `studio-rpc-usb-uart` snippet in
[build.yaml](build.yaml)), but **no key is bound to `&studio_unlock`**, so it cannot be unlocked and cannot
change anything. That is deliberate: Studio's keymap is stored in the settings partition and silently
overrides the compiled `.keymap` until a "Restore Stock Settings" is performed, which is a confusing failure
mode to hit by accident. Bind `&studio_unlock` to something temporarily if you ever actually want it.

Studio could not replace flashing anyway — it edits layer bindings, but behavior properties such as a
tap-dance `tapping-term-ms` are compile-time devicetree values.

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
