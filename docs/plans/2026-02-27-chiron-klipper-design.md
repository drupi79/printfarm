# Anycubic Chiron Klipper Upgrade — Design Document

**Date:** 2026-02-27
**Status:** Design approved, pending implementation

---

## Overview

Full conversion of an Anycubic Chiron (dead stock mainboard) to Klipper with significant
hardware upgrades: BTT SKR 3 EZ mainboard, CAN bus toolhead, true dual-Z leveling, MGN12H
X-axis linear rail, custom HGX Lite toolhead on EVA 3 framework, and a custom 6mm MIC-6
aluminum bed with AC silicone heater.

**Build volume:** 400×400×450mm (stock Chiron)
**Bed plate dimensions:** 410×430mm (stock size retained)
**Kinematics:** Cartesian bed-slinger
**Mains voltage:** 120VAC / 60Hz

---

## Section 1: Electronics & Compute

| Component | Choice | Notes |
|---|---|---|
| Host | Raspberry Pi 5 4GB | Same spec as Ender 7 |
| Mainboard | BTT SKR 3 EZ | STM32H743, EZ driver slots |
| CAN bridge | BTT U2C v2.1 | USB-to-CAN, transparent to Klipper |
| Toolhead board | BTT EBB36 v1.2 | Stacks behind FYSETC 36mm NEMA14 motor |
| X/Y drivers | BTT EZ2209 ×2 | EZ format — not standard StepSticks |
| Z/Z1 drivers | BTT EZ2209 ×2 | Independent dual-Z |
| E driver | Unused | Extruder handled by EBB36 on toolhead |

**MCU architecture:**
```
Pi (USB) ──► BTT U2C ──► CAN bus ──► BTT EBB36 (toolhead)
Pi (USB) ──► BTT SKR 3 EZ (main MCU)
```

Klipper MCU entries: `[mcu]` (SKR 3 EZ), `[mcu EBBCan]` (EBB36). U2C has no Klipper entry.
CAN bus speed: 1Mbit/s. EBB36 UUID discovered via `canbus_query.py can0`.

**SKR 3 EZ firmware build (critical):**
- MCU: STM32H743
- Bootloader offset: **128KiB** (not 8KiB or 32KiB)
- Crystal: **25MHz** (wrong value = board appears dead after flash)
- Communication: USB (on PA11/PA12)
- Flash via SD card as `firmware.bin`

---

## Section 2: Motion System & Homing

**Kinematics:** Cartesian bed-slinger. X/Y standard steppers. Dual Z (independent).

**Homing:**
- X: Mechanical switch **frame-mounted on left gantry upright** (X min endstop)
  - Klipper: `endstop_pin: PG6` (SKR 3 EZ XSTOP pin — verify against pinout PDF)
  - Wire runs up the frame to electronics bay — no endstop wire in umbilical
  - Standard homing direction (positive_dir: false, position_endstop: 0)
  - Apollo Lander has no toolhead endstop provision; frame mount is the correct approach
- Y: Stock mechanical switch on frame → SKR 3 EZ endstop input
- Z: Stock optical endstop on frame → SKR 3 EZ endstop input
  - Uses physical endstop, NOT `probe:z_virtual_endstop`
  - BDsensor does NOT handle Z homing (`collision_homing: 0`)

**Dual Z (Z_TILT_ADJUST):**
- Two independent Z motors: SKR 3 EZ M3 (Z) and M4 (Z1)
- Left leadscrew (Z) and right leadscrew (Z1)
- `[z_tilt]` config (not `quad_gantry_level` — this is a bed-slinger)
- z_positions: approximately (-30, 215) and (440, 215) — **confirm by measuring actual
  Chiron frame leadscrew positions with calipers**
- 3 probe points for better convergence: left, center, right at mid-Y
- retry_tolerance: 0.0075mm

**BDsensor (mesh only):**
- `collision_homing: 0` — optical endstop handles Z homing
- `collision_calibrate: 0` — standard PROBE_CALIBRATE workflow
- `no_stop_probe` enabled for fast probing
- RTL: `rt_sample_time` TBD after initial tuning (reference: D6=25ms, Ender 7=19ms)
- Wired to EBB36 I2C — short run avoids cable chain noise

**Input shaper:** ADXL345 built into EBB36. `probe_points: 205, 215, 20` (bed center).

**safe_z_home:** Nozzle at bed center. Probe coordinates = nozzle + BDsensor offset (measure
after mounting). Follow probe-vs-nozzle offset math documented in existing farm configs.

---

## Section 3: Toolhead

**X-axis motion:**
- MGN12H linear rail, **500mm**, top-mounted on upper 2040 extrusion
- Rail bolts directly to 2040 top face with M3 T-nuts (no printed adapter for rail)
- Lower 2040 extrusion: passive V-slot wheel guide (single rail, not dual)
- Replaces stock V-slot wheel carriage entirely

**Toolhead framework: Apollo Lander (Squirrelf3D) — TZ-E3 2.0 edition**
Self-contained shroud with integrated dual 5015 part cooling, designed specifically for the
TZ-E3 2.0 hotend. Files are in `ApolloLander - Trianglelab - TZ-E3 2.0_2025_02_14/`.

| Part | File | Notes |
|---|---|---|
| Main body | `MainBody Universal/Trianglelab - TZ-E3 2.0 - MainBody Universal - Standard.stl` | Universal fits all non-Ender5 printers |
| Backplate | `Backplate - LinearRails - MGN12H/Orbiter Extruder/Orbiter 2.x - back motor - mgn12h - Trianglelab - TZ-E3 2.0.stl` | Back motor = motor faces rearward, cleaner umbilical routing |
| BDsensor mount | `BDSensor Apollo Mount.stl` | Side-mount bracket for BDsensor |

**Extruder:** Orbiter 2.x — self-contained planetary gearbox extruder with integrated LDO motor.
No separate motor required. Klipper config: `rotation_distance: 4.637`, no `gear_ratio` entry.

**Hotend:** TZ E3 2.0 (V6 groove mount — natively supported by Apollo Lander body).

**Cooling:**
- Heatbreak: always-on fan → EBB36 heater fan output
- Part cooling: dual 5015 blowers → EBB36 fan outputs (built into Apollo Lander body)

**Sensors on toolhead (all wired to EBB36):**
- BDsensor → EBB36 I2C header (mounted via `BDSensor Apollo Mount.stl`)
- Filament runout sensor → EBB36 endstop input
- ADXL345 → built into EBB36
- ⚠️ **X endstop NOT on toolhead** — see Section 2 (frame-mounted, wired to SKR 3 EZ)

**EBB36 mounting:** ⚠️ Open item — Apollo Lander backplate has no native EBB36 mount.
Options: printed bracket on rear of backplate, bracket clipped to MGN12H carriage, or community
remix. Resolve before printing backplate.

---

## Section 4: Bed System

**Plate:** MIC-6 cast aluminum tooling plate, 6mm thick, 410×430mm
- NOT regular 6061 (warps under thermal cycling)
- Source: SendCutSend (upload DXF with hole pattern) or Online Metals cut + local shop for drilling
- Verify Chiron mounting hole pattern before ordering — 6mm plate is thicker than stock,
  may need longer bed spring standoffs
- Kinematic or clearance-hole mounting to allow ~0.8mm thermal expansion at ABS temps

**Heater:** Keenovo custom AC silicone heater, 410×430mm, 750-1000W, 120VAC
- Adhesive-backed (bonded directly to plate)
- Integrated NTC thermistor (add this option when ordering)
- Order direct from keenovo.com — 1-3 week lead time
- This is the longest-lead item in the build — order first

**SSR:** Crydom D1D40 (40A, 120VAC) — NOT AliExpress Fotek counterfeit
- Mount to 50×50mm+ aluminum heatsink with thermal paste
- SKR 3 EZ outputs 3.3V logic — add buffer transistor on control line for reliable trigger
- Source from Digi-Key or Mouser

**Safety (non-negotiable):**
- 10A fast-blow fuse on AC mains feed, before SSR
- 130°C thermal fuse in series with heater
- Bed plate earth-grounded to chassis (ring terminal, 14AWG wire)
- 14AWG silicone high-flex wire for all AC heater leads

**Print surface:** Energetic 410×430mm textured PEI spring steel sheet + magnetic base
- Search "Anycubic Chiron PEI spring steel" on AliExpress
- ⚠️ Verify exact dimensions before ordering — 410×430 is non-standard
- Textured PEI preferred: better for PLA/PETG/ASA versatility

**Klipper bed config:**
```ini
[heater_bed]
pwm_cycle_time: 0.0166    # 60Hz / 120VAC North America
max_temp: 130
# PID values from PID_CALIBRATE — expect high Kp/Kd due to 6mm aluminum thermal mass
# Run: PID_CALIBRATE HEATER=heater_bed TARGET=60
# Expect 20-40 minute calibration run
```

---

## Section 5: Cable Management

**X-axis toolhead (umbilical — no cable chain):**
- 4-wire braided sleeve umbilical: 24V+, GND, CAN H, CAN L
- Hangs freely from fixed anchor on upper gantry frame to toolhead
- CAN's noise resistance makes long umbilical runs reliable
- Shielded cable recommended for CAN H/L pair
- Strain relief at both ends; EVA 3 has umbilical anchor built in

**Y-axis bed wiring:**
- AC heater leads: 14AWG silicone high-flex, routed with short cable sleeve along Y travel
- Bed thermistor: standard 2-wire, alongside heater leads
- Strain relief at bed carriage and frame anchor points

**Electronics bay:**
- SSR mounted away from bed radiant heat, with heatsink
- AC wiring clearly separated from DC/signal wiring
- All mains connections properly insulated and strain-relieved

---

## Section 6: Klipper Software & Config

**Config location:** `klipper/printers/chiron/printer.cfg`
Uses shared `start.cfg` and `macros.cfg` per farm convention.

**Chiron-specific PRINT_START** (added to `printer.cfg`, overrides shared version):
```
G28 → Z_TILT_ADJUST → G28 Z → BED_MESH_CALIBRATE → Smart_Park → heat extruder → LINE_PURGE
```
The `G28 Z` re-home after `Z_TILT_ADJUST` is mandatory — tilt correction moves both Z motors
and leaves Z position uncertain. Re-homing re-establishes clean Z=0 at the optical switch.

**Key config sections:**
```ini
[mcu]
serial: /dev/serial/by-id/usb-Klipper_stm32h743xx_XXXX-if00

[mcu EBBCan]
canbus_uuid: <discovered via canbus_query.py can0>

[printer]
kinematics: cartesian
max_velocity: 500
max_accel: 3000    # Conservative starting value — tune after input shaper

[stepper_z]
endstop_pin: <SKR 3 EZ ZSTOP pin>    # Physical optical endstop, NOT probe:z_virtual_endstop

[stepper_x]
endstop_pin: EBBCan:<pin>             # X endstop on toolhead via EBB36

[BDsensor]
sda_pin: EBBCan:<pin>
scl_pin: EBBCan:<pin>
collision_homing: 0
collision_calibrate: 0
no_stop_probe:
rt_sample_time: <TBD — tune after initial setup>
```

**First-boot checklist (in order):**
1. `QUERY_ENDSTOPS` — verify X, Y, Z all trigger correctly
2. `G28` with hand on emergency stop — verify all axes home safely
3. `PROBE_ACCURACY` — verify BDsensor std dev < 0.010mm
4. `Z_TILT_ADJUST` standalone — confirm both Z motors move and converge
5. `BED_MESH_CALIBRATE PROFILE=initial` — full mesh
6. `PROBE_CALIBRATE` — paper test, set z_offset
7. `SAVE_CONFIG`
8. `PID_CALIBRATE HEATER=extruder TARGET=220`
9. `PID_CALIBRATE HEATER=heater_bed TARGET=60`
10. `SAVE_CONFIG`
11. `SHAPER_CALIBRATE` — input shaper via EBB36 ADXL345
12. First print at conservative speeds

---

## Bill of Materials Summary

| Item | Recommendation | Est. Cost | Lead Time |
|---|---|---|---|
| BTT SKR 3 EZ | BTT official or AliExpress BTT store | ~$45 | Standard |
| BTT EZ2209 ×4 | BTT official | ~$10 ea | Standard |
| BTT U2C v2.1 | BTT official | ~$18 | Standard |
| BTT EBB36 v1.2 | BTT official | ~$25 | Standard |
| BDsensor (standard/VB — NOT BDsensor-M) | BIQU/BTT | ~$40 | Standard |
| MGN12H 500mm rail + block | Trianglelab or reputable AliExpress | ~$25 | Standard |
| Orbiter 2.x extruder | LDO / Trianglelab (includes integrated motor) | ~$50-60 | Standard |
| Apollo Lander toolhead prints | Print in-house (PETG/ASA) — files in repo folder | ~$3 filament | — |
| X endstop microswitch + bracket | Print bracket, standard microswitch | ~$2 | — |
| MIC-6 6mm 410×430mm plate | SendCutSend + Online Metals | ~$90-120 | 1-2 weeks |
| Keenovo AC heater 410×430mm | keenovo.com direct | ~$65-80 | **1-3 weeks** |
| Crydom D1D40 SSR | Digi-Key / Mouser | ~$35 | Standard |
| SSR heatsink + thermal paste | Amazon | ~$8 | Standard |
| 10A fuse + holder | Amazon | ~$5 | Standard |
| 130°C thermal fuse | Amazon | ~$4 | Standard |
| Energetic PEI 410×430mm | AliExpress (verify size) | ~$30 | 2-3 weeks |
| Braided umbilical sleeve + CAN cable | Amazon / AliExpress | ~$15 | Standard |
| M3 T-nuts, hardware | Amazon | ~$10 | Standard |
| **Total** | | **~$450-480** | |

> ⚠️ Order Keenovo heater first — longest lead time in the build.

---

## EVA 3 Toolhead Compatibility (Researched 2026-02-27)

EVA 3 current release: **v3.0.2** (July 2022). Main repo: https://github.com/EVA-3D/eva-main

All required modules confirmed via community Printables designs:

| Component | Module | Printables Link |
|---|---|---|
| MGN12H top carriage | Universal MGN12 Top Mount | https://www.printables.com/model/372296 |
| HGX Lite extruder top | EVA 3 HGX LITE drive mount | https://www.printables.com/model/776411 |
| EBB36 back/board mount | EBB36 + HGX Lite combined mount | https://www.printables.com/model/881992 |
| BDsensor probe mount | Adjustable BD Sensor Mount EVA 3 | https://www.printables.com/model/1116783 |
| V6/groove mount front | EVA 3 native (official release) | — |

**Key notes:**
- MGN12H is NOT natively supported by EVA 3 — use community MGN12H top (model 372296 recommended as universal C/H)
- HGX Lite tops are designed for the 36mm round NEMA14 motor — this IS the standard format; no custom collar needed
- Use model 776411 for HGX Lite V1 (standard), model 1143215 for V2.0 (oblique tooth gear version) — verify which you have
- The combined EBB36 + HGX Lite mount (881992) handles both board mounting and extruder top in one piece
- Verify each model's description targets EVA 3.0.x (not EVA 2.x) before printing
- EVA 3 has native V6/groove mount front — TZ E3 2.0 is fully supported without community module

---

## Open Items (Require Physical Measurement or Research)

1. **Z_TILT z_positions** — measure actual leadscrew X positions on your Chiron frame with calipers
2. **BDsensor x/y offset** — measure after mounting toolhead (`BDSensor Apollo Mount.stl`)
3. **EBB36 pin assignments** — verify endstop and I2C pins against EBB36 v1.2 pinout diagram
4. **Chiron bed mounting hole pattern** — measure before ordering MIC-6 plate
5. **EBB36 mounting on Apollo Lander backplate** — no native mount; use one of these Orbiter-direct mounts (verify fit behind "back motor" backplate before printing):
   - Primary: [EBB36 Mount with Cable Strain Relief for Orbiter v2.0 — djos_1475](https://www.printables.com/model/316984) — attaches to Orbiter motor body via M3×15/20mm hex standoffs, toolhead-agnostic
   - With PG7 cable gland: [EBB36 mount for Orbiter 2 and PG7 Gland, Optimized — Blargedy](https://www.printables.com/model/791804)
   - With dual fan JST splitter: [Orbiter 2/2.5 EBB36 Mount w/ Dual Fan Splitter — jinetix](https://www.printables.com/model/1211811)
   - Fallback collection: [KayosMaker CANboard_Mounts — GitHub](https://github.com/KayosMaker/CANboard_Mounts)
6. **X endstop bracket** — mount microswitch on left gantry 2040 upright:
   - Primary: [Adjustable Endstop Mount for 2040 Extrusion — evil genius](https://www.printables.com/model/447865) — adjustable position for dialing in X=0 trigger point, no screws needed to retain switch
   - Fallback: [2040 Microswitch Endstop Holder — ThrasherHT](https://www.printables.com/model/180856) — simpler fixed mount, includes STEP for modification
7. **rt_sample_time** — set after initial printing, tune based on Chiron's speed characteristics
8. **Orbiter 2.x motor orientation** — "back motor" backplate selected (motor faces rearward); confirm this works with EBB36 bracket solution
