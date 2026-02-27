# Anycubic Chiron Klipper Build — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Convert an Anycubic Chiron with a dead mainboard into a fully Klipper-capable printer with CAN bus toolhead, MGN12H X-axis, dual Z leveling, and a custom 6mm AC-heated bed.

**Architecture:** Hardware build followed by Klipper config authoring. Config lives at `klipper/printers/chiron/printer.cfg` and includes the farm's shared `start.cfg` and `macros.cfg`. Three MCUs: SKR 3 EZ (USB), EBB36 (CAN). Calibration runs last, on the physical printer.

**Design doc:** `docs/plans/2026-02-27-chiron-klipper-design.md`

**Reference configs:** `klipper/printers/wanhao_d6/printer.cfg` (EBB42 CAN pattern), `klipper/printers/ender7/printer.cfg` (BDsensor + dual Z pattern)

---

## Phase 1: Measurements & Pre-Order Verification

### Task 1: Measure Chiron carriage and frame geometry

**Tools needed:** Digital calipers, ruler, paper + pencil.

**Step 1: Measure the X-axis upper 2040 extrusion**

Measure the full length of the upper 2040 X extrusion. Expected: ~500mm. This confirms the MGN12H rail length to order (match extrusion length).

Record: upper extrusion length = ____mm

**Step 2: Measure the stock carriage bolt pattern**

Even though we're replacing the carriage with MGN12H, document the stock pattern for reference:
- Hole center-to-center horizontal: ____mm
- Hole center-to-center vertical: ____mm
- Hole diameter: ____mm

**Step 3: Measure the Z leadscrew X positions**

With the printer powered off and the bed roughly centered in Y, measure the horizontal distance from the nozzle centerline to each Z leadscrew center (left and right columns). These are the `z_positions` values for `[z_tilt]`.

Record:
- Left leadscrew nozzle X offset from bed left edge: ____mm (will be negative, outside bed)
- Right leadscrew nozzle X offset from bed right edge: ____mm (will be positive, outside bed)

Example: if bed left edge is at nozzle X=0 and left leadscrew is 30mm further left: z_position left X = -30

**Step 4: Measure bed mounting hole pattern**

Measure the 4 bed spring screw positions on the Chiron bed carriage (XY positions of each hole center, relative to bed front-left corner). You will need this to drill the MIC-6 plate.

Record all 4 hole positions.

**Step 5: Measure vertical rail-to-rail spacing**

Measure center-to-center distance between the two 2040 X extrusions (upper and lower). Record for toolhead clearance verification.

Record: vertical rail spacing = ____mm

**Step 6: Record all measurements in the design doc**

```bash
# Edit the design doc to add your measurements under a new "Measured Values" section
# File: docs/plans/2026-02-27-chiron-klipper-design.md
```

Commit:
```bash
git add docs/plans/2026-02-27-chiron-klipper-design.md
git commit -m "Chiron: add measured frame and carriage dimensions"
```

---

### Task 2: Verify EVA 3 toolhead compatibility

**Step 1: Check EVA 3 for HGX Lite top with round-body NEMA14**

Visit `github.com/EVA-3D/eva-main` and the EVA 3 Printables collection. Search for "HGX Lite" in community contributions.

Verify:
- Does an HGX Lite extruder top exist for EVA 3? (yes/no)
- Does it support a **round-body** NEMA14 (36mm diameter), not just flat-face? (yes/no)
- If no round-body support: note that a custom collar/clamp plate will need to be designed

**Step 2: Check EVA 3 for EBB36 back plate**

Search EVA 3 community contributions for "EBB36" back plate. Verify it exists and download the STL.

**Step 3: Check for BDsensor mount compatible with EVA 3 front**

Search EVA 3 community or BDsensor GitHub for a mount that fits the EVA 3 front face. Note if a custom bracket will be needed.

**Step 4: Document findings**

Add a "Toolhead Compatibility" note to the design doc with links to all confirmed STLs, and flag anything that needs custom design work.

```bash
git add docs/plans/2026-02-27-chiron-klipper-design.md
git commit -m "Chiron: document EVA 3 compatibility findings"
```

---

### Task 3: Place orders (priority order)

Order in this sequence — Keenovo has the longest lead time.

**Step 1: Order Keenovo AC silicone heater (order first)**

Go to keenovo.com. Configure:
- Size: 410×430mm
- Voltage: 120VAC
- Wattage: 750-1000W (ask Keenovo for recommendation for this size/voltage)
- Backing: adhesive PSA
- Thermistor: add NTC 100K thermistor option

Confirm order and record estimated delivery date.

**Step 2: Order MIC-6 aluminum plate**

Option A (SendCutSend): Upload DXF file with:
- Outer dimensions: 410×430mm
- Material: 6mm MIC-6 cast aluminum tooling plate
- Holes: 4× M4 clearance holes at your measured bed spring positions (use clearance holes, not threaded)
- Note: you may need to create the DXF in FreeCAD, Fusion 360, or similar

Option B: Order raw 410×430mm MIC-6 from onlinemetals.com (cut to size), then take to local machine shop for hole drilling.

**Step 3: Order PEI sheet**

Search AliExpress for "Anycubic Chiron PEI spring steel" or "410x430 PEI spring steel". Find Energetic brand listing. **Verify dimensions are exactly 410×430mm before ordering** — some listings show similar sizes. Add magnetic base if not included.

**Step 4: Order electronics**

From BTT official AliExpress store or West3D:
- BTT SKR 3 EZ
- BTT EZ2209 ×4
- BTT U2C v2.1
- BTT EBB36 v1.2

From BIQU/BTT:
- BDsensor-M

From Digi-Key or Mouser:
- Crydom D1D40 SSR (40A, 120VAC)

From Amazon/local:
- 500mm MGN12H linear rail + carriage block (Trianglelab or quality AliExpress brand)
- M3 T-nuts (roll-in, spring-loaded, 2020/2040 compatible), M3×8 SHCS ×20
- SSR aluminum heatsink (50×50mm min)
- Thermal paste
- 10A fast-blow fuse + fuse holder (panel-mount)
- 130°C thermal fuse ×2
- 14AWG silicone high-flex wire (red + black, 1m each for AC leads)
- Braided cable sleeve (4-6mm diameter, 1m) for umbilical
- 24AWG shielded twisted pair wire (CAN cable, 1m)
- Buffer transistor for SSR logic (2N2222 or BC337) + 1kΩ resistor

---

## Phase 2: Bed System

### Task 4: Prepare and install MIC-6 bed plate

**Prerequisites:** MIC-6 plate received with holes drilled. Chiron bed carriage accessible.

**Step 1: Clean plate surfaces**

Wipe both faces of the MIC-6 plate with IPA. Remove any cutting oil or machining residue.

**Step 2: Verify hole pattern**

Test-fit the plate on the Chiron bed carriage spring screws. All 4 holes should align. If not, a machine shop can enlarge holes slightly.

**Step 3: Check standoff height**

With the plate on the carriage, verify that the bed springs + standoffs position the plate at the correct height for the toolhead's Z range. The 6mm plate is thicker than stock — you may need longer bed spring standoffs (M3 or M4 standoffs, 5-10mm longer than stock). Adjust as needed.

---

### Task 5: Install AC silicone heater

**Prerequisites:** Keenovo heater received. MIC-6 plate installed and verified.

⚠️ **This task involves 120VAC mains wiring. Ensure the printer is unplugged from mains power for the entire task.**

**Step 1: Plan heater wire routing**

Before removing the adhesive liner, determine how the heater leads will route to the SSR. The leads must reach from the underside of the bed to the electronics bay via the Y-axis cable routing path.

**Step 2: Bond heater to plate**

- Wipe plate underside with IPA, let dry fully
- Remove Keenovo adhesive liner
- Align heater to plate underside (centered)
- Apply from center outward, pressing firmly to eliminate bubbles
- Apply firm pressure across entire surface for 60 seconds

**Step 3: Install thermal fuse**

Attach the 130°C thermal fuse to the heater surface or aluminum plate edge with kapton tape. Wire it in series with the heater circuit (on the Live/Hot wire).

**Step 4: Route heater leads**

Route 14AWG silicone leads from heater through the Y-axis cable path to the electronics bay. Leave enough slack for full Y-axis travel. Secure with cable clips. Terminate with appropriate connectors for SSR connection.

**Step 5: Ground the bed plate**

Run a 14AWG green/yellow wire from a bolt on the MIC-6 plate frame to the printer chassis ground point and from chassis to PSU earth terminal. Use ring terminals. This is mandatory safety grounding.

---

### Task 6: Install SSR and mains wiring

⚠️ **120VAC mains wiring. Printer must be unplugged for this entire task. Double-check before applying power.**

**Step 1: Mount SSR with heatsink**

Apply thermal paste to SSR bottom face. Mount SSR to heatsink with provided hardware or M4 screws. Mount heatsink assembly in electronics bay, away from direct bed radiant heat. Ensure airflow around heatsink.

**Step 2: Wire SSR control side (DC)**

SSR control input (terminals 3 and 4, low-voltage DC side):
- Wire terminal 3 (+) to SKR 3 EZ bed heater output pin (through buffer transistor — see Step 3)
- Wire terminal 4 (-) to SKR 3 EZ GND

**Step 3: Buffer transistor for 3.3V logic**

The SKR 3 EZ outputs 3.3V logic which is marginal for SSR trigger. Wire a buffer:
```
SKR 3 EZ bed pin ──[1kΩ]──► BC337 base
BC337 collector ──────────► SSR terminal 3 (+)
BC337 emitter ────────────► GND
+5V ──────────────────────► SSR terminal 3 (+) via 470Ω  [pulls up to 5V]
```
Alternatively: use a logic-level N-channel MOSFET (e.g. 2N7000) in the same configuration.

**Step 4: Wire SSR load side (AC mains)**

SSR load terminals (1 and 2, high-voltage AC side):
- Terminal 1: AC Neutral wire
- Terminal 2: AC Live wire (via thermal fuse → SSR → heater Live)

Mains wiring order (Live wire path):
```
Mains Live ──► 10A fuse ──► SSR terminal 2 ──► heater Live lead
Mains Neutral ─────────────► SSR terminal 1 ──► heater Neutral lead
```

**Step 5: Verify wiring before applying power**

With printer still unplugged:
- Double-check all AC connections are secure, insulated, and strain-relieved
- Verify SSR control wires are on the correct (DC) terminals
- Verify thermal fuse is in series on the Live wire
- Verify bed plate ground wire is connected

**Step 6: Test SSR function**

Plug printer in. In Klipper (after config is set up — revisit this step during Task 13):
```
SET_HEATER_TEMPERATURE HEATER=heater_bed TARGET=30
```
Verify SSR LED indicator illuminates and bed begins warming. Immediately:
```
SET_HEATER_TEMPERATURE HEATER=heater_bed TARGET=0
```

---

## Phase 3: Electronics Installation

### Task 7: Install SKR 3 EZ and flash firmware

**Step 1: Install EZ2209 drivers**

Insert 4× BTT EZ2209 into M1 (X), M2 (Y), M3 (Z), M4 (Z1) slots. The E0 slot (M5) is left empty — extruder is on EBB36.

Verify orientation matches BTT SKR 3 EZ driver diagram.

**Step 2: Mount SKR 3 EZ in electronics bay**

Mount board using M3 standoffs. Connect existing stepper motor cables (X, Y, Z-left, Z-right). Connect endstop cables (X will come from EBB36 later — leave X endstop disconnected for now). Connect Y and Z optical endstop cables.

**Step 3: Build Klipper firmware for SKR 3 EZ**

On the Pi:
```bash
cd ~/klipper
make menuconfig
```

Settings (must match exactly):
```
Micro-controller Architecture: STMicroelectronics STM32
Processor model: STM32H743
Bootloader offset: 128KiB bootloader      ← CRITICAL
Clock reference: 25 MHz crystal           ← CRITICAL
Communication interface: USB (on PA11/PA12)
```

```bash
make clean
make
```

Expected output: `out/klipper.bin`

**Step 4: Flash firmware to SKR 3 EZ**

```bash
cp out/klipper.bin /path/to/sdcard/firmware.bin
```

With printer powered off, insert SD card into the face-accessible slot on the SKR 3 EZ (not the bottom slot). Power on. Wait 20-30 seconds. Power off. Remove SD card.

Verify flash succeeded: the file should now be named `firmware.CUR` on the card.

**Step 5: Find SKR 3 EZ USB serial path**

```bash
ls /dev/serial/by-id/
```

Expected: `usb-Klipper_stm32h743xx_XXXXXXXXXXXX-if00`

Record this path for use in `printer.cfg`.

---

### Task 8: Set up CAN bus (U2C + EBB36)

**Step 1: Connect U2C to Pi**

Connect BTT U2C v2.1 to the Pi via USB. Connect CAN bus cable (4-wire: 24V+, GND, CAN H, CAN L) from U2C CAN output to EBB36 CAN input.

Verify U2C and EBB36 termination resistors: U2C has a 120Ω termination jumper — enable it. EBB36 has a 120Ω termination jumper — enable it. (Both ends of the CAN bus must be terminated.)

**Step 2: Configure CAN interface on Pi**

Create `/etc/network/interfaces.d/can0`:
```
allow-hotplug can0
iface can0 can static
    bitrate 1000000
    up ifconfig $IFACE txqueuelen 1024
```

Apply:
```bash
sudo ifup can0
ip link show can0
```

Expected: `can0: <NOARP,UP,LOWER_UP,ECHO> mtu 16 qdisc pfifo_fast state UP`

**Step 3: Flash Klipper firmware to EBB36 via Katapult**

The EBB36 v1.2 uses STM32G0B1 with Katapult (CanBoot) bootloader. Build EBB36 firmware:

```bash
cd ~/klipper
make menuconfig
```

Settings for EBB36 v1.2:
```
Micro-controller Architecture: STMicroelectronics STM32
Processor model: STM32G0B1
Bootloader offset: 8KiB bootloader
Clock reference: 8 MHz crystal
Communication interface: CAN bus (on PB0/PB1)
CAN bus speed: 1000000
```

```bash
make clean
make
```

Flash via Katapult:
```bash
python3 ~/katapult/scripts/flash_can.py -i can0 -u <EBB36_UUID> -f ~/klipper/out/klipper.bin
```

(If EBB36 UUID is unknown, use DFU mode via USB-C on the EBB36 for initial flash — see BTT EBB36 documentation.)

**Step 4: Discover EBB36 CAN UUID**

```bash
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
```

Expected output:
```
Found canbus_uuid=xxxxxxxxxxxx
```

Record UUID for `printer.cfg`.

---

## Phase 4: X-Axis MGN12H Rail

### Task 9: Install MGN12H rail on upper 2040 extrusion

**Prerequisites:** All V-slot wheel carriage hardware removed from upper 2040 extrusion.

**Step 1: Clean the 2040 extrusion top face**

Wipe the top face of the upper 2040 extrusion with IPA. Remove any debris from the T-slot channel.

**Step 2: Insert M3 T-nuts into top T-slot**

Insert spring-loaded roll-in M3 T-nuts into the top channel of the upper 2040 at positions matching the MGN12H rail hole pitch (15mm apart). Install enough T-nuts to cover the full 500mm rail length (roughly 30-35 T-nuts).

**Step 3: Position and align rail**

Lay the 500mm MGN12H rail on the top face of the 2040 extrusion. Use a straight edge against the extrusion face to align the rail parallel to the extrusion. Center the rail on the extrusion.

**Step 4: Bolt rail down**

Thread M3×8 SHCS into each T-nut through the rail holes. Start from center, work outward. Do not fully tighten yet — keep rail adjustable.

**Step 5: Align rail precisely**

Run the MGN12H carriage block back and forth. It should move smoothly with no binding. If there is binding, the rail is not parallel — loosen and readjust. Once smooth, fully tighten all M3 bolts in order (center to ends).

**Step 6: Verify smooth carriage travel**

Push carriage from end to end by hand. Should move with minimal resistance, no roughness, no side play. If rough: check for debris or misalignment.

---

### Task 10: Assemble and mount EVA 3 toolhead

**Prerequisites:** All EVA 3 STL files printed (PETG or ASA). HGX Lite, TZ E3 2.0, dual 5015 fans, EBB36 in hand.

**Step 1: Print EVA 3 components**

Print in PETG or ASA (heat resistant — near the hotend):
- EVA 3 core body (MGN12H mount)
- HGX Lite extruder top (community version for round-body NEMA14 — or custom collar if needed)
- V6 groove mount hotend front
- Dual 5015 fan duct
- EBB36 back plate
- BDsensor mount bracket

**Step 2: Assemble hotend into EVA 3 front**

Insert TZ E3 2.0 into V6 groove mount front section. Secure with groove mount retaining screws. Route heater cartridge and thermistor wires through EVA 3 body channels.

**Step 3: Mount HGX Lite extruder**

Attach FYSETC 36mm round body NEMA14 motor to HGX Lite extruder body. Stack EBB36 directly behind motor using motor mount holes (M3×8). Route motor cable from EBB36 stepper connector to motor.

Mount HGX Lite + motor + EBB36 assembly to EVA 3 extruder top. Secure with M3 screws.

**Step 4: Mount dual 5015 fans**

Attach dual 5015 blowers to EVA 3 fan duct sections. Connect fan cables to EBB36 fan outputs.

**Step 5: Mount BDsensor**

Attach BDsensor to BDsensor mount bracket. Mount bracket to EVA 3 front face so sensor tip is 0-2mm offset from nozzle tip height. Measure and record x_offset and y_offset (distance from nozzle centerline to sensor centerline).

**Step 6: Mount assembled toolhead to MGN12H carriage**

Bolt EVA 3 core body to MGN12H carriage block using M3×8 screws through carriage block holes. Verify toolhead is square to the rail.

**Step 7: Wire X endstop to EBB36**

Connect the X endstop switch signal and GND wires to the EBB36 endstop input header. Verify pin assignment against EBB36 v1.2 pinout diagram. Record pin name (e.g. `PB6`).

**Step 8: Wire filament runout sensor to EBB36**

Connect filament runout sensor to the second EBB36 endstop input. Mount sensor body at a convenient location in the filament path. Record pin name.

**Step 9: Route and connect umbilical**

Route 4-wire CAN umbilical (24V+, GND, CAN H, CAN L) from U2C/electronics bay, through braided sleeve, to EBB36 CAN input connector. Anchor sleeve at frame with a printed or purchased cable anchor. Leave enough slack for full X travel + 10% extra.

Connect BDsensor SDA, SCL, 5V, GND to EBB36 I2C header.

---

## Phase 5: Klipper Configuration

### Task 11: Create printer.cfg scaffold

**Files:**
- Create: `klipper/printers/chiron/printer.cfg`

**Step 1: Create the chiron config directory**

```bash
mkdir -p klipper/printers/chiron
```

**Step 2: Write the initial printer.cfg**

Create `klipper/printers/chiron/printer.cfg` with the following content. Fill in your measured values and discovered UUIDs where indicated:

```ini
# Klipper Configuration - Anycubic Chiron
# Board: BTT SKR 3 EZ (STM32H743, 128KiB bootloader, USB)
# Toolhead: BTT EBB36 CAN (canbus_uuid: <YOUR_UUID>)
# Hotend: TZ E3 2.0 (V6 groove mount)
# Extruder: HGX Lite + FYSETC 36mm NEMA14 (EBB36)
# Probe: BDsensor-M (EBB36 I2C, mesh only — optical endstop for Z home)
# Bed: MIC-6 6mm, 410x430mm, Keenovo AC heater via Crydom SSR
# Host: Raspberry Pi 5 4GB

# ----------------- INCLUDES -----------------
[include mainsail.cfg]
[include KAMP_Settings.cfg]
[include start.cfg]
[include macros.cfg]

# ----------------- MCU -----------------
[mcu]
serial: /dev/serial/by-id/usb-Klipper_stm32h743xx_<YOUR_SERIAL>-if00

[mcu EBBCan]
canbus_uuid: <YOUR_EBB36_UUID>

[mcu rpi]
serial: /tmp/klipper_host_mcu

# ----------------- PRINTER -----------------
[printer]
kinematics: cartesian
max_velocity: 300
max_accel: 3000
max_z_velocity: 10
max_z_accel: 100
square_corner_velocity: 5.0

# ----------------- EXCLUDE OBJECT -----------------
# (also in start.cfg — kept here for clarity, no conflict)
```

**Step 3: Commit scaffold**

```bash
git add klipper/printers/chiron/printer.cfg
git commit -m "Chiron: add printer.cfg scaffold"
```

---

### Task 12: Add stepper configuration

**Files:**
- Modify: `klipper/printers/chiron/printer.cfg`

Add stepper sections. Replace all pin values with actual SKR 3 EZ pin names from the BTT SKR 3 EZ pinout diagram PDF (available on BTT GitHub).

```ini
# ----------------- STEPPERS -----------------
[stepper_x]
step_pin: PF13          # SKR 3 EZ M1 STEP — verify against BTT pinout
dir_pin: PF12           # SKR 3 EZ M1 DIR
enable_pin: !PF14       # SKR 3 EZ M1 EN
microsteps: 16
rotation_distance: 40   # Standard 2GT belt, 20T pulley
endstop_pin: EBBCan:PB6 # X endstop on toolhead — verify EBB36 v1.2 pinout
position_endstop: 0
position_max: 400
homing_speed: 50

[tmc2209 stepper_x]
uart_pin: PC4           # SKR 3 EZ M1 UART — verify
run_current: 0.800
stealthchop_threshold: 0

[stepper_y]
step_pin: PG0           # SKR 3 EZ M2 STEP — verify
dir_pin: PG1            # SKR 3 EZ M2 DIR
enable_pin: !PF15       # SKR 3 EZ M2 EN
microsteps: 16
rotation_distance: 40
endstop_pin: PG9        # SKR 3 EZ Y endstop — verify
position_endstop: 0
position_max: 430
homing_speed: 50

[tmc2209 stepper_y]
uart_pin: PD11          # SKR 3 EZ M2 UART — verify
run_current: 0.800
stealthchop_threshold: 0

# Z left leadscrew
[stepper_z]
step_pin: PF11          # SKR 3 EZ M3 STEP — verify
dir_pin: !PG3           # SKR 3 EZ M3 DIR — verify polarity
enable_pin: !PG5        # SKR 3 EZ M3 EN
rotation_distance: 8    # T8 leadscrew, 8mm pitch
microsteps: 16
endstop_pin: PG10       # SKR 3 EZ Z optical endstop — verify
position_endstop: 0
position_min: -3
position_max: 450
homing_speed: 10
second_homing_speed: 3
homing_retract_dist: 5
homing_retract_speed: 5

[tmc2209 stepper_z]
uart_pin: PC6           # SKR 3 EZ M3 UART — verify
run_current: 0.650
stealthchop_threshold: 999999

# Z right leadscrew
[stepper_z1]
step_pin: PG4           # SKR 3 EZ M4 STEP — verify
dir_pin: PC1            # SKR 3 EZ M4 DIR — verify polarity
enable_pin: !PA0        # SKR 3 EZ M4 EN
rotation_distance: 8
microsteps: 16

[tmc2209 stepper_z1]
uart_pin: PC7           # SKR 3 EZ M4 UART — verify
run_current: 0.650
stealthchop_threshold: 999999
```

**Step 4: Commit**

```bash
git add klipper/printers/chiron/printer.cfg
git commit -m "Chiron: add stepper configuration"
```

---

### Task 13: Add extruder, heaters, and fans

**Files:**
- Modify: `klipper/printers/chiron/printer.cfg`

```ini
# ----------------- EXTRUDER -----------------
[extruder]
step_pin: EBBCan:PD0
dir_pin: EBBCan:PD1
enable_pin: !EBBCan:PD2
microsteps: 16
rotation_distance: 47.088   # HGX Lite 7.5:1 gear ratio base
gear_ratio: 7.5:1
nozzle_diameter: 0.400
filament_diameter: 1.750
heater_pin: EBBCan:PB13
sensor_type: ATC Semitec 104GT-2  # TZ E3 2.0 thermistor — verify
sensor_pin: EBBCan:PA3
min_temp: 0
max_temp: 300
max_extrude_only_distance: 200.0
max_extrude_cross_section: 5
min_extrude_temp: 170
pressure_advance: 0.04      # Starting value — tune with PA tower

[tmc2209 extruder]
uart_pin: EBBCan:PA15
run_current: 0.650
stealthchop_threshold: 0

# ----------------- FIRMWARE RETRACTION -----------------
[firmware_retraction]
retract_length: 0.6
retract_speed: 40
unretract_extra_length: 0
unretract_speed: 40

# ----------------- HEATER BED -----------------
[heater_bed]
heater_pin: PD7             # SKR 3 EZ HB pin — verify
sensor_type: Generic 3950   # Keenovo NTC 100K thermistor
sensor_pin: PA1             # SKR 3 EZ TB pin — verify
min_temp: 0
max_temp: 130
pwm_cycle_time: 0.0166      # 60Hz / 120VAC

# ----------------- FANS -----------------
[heater_fan heatbreak_cooling_fan]
pin: EBBCan:PA0             # EBB36 fan output — verify
heater: extruder
heater_temp: 50.0

[fan]
pin: EBBCan:PA2             # EBB36 part cooling fan output — verify

[heater_fan controller_fan]
pin: PD15                   # SKR 3 EZ FAN2 — verify
```

**Step 5: Commit**

```bash
git add klipper/printers/chiron/printer.cfg
git commit -m "Chiron: add extruder, heaters, and fans"
```

---

### Task 14: Add BDsensor, homing, and bed mesh

**Files:**
- Modify: `klipper/printers/chiron/printer.cfg`

Replace `<X_OFFSET>` and `<Y_OFFSET>` with your measured BDsensor-to-nozzle offsets.
Replace `<Z_LEFT_X>` and `<Z_RIGHT_X>` with your measured leadscrew positions from Task 1.

```ini
# ----------------- ADXL345 (INPUT SHAPER) -----------------
[adxl345]
cs_pin: EBBCan:PB12
spi_software_sclk_pin: EBBCan:PB10
spi_software_mosi_pin: EBBCan:PB11
spi_software_miso_pin: EBBCan:PB2

[resonance_tester]
accel_chip: adxl345
probe_points: 205, 215, 20   # Bed center (400/2, 430/2)

# ----------------- BDSENSOR -----------------
# BDsensor is MESH ONLY — Z homes to optical endstop, not BDsensor
[BDsensor]
sda_pin: EBBCan:PB8
scl_pin: EBBCan:PB9
delay: 10
x_offset: <X_OFFSET>        # Measure: nozzle X minus sensor X (positive = sensor behind nozzle)
y_offset: <Y_OFFSET>        # Measure: nozzle Y minus sensor Y
no_stop_probe:
position_endstop: 1.2
collision_homing: 0          # ← CRITICAL: use optical endstop, not BDsensor, for Z homing
collision_calibrate: 0       # ← CRITICAL: use PROBE_CALIBRATE paper test for z_offset
speed: 3
rt_sample_time: 20           # Starting value — tune after initial printing

# ----------------- HOMING -----------------
# safe_z_home uses NOZZLE coordinates
# home_xy_position = nozzle position where probe lands at bed center
# Bed center (nozzle): X=200, Y=215
# Probe lands at: X=200+x_offset, Y=215+y_offset
[safe_z_home]
home_xy_position: 200, 215
speed: 80
z_hop: 10
z_hop_speed: 10

# ----------------- Z_TILT -----------------
[z_tilt]
# z_positions: nozzle XY directly above each leadscrew
# These extend outside the bed boundary — that is correct
z_positions:
    <Z_LEFT_X>, 215       # Left leadscrew nozzle position (negative X, outside bed)
    <Z_RIGHT_X>, 215      # Right leadscrew nozzle position (positive X, outside bed)
points:
    30, 215               # Left probe point (probe coords, nozzle ~30mm from left edge)
    200, 215              # Center probe point
    370, 215              # Right probe point (nozzle ~30mm from right edge)
speed: 150
horizontal_move_z: 5
retries: 5
retry_tolerance: 0.0075

# ----------------- BED MESH -----------------
# Limits in PROBE coordinates (probe_pos = nozzle_pos + BDsensor offset)
# Adjust mesh_min/max after measuring actual BDsensor offset
[bed_mesh]
speed: 150
horizontal_move_z: 2
zero_reference_position: 200, 215   # Probe coords when nozzle at bed center — adjust after offset measurement
mesh_min: 10, 10          # Update after measuring BDsensor offset
mesh_max: 390, 420        # Update after measuring BDsensor offset
probe_count: 9, 9
algorithm: bicubic
fade_start: 1
fade_end: 10
fade_target: 0

# ----------------- BED SCREWS -----------------
[screws_tilt_adjust]
screw1: 30, 30
screw1_name: front left
screw2: 370, 30
screw2_name: front right
screw3: 370, 390
screw3_name: rear right
screw4: 30, 390
screw4_name: rear left
horizontal_move_z: 10
speed: 150
screw_thread: CW-M4       # Update to match your actual bed spring screws

# ----------------- MISC -----------------
[virtual_sdcard]
path: /home/pi/printer_data/gcodes
on_error_gcode: CANCEL_PRINT

[force_move]
enable_force_move: true

[gcode_arcs]
resolution: 0.1

[filament_switch_sensor filament_sensor]
pause_on_runout: true
switch_pin: EBBCan:<FILAMENT_SENSOR_PIN>   # Verify EBB36 pin

# ----------------- CHIRON PRINT_START OVERRIDE -----------------
# Overrides shared start.cfg PRINT_START to add Z_TILT step
[gcode_macro PRINT_START]
description: Chiron start — home, Z_TILT, mesh, print
gcode:
    {% set EXTRUDER = params.EXTRUDER_TEMP|default(200)|int %}
    {% set BED = params.BED_TEMP|default(60)|int %}

    RESPOND MSG="Chiron PRINT_START - Bed: {BED}C Extruder: {EXTRUDER}C"

    M104 S150         # Pre-heat extruder to standby (prevents ooze, not full temp)
    M140 S{BED}
    M190 S{BED}       # Wait for bed — dimensionally stable before homing

    _CHOME            # Home X, Y, Z (conditional — skips if already homed)

    Z_TILT_ADJUST     # Level gantry using dual Z before meshing

    G28 Z             # Re-home Z after tilt (re-establishes clean Z=0 at optical switch)

    BED_MESH_CALIBRATE   # Adaptive mesh (KAMP) — on a level gantry

    Smart_Park        # KAMP park near print area

    M104 S{EXTRUDER}
    M109 S{EXTRUDER}  # Heat to full print temp

    LINE_PURGE        # KAMP purge line

    G92 E0
    SET_VELOCITY_LIMIT ACCEL=500   # First layer accel cap

    RESPOND MSG="Print starting!"
```

**Step 6: Commit**

```bash
git add klipper/printers/chiron/printer.cfg
git commit -m "Chiron: add BDsensor, homing, bed mesh, Z_TILT, PRINT_START"
```

---

### Task 15: Deploy config and verify Klipper starts

**Step 1: Deploy config to Pi**

```bash
scp klipper/shared/macros.cfg klipper/shared/start.cfg pi@chiron:~/printer_data/config/
scp klipper/printers/chiron/printer.cfg pi@chiron:~/printer_data/config/
```

**Step 2: Restart Klipper**

```bash
ssh pi@chiron
sudo systemctl restart klipper
```

**Step 3: Check for config errors**

In Mainsail (or `journalctl -u klipper -f`): confirm Klipper starts with no errors.

Common errors and fixes:
- `Unknown pin` — wrong pin name, check BTT SKR 3 EZ pinout PDF
- `canbus_uuid not found` — CAN bus not running or wrong UUID; re-run `canbus_query.py`
- `BDsensor error` — check SDA/SCL wiring on EBB36
- `Extruder not found` — EBB36 not communicating; check CAN UUID and wiring

Iterate on config until Klipper starts cleanly with no errors.

**Step 4: Commit any pin corrections**

```bash
git add klipper/printers/chiron/printer.cfg
git commit -m "Chiron: fix pin assignments from initial deployment"
```

---

## Phase 6: Calibration

### Task 16: Endstop and motion verification

**Step 1: Verify endstops**

In Mainsail console:
```
QUERY_ENDSTOPS
```
Expected: `x:open y:open z:open` (with no triggers active).

Manually trigger each switch/sensor and re-run `QUERY_ENDSTOPS`:
- Trigger X switch on toolhead → `x:triggered`
- Trigger Y switch → `y:triggered`
- Block Z optical endstop → `z:triggered`

**Step 2: Test X and Y homing (no Z)**

With hand on emergency stop:
```
G28 X
G28 Y
```
Verify X homes to X=0 (left), Y homes to Y=0 (front). Watch for correct direction.

If direction is wrong: flip the `dir_pin` polarity in `[stepper_x]` or `[stepper_y]` (add or remove `!` prefix).

**Step 3: Test Z homing**

```
G28 Z
```
Watch Z lower slowly. Verify it stops at the optical endstop. Verify `position_endstop: 0` is correct.

**Step 4: Test full home**

```
G28
```
Verify all three axes home correctly in sequence.

---

### Task 17: BDsensor verification and Z_TILT

**Step 1: Verify BDsensor communication**

In Mainsail console:
```
PROBE_ACCURACY
```
Run 10 samples. Expected: standard deviation < 0.010mm. If BDsensor errors appear, check wiring and I2C pins.

**Step 2: Run Z_TILT_ADJUST**

```
Z_TILT_ADJUST
```
Watch both Z motors move. Confirm it converges within 5 retries and reports tilt < 0.0075mm.

If it does not converge: re-check `z_positions` values in `[z_tilt]` against your physical measurements.

**Step 3: Re-home Z after Z_TILT**

```
G28 Z
```

**Step 4: Probe calibration (set z_offset)**

```
PROBE_CALIBRATE
```
Follow the paper test procedure. Adjust Z until paper has slight friction. Accept with `TESTZ Z=0` and `ACCEPT`. Save:
```
SAVE_CONFIG
```

---

### Task 18: Bed mesh and PID calibration

**Step 1: Full bed mesh**

```
BED_MESH_CALIBRATE PROFILE=initial
SAVE_CONFIG
```

Inspect mesh in Mainsail. Expect variation < 0.5mm across 410×430 with MIC-6.

**Step 2: Update mesh_min/mesh_max for BDsensor offset**

With BDsensor offset now confirmed from physical measurement, update `[bed_mesh]` `mesh_min`, `mesh_max`, and `zero_reference_position` in `printer.cfg` to reflect probe coordinates (nozzle position + offset).

Commit corrected values:
```bash
git add klipper/printers/chiron/printer.cfg
git commit -m "Chiron: update bed mesh limits for measured BDsensor offset"
```

**Step 3: PID calibration — extruder**

With nozzle clear of bed:
```
PID_CALIBRATE HEATER=extruder TARGET=220
SAVE_CONFIG
```

**Step 4: PID calibration — bed**

```
PID_CALIBRATE HEATER=heater_bed TARGET=60
SAVE_CONFIG
```
⚠️ This will take 20-40 minutes for a 6mm aluminum plate. Do not cancel early.

Run a second calibration for ASA if needed:
```
PID_CALIBRATE HEATER=heater_bed TARGET=100
SAVE_CONFIG
```

---

### Task 19: Input shaper calibration

**Step 1: Ensure ADXL345 on EBB36 is working**

```
ACCELEROMETER_QUERY
```
Expected: returns X/Y/Z acceleration values. If it errors: check SPI pin assignments on EBB36.

**Step 2: Run input shaper calibration**

```
SHAPER_CALIBRATE
SAVE_CONFIG
```

Review recommended shaper type and frequency. Update `max_accel` in `[printer]` to the recommended value.

Commit:
```bash
git add klipper/printers/chiron/printer.cfg
git commit -m "Chiron: input shaper calibration complete, update max_accel"
```

---

### Task 20: First print and final cleanup

**Step 1: Extruder rotation_distance calibration**

Before first print, calibrate extruder:
1. Heat nozzle to 220°C
2. Mark filament 120mm from extruder entrance
3. Extrude 100mm: `MOVE_EXTRUDER DISTANCE=100` (or via Mainsail)
4. Measure remaining distance to mark
5. Adjust `rotation_distance` proportionally:
   `new_rotation_distance = old × (actual_extruded / 100)`

Commit corrected value.

**Step 2: First test print**

Slice a small calibration print (20mm cube or similar). Start print and monitor:
- First layer adhesion and height
- Z_TILT correction running at start
- BDsensor RTL active during print (watch Z position micro-adjustments in Mainsail)
- Part cooling fans responding
- Filament runout sensor not triggering falsely

**Step 3: Tune rt_sample_time for BDsensor RTL**

After first successful prints, tune `rt_sample_time` based on print quality. The Chiron's speed characteristics will determine the optimal value (reference: D6=25ms at 3900 max_accel, Ender 7=19ms at 4400 max_accel). Start at 20ms, adjust based on observed RTL correction behavior.

Commit final value.

**Step 4: Update CLAUDE.md and MEMORY.md**

Add Chiron to the farm documentation:
- `CLAUDE.md` — add Chiron printer-specific notes section
- `README.md` — add Chiron to printer inventory table
- `MEMORY.md` — add Chiron to printer inventory

```bash
git add CLAUDE.md README.md klipper/printers/chiron/printer.cfg
git commit -m "Chiron: first print complete, add to farm documentation"
git push
```

---

## Open Items Checklist

Before starting Phase 2, confirm:

- [ ] Chiron leadscrew X positions measured (Task 1)
- [ ] Bed mounting hole pattern measured (Task 1)
- [ ] Keenovo heater ordered (Task 3 — longest lead time)
- [ ] EVA 3 HGX Lite round-body NEMA14 top confirmed (Task 2)
- [ ] All electronics ordered (Task 3)

Before starting Phase 5 (config), confirm:

- [ ] SKR 3 EZ USB serial path recorded
- [ ] EBB36 CAN UUID recorded
- [ ] BDsensor x/y offset measured
- [ ] EBB36 endstop pin assignments verified against pinout diagram
