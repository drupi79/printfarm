# Ender 7 OrcaSlicer Profile Optimization — Design Document

**Date:** 2026-02-27
**Status:** Design approved, pending implementation

---

## Overview

Update and expand the Ender 7 OrcaSlicer profiles. Fix correctness issues in the printer profile, restructure process profiles into quality/balanced/speed tiers, and add PLA and PETG filament profiles.

**Scope:** `orca_profiles/ender7/`

---

## Section 1: Printer Profile Fixes

File: `printer_profile.json`

| Field | Current | New | Reason |
|---|---|---|---|
| `printer_structure` | `"undefine"` | `"corexy"` | Correct kinematics — affects seam placement and travel logic |
| `use_firmware_retraction` | `"0"` | `"1"` | Delegate to Klipper's calibrated `[firmware_retraction]` (0.6mm, 40/35mm/s) |
| `printable_area` | `5x5→245x245` | `5x5→255x255` | Bed is 260×260; current value wastes 15mm per axis |
| `machine_max_speed_z` | `"10"` | `"20"` | Match Klipper `max_z_velocity: 20` |

When `use_firmware_retraction: "1"`, OrcaSlicer emits G10/G11 and Klipper applies its own values. The `retraction_length`, `retraction_speed`, and `deretraction_speed` fields in the profile are kept but ignored by Klipper.

---

## Section 2: Process Profiles

Three profiles replacing the current two. All inherit from `0.20mm Standard @MyKlipper`.

| File | Renamed From | Outer/Inner/Top/Travel Accel | Walls | Use Case |
|---|---|---|---|---|
| `quality.json` | `standard_conservative.json` | 1800 / 2000 / 1800 / 2000 | 3 | Detailed parts, overhangs, best surface finish |
| `balanced.json` | `standard.json` | 3250 / 3250 / 3250 / 4400 | 3 | Everyday printing — good quality at reasonable speed |
| `speed.json` | new | 4000 / 4400 / 3500 / 4400 | 2 | Functional parts, drafts — time over finish |

`quality.json` retains tree support, overhang speeds, and 5-shell top/bottom from current `standard_conservative.json`.

`speed.json` uses 4/3 top/bottom shells and max accels within input shaper limits (MZV@63.2Hz X, 3hump_ei@82.2Hz Y, max_accel 4400).

Old files (`standard.json`, `standard_conservative.json`) are deleted after new profiles are created.

---

## Section 3: Filament Profiles

Two new profiles. Existing `filament_numakers_pla.json` kept unchanged.

### `filament_generic_pla.json`

- Inherits: `Generic PLA @System`
- Compatible: `MyKlipper 0.4 nozzle`
- Nozzle: 220°C (220°C first layer)
- Bed: 60°C
- Pressure advance: 0.04
- Max volumetric speed: 18 mm³/s

### `filament_generic_petg.json`

- Inherits: `Generic PETG @System`
- Compatible: `MyKlipper 0.4 nozzle`
- Nozzle: 240°C (245°C first layer)
- Bed: 75°C (matches PREHEAT_PETG macro)
- Pressure advance: 0.05
- Max volumetric speed: 12 mm³/s
