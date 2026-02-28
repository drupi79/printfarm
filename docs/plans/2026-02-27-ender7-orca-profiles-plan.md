# Ender 7 OrcaSlicer Profile Optimization — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Fix correctness issues in the Ender 7 OrcaSlicer printer profile and build a complete set of process and filament profiles.

**Architecture:** All files live in `orca_profiles/ender7/`. Process profiles inherit from `0.20mm Standard @MyKlipper` (OrcaSlicer parent), except `print_in_place.json` which is standalone. Filament profiles inherit from OrcaSlicer system profiles. No automated tests — validate each file with `python3 -m json.tool <file>` after writing.

**Tech Stack:** OrcaSlicer JSON profile format (v2.3.1.10)

**Design doc:** `docs/plans/2026-02-27-ender7-orca-profiles-design.md`

---

### Task 1: Fix printer_profile.json

**Files:**
- Modify: `orca_profiles/ender7/printer_profile.json`

**Step 1: Make four field changes**

Change `printer_structure`:
```json
"printer_structure": "corexy",
```

Change `use_firmware_retraction`:
```json
"use_firmware_retraction": "1",
```

Change `printable_area`:
```json
"printable_area": [
    "5x5",
    "255x5",
    "255x255",
    "5x255"
],
```

Change `machine_max_speed_z` (both values):
```json
"machine_max_speed_z": [
    "20",
    "20"
],
```

**Step 2: Validate JSON**

```bash
python3 -m json.tool orca_profiles/ender7/printer_profile.json > /dev/null && echo OK
```
Expected: `OK`

**Step 3: Commit**

```bash
git add orca_profiles/ender7/printer_profile.json
git commit -m "Ender 7: fix printer profile — corexy, firmware retraction, printable area, Z speed"
```

---

### Task 2: Create quality.json

**Files:**
- Create: `orca_profiles/ender7/quality.json`
- Delete: `orca_profiles/ender7/standard_conservative.json` (after creating quality.json)

**Step 1: Write quality.json**

```json
{
    "bottom_shell_layers": "5",
    "bridge_flow": "0.95",
    "bridge_speed": "25",
    "brim_type": "no_brim",
    "default_acceleration": "2000",
    "elefant_foot_compensation": "0.1",
    "from": "User",
    "infill_jerk": "12",
    "inherits": "0.20mm Standard @MyKlipper",
    "initial_layer_infill_speed": "50",
    "initial_layer_line_width": "0.5",
    "inner_wall_acceleration": "2000",
    "inner_wall_jerk": "7",
    "inner_wall_line_width": "0.45",
    "internal_solid_infill_line_width": "0.4",
    "line_width": "0.4",
    "minimum_sparse_infill_area": "10",
    "name": "0.20mm Quality @Ender7",
    "only_one_wall_top": "1",
    "outer_wall_acceleration": "1800",
    "outer_wall_jerk": "7",
    "outer_wall_line_width": "0.4",
    "overhang_2_4_speed": "20",
    "overhang_3_4_speed": "15",
    "precise_outer_wall": "0",
    "print_settings_id": "0.20mm Quality @Ender7",
    "reduce_crossing_wall": "1",
    "sparse_infill_line_width": "0.45",
    "sparse_infill_pattern": "honeycomb",
    "support_line_width": "0.4",
    "support_on_build_plate_only": "1",
    "support_style": "organic",
    "support_top_z_distance": "0.25",
    "support_type": "tree(auto)",
    "top_shell_layers": "5",
    "top_surface_acceleration": "1800",
    "top_surface_jerk": "7",
    "top_surface_line_width": "0.4",
    "travel_acceleration": "2000",
    "travel_speed": "200",
    "version": "2.3.1.10",
    "wall_loops": "3",
    "wall_sequence": "outer wall/inner wall"
}
```

**Step 2: Validate JSON**

```bash
python3 -m json.tool orca_profiles/ender7/quality.json > /dev/null && echo OK
```
Expected: `OK`

**Step 3: Delete old file**

```bash
git rm orca_profiles/ender7/standard_conservative.json
```

**Step 4: Commit**

```bash
git add orca_profiles/ender7/quality.json
git commit -m "Ender 7: add quality process profile, remove standard_conservative"
```

---

### Task 3: Create balanced.json

**Files:**
- Create: `orca_profiles/ender7/balanced.json`
- Delete: `orca_profiles/ender7/standard.json` (after creating balanced.json)

**Step 1: Write balanced.json**

```json
{
    "bridge_flow": "0.95",
    "default_acceleration": "4400",
    "elefant_foot_compensation": "0.1",
    "from": "User",
    "infill_jerk": "12",
    "inherits": "0.20mm Standard @MyKlipper",
    "initial_layer_line_width": "0.5",
    "inner_wall_acceleration": "3250",
    "inner_wall_jerk": "7",
    "inner_wall_line_width": "0.45",
    "internal_solid_infill_line_width": "0.4",
    "line_width": "0.4",
    "minimum_sparse_infill_area": "10",
    "name": "0.20mm Balanced @Ender7",
    "only_one_wall_top": "1",
    "outer_wall_acceleration": "3250",
    "outer_wall_jerk": "7",
    "outer_wall_line_width": "0.4",
    "precise_outer_wall": "0",
    "print_settings_id": "0.20mm Balanced @Ender7",
    "reduce_crossing_wall": "1",
    "small_area_infill_flow_compensation": "1",
    "sparse_infill_line_width": "0.45",
    "support_line_width": "0.4",
    "top_surface_acceleration": "3250",
    "top_surface_jerk": "7",
    "top_surface_line_width": "0.4",
    "travel_acceleration": "4400",
    "version": "2.3.1.10",
    "wall_loops": "3",
    "wall_sequence": "outer wall/inner wall"
}
```

**Step 2: Validate JSON**

```bash
python3 -m json.tool orca_profiles/ender7/balanced.json > /dev/null && echo OK
```
Expected: `OK`

**Step 3: Delete old file**

```bash
git rm orca_profiles/ender7/standard.json
```

**Step 4: Commit**

```bash
git add orca_profiles/ender7/balanced.json
git commit -m "Ender 7: add balanced process profile, remove standard"
```

---

### Task 4: Create speed.json

**Files:**
- Create: `orca_profiles/ender7/speed.json`

**Step 1: Write speed.json**

```json
{
    "bottom_shell_layers": "3",
    "bridge_flow": "0.95",
    "default_acceleration": "4400",
    "elefant_foot_compensation": "0.1",
    "from": "User",
    "infill_jerk": "12",
    "inherits": "0.20mm Standard @MyKlipper",
    "initial_layer_line_width": "0.5",
    "inner_wall_acceleration": "4400",
    "inner_wall_jerk": "9",
    "inner_wall_line_width": "0.45",
    "internal_solid_infill_line_width": "0.45",
    "line_width": "0.45",
    "minimum_sparse_infill_area": "10",
    "name": "0.20mm Speed @Ender7",
    "only_one_wall_top": "1",
    "outer_wall_acceleration": "4000",
    "outer_wall_jerk": "9",
    "outer_wall_line_width": "0.4",
    "precise_outer_wall": "0",
    "print_settings_id": "0.20mm Speed @Ender7",
    "reduce_crossing_wall": "1",
    "sparse_infill_line_width": "0.5",
    "support_line_width": "0.4",
    "top_shell_layers": "4",
    "top_surface_acceleration": "3500",
    "top_surface_jerk": "9",
    "top_surface_line_width": "0.4",
    "travel_acceleration": "4400",
    "version": "2.3.1.10",
    "wall_loops": "2",
    "wall_sequence": "outer wall/inner wall"
}
```

**Step 2: Validate JSON**

```bash
python3 -m json.tool orca_profiles/ender7/speed.json > /dev/null && echo OK
```
Expected: `OK`

**Step 3: Commit**

```bash
git add orca_profiles/ender7/speed.json
git commit -m "Ender 7: add speed process profile"
```

---

### Task 5: Create print_in_place.json

**Files:**
- Create: `orca_profiles/ender7/print_in_place.json`

Adapted from Wanhao D6 profile with accels tuned up for CoreXY (outer: 800, inner: 1000, top: 800, travel: 2000). Standalone — inherits `""` so OrcaSlicer system defaults are the base.

**Step 1: Write print_in_place.json**

```json
{
    "bottom_shell_layers": "5",
    "bridge_density": "100%",
    "bridge_flow": "0.95",
    "bridge_speed": "25",
    "brim_object_gap": "0.22",
    "brim_type": "outer_only",
    "brim_width": "3",
    "default_acceleration": "1000",
    "detect_thin_wall": "1",
    "elefant_foot_compensation": "0.15",
    "enable_arc_fitting": "1",
    "enable_overhang_bridge_fan": "1",
    "from": "User",
    "gap_infill_speed": "50",
    "infill_jerk": "5",
    "inherits": "",
    "initial_layer_acceleration": "300",
    "initial_layer_infill_speed": "30",
    "initial_layer_jerk": "5",
    "initial_layer_line_width": "0.5",
    "initial_layer_print_height": "0.2",
    "initial_layer_speed": "20",
    "inner_wall_acceleration": "1000",
    "inner_wall_jerk": "5",
    "inner_wall_line_width": "0.42",
    "inner_wall_speed": "40",
    "internal_solid_infill_line_width": "0.42",
    "internal_solid_infill_pattern": "rectilinear",
    "internal_solid_infill_speed": "60",
    "is_custom_defined": "0",
    "layer_height": "0.20",
    "line_width": "0.42",
    "name": "0.20mm PRINT-IN-PLACE @Ender7",
    "only_one_wall_top": "0",
    "outer_wall_acceleration": "800",
    "outer_wall_jerk": "3",
    "outer_wall_line_width": "0.40",
    "outer_wall_speed": "30",
    "overhang_2_4_speed": "25",
    "overhang_3_4_speed": "15",
    "overhang_4_4_speed": "10",
    "precise_outer_wall": "1",
    "print_settings_id": "0.20mm PRINT-IN-PLACE @Ender7",
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
    "sparse_infill_speed": "70",
    "support_bottom_z_distance": "0.25",
    "support_line_width": "0.38",
    "support_on_build_plate_only": "0",
    "support_speed": "60",
    "support_style": "organic",
    "support_top_z_distance": "0.3",
    "support_type": "tree(auto)",
    "top_shell_layers": "5",
    "top_shell_thickness": "1.0",
    "top_surface_acceleration": "800",
    "top_surface_jerk": "3",
    "top_surface_line_width": "0.42",
    "top_surface_pattern": "monotoniclines",
    "top_surface_speed": "30",
    "travel_acceleration": "2000",
    "travel_jerk": "7",
    "travel_speed": "150",
    "version": "2.3.1.10",
    "wall_loops": "4",
    "wall_sequence": "inner wall/outer wall",
    "fan_max_speed": "100",
    "fan_min_speed": "30"
}
```

**Step 2: Validate JSON**

```bash
python3 -m json.tool orca_profiles/ender7/print_in_place.json > /dev/null && echo OK
```
Expected: `OK`

**Step 3: Commit**

```bash
git add orca_profiles/ender7/print_in_place.json
git commit -m "Ender 7: add print-in-place process profile"
```

---

### Task 6: Create filament_generic_pla.json

**Files:**
- Create: `orca_profiles/ender7/filament_generic_pla.json`

**Step 1: Write filament_generic_pla.json**

```json
{
    "compatible_printers": [
        "MyKlipper 0.4 nozzle"
    ],
    "enable_pressure_advance": [
        "1"
    ],
    "filament_max_volumetric_speed": [
        "18"
    ],
    "filament_settings_id": [
        "Generic PLA @Ender7"
    ],
    "filament_vendor": [
        "Generic"
    ],
    "from": "User",
    "hot_plate_temp": [
        "60"
    ],
    "hot_plate_temp_initial_layer": [
        "60"
    ],
    "inherits": "Generic PLA @System",
    "is_custom_defined": "0",
    "name": "Generic PLA @Ender7",
    "nozzle_temperature": [
        "220"
    ],
    "nozzle_temperature_initial_layer": [
        "220"
    ],
    "pressure_advance": [
        "0.04"
    ],
    "version": "2.3.1.10"
}
```

**Step 2: Validate JSON**

```bash
python3 -m json.tool orca_profiles/ender7/filament_generic_pla.json > /dev/null && echo OK
```
Expected: `OK`

**Step 3: Commit**

```bash
git add orca_profiles/ender7/filament_generic_pla.json
git commit -m "Ender 7: add generic PLA filament profile"
```

---

### Task 7: Create filament_generic_petg.json

**Files:**
- Create: `orca_profiles/ender7/filament_generic_petg.json`

**Step 1: Write filament_generic_petg.json**

```json
{
    "compatible_printers": [
        "MyKlipper 0.4 nozzle"
    ],
    "enable_pressure_advance": [
        "1"
    ],
    "filament_max_volumetric_speed": [
        "12"
    ],
    "filament_settings_id": [
        "Generic PETG @Ender7"
    ],
    "filament_vendor": [
        "Generic"
    ],
    "from": "User",
    "hot_plate_temp": [
        "75"
    ],
    "hot_plate_temp_initial_layer": [
        "75"
    ],
    "inherits": "Generic PETG @System",
    "is_custom_defined": "0",
    "name": "Generic PETG @Ender7",
    "nozzle_temperature": [
        "240"
    ],
    "nozzle_temperature_initial_layer": [
        "245"
    ],
    "pressure_advance": [
        "0.05"
    ],
    "version": "2.3.1.10"
}
```

**Step 2: Validate JSON**

```bash
python3 -m json.tool orca_profiles/ender7/filament_generic_petg.json > /dev/null && echo OK
```
Expected: `OK`

**Step 3: Commit**

```bash
git add orca_profiles/ender7/filament_generic_petg.json
git commit -m "Ender 7: add generic PETG filament profile"
```

---

### Task 8: Update CLAUDE.md and push

**Files:**
- Modify: `CLAUDE.md`

**Step 1: Update the OrcaSlicer Profiles section**

In the `## OrcaSlicer Profiles` section, find the line about the Ender 7 (it currently has no exception listed). Update the description so it's clear the Ender 7 now has the full standard set:

The standard profile set is: `printer_profile.json`, `quality.json`, `balanced.json`, `speed.json`, `print_in_place.json`, plus filament profiles. No exceptions needed for the Ender 7 — it now matches the standard layout.

Also update the Ender 7 notes to remove any references to old profile names (standard.json, standard_conservative.json).

**Step 2: Commit and push everything**

```bash
git add CLAUDE.md
git commit -m "Ender 7: update CLAUDE.md — OrcaSlicer profiles now complete"
git push
```
