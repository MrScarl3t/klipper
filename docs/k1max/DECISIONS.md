# K1 Max modernization — decisions

Only consequential decisions. Each records the evidence behind it.

## D1 — Fix forward from `aa55619b`, pinning upstream `ee2c6fb6`

**Decision.** Development continues from the existing merge commit `aa55619b`
on branch `k1max/modernize-20261027`. The upstream target stays pinned at
`ee2c6fb6` (2026-10-21). No re-merge, no re-base of the pushed `alpha` branch.

**Why.** `aa55619b` merges `ee2c6fb6` cleanly: the resulting tree is exactly
upstream plus 33 added fork files and 5 modified upstream files
(`homing.py`, `probe.py`, `temperature_mcu.py`, `stepper.py`, `toolhead.py`).
The fork-side code that survived is small and reviewable. Re-doing the merge
would rewrite a published commit for no gain in compatibility, because the
blockers live in the vendor config/module layer, not in the merge itself.

**Cost.** The audit document's "adapt" dispositions for
`mcu.py`/`serialhdl.py`/`clocksync.py` were not carried out (D2). The audit
text must therefore be read as *intent*, not as a description of the tree.

## D2 — Accept the dropped non-critical-MCU reconnect support

**Decision.** Do not re-add `is_non_critical`, `reconnect_interval`,
`serialhdl.check_connect()`, or `clocksync.disconnect()`. The merge took
upstream for those three files and that is kept.

**Why.** (a) The feature is inert in this repository: no config bundle sets
`is_non_critical`. (b) The pre-merge audit itself ranked it the top
safety-invariant weakening, because it lets klippy run with a configured MCU
missing or stale. (c) Restoring it would re-introduce a 660-line divergence in
`klippy/mcu.py` against a moving upstream. The rationale "the vendor UART
needs it" is unverified — there is no evidence that the K1 needs reconnection
rather than a normal shutdown-and-restart.

**Consequence to record, not fix.** If a K1 Max deployment proves to need
tolerance for a missing MCU, that returns as a separate, explicitly-scoped
slice with a print-in-progress guard.

## D3 — `max_accel_to_decel` is not replaced by a numeric translation

**Decision.** Remove `max_accel_to_decel` from the bundles rather than
converting its value into `minimum_cruise_ratio`. Where a replacement is
wanted, set `minimum_cruise_ratio` to upstream's default `0.5` and record it.

**Why.** The two options are not equivalent. Upstream (`docs/Config_Changes.md`
20240313, removal confirmed 20251108) replaced a pseudo-acceleration limit with
a *ratio* of move distance that must be travelled at cruise speed. The
relation is `mcr_pseudo_accel = max_accel * (1 - minimum_cruise_ratio)`
(`klippy/toolhead.py:540`). Naively mapping `20000`/`20000` would imply
`minimum_cruise_ratio: 0.0`, i.e. disabling the feature and restoring the exact
short-move behaviour the upstream change was made to remove. A deliberate
default is safer and honest about the behaviour change.

## D4 — Treat `square_corner_max_velocity` as a dead vendor option

**Decision.** Delete it from `K1_MAX` and `K1C` `printer.cfg`.

**Why.** `git log --all -S'square_corner_max_velocity' -- klippy/ src/` returns
nothing: no code in this repository's history ever read it. It has existed only
as an unread `[printer]` key since the initial vendor config commit `2e906724`,
and `check_unused_options()` (`klippy/configfile.py:424`) raises on unread
options, so it now prevents startup. Deleting it changes no behaviour.

## D5 — Vendor pin assignments stay verbatim

**Decision.** No pin, rotation distance, microstep count, current, or thermistor
type in `config/K1*` is modified by this project unless a specific defect is
proven. Changes are limited to options upstream removed or renamed.

**Why.** Pins and electrical values come from the vendor configs and cannot be
derived from source. Guessing them risks hardware damage. The prtouch sensor
pins in particular (PA0–PA7 on `leveling_mcu`, swap pins `PC10`/`PB1`) are
copied as-is and remain unvalidated.

## D6 — Documentation lives in `docs/k1max/`, not in `CLAUDE.md`

**Decision.** Detailed records go in `docs/k1max/`; `CLAUDE.md` keeps only
short permanent rules.

**Why.** Token budget when a new session starts, and to keep rules separate
from findings that will change.

**Known gap.** `CLAUDE.md` points at `docs/k1-max-port.md`, which does not
exist. Correcting a user-authored instruction file was not done without
approval; the pointer is recorded here and in `VALIDATION.md` instead.
