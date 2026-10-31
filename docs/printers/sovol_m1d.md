# Sovol M1D — preliminary notes

**Status:** Kickstarter-backed as an early backer: **Essential edition** (open frame, no chamber heater). Sovol has committed to selling the enclosure separately later; a DIY enclosure is another option. Not in the farm yet. Sovol's store estimates shipping late December 2026 to early January 2027. That is a slip of about a month: on 2026-09-28 Sovol quoted late November to mid-December. The Kickstarter backer ship date is unconfirmed, because the backer updates need a logged-in browser.
**Researched:** 2026-10-11, from pre-release manufacturer and press info. Everything here is unverified until the printer arrives, and specs may change before shipping.

## Specs (manufacturer-stated)

| Item | Value |
|------|-------|
| Architecture | "DualX": IDEX with two independent carriages. The left carriage is a fixed head; the right carriage is a toolchanger with 6 docked heads (7 tools total). Swap time is about 5 s, which per reviewers excludes travel, wipe and prime. The lock is a patented, motor-driven, all-metal auto-grip. Keep the dock free of debris, or heads won't seat (Sovol maintenance guide). Docked heads preheat to printing temperature (sovol.uk). |
| Kinematics | **Cartesian with IDEX.** Sovol confirmed this in a livestream (reported by the owner, 2026-10-11). A china-gadgets.de spec table that says CoreXY is wrong. |
| Build volume | Single head / right-side 6-head switching: 300×300×350 mm. All 7 heads: 240×300×350 mm. Copy: 178×300×350 ×2. Mirror: 138×300×350 ×2. |
| Speed / accel / flow | 600 mm/s, 10,000 mm/s², 30 mm³/s (Sovol PLA) |
| Hotend / extruder | All-metal hotend with a ceramic AC heater, 0.4 mm hardened steel nozzle (0.6 and 0.8 optional), 300 °C. A 350 °C hotend/nozzle kit is planned. Each head has its own all-metal extruder; the fixed head is "high-flow" with hardened drive gears (sovol.uk). Gear ratio, thermistor, nozzle system and spare toolhead price are not published. |
| Bed | Textured PEI spring-steel sheet (single-sided), 100 °C. The page says this can be raised to 120 °C in config. AC bed heater, 800 W. |
| Chamber | Essential: open frame. Advanced: enclosure with active heating to 60 °C (shipped in a separate box). |
| Leveling | Eddy-current scanning probe (non-contact), mechanical Z-lift with real-time Z-offset compensation |
| Calibration | Rear camera (1280×720) scans the nozzle tips to set XY tool offsets, claimed 2.5× faster than probing; no accuracy figure is published. Two toolhead cameras (1600×1200 each) check extrusion. Accelerometer for input shaping ("vibration compensation"). Eddy sensor chip not published; Sovol's other eddy probes use the LDC1612. |
| Filament system | 6-channel auto feed: runout, clog and tangle detection, power-loss recovery. 1.75 mm. |
| Cooling | 3010 hotend fan per head; 5020 part-cooling blower |
| UI | 5" touchscreen plus a web dashboard (UI not named). Chamber camera (1280×720) with timelapse; AI failure detection is due in a later software update. |
| Firmware | Klipper-based. The 2026-07-24 press release promises to open-source the "Klipper-based firmware configurations" with documentation; the store page says "Fully Open-Source", while the FAQ says "will" be opened. Klipper version/fork unknown. No M1D repo exists under github.com/Sovol3d yet (checked 2026-10-11). |
| Mainboard / host | Not published. 32 GB eMMC and a 4010 mainboard fan (CNX), which suggests a built-in Linux host rather than a Pi. |
| Slicer | OrcaSlicer. Sovol ships a fork (github.com/Sovol3d/Sovol-OrcaSlicer, latest release v2.3.0.4 from 2025-09). No M1D profile exists in Sovol's fork or upstream Orca yet (checked 2026-10-11). Sovol's setup guide says to pick the M1D printer profile *first* (switching later resets filament and color assignments), map one filament per toolhead, then choose Multi, Copy or Mirror mode. The two heads print at the same time only in Copy and Mirror. |
| Power | 115–240 VAC; PSUs 150 W + 300 W |
| Package | Printer 682×682×461 mm, 39.3 kg; enclosure kit 592×592×215 mm, 12 kg |

## Open questions
- Mainboard, SoC and eddy sensor chip; when Sovol publishes the M1D repo and under what licence
- Spare toolhead price and nozzle system
- Backer ship date (check the Kickstarter Updates tab)
- How reliably toolhead swaps repeat over thousands of cycles, and ooze from preheated docked heads. No hands-on reviews exist yet; all coverage is pre-order analysis.

## Expected fit with the farm
- **Treat it as self-contained, like the K1 Max and SparkX i7.** Toolchanging, IDEX modes, the camera XY calibration and the eddy probe will rely on Sovol's own macros, so it shouldn't use `klipper/shared/`. PRINT_START will probably have a different name and different parameters.
- Sovol eddy probes have needed Sovol's patched `sensor_ldc1612.c`; mainline Klipper had I2C issues with the SV08 eddy kit (see eddy-ng issue #105 below). **Don't move it to mainline Klipper** until the firmware has been reviewed.
- Essential is open-frame. Plan for PLA, PETG, TPU and PVA; Sovol only recommends ABS, ASA, PA and PC with the heated enclosure. For those materials, use the K1 Max (enclosed) for now.
- Orca: start from Sovol's system profile. Multi-tool profiles (7 extruders, IDEX copy/mirror) are new territory for this repo. Upstream OrcaSlicer merged generic IDEX/IQEX parallel printing on 2026-10-10 (PR #13086: `is_imex`, per-plate Copy/Mirror, `imex_mode_gcodes`, per-tool PA). It's not in a release yet, and the PR thread expects the M1D community to adopt it. Check whether Sovol's profile uses it.

## On arrival checklist
- [ ] Record the firmware version, mainboard/SoC, host hostname and IP
- [ ] Identify the host: `cat /proc/device-tree/model`, `cat /proc/cpuinfo`, `lsblk` (expect 32 GB eMMC); Klipper fork and version: `git -C ~/klipper remote -v`, `git -C ~/klipper log -1`
- [ ] Confirm the probe (`[probe_eddy_current]` `sensor_type`) and offsets, extruder type, `rotation_distance` and `gear_ratio`; note how IDEX and toolchanging are configured (`[dual_carriage]` or a custom module)
- [ ] Run `ACCELEROMETER_QUERY` once
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
- Sovol press release (2026-07-24): https://www.prnewswire.com/news-releases/sovol-introduces-m1d-combining-multi-color-multi-material-printing-and-dual-extruder-productivity-in-an-open-source-platform-302833992.html
- Sovol M1D landing page: https://www.sovol3d.com/pages/sovol-m1d-landing-page
- Sovol first-print setup guide: https://www.sovol3d.com/blogs/news/sovol-m1d-first-print-setup-a-beginners-guide
- Sovol maintenance guide: https://www.sovol3d.com/blogs/news/tool-changing-3d-printer-maintenance-how-to-keep-your-m1d-printing-reliably
- sovol.uk overview: https://sovol.uk/blogs/news/sovol-m1d-multi-color-multi-material-3d-printer
- JCPRINTFARM pre-order analysis (ship-date quote from 2026-09-28): https://www.jcprintfarm.com/blogs/3d-printing-tech/sovol-m1d-review
- OrcaSlicer PR #13086 (IDEX/IQEX parallel printing): https://github.com/OrcaSlicer/OrcaSlicer/pull/13086
- eddy-ng issue on the Sovol eddy probe and mainline Klipper: https://github.com/vvuk/eddy-ng/issues/105
