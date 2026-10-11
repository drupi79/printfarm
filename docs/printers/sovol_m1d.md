# Sovol M1D — preliminary notes

**Status:** Kickstarter-backed as an early backer: **Essential edition** (open frame, no chamber heater). Sovol has committed to selling the enclosure separately later; a DIY enclosure is another option. Not in the farm yet. Sovol's store estimates shipping late December 2026 to early January 2027.
**Researched:** 2026-10-11, from pre-release manufacturer and press info. Everything here is unverified until the printer arrives, and specs may change before shipping.

## Specs (manufacturer-stated)

| Item | Value |
|------|-------|
| Architecture | "DualX": IDEX with two independent carriages. The left carriage is a fixed head; the right carriage is a toolchanger with 6 docked heads (7 tools total). Swap time is about 5 s. |
| Kinematics | **Cartesian with IDEX.** Sovol confirmed this in a livestream (reported by the owner, 2026-10-11). A china-gadgets.de spec table that says CoreXY is wrong. |
| Build volume | Single head / right-side 6-head switching: 300×300×350 mm. All 7 heads: 240×300×350 mm. Copy: 178×300×350 ×2. Mirror: 138×300×350 ×2. |
| Speed / accel / flow | 600 mm/s, 10,000 mm/s², 30 mm³/s (Sovol PLA) |
| Hotend | All-metal, 0.4 mm hardened steel nozzle (0.6 and 0.8 optional), 300 °C. A 350 °C hotend kit is planned. |
| Bed | Textured PEI spring-steel sheet (single-sided), 100 °C. The page says this can be raised to 120 °C in config. AC bed heater, 800 W. |
| Chamber | Essential: open frame. Advanced: enclosure with active heating to 60 °C (shipped in a separate box). |
| Leveling | Eddy-current scanning probe (non-contact), mechanical Z-lift with real-time Z-offset compensation |
| Calibration | Rear camera for XY tool offsets; 2 toolhead cameras for extrusion; accelerometer for input shaping ("vibration compensation") |
| Filament system | 6-channel auto feed: runout, clog and tangle detection, power-loss recovery. 1.75 mm. |
| Cooling | 3010 hotend fan per head; 5020 part-cooling blower |
| UI | 5" touchscreen. Chamber camera with AI failure detection and timelapse. |
| Firmware | Klipper (per Slice Lab; Sovol says hardware, firmware and software will be open-sourced). No M1D repo exists under github.com/Sovol3d yet (checked 2026-10-11). |
| Mainboard / host | Not published |
| Slicer | OrcaSlicer (Sovol ships a fork: github.com/Sovol3d/Sovol-OrcaSlicer) |
| Power | 115–240 VAC; PSUs 150 W + 300 W |
| Package | Printer 682×682×461 mm, 39.3 kg; enclosure kit 592×592×215 mm, 12 kg |

## Expected fit with the farm
- **Treat it as self-contained, like the K1 Max and SparkX i7.** Toolchanging, IDEX modes, the camera XY calibration and the eddy probe will rely on Sovol's own macros, so it shouldn't use `klipper/shared/`. PRINT_START will probably have a different name and different parameters.
- Sovol eddy probes have needed Sovol's patched `sensor_ldc1612.c`; mainline Klipper had I2C issues with the SV08 eddy kit (see eddy-ng issue #105 below). **Don't move it to mainline Klipper** until the firmware has been reviewed.
- Essential is open-frame. Plan for PLA, PETG, TPU and PVA; Sovol only recommends ABS, ASA, PA and PC with the heated enclosure. For those materials, use the K1 Max (enclosed) for now.
- Orca: start from Sovol's system profile. Multi-tool profiles (7 extruders, IDEX copy/mirror) are new territory for this repo.

## On arrival checklist
- [ ] Record the firmware version, mainboard/SoC, host hostname and IP
- [ ] Confirm probe type and offsets, extruder type and rotation_distance
- [ ] Pull the stock config into `Printer_Backup/sovol_m1d/` before changing anything
- [ ] Create `klipper/printers/sovol_m1d/` (self-contained) and `orca_profiles/sovol_m1d/`
- [ ] Update the README printer table and CLAUDE.md with the real values, and remove the "preliminary" labels
- [ ] Check the Sovol3d GitHub for the open-source release

## Sources
- Sovol store page: https://www.sovol3d.com/products/sovol-m1d-3d-printer
- Slice Lab guide: https://slice-lab.com/en/guide-sovol-m1d
- CNX Software: https://www.cnx-software.com/2026/07/29/sovol-m1d-hybrid-idex-3d-printer-features-6-interchangeable-toolheads-for-multi-color-and-multi-material-prints/
- All3DP: https://all3dp.com/4/sovol-is-bringing-idex-back-with-its-upcoming-m1d-toolchanger-3d-printer/
- china-gadgets.de (wrongly lists CoreXY): https://www.china-gadgets.de/sovol-m1d-idex-toolchanger-news/
- eddy-ng issue on the Sovol eddy probe and mainline Klipper: https://github.com/vvuk/eddy-ng/issues/105
