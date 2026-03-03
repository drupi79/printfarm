# Ender 3 V2 Marlin — Hardware Notes

## Firmware
- MRisCoC Professional Firmware (Marlin 2.x) — stock build
- Source: https://github.com/mriscoc/Ender3V2S1 (Ender3V2 + BLTouch variant)
- No custom Configuration.h — using pre-built .bin
- Note: stock .bin E-steps are configured for the stock Bowden extruder — EEPROM must be updated for BMG before first print

## Hardware
- Board: Creality 4.2.2 (TMC2208 standalone, no UART/sensorless homing)
- Kinematics: Cartesian i3
- Physical bed: 220×220mm; build height: 250mm
- Extruder: BMG clone, direct drive, 57:11 gear ratio
- Hotend: TZ E3 2.0
- Probe: BLTouch (firmware: Ender3V2-422-BLTUBL-MPC-20260106.bin)
- Bed leveling: UBL (Unified Bed Leveling) — mesh saved to slot 0, loaded each print via G29 L0

## UBL Setup (one-time, run via terminal or LCD)
```
G28       ; home
G29 P1    ; probe the mesh (automatic phase 1)
G29 P3    ; fill unprobeable points
G29 S0    ; save mesh to slot 0
M500      ; save to EEPROM
```
Start gcode then loads the saved mesh each print: `G29 L0` + `M420 S1`

## Calibration (fill in after tuning)
Apply values via EEPROM: M92 (steps), M851 (probe offset), then M500 to save.
- E-steps: TBD (calibrate for BMG — set via `M92 E<value>`, save with M500)
- BLTouch X offset: TBD
- BLTouch Y offset: TBD
- Z offset: TBD
