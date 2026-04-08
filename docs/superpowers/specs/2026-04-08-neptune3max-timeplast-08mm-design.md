# Neptune 3 Max — Timeplast Soap 0.8mm OrcaSlicer Profile Design

**Date:** 2026-04-08  
**Printer:** Elegoo Neptune 3 Max  
**Filament:** Timeplast Soap Gen 5 (unlabeled roll, confirmed by product match)  
**Nozzle:** 0.8mm  

---

## Goal

Add OrcaSlicer support for printing Timeplast Soap Gen 5 on the Neptune 3 Max with a 0.8mm nozzle. Timeplast mandates 0.8mm as the minimum nozzle size for this material.

Three files will be added to `orca_profiles/neptune3_max/`:

1. `printer_08mm.json` — 0.8mm printer variant profile
2. `filament_soap.json` — Timeplast Soap filament profile
3. `08mm_standard.json` — 0.30mm standard process profile for 0.8mm nozzle

---

## Source Settings

Settings sourced from Timeplast's official Soap Gen 5 product page.

| Parameter | Value |
|---|---|
| Nozzle temp | 230°C (range 230–245°C; drop 10°C if print fails) |
| Bed temp | 75°C |
| Flow ratio | 92% |
| Pressure advance | 0.05 |
| Retraction distance | 0.4mm |
| Retraction speed | 15mm/s |
| Max volumetric speed | 12 mm³/s |
| Layer height | 0.30mm |
| First layer height | 0.35mm |
| Fan max | 15% |
| Fan min | 0% |
| No cooling first | 3 layers |
| Fan always on | OFF |
| Outer wall speed | 60mm/s |
| Inner wall speed | 90mm/s |
| Infill speed | 100–120mm/s (using 110) |
| Travel speed | 160mm/s |
| First layer speed | 15mm/s |
| Wall loops | 2 |
| Top/bottom shells | 3 |
| Infill density | 20% |
| Elephant foot comp | 0.10mm |

---

## File 1: `printer_08mm.json`

Cloned from `printer_06mm.json`. Changes:

- `nozzle_diameter`: `"0.8"`
- `printer_variant`: `"0.8"`
- `name` / `printer_settings_id`: `"Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.8mm)"`
- `max_layer_height`: `"0.60"` (75% of 0.8mm)
- `min_layer_height`: `"0.16"` (20% of 0.8mm)
- `default_print_profile`: `"0.30mm STANDARD @Neptune3Max (0.8)"`
- `default_filament_profile`: `["Timeplast Soap @Neptune3Max"]`
- `printer_notes`: updated to reflect 0.8mm nozzle

All machine limits, bed mesh coords, start/end gcode, retraction, and z_hop are inherited unchanged from the base machine — these are machine-level settings not specific to nozzle size.

---

## File 2: `filament_soap.json`

New filament profile. Follows the pattern of `orca_profiles/wanhao_d6/filament_pla.json`.

Key settings:

- `name`: `"Timeplast Soap @Neptune3Max"`
- `filament_type`: `"PVA"` (closest OrcaSlicer category for water-soluble specialty material)
- Nozzle temp: `230°C` (first layer: `230°C`)
- Bed temp: `75°C` (first layer: `75°C`)
- `fan_max_speed`: `"15"`
- `fan_min_speed`: `"0"`
- `slow_down_for_layer_cooling`: `"1"`
- `fan_cooling_layer_time`: first 3 layers no fan (achieved via `close_fan_the_first_x_layers: "3"`)
- `fan_always_on`: `"0"`
- `overhang_fan_speed`: `"15"`
- `flow_ratio`: `"0.92"`
- `filament_max_volumetric_speed`: `"12"`
- `pressure_advance`: `"0.05"`
- `retraction_length`: `"0.4"` (filament-level override)
- `retraction_speed`: `"15"`
- `use_firmware_retraction`: `"0"` — **overrides** the printer-level `use_firmware_retraction: 1` so OrcaSlicer controls retraction directly with Timeplast's specific values

---

## File 3: `08mm_standard.json`

Standalone process profile (no `inherits` — mirrors `06mm_standard.json` approach). Contains 0.8mm line widths and Timeplast-appropriate speeds.

Key settings:

- `name` / `print_settings_id`: `"0.30mm STANDARD @Neptune3Max (0.8)"`
- `layer_height`: `"0.30"`
- `initial_layer_print_height`: `"0.35"`
- `line_width`: `"0.80"`
- `initial_layer_line_width`: `"0.80"`
- `outer_wall_line_width`: `"0.80"`
- `inner_wall_line_width`: `"0.80"`
- `internal_solid_infill_line_width`: `"0.84"` (slightly wider for solid fill)
- `sparse_infill_line_width`: `"0.88"` (wider for infill)
- `top_surface_line_width`: `"0.80"`
- `support_line_width`: `"0.70"`
- `wall_loops`: `"2"`
- `top_shell_layers`: `"3"`, `bottom_shell_layers`: `"3"`
- `top_shell_thickness`: `"0.9"`, `bottom_shell_thickness`: `"0.9"`
- `sparse_infill_density`: `"20%"`, pattern: `"gyroid"`
- `elefant_foot_compensation`: `"0.10"`
- `enable_arc_fitting`: `"1"`
- Speeds: outer wall `60`, inner wall `90`, infill `110`, travel `160`, initial layer `15`, initial layer infill `30`, gap infill `80`
- Overhang speeds: conservative (2/4: 45, 3/4: 25, 4/4: 15) — soap bridges poorly
- `default_acceleration`: `"2500"`, `travel_acceleration`: `"3000"`, `initial_layer_acceleration`: `"500"`
- `outer_wall_acceleration`: `"2000"`, `inner_wall_acceleration`: `"2500"`, `top_surface_acceleration`: `"2000"`
- Support: organic tree, `support_top_z_distance: "0.30"`, `support_bottom_z_distance: "0.25"`
- Seam: aligned, with slope (matches existing profiles)
- `brim_type`: `"no_brim"` (bed at 75°C provides good adhesion)
- `skirt_loops`: `"2"`
- Fan settings: `fan_max_speed: "15"`, `fan_min_speed: "0"` — consistent with filament profile
- `slow_down_for_layer_cooling`: `"1"`
- `enable_overhang_bridge_fan`: `"1"`
- `bridge_speed`: `"25"`, `bridge_flow`: `"0.90"`

---

## Key Decision: Firmware Retraction Override

The Neptune 3 Max printer profile sets `use_firmware_retraction: 1`, which means Klipper's `[firmware_retraction]` section controls retraction at runtime. Timeplast requires a specific slow retraction (0.4mm @ 15mm/s) that differs from the farm standard.

**Decision:** Set `use_firmware_retraction: "0"` in `filament_soap.json` to let OrcaSlicer inject retraction gcode directly for this filament. This avoids requiring a Klipper config change before every soap print.

---

## What Is Not Changing

- `klipper/printers/neptune3_max/printer.cfg` — no changes
- `klipper/shared/start.cfg` / `macros.cfg` — no changes
- Existing Neptune 3 Max OrcaSlicer profiles — untouched
