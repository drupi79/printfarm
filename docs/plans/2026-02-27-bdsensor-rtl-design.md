# BDsensor Real-Time Leveling (RTL) Design

**Date:** 2026-02-27
**Printers:** Wanhao D6 (BDsensor v1.2), Ender 7 (BDsensor v1.2b)

## Goal

Enable BDsensor Real-Time Leveling on both printers in hybrid mode: the existing adaptive bed mesh
(KAMP) runs at print start as usual, and RTL supplements it by continuously correcting Z in real-time
during the print.

## Mode: Hybrid (Mesh + RTL)

RTL does not replace the pre-print bed mesh scan. The adaptive mesh runs at startup as always, and
RTL applies additional real-time Z correction throughout the print. This gives the best of both:
coarse bed shape correction from the mesh, fine real-time correction from RTL.

RTL is always on — no toggle macros.

## Changes

### Ender 7 (`klipper/printers/ender7/printer.cfg`)

1. **`[printer]` — `max_accel`: 6500 → 4400**
   - Brings Ender 7 in line with the rest of the farm (V2 twins, Ender 3 Pro all use 4400)
   - The existing X input shaper value (106Hz) is flagged as suspicious/unverified; running at 6500
     against an unverified shaper calibration is risky
   - First-layer accel cap (SET_VELOCITY_LIMIT ACCEL=500 in PRINT_START) is unaffected

2. **`[BDsensor]` — `rt_sample_time: 19`** — already present, no change to value
   - Add inline comment clarifying it enables RTL

### Wanhao D6 (`klipper/printers/wanhao_d6/printer.cfg`)

1. **`[BDsensor]` — add `rt_sample_time: 25`**
   - 25ms chosen over Ender 7's 19ms because the D6 is a slower Cartesian printer (max_accel 3900
     vs Ender 7's CoreXY at 4400). Less frequent sampling is sufficient to keep up with motion.
   - RTL activates automatically from this parameter — no PRINT_START changes needed

### Shared `start.cfg` — no changes

RTL activates automatically from the sensor config. The adaptive mesh + RTL hybrid requires no
PRINT_START coordination. The `BED_MESH_CALIBRATE` macro and `PRINT_START` flow are unchanged.

## rt_sample_time Rationale

| Printer   | Sensor  | max_accel | rt_sample_time |
|-----------|---------|-----------|----------------|
| Ender 7   | v1.2b   | 4400      | 19ms           |
| Wanhao D6 | v1.2    | 3900      | 25ms           |

Lower `rt_sample_time` = more frequent Z sampling = better tracks fast toolhead motion.
Higher value is appropriate for slower printers; there is no benefit to sampling faster than the
toolhead can meaningfully change position relative to bed surface.
