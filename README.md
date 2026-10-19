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
| 3 | GAME  | triple-tap `D` |
| 4 | GNUM  | hold left inner thumb while on GAME, or touch the trackpad |
| 5 | RES1  | triple-tap `C` (reserved, unmapped) |
| 6 | RES2  | triple-tap `H` (reserved, unmapped) |
| 7 | RES3  | triple-tap `,` (reserved, unmapped) |
| 8 | ALT   | hold both inner thumbs (NAV + SYM tri-layer) |

Every key on the reserved layers, and the right half of ALT, is `&to 0` — a single tap gets you back to BASE
(except the two ALT reset keys described below). GNUM is the exception: its right half is a numpad, so the
escape there is the **right inner thumb**.

## BASE

Colemak-DH. Mods live on the **home row**: `A R S T` / `N E I O` give GUI / CTRL / ALT / SHIFT on hold,
and type their letter on tap. The top two rows are plain `&kp` — no tap-dance anywhere near them.

Home-row mods are a single hold-tap, `hrm`, with `&kp` on both branches. Nothing is nested inside it.
Earlier revisions wrapped a tap-dance in the hold-tap's tap branch; even with the hold-tap outermost that
left a perceptible delay on shifted letters, so the tap-dances moved off the mod keys entirely.
`flavor = "hold-preferred"` resolves the hold on the *press* of the next key, not its release, so a capital
appears immediately. `require-prior-idle-ms = <150>` stops fast letter rolls from firing a modifier;
`quick-tap-ms = <175>` lets a repeated letter repeat.

The bottom row keeps the tap-dances (`td0`–`td7`) on `Z X C D` / `H , . /`: a double-tap types the letter
twice (so `add`, `book` and `jazz` work), and a **triple-tap** switches Bluetooth profile (`BT ←`, `BT →`,
`BT CLR`) or jumps to a layer. Note that these triple-taps are now reachable with plain taps — typing an
ellipsis `...` fires `BT CLR` and unpairs the keyboard.

`td10`–`td13` and the copy/paste macros (`⌃C`, `⇧⌃C`, `⌃V`, `⇧⌃V`) are still defined but no longer bound
to any key, since the outer columns are plain letters now.

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

Both halves carry digits, but they send **different scancodes**:

| Half | Keycodes | Use |
|------|----------|-----|
| Left  | `N0`–`N9`, `PLUS`/`EQUAL`/`MINUS` | top-row digits — what most apps and text fields expect |
| Right | `KP_N0`–`KP_N9`, `KP_PLUS`/`KP_MINUS`/`KP_MULTIPLY`/`KP_DOT`/`KP_EQUAL` | a real numpad, for apps that distinguish it (games, CAD, spreadsheets) |

The right half is laid out like a physical numpad — digits in the middle three columns, `+` `-` `0` down the
inner column, `.` `*` `=` down the outer:

```
+  7 8 9 .
-  4 5 6 *
0  1 2 3 =
```

Tap the right inner thumb to return to BASE.

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
