# K1 Max modernization — plan

Goal: a maintainable, tested update of this fork with explicitly documented
K1 Max compatibility. Not universal compatibility, not hardware readiness.

Base: `aa55619b` (fork tree containing the upstream `ee2c6fb6` merge).
Strategy: fix forward from `aa55619b`, re-applying only the audit items the
configured bundles actually need, one reviewed slice at a time. See
`DECISIONS.md` D1.

## Phase 0 — Preflight and preservation

- [x] Repository, remotes, branch, worktree, in-progress-operation audit.
- [x] Fidelity check of the `ee2c6fb6` merge against its upstream parent.
- [x] Preservation: branch `k1max/modernize-20261027` at `aa55619b`;
      `alpha` and `master` untouched; `refs/backup/pre-upstream-merge-20261027`
      retained.
- [x] Untracked `CLAUDE.md` from the prior session preserved verbatim.

## Phase 1 — Reproducible baseline (blocked)

Acceptance: `check_whitespace.sh` and the klippy regression suite run on the
untouched tree, with pre-existing failures recorded separately.

Blocker: WSL2 Ubuntu 26.04 has no gcc/make and no cffi/greenlet.
Needs `apt install build-essential python3.12-venv libffi-dev` plus a venv.

## Phase 2 — Host-side module compatibility

Slices, in dependency order:

- **S2.1 prtouch probe adapter.** `prtouch_v2.py` must construct a probe whose
  interface matches current `probe.PrinterProbe`. The wrapper has no
  `start_probe_session`, so either a shim adapter supplies one, or the vendor
  probe is wired through the older `ProbeEndstopWrapper` contract.
  Acceptance: `[prtouch_v2]` loads and `G28`/`G29` paths resolve to wrapper
  methods (`run_G28_Z`, `run_G29_Z`, `run_to_next`) without upstream API
  errors. Validation: config-load smoke test with a host-simulator dictionary.
- **S2.2 Config option migration.** Replace `max_accel_to_decel` with
  `minimum_cruise_ratio` (semantics differ — see D3), drop
  `square_corner_max_velocity`, migrate `ACCEL_TO_DECEL`/`SET_VELOCITY_LIMIT`
  macro calls, remove `printer.print_stats.z_pos`/`power_loss` references.
  Acceptance: all three bundles parse with no unused-option errors.
- **S2.3 `custom_macro.py`.** Remove the blocking `time.sleep()` pump loop and
  the `heaters.can_break_flag` dependency, which does not exist in this tree
  (pre-existing defect, present before the merge too). Keep the command surface.
- **S2.4 Dead modules.** Decide `metadata.py` (dead, unreferenced) and
  `tool.py` (unreferenced) — remove or document.

## Phase 3 — Config bundles

- Add a `test/klippy` smoke entry that parses `config/K1*` against a host
  simulator dictionary, so config regressions are caught without hardware.
- Keep pin assignments verbatim. No pin or electrical value may be invented.
- `config/K1*` retain vendor sections (`[prtouch_v2]`, `[fan_feedback]`,
  `[bl24c16f]`) that need the modules from Phase 2.

## Phase 4 — Deployment and recovery documentation

`RECOVERY.md` may only describe a procedure that has been verified for the
actual installation method. Until then it stays marked unverified.

## Phase 5 — Hardware validation (separate approval gate)

Not authorized. Requires printer access, firmware backup, and a staged plan
with abort criteria. Software validation does not imply hardware readiness.

## Known scope limits

- Upstream `src/` has no GD32F303/GD32E230 MCU target. Building replacement
  firmware for the K1 mainboard, toolhead, or leveling MCU from this tree is
  not possible without vendor patches. `fw/K1/*.bin` are vendor binaries and
  have not been inspected or validated.
- The prtouch wrapper `.so` files are CPython 3.8 / MIPS32 little-endian and
  ABI-bound to a host Python version. Any host Python change breaks them
  irrecoverably.
