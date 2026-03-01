# Marlin Ender 3 V2 Farm Addition — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add the Ender 3 V2 (MRisCoC Marlin 2.x, stock firmware) to the repo with OrcaSlicer profiles and hardware documentation.

**Architecture:** New `marlin/ender3_v2_marlin/HARDWARE_NOTES.md` captures hardware spec and calibration placeholders. OrcaSlicer profiles in `orca_profiles/ender3_v2_marlin/` use `gcode_flavor: "marlin2"` and real Marlin start/end gcode instead of Klipper macro calls. Process profiles are identical to V2 Twins except for name suffix.

**Tech Stack:** OrcaSlicer JSON profiles, Markdown

---

### Task 1: Create hardware notes doc

**Files:**
- Create: `marlin/ender3_v2_marlin/HARDWARE_NOTES.md`

**Step 1: Create the directory and file**

```bash
mkdir -p marlin/ender3_v2_marlin
```

Then write `marlin/ender3_v2_marlin/HARDWARE_NOTES.md`:

```markdown
# Ender 3 V2 Marlin — Hardware Notes

## Firmware
- MRisCoC Professional Firmware (Marlin 2.x) — stock build
- Source: https://github.com/mriscoc/Ender3V2S1 (Ender3V2 + BLTouch variant)
- No custom Configuration.h — using pre-built .bin

## Hardware
- Board: Creality 4.2.2 (TMC2208 standalone)
- Kinematics: Cartesian i3
- Build volume: 250×235×250mm
- Extruder: BMG clone, direct drive, 57:11 gear ratio
- Hotend: TZ E3 2.0
- Probe: BLTouch / CR Touch
- Bed leveling: Bilinear ABL (G29 full probe each print)

## Calibration (fill in after tuning)
- E-steps: TBD (BMG stock ~415 steps/mm — calibrate before first print)
- BLTouch X offset: TBD
- BLTouch Y offset: TBD
- Z offset: TBD
```

**Step 2: Verify file was created**

```bash
cat marlin/ender3_v2_marlin/HARDWARE_NOTES.md
```

Expected: file contents printed without error.

**Step 3: Commit**

```bash
git add marlin/ender3_v2_marlin/HARDWARE_NOTES.md
git commit -m "Add Marlin Ender 3 V2 hardware notes"
```

---

### Task 2: Create OrcaSlicer printer profile

**Files:**
- Create: `orca_profiles/ender3_v2_marlin/printer_profile.json`

This is based on `orca_profiles/ender3_v2_twin1/printer_profile.json` with these changes:
- `gcode_flavor`: `"marlin2"` (Marlin 2.x; `"marlin"` is for 1.x)
- `machine_start_gcode`: full Marlin sequence (no PRINT_START macro)
- `machine_end_gcode`: full Marlin sequence (no PRINT_END macro)
- `machine_pause_gcode`: `"M600"` (Marlin filament change)
- `name` / `printer_settings_id`: updated to Marlin variant
- `default_print_profile`: `"0.20mm BALANCED @Ender3V2-Marlin"`
- `printer_notes`: updated for this printer

**Step 1: Create the directory and file**

```bash
mkdir -p orca_profiles/ender3_v2_marlin
```

Write `orca_profiles/ender3_v2_marlin/printer_profile.json`:

```json
{
    "adaptive_bed_mesh_margin": "0",
    "auxiliary_fan": "1",
    "bbl_use_printhost": "0",
    "bed_custom_model": "",
    "bed_custom_texture": "",
    "bed_exclude_area": [
        "0x0"
    ],
    "bed_mesh_max": "200,223",
    "bed_mesh_min": "10,10",
    "bed_mesh_probe_distance": "50,50",
    "before_layer_change_gcode": ";BEFORE_LAYER_CHANGE\n;[layer_z]\nG92 E0\n",
    "best_object_pos": "0.5,0.5",
    "change_extrusion_role_gcode": "",
    "change_filament_gcode": "M600",
    "cooling_tube_length": "5",
    "cooling_tube_retraction": "91.5",
    "default_bed_type": "",
    "default_filament_profile": [
        "Creality Generic PLA"
    ],
    "default_print_profile": "0.20mm BALANCED @Ender3V2-Marlin",
    "deretraction_speed": [
        "40"
    ],
    "disable_m73": "0",
    "emit_machine_limits_to_gcode": "1",
    "enable_filament_ramming": "1",
    "enable_long_retraction_when_cut": "0",
    "extra_loading_move": "-2",
    "extruder_clearance_height_to_lid": "25",
    "extruder_clearance_height_to_rod": "25",
    "extruder_clearance_radius": "47",
    "extruder_colour": [
        "#FCE94F"
    ],
    "extruder_offset": [
        "0x0"
    ],
    "fan_kickstart": "0",
    "fan_speedup_overhangs": "1",
    "fan_speedup_time": "0",
    "from": "User",
    "gcode_flavor": "marlin2",
    "head_wrap_detect_zone": [],
    "high_current_on_filament_swap": "0",
    "host_type": "octoprint",
    "inherits": "Creality Ender-3 V2 0.4 nozzle",
    "layer_change_gcode": "",
    "long_retractions_when_cut": [
        "0"
    ],
    "machine_end_gcode": "G91\nG1 Z10 F3000\nG90\nG0 X0 Y220 F3000\nM104 S0\nM140 S0\nM106 S0\nM84 X Y E",
    "machine_load_filament_time": "0",
    "machine_max_acceleration_e": [
        "3000",
        "3000"
    ],
    "machine_max_acceleration_extruding": [
        "4400",
        "500"
    ],
    "machine_max_acceleration_retracting": [
        "4400",
        "1000"
    ],
    "machine_max_acceleration_travel": [
        "4400",
        "1250"
    ],
    "machine_max_acceleration_x": [
        "4400",
        "500"
    ],
    "machine_max_acceleration_y": [
        "4400",
        "500"
    ],
    "machine_max_acceleration_z": [
        "100",
        "100"
    ],
    "machine_max_jerk_e": [
        "5",
        "5"
    ],
    "machine_max_jerk_x": [
        "8",
        "8"
    ],
    "machine_max_jerk_y": [
        "8",
        "8"
    ],
    "machine_max_jerk_z": [
        "0.4",
        "0.4"
    ],
    "machine_max_junction_deviation": [
        "0",
        "0"
    ],
    "machine_max_speed_e": [
        "60",
        "60"
    ],
    "machine_max_speed_x": [
        "500",
        "500"
    ],
    "machine_max_speed_y": [
        "500",
        "500"
    ],
    "machine_max_speed_z": [
        "10",
        "10"
    ],
    "machine_min_extruding_rate": [
        "0",
        "0"
    ],
    "machine_min_travel_rate": [
        "0",
        "0"
    ],
    "machine_pause_gcode": "M600",
    "machine_start_gcode": "M220 S100 ; Reset feedrate\nM221 S100 ; Reset flow\nM190 S[first_layer_bed_temperature]\nG28\nG29\nM109 S[first_layer_temperature]\nG92 E0\nG1 Z2.0 F3000\nG1 X1.1 Y20 Z0.3 F5000\nG1 X1.1 Y200 F1500 E15\nG1 X1.4 Y200 F5000\nG1 X1.4 Y20 F1500 E30\nG92 E0\nG1 Z2.0 F3000",
    "machine_tool_change_time": "0",
    "machine_unload_filament_time": "0",
    "manual_filament_change": "0",
    "max_layer_height": [
        "0.32"
    ],
    "max_resonance_avoidance_speed": "120",
    "min_layer_height": [
        "0.08"
    ],
    "min_resonance_avoidance_speed": "70",
    "name": "Creality Ender-3 V2 Marlin",
    "nozzle_diameter": [
        "0.4"
    ],
    "nozzle_height": "2.5",
    "nozzle_hrc": "0",
    "nozzle_type": "brass",
    "nozzle_volume": "0",
    "parking_pos_retraction": "92",
    "pellet_modded_printer": "0",
    "preferred_orientation": "0",
    "printable_area": [
        "0x0",
        "250x0",
        "250x235",
        "0x235"
    ],
    "printable_height": "250",
    "printer_model": "Creality Ender-3 V2",
    "printer_notes": "Ender 3 V2 Marlin — MRisCoC Professional Firmware (Marlin 2.x, stock build)\n- 4.2.2 Board (TMC2208 standalone)\n- BMG Direct Drive Extruder (57:11 gear ratio)\n- TZ E3 2.0 Hotend\n- BLTouch — offset TBD (calibrate before first print)\n- E-steps TBD (~415 steps/mm for BMG — calibrate before first print)\n- Input Shaping: N/A (Marlin firmware)",
    "printer_settings_id": "Creality Ender-3 V2 Marlin",
    "printer_structure": "i3",
    "printer_technology": "FFF",
    "printer_variant": "0.4",
    "printhost_authorization_type": "key",
    "printhost_ssl_ignore_revoke": "0",
    "printing_by_object_gcode": "",
    "purge_in_prime_tower": "1",
    "resonance_avoidance": "0",
    "retract_before_wipe": [
        "70%"
    ],
    "retract_length_toolchange": [
        "1"
    ],
    "retract_lift_above": [
        "0"
    ],
    "retract_lift_below": [
        "0"
    ],
    "retract_lift_enforce": [
        "All Surfaces"
    ],
    "retract_restart_extra": [
        "0"
    ],
    "retract_restart_extra_toolchange": [
        "0"
    ],
    "retract_when_changing_layer": [
        "1"
    ],
    "retraction_distances_when_cut": [
        "18"
    ],
    "retraction_length": [
        "0.6"
    ],
    "retraction_minimum_travel": [
        "1.5"
    ],
    "retraction_speed": [
        "40"
    ],
    "scan_first_layer": "0",
    "silent_mode": "0",
    "single_extruder_multi_material": "1",
    "support_air_filtration": "0",
    "support_chamber_temp_control": "0",
    "support_multi_bed_types": "0",
    "template_custom_gcode": "",
    "thumbnails": "48x48/PNG, 300x300/PNG",
    "thumbnails_format": "PNG",
    "time_cost": "0",
    "time_lapse_gcode": "",
    "travel_slope": [
        "3"
    ],
    "upward_compatible_machine": [],
    "use_firmware_retraction": "1",
    "use_relative_e_distances": "1",
    "version": "2.1.1.0",
    "wipe": [
        "0"
    ],
    "wipe_distance": [
        "1"
    ],
    "z_hop": [
        "0.2"
    ],
    "z_hop_types": [
        "Normal Lift"
    ],
    "z_offset": "0"
}
```

**Step 2: Validate JSON**

```bash
python3 -c "import json; json.load(open('orca_profiles/ender3_v2_marlin/printer_profile.json')); print('Valid JSON')"
```

Expected output: `Valid JSON`

**Step 3: Commit**

```bash
git add orca_profiles/ender3_v2_marlin/printer_profile.json
git commit -m "Add OrcaSlicer printer profile for Ender 3 V2 Marlin"
```

---

### Task 3: Create process profiles

**Files:**
- Create: `orca_profiles/ender3_v2_marlin/quality.json`
- Create: `orca_profiles/ender3_v2_marlin/balanced.json`
- Create: `orca_profiles/ender3_v2_marlin/speed.json`
- Create: `orca_profiles/ender3_v2_marlin/print_in_place.json`

These are identical to the V2 Twin1 process profiles except the name suffix changes from
`Twin1` → `Marlin` in `name` and `print_settings_id`.

**Step 1: Write `orca_profiles/ender3_v2_marlin/balanced.json`**

```json
{
    "bottom_shell_layers": "4",
    "bridge_flow": "0.95",
    "bridge_speed": "30",
    "brim_object_gap": "0.22",
    "brim_type": "no_brim",
    "default_acceleration": "3000",
    "elefant_foot_compensation": "0.15",
    "enable_arc_fitting": "1",
    "enable_overhang_bridge_fan": "1",
    "fan_max_speed": "100",
    "fan_min_speed": "20",
    "from": "User",
    "infill_jerk": "6",
    "inherits": "0.20mm Standard @Creality Ender3V2",
    "initial_layer_acceleration": "500",
    "initial_layer_infill_speed": "40",
    "initial_layer_jerk": "5",
    "initial_layer_line_width": "0.48",
    "initial_layer_speed": "25",
    "inner_wall_acceleration": "3000",
    "inner_wall_jerk": "6",
    "inner_wall_speed": "120",
    "internal_solid_infill_line_width": "0.42",
    "internal_solid_infill_pattern": "rectilinear",
    "internal_solid_infill_speed": "150",
    "is_custom_defined": "0",
    "line_width": "0.42",
    "name": "0.20mm BALANCED @Ender3V2-Marlin",
    "only_one_wall_top": "0",
    "outer_wall_acceleration": "2500",
    "outer_wall_jerk": "5",
    "outer_wall_line_width": "0.42",
    "outer_wall_speed": "90",
    "overhang_2_4_speed": "50",
    "overhang_3_4_speed": "30",
    "overhang_4_4_speed": "20",
    "precise_outer_wall": "0",
    "print_settings_id": "0.20mm BALANCED Ender3V2-Marlin",
    "reduce_crossing_wall": "1",
    "seam_position": "aligned",
    "seam_slope_conditional": "1",
    "seam_slope_entire_loop": "1",
    "seam_slope_inner_walls": "1",
    "skirt_loops": "2",
    "slow_down_for_layer_cooling": "1",
    "sparse_infill_density": "15%",
    "sparse_infill_pattern": "gyroid",
    "sparse_infill_speed": "180",
    "support_bottom_z_distance": "0.25",
    "support_line_width": "0.38",
    "support_on_build_plate_only": "0",
    "support_speed": "100",
    "support_style": "organic",
    "support_top_z_distance": "0.3",
    "support_type": "tree(auto)",
    "top_shell_layers": "5",
    "top_surface_acceleration": "2500",
    "top_surface_jerk": "5",
    "top_surface_speed": "60",
    "travel_acceleration": "4000",
    "travel_jerk": "8",
    "travel_speed": "300",
    "version": "2.1.1.0",
    "wall_loops": "3",
    "wall_sequence": "inner wall/outer wall"
}
```

**Step 2: Write `orca_profiles/ender3_v2_marlin/quality.json`**

```json
{
    "bottom_shell_layers": "4",
    "bridge_density": "100%",
    "bridge_flow": "0.95",
    "bridge_speed": "30",
    "brim_object_gap": "0.22",
    "brim_type": "no_brim",
    "default_acceleration": "2500",
    "detect_thin_wall": "1",
    "elefant_foot_compensation": "0.15",
    "enable_arc_fitting": "1",
    "enable_overhang_bridge_fan": "1",
    "fan_max_speed": "100",
    "fan_min_speed": "20",
    "from": "User",
    "infill_jerk": "5",
    "inherits": "0.20mm Standard @Creality Ender3V2",
    "initial_layer_acceleration": "500",
    "initial_layer_infill_speed": "35",
    "initial_layer_jerk": "5",
    "initial_layer_line_width": "0.48",
    "initial_layer_speed": "25",
    "inner_wall_acceleration": "2000",
    "inner_wall_jerk": "5",
    "inner_wall_speed": "80",
    "internal_solid_infill_line_width": "0.42",
    "internal_solid_infill_pattern": "rectilinear",
    "internal_solid_infill_speed": "100",
    "is_custom_defined": "0",
    "line_width": "0.42",
    "name": "0.20mm QUALITY @Ender3V2-Marlin",
    "only_one_wall_top": "0",
    "outer_wall_acceleration": "2000",
    "outer_wall_jerk": "3",
    "outer_wall_line_width": "0.42",
    "outer_wall_speed": "70",
    "overhang_2_4_speed": "40",
    "overhang_3_4_speed": "25",
    "overhang_4_4_speed": "15",
    "precise_outer_wall": "1",
    "print_settings_id": "0.20mm QUALITY Ender3V2-Marlin",
    "reduce_crossing_wall": "1",
    "seam_position": "aligned",
    "seam_slope_conditional": "1",
    "seam_slope_entire_loop": "1",
    "seam_slope_inner_walls": "1",
    "skirt_loops": "2",
    "slow_down_for_layer_cooling": "1",
    "sparse_infill_density": "15%",
    "sparse_infill_pattern": "gyroid",
    "sparse_infill_speed": "120",
    "support_bottom_z_distance": "0.25",
    "support_line_width": "0.38",
    "support_on_build_plate_only": "0",
    "support_speed": "80",
    "support_style": "organic",
    "support_top_z_distance": "0.3",
    "support_type": "tree(auto)",
    "top_shell_layers": "5",
    "top_surface_acceleration": "2000",
    "top_surface_jerk": "3",
    "top_surface_speed": "50",
    "travel_acceleration": "3000",
    "travel_jerk": "7",
    "travel_speed": "300",
    "version": "2.1.1.0",
    "wall_loops": "3",
    "wall_sequence": "inner wall/outer wall"
}
```

**Step 3: Write `orca_profiles/ender3_v2_marlin/speed.json`**

```json
{
    "bottom_shell_layers": "4",
    "bridge_flow": "0.95",
    "bridge_speed": "35",
    "brim_object_gap": "0.22",
    "brim_type": "no_brim",
    "default_acceleration": "3500",
    "elefant_foot_compensation": "0.15",
    "enable_arc_fitting": "1",
    "enable_overhang_bridge_fan": "1",
    "fan_max_speed": "100",
    "fan_min_speed": "20",
    "from": "User",
    "infill_jerk": "7",
    "inherits": "0.20mm Standard @Creality Ender3V2",
    "initial_layer_acceleration": "500",
    "initial_layer_infill_speed": "40",
    "initial_layer_jerk": "5",
    "initial_layer_line_width": "0.48",
    "initial_layer_speed": "30",
    "inner_wall_acceleration": "3500",
    "inner_wall_jerk": "7",
    "inner_wall_speed": "150",
    "internal_solid_infill_line_width": "0.45",
    "internal_solid_infill_pattern": "rectilinear",
    "internal_solid_infill_speed": "200",
    "is_custom_defined": "0",
    "line_width": "0.45",
    "name": "0.20mm SPEED @Ender3V2-Marlin",
    "only_one_wall_top": "1",
    "outer_wall_acceleration": "3000",
    "outer_wall_jerk": "6",
    "outer_wall_line_width": "0.42",
    "outer_wall_speed": "120",
    "overhang_2_4_speed": "60",
    "overhang_3_4_speed": "35",
    "overhang_4_4_speed": "20",
    "precise_outer_wall": "0",
    "print_settings_id": "0.20mm SPEED Ender3V2-Marlin",
    "reduce_crossing_wall": "1",
    "seam_position": "aligned",
    "seam_slope_conditional": "1",
    "seam_slope_entire_loop": "1",
    "seam_slope_inner_walls": "1",
    "skirt_loops": "1",
    "slow_down_for_layer_cooling": "1",
    "sparse_infill_density": "15%",
    "sparse_infill_pattern": "gyroid",
    "sparse_infill_speed": "250",
    "support_bottom_z_distance": "0.2",
    "support_line_width": "0.4",
    "support_on_build_plate_only": "0",
    "support_speed": "150",
    "support_style": "organic",
    "support_top_z_distance": "0.25",
    "support_type": "tree(auto)",
    "top_shell_layers": "4",
    "top_surface_acceleration": "3000",
    "top_surface_jerk": "6",
    "top_surface_speed": "80",
    "travel_acceleration": "4400",
    "travel_jerk": "8",
    "travel_speed": "350",
    "version": "2.1.1.0",
    "wall_loops": "3",
    "wall_sequence": "inner wall/outer wall"
}
```

**Step 4: Write `orca_profiles/ender3_v2_marlin/print_in_place.json`**

```json
{
    "bottom_shell_layers": "4",
    "bridge_density": "100%",
    "bridge_flow": "0.95",
    "bridge_speed": "30",
    "brim_object_gap": "0.22",
    "brim_type": "outer_only",
    "brim_width": "3",
    "default_acceleration": "1500",
    "detect_thin_wall": "1",
    "elefant_foot_compensation": "0.15",
    "enable_arc_fitting": "1",
    "enable_overhang_bridge_fan": "1",
    "fan_max_speed": "100",
    "fan_min_speed": "30",
    "from": "User",
    "infill_jerk": "5",
    "inherits": "0.20mm Standard @Creality Ender3V2",
    "initial_layer_acceleration": "500",
    "initial_layer_infill_speed": "35",
    "initial_layer_jerk": "5",
    "initial_layer_line_width": "0.48",
    "initial_layer_speed": "25",
    "inner_wall_acceleration": "1500",
    "inner_wall_jerk": "5",
    "inner_wall_speed": "60",
    "internal_solid_infill_line_width": "0.42",
    "internal_solid_infill_pattern": "rectilinear",
    "internal_solid_infill_speed": "80",
    "is_custom_defined": "0",
    "line_width": "0.42",
    "name": "0.20mm PRINT-IN-PLACE @Ender3V2-Marlin",
    "only_one_wall_top": "0",
    "outer_wall_acceleration": "1200",
    "outer_wall_jerk": "3",
    "outer_wall_line_width": "0.40",
    "outer_wall_speed": "45",
    "overhang_2_4_speed": "40",
    "overhang_3_4_speed": "25",
    "overhang_4_4_speed": "15",
    "precise_outer_wall": "1",
    "print_settings_id": "0.20mm PRINT-IN-PLACE Ender3V2-Marlin",
    "reduce_crossing_wall": "1",
    "seam_position": "aligned",
    "seam_slope_conditional": "1",
    "seam_slope_entire_loop": "1",
    "seam_slope_inner_walls": "1",
    "skirt_loops": "2",
    "slow_down_for_layer_cooling": "1",
    "slow_down_min_speed": "20",
    "sparse_infill_density": "15%",
    "sparse_infill_pattern": "grid",
    "sparse_infill_speed": "100",
    "support_bottom_z_distance": "0.25",
    "support_line_width": "0.38",
    "support_on_build_plate_only": "0",
    "support_speed": "100",
    "support_style": "organic",
    "support_top_z_distance": "0.3",
    "support_type": "tree(auto)",
    "top_shell_layers": "5",
    "top_surface_acceleration": "1200",
    "top_surface_jerk": "3",
    "top_surface_pattern": "monotoniclines",
    "top_surface_speed": "50",
    "travel_acceleration": "2500",
    "travel_jerk": "7",
    "travel_speed": "250",
    "version": "2.1.1.0",
    "wall_loops": "3",
    "wall_sequence": "inner wall/outer wall"
}
```

**Step 5: Validate all four JSON files**

```bash
for f in orca_profiles/ender3_v2_marlin/balanced.json \
          orca_profiles/ender3_v2_marlin/quality.json \
          orca_profiles/ender3_v2_marlin/speed.json \
          orca_profiles/ender3_v2_marlin/print_in_place.json; do
  python3 -c "import json; json.load(open('$f')); print('Valid: $f')"
done
```

Expected: four `Valid: ...` lines, no errors.

**Step 6: Commit**

```bash
git add orca_profiles/ender3_v2_marlin/
git commit -m "Add OrcaSlicer process profiles for Ender 3 V2 Marlin"
```

---

### Task 4: Update project memory

**Files:**
- Modify: `/home/drupi/.claude/projects/-home-drupi-printfarm/memory/MEMORY.md`

**Step 1: Add the printer to the inventory table**

In the Printer Inventory table, add a new row after the Ender 3 V2 Twin 2 row:

```
| Ender 3 V2 Marlin | 4.2.2 (STM32F103) | Cartesian | 250x235 | BMG DD | TZ E3 2.0 | BLTouch | N/A (Marlin) | Pi (TBD) |
```

Note: Host Pi details are TBD — update this after the printer is physically set up.

**Step 2: Add a note under "Key Config Notes" or a new section**

Add to the relevant section:

```markdown
## Marlin Printer Notes
- Ender 3 V2 Marlin: MRisCoC Professional Firmware (Marlin 2.x, stock build); OrcaSlicer gcode_flavor: marlin2; uses G28+G29 start sequence, no Klipper macros
- E-steps and BLTouch offset TBD — see marlin/ender3_v2_marlin/HARDWARE_NOTES.md
```

**Step 3: Commit**

```bash
git add /home/drupi/.claude/projects/-home-drupi-printfarm/memory/MEMORY.md
git commit -m "Update memory: add Ender 3 V2 Marlin to printer inventory"
```

---

## Post-Implementation Checklist

After completing all tasks:

- [ ] `marlin/ender3_v2_marlin/HARDWARE_NOTES.md` exists
- [ ] `orca_profiles/ender3_v2_marlin/` contains 5 files (printer_profile + 4 process profiles)
- [ ] All JSON files pass `python3 -c "import json; json.load(open(...))"` validation
- [ ] MEMORY.md updated with new printer row
- [ ] 4 commits total (one per task)

## After First Calibration

Once the printer is physically set up and calibrated, update `HARDWARE_NOTES.md` with:
- Actual E-steps (M503 → steps_per_mm for E)
- BLTouch X/Y offset
- Z offset
