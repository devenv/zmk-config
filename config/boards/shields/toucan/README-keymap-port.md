# toucan.keymap — port from seven

`toucan.keymap` is a direct port of `../seven/seven.keymap` (same 6 layers,
same hold-tap behaviors, same combo), remapped onto the Toucan's physical
key positions. `seven.conf`'s settings are host/global, not shield-specific,
and are not part of this port — see `config/seven.conf` if those need
carrying over to Toucan too.

## Key count

- seven: 5 cols x 3 rows + 3 thumb keys, per hand -> 36 keys total.
- Toucan: 6 cols x 3 rows + 3 thumb keys, per hand -> 42 keys total.

Toucan is a strict superset column-wise: every seven key position exists on
Toucan, plus one new column per hand. **Nothing from seven was dropped.**
The thumb clusters are the same size (3 per hand) and map 1:1, left-to-right,
unchanged.

## Column mapping (per row, 0-indexed)

Toucan's physical layout inserts its extra column innermost — right next to
the split gap — one per hand, matching `boards/shields/toucan/toucan.dtsi`'s
matrix-transform ordering (`SW1..SW6` per hand, `SW6` innermost).

| seven col | 0 | 1 | 2 | 3 | 4 | -  | -  | 5 | 6 | 7 | 8 | 9 |
|-----------|---|---|---|---|---|----|----|---|---|---|---|---|
| toucan col| 0 | 1 | 2 | 3 | 4 | 5* | 6* | 7 | 8 | 9 | 10| 11|

`*` = new, no seven source. Left hand columns 0-4 keep identical indices;
everything right-hand shifts by +2 to make room for the two new inner
columns (5 = left-inner-new, 6 = right-inner-new).

This mapping is applied identically to all three non-thumb rows, for all
six layers (default/symbols/extra/gaming/left_num/system).

## The two new columns (5, 6)

Bound to `&none` (inert) on every layer. seven defines no source key for
these physical positions, so there's no principled default to port — rather
than guess a binding, they're left inert and easily reassignable later via
ZMK Studio (enabled in `build.yaml` for `toucan_left` — `-DCONFIG_ZMK_STUDIO=y`
+ `studio-rpc-usb-uart` snippet). Flagged to Boris as an open question rather
than picked unilaterally.

## Combo

`combo_print_screen_r_t` (key-positions `<3 4>` = R, T on row 0, `EXTRA`
layer only) needed no change: both positions live in the left hand's
columns 0-4, which are numerically unchanged between seven and Toucan.

## What did NOT change

- All 6 `#define` layer indices (DFLT=0 … SYSTEM=5) — same values, same layer
  order.
- All hold-tap behaviors (`hpl`/`bpl`/`tpl`/`hpm`/`bpm`/`tpm`) — copied
  verbatim, tapping terms and flavors unchanged.
- Thumb row bindings — copied 1:1 by position, no remapping needed (thumb
  cluster sizes match).

## Trackpad interaction (not part of the port, from beekeeb's stock Toucan
config, kept as-is)

`config/boards/shields/toucan/toucan.dtsi`'s `glidepoint_listener` scroller
sub-node activates scroll mode "while layer 1 or 2 is held" (`layers = <1 2>`).
Those numbers weren't changed for the port — they happen to already match
this keymap's `SYMBOLS` (1) and `EXTRA` (2) layer indices, so scroll-while-held
behaves the same as beekeeb's own default (scroll while SYM/NAV-equivalent
layer held) with zero edits needed.
