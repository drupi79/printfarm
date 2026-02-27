# printfarm

Klipper configs and OrcaSlicer profiles for a home 3D printer farm.

## Printers

| Printer | Kinematics | Bed | Probe | Hotend | Host |
|---------|-----------|-----|-------|--------|------|
| Ender 3 V2 Twin 1 | Cartesian | 220×220 | BLTouch | TZ E3 2.0 | Intel NUC (shared) |
| Ender 3 V2 Twin 2 | Cartesian | 220×220 | BLTouch | TZ E3 2.0 | Intel NUC (shared) |
| Ender 3 Pro | Cartesian | 220×220 | BLTouch | Spider V3 Pro | Pi 4 8GB |
| Ender 3 OG | Cartesian | ~220×220 | BLTouch | Spider V3 Pro | Pi 3 |
| Ender 7 | CoreXY | 260×260 | BDsensor | TZ E3 2.0 | Pi 5 4GB |
| Neptune 3 Max | Cartesian | 430×430 | Inductive (stock) | Copperhead | Pi 5 8GB |
| Wanhao D6 | Cartesian | 200×200 | BDsensor (EBB42 CAN) | Bambu X1C (HICTOP) | Pi 4 8GB |
| K1 Max | CoreXY | 300×300 | Strain gauge | Unicorn | — |

All Klipper printers run direct drive BMG or equivalent extruders, firmware retraction (0.6mm), and KAMP adaptive bed meshing. The Wanhao D6 also uses a BTT EBB42 CAN toolhead. The K1 Max is rooted via Guilouz Helper Script with Creality metal extruder and Unicorn hotend upgrades.

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
  <name>/          # Printer + process + filament profiles per machine
Printer_Backup/
  <name>/          # Full Pi config snapshots (mainsail.cfg, KAMP_Settings.cfg,
                   # moonraker.conf, crowsnest.conf, etc.) — reference only
```

## Deploying to a Printer

Copy shared files and the printer-specific config to the Pi's Klipper config directory (typically `~/printer_data/config/`), then restart Klipper.

```bash
scp klipper/shared/*.cfg klipper/printers/<name>/printer.cfg pi@<hostname>:~/printer_data/config/
```
