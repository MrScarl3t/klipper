# K1 Max modernization — validation log

Every entry records the command, the environment, and the actual outcome.
Nothing here is hardware-validated unless it says so explicitly.

## Environments

| ID | Environment | Notes |
| --- | --- | --- |
| E1 | Windows 11, Git Bash, Python 3.12.10 (system) | Development host. No `termios`, no `cffi`, no `greenlet`. Cannot run klippy. |
| E2 | WSL2 Ubuntu 26.04.1 LTS, `/usr/bin/python3.12` (3.12), `git` 2.53 | No `gcc`, `make`, `cffi`, `greenlet`, `python2`. venv module available. |
| E3 | Printer (K1 Max host, CPython 3.8 mipsel) | **Not accessed.** No evidence collected. |

## Checks run

### V1 — Fork-modified modules compile

```
python -c "compile(open(f).read(), f, 'exec')" for each fork-modified module
```

Environment E1. Result: **pass**. One pre-existing `SyntaxWarning`
(`invalid escape sequence '\d'`) in `klippy/extras/tool.py:14`, which predates
this work. No other module failed.

Modules covered: `homing.py`, `probe.py`, `temperature_mcu.py`,
`custom_macro.py`, `fan_feedback.py`, `bl24c16f.py`, `metadata.py`,
`prtouch_v2.py`, `tool.py`, `stepper.py`, `toolhead.py`.

### V2 — klippy import test on Windows

```
python klippy/klippy.py --import-test
```

Environment E1. Result: **fails, environment limitation**.
`ModuleNotFoundError: No module named 'termios'`. Windows has no `pty`/`tty`.
This is not a code defect and says nothing about the fork.

### V3 — Merge composition check

```
git diff --name-only ee2c6fb6 aa55619b | wc -l
git diff --name-status ee2c6fb6 aa55619b | grep -v '^A'
```

Result: 38 paths differ from upstream; 33 are fork additions and 5 are
modifications to upstream files (`klippy/extras/homing.py`,
`klippy/extras/probe.py`, `klippy/extras/temperature_mcu.py`,
`klippy/stepper.py`, `klippy/toolhead.py`). `klippy/mcu.py`,
`klippy/serialhdl.py`, and `klippy/clocksync.py` are byte-identical to
upstream, i.e. the fork's non-critical-MCU hunks were dropped. See
`DECISIONS.md` D2.

### V4 — Config option support check

Static grep against `klippy/`, `docs/Config_Reference.md`,
`docs/Config_Changes.md`:

| Option / expression | Sites | Upstream support | Verdict |
| --- | --- | --- | --- |
| `max_accel_to_decel` | 6 | removed 20251108 (`Config_Changes.md:121`) | breaks config load |
| `square_corner_max_velocity` | 2 | never existed; never read by code | breaks config load |
| `SET_VELOCITY_LIMIT ACCEL_TO_DECEL` | 9 | removed | breaks at run time |
| `printer.toolhead.max_accel_to_decel` | 4 | status key removed 20240313 | breaks template |
| `printer.print_stats.z_pos` | 3 | never existed (vendor patch reverted) | breaks template |
| `printer.print_stats.power_loss` | 3 | never existed | breaks template |

`klippy/configfile.py:424 check_unused()` raises `error` on both unread
sections and unread options, so the first two rows abort klippy before any
printer object is created.

### V5 — Vendor module API check

| Call site | Against | Result |
| --- | --- | --- |
| `prtouch_v2.py:16` `PrinterProbe(config, vrt)` | `probe.py:621 PrinterProbe.__init__(self, config)` | **TypeError** — signature mismatch |
| `probe.py` probe-session path | wrapper `.so` symbol surface | `start_probe_session` **absent** from both wrappers |
| `homing.py:333/364` `mcu_probe.run_G29_Z` / `run_G28_Z` | v2 wrapper | symbols present |
| `probe.py:486` `mcu_probe.run_to_next` | v2 wrapper | symbol present |
| `custom_macro.py:65` `heaters.can_break_flag` | `klippy/extras/heaters.py` | **absent** (also absent pre-merge) |
| `custom_macro.py:44` `CHECK_BED_MESH` | `klippy/extras/bed_mesh.py` | **absent**, command never registered |
| `fan_feedback.py` `config_fancheck`/`query_fancheck`/`fan_status` | `src/` | **absent** from upstream MCU source; vendor firmware only |
| `bl24c16f.py` `bus.MCU_I2C_from_config` | `klippy/extras/bus.py:302` | present |

Symbol presence was checked with byte-level `grep -a` because `strings` is not
available in E1/E2. This is a coarse test: it can produce false positives if a
name appears in a string table without a matching attribute.

### V6 — MCU target availability

```
grep -oP 'config MACH_STM32\w+' src/stm32/Kconfig
```

Result: no `MACH_STM32F303`. No `GD32` or `GD32E230` anywhere in `src/`.
The three K1 MCUs therefore cannot be built from this tree. `fw/K1/*.bin`
are vendor binaries and were **not** inspected or validated.

### V7 — Whitespace / style

Not yet run. `./scripts/check_whitespace.sh` requires a POSIX shell; it was not
executed in this checkpoint because no source file was modified. It must be run
before the first code slice.

## Not run — explicit list

| Check | Blocker |
| --- | --- |
| `scripts/test_klippy.py` regression suite | needs host simulator dictionaries from `scripts/ci-build.sh`, which needs `make` + host gcc |
| MCU compile matrix (42 targets) | needs `build-essential` and the ARM cross-toolchain |
| K1 config parse (no test entry exists) | needs a test entry and a dictionary; also blocked on the vendor modules |
| `check_whitespace.sh` | not yet executed (see V7) |
| Any printer-side check | no printer access; hardware gate not opened |

## Documentation gaps found

| Item | Detail |
| --- | --- |
| `CLAUDE.md` → `docs/k1-max-port.md` | file does not exist |
| `docs/k1-max-fork-audit.md:496,629` → `docs/k1-max-validation.md` | file does not exist |
| `docs/k1-max-fork-audit.md` §9 | dispositions labelled "adapt" for `mcu.py`/`serialhdl.py`/`clocksync.py` do not match the merged tree; read as intent, per `DECISIONS.md` D1/D2 |

No instruction file belonging to the user was edited to fix these pointers.
