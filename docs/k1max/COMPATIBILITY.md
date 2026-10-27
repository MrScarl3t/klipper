# K1 Max compatibility matrix

Scope: the three config bundles in this tree (`K1_MAX_CR4CU220812S12`,
`K1_CR4CU220812S12`, `K1C_CR4CU220812S12`) against host code pinned at
upstream `ee2c6fb6` as merged in `aa55619b`.

Status vocabulary: **OK** (verified compatible by inspection), **BROKEN**
(fails by inspection), **UNKNOWN** (needs hardware or evidence not available),
**N/A** (not present in this configuration).

"Verified" here means static inspection of this checkout. Nothing in this
document is hardware-validated.

## Host software and build

| Component | Current evidence | Target requirement | Planned action | Validation | Status |
| --- | --- | --- | --- | --- | --- |
| `klippy/chelper/__init__.py` build path | Identical to upstream `ee2c6fb6` — auto-build branch restored | Upstream build path | none | `--import-test` in Linux venv | OK (restored) |
| Prebuilt `klippy/chelper/c_helper.so` | Tracked in git despite `*.so` in `.gitignore` (tracked before ignore rule applied); MIPS32 LE, 63532 B, exports the 2024 chelper ABI | Must not be loaded | Ignored by code now; file still in tree | none | OK (inert), cleanup deferred |
| `klippy/extras/prtouch_{v1,v2}_wrapper...so` | CPython 3.8 / EM_MIPS, 1.2 MB and 1.1 MB | Vendor host Python 3.8 mipsel | Retain untouched | not loadable on x86 dev host | UNKNOWN (host-Python bound) |
| Host Python version | Vendor image is CPython 3.8 mipsel per wrapper filename | `klippy-requirements.txt` supports 3.12+ too | Document constraint | — | UNKNOWN (printer not inspected) |

## Vendor host modules

| Module | Evidence | Status |
| --- | --- | --- |
| `klippy/extras/prtouch_v2.py` | `probe.PrinterProbe(config, vrt)` at line 16; current signature is `PrinterProbe(config)`. Wrapper has no `start_probe_session`, which `PrinterProbe` now requires | **BROKEN** |
| `klippy/extras/custom_macro.py` | Blocking `time.sleep(1)` loop at line 66; depends on `heaters.can_break_flag`, absent from this tree before *and* after the merge; calls `CHECK_BED_MESH`, undefined in `bed_mesh.py` | **BROKEN** |
| `klippy/extras/fan_feedback.py` | Issues vendor MCU commands `config_fancheck` / `query_fancheck` / `fan_status`, absent from upstream `src/`; requires vendor `nozzle_mcu` firmware | UNKNOWN (vendor-firmware dependent) |
| `klippy/extras/bl24c16f.py` | Uses `bus.MCU_I2C_from_config`, present; `load_config` + `load_config_prefix` present | OK by inspection |
| `klippy/extras/metadata.py` | Standalone argparse script, not a klippy module; unreferenced by any config | **dead** |
| `klippy/extras/tool.py` | Unreferenced; `SyntaxWarning: invalid escape sequence '\d'` at line 14 | **dead** |
| `klippy/stepper.py` `get_pin_info()` | 2-line additive accessor present; `_dir_pin`/`_step_pin` still exist upstream | OK |
| `klippy/toolhead.py` `move_queue` alias | One-line alias at line 208, commented | OK |
| `klippy/extras/temperature_mcu.py` `gd32f303xe` | Registered in `cfg_funcs`; uses `self.calc_base`, present | OK (die-temp reporting only) |
| `klippy/extras/homing.py` prtouch dispatch | Patches `cmd_G28` and `probing_move`; `probe_type` is set in `PrinterHoming.__init__`. `has_section('prtouch_v2')` works | Partially OK — correctness unknown until the probe adapter loads (**BROKEN** dependency) |
| `klippy/extras/probe.py` prtouch path | Guards `_raise_tool`/`_move_next` on `self.probe_type`; calls `mcu_probe.run_to_next`, `run_G29_Z`, `run_G28_Z`, all present in the wrapper | Partially OK — same blocker |
| non-critical MCU reconnect | **Not in the merge result.** Pre-merge had `is_non_critical`, `reconnect_interval`, `check_connect`, `clocksync.disconnect()`; the merge took upstream for `mcu.py`, `serialhdl.py`, `clocksync.py` | Dropped (recorded in DECISIONS D2) |
| `config_reset` guard relaxation | Not present after the merge; upstream `force_local_shutdown()` path is used | Dropped, consistent with audit "default revert" |

## Configuration

| Item | Evidence | Status |
| --- | --- | --- |
| `max_accel_to_decel` | 6 sites (3 bundles × `printer.cfg` and `factory_printer.cfg`). Removed upstream 20251108 → `Config_Changes.md:121` | **BROKEN** (klippy aborts on unused option) |
| `square_corner_max_velocity` | 2 sites (`K1_MAX`, `K1C`). Never existed upstream and was never read by any code in this repo's history | **BROKEN** (dead vendor option) |
| `SET_VELOCITY_LIMIT ACCEL_TO_DECEL=…` | 9 macro sites across the three bundles | **BROKEN** at run time (command arg removed) |
| `printer.toolhead.max_accel_to_decel` | Template reads in each bundle's `gcode_macro.cfg` | **BROKEN** at template evaluation |
| `printer.print_stats.z_pos` / `power_loss` | 6 references across bundles' `sensorless.cfg` | **BROKEN** at template evaluation; the vendor print_stats patch was reverted by the fork author |
| `[prtouch_v2]` pin map | `pres0..3` on `leveling_mcu` PA0–PA7, `step_swap_pin: PC10`, `pres_swap_pin: leveling_mcu:PB1` | UNKNOWN — vendor config copied verbatim; not validated |
| Kinematics / motion limits | `corexy`, `max_velocity 800`, `max_accel 20000`, `max_z_velocity 20`, `max_z_accel 300` | OK by inspection |
| Thermal protection | `verify_heater extruder` + `verify_heater heater_bed` present with `check_gain_time 120`, `heating_gain 1.0`, `hysteresis 10`; `EPCOS 100K B57560G104F` thermistors | OK by inspection; calibration unverified |
| Sensorless homing | `tmc2209` `diag_pin` + `virtual_endstop` on X/Y/Z, plus a 60-line `homing_override` | UNKNOWN (hardware) |
| Acceleration measurement | `[adxl345]` soft-SPI on `nozzle_mcu` PA4–PA7 | UNKNOWN; shares pins with prtouch pres1 (PA3/PA4) at rest |
| Filament sensors | Two `filament_switch_sensor` sections (`PC15`, `nozzle_mcu:PA10`) | OK by inspection |
| Display | `[display_status]` only; no `[display]` → the stock K1 screen is driven outside klippy | N/A for klippy |
| Moonraker / UI | No Moonraker config in tree; `fan_feedback` registers a `get_cx_fan_status` webhook endpoint | UNKNOWN |
| Camera / LiDAR | No configuration present | N/A |
| Automated config coverage | `test/klippy/printers.test` contains 0 K1 entries | **missing** |

## MCU firmware

| MCU | Board | Upstream `src/` support | Status |
| --- | --- | --- | --- |
| main (`/dev/ttyS7`) | GD32F303RET6, `CR4CU220812S12` | **none** — no `MACH_STM32F303` in `src/stm32/Kconfig` | UNKNOWN — vendor binary `fw/K1/mcu0_*.bin` |
| `nozzle_mcu` (`/dev/ttyS1`) | GD32F303CBT6 | **none** | UNKNOWN — vendor binary `fw/K1/noz0_*.bin` |
| `leveling_mcu` (`/dev/ttyS9`) | GD32E230F8P6 | **none** | UNKNOWN — vendor binary `fw/K1/bed0_*.bin` |
| `rpi` (`/tmp/klipper_host_mcu`) | host process | `src/linux` present | UNKNOWN — host process not built or deployed |

Consequence: this tree can supply the **host** half of a K1 Max deployment but
cannot produce replacement firmware for any of the three printer MCUs. The
vendor binaries in `fw/K1/` were not inspected; their protocol version is
undetermined.

## Legend of evidence sources

- Upstream behaviour: `git show upstream/master:<path>`, `docs/Config_Changes.md`.
- Fork behaviour: working tree at `aa55619b`, plus `git show 3423dc08:<path>`
  for pre-merge comparisons.
- Wrapper symbol surface: byte-level `grep -a` on the `.so` files (no
  `strings` on this host). This is a coarse check, not a symbol-table dump.
