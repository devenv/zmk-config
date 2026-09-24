# toucan.keymap — port from seven

This is the beekeeb **Toucan2** (github.com/beekeeb/zmk-keyboard-toucan2),
confirmed by Boris (q413: "Toucan2 36 Keys Wireless Split Keyboard with
Multi-touch Trackpad" — the shop.beekeeb.com/products/toucan2 listing's
36-key variant), **not** the original Toucan (zmk-keyboard-toucan, single-
touch cirque trackpad) this lane started from. The shield directory is still
called `toucan` inside beekeeb's `zmk-keyboard-toucan2` repo — that's their
naming, not a leftover from the wrong board.

`toucan.keymap` ports `../seven/seven.keymap`'s 6 layers (same hold-tap
behaviors, same combo) onto Toucan2's physical key positions, plus one new
`mouse_layer` (see below).

## Key count: 36 vs 42 firmware, one PCB

seven and the "36 Keys" Toucan2 variant both have 36 keys. But beekeeb's own
`zmk-keyboard-toucan2` repo (and every one of ~50 customer-generated forks
checked, including one a buyer named `-36` after their own order) ships only
one firmware config, sized for the **42-key** layout (6 cols x 3 rows + 3
thumb, per hand). There is no separate 36-key firmware repo or build target
anywhere in that fork population — checked via GitHub's API, not assumed.

42 - 36 = 6 keys total = **3 keys per hand = exactly 1 unpopulated column
per hand** (3 rows x 1 col), not 2 as originally assumed below (that math
error is why the first flash was wrong — see next section). The 36-key SKU
is the same PCB/matrix with 1 column per hand left unpopulated, sharing one
firmware.

## Column mapping (per row, 0-indexed) — corrected after first flash

**First attempt (wrong):** assumed 2 unpopulated columns per hand, both at
the *innermost* edge next to the split gap (toucan cols 5,6), leaving
seven's columns 0-4 mapped straight across to toucan cols 0-4 unchanged.
Flashed to the left half and confirmed by Boris: pressing the physically
leftmost key produced `W`, not `Q` — i.e. seven's column 0 (`Q`) was landing
one position inward of where it actually needed to be. That's inconsistent
with a 2-column gap at the inner edge; it's exactly what you'd see if the
*single* unpopulated column is at the **outer** edge (toucan col 0 / col 11,
`SW1` on each hand) instead, pushing every real key one position inward.

**Corrected mapping**, matrix-transform's `SW1..SW6` per hand (`SW1`
outermost/pinky, `SW6` innermost, at the split gap — see `toucan.dtsi`):

| seven col  | -  | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | -  |
|------------|----|---|---|---|---|---|---|---|---|---|---|----|
| toucan col | 0* | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10| 11*|

`*` = toucan col 0 and col 11 (outermost each hand, `SW1`) — no seven
source, likely physically unpopulated on the 36-key board. Bound to `&none`
on every layer — no principled default to port, and easily reassigned later
via ZMK Studio (enabled for `toucan_left` in `build.yaml`). This mapping
(seven col N → toucan col N+1, uniformly) applies identically to all three
non-thumb rows, for all six ported layers. Thumb clusters are unaffected
(same size, 3 per hand, unchanged position — the outer-column shift only
touches the 3 main rows).

## Combo

`combo_print_screen_r_t` (R, T on row 0, `EXTRA` layer) shifted with the
column fix above: seven's flat positions `<3 4>` become toucan's `<4 5>`,
since R/T are seven columns 3/4 → toucan columns 4/5, and row 0's flat
key-position index equals its column index.

## New: `mouse_layer` (index 6, `MOUSE`)

Not a seven layer — added to preserve Toucan2 hardware behavior that would
otherwise silently break. `toucan.dtsi`'s `is_touching_processor` momentarily
activates a layer while a finger touches the trackpad (stock beekeeb keymap:
their own layer 4, "mouse", puts click buttons on the right thumb keys).
Left unchanged, it would have activated *this* keymap's layer 4 (`LNUM`,
seven's left-number layer) on every trackpad touch — a real collision, not
a cosmetic one.

Fix: added a 7th layer (`MOUSE`, index 6) that's `&trans` everywhere except
the right thumb cluster (`&mkp LCLK`/`RCLK`/`MCLK`, matching beekeeb's
stock positions), and repointed `is_touching_processor`'s binding in
`toucan.dtsi` from `&mo 4` to `&mo 6`. The trackpad's own native tap-to-click
/ two-finger-tap (`single-tap`, `two-finger-tap` in `toucan_right.overlay`)
already provides click gestures independently of this layer — this is an
alternate/backup click method, kept for parity with stock rather than
independently judged necessary.

## What did NOT change

- All 6 ported `#define` layer indices (DFLT=0 ... SYSTEM=5) — same values,
  same order, same behavior. `MOUSE=6` is additive.
- All hold-tap behaviors (`hpl`/`bpl`/`tpl`/`hpm`/`bpm`/`tpm`) — copied
  verbatim.
- Thumb row bindings on the 6 ported layers — copied 1:1 by position, with
  one exception: DFLT's right thumb #2 (between BACKSPACE and the outer
  `&mkp LCLK`) is `&mkp RCLK` instead of seven's `&mo SYMBOLS` (Boris,
  2026-09-24). SYMBOLS is still reachable via the left `&bpl SYMBOLS SPACE`
  hold, which also still drives scroll-while-held.
- Trackpad gestures (pinch-zoom, 3-finger swipe -> Mission Control, native
  tap-to-click) — all wired directly into `toucan.dtsi`'s input-processor
  chain, independent of keymap layers, untouched from beekeeb's stock config.
  `TOUCAN_WIN_MODE` stays undefined (Mac gesture shortcuts — Boris is on a
  Mac).
- Scroll-while-held (`trackpad_listener`'s `scroller` node, layers `<1 2>`)
  -- already matches this keymap's SYMBOLS(1)/EXTRA(2), no edit needed.
