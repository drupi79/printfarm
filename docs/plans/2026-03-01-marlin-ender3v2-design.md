# Marlin Ender 3 V2 — Farm Addition Design

**Date:** 2026-03-01
**Printer:** Ender 3 V2 running MRisCoC Professional Firmware (Marlin 2.x)
**Repo name:** `ender3_v2_marlin`

## Context

The farm is otherwise all-Klipper. This printer breaks that pattern — it runs MRisCoC
Professional Firmware (a Marlin 2.x distribution) on the stock Creality 4.2.2 board with
no custom firmware configuration. Hardware is identical to the V2 Klipper Twins (BMG DD,
TZ E3 2.0, BLTouch).

## Approach: Option A — OrcaSlicer profiles + hardware notes doc

Since the firmware is a stock (pre-built) MRisCoC build, there are no custom
`Configuration.h` files to track. Hardware configuration is captured in a markdown doc
instead. OrcaSlicer profiles are the primary artifact.

## Repo Structure

```
orca_profiles/
  ender3_v2_marlin/
    printer_profile.json
    quality.json
    balanced.json
    speed.json
    print_in_place.json

marlin/                         ← new top-level dir, parallel to klipper/
  ender3_v2_marlin/
    HARDWARE_NOTES.md
```

The `marlin/` top-level directory makes firmware type explicit and scales if more
Marlin printers are added.

## Hardware

| Component | Spec |
|-----------|------|
| Board | Creality 4.2.2 (TMC2208 standalone) |
| Kinematics | Cartesian i3 |
| Build volume | 250×235×250mm |
| Extruder | BMG clone, direct drive, 57:11 gear ratio |
| Hotend | TZ E3 2.0 |
| Probe | BLTouch / CR Touch |
| Bed leveling | Bilinear ABL — G29 full probe each print |

## OrcaSlicer Printer Profile

Inherits from `"Creality Ender-3 V2 0.4 nozzle"` (same parent as V2 Twins).
Machine limits match V2 Twins: max_accel 4400, retraction 0.6mm.

Key differences from the Klipper Twin profiles:

| Setting | Klipper Twins | This printer |
|---------|--------------|--------------|
| `gcode_flavor` | `klipper` | `marlin` |
| `machine_start_gcode` | `PRINT_START EXTRUDER_TEMP=... BED_TEMP=...` | Full Marlin gcode (see below) |
| `machine_end_gcode` | `PRINT_END` | Full Marlin gcode (see below) |
| `machine_pause_gcode` | `PAUSE` | `M600` |
| `use_firmware_retraction` | `1` | `1` (MRisCoC supports it) |

### Start Gcode

```gcode
M220 S100      ; Reset feedrate
M221 S100      ; Reset flow
M190 S[first_layer_bed_temperature]   ; Wait for bed temp
G28            ; Home all axes
G29            ; BLTouch bilinear mesh (full probe each print)
M109 S[first_layer_temperature]       ; Wait for hotend temp
G92 E0
G1 Z2.0 F3000
G1 X1.1 Y20 Z0.3 F5000
G1 X1.1 Y200  F1500 E15  ; Purge line (left edge)
G1 X1.4 Y200  F5000
G1 X1.4 Y20   F1500 E30
G92 E0
G1 Z2.0 F3000
```

### End Gcode

```gcode
G91             ; Relative positioning
G1 Z10 F3000   ; Raise Z
G90             ; Absolute positioning
G0 X0 Y220 F3000  ; Park toolhead
M104 S0        ; Hotend off
M140 S0        ; Bed off
M106 S0        ; Fan off
M84 X Y E      ; Disable motors (keep Z holding)
```

## OrcaSlicer Process Profiles

Four profiles (quality, balanced, speed, print_in_place) inheriting from
`"0.20mm Standard @Creality Ender3 V2"`. Names suffixed `@Ender3V2-Marlin`.

## HARDWARE_NOTES.md Contents

Documents firmware version, hardware components, and calibration values (E-steps,
BLTouch offsets, Z offset) with TBD placeholders to fill in after first calibration.
