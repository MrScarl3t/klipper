# K1 Max — deployment prerequisites and recovery

**Status: UNVERIFIED.** Nothing in this file has been executed against a
printer. It records what must be established *before* a deployment attempt, so
that the gaps are visible. It is not an installation guide.

## Why this is unverified

A Klipper recovery procedure depends on details that cannot be read from this
repository:

- How Klipper was originally installed on this specific printer (vendor image,
  community installer script, or manual). The recovery path differs completely
  between them.
- Whether the host filesystem is writable and persistent, and whether SSH is
  enabled.
- How the three MCU firmware images are normally flashed on this unit
  (vendor `mcu_update` init.d script via `fw/`, SD card, DFU, or not at all).
- Whether the stock display/UI services are part of the same init system and
  must be stopped or preserved.

`fw/K1/*.bin` are vendor binaries of unknown provenance and version. They are
in the tree unmodified and have not been inspected, hashed against a vendor
source, or tested.

## Prerequisites that must be satisfied first

1. **Identify the installed firmware.** Record the exact Creality firmware
   version string and any prior Klipper modification. Do not infer it from the
   configs.
2. **Record board revisions.** Mainboard, toolhead (`nozzle_mcu`) and leveling
   (`leveling_mcu`) board revision strings as printed by the printer.
3. **Take a full backup** of the existing configuration, the host Klipper
   tree, any `printer_data` directory, and the currently installed MCU
   firmware images — before any change.
4. **Prove the recovery path works** while the printer is still in its known
   state. Until that is demonstrated, there is no rollback.
5. **Decide the host Python version.** The vendor prtouch wrappers are built
   for CPython 3.8 mipsel; a host Python change breaks them permanently.
   Confirm the deployed interpreter before shipping any host change.

## What this tree cannot currently provide

- Replacement firmware for any K1 MCU (no GD32 targets in `src/`, see
  `VALIDATION.md` V6).
- A working `[prtouch_v2]` probe integration (see `COMPATIBILITY.md`; the
  probe adapter does not match the current probe API).
- A K1 config that loads without an unused-option error.

That is: even if recovery were proven, this tree is not deployable today.

## Recovery outline — skeleton only, not validated

The following is the shape a procedure must take, with every placeholder
resolved against real hardware evidence before use:

1. Restore the backed-up host Klipper tree and configuration.
2. Restore the previously installed MCU firmware images by whichever mechanism
   step 4 above demonstrated to work.
3. Restart the printer's services and confirm the stock stack returns to its
   pre-change state.
4. Verify by printing nothing but running the stock startup checks; do not
   assume "the file was copied back" means recovery succeeded.

## Abort criteria for any future deployment attempt

Stop and roll back if any of the following occur:

- klippy does not reach "ready" with the stock load sequence.
- Any MCU reports a shutdown, a protocol mismatch, or a missing required
  command at `finalize_config`.
- `verify_heater`, temperature limits, or endstop behaviour differ from the
  known-good baseline.
- Any unexpected motion or heater activity.

Never disable heater verification, temperature limits, or shutdown handling to
get past a failure.
