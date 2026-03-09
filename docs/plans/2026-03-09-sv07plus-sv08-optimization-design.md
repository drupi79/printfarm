# SV07+ & SV08 Config Optimization Design

**Date:** 2026-03-09
**Scope:** Full optimization of Klipper configs and OrcaSlicer profiles for two external printers (not part of the printfarm). Output goes to `Printer_Backup/Sv07+_optimized/` and `Printer_Backup/Sv08_optimized/`.

---

## Goals

- Fix bugs and deprecated config in both printer.cfg files
- Restructure macros to follow printfarm pattern (PRINT_START takes EXTRUDER_TEMP/BED_TEMP params, does all work)
- Create four OrcaSlicer process profiles per printer matching printfarm style: quality, balanced, speed, print_in_place
- Set per-profile accel limits in OrcaSlicer based on input shaper data; leave Klipper max_accel high

---

## Output Structure

```
Printer_Backup/
  Sv07+_optimized/
    printer.cfg
    printer_profile.json
    quality.json
    balanced.json
    speed.json
    print_in_place.json
  Sv08_optimized/
    printer.cfg
    printer_profile.json
    quality.json
    balanced.json
    speed.json
    print_in_place.json
```

---

## SV07+ Hardware Summary

| Property | Value |
|----------|-------|
| Kinematics | Cartesian |
| Board | STM32F103, 28KiB bootloader, USART1 |
| Bed | 300×300×350mm |
| Z | Dual steppers (Z_TILT_ADJUST) |
| Homing | Sensorless TMC2209 virtual endstop (X/Y) |
| Probe | Inductive, x_offset: +27, y_offset: -20 |
| Input shaper | MZV @ 40.8Hz (X), MZV @ 32.0Hz (Y) — SAVE_CONFIG |
| Pressure advance | 0.02 |
| Extruder rotation_distance | 4.59 |

---

## SV08 Hardware Summary

| Property | Value |
|----------|-------|
| Kinematics | CoreXY |
| Board | Dual MCU: main STM32F103 + extra_mcu STM32F103 (toolhead) |
| Bed | 350×350×345mm |
| Z | Quad gantry level (4 steppers, 80:12 gear ratio) |
| Homing | Sensorless X+/Y+ via custom homing_override |
| Probe | Inductive on extra_mcu (x_offset: -17, y_offset: +10) + probe_pressure for Z offset cal |
| Input shaper | 3hump_ei @ 64.2Hz (X), ZV @ 38.8Hz (Y) — SAVE_CONFIG |
| Pressure advance | 0.025 |
| Extruder rotation_distance | 6.5 |

---

## Klipper Config Changes

### SV07+

**Bug fixes / deprecations:**
- Remove `max_accel_to_decel: 5000` (deprecated) → add `minimum_cruise_ratio: 0.5`
- Remove dead `M205` macro (calls `M105` — does nothing useful)
- Widen `resonance_tester` range: `min_freq: 5`, `max_freq: 133` (current 1–100 is fine but standardize)

**Macro improvements:**
- `LOAD_FILAMENT` / `UNLOAD_FILAMENT`: add `TEMP=220` parameter, heat nozzle before extruding
- Replace empty `PRINT_START` with full macro: `PRINT_START EXTRUDER_TEMP= BED_TEMP=`
  - Heat bed to BED_TEMP
  - `G28` (home all)
  - `Z_TILT_ADJUST`
  - `BED_MESH_CALIBRATE` (loads default mesh — no KAMP on this printer)
  - Park at front
  - Heat extruder to EXTRUDER_TEMP
  - Short purge line
- Replace minimal `PRINT_END` with proper end macro: retract, raise Z, park, turn off heaters/fan, disable motors
- Update `CANCEL_PRINT` to call `PRINT_END` logic cleanly

**Quality of life:**
- Add `[gcode_arcs] resolution: 0.1`
- Add `[exclude_object]`

**Preserve untouched:**
- All SAVE_CONFIG values (PIDs, z_offset, bed_mesh, input_shaper)
- All stepper/driver pin assignments and run_current values
- Probe offsets, safe_z_home, z_tilt, bed_mesh, screws_tilt_adjust coords

### SV08

**Bug fixes:**
- Fix duplicate `speed:` lines in `[probe]`: remove dead `speed: 15.0`, keep `speed: 5.0`
- Fix `damping_ratio_x: 0.001` / `damping_ratio_y: 0.001` → `0.1` (Klipper default; 0.001 is non-physical)
- Fix `gcode_arcs resolution: 1.0` → `0.1`
- Widen `resonance_tester`: `min_freq: 5`, `max_freq: 133` (current 20–40 misses the 64.2Hz X shaper)

**Macro improvements:**
- Add full `PRINT_START EXTRUDER_TEMP= BED_TEMP=` macro:
  - Heat bed to BED_TEMP
  - `G28` (triggers homing_override → sensorless Y then X, then Z via probe)
  - `QUAD_GANTRY_LEVEL`
  - `G28 Z` (re-home Z after QGL)
  - `BED_MESH_CALIBRATE`
  - Park at front
  - Heat extruder to EXTRUDER_TEMP
  - Short purge line (relative move purge, same style as printfarm)
- Add `PRINT_END`: retract, raise Z, park, turn off heaters/fans, disable motors
- `LOAD_FILAMENT` / `UNLOAD_FILAMENT`: add `TEMP=220` parameter, heat before extruding

**Update OrcaSlicer start gcode (in printer_profile.json):**
- `machine_start_gcode: "PRINT_START EXTRUDER_TEMP=[nozzle_temperature_initial_layer] BED_TEMP=[bed_temperature_initial_layer_single]"`
- `machine_end_gcode: "PRINT_END"`
- Remove the current massive inline gcode block

**Preserve untouched:**
- All SAVE_CONFIG values (probe z_offset, bed_mesh, input_shaper)
- homing_override logic (sensorless sequence is correct)
- All stepper/driver pin assignments, run_current, QGL coords
- probe and probe_pressure configs, z_offset_calibration section
- Custom thermistor definitions

---

## OrcaSlicer Process Profiles

Profiles follow printfarm conventions:
- `inherits`: references the printer's own base profile name
- `wall_sequence: outer wall/inner wall`
- `only_one_wall_top: 1`
- `reduce_crossing_wall: 1`
- `bridge_flow: 0.95`
- `elefant_foot_compensation: 0.1`
- Organic tree support on quality profiles

### Accel Targets

Input shaper data → recommended max accel:
- **SV07+**: MZV @ 32.0Hz (Y, limiting axis) → conservative cap ~3,400 mm/s²
- **SV08**: ZV @ 38.8Hz (Y, limiting axis) → conservative cap ~8,000 mm/s²

| Profile | SV07+ default_accel | SV07+ outer_wall_accel | SV08 default_accel | SV08 outer_wall_accel |
|---------|--------------------|-----------------------|--------------------|-----------------------|
| Quality | 1500 | 1200 | 3000 | 2500 |
| Balanced | 3000 | 2500 | 6000 | 5000 |
| Speed | 3400 | 3000 | 8000 | 7000 |
| Print-in-place | 800 | 600 | 1500 | 1000 |

### Per-Profile Settings

**Quality:**
- 3 walls, 5 top/bottom layers
- `outer_wall_speed: 40`, `inner_wall_speed: 60`, `sparse_infill_speed: 100`
- `support_type: tree(auto)`, `support_style: organic`
- `sparse_infill_pattern: honeycomb`

**Balanced:**
- 3 walls, standard layer counts
- Mid-range speeds, standard infill

**Speed:**
- 2 walls, 4 top/bottom layers
- Wider line widths (0.45–0.5mm)
- Higher jerk values

**Print-in-place:**
- 4 walls, `precise_outer_wall: 1`
- Seam slopes enabled
- `brim_type: outer_only`, `brim_width: 3`
- Very conservative speeds (outer 30mm/s, inner 40mm/s)
- `wall_sequence: inner wall/outer wall` (exception — reduces seam artifacts)
- `default_acceleration: 800` (SV07+) / `1500` (SV08)

---

## What Is NOT Changed

- Calibrated values: PIDs, z_offset, input shaper frequencies, pressure advance
- Pin assignments and hardware topology
- Sensorless homing thresholds (driver_sgthrs)
- Z_TILT / QGL coordinates
- Bed mesh boundaries
- Filament runout sensor config
- PLR (power loss recovery) include on SV07+
