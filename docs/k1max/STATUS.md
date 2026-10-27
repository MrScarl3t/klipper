# K1 Max modernization — status

Last updated: 2026-10-27

## Baseline

| Item | Value |
| --- | --- |
| Starting commit | `aa55619b66fa5aaf7d57c6013b966fa47a149d4e` |
| Fork remote (`origin`) | `https://github.com/MrScarl3t/klipper` |
| Upstream (`upstream`) | `https://github.com/Klipper3d/klipper`, branch `master` |
| Pinned upstream target | `ee2c6fb6b00b2b5d097ef70d0868df7d74943b51` (2026-10-21) |
| Divergence vs target | 0 behind, 57 ahead (fork commits) |
| Pre-merge backup ref | `refs/backup/pre-upstream-merge-20261027` → `3423dc08` |
| Development branch | `k1max/modernize-20261027` (tracks `origin/k1max/modernize-20261027`) |

Branch `alpha` is untouched at `aa55619b`; `master` is untouched at `3423dc08`.

## Last completed checkpoint

`docs(k1max): establish modernization baseline and compatibility matrix`
— records only, no source changes.

## Current work

Records in `docs/k1max/`. Nothing in `klippy/`, `src/`, or `config/` has been
modified on this branch yet.

## Test status

| Check | Result |
| --- | --- |
| Fork-modified Python modules compile (`compile()`) | pass, one pre-existing `SyntaxWarning` in `klippy/extras/tool.py:14` |
| `python klippy/klippy.py --import-test` on Windows | **fails** — no `termios` on Windows, no `cffi` installed |
| Linux test suite (`scripts/ci-build.sh`) | **not run** — WSL Ubuntu has no gcc/make/cffi/greenlet |
| MCU builds | **not run** |
| K1/K1C/K1_MAX config parse | **not run**; configs are not in `test/klippy/printers.test`, so there is no automated coverage |

No baseline is established yet. See `VALIDATION.md` for the exact blocker.

## Open blockers

1. **No reproducible Linux test environment.** Baseline requires a Python
   venv plus `build-essential` in WSL2. Approval for `apt` installs is pending.
2. **`[prtouch_v2]` cannot load.** `klippy/extras/prtouch_v2.py:16` passes
   `(config, vrt)` to `probe.PrinterProbe`, whose current signature is
   `(config)`. The vendor wrapper also exposes no `start_probe_session`, which
   the current probe path calls. Fatal for all three K1 config bundles.
3. **Config options removed upstream.** `max_accel_to_decel`,
   `square_corner_max_velocity`, `SET_VELOCITY_LIMIT ACCEL_TO_DECEL`,
   `printer.toolhead.max_accel_to_decel`, `printer.print_stats.z_pos`,
   `printer.print_stats.power_loss`. `check_unused_options()` raises on the
   first two classes, so klippy aborts at config load.
4. **MCU firmware is out of scope for this tree.** Upstream `src/` has no
   GD32F303/GD32E230 target; the K1 boards run vendor-built binaries.

## Exact next action

Establish the reproducible baseline (phase 8): in WSL2 Ubuntu, create an
isolated venv, install the test requirements from `scripts/klipper-requirements.txt`
and `scripts/tests-requirements.txt`, install `build-essential`, then run
`./scripts/check_whitespace.sh` and
`python scripts/test_klippy.py -d <dictdir> test/klippy/*.test` on the
untouched `aa55619b` tree. Needs approval for the `apt` install.

## Detailed records

- `PLAN.md` — phases, acceptance criteria, blockers
- `COMPATIBILITY.md` — component matrix with evidence
- `DECISIONS.md` — consequential decisions and rationale
- `VALIDATION.md` — commands, environments, results, untested areas
- `RECOVERY.md` — deployment prerequisites and recovery (not yet verified)
- `../k1-max-fork-audit.md` — pre-merge fork inventory and dispositions
