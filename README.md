# printfarm

Klipper configs and OrcaSlicer profiles for a home 3D printer farm.

## Printers

| Printer | Kinematics | Bed | Probe | Hotend |
|---------|-----------|-----|-------|--------|
| Ender 3 V2 Twin 1 | Cartesian | 220×220 | BLTouch | TZ E3 2.0 |
| Ender 3 V2 Twin 2 | Cartesian | 220×220 | BLTouch | TZ E3 2.0 |
| Ender 3 Pro | Cartesian | 220×220 | BLTouch | Spider V3 Pro |
| Ender 3 OG | Cartesian | ~220×220 | BLTouch | Spider V3 Pro |
| Ender 7 | CoreXY | 260×260 | BDsensor | TZ E3 2.0 |
| Neptune 3 Max | Cartesian | 430×430 | Inductive (stock) | Copperhead |
| Wanhao D6 | Cartesian | 220×220 | BDsensor (EBB42 CAN) | Bambu X1C (HICTOP) |

All Klipper printers run direct drive BMG or equivalent extruders, firmware retraction (0.6mm), and KAMP adaptive bed meshing. The Wanhao D6 also uses a BTT EBB42 CAN toolhead and Palette 2 for multi-material.

> **OrcaSlicer profiles only (no Klipper config in this repo):** Creality K1 Max (300×300)

## Structure

```
klipper/
  shared/          # Deployed to every printer
    macros.cfg
    start.cfg      # PRINT_START/END, PAUSE/RESUME, adaptive mesh
    start_manual_purge.cfg
  printers/
    <name>/printer.cfg
orca_profiles/
  <name>/          # Printer + process profiles per machine
```

## Deploying to a Printer

Copy shared files and the printer-specific config to the Pi's Klipper config directory (typically `~/printer_data/config/`), then restart Klipper.

```bash
scp klipper/shared/*.cfg klipper/printers/<name>/printer.cfg pi@<hostname>:~/printer_data/config/
```
