# K1 Max fork audit (pre-upstream-merge)

This document inventories every modification this fork carries relative to
official Klipper, classifies it, and records the disposition chosen for the
port to current upstream. It was written **before** the upstream update was
integrated, so it reflects the fork as it stood at `3423dc08`.

## 1. Repository and Git baseline

| Item | Value |
| --- | --- |
| Repository | `C:\Users\Thijs\gitproj\klipper` |
| Remote `origin` | `https://github.com/MrScarl3t/klipper` (fork, **not** upstream) |
| Original fork HEAD | `3423dc0896c4dc12c9173d86c493dac8c640462b` (2024-04-08, "Merge remote-tracking branch 'upstream/master'") |
| Merge base with upstream | `01c7befacb5b71643130433f2afb3f22ece68d71` (2024-04-05) |
| Last upstream commit already merged into fork | `deed38a6` (merged `71604b71`) |
| Local work branch | `alpha` |
| Pre-merge backup ref | `refs/backup/pre-upstream-merge-20261027` → `3423dc08` |
| Official upstream | `https://github.com/Klipper3d/klipper` |
| Upstream default branch | **`master`** (verified via remote HEAD; upstream has no `main`) |
| Fetched upstream target commit | `ee2c6fb6b00b2b5d097ef70d0868df7d74943b51` (2026-10-21) |
| Remote `upstream` | added locally, fetch-only; no push configured |

Upstream commits in `01c7befa..ee2c6fb6`: **1166**.

Verification notes:

- `origin/HEAD -> origin/master` is the *fork's* master, which is also at
  `3423dc08`. It is not an upstream tracking branch. The fork never had a
  distinct fork-only mainline: `alpha` and `master` were created at the same
  commit.
- The fork's tree is not a mirror of `klipper3d/klipper` at any tag: it is a
  fork of upstream plus 35 non-merge commits of Creality vendor code
  (directories `config/K1*`, `fw/K1/`, `klippy/extras/*.py`).

## 2. Fork-only commit inventory

35 non-merge commits between the merge base and the fork HEAD. Listed in
topological order; → marks commits whose effect was later reverted inside the
fork itself and therefore is **not** part of the effective delta.

| Commit | Date | Subject | Effective? |
| --- | --- | --- | --- |
| `1c59d01c` | 2023-12-14 | Add FW folder for mcu_update init.d script | yes |
| `2e906724` | 2023-12-14 | Add K1/Max cfg files | yes |
| `dc3f84e3` | 2023-12-14 | `#sent_time` alteration in clocksync.py | yes |
| `a9775ce0` | 2023-12-14 | Update bed_mesh.py with creality code | → reverted by `e2f1171a` |
| `12cfe7d7` | 2023-12-14 | add prtouch to homing.py | yes |
| `9917f1c5` | 2023-12-14 | patch print_stats.py for z_pos | → reverted by `34478db4` |
| `8a71515d` | 2023-12-14 | add patches for prtouch to probe.py | yes |
| `b153c0ec` | 2023-12-14 | Add `get_pin_info` to stepper.py | yes |
| `0deadf1d` | 2023-12-14 | creality patches to toolhead.py | → reverted by `9446f72e` |
| `4d4e161e` | 2023-12-14 | add creality files (7 new extras + chelper .so) | yes |
| `e2f1171a` | 2023-12-14 | Revert bed_mesh.py patch | (revert) |
| `ca5b718a` | 2023-12-14 | add prtouch wrapper .so's | yes |
| `693d1e51` | 2023-12-14 | add patch to temperature_mcu for gd32 soc | yes |
| `ca8821fd` | 2023-12-16 | chelper `__init__.py` | yes |
| `b4de9783` | 2023-12-16 | Create chelper Makefile | yes |
| `eec22cd2` | 2023-12-16 | replace `c_helper.so` | yes |
| `44a6a3fc` | 2023-12-16 | fix `stepper_z_sensorless_flag` init | yes |
| `245519f5` | 2023-12-19 | K1 printer.cfg −6 lines | yes |
| `259f03c0` | 2023-12-22 | `.github` config.yml | yes (unrelated) |
| `412e0f5a` | 2023-12-22 | `.github` config.yml gutting | yes (unrelated) |
| `9446f72e` | 2023-12-24 | Revert toolhead.py patch | (revert) |
| `34478db4` | 2023-12-26 | Revert print_stats.py patch | (revert) |
| `19309aaa` | 2024-01-20 | Delete one fw .bin | yes |
| `e349edd7` | 2024-01-20 | Add fw .bin | yes |
| `71736abd` | 2024-01-22 | Fix missing MoveQueue in prtouch calls | → reverted by `4826595b` |
| `4826595b` | 2024-01-22 | toolhead: backfill `move_queue` | yes |
| `5f9df017` | 2024-02-14 | Updated PRtouch v2 | yes |
| `e620de1f` | 2023-08-02 | noncritical mcus | yes |
| `499de9ee` | 2023-08-02 | support klipper mcus | yes |
| `62a0f667` | 2024-02-17 | Delete prtouch_v1 wrapper .so | yes |
| `e1288ae1` | 2024-02-17 | Delete prtouch_v2.py | yes |
| `1dd56459` | 2024-02-17 | K1C: re-add prtouch_v1 .so + prtouch_v2.py | yes |
| `9a69e58a` | 2024-02-17 | K1C Config Files | yes |
| `819d301a` | 2024-03-09 | Update custom_macro.py | yes |

Effective diff `01c7befa..3423dc08`: **44 files, +6806 / −29**.

Two whole categories deserve emphasis because they are easy to misread as
"vendor code still in use":

- **bed_mesh.py** and **print_stats.py** vendor patches were added and then
  reverted by the fork author. The fork's effective tree contains *upstream*
  bed_mesh.py and print_stats.py, yet the config files still call the vendor
  extensions those patches would have provided (`printer.print_stats.z_pos`,
  `printer.print_stats.power_loss`, `CHECK_BED_MESH`). Those hooks are
  **dangling** in the fork as shipped (see §5).
- **toolhead.py** vendor patch was likewise reverted; only the one-line
  `self.move_queue = self.lookahead` backfill from `4826595b` remains.

## 3. Classification scheme

- **K1-specific** — meaningless or unsafe on generic hardware; only exists
  because the printer is a Creality K1 series machine.
- **General fix** — a genuine defect fix that upstream does not have, but
  which is not hardware-specific.
- **Packaging/integration** — vendor integration surface: prebuilt binaries,
  external host software contracts, config bundles.
- **Unrelated** — does not serve the K1 support goal.

Dispositions: **retain** (keep as-is), **adapt** (keep, but reimplement
against current upstream APIs), **replace** (drop fork code in favour of an
upstream equivalent), **remove** (delete), **investigate** (cannot be
resolved by code inspection alone).

## 4. Modification-by-modification audit

### 4.1 `klippy/chelper/__init__.py` + `c_helper.so` + `Makefile`

- Commits: `4d4e161e`, `ca8821fd`, `893f3569`, `b4de9783`, `eec22cd2`.
- What: the auto-build branch of `get_ffi()` is commented out, so klippy
  always loads the vendored `klippy/chelper/c_helper.so` (32-bit MIPS32r2
  ELF with debug info, 63532 bytes) instead of compiling the C helpers for
  the host. A hand-written `Makefile` was added to rebuild it manually.
- Why: the printer's host is a MIPS32 little-endian SoC; the target image
  ships no gcc, so the fork vendors a cross-compiled `.so`.
- Classification: packaging/integration.
- Upstream equivalent: no. Upstream always builds `c_helper.so` at first run
  and `cffi` asserts every symbol it declares.
- **Verified incompatibility**: the vendored `.so` exports the merge-base
  chelper API. Against upstream `ee2c6fb6` it is missing
  `steppersync_alloc_syncemitter`, `steppersync_setup_movequeue`,
  `itersolve_get_trapq`, `itersolve_get_gen_steps_pre_active`,
  `itersolve_get_gen_steps_post_active`, `trdispatch_mcu` (renamed) and
  `pyhelper_logging_callback`. `cffi` raises at import time on any missing
  symbol, so **the vendored `.so` cannot work with current upstream code**.
- Disposition: **replace**. Restore upstream's build path verbatim and stop
  depending on the vendored binary. The `.so` remains in the tree for the
  vendor's own image but is no longer preferred; `*.so` is gitignored anyway.
  Building for the printer now requires a cross-toolchain on the build host
  (documented as a deployment step, not a code change).

### 4.2 `klippy/clocksync.py`

- Commits: `dc3f84e3`, `e620de1f`.
- What: adds `disconnect()` (stops the `get_clock` timer), and retries
  `get_uptime` once when `params['#sent_time']` is falsy before using it as
  `time_avg`.
- Why: needed by the non-critical-MCU reconnect path; the retry works around
  a first-response race on the vendor UART.
- Classification: K1-specific (only exercised by §4.7).
- Upstream equivalent: no `disconnect()`. Upstream clocksync is otherwise
  unchanged in the relevant area but `is_active()` was added upstream, which
  the restart helper now consults.
- Disposition: **adapt** — keep `disconnect()` (renamed/behaviour unchanged)
  and the retry, re-expressed against upstream's current `ClockSync` (which
  now guards with `is_active()`). Must not break upstream's
  `connect_file()` path.

### 4.3 `klippy/extras/temperature_mcu.py`

- Commit: `693d1e51`.
- What: new `gd32f303xe` MCU type entry using a fixed slope
  `3.3 / -0.004100` and `base_temperature = calc_base(25., 1.45/3.3)`.
- Why: the vendor GD32F303RE firmware reports a raw ADC value for which the
  generic STM32 factory-calibration path yields nonsense; the vendor constant
  matches the on-die sensor of that part.
- Classification: K1-specific.
- Upstream equivalent: no (upstream supports STM32/GD32 families generically
  but not `gd32f303xe`).
- Safety: this only affects `temperature_mcu` reporting of the MCU's own die
  temperature, which the K1 config exposes as `[temperature_sensor
  mcu_temp]` with `min_temp: 0 / max_temp: 100`. An inaccurate slope can
  therefore trip or mask a `temperature_mcu` fault, but it is not a heater
  control loop. **Not a thermal-protection weakening** — `verify_heater` and
  extruder/bed limits are untouched.
- Disposition: **retain**, adapted to upstream's current `cfg_funcs` tuple
  layout.

### 4.4 `klippy/stepper.py` — `get_pin_info()`

- Commit: `b153c0ec`.
- What: exposes `(dir_pin, step_pin, invert_dir, invert_step)` of an
  `MCU_stepper`.
- Why: consumed by the closed-source prtouch wrapper `.so` (symbol present in
  the binary at load offset, see §4.9) to drive the Z stepper directly.
- Classification: K1-specific.
- Upstream equivalent: none. Upstream `_dir_pin`/`_step_pin` still exist with
  the same names in `MCU_stepper.__init__` at `ee2c6fb6`.
- Disposition: **retain** — a 2-line additive accessor; verified harmless.

### 4.5 `klippy/toolhead.py` — `move_queue` alias

- Commits: `71736abd` → `4826595b`.
- What: `self.move_queue = self.lookahead` after the LookAheadQueue is
  created.
- Why: the closed-source prtouch wrappers were built against the pre-2018
  Klipper internal name `move_queue`; without the alias they raise
  `AttributeError` at `probe_prepare()`.
- Classification: packaging/integration (ABI shim for a binary blob).
- Upstream equivalent: none; upstream has used `lookahead` for years.
- Disposition: **retain** as a one-line clearly-commented compatibility alias.
  Verified `LookAheadQueue` is still the object stored in `self.lookahead` at
  `ee2c6fb6`.

### 4.6 `klippy/extras/homing.py`

- Commits: `12cfe7d7`, `44a6a3fc`.
- What:
  1. `Homing.stepper_z_sensorless_flag` (initialised `False`) which, when
     true, calls `hmove.homing_move(homepos, speed, False, True, False)`.
  2. `PrinterHoming.probe_type` = `"prtouch_v2"` if a `[prtouch_v2]` section
     exists, else `"bltouch"` if `[bltouch]` exists.
  3. `_probe()` bypasses `HomingMove` entirely and calls
     `probe.mcu_probe.run_G29_Z()` for `prtouch_v2`.
  4. `cmd_G28()` homes X and Y per-axis through the kinematics, then calls
     `probe.mcu_probe.run_G28_Z()` for Z, instead of `kin.home(state)` for
     the whole axis set.
  5. New `STEPPER_Z_SENEORLESS` gcode command (note the typo in the name)
     that calls `kin.home_z_with_sensorless(...)`.
- Why: the prtouch device does its own Z sweep and probing inside the
  closed-source wrapper; klippy must hand Z homing to it and must not
  interleave its own endstop moves.
- Classification: K1-specific.
- Upstream equivalent: no, and upstream has **rewritten** this area: `Homing`
  is now a thin facade, kin classes own `home_rails()`, probe-based Z homing
  goes through `_do_home_z_via_probe()` with a probe *session*, and
  `PrinterHoming` grew `probing_move()` and a mandatory
  `manual_home(..., probe_pos, triggered, check_triggered)` signature.
- Item 5 is **dead code**: `home_z_with_sensorless` is not defined by any
  kinematics class in this repository (grep over `klippy/` finds only the
  call site and the `hasattr` guard). Calling
  `STEPPER_Z_SENEORLESS` raises `AttributeError`. No config or macro invokes
  it.
- Item 1's flag is never set to `True` anywhere in the repository, so the
  branch is unreachable.
- Disposition: items 1 and 5 **remove** (dead, unverifiable, and item 1's
  call signature is upstream-incompatible). Items 2–4 **adapt**: keep the
  prtouch dispatch but re-express it against upstream's new homing structure,
  which means hooking `kin.home()` results *before* serialising Z, and
  routing Z through the wrapper. This is the highest-risk item in the port
  because upstream's Z-homing semantics changed; see §7.

### 4.7 `klippy/mcu.py`, `klippy/serialhdl.py`, `klippy/clocksync.py` — non-critical MCUs

- Commits: `e620de1f`, `499de9ee`.
- What:
  - `MCU` keeps `self._config`; new options `is_non_critical` (bool) and
    `reconnect_interval` (float, default 2.0).
  - A reactor timer `non_critical_recon_event` that periodically retries
    `recon_mcu()`.
  - `_handle_starting()` no longer async-shuts-down klippy for a non-critical
    MCU.
  - On comms timeout, `handle_non_critical_disconnect()` runs instead of
    `invoke_shutdown`: it marks the MCU disconnected, stops clocksync,
    disconnects the serial port and re-arms the retry timer.
  - `_mcu_identify()` probes the tty with `serialhdl.check_connect()` first
    and returns `False` (without raising) when the port is absent, so
    klippy can start with that MCU missing.
  - `recon_mcu()` re-identifies, resets OID/command/QUEUE state
    (`reset_to_initial_state()`) and reconnects.
  - `serialhdl.SerialReader.check_connect(serialport, baud, rts=True)` opens
    and closes a `pyserial` port to test presence; `connect()` breaks the
    retry loop early if already connected; `raw_send`/`raw_send_wait_ack`
    become no-ops when `serialqueue is None`.
- Why: on a K1 Max the `leveling_mcu` (GD32E230, /dev/ttyS9) and
  `nozzle_mcu` (/dev/ttyS1) can be absent or late at boot (USB/board power
  sequencing). Stock firmware tolerates this and Moonraker/Creality's host
  still starts; stock klippy would refuse to start or shut down the whole
  printer.
- Classification: K1-specific.
- Upstream equivalent: **none**, but upstream has refactored the entire
  connection lifecycle into `MCUConnectHelper` + `MCURestartHelper`, moved
  `_serial`/`_baud`/`_serialport`/shutdown tracking out of `MCU`, added
  `force_local_shutdown()`, `is_fileoutput()` handling and a
  `klippy:analyze_shutdown` hook. A textual merge of this patch is not
  possible.
- **Safety analysis** (material): tolerating a missing MCU is a deliberate
  weakening of an upstream invariant ("all configured MCUs must be present
  and healthy"). On a K1 Max the two affected MCUs are the nozzle board and
  the bed-leveling board. Consequences if one silently drops mid-print:
  heater control lives on the *main* MCU, so thermal protection is
  unaffected; but the extruder stepper and fans live on `nozzle_mcu`.
  A disconnected `nozzle_mcu` therefore means uncoordinated motion on the
  extruder. The fork does not add a "printer must stop" rule for that case
  beyond what klippy already does with a missing queue.
- In this repository **no config sets `is_non_critical`**, so as shipped the
  feature is inert — every MCU is critical. Retaining it is therefore a
  no-op behaviourally; a wrong implementation would only bite if someone
  opts in.
- Disposition: **adapt with a conservative reimplementation.** Port the
  feature onto `MCUConnectHelper`, but (a) keep it opt-in via
  `is_non_critical`, default false; (b) when a non-critical MCU disappears
  while the printer is *printing*, shut down rather than silently continue —
  a mid-print loss is not the boot-order case this exists for; (c) verify
  the reconnect path also re-registers endstops and probe objects.
  If (c) cannot be satisfied safely it will be left out and documented as
  dropped; see `docs/k1-max-upstream-update.md`.

### 4.8 `src/basecmd.c`, `src/linux/main.c` — `config_reset` guard

- Commit: `499de9ee`.
- What: comments out
  `if (! sched_is_shutdown()) shutdown("config_reset only available when
  shutdown");` in both the AVR/ARM-style `config_reset` and the Linux-MCU
  `command_config_reset`.
- Why: vendor MCU firmware on the K1 is flashed with `restart_method:
  command`. During `FIRMWARE_RESTART`, upstream first forces the MCU into a
  shutdown, *then* sends `config_reset`. The vendor firmware rejects that
  ordering, so the fork lets `config_reset` work in any state.
- Classification: K1-specific.
- Upstream equivalent: no; upstream relies on the guard to make
  `config_reset` safe (it clears `oid_count`, `config_crc` and the config
  while running).
- **Safety analysis**: this touches firmware-side state reset only. It does
  not disable thermal protection, endstop checks or `sched_is_shutdown`
  semantics anywhere else. However, upstream's `_restart_via_command()` at
  `ee2c6fb6` now calls `force_local_shutdown()` before sending
  `config_reset`, which means **upstream's own flow already produces the
  "shutdown first" ordering** the fork was compensating for.
- Disposition: **investigate → likely remove**. First confirm whether the
  vendored `.bin` firmware actually requires the relaxed guard; if it does,
  keep the relaxation with an explicit comment and document it as a known
  deviation. If upstream's `force_local_shutdown()` ordering satisfies the
  vendor firmware, **revert to upstream** and delete the fork commit's
  effect. Default to upstream unless evidence says otherwise — matching
  upstream here is the safer choice.

### 4.9 `klippy/extras/prtouch_v2.py` + the two `prtouch_v*_wrapper` `.so`s

- Commits: `4d4e161e`, `ca5b718a`, `5f9df017`, `62a0f667`, `e1288ae1`,
  `1dd56459`.
- What: `prtouch_v2.py` (26 lines) selects `prtouch_v1_wrapper` or
  `prtouch_v2_wrapper` by `pr_version`, instantiates `PRTouchEndstopWrapper`,
  registers it to the printer as `probe` via `probe.PrinterProbe`, and
  returns the wrapper (so `[prtouch_v2]` also becomes a lookup target used by
  `custom_macro.py`).
- Why: the K1 Max has no conventional Z endstop or BLTouch. Bed leveling and
  Z homing use four pressure-strain sensors on the `leveling_mcu` plus the
  nozzle as a contact probe. Creality ships the implementation as prebuilt
  CPython 3.8 MIPS32 (`mipsel`) Cython extension modules
  (`*.cpython-38-mipsel-linux-gnu.so`, 1.17 MB / 1.21 MB) — no source.
- Classification: K1-specific / packaging-integration.
- Upstream equivalent: none.
- **Verified facts about the binaries** (from the ELF dynamic symbol table
  and embedded strings):
  - They export exactly one module init each (`PyInit_prtouch_v2_wrapper`),
    plus 154 dynamic symbols total; they contain **no** chelper symbols, so
    they link against klippy through the Python API only, not ABI.
  - They reference these klippy attributes: `query_endstop`,
    `probe_prepare`, `probe_finish`, `lookup_object`, `toolhead`,
    **`move_queue`**, `get_last_move_time`, `set_position`, `wait_moves`,
    `get_position`, `multi_probe_begin`, `multi_probe_end`,
    `command_error`, `get_mcu`, `virtual_sdcard`, `heaters`, `bed_mesh`,
    `kinematics`, and the vendor entry points `run_G28_Z`, `run_G29_Z`,
    `run_to_next`, `change_hot_min_temp`.
  - They do **not** reference `run_probe`, `get_probe_params`, `get_offsets`,
    `mcu_probe`, `probe_session`, `home_rails`, `flush_step_generation`,
    `estimated_print_time`, `itersolve*` or `stepcompress*`.
  - Consequence: the objects are **CPython-3.8 mipsel only**. They cannot be
    imported by any other interpreter or architecture, which means the
    ported tree cannot be validated on this Windows/x86-64 machine beyond
    static analysis.
- Disposition: **retain with adaptation**, because deleting them removes the
  printer's only Z probe. Adaptation required:
  - `probe.PrinterProbe(config)` at upstream `ee2c6fb6` has **no** second
    argument; it constructs its own `ProbeEndstopWrapper`. Passing a vendor
    wrapper must keep working, so `prtouch_v2.py` has to build the probe
    object explicitly rather than rely on upstream's constructor signature.
  - Upstream now requires `probe.get_offsets()` / `get_probe_params()` and
    `start_probe_session()` on the `probe` object for `BED_MESH_CALIBRATE`,
    `PROBE`, `PROBE_CALIBRATE`, `PROBE_ACCURACY`, and for probe-based Z
    homing. The wrapper only exposes `probe_prepare`/`probe_finish`/
    `run_G29_Z`, so those code paths need a thin adapter that drives the
    wrapper and returns a `manual_probe.ProbeResult`.
  - The `.so` files are an unfixable black box: any behavioural defect inside
    them is out of reach.

### 4.10 `klippy/extras/probe.py`

- Commit: `8a71515d`.
- What: `ProbePointsHelper` gains the same `probe_type` detection; skips the
  `horizontal_move_z` lift for `prtouch_v2`, and uses
  `probe.mcu_probe.run_to_next(nextpos)` instead of `toolhead.manual_move`.
- Why: the wrapper performs the entire move-and-probe cycle for each mesh
  point (it owns the Z sweep, so an external Z lift would corrupt the sensor
  baseline).
- Classification: K1-specific.
- Upstream equivalent: no; upstream rewrote `ProbePointsHelper` around probe
  *sessions* (`start_probe_session`, `run_probe`, `pull_probed_results`,
  `end_probe_session`) and moved the lift into `_raise_tool()`.
- Disposition: **adapt**. Implement the prtouch path as a probe-session
  implementation instead of monkey-patching `_move_next`/`_raise_tool`, so
  `bed_mesh`, `screws_tilt_adjust`, `probe_accuracy` and friends all keep
  working through one adapter.

### 4.11 `klippy/extras/custom_macro.py` (144 lines)

- Commits: `4d4e161e`, `819d301a`.
- What: `[custom_macro]` section providing `CX_PRINT_LEVELING_CALIBRATION`,
  `CX_CLEAN_CALIBRATION_FLAGS`, `CX_PRINT_DRAW_ONE_LINE`, `CX_ROUGH_G28`,
  `CX_NOZZLE_CLEAR`, `SET_QMODE_FLAG` plus a `get_status()` with
  `leveling_calibration`, `default_extruder_temp`, `default_bed_temp`,
  `g28_ext_temp`, `qmode_flag`. These names are the API that Creality's
  closed-source `printer_data` scripts and the stock LCD/UI call.
- Why: host-side glue for the stock K1 Max printing workflow.
- Classification: packaging/integration.
- Upstream equivalent: none.
- **Defects found by inspection**:
  - `cmd_CX_PRINT_DRAW_ONE_LINE` busy-waits with `time.sleep(1)` inside a
    gcode command handler — this blocks klippy's single-threaded reactor,
    freezing all MCU communication, heater polling and webhooks for the whole
    heat-up. That is a real hazard: `verify_heater` runs in the reactor, so a
    long blocking sleep can itself trigger heater faults or mask a real one.
  - It reads `self.pheaters.can_break_flag`, an attribute that **does not
    exist** on `heaters` in this repository (neither fork nor upstream), so
    it raises `AttributeError` on first use.
  - `cmd_SET_QMODE_FLAG` reads `virtual_sdcard.speed_mode_path`, which also
    **does not exist**. It is wrapped in `try/except` + `logging.error`, so
    it silently does nothing.
  - `cmd_CX_ROUGH_G28` calls `self.prtouch.change_hot_min_temp(...)`; the
    `[prtouch_v2]` wrapper binary does contain that symbol name, so this
    path is plausible, but it is unverifiable here.
  - The module's file header is copy-pasted from `[temperature_sensor
    ds18b20]` ("Support for 1-wire based temperature sensors", "Copyright
    (C) 2020 Alan Lord") and is wrong.
- Disposition: **adapt**. Keep the command surface (external host software
  depends on the names), fix the reactor-blocking wait, and replace the two
  missing-attribute reads with either a small real implementation or an
  explicit no-op. See §5 for the approved stub policy.

### 4.12 `klippy/extras/fan_feedback.py` (116 lines)

- Commit: `4d4e161e`.
- What: `[fan_feedback]` sends a `config_fancheck oid=... fan_num=...` MCU
  command and registers handlers for the `fan_status` MCU response; exposes
  a webhook endpoint `get_cx_fan_status`.
- Why: stock K1 Max has tach-less fans; the vendor firmware implements
  `fancheck`/`fan_status` to detect a stalled fan.
- Classification: K1-specific / packaging-integration.
- Upstream equivalent: none, and **there is no `fancheck` implementation in
  `src/`** in this repository — no `config_fancheck`, no `fan_status`
  response, no GD32 port at all. The command exists only in Creality's
  prebuilt MCU firmware. So this module is functional *only* against the
  vendor `.bin` images.
- Safety: unchanged. It is purely observational; no fan control depends on
  it.
- Disposition: **retain as-is** (it is inert-but-harmless against upstream
  firmware: `config_fancheck` would fail at MCU config time, which *is* a
  startup failure — so it must be verified whether unknown commands are
  rejected). Because the K1 configs include `[fan_feedback]`, and an unknown
  command aborts MCU configuration, this needs an explicit guard: wrap
  registration so the section degrades to a no-op when the MCU does not
  advertise `fan_status`. That is a behaviour-preserving change.

### 4.13 `klippy/extras/bl24c16f.py` (301 lines)

- Commit: `4d4e161e`.
- What: I2C EEPROM (24C16) support on `[bl24c16f]` with `i2c_mcu: rpi`,
  `i2c_bus: i2c.2`, `i2c_speed: 400000`, plus `EEPROM_*` gcode commands.
- Why: the K1 Max stock host reads/writes factory calibration and print
  counters from an EEPROM on the host MCU's I2C bus.
- Classification: packaging/integration.
- Upstream equivalent: no dedicated module, but upstream's `bus.py` provides
  `MCU_I2C` and `setup_i2c_bus` with the same config keys — the module should
  be re-pointed at that API.
- Defect: contains an invalid escape sequence (`"\d"` in a regex string),
  which is a `SyntaxWarning` on Python 3.12+ and an error under `-W error`.
- Disposition: **retain, adapt API usage, fix the escape**. It is only
  active because the K1 configs load it, and it is required by the stock
  host software contract.

### 4.14 `klippy/extras/metadata.py` (1196 lines)

- Commit: `4d4e161e`.
- What: a slicer-metadata parser (classes for PrusaSlicer, Slic3r, Cura,
  Simplify3D, KISSlicer, IdeaMaker, IceSL, Orca...).
- Why: copied from Creality's tree.
- Classification: **unrelated / dead** — no `load_config`, not importable as
  a Klipper module, imported by nothing in `klippy/`, referenced by no
  config. Creality's own Moonraker-side scripts import it out-of-tree.
- Disposition: **remove**, and record the removal. If external host code
  really imports it out-of-tree, it should live outside the klippy package.
  (Removing dead code that cannot execute cannot break the printer; keeping
  it costs 1196 lines of review debt.)

### 4.15 `klippy/extras/tool.py` (31 lines)

- Commit: `4d4e161e`.
- What: writes `{"reqId","dn","code","data"}` JSON into
  `/usr/data/creality/userdata/config/pipe.json`, but only if
  `ping -c 2 -w 2 api.crealitycloud.com` succeeds.
- Why: Creality Cloud status reporting from the closed-source host scripts.
- Classification: packaging/integration, **and privacy-relevant**: it
  performs an outbound reachability probe to a vendor cloud endpoint from
  inside klippy, and drops a message offline when it fails.
- Upstream equivalent: none. Not imported by klippy; only the vendor scripts
  call `tool.reportInformation(...)`.
- Disposition: **retain but do not change behaviour**, and document it
  prominently: it silently performs a network probe. It is dead code unless
  vendor scripts import it. Recommendation for the user: delete it unless
  they use Creality Cloud. Left in place because removing it could break
  the stock host software contract; flagged in
  `docs/k1-max-validation.md`.

### 4.16 `config/K1_CR4CU220812S12/`, `config/K1C_CR4CU220812S12/`,
`config/K1_MAX_CR4CU220812S12/`

- Commits: `2e906724`, `245519f5`, `9a69e58a`.
- What: three complete vendor config bundles (5 files each:
  `printer.cfg`, `factory_printer.cfg`, `gcode_macro.cfg`,
  `printer_params.cfg`, `sensorless.cfg`), ~1600–1700 lines each.
- Why: the shipped printer configuration.
- Classification: K1-specific.
- **Verified upstream-incompatibilities** (see `docs/Config_Changes.md` in
  upstream):
  - `max_accel_to_decel:` in `[printer]` — removed upstream 2025-08-11.
    Present at `K1_MAX/printer.cfg:405`, `K1_MAX/factory_printer.cfg:330`,
    `K1/printer.cfg:316`, `K1/factory_printer.cfg:330`,
    `K1C/printer.cfg:347`, `K1C/factory_printer.cfg:414`. Current upstream
    **raises** on unknown options in `[printer]`, so klippy will refuse to
    start.
  - `SET_VELOCITY_LIMIT ACCEL_TO_DECEL=` — removed with the same change.
    Present in all three `gcode_macro.cfg` (K1_MAX:106,124,601;
    K1:106,124,601; K1C:112,150,457,696) and in `custom_macro.py:76`.
  - `printer.toolhead.max_accel_to_decel` — no longer a status field
    (K1_MAX/gcode_macro.cfg:93 and the K1/K1C equivalents).
  - `printer.print_stats.z_pos` / `.power_loss` — **never existed** in
    either fork or upstream (the vendor `print_stats.py` patch was reverted
    by `34478db4`). Used at `sensorless.cfg:26` and `:110` in all three
    bundles. Jinja2 raises `UndefinedError` on attribute access of a missing
    key, so `_IF_HOME_Z` and `_HOME_Z` are **broken in the fork as shipped**.
- Disposition: **adapt** — mechanical migration to `minimum_cruise_ratio`,
  drop `ACCEL_TO_DECEL`, and reimplement the two status hooks.

### 4.17 `fw/K1/*.bin` (7 vendor MCU firmware images)

- Commits: `1c59d01c`, `19309aaa`, `e349edd7`.
- What: 7 prebuilt GD32 firmware images (`bed0_*`, `mcu0_*`, `noz0_*`,
  ~31–43 KB each) used by Creality's `mcu_update` init.d script.
- Why: lets the stock host re-flash the three MCUs from the klipper tree.
- Classification: packaging/integration.
- Upstream equivalent: none; these are opaque binaries with no source in this
  repository (`src/` has no GD32 port and no `fancheck`).
- Disposition: **retain, untouched.** They are not executed by klippy; they
  are consumed by an external script. Explicitly **not** validated (we must
  not flash firmware). Documented as untrusted opaque blobs.

### 4.18 `.github/ISSUE_TEMPLATE/config.yml`

- Commits: `259f03c0`, `412e0f5a`.
- What: replaced upstream's contact links with a single empty line.
- Why: unrelated to the K1 (fork author tidying their own GitHub UI).
- Classification: **unrelated**.
- Disposition: **revert to upstream** on merge (no reason to keep a
  fork-local divergence in a file that has no effect on the printer).

### 4.19 Not modified, verified as clean

`klippy/extras/bed_mesh.py`, `klippy/extras/print_stats.py`,
`klippy/toolhead.py` (beyond the alias), `klippy/gcode.py`,
`klippy/kinematics/*`, `klippy/stepper_enable.py`, `klippy/extras/heaters.py`,
`klippy/extras/verify_heater` (moved upstream), `klippy/extras/fan.py`,
`klippy/clocksync.py` (beyond §4.2). No fork change weakens
`verify_heater`, `max_temp`, `min_extrude_temp`, `heater_bed` limits, or
endstop checking.

## 5. Dangling vendor hooks (pre-existing defects in the fork)

These are referenced by fork config but not provided by fork *or* upstream
code. They are broken today, independent of any upstream update:

| Hook | Referenced at | Status |
| --- | --- | --- |
| `printer.print_stats.z_pos` | all `sensorless.cfg:26` | absent (vendor patch reverted) |
| `printer.print_stats.power_loss` | all `sensorless.cfg:26` | absent |
| `printer.print_stats.z_pos` (second use) | all `sensorless.cfg:110` | absent |
| `printer.toolhead.max_accel_to_decel` | all `gcode_macro.cfg:93` | exists in fork, removed upstream |
| `virtual_sdcard.speed_mode_path` | `custom_macro.py:131` | absent (silently swallowed) |
| `heaters.can_break_flag` | `custom_macro.py:60,64,65,68` | absent (raises) |
| `kin.home_z_with_sensorless` | `homing.py` `STEPPER_Z_SENEORLESS` | absent (dead command) |
| `CHECK_BED_MESH` | `custom_macro.py:44` | absent (vendor bed_mesh patch reverted) |

Approved policy for these: reimplement **minimal, clearly marked** K1 Max
additions where the printer needs them, rather than resurrect large vendor
patches:

- `print_stats.z_pos` — report the toolhead's current Z (from
  `toolhead.get_position()[2]` / the gcode move's requested position) so
  `_IF_HOME_Z` / `_HOME_Z` can make their safe-Z decision.
- `print_stats.power_loss` — a flag the vendor host sets after a power-loss
  recovery. Minimal implementation: an initialised-to-0 status field updated
  by a small `SET_POWER_LOSS_STATE`-style path *only if* a caller exists;
  otherwise expose constant `0` and document that it is a stub.
- `virtual_sdcard.speed_mode_path` — expose the vendor path
  (`/usr/data/creality/userdata/config/...`-style) so `SET_QMODE_FLAG` can
  write its JSON, or make `SET_QMODE_FLAG` a documented no-op.
- `heaters.can_break_flag` — must **not** be a real mutable flag on the
  heater object (that would be a thermal-safety footgun). Reimplement
  `CX_PRINT_DRAW_ONE_LINE` in terms of a proper non-blocking wait
  (`reactor.register_timer` / `M109`-style) that observes the real
  `set_temperature` + `can_extrude` state, and drop the flag entirely.

## 6. Safety-relevant summary

Nothing in the fork disables thermal runaway protection, `max_temp`,
`min_extrude_temp`, `verify_heater`, endstop checking, or
`max_z_velocity`/`max_z_accel` limits. The items that *do* weaken upstream
invariants are, in order of concern:

1. **`is_non_critical` MCU tolerance** (§4.7) — lets klippy run with a
   configured MCU missing/stale. Inert in this repo (nobody sets it), but a
   real weakening if enabled. Port with a print-in-progress guard.
2. **`config_reset` guard relaxation** (§4.8) — allows a firmware state reset
   outside shutdown. Upstream's new `force_local_shutdown()` ordering may
   make it unnecessary; default to reverting.
3. **Blocking `time.sleep()` in a gcode handler** (§4.11) — starves the
   reactor, indirectly degrading `verify_heater`. Must be fixed.
4. **`MAX_ACCEL_TO_DECEL`/`ACCEL_TO_DECEL` removal** (§4.16) — a *startup*
   failure, not a safety failure, but it must be migrated or the printer
   will not start at all.

## 7. Critical uncertainties (unresolvable by inspection)

1. Whether the vendor MCU `.bin` firmware accepts the command ordering used
   by upstream `ee2c6fb6`'s `MCURestartHelper` (`force_local_shutdown()` then
   `config_reset`). Decides §4.8.
2. Whether the prtouch wrapper `.so` remains binary-compatible with the
   *host Python* actually deployed on the printer (CPython 3.8 mipsel). Any
   move to a newer Python on the printer breaks it irrecoverably.
3. Whether the vendor `nozzle_mcu`/`leveling_mcu` firmware implements any
   protocol command added by upstream between 2024-04 and 2026-10 that
   klippy now *requires* at config time. Upstream added MCU commands over
   that window (e.g. `motion_queuing`/`syncemitter` paths, `config_reset`
   usage, `trdispatch`). A missing required command fails at
   `finalize_config`, which looks like "klippy won't start" — needs the
   staged validation in `docs/k1-max-validation.md`.
4. Whether the K1 Max's four prtouch sensor pins (PA0–PA7 on
   `leveling_mcu`) and the `PC10`/`PB1` swap pins are still valid after any
   vendor firmware update — pin assignments were **not** changed by this
   port and are taken verbatim from the vendor config.
5. Whether `[fan_feedback]`'s `config_fancheck` is rejected as an unknown
   command by vendor firmware (it is implemented there, so presumably not,
   but it cannot be verified without the printer).

## 8. Predicted merge conflict set

Files where both the fork and upstream changed overlapping regions, based
on the 1166-commit upstream delta:

`klippy/chelper/__init__.py`, `klippy/clocksync.py`,
`klippy/extras/homing.py`, `klippy/extras/probe.py`,
`klippy/extras/temperature_mcu.py`, `klippy/mcu.py`,
`klippy/serialhdl.py`, `klippy/stepper.py`, `klippy/toolhead.py`,
`src/basecmd.c`, `src/linux/main.c`, `.github/ISSUE_TEMPLATE/config.yml`.

## 9. Disposition summary

| Area | Disposition |
| --- | --- |
| chelper build path | replace with upstream; drop prebuilt `.so` dependence |
| clocksync `disconnect()` + retry | adapt |
| `temperature_mcu` gd32f303xe | retain (adapted) |
| `stepper.get_pin_info()` | retain |
| toolhead `move_queue` alias | retain (with comment) |
| homing prtouch dispatch | adapt (items 1,5 removed as dead) |
| non-critical MCU reconnect | adapt, conservatively |
| `config_reset` guard | investigate → default revert |
| prtouch wrapper `.so` + `prtouch_v2.py` | retain, adapt probe adapter |
| probe.py prtouch path | adapt to probe sessions |
| `custom_macro.py` | adapt (keep command surface, fix reactor block) |
| `fan_feedback.py` | retain, add graceful degradation |
| `bl24c16f.py` | retain, adapt to `bus.py`, fix escape |
| `metadata.py` | remove (dead) |
| `tool.py` | retain, document network probe |
| `config/K1*` | adapt (`minimum_cruise_ratio`, status stubs) |
| `fw/K1/*.bin` | retain untouched (not validated) |
| `.github` config.yml | revert to upstream |
