# K1 Max native Klipper port

## Project
- Root: C:\Users\Thijs\gitproj\klipper
- Goal: upstream/native Klipper on the K1 Max's original hardware.
- Windows development host; verify the shell and Linux/WSL environment.
- Verify the checkout, hardware revision, and vendor/upstream compatibility.
- Never guess pins, MCU targets, serial paths, or electrical settings.

## Work efficiently
- Work only on the requested task; avoid unrelated refactors.
- Search relevant paths/symbols first, then read necessary file ranges.
- Exclude generated files, binaries, dependencies, and large logs from
  initial searches; inspect them only when relevant.
- Reuse findings already in context; reread files when changed or uncertain.
- Narrow searches when output is truncated; expand when evidence requires it.
- Keep tool output focused, preserving failures and exit status.
- Prefer targeted diffs over printing complete files.
- Use subagents only for substantial, separable work.
- Do not produce a long plan for a straightforward edit.
- Ask only when uncertainty blocks correctness or hardware safety.

## Repository
- klippy/: host Python; klippy/extras/: integrations.
- src/: MCU firmware; config/: printer configurations.
- scripts/: tooling; test/klippy/: regression tests; docs/: reference.
- Read docs/k1-max-port.md only when hardware or porting details are needed.

## Changes and checks
- Check git status --short before editing; preserve existing changes.
- Make the smallest correct change using existing project patterns.
- Use this checkout's documented checks; do not invent test commands.
- Run targeted checks first; broaden for shared-code or safety-critical changes.
- Run git diff --check after edits.
- Distinguish tests passed, failed, and not run; state blockers.
- Do not commit, push, reset, clean, or discard work without permission.

## Hardware safety
- Offline edits/tests only unless live actions are explicitly approved.
- Ask before deployment, flashing, service restarts, G-code, motion, or heat.
- Verify exact hardware and establish backup/recovery before deployment.
- Preserve thermal, motion, probing, and shutdown protections.

## Responses
- Default: brief result, changed files, checks, remaining risks.
- Do not repeat the request, dump logs, or reproduce unchanged code.
- Explain more when requested or necessary for correctness/safety.

## Compaction
Preserve the goal, changed files, verified hardware facts, compatibility
findings, test results, blockers, and next steps. Drop repetitive logs.