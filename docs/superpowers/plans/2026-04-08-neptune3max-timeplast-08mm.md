# Neptune 3 Max Timeplast Soap 0.8mm OrcaSlicer Profiles — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add three OrcaSlicer JSON profiles to `orca_profiles/neptune3_max/` so the Neptune 3 Max can slice Timeplast Soap Gen 5 filament with a 0.8mm nozzle.

**Architecture:** Three standalone files — a printer variant profile (`printer_08mm.json`), a filament profile (`filament_soap.json`), and a process profile (`08mm_standard.json`). The printer profile disables firmware retraction so the filament profile can supply Timeplast's specific slow retraction values directly.

**Tech Stack:** OrcaSlicer 2.3.2.0 JSON profile format. No build step — files are imported manually into OrcaSlicer.

---

## File Map

| File | Role |
|---|---|
| `orca_profiles/neptune3_max/printer_08mm.json` | **Create** — machine variant for 0.8mm nozzle; disables firmware retraction |
| `orca_profiles/neptune3_max/filament_soap.json` | **Create** — Timeplast Soap temps, cooling, flow, PA, retraction |
| `orca_profiles/neptune3_max/08mm_standard.json` | **Create** — 0.30mm process profile: speeds, line widths, layer heights, fan |

No existing files are modified.

---

## Task 1: Printer variant profile (`printer_08mm.json`)

**Files:**
- Create: `orca_profiles/neptune3_max/printer_08mm.json`

- [ ] **Step 1: Create the file**

```json
{
    "adaptive_bed_mesh_margin": "0",
    "auxiliary_fan": "0",
    "bbl_use_printhost": "0",
    "bed_custom_model": "",
    "bed_custom_texture": "",
    "bed_exclude_area": [
        "0x0"
    ],
    "bed_mesh_max": "397,415",
    "bed_mesh_min": "33,16",
    "bed_mesh_probe_distance": "50,50",
    "before_layer_change_gcode": ";BEFORE_LAYER_CHANGE\nG92 E0\n;[layer_z]\n",
    "best_object_pos": "0.5,0.5",
    "change_extrusion_role_gcode": "",
    "change_filament_gcode": "M600",
    "cooling_tube_length": "5",
    "cooling_tube_retraction": "91.5",
    "default_bed_type": "",
    "default_filament_profile": [
        "Timeplast Soap @Neptune3Max"
    ],
    "default_print_profile": "0.30mm STANDARD @Neptune3Max (0.8)",
    "deretraction_speed": [
        "40"
    ],
    "disable_m73": "0",
    "emit_machine_limits_to_gcode": "1",
    "enable_filament_ramming": "1",
    "enable_long_retraction_when_cut": "0",
    "extra_loading_move": "-2",
    "extruder_clearance_height_to_lid": "34",
    "extruder_clearance_height_to_rod": "34",
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
    "gcode_flavor": "klipper",
    "head_wrap_detect_zone": [],
    "high_current_on_filament_swap": "0",
    "host_type": "octoprint",
    "inherits": "Elegoo Neptune 3 Max 0.4 nozzle",
    "label_objects": "1",
    "layer_change_gcode": ";AFTER_LAYER_CHANGE\n;[layer_z]",
    "long_retractions_when_cut": [
        "0"
    ],
    "machine_end_gcode": "PRINT_END",
    "machine_load_filament_time": "0",
    "machine_max_acceleration_e": [
        "3000",
        "3000"
    ],
    "machine_max_acceleration_extruding": [
        "3000",
        "500"
    ],
    "machine_max_acceleration_retracting": [
        "3000",
        "1000"
    ],
    "machine_max_acceleration_travel": [
        "3000",
        "500"
    ],
    "machine_max_acceleration_x": [
        "3000",
        "500"
    ],
    "machine_max_acceleration_y": [
        "3000",
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
        "15",
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
    "machine_pause_gcode": "PAUSE",
    "machine_start_gcode": "PRINT_START EXTRUDER_TEMP=[first_layer_temperature] BED_TEMP=[first_layer_bed_temperature]",
    "machine_tool_change_time": "0",
    "machine_unload_filament_time": "0",
    "manual_filament_change": "0",
    "max_layer_height": [
        "0.60"
    ],
    "max_resonance_avoidance_speed": "120",
    "min_layer_height": [
        "0.16"
    ],
    "min_resonance_avoidance_speed": "70",
    "name": "Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.8mm)",
    "nozzle_diameter": [
        "0.8"
    ],
    "nozzle_height": "2.5",
    "nozzle_hrc": "0",
    "nozzle_type": "hardened_steel",
    "nozzle_volume": "0",
    "parking_pos_retraction": "92",
    "pellet_modded_printer": "0",
    "preferred_orientation": "0",
    "printable_area": [
        "0x0",
        "420x0",
        "420x420",
        "0x420"
    ],
    "printable_height": "500",
    "printer_model": "Elegoo Neptune 3 Max",
    "printer_notes": "Optimized for Neptune 3 Max - 0.8mm nozzle:\n- ZNP Robin Nano DW v2.2 board\n- Direct Drive extruder\n- Slice Engineering Copperhead hotend\n- 0.8mm nozzle\n- Stock probe (offset: -28.5, 22)\n- Input Shaping (mzv @ 52.2Hz X, ei @ 40.0Hz Y)\n- Max Accel: 3000 mm/s\u00b2\n- Firmware retraction DISABLED — retraction controlled by filament profile\n- Paired with Timeplast Soap filament profile",
    "printer_settings_id": "Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.8mm)",
    "printer_structure": "i3",
    "printer_technology": "FFF",
    "printer_variant": "0.8",
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
        "0.4"
    ],
    "retraction_minimum_travel": [
        "1.5"
    ],
    "retraction_speed": [
        "15"
    ],
    "scan_first_layer": "0",
    "silent_mode": "0",
    "single_extruder_multi_material": "1",
    "support_air_filtration": "0",
    "support_chamber_temp_control": "0",
    "support_multi_bed_types": "0",
    "template_custom_gcode": "",
    "thumbnails": "80x80/PNG, 300x300/PNG",
    "thumbnails_format": "PNG",
    "time_cost": "0",
    "time_lapse_gcode": "",
    "travel_slope": [
        "3"
    ],
    "upward_compatible_machine": [],
    "use_firmware_retraction": "0",
    "use_relative_e_distances": "1",
    "version": "2.3.2.0",
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

Save to `orca_profiles/neptune3_max/printer_08mm.json`.

> **Note:** `use_firmware_retraction` is set to `"0"` here (unlike the 0.4mm and 0.6mm profiles which use `"1"`). This lets the filament profile's retraction values take effect. `retraction_length` and `retraction_speed` are set to Timeplast values (0.4mm, 15mm/s) at the machine level as a fallback; the filament profile reinforces them.

- [ ] **Step 2: Validate JSON**

```bash
python3 -m json.tool orca_profiles/neptune3_max/printer_08mm.json > /dev/null && echo "VALID"
```

Expected output: `VALID`

- [ ] **Step 3: Commit**

```bash
git add orca_profiles/neptune3_max/printer_08mm.json
git commit -m "feat: Neptune 3 Max — add 0.8mm printer variant profile for Timeplast Soap"
```

---

## Task 2: Filament profile (`filament_soap.json`)

**Files:**
- Create: `orca_profiles/neptune3_max/filament_soap.json`

- [ ] **Step 1: Create the file**

```json
{
    "name": "Timeplast Soap @Neptune3Max",
    "from": "User",
    "inherits": "Generic PVA",
    "filament_type": [
        "PVA"
    ],
    "nozzle_temperature": [
        "230"
    ],
    "nozzle_temperature_initial_layer": [
        "230"
    ],
    "bed_temperature": [
        "75"
    ],
    "bed_temperature_initial_layer": [
        "75"
    ],
    "close_fan_the_first_x_layers": [
        "3"
    ],
    "fan_always_on": [
        "0"
    ],
    "overhang_fan_speed": [
        "15"
    ],
    "filament_flow_ratio": [
        "0.92"
    ],
    "filament_max_volumetric_speed": [
        "12"
    ],
    "filament_retract_length": [
        "0.4"
    ],
    "filament_retract_speed": [
        "15"
    ],
    "filament_deretract_speed": [
        "15"
    ],
    "pressure_advance": [
        "0.05"
    ],
    "version": "2.3.2.0"
}
```

Save to `orca_profiles/neptune3_max/filament_soap.json`.

> **Notes:**
> - `inherits: "Generic PVA"` — closest OrcaSlicer built-in category for a water-based specialty material. All relevant fields are explicitly overridden so the parent's temps/speeds don't bleed through.
> - `filament_retract_length` / `filament_retract_speed` / `filament_deretract_speed` take effect because `use_firmware_retraction: "0"` is set in `printer_08mm.json`.
> - `filament_flow_ratio` is the correct field name at the filament level in OrcaSlicer 2.3.x (not `flow_ratio`).
> - `pressure_advance` here sets the default PA for this filament; it can be tuned via Klipper's `PRESSURE_ADVANCE` command and written back if needed.

- [ ] **Step 2: Validate JSON**

```bash
python3 -m json.tool orca_profiles/neptune3_max/filament_soap.json > /dev/null && echo "VALID"
```

Expected output: `VALID`

- [ ] **Step 3: Commit**

```bash
git add orca_profiles/neptune3_max/filament_soap.json
git commit -m "feat: Neptune 3 Max — add Timeplast Soap filament profile"
```

---

## Task 3: Process profile (`08mm_standard.json`)

**Files:**
- Create: `orca_profiles/neptune3_max/08mm_standard.json`

- [ ] **Step 1: Create the file**

```json
{
    "name": "0.30mm STANDARD @Neptune3Max (0.8)",
    "print_settings_id": "0.30mm STANDARD @Neptune3Max (0.8)",
    "from": "User",
    "inherits": "",
    "version": "2.3.2.0",
    "is_custom_defined": "0",
    "layer_height": "0.30",
    "initial_layer_print_height": "0.35",
    "line_width": "0.80",
    "initial_layer_line_width": "0.80",
    "outer_wall_line_width": "0.80",
    "inner_wall_line_width": "0.80",
    "top_surface_line_width": "0.80",
    "internal_solid_infill_line_width": "0.84",
    "sparse_infill_line_width": "0.88",
    "support_line_width": "0.70",
    "wall_loops": "2",
    "top_shell_layers": "3",
    "top_shell_thickness": "0.9",
    "bottom_shell_layers": "3",
    "bottom_shell_thickness": "0.9",
    "sparse_infill_density": "20%",
    "sparse_infill_pattern": "gyroid",
    "internal_solid_infill_pattern": "rectilinear",
    "only_one_wall_top": "0",
    "detect_thin_wall": "1",
    "reduce_crossing_wall": "1",
    "wall_sequence": "inner-outer-inner wall",
    "outer_wall_speed": "60",
    "inner_wall_speed": "90",
    "sparse_infill_speed": "110",
    "internal_solid_infill_speed": "90",
    "top_surface_speed": "50",
    "gap_infill_speed": "60",
    "support_speed": "60",
    "bridge_speed": "25",
    "internal_bridge_speed": "60%",
    "travel_speed": "160",
    "initial_layer_speed": "15",
    "initial_layer_infill_speed": "30",
    "default_acceleration": "2500",
    "outer_wall_acceleration": "2000",
    "inner_wall_acceleration": "2500",
    "top_surface_acceleration": "2000",
    "travel_acceleration": "3000",
    "initial_layer_acceleration": "500",
    "infill_jerk": "6",
    "outer_wall_jerk": "5",
    "inner_wall_jerk": "6",
    "top_surface_jerk": "5",
    "initial_layer_jerk": "5",
    "travel_jerk": "7",
    "overhang_1_4_speed": "0",
    "overhang_2_4_speed": "45",
    "overhang_3_4_speed": "25",
    "overhang_4_4_speed": "15",
    "bridge_flow": "0.90",
    "fan_max_speed": "15",
    "fan_min_speed": "0",
    "enable_overhang_bridge_fan": "1",
    "overhang_fan_speed": "15",
    "slow_down_for_layer_cooling": "1",
    "elefant_foot_compensation": "0.10",
    "enable_arc_fitting": "1",
    "seam_position": "aligned",
    "seam_slope_type": "external",
    "seam_slope_conditional": "1",
    "seam_slope_entire_loop": "1",
    "seam_slope_inner_walls": "1",
    "brim_type": "no_brim",
    "brim_object_gap": "0.30",
    "skirt_loops": "2",
    "support_type": "tree(auto)",
    "support_style": "organic",
    "support_on_build_plate_only": "0",
    "support_top_z_distance": "0.30",
    "support_bottom_z_distance": "0.25",
    "precise_outer_wall": "0",
    "precise_z_height": "1",
    "wipe_on_loops": "1",
    "wipe_before_external_loop": "1"
}
```

Save to `orca_profiles/neptune3_max/08mm_standard.json`.

> **Notes:**
> - `fan_max_speed: "15"` and `fan_min_speed: "0"` enforce Timeplast's cooling requirements at the process level. These are consistent with the filament profile's `close_fan_the_first_x_layers: "3"`.
> - Overhang speeds are conservative (2/4: 45, 3/4: 25, 4/4: 15) — soap doesn't bridge well.
> - Top surface speed capped at 50mm/s (below the 60mm/s outer wall) for better soap surface finish.
> - No `inherits` — fully self-contained, mirrors the `06mm_standard.json` approach.

- [ ] **Step 2: Validate JSON**

```bash
python3 -m json.tool orca_profiles/neptune3_max/08mm_standard.json > /dev/null && echo "VALID"
```

Expected output: `VALID`

- [ ] **Step 3: Commit**

```bash
git add orca_profiles/neptune3_max/08mm_standard.json
git commit -m "feat: Neptune 3 Max — add 0.30mm standard process profile for 0.8mm nozzle"
```

---

## Task 4: Import and verify in OrcaSlicer

- [ ] **Step 1: Import the three profiles into OrcaSlicer**

In OrcaSlicer:
1. Go to **Printer** tab → click the dropdown → **Import** → select `printer_08mm.json`
2. Go to **Filament** tab → **Import** → select `filament_soap.json`
3. Go to **Process** tab → **Import** → select `08mm_standard.json`

- [ ] **Step 2: Verify profile loads correctly**

Select printer `"Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.8mm)"`. Confirm:
- Nozzle diameter shows **0.8mm**
- Default filament auto-selects **Timeplast Soap @Neptune3Max**
- Default process auto-selects **0.30mm STANDARD @Neptune3Max (0.8)**

Switch to filament profile. Confirm:
- Nozzle temp: **230°C**
- Bed temp: **75°C**
- Fan always on: **OFF**
- First X layers no fan: **3**

Switch to process profile. Confirm:
- Layer height: **0.30mm**
- First layer height: **0.35mm**
- Outer wall speed: **60mm/s**
- Fan max: **15%**

- [ ] **Step 3: Slice a test object and inspect gcode**

Slice any small object (e.g., a 20mm cube). In the preview, verify:
- First layer temp: `M104 S230` / `M109 S230`
- Bed temp: `M140 S75` / `M190 S75`
- No `M106` fan commands in first 3 layers
- `M106 S38` (15% of 255) appears after layer 3
- Retraction gcode (`G1 E-0.4 F900`) appears rather than `G10` firmware retraction

- [ ] **Step 4: Commit final verification note**

```bash
git commit --allow-empty -m "chore: Neptune 3 Max 0.8mm Timeplast profiles verified in OrcaSlicer"
```
