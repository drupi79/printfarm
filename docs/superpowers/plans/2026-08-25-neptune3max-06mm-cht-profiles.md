# Neptune 3 Max 0.6mm CHT Profile Set Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a parallel OrcaSlicer profile set (printer + 3 process + 2 filament) for the Slice Engineering Diamondback CHT v6 0.6mm nozzle on the Neptune 3 Max.

**Architecture:** Six new JSON files written to both the repo (`orca_profiles/neptune3_max/`) and the live OrcaSlicer user directory (`/mnt/c/Users/drupi/AppData/Roaming/OrcaSlicer/user/d95bb8be-f499-4d5e-ac76-5aaa5f7267cd/`). Each OrcaSlicer file gets a paired `.info` sidecar. The CHT profiles are entirely self-contained — no existing profiles are modified.

**Tech Stack:** JSON, OrcaSlicer 2.3.x profile format, bash for JSON validation

---

## File Map

| Repo path | OrcaSlicer path (relative to user dir) |
|---|---|
| `orca_profiles/neptune3_max/printer_06mm_cht.json` | `machine/Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT).json` |
| `orca_profiles/neptune3_max/06mm_cht_standard.json` | `process/0.30mm STANDARD @Neptune3Max (0.6 CHT).json` |
| `orca_profiles/neptune3_max/06mm_cht_fine.json` | `process/0.24mm FINE @Neptune3Max (0.6 CHT).json` |
| `orca_profiles/neptune3_max/06mm_cht_draft.json` | `process/0.40mm DRAFT @Neptune3Max (0.6 CHT).json` |
| `orca_profiles/neptune3_max/filament_pla_cht.json` | `filament/PLA @Neptune3Max CHT.json` |
| `orca_profiles/neptune3_max/filament_petg_cht.json` | `filament/PETG @Neptune3Max CHT.json` |

**OrcaSlicer user dir** (used throughout): `/mnt/c/Users/drupi/AppData/Roaming/OrcaSlicer/user/d95bb8be-f499-4d5e-ac76-5aaa5f7267cd`

**Validation command** (used throughout): `python3 -m json.tool <file> > /dev/null && echo OK`

---

## Task 1: Create the CHT printer profile (repo)

**Files:**
- Create: `orca_profiles/neptune3_max/printer_06mm_cht.json`

- [ ] **Step 1: Write the repo printer profile**

Create `/home/drupi/printfarm/orca_profiles/neptune3_max/printer_06mm_cht.json` with this exact content:

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
        "Generic PLA @Elegoo"
    ],
    "default_print_profile": "0.30mm STANDARD @Neptune3Max (0.6 CHT)",
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
        "0.45"
    ],
    "max_resonance_avoidance_speed": "120",
    "min_layer_height": [
        "0.12"
    ],
    "min_resonance_avoidance_speed": "70",
    "name": "Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT)",
    "nozzle_diameter": [
        "0.6"
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
    "printer_notes": "Optimized for Neptune 3 Max - 0.6mm Diamondback CHT v6 nozzle:\n- ZNP Robin Nano DW v2.2 board\n- Direct Drive extruder\n- Slice Engineering Copperhead hotend\n- Slice Engineering Diamondback CHT v6 0.6mm nozzle\n- Stock probe (offset: -28.5, 22)\n- Input Shaping (mzv @ 52.2Hz X, ei @ 40.0Hz Y)\n- Max Accel: 3000 mm/s²\n- Max flow (Copperhead + CHT v6 0.6mm): ~35mm³/s PLA | ~25mm³/s PETG",
    "printer_settings_id": "Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT)",
    "printer_structure": "i3",
    "printer_technology": "FFF",
    "printer_variant": "0.6",
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
    "thumbnails": "80x80/PNG, 300x300/PNG",
    "thumbnails_format": "PNG",
    "time_cost": "0",
    "time_lapse_gcode": "",
    "travel_slope": [
        "3"
    ],
    "upward_compatible_machine": [],
    "use_firmware_retraction": "1",
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

- [ ] **Step 2: Validate JSON**

```bash
python3 -m json.tool /home/drupi/printfarm/orca_profiles/neptune3_max/printer_06mm_cht.json > /dev/null && echo OK
```

Expected: `OK`

- [ ] **Step 3: Spot-check critical fields**

```bash
python3 -c "
import json
d = json.load(open('/home/drupi/printfarm/orca_profiles/neptune3_max/printer_06mm_cht.json'))
assert d['name'] == 'Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT)'
assert d['default_print_profile'] == '0.30mm STANDARD @Neptune3Max (0.6 CHT)'
assert d['nozzle_diameter'] == ['0.6']
assert 'CHT' in d['printer_notes']
assert '35' in d['printer_notes']
print('All assertions passed')
"
```

Expected: `All assertions passed`

---

## Task 2: Create the CHT printer profile (live OrcaSlicer)

**Files:**
- Create: `machine/Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT).json` (in OrcaSlicer user dir)
- Create: `machine/Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT).info` (in OrcaSlicer user dir)

Variables used throughout Tasks 2–7:
```
ORCA_USER=/mnt/c/Users/drupi/AppData/Roaming/OrcaSlicer/user/d95bb8be-f499-4d5e-ac76-5aaa5f7267cd
```

- [ ] **Step 1: Write the live machine JSON**

```bash
ORCA_USER="/mnt/c/Users/drupi/AppData/Roaming/OrcaSlicer/user/d95bb8be-f499-4d5e-ac76-5aaa5f7267cd"
```

Create `$ORCA_USER/machine/Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT).json`:

```json
{
    "bed_mesh_max": "397,415",
    "bed_mesh_min": "33,16",
    "before_layer_change_gcode": ";BEFORE_LAYER_CHANGE\nG92 E0\n;[layer_z]\n",
    "default_print_profile": "0.30mm STANDARD @Neptune3Max (0.6 CHT)",
    "deretraction_speed": [
        "40"
    ],
    "from": "User",
    "gcode_flavor": "klipper",
    "inherits": "Elegoo Neptune 3 Max 0.4 nozzle",
    "machine_end_gcode": "PRINT_END",
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
    "machine_max_speed_z": [
        "15",
        "10"
    ],
    "machine_pause_gcode": "PAUSE",
    "machine_start_gcode": "PRINT_START EXTRUDER_TEMP=[first_layer_temperature] BED_TEMP=[first_layer_bed_temperature]",
    "max_layer_height": [
        "0.45"
    ],
    "min_layer_height": [
        "0.12"
    ],
    "name": "Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT)",
    "nozzle_diameter": [
        "0.6"
    ],
    "print_host": "192.168.1.192",
    "printer_notes": "Optimized for Neptune 3 Max - 0.6mm Diamondback CHT v6 nozzle:\n- ZNP Robin Nano DW v2.2 board\n- Direct Drive extruder\n- Slice Engineering Copperhead hotend\n- Slice Engineering Diamondback CHT v6 0.6mm nozzle\n- Stock probe (offset: -28.5, 22)\n- Input Shaping (mzv @ 52.2Hz X, ei @ 40.0Hz Y)\n- Max Accel: 3000 mm/s²\n- Max flow (Copperhead + CHT v6 0.6mm): ~35mm³/s PLA | ~25mm³/s PETG",
    "printer_settings_id": "Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT)",
    "printer_structure": "i3",
    "printer_variant": "0.6",
    "retract_before_wipe": [
        "70%"
    ],
    "retract_length_toolchange": [
        "1"
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
    "support_air_filtration": "0",
    "support_chamber_temp_control": "0",
    "thumbnails": "48x48/PNG, 300x300/PNG",
    "use_firmware_retraction": "1",
    "version": "2.3.2.0",
    "wipe": [
        "0"
    ],
    "z_hop": [
        "0.2"
    ],
    "z_hop_types": [
        "Normal Lift"
    ]
}
```

- [ ] **Step 2: Write the .info sidecar**

Create `$ORCA_USER/machine/Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT).info`:

```
sync_info = 
user_id = d95bb8be-f499-4d5e-ac76-5aaa5f7267cd
setting_id = 6e568e2c-8328-48f0-b189-ffc8b6a04a3a
base_id = A1wtoqSMiR2yvdQz
updated_time = 1787717804
```

- [ ] **Step 3: Validate**

```bash
ORCA_USER="/mnt/c/Users/drupi/AppData/Roaming/OrcaSlicer/user/d95bb8be-f499-4d5e-ac76-5aaa5f7267cd"
python3 -m json.tool "$ORCA_USER/machine/Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT).json" > /dev/null && echo OK
ls "$ORCA_USER/machine/" | grep CHT
```

Expected: `OK` then two lines — the `.json` and `.info` files.

---

## Task 3: Create Standard process profile (0.30mm)

**Files:**
- Create: `orca_profiles/neptune3_max/06mm_cht_standard.json`
- Create: `process/0.30mm STANDARD @Neptune3Max (0.6 CHT).json` (in OrcaSlicer user dir)
- Create: `process/0.30mm STANDARD @Neptune3Max (0.6 CHT).info` (in OrcaSlicer user dir)

- [ ] **Step 1: Write the repo process profile**

Create `/home/drupi/printfarm/orca_profiles/neptune3_max/06mm_cht_standard.json`:

```json
{
    "bottom_shell_layers": "4",
    "bottom_shell_thickness": "1.2",
    "bridge_flow": "0.90",
    "bridge_speed": "35",
    "brim_object_gap": "0.30",
    "brim_type": "no_brim",
    "default_acceleration": "2500",
    "detect_thin_wall": "1",
    "elefant_foot_compensation": "0.15",
    "enable_arc_fitting": "1",
    "enable_overhang_bridge_fan": "1",
    "fan_max_speed": "100",
    "fan_min_speed": "20",
    "from": "User",
    "gap_infill_speed": "180",
    "infill_jerk": "6",
    "inherits": "",
    "initial_layer_acceleration": "500",
    "initial_layer_infill_speed": "40",
    "initial_layer_jerk": "5",
    "initial_layer_line_width": "0.72",
    "initial_layer_print_height": "0.30",
    "initial_layer_speed": "22",
    "inner_wall_acceleration": "2500",
    "inner_wall_jerk": "6",
    "inner_wall_line_width": "0.68",
    "internal_bridge_speed": "150%",
    "internal_solid_infill_line_width": "0.72",
    "internal_solid_infill_pattern": "rectilinear",
    "internal_solid_infill_speed": "200",
    "is_custom_defined": "0",
    "layer_height": "0.30",
    "line_width": "0.68",
    "name": "0.30mm STANDARD @Neptune3Max (0.6 CHT)",
    "only_one_wall_top": "0",
    "outer_wall_acceleration": "2000",
    "outer_wall_jerk": "5",
    "outer_wall_line_width": "0.65",
    "outer_wall_speed": "100",
    "overhang_1_4_speed": "0",
    "overhang_2_4_speed": "55",
    "overhang_3_4_speed": "35",
    "overhang_4_4_speed": "18",
    "precise_outer_wall": "0",
    "precise_z_height": "1",
    "print_settings_id": "0.30mm STANDARD @Neptune3Max (0.6 CHT)",
    "reduce_crossing_wall": "1",
    "seam_position": "aligned",
    "seam_slope_conditional": "1",
    "seam_slope_entire_loop": "1",
    "seam_slope_inner_walls": "1",
    "seam_slope_type": "external",
    "skirt_loops": "2",
    "slow_down_for_layer_cooling": "1",
    "sparse_infill_density": "15%",
    "sparse_infill_line_width": "0.75",
    "sparse_infill_pattern": "gyroid",
    "sparse_infill_speed": "260",
    "support_bottom_z_distance": "0.25",
    "support_line_width": "0.55",
    "support_on_build_plate_only": "0",
    "support_speed": "150",
    "support_style": "organic",
    "support_top_z_distance": "0.30",
    "support_type": "tree(auto)",
    "top_shell_layers": "4",
    "top_shell_thickness": "1.2",
    "top_surface_acceleration": "2000",
    "top_surface_jerk": "5",
    "top_surface_line_width": "0.65",
    "top_surface_pattern": "monotonicline",
    "top_surface_speed": "80",
    "travel_acceleration": "3000",
    "travel_jerk": "7",
    "travel_speed": "300",
    "version": "2.3.2.0",
    "wall_loops": "3",
    "wall_sequence": "inner-outer-inner wall",
    "wipe_before_external_loop": "1",
    "wipe_on_loops": "1"
}
```

- [ ] **Step 2: Validate and spot-check**

```bash
python3 -m json.tool /home/drupi/printfarm/orca_profiles/neptune3_max/06mm_cht_standard.json > /dev/null && echo OK

python3 -c "
import json
d = json.load(open('/home/drupi/printfarm/orca_profiles/neptune3_max/06mm_cht_standard.json'))
assert d['name'] == '0.30mm STANDARD @Neptune3Max (0.6 CHT)'
assert d['outer_wall_speed'] == '100'
assert d['sparse_infill_speed'] == '260'
assert d['internal_solid_infill_speed'] == '200'
assert d['top_surface_speed'] == '80'
assert d['gap_infill_speed'] == '180'
print('All assertions passed')
"
```

Expected: `OK` then `All assertions passed`

- [ ] **Step 3: Copy to OrcaSlicer and write .info**

```bash
ORCA_USER="/mnt/c/Users/drupi/AppData/Roaming/OrcaSlicer/user/d95bb8be-f499-4d5e-ac76-5aaa5f7267cd"
cp /home/drupi/printfarm/orca_profiles/neptune3_max/06mm_cht_standard.json \
   "$ORCA_USER/process/0.30mm STANDARD @Neptune3Max (0.6 CHT).json"
```

Create `$ORCA_USER/process/0.30mm STANDARD @Neptune3Max (0.6 CHT).info`:

```
sync_info = 
user_id = d95bb8be-f499-4d5e-ac76-5aaa5f7267cd
setting_id = cf6ba39a-8f67-49d9-8f7e-71c4ad4ed0e0
base_id = kEY8hF7s5lME6fSa
updated_time = 1787717804
```

- [ ] **Step 4: Verify OrcaSlicer copy**

```bash
ORCA_USER="/mnt/c/Users/drupi/AppData/Roaming/OrcaSlicer/user/d95bb8be-f499-4d5e-ac76-5aaa5f7267cd"
python3 -m json.tool "$ORCA_USER/process/0.30mm STANDARD @Neptune3Max (0.6 CHT).json" > /dev/null && echo OK
ls "$ORCA_USER/process/" | grep "CHT"
```

Expected: `OK` then the two new `.json` and `.info` files listed.

---

## Task 4: Create Fine process profile (0.24mm)

**Files:**
- Create: `orca_profiles/neptune3_max/06mm_cht_fine.json`
- Create: `process/0.24mm FINE @Neptune3Max (0.6 CHT).json` (in OrcaSlicer user dir)
- Create: `process/0.24mm FINE @Neptune3Max (0.6 CHT).info` (in OrcaSlicer user dir)

- [ ] **Step 1: Write the repo process profile**

Create `/home/drupi/printfarm/orca_profiles/neptune3_max/06mm_cht_fine.json`:

```json
{
    "bottom_shell_layers": "5",
    "bottom_shell_thickness": "1.2",
    "bridge_flow": "0.90",
    "bridge_speed": "35",
    "brim_object_gap": "0.30",
    "brim_type": "no_brim",
    "default_acceleration": "2500",
    "detect_thin_wall": "1",
    "elefant_foot_compensation": "0.15",
    "enable_arc_fitting": "1",
    "enable_overhang_bridge_fan": "1",
    "fan_max_speed": "100",
    "fan_min_speed": "20",
    "from": "User",
    "gap_infill_speed": "160",
    "infill_jerk": "6",
    "inherits": "",
    "initial_layer_acceleration": "500",
    "initial_layer_infill_speed": "35",
    "initial_layer_jerk": "5",
    "initial_layer_line_width": "0.72",
    "initial_layer_print_height": "0.24",
    "initial_layer_speed": "20",
    "inner_wall_acceleration": "2000",
    "inner_wall_jerk": "5",
    "inner_wall_line_width": "0.66",
    "internal_bridge_speed": "100%",
    "internal_solid_infill_line_width": "0.68",
    "internal_solid_infill_pattern": "rectilinear",
    "internal_solid_infill_speed": "170",
    "is_custom_defined": "0",
    "layer_height": "0.24",
    "line_width": "0.66",
    "name": "0.24mm FINE @Neptune3Max (0.6 CHT)",
    "only_one_wall_top": "0",
    "outer_wall_acceleration": "1500",
    "outer_wall_jerk": "5",
    "outer_wall_line_width": "0.62",
    "outer_wall_speed": "100",
    "overhang_1_4_speed": "0",
    "overhang_2_4_speed": "50",
    "overhang_3_4_speed": "30",
    "overhang_4_4_speed": "15",
    "precise_outer_wall": "1",
    "precise_z_height": "1",
    "print_settings_id": "0.24mm FINE @Neptune3Max (0.6 CHT)",
    "reduce_crossing_wall": "1",
    "seam_position": "aligned",
    "seam_slope_conditional": "1",
    "seam_slope_entire_loop": "1",
    "seam_slope_inner_walls": "1",
    "seam_slope_type": "external",
    "skirt_loops": "2",
    "slow_down_for_layer_cooling": "1",
    "sparse_infill_density": "15%",
    "sparse_infill_line_width": "0.72",
    "sparse_infill_pattern": "gyroid",
    "sparse_infill_speed": "220",
    "support_bottom_z_distance": "0.24",
    "support_line_width": "0.50",
    "support_on_build_plate_only": "0",
    "support_speed": "120",
    "support_style": "organic",
    "support_top_z_distance": "0.24",
    "support_type": "tree(auto)",
    "top_shell_layers": "5",
    "top_shell_thickness": "1.2",
    "top_surface_acceleration": "1500",
    "top_surface_jerk": "5",
    "top_surface_line_width": "0.62",
    "top_surface_pattern": "monotonicline",
    "top_surface_speed": "75",
    "travel_acceleration": "3000",
    "travel_jerk": "7",
    "travel_speed": "300",
    "version": "2.3.2.0",
    "wall_loops": "4",
    "wall_sequence": "inner-outer-inner wall",
    "wipe_before_external_loop": "1",
    "wipe_on_loops": "1"
}
```

- [ ] **Step 2: Validate and spot-check**

```bash
python3 -m json.tool /home/drupi/printfarm/orca_profiles/neptune3_max/06mm_cht_fine.json > /dev/null && echo OK

python3 -c "
import json
d = json.load(open('/home/drupi/printfarm/orca_profiles/neptune3_max/06mm_cht_fine.json'))
assert d['name'] == '0.24mm FINE @Neptune3Max (0.6 CHT)'
assert d['layer_height'] == '0.24'
assert d['outer_wall_speed'] == '100'
assert d['sparse_infill_speed'] == '220'
assert d['internal_solid_infill_speed'] == '170'
assert d['top_surface_speed'] == '75'
assert d['gap_infill_speed'] == '160'
print('All assertions passed')
"
```

Expected: `OK` then `All assertions passed`

- [ ] **Step 3: Copy to OrcaSlicer and write .info**

```bash
ORCA_USER="/mnt/c/Users/drupi/AppData/Roaming/OrcaSlicer/user/d95bb8be-f499-4d5e-ac76-5aaa5f7267cd"
cp /home/drupi/printfarm/orca_profiles/neptune3_max/06mm_cht_fine.json \
   "$ORCA_USER/process/0.24mm FINE @Neptune3Max (0.6 CHT).json"
```

Create `$ORCA_USER/process/0.24mm FINE @Neptune3Max (0.6 CHT).info`:

```
sync_info = 
user_id = d95bb8be-f499-4d5e-ac76-5aaa5f7267cd
setting_id = 3b672401-76cf-4c06-b2cc-e9e77926221c
base_id = wmtgfwzyJlO7rIMA
updated_time = 1787717804
```

- [ ] **Step 4: Verify**

```bash
ORCA_USER="/mnt/c/Users/drupi/AppData/Roaming/OrcaSlicer/user/d95bb8be-f499-4d5e-ac76-5aaa5f7267cd"
python3 -m json.tool "$ORCA_USER/process/0.24mm FINE @Neptune3Max (0.6 CHT).json" > /dev/null && echo OK
```

Expected: `OK`

---

## Task 5: Create Draft process profile (0.40mm)

**Files:**
- Create: `orca_profiles/neptune3_max/06mm_cht_draft.json`
- Create: `process/0.40mm DRAFT @Neptune3Max (0.6 CHT).json` (in OrcaSlicer user dir)
- Create: `process/0.40mm DRAFT @Neptune3Max (0.6 CHT).info` (in OrcaSlicer user dir)

- [ ] **Step 1: Write the repo process profile**

Create `/home/drupi/printfarm/orca_profiles/neptune3_max/06mm_cht_draft.json`:

```json
{
    "bottom_shell_layers": "3",
    "bottom_shell_thickness": "1.2",
    "bridge_flow": "0.88",
    "bridge_speed": "30",
    "brim_object_gap": "0.30",
    "brim_type": "no_brim",
    "default_acceleration": "2500",
    "detect_thin_wall": "1",
    "elefant_foot_compensation": "0.15",
    "enable_arc_fitting": "1",
    "enable_overhang_bridge_fan": "1",
    "fan_max_speed": "100",
    "fan_min_speed": "20",
    "from": "User",
    "gap_infill_speed": "150",
    "infill_jerk": "6",
    "inherits": "",
    "initial_layer_acceleration": "500",
    "initial_layer_infill_speed": "40",
    "initial_layer_jerk": "5",
    "initial_layer_line_width": "0.75",
    "initial_layer_print_height": "0.30",
    "initial_layer_speed": "20",
    "inner_wall_acceleration": "2500",
    "inner_wall_jerk": "6",
    "inner_wall_line_width": "0.74",
    "internal_bridge_speed": "200%",
    "internal_solid_infill_line_width": "0.80",
    "internal_solid_infill_pattern": "rectilinear",
    "internal_solid_infill_speed": "180",
    "is_custom_defined": "0",
    "layer_height": "0.40",
    "line_width": "0.72",
    "name": "0.40mm DRAFT @Neptune3Max (0.6 CHT)",
    "only_one_wall_top": "0",
    "outer_wall_acceleration": "2000",
    "outer_wall_jerk": "5",
    "outer_wall_line_width": "0.68",
    "outer_wall_speed": "85",
    "overhang_1_4_speed": "0",
    "overhang_2_4_speed": "45",
    "overhang_3_4_speed": "28",
    "overhang_4_4_speed": "14",
    "precise_outer_wall": "0",
    "precise_z_height": "0",
    "print_settings_id": "0.40mm DRAFT @Neptune3Max (0.6 CHT)",
    "reduce_crossing_wall": "1",
    "seam_position": "aligned",
    "seam_slope_conditional": "1",
    "seam_slope_entire_loop": "0",
    "seam_slope_inner_walls": "0",
    "seam_slope_type": "external",
    "skirt_loops": "1",
    "slow_down_for_layer_cooling": "1",
    "sparse_infill_density": "15%",
    "sparse_infill_line_width": "0.80",
    "sparse_infill_pattern": "gyroid",
    "sparse_infill_speed": "230",
    "support_bottom_z_distance": "0.25",
    "support_line_width": "0.60",
    "support_on_build_plate_only": "0",
    "support_speed": "150",
    "support_style": "organic",
    "support_top_z_distance": "0.40",
    "support_type": "tree(auto)",
    "top_shell_layers": "3",
    "top_shell_thickness": "1.2",
    "top_surface_acceleration": "2000",
    "top_surface_jerk": "5",
    "top_surface_line_width": "0.68",
    "top_surface_pattern": "monotonicline",
    "top_surface_speed": "65",
    "travel_acceleration": "3000",
    "travel_jerk": "7",
    "travel_speed": "300",
    "version": "2.3.2.0",
    "wall_loops": "3",
    "wall_sequence": "inner-outer-inner wall",
    "wipe_before_external_loop": "1",
    "wipe_on_loops": "1"
}
```

- [ ] **Step 2: Validate and spot-check**

```bash
python3 -m json.tool /home/drupi/printfarm/orca_profiles/neptune3_max/06mm_cht_draft.json > /dev/null && echo OK

python3 -c "
import json
d = json.load(open('/home/drupi/printfarm/orca_profiles/neptune3_max/06mm_cht_draft.json'))
assert d['name'] == '0.40mm DRAFT @Neptune3Max (0.6 CHT)'
assert d['layer_height'] == '0.40'
assert d['outer_wall_speed'] == '85'
assert d['sparse_infill_speed'] == '230'
assert d['internal_solid_infill_speed'] == '180'
assert d['top_surface_speed'] == '65'
assert d['gap_infill_speed'] == '150'
print('All assertions passed')
"
```

Expected: `OK` then `All assertions passed`

- [ ] **Step 3: Copy to OrcaSlicer and write .info**

```bash
ORCA_USER="/mnt/c/Users/drupi/AppData/Roaming/OrcaSlicer/user/d95bb8be-f499-4d5e-ac76-5aaa5f7267cd"
cp /home/drupi/printfarm/orca_profiles/neptune3_max/06mm_cht_draft.json \
   "$ORCA_USER/process/0.40mm DRAFT @Neptune3Max (0.6 CHT).json"
```

Create `$ORCA_USER/process/0.40mm DRAFT @Neptune3Max (0.6 CHT).info`:

```
sync_info = 
user_id = d95bb8be-f499-4d5e-ac76-5aaa5f7267cd
setting_id = c832a6db-be74-4149-8e54-3024903eea06
base_id = XzvPDiuLRwxVi36B
updated_time = 1787717804
```

- [ ] **Step 4: Verify**

```bash
ORCA_USER="/mnt/c/Users/drupi/AppData/Roaming/OrcaSlicer/user/d95bb8be-f499-4d5e-ac76-5aaa5f7267cd"
python3 -m json.tool "$ORCA_USER/process/0.40mm DRAFT @Neptune3Max (0.6 CHT).json" > /dev/null && echo OK
```

Expected: `OK`

---

## Task 6: Create PLA filament profile

**Files:**
- Create: `orca_profiles/neptune3_max/filament_pla_cht.json`
- Create: `filament/PLA @Neptune3Max CHT.json` (in OrcaSlicer user dir)
- Create: `filament/PLA @Neptune3Max CHT.info` (in OrcaSlicer user dir)

- [ ] **Step 1: Write the repo filament profile**

Create `/home/drupi/printfarm/orca_profiles/neptune3_max/filament_pla_cht.json`:

```json
{
    "compatible_printers": [
        "Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT)"
    ],
    "enable_pressure_advance": [
        "1"
    ],
    "fan_min_speed": [
        "60"
    ],
    "filament_max_volumetric_speed": [
        "35"
    ],
    "filament_settings_id": [
        "PLA @Neptune3Max CHT"
    ],
    "from": "User",
    "hot_plate_temp": [
        "65"
    ],
    "hot_plate_temp_initial_layer": [
        "65"
    ],
    "inherits": "Generic PLA @Elegoo",
    "name": "PLA @Neptune3Max CHT",
    "nozzle_temperature": [
        "215"
    ],
    "nozzle_temperature_initial_layer": [
        "215"
    ],
    "slow_down_layer_time": [
        "0"
    ],
    "version": "2.3.2.0"
}
```

- [ ] **Step 2: Validate and spot-check**

```bash
python3 -m json.tool /home/drupi/printfarm/orca_profiles/neptune3_max/filament_pla_cht.json > /dev/null && echo OK

python3 -c "
import json
d = json.load(open('/home/drupi/printfarm/orca_profiles/neptune3_max/filament_pla_cht.json'))
assert d['name'] == 'PLA @Neptune3Max CHT'
assert d['filament_max_volumetric_speed'] == ['35']
assert d['compatible_printers'] == ['Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT)']
assert d['nozzle_temperature'] == ['215']
assert d['hot_plate_temp'] == ['65']
print('All assertions passed')
"
```

Expected: `OK` then `All assertions passed`

- [ ] **Step 3: Copy to OrcaSlicer and write .info**

```bash
ORCA_USER="/mnt/c/Users/drupi/AppData/Roaming/OrcaSlicer/user/d95bb8be-f499-4d5e-ac76-5aaa5f7267cd"
cp /home/drupi/printfarm/orca_profiles/neptune3_max/filament_pla_cht.json \
   "$ORCA_USER/filament/PLA @Neptune3Max CHT.json"
```

Create `$ORCA_USER/filament/PLA @Neptune3Max CHT.info`:

```
sync_info = 
user_id = d95bb8be-f499-4d5e-ac76-5aaa5f7267cd
setting_id = 392a3b91-aed7-4b51-ac8c-5509473f2e73
base_id = qu5MQ9JwQmQPHQ7y
updated_time = 1787717804
```

- [ ] **Step 4: Verify**

```bash
ORCA_USER="/mnt/c/Users/drupi/AppData/Roaming/OrcaSlicer/user/d95bb8be-f499-4d5e-ac76-5aaa5f7267cd"
python3 -m json.tool "$ORCA_USER/filament/PLA @Neptune3Max CHT.json" > /dev/null && echo OK
ls "$ORCA_USER/filament/" | grep CHT
```

Expected: `OK` then `PLA @Neptune3Max CHT.info` and `PLA @Neptune3Max CHT.json`

---

## Task 7: Create PETG filament profile

**Files:**
- Create: `orca_profiles/neptune3_max/filament_petg_cht.json`
- Create: `filament/PETG @Neptune3Max CHT.json` (in OrcaSlicer user dir)
- Create: `filament/PETG @Neptune3Max CHT.info` (in OrcaSlicer user dir)

- [ ] **Step 1: Write the repo filament profile**

Create `/home/drupi/printfarm/orca_profiles/neptune3_max/filament_petg_cht.json`:

```json
{
    "compatible_printers": [
        "Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT)"
    ],
    "enable_pressure_advance": [
        "1"
    ],
    "fan_min_speed": [
        "20"
    ],
    "filament_max_volumetric_speed": [
        "25"
    ],
    "filament_settings_id": [
        "PETG @Neptune3Max CHT"
    ],
    "from": "User",
    "hot_plate_temp": [
        "80"
    ],
    "hot_plate_temp_initial_layer": [
        "80"
    ],
    "inherits": "Generic PETG @Elegoo",
    "name": "PETG @Neptune3Max CHT",
    "nozzle_temperature": [
        "240"
    ],
    "nozzle_temperature_initial_layer": [
        "240"
    ],
    "slow_down_layer_time": [
        "0"
    ],
    "version": "2.3.2.0"
}
```

- [ ] **Step 2: Validate and spot-check**

```bash
python3 -m json.tool /home/drupi/printfarm/orca_profiles/neptune3_max/filament_petg_cht.json > /dev/null && echo OK

python3 -c "
import json
d = json.load(open('/home/drupi/printfarm/orca_profiles/neptune3_max/filament_petg_cht.json'))
assert d['name'] == 'PETG @Neptune3Max CHT'
assert d['filament_max_volumetric_speed'] == ['25']
assert d['compatible_printers'] == ['Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT)']
assert d['nozzle_temperature'] == ['240']
assert d['hot_plate_temp'] == ['80']
assert d['fan_min_speed'] == ['20']
print('All assertions passed')
"
```

Expected: `OK` then `All assertions passed`

- [ ] **Step 3: Copy to OrcaSlicer and write .info**

```bash
ORCA_USER="/mnt/c/Users/drupi/AppData/Roaming/OrcaSlicer/user/d95bb8be-f499-4d5e-ac76-5aaa5f7267cd"
cp /home/drupi/printfarm/orca_profiles/neptune3_max/filament_petg_cht.json \
   "$ORCA_USER/filament/PETG @Neptune3Max CHT.json"
```

Create `$ORCA_USER/filament/PETG @Neptune3Max CHT.info`:

```
sync_info = 
user_id = d95bb8be-f499-4d5e-ac76-5aaa5f7267cd
setting_id = c6c0c6bf-b649-459e-a217-6c820dd84b70
base_id = xEVxvCcj9LN3aw5B
updated_time = 1787717804
```

- [ ] **Step 4: Verify**

```bash
ORCA_USER="/mnt/c/Users/drupi/AppData/Roaming/OrcaSlicer/user/d95bb8be-f499-4d5e-ac76-5aaa5f7267cd"
python3 -m json.tool "$ORCA_USER/filament/PETG @Neptune3Max CHT.json" > /dev/null && echo OK
ls "$ORCA_USER/filament/" | grep CHT
```

Expected: `OK` then both CHT filament files listed.

---

## Task 8: Commit to repo

- [ ] **Step 1: Stage all new repo files**

```bash
cd /home/drupi/printfarm
git add orca_profiles/neptune3_max/printer_06mm_cht.json \
        orca_profiles/neptune3_max/06mm_cht_standard.json \
        orca_profiles/neptune3_max/06mm_cht_fine.json \
        orca_profiles/neptune3_max/06mm_cht_draft.json \
        orca_profiles/neptune3_max/filament_pla_cht.json \
        orca_profiles/neptune3_max/filament_petg_cht.json
git status
```

Expected: 6 new files staged under `orca_profiles/neptune3_max/`.

- [ ] **Step 2: Commit**

```bash
cd /home/drupi/printfarm
git commit -m "$(cat <<'EOF'
feat: Neptune 3 Max — add 0.6mm Diamondback CHT v6 profile set

Adds parallel OrcaSlicer profile set for the Slice Engineering
Diamondback CHT v6 nozzle: printer profile, fine/standard/draft
process profiles, and PLA + PETG filament profiles.

CHT filament caps: 35mm³/s PLA, 25mm³/s PETG (vs 17mm³/s standard).
Process speeds tuned to use the higher flow headroom.

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
EOF
)"
```

- [ ] **Step 3: Confirm**

```bash
cd /home/drupi/printfarm
git log --oneline -3
git show --stat HEAD
```

Expected: The new commit at the top, listing all 6 new files.
