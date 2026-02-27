# BDsensor RTL + Ender 7 Accel Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Enable BDsensor Real-Time Leveling in hybrid mode on the Wanhao D6 and Ender 7, and reduce Ender 7 max_accel from 6500 to 4400.

**Architecture:** Three config-only edits across two printer.cfg files. RTL is activated by adding `rt_sample_time` to the `[BDsensor]` section; no changes to shared start.cfg or macros. The existing KAMP adaptive mesh runs at print start as before; RTL supplements it in real-time during the print.

**Tech Stack:** Klipper config (.cfg files). No code, no tests — verification is done by restarting Klipper on each printer and observing a test print.

**Design doc:** `docs/plans/2026-02-27-bdsensor-rtl-design.md`

---

### Task 1: Ender 7 — reduce max_accel and annotate rt_sample_time

**Files:**
- Modify: `klipper/printers/ender7/printer.cfg`

**Step 1: Edit `[printer]` max_accel**

In `klipper/printers/ender7/printer.cfg`, find the `[printer]` section:

```ini
[printer]
kinematics: corexy
max_velocity: 500
max_accel: 6500
```

Change to:

```ini
[printer]
kinematics: corexy
max_velocity: 500
max_accel: 4400         # Reduced from 6500 — matches farm standard; X shaper unverified
```

**Step 2: Annotate rt_sample_time in `[BDsensor]`**

Find the existing `rt_sample_time: 19` line in the `[BDsensor]` section:

```ini
rt_sample_time: 19
```

Change to:

```ini
rt_sample_time: 19      # Enables RTL — samples Z every 19ms during print (hybrid mode: mesh + RTL)
```

**Step 3: Verify the diff looks right**

Run:
```bash
git diff klipper/printers/ender7/printer.cfg
```

Expected: two-line diff — `max_accel` value change, `rt_sample_time` comment addition. No other changes.

**Step 4: Commit**

```bash
git add klipper/printers/ender7/printer.cfg
git commit -m "Ender 7: reduce max_accel to 4400, annotate BDsensor rt_sample_time"
```

---

### Task 2: Wanhao D6 — add rt_sample_time for RTL

**Files:**
- Modify: `klipper/printers/wanhao_d6/printer.cfg`

**Step 1: Add rt_sample_time to `[BDsensor]`**

In `klipper/printers/wanhao_d6/printer.cfg`, find the `[BDsensor]` section. It currently ends with:

```ini
collision_calibrate: 1
speed: 3
homing_cmd: G28
```

Add `rt_sample_time` after `homing_cmd`:

```ini
collision_calibrate: 1
speed: 3
homing_cmd: G28
rt_sample_time: 25      # Enables RTL — samples Z every 25ms during print (hybrid mode: mesh + RTL)
```

25ms (vs Ender 7's 19ms) is appropriate for this slower Cartesian printer (max_accel 3900).

**Step 2: Verify the diff looks right**

Run:
```bash
git diff klipper/printers/wanhao_d6/printer.cfg
```

Expected: single-line addition of `rt_sample_time: 25` inside `[BDsensor]`. No other changes.

**Step 3: Commit**

```bash
git add klipper/printers/wanhao_d6/printer.cfg
git commit -m "Wanhao D6: enable BDsensor RTL (rt_sample_time: 25, hybrid mode)"
```

---

### Task 3: Deploy and verify — Ender 7

**Step 1: Deploy config to Ender 7**

```bash
scp klipper/printers/ender7/printer.cfg pi@ender7:~/printer_data/config/
```

**Step 2: Restart Klipper**

SSH into the Ender 7 Pi and restart:
```bash
ssh pi@ender7
sudo systemctl restart klipper
```

Or via Mainsail UI → Machine → Restart Klipper.

**Step 3: Check for config errors**

In Mainsail (or `journalctl -u klipper -f` on the Pi), confirm Klipper starts cleanly with no errors. Expected: no errors related to `[BDsensor]` or `[printer]`.

**Step 4: Test RTL during a print**

Start a small test print. During the first layer, observe the Z position in Mainsail — with RTL active, small Z micro-adjustments will appear in real-time as the BDsensor reads the bed surface. This confirms RTL is running.

---

### Task 4: Deploy and verify — Wanhao D6

**Step 1: Deploy config to D6**

```bash
scp klipper/shared/macros.cfg klipper/shared/start.cfg pi@wanhao_d6:~/printer_data/config/
scp klipper/printers/wanhao_d6/printer.cfg pi@wanhao_d6:~/printer_data/config/
```

(Note: only `printer.cfg` changed, but deploy shared files too for consistency.)

**Step 2: Restart Klipper**

```bash
ssh pi@wanhao_d6
sudo systemctl restart klipper
```

**Step 3: Check for config errors**

Confirm Klipper starts cleanly in Mainsail. No errors expected.

**Step 4: Test RTL during a print**

Same as Ender 7 — start a small test print and observe real-time Z micro-adjustments during the first layer to confirm RTL is active.

---

### Task 5: Update MEMORY.md

**Files:**
- Modify: `/home/drupi/.claude/projects/-home-drupi-printfarm/memory/MEMORY.md`

Update the Known Pending Issues section to remove any note suggesting RTL is not configured, and note that RTL is now enabled on both printers.
