# Sovol SV07 Plus — live configuration snapshot (2026-10-31)

Pulled from the printer at `192.168.1.200` (Moonraker `:7125`, SSH `mks` / `makerbase`).
Klipper `v0.11.0-122-ge6ef48cd`, Moonraker Sovol fork `v0.8.0-23-g18f5ff4-dirty`.

This is **not** the same setup as `../Sv07+_optimized/`, the config published to
`drupi79/sovol-sv07plus-optimized`. This printer runs the **stock Sovol `printer.cfg`
plus the printfarm shared macros**:

```
[include plr.cfg]            # Sovol power-loss recovery
[include KAMP_Settings.cfg]  # KAMP: Adaptive_Meshing, Line_Purge, Smart_Park
[include start.cfg]          # = klipper/shared/start.cfg (identical, not duplicated here)
[include macros.cfg]         # shared macros.cfg + appended Sovol section (see below)
```

Don't push `Sv07+_optimized/printer.cfg` or its Orca profile to this printer. Its
`PRINT_START` and the plain `PRINT_START` start gcode skip the Z tilt and power-loss
handling this setup needs.

## Changes made on the printer

| File | Change | Backup on printer |
|------|--------|-------------------|
| `printer.cfg` | Removed Sovol's stub `PRINT_START`/`PRINT_END` (they came after `[include start.cfg]` and overrode it, so prints started cold: *"Extrude below minimum temp"*) | `printer.cfg.bak-20261031-pre-startcfg` |
| `printer.cfg` | Added `[firmware_retraction]` 0.5 mm @ 40 mm/s (KAMP `LINE_PURGE` and Orca use G10/G11) | `printer.cfg.bak-20261031-pre-fwretract` |
| `macros.cfg` | Commented out the duplicate Sovol `CANCEL_PRINT`, `START_PRINT`, `END_PRINT`, `LOAD_FILAMENT`, `UNLOAD_FILAMENT`, `M205`. Kept Sovol `PAUSE`/`RESUME` (`RESUME` checks the runout sensor) and `LED_ON`/`LED_OFF`. Added `G31` + `clear_last_file` to the shared `CANCEL_PRINT` so a cancel clears power-loss state | `macros.cfg.bak-20261031-pre-dedupe` |
| `~/moonraker/.../file_manager/metadata.py` | See `moonraker_metadata_orca.patch`: added the `OrcaSlicer` slicer alias (this build predates Orca support) and fixed a Sovol debug line that crashed on every non-Sovol file. Fixes missing thumbnails and print-time estimates | `metadata.py.bak-20261031` |

`moonraker.conf` already had `[file_manager] enable_object_processing: True`, which KAMP needs.

**Moonraker updates:** an update through the update manager will either refuse
(the repo is dirty) or wipe the `metadata.py` patch. If thumbnails or times disappear
after an update, copy `moonraker_metadata_orca.patch` to the printer's home folder and run
`cd ~/moonraker && patch -p0 moonraker/components/file_manager/metadata.py < ~/moonraker_metadata_orca.patch`.

## Print start sequence

The OrcaSlicer preset's start gcode handles the Sovol-specific steps, then hands off to
the shared `PRINT_START`:

```
SAVE_VARIABLE VARIABLE=was_interrupted VALUE=True   ; Sovol PLR flag
G28
Z_TILT_ADJUST                                       ; dual Z — shared start.cfg doesn't do this
G28 Z
PRINT_START EXTRUDER_TEMP=... BED_TEMP=...          ; start.cfg: heat, adaptive mesh, Smart_Park, LINE_PURGE
```

End gcode: `PRINT_END`, then clear the PLR state (`was_interrupted=False`, `clear_plr`,
`clear_last_file`).

`BED_MESH_CALIBRATE` comes from `start.cfg`, not KAMP's `Adaptive_Meshing.cfg`
(start.cfg is included later), so `variable_mesh_margin` in `KAMP_Settings.cfg` has no
effect. The purge and Smart Park settings still apply.

## OrcaSlicer presets (`orca/`)

Copied from the Orca Cloud user folder. Orca stores user presets as **differences from
the parent** only, so settings equal to the Sovol system profile don't appear. Parents:

- Printer `Sovol SV07 Plus 0.4 nozzle Optimized` inherits `Sovol SV07 Plus 0.4 nozzle`.
  It has `use_firmware_retraction: 1`, the start/end gcode above, and accel limits of 12000.
- Process presets inherit `0.20mm Standard @Sovol SV07 Plus`. All have
  `exclude_object: 1` (Label Objects). The stock Sovol process has it **off**, which
  stops KAMP from adapting.

## Known quirks

- `PREHEAT_PLA` / `PETG` / `ABS` (shared `macros.cfg`) run a 10-minute `G4` heat soak
  that blocks all gcode. KlipperScreen and new prints look stuck until it finishes.
- `printer.cfg` still uses the deprecated `max_accel_to_decel` (accepted on Klipper v0.11).
