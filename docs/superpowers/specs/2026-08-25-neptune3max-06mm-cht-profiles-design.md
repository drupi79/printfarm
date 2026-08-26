# Neptune 3 Max — 0.6mm Diamondback CHT v6 OrcaSlicer Profile Set

**Date:** 2026-08-25  
**Scope:** New parallel profile set for the Slice Engineering Diamondback CHT v6 nozzle on the Neptune 3 Max Copperhead hotend. Does not replace or modify the existing 0.6mm Diamondback (non-CHT) profiles.

---

## Context

The Neptune 3 Max Copperhead hotend now has a 0.6mm Diamondback CHT v6 nozzle installed. The CHT (Continuous High-flow Technology) tri-channel internal geometry roughly doubles volumetric flow capacity vs. a standard nozzle of the same diameter. The existing 0.6mm profile set documents `~27mm³/s PLA / ~21mm³/s PETG` for the standard Diamondback; the CHT raises this to `~35mm³/s PLA / ~25mm³/s PETG` with the Copperhead.

The existing 0.6mm profiles have print speeds already set above the volumetric limit of the standard nozzle (e.g., infill at 200mm/s but effective speed capped ~75mm/s by the 17mm³/s filament profile). The CHT profiles let those speeds actually run by raising the cap and tuning process speeds upward accordingly.

---

## Files to Create

### Repo (`orca_profiles/neptune3_max/`)

| Filename | OrcaSlicer name |
|---|---|
| `printer_06mm_cht.json` | Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT) |
| `06mm_cht_fine.json` | 0.24mm FINE @Neptune3Max (0.6 CHT) |
| `06mm_cht_standard.json` | 0.30mm STANDARD @Neptune3Max (0.6 CHT) |
| `06mm_cht_draft.json` | 0.40mm DRAFT @Neptune3Max (0.6 CHT) |
| `filament_pla_cht.json` | PLA @Neptune3Max CHT |
| `filament_petg_cht.json` | PETG @Neptune3Max CHT |

All 6 files also go into the live OrcaSlicer user directory with matching `.info` sidecar files.

---

## Printer Profile (`printer_06mm_cht.json`)

Inherits: `Elegoo Neptune 3 Max 0.4 nozzle` (same as existing 0.6mm profile)

Key fields that differ from the existing 0.6mm profile:

| Field | Value |
|---|---|
| `name` | Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT) |
| `printer_settings_id` | Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT) |
| `default_print_profile` | 0.30mm STANDARD @Neptune3Max (0.6 CHT) |
| `printer_notes` | Updated — CHT nozzle, ~35mm³/s PLA / ~25mm³/s PETG |

All machine limits (accel, jerk, speed, bed mesh, retraction, z-hop, etc.) carry over unchanged from `printer_06mm.json`.

---

## Process Profiles

All three inherit nothing (`"inherits": ""`). Full field sets, matching the existing non-CHT equivalents except for the speed deltas below.

### Speed Changes vs. Existing 0.6mm Profiles

| Setting | Fine (0.24mm) | Standard (0.30mm) | Draft (0.40mm) |
|---|---|---|---|
| `outer_wall_speed` | 90 → **100** | 85 → **100** | 70 → **85** |
| `internal_solid_infill_speed` | 130 → **170** | 160 → **200** | 140 → **180** |
| `sparse_infill_speed` | 160 → **220** | 200 → **260** | 170 → **230** |
| `top_surface_speed` | 60 → **75** | 65 → **80** | 55 → **65** |
| `gap_infill_speed` | 120 → **160** | 150 → **180** | 120 → **150** |
| `support_speed` | 100 → **120** | 120 → **150** | 120 → **150** |

All other settings (layer height, line widths, accelerations, shell counts, seam, bridge flow, overhang speeds, etc.) are identical to the corresponding non-CHT profile.

---

## Filament Profiles

### PLA @Neptune3Max CHT

| Field | Value |
|---|---|
| `inherits` | Generic PLA @Elegoo |
| `filament_max_volumetric_speed` | 35 mm³/s |
| `nozzle_temperature` | 215°C |
| `nozzle_temperature_initial_layer` | 215°C |
| `hot_plate_temp` | 65°C |
| `hot_plate_temp_initial_layer` | 65°C |
| `fan_min_speed` | 60% |
| `enable_pressure_advance` | 1 |
| `slow_down_layer_time` | 0 |
| `compatible_printers` | ["Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT)"] |

### PETG @Neptune3Max CHT

| Field | Value |
|---|---|
| `inherits` | Generic PETG @Elegoo |
| `filament_max_volumetric_speed` | 25 mm³/s |
| `nozzle_temperature` | 240°C |
| `nozzle_temperature_initial_layer` | 240°C |
| `hot_plate_temp` | 80°C |
| `hot_plate_temp_initial_layer` | 80°C |
| `fan_min_speed` | 20% |
| `enable_pressure_advance` | 1 |
| `slow_down_layer_time` | 0 |
| `compatible_printers` | ["Elegoo Neptune 3 Max - Klipper OPTIMIZED (0.6mm CHT)"] |

---

## OrcaSlicer `.info` Sidecar Files

Each JSON in the live OrcaSlicer directory gets a paired `.info` file with:
- `sync_info` = (empty)
- `user_id` = d95bb8be-f499-4d5e-ac76-5aaa5f7267cd
- `setting_id` = freshly generated UUID v4
- `base_id` = 16-char random alphanumeric
- `updated_time` = current Unix timestamp

---

## Out of Scope

- Modifying existing 0.6mm profiles
- ASA or other material filament profiles (add later after tuning)
- Pressure advance value (user must calibrate after install — PA is enabled but uses inherited default)
