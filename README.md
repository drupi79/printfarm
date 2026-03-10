# printfarm

Klipper configs, OrcaSlicer profiles, and Marlin hardware notes for a home 3D printer farm.

## Printers

| Printer | Kinematics | Bed | Probe | Hotend | Host |
|---------|-----------|-----|-------|--------|------|
| Ender 3 V2 Twin 1 | Cartesian | 220×220 | BLTouch | TZ E3 2.0 | Intel NUC (shared) |
| Ender 3 V2 Twin 2 | Cartesian | 220×220 | BLTouch | TZ E3 2.0 | Intel NUC (shared) |
| Ender 3 V2 Marlin | Cartesian | 220×220 | BLTouch | TZ E3 2.0 | TBD |
| Ender 3 Pro | Cartesian | 220×220 | BLTouch | Spider V3 Pro | Pi 4 8GB |
| Ender 3 OG | Cartesian | ~220×220 | BLTouch | Spider V3 Pro | Pi 3 |
| Ender 7 | CoreXY | 260×260 | BDsensor | TZ E3 2.0 | Pi 5 4GB |
| Neptune 3 Max | Cartesian | 430×430 | Inductive (stock) | Copperhead | Pi 5 8GB |
| Wanhao D6 | Cartesian | 200×200 | BDsensor (EBB42 CAN) | Bambu X1C (HICTOP) | Pi 4 8GB |
| K1 Max | CoreXY | 300×300 | Strain gauge | Unicorn | — |

All Klipper printers run direct drive BMG or equivalent extruders, firmware retraction (0.6mm), and KAMP adaptive bed meshing. The Wanhao D6 uses a BTT EBB42 CAN toolhead. The K1 Max is rooted via Guilouz Helper Script and runs its own macro set (not shared) with a Creality metal extruder and Unicorn hotend upgrade.

> **Marlin (MRisCoC) — no Klipper config:** Ender 3 V2 Marlin — hardware notes in `marlin/ender3_v2_marlin/`

## Structure

```
klipper/
  shared/          # Deployed to every Klipper printer (except K1 Max)
    macros.cfg
    start.cfg      # PRINT_START/END, PAUSE/RESUME, adaptive mesh
    start_manual_purge.cfg
  printers/
    <name>/printer.cfg          # Most printers: single file + shared includes
    k1_max/                     # K1 Max: self-contained (no shared macros)
      printer.cfg
      gcode_macro.cfg           # K1-specific macros (fan mapping, Qmode, prtouch)
      printer_params.cfg
      sensorless.cfg
marlin/
  <name>/          # Marlin printer hardware notes (no Configuration.h — stock builds)
orca_profiles/
  <name>/          # Printer + process + filament profiles per machine
Printer_Backup/
  <name>/          # Full Pi config snapshots (mainsail.cfg, KAMP_Settings.cfg,
                   # moonraker.conf, crowsnest.conf, etc.) — reference only
```

## Deploying to a Printer

Most printers — copy shared files and the printer-specific config:

```bash
scp klipper/shared/*.cfg klipper/printers/<name>/printer.cfg pi@<hostname>:~/printer_data/config/
```

K1 Max — copy all four files in `klipper/printers/k1_max/` (do **not** overwrite `Helper-Script/` files managed by the Guilouz helper script):

```bash
scp klipper/printers/k1_max/*.cfg root@k1max:~/printer_data/config/
```
