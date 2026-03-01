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
- Probe: BLTouch or CR Touch (confirm and update which unit is installed)
- Bed leveling: Bilinear ABL (G29 full probe each print)

## Calibration (fill in after tuning)
Apply values via EEPROM: M92 (steps), M851 (probe offset), then M500 to save.
- E-steps: TBD (calibrate for BMG — set via `M92 E<value>`, save with M500)
- BLTouch X offset: TBD
- BLTouch Y offset: TBD
- Z offset: TBD
