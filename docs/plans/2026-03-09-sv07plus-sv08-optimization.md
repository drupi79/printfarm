# SV07+ & SV08 Config Optimization Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create optimized Klipper configs and OrcaSlicer profiles for SV07+ and SV08 in new `_optimized` output folders.

**Architecture:** Copy-and-improve approach — preserve all calibrated values (PIDs, z_offset, input shaper, PA), fix bugs, restructure macros to printfarm pattern (PRINT_START takes params, does all work), create four process profiles per printer matching printfarm style with accel scaled to input shaper data.

**Tech Stack:** Klipper CFG, OrcaSlicer JSON

**Design doc:** `docs/plans/2026-03-09-sv07plus-sv08-optimization-design.md`

---

### Task 1: Create output directories

**Files:**
- Create: `Printer_Backup/Sv07+_optimized/` (directory)
- Create: `Printer_Backup/Sv08_optimized/` (directory)

**Step 1: Create dirs**

```bash
mkdir -p "Printer_Backup/Sv07+_optimized"
mkdir -p "Printer_Backup/Sv08_optimized"
```

**Step 2: Verify**
```bash
ls Printer_Backup/
```
Expected: both new dirs appear alongside `Sv07+/` and `Sv08/`.

---

### Task 2: Write SV07+ printer.cfg

**Files:**
- Read: `Printer_Backup/Sv07+/printer (1).cfg`
- Create: `Printer_Backup/Sv07+_optimized/printer.cfg`

**Step 1: Write the file**

Write `Printer_Backup/Sv07+_optimized/printer.cfg` with the following content. Key changes from original:
- `[printer]`: removed deprecated `max_accel_to_decel`, added `minimum_cruise_ratio: 0.5`
- `[resonance_tester]`: widened to `min_freq: 5`, `max_freq: 133`
- Added `[gcode_arcs]` and `[exclude_object]`
- Removed pointless `M205` macro
- `PRINT_START`: full heating/homing/mesh/purge macro (replaces empty original)
- `PRINT_END`: proper park/shutdown (replaces minimal original)
- `START_PRINT` / `END_PRINT`: kept as thin aliases for backwards compat
- `CANCEL_PRINT`: simplified to call PRINT_END logic
- `LOAD_FILAMENT` / `UNLOAD_FILAMENT`: added `TEMP=220` param with heat-before-extrude
- All SAVE_CONFIG values preserved exactly

```ini
[include plr.cfg]

[mcu]
serial: /dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
restart_method: command

[printer]
kinematics: cartesian
max_velocity: 300
max_accel: 12000
minimum_cruise_ratio: 0.5
max_z_velocity: 15
max_z_accel: 100
square_corner_velocity: 6

[virtual_sdcard]
path: /home/mks/printer_data/gcodes

[pause_resume]

[display_status]

[idle_timeout]
gcode:
    RESPOND TYPE=echo MSG="No operations in 10min!"
timeout: 600

[mcu rpi]
serial: /tmp/klipper_host_mcu

[adxl345]
cs_pin: rpi:None
spi_bus: spidev0.0

[verify_heater extruder]
max_error: 60
check_gain_time: 20
hysteresis: 5
heating_gain: 2

[verify_heater heater_bed]
max_error: 180
check_gain_time: 120
hysteresis: 5
heating_gain: 2

[resonance_tester]
accel_chip: adxl345
probe_points:
    150, 150, 20
accel_per_hz: 100
min_freq: 5
max_freq: 133
max_smoothing: 0.2
hz_per_sec: 0.5

[respond]

[gcode_arcs]
resolution: 0.1

[exclude_object]

#############################################################################################################
# GCODE MACROS
#############################################################################################################

[gcode_macro PRINT_START]
description: Full print start — heat, home, Z_TILT, mesh, purge
gcode:
    {% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(200)|float %}
    {% set BED_TEMP = params.BED_TEMP|default(60)|float %}
    SAVE_VARIABLE VARIABLE=was_interrupted VALUE=True
    M140 S{BED_TEMP}                    ; start bed heating
    M84 E                               ; disable E motor for probe accuracy
    G28                                 ; home all axes
    Z_TILT_ADJUST                       ; level dual Z
    G28 Z                               ; re-home Z after tilt
    M190 S{BED_TEMP}                    ; wait for bed temp
    BED_MESH_CALIBRATE                  ; probe fresh mesh
    G90
    G1 X5 Y20 Z5 F6000                  ; move to purge start position
    M109 S{EXTRUDER_TEMP}               ; heat extruder to print temp
    G92 E0
    G1 Z0.3 F600                        ; drop to purge height
    G1 Y100 E15 F500                    ; purge line forward
    G1 Y110 E2 F500                     ; short extra prime
    G92 E0
    G1 Z2.0 F3000                       ; raise Z before print

[gcode_macro PRINT_END]
description: Full print end — retract, park, shutdown
gcode:
    SAVE_VARIABLE VARIABLE=was_interrupted VALUE=False
    RUN_SHELL_COMMAND CMD=clear_plr
    clear_last_file
    G91
    G1 E-2 F500                         ; retract
    G1 Z5 F3000                         ; raise Z
    G90
    G1 X10 Y290 F6000                   ; park at rear
    M106 S0                             ; fan off
    TURN_OFF_HEATERS
    M84 X Y E                           ; disable motors (keep Z)
    RESPOND TYPE=echo MSG="Print finished!"

[gcode_macro START_PRINT]
description: Alias for PRINT_START (backwards compatibility)
gcode:
    PRINT_START {rawparams}

[gcode_macro END_PRINT]
description: Alias for PRINT_END (backwards compatibility)
gcode:
    PRINT_END

[gcode_macro CANCEL_PRINT]
description: Cancel the actual running print
rename_existing: CANCEL_PRINT_BASE
gcode:
    CANCEL_PRINT_BASE
    SAVE_VARIABLE VARIABLE=was_interrupted VALUE=False
    RUN_SHELL_COMMAND CMD=clear_plr
    clear_last_file
    G91
    G1 E-2 F500
    G1 Z5 F3000
    G90
    G1 X10 Y290 F6000
    M106 S0
    TURN_OFF_HEATERS
    M84 X Y E
    RESPOND TYPE=echo MSG="Print cancelled!"

[gcode_macro PAUSE]
description: Pause the actual running print
rename_existing: PAUSE_BASE
gcode:
    RESPOND TYPE=echo MSG="Pause Print!"
    {% set x = params.X|default(10) %}
    {% set y = params.Y|default(290) %}
    {% set z = params.Z|default(10)|float %}
    {% set e = params.E|default(1) %}
    {% set max_z = printer.toolhead.axis_maximum.z|float %}
    {% set act_z = printer.toolhead.position.z|float %}
    {% set lift_z = z|abs %}
    {% if act_z < (max_z - lift_z) %}
        {% set z_safe = lift_z %}
    {% else %}
        {% set z_safe = max_z - act_z %}
    {% endif %}
    PAUSE_BASE
    G91
    {% if printer.extruder.can_extrude|lower == 'true' %}
      G1 E-{e} F500
    {% else %}
      {action_respond_info("Extruder not hot enough")}
    {% endif %}
    {% if "xyz" in printer.toolhead.homed_axes %}
      G1 Z{z_safe}
      G90
      G1 X{x} Y{y} F6000
    {% else %}
      {action_respond_info("Printer not homed")}
    {% endif %}

[gcode_macro RESUME]
description: Resume the actual running print
rename_existing: RESUME_BASE
gcode:
    RESPOND TYPE=echo MSG="Resuming print!"
    {% if printer["filament_switch_sensor my_sensor"].filament_detected == True %}
        {% set e = params.E|default(1) %}
        {% if 'VELOCITY' in params|upper %}
            {% set get_params = ('VELOCITY=' + params.VELOCITY) %}
        {% else %}
            {% set get_params = "" %}
        {% endif %}
        G91
        {% if printer.extruder.can_extrude|lower == 'true' %}
          G1 E{e} F400
        {% else %}
            {action_respond_info("Extruder not hot enough")}
        {% endif %}
        RESUME_BASE {get_params}
    {% else %}
        RESPOND TYPE=echo MSG="Please insert filament in sensor!"
    {% endif %}

[gcode_macro LOAD_FILAMENT]
description: Load filament — TEMP=220
gcode:
    {% set TEMP = params.TEMP|default(220)|float %}
    M104 S{TEMP}
    RESPOND TYPE=echo MSG="Heating to {TEMP}C..."
    M109 S{TEMP}
    G91
    G1 E30 F300
    G1 E10 F150
    G90
    RESPOND TYPE=echo MSG="Filament loaded!"

[gcode_macro UNLOAD_FILAMENT]
description: Unload filament — TEMP=220
gcode:
    {% set TEMP = params.TEMP|default(220)|float %}
    M104 S{TEMP}
    RESPOND TYPE=echo MSG="Heating to {TEMP}C..."
    M109 S{TEMP}
    G91
    G1 E-30 F300
    G90
    RESPOND TYPE=echo MSG="Filament unloaded!"

[gcode_macro LED_ON]
gcode:
    SET_PIN PIN=my_led VALUE=1

[gcode_macro LED_OFF]
gcode:
    SET_PIN PIN=my_led VALUE=0

#############################################################################################################
# STEPPERS — TMC2209
#############################################################################################################

[stepper_x]
step_pin: PD15
dir_pin: PD14
enable_pin: !PC7
microsteps: 16
rotation_distance: 40
endstop_pin: tmc2209_stepper_x:virtual_endstop
homing_retract_dist: 0
position_endstop: -12
position_min: -12
position_max: 302
homing_speed: 50
step_pulse_duration: 0.000002

[tmc2209 stepper_x]
uart_pin: PE3
run_current: 1.2
uart_address: 3
interpolate: True
driver_sgthrs: 95
stealthchop_threshold: 0
diag_pin: ^PD10

[stepper_y]
step_pin: PB7
dir_pin: PB6
enable_pin: !PB9
microsteps: 16
rotation_distance: 40
endstop_pin: tmc2209_stepper_y:virtual_endstop
homing_retract_dist: 0
position_endstop: -6
position_min: -6
position_max: 302
homing_speed: 50
step_pulse_duration: 0.000002

[tmc2209 stepper_y]
uart_pin: PE4
run_current: 1.2
uart_address: 3
interpolate: True
driver_sgthrs: 95
stealthchop_threshold: 0
diag_pin: ^PE0

[stepper_z]
step_pin: PB3
dir_pin: !PD7
enable_pin: !PB5
microsteps: 16
rotation_distance: 8
endstop_pin: probe:z_virtual_endstop
position_max: 355
position_min: -4
homing_speed: 10

[stepper_z1]
step_pin: PA7
dir_pin: !PA6
enable_pin: !PC5
microsteps: 16
rotation_distance: 8
endstop_pin: probe:z_virtual_endstop

[extruder]
max_extrude_only_distance: 100.0
step_pin: PD1
dir_pin: !PD0
enable_pin: !PD4
microsteps: 16
rotation_distance: 4.59
nozzle_diameter: 0.400
filament_diameter: 1.750
heater_pin: PA1
sensor_type: EPCOS 100K B57560G104F
sensor_pin: PA4
control: pid
pressure_advance: 0.02
pressure_advance_smooth_time: 0.035
max_extrude_cross_section: 500
instantaneous_corner_velocity: 10
max_extrude_only_velocity: 2000
max_extrude_only_accel: 10000
pid_Kp: 24.522
pid_Ki: 1.397
pid_Kd: 107.590
min_temp: 0
max_temp: 305
min_extrude_temp: 150

[tmc2209 extruder]
uart_pin: PE7
run_current: 0.6
uart_address: 3
interpolate: True

[heater_bed]
heater_pin: PA2
sensor_type: EPCOS 100K B57560G104F
sensor_pin: PA3
control: pid
pid_Kp: 54.027
pid_Ki: 0.770
pid_Kd: 948.182
min_temp: 0
max_temp: 105

[probe]
pin: PD13
x_offset: 27
y_offset: -20
speed: 5.0

[filament_switch_sensor my_sensor]
switch_pin: PD11

[safe_z_home]
home_xy_position: 123, 170
speed: 80
z_hop: 5
z_hop_speed: 10

[z_tilt]
z_positions: -8, 170
             260, 170
points: -8, 170
        260, 170
speed: 200
horizontal_move_z: 5
retries: 20
retry_tolerance: 0.005

[bed_mesh]
speed: 200
horizontal_move_z: 5
mesh_min: 17, 15
mesh_max: 285, 282
probe_count: 5, 5
algorithm: bicubic
bicubic_tension: 0.3
fade_start: 0.2
fade_end: 5.0
mesh_pps: 4, 4
move_check_distance: 3

[screws_tilt_adjust]
screw1: 4, 58
screw1_name: front left screw
screw2: 243, 58
screw2_name: front right screw
screw3: 243, 290
screw3_name: rear right screw
screw4: 4, 290
screw4_name: rear left screw
horizontal_move_z: 5
speed: 200
screw_thread: CW-M4

[heater_fan hotend_fan]
pin: PE11

[multi_pin fan_pins]
pins: PE9, PE13

[fan]
pin: multi_pin:fan_pins

[output_pin my_led]
pin: PC4
pwm: 1
value: 1
cycle_time: 0.010

[controller_fan Fan_Board]
pin: PD3
fan_speed: 1.0
idle_timeout: 120
heater: heater_bed, extruder
stepper: stepper_x, stepper_y, stepper_z, stepper_z1

[input_shaper]
damping_ratio_x: 0.05
damping_ratio_y: 0.1

#*# <---------------------- SAVE_CONFIG ---------------------->
#*# DO NOT EDIT THIS BLOCK OR BELOW. The contents are auto-generated.
#*#
#*# [probe]
#*# z_offset = 1.800
#*#
#*# [input_shaper]
#*# shaper_type_x = mzv
#*# shaper_freq_x = 40.8
#*# shaper_type_y = mzv
#*# shaper_freq_y = 32.0
#*#
#*# [bed_mesh default]
#*# version = 1
#*# points =
#*# 	  -0.242500, -0.240000, -0.175000, -0.172500, -0.090000
#*# 	  -0.052500, -0.042500, -0.010000, -0.032500, 0.017500
#*# 	  -0.017500, -0.022500, -0.002500, -0.050000, -0.037500
#*# 	  -0.035000, -0.050000, -0.032500, -0.092500, -0.072500
#*# 	  -0.170000, -0.187500, -0.155000, -0.195000, -0.147500
#*# x_count = 5
#*# y_count = 5
#*# mesh_x_pps = 4
#*# mesh_y_pps = 4
#*# algo = bicubic
#*# tension = 0.3
#*# min_x = 17.0
#*# max_x = 285.0
#*# min_y = 15.0
#*# max_y = 282.0
```

**Step 2: Verify**

Open the file and check:
- No `max_accel_to_decel` anywhere
- `minimum_cruise_ratio: 0.5` present in `[printer]`
- No `M205` macro
- `PRINT_START` has `Z_TILT_ADJUST` and `BED_MESH_CALIBRATE`
- `LOAD_FILAMENT` has `TEMP` param and `M109`
- SAVE_CONFIG block matches original exactly

**Step 3: Commit**
```bash
git add "Printer_Backup/Sv07+_optimized/printer.cfg"
git commit -m "feat: SV07+ optimized printer.cfg — fixed macros, deprecated params, gcode_arcs"
```

---

### Task 3: Write SV08 printer.cfg

**Files:**
- Read: `Printer_Backup/Sv08/printer.cfg`
- Create: `Printer_Backup/Sv08_optimized/printer.cfg`

**Step 1: Write the file**

Key changes from original:
- `[resonance_tester]`: `min_freq: 5`, `max_freq: 133` (original 20–40 missed the 64.2Hz X shaper)
- `[input_shaper]`: `damping_ratio_x: 0.1`, `damping_ratio_y: 0.1` (original 0.001 is non-physical)
- `[probe]`: removed dead `speed: 15.0` line, kept `speed: 5.0`
- `[gcode_arcs]`: changed `resolution: 1.0` → `0.1`
- Added `PRINT_START` macro (full heat/home/QGL/mesh/purge)
- Added `PRINT_END` macro (replaces inline slicer gcode)
- Added `LOAD_FILAMENT` / `UNLOAD_FILAMENT` with TEMP param
- All SAVE_CONFIG values preserved exactly
- All includes preserved (mainsail, timelapse, get_ip, plr, Macro.cfg, moonraker_obico_macros, GP3D_Macro.cfg)

⚠️ **Macro conflict note:** `Macro.cfg` (included but not in this repo) may define `START_PRINT`. Our new `PRINT_START` is a different name so no conflict. If Macro.cfg also defines `PRINT_START`, Klipper will error — in that case add `rename_existing: PRINT_START_BASE` to our macro.

```ini
[include mainsail.cfg]
[include timelapse.cfg]
[include get_ip.cfg]
[include plr.cfg]
[include Macro.cfg]
[include moonraker_obico_macros.cfg]
[include GP3D_Macro.cfg]

[mcu]
serial: /dev/serial/by-id/usb-Klipper_stm32f103xe_36FFDA05334D413227670651-if00
restart_method: command

[mcu extra_mcu]
serial: /dev/serial/by-id/usb-Klipper_stm32f103xe_31FF6E064759343944131957-if00
restart_method: command

[adxl345]
cs_pin: extra_mcu:PB12
axes_map: x,z,y

[exclude_object]

[resonance_tester]
accel_chip: adxl345
probe_points:
    175, 175, 30
accel_per_hz: 50
min_freq: 5
max_freq: 133
max_smoothing: 0.2
hz_per_sec: 0.5

[temperature_sensor mcu_temp]
sensor_type: temperature_mcu
min_temp: 0
max_temp: 100

[temperature_sensor Host_temp]
sensor_type: temperature_host
min_temp: 0
max_temp: 110

[temperature_sensor Toolhead_Temp]
sensor_type: temperature_mcu
sensor_mcu: extra_mcu

[virtual_sdcard]
path: /home/sovol/printer_data/gcodes/

[pause_resume]

[printer]
kinematics: corexy
max_velocity: 700
max_accel: 40000
max_accel_to_decel: 10000
max_z_velocity: 20
max_z_accel: 500
square_corner_velocity: 5.0

[input_shaper]
damping_ratio_x: 0.1
damping_ratio_y: 0.1

[probe]
pin: extra_mcu:PB6
x_offset: -17
y_offset: 10
speed: 5.0
samples: 2
sample_retract_dist: 2.0
lift_speed: 50
samples_result: average
samples_tolerance: 0.016
samples_tolerance_retries: 2

[probe_pressure]
pin: ^!PE12
x_offset: 0
y_offset: 0
z_offset: 0
speed: 1.0

[homing_override]
gcode:
   {% if not 'Z' in params and not 'Y' in params and 'X' in params %}
     G28 X
     G0 X348 F1200
   {% elif not 'Z' in params and not 'X' in params and 'Y' in params %}
     G28 Y
     G0 Y360  F1200
   {% elif not 'Z' in params and 'X' in params and 'Y' in params %}
     G28 Y
     G0 Y360  F1200
     G4 P2000
     G28 X
     G0 X348  F1200
   {% elif 'Z' in params and not 'X' in params and not 'Y' in params %}
     G90
     G0  X191 Y165 F3600
     G28 Z
     G0  Z10 F600
   {% else %}
     G90
     G0 Z5 F300
     G28 Y
     G0 Y360  F1200
     G4 P2000
     G28 X
     G0 X348  F1200
     G90
     G0  X191 Y165 F3600
     G28 Z
     G0  Z10 F600
   {% endif %}
axes: xyz
set_position_z: 0

[bed_mesh]
speed: 500
horizontal_move_z: 5
mesh_min: 10, 10
mesh_max: 333, 340
probe_count: 9, 9
algorithm: bicubic
bicubic_tension: 0.4
split_delta_z: 0.016
mesh_pps: 3, 3
adaptive_margin: 5
fade_start: 0
fade_end: 10
fade_target: 0

[quad_gantry_level]
gantry_corners:
    -60, -10
    410, 420
points:
    36, 10
    36, 320
    346, 320
    346, 10
speed: 400
horizontal_move_z: 10
retry_tolerance: 0.05
retries: 5
max_adjust: 30

[z_offset_calibration]
center_xy_position: 191, 165
endstop_xy_position: 289, 361
z_hop: 10
z_hop_speed: 15

[stepper_x]
step_pin: PE2
dir_pin: !PE0
enable_pin: !PE3
rotation_distance: 40
microsteps: 16
full_steps_per_rotation: 200
endstop_pin: tmc2209_stepper_x:virtual_endstop
position_min: 0
position_endstop: 355
position_max: 355
homing_speed: 30
homing_retract_dist: 0
homing_positive_dir: True

[tmc2209 stepper_x]
uart_pin: PE1
interpolate: True
run_current: 1.5
sense_resistor: 0.150
stealthchop_threshold: 0
uart_address: 3
driver_sgthrs: 65
diag_pin: PE15

[stepper_y]
step_pin: PB8
dir_pin: !PB6
enable_pin: !PB9
rotation_distance: 40
microsteps: 16
full_steps_per_rotation: 200
endstop_pin: tmc2209_stepper_y:virtual_endstop
position_min: 0
position_endstop: 364
position_max: 364
homing_speed: 30
homing_retract_dist: 0
homing_positive_dir: true

[tmc2209 stepper_y]
uart_pin: PB7
interpolate: True
run_current: 1.5
sense_resistor: 0.150
stealthchop_threshold: 0
uart_address: 3
driver_sgthrs: 65
diag_pin: PE13

[stepper_z]
step_pin: PC0
dir_pin: PE5
enable_pin: !PC1
rotation_distance: 40
gear_ratio: 80:12
microsteps: 16
endstop_pin: probe:z_virtual_endstop
position_max: 347
position_min: -5
homing_speed: 15.0
homing_retract_dist: 5.0
homing_retract_speed: 15.0
second_homing_speed: 10.0

[tmc2209 stepper_z]
uart_pin: PE6
interpolate: true
run_current: 0.58
hold_current: 0.58
sense_resistor: 0.150
stealthchop_threshold: 999999
uart_address: 3

[stepper_z1]
step_pin: PD3
dir_pin: !PD1
enable_pin: !PD4
rotation_distance: 40
gear_ratio: 80:12
microsteps: 16

[tmc2209 stepper_z1]
uart_pin: PD2
interpolate: true
run_current: 0.58
hold_current: 0.58
sense_resistor: 0.150
stealthchop_threshold: 999999
uart_address: 3

[stepper_z2]
step_pin: PD7
dir_pin: PD5
enable_pin: !PB5
rotation_distance: 40
gear_ratio: 80:12
microsteps: 16

[tmc2209 stepper_z2]
uart_pin: PD6
interpolate: true
run_current: 0.58
hold_current: 0.58
sense_resistor: 0.150
stealthchop_threshold: 999999
uart_address: 3

[stepper_z3]
step_pin: PD11
dir_pin: !PD9
enable_pin: !PD12
rotation_distance: 40
gear_ratio: 80:12
microsteps: 16

[tmc2209 stepper_z3]
uart_pin: PD10
interpolate: true
run_current: 0.58
hold_current: 0.58
sense_resistor: 0.150
stealthchop_threshold: 999999
uart_address: 3

[thermistor my_thermistor_e]
temperature1: 25
resistance1: 110000
temperature2: 100
resistance2: 7008
temperature3: 220
resistance3: 435

[extruder]
step_pin: extra_mcu:PA8
dir_pin: extra_mcu:PB8
enable_pin: !extra_mcu:PB11
rotation_distance: 6.5
microsteps: 16
full_steps_per_rotation: 200
nozzle_diameter: 0.400
filament_diameter: 1.75
max_extrude_only_distance: 150
heater_pin: extra_mcu:PB9
sensor_type: my_thermistor_e
pullup_resistor: 11500
sensor_pin: extra_mcu:PA5
min_temp: 5
max_temp: 305
max_power: 1.0
min_extrude_temp: 150
control: pid
pid_kp: 33.838
pid_ki: 5.223
pid_kd: 47.752
pressure_advance: 0.025
pressure_advance_smooth_time: 0.035

[tmc2209 extruder]
uart_pin: extra_mcu:PB10
interpolate: True
run_current: 0.8
hold_current: 0.8
uart_address: 3
sense_resistor: 0.150

[verify_heater extruder]
max_error: 120
check_gain_time: 30
hysteresis: 5
heating_gain: 2

[filament_switch_sensor filament_sensor]
pause_on_runout: True
event_delay: 3.0
pause_delay: 0.5
switch_pin: PE9

[thermistor my_thermistor]
temperature1: 25
resistance1: 100000
temperature2: 50
resistance2: 18085.4
temperature3: 100
resistance3: 5362.6

[heater_bed]
heater_pin: PA0
sensor_type: my_thermistor
sensor_pin: PC5
max_power: 1.0
min_temp: 5
max_temp: 105
control: pid
pid_kp: 73.571
pid_ki: 1.820
pid_kd: 783.849

[verify_heater heater_bed]
max_error: 120
check_gain_time: 40
hysteresis: 5
heating_gain: 2

[fan_generic fan0]
pin: extra_mcu:PA7
max_power: 1.0

[fan_generic fan1]
pin: extra_mcu:PB1
max_power: 1.0

[fan_generic fan3]
pin: PA2
max_power: 1.0

[heater_fan hotend_fan]
pin: extra_mcu:PA6
max_power: 1.0
kick_start_time: 0.5
heater: extruder
heater_temp: 45
tachometer_pin: extra_mcu:PA1
tachometer_ppr: 1
tachometer_poll_interval: 0.0013

[board_pins]
aliases:
    EXP1_1=PA8,   EXP1_2=PC9,
    EXP1_3=PA10,  EXP1_4=PA9,
    EXP1_5=PC11,  EXP1_6=PC10,
    EXP1_7=PD8,   EXP1_8=PC12,
    EXP1_9=<GND>, EXP1_10=<5V>,
    EXP2_1=PB14,  EXP2_2=PB13,
    EXP2_3=PC7,   EXP2_4=PB12,
    EXP2_5=PC6,   EXP2_6=PB15,
    EXP2_7=PC8,   EXP2_8=<RST>,
    EXP2_9=<GND>, EXP2_10=<5V>

[display]
lcd_type: uc1701
cs_pin: EXP1_3
a0_pin: EXP1_4
rst_pin: EXP1_5
encoder_pins: ^EXP2_5, ^EXP2_3
click_pin: ^!EXP1_2
contrast: 63
spi_software_miso_pin: EXP2_1
spi_software_mosi_pin: EXP2_6
spi_software_sclk_pin: EXP2_2

[output_pin beeper]
pin: EXP1_1
pwm: False
value: 0

[neopixel Screen_Colour]
pin: EXP1_6
chain_count: 3
color_order: RGB
initial_RED: 0.5
initial_GREEN: 0.4
initial_BLUE: 0.7

[gcode_arcs]
resolution: 0.1

[output_pin main_led]
pin: PA3
pwm: 1
value: 1
cycle_time: 0.010

[idle_timeout]
gcode: _IDLE_TIMEOUT
timeout: 3600

[save_variables]
filename: /home/sovol/printer_data/config/saved_variables.cfg

#############################################################################################################
# GCODE MACROS
#############################################################################################################

[gcode_macro PRINT_START]
description: Full print start — heat bed, home, QGL, mesh, purge, heat extruder
gcode:
    {% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(200)|float %}
    {% set BED_TEMP = params.BED_TEMP|default(60)|float %}
    M140 S{BED_TEMP}                    ; start bed heating (don't wait yet)
    G28                                 ; home all (triggers homing_override)
    M190 S{BED_TEMP}                    ; wait for bed — warm bed before QGL for consistency
    QUAD_GANTRY_LEVEL                   ; level the gantry
    G28 Z                               ; re-home Z after QGL
    BED_MESH_CALIBRATE                  ; probe fresh mesh
    G90
    G1 X5 Y10 Z5 F9000                  ; move to purge start position
    M109 S{EXTRUDER_TEMP}               ; heat extruder
    G92 E0
    G1 Z0.3 F600                        ; drop to purge height
    G1 Y100 E20 F500                    ; purge line forward
    G1 Y110 E2 F500                     ; short extra prime
    G92 E0
    G1 Z2.0 F3000                       ; raise Z before print

[gcode_macro PRINT_END]
description: Full print end — retract, park, shutdown
gcode:
    G91
    G1 E-2 F500                         ; retract
    G1 Z10 F3000                        ; raise Z
    G90
    G1 X175 Y10 F9000                   ; park at front center
    SET_FAN_SPEED FAN=fan0 SPEED=0
    SET_FAN_SPEED FAN=fan1 SPEED=0
    TURN_OFF_HEATERS
    M84

[gcode_macro LOAD_FILAMENT]
description: Load filament — TEMP=220
gcode:
    {% set TEMP = params.TEMP|default(220)|float %}
    M104 S{TEMP}
    RESPOND TYPE=echo MSG="Heating to {TEMP}C..."
    M109 S{TEMP}
    G91
    G1 E30 F300
    G1 E10 F150
    G90
    RESPOND TYPE=echo MSG="Filament loaded!"

[gcode_macro UNLOAD_FILAMENT]
description: Unload filament — TEMP=220
gcode:
    {% set TEMP = params.TEMP|default(220)|float %}
    M104 S{TEMP}
    RESPOND TYPE=echo MSG="Heating to {TEMP}C..."
    M109 S{TEMP}
    G91
    G1 E-30 F300
    G90
    RESPOND TYPE=echo MSG="Filament unloaded!"

#*# <---------------------- SAVE_CONFIG ---------------------->
#*# DO NOT EDIT THIS BLOCK OR BELOW. The contents are auto-generated.
#*#
#*# [probe]
#*# z_offset = 1.755
#*#
#*# [bed_mesh default]
#*# version = 1
#*# points =
#*# 	-0.114094, -0.111281, -0.063469, -0.011906, 0.011531, 0.013406, -0.016594, -0.069094, -0.136594
#*# 	-0.052219, -0.076594, -0.039094, 0.007781, 0.030281, 0.032156, 0.016219, -0.018469, -0.065344
#*# 	0.038719, 0.007781, 0.029344, 0.064969, 0.090281, 0.094031, 0.080906, 0.053719, 0.016219
#*# 	0.071531, 0.031219, 0.052781, 0.080906, 0.097781, 0.096844, 0.081844, 0.055594, 0.024656
#*# 	0.049031, 0.025594, 0.043406, 0.073406, 0.094969, 0.094031, 0.069656, 0.027469, 0.016219
#*# 	0.050906, 0.035906, 0.039656, 0.078094, 0.094969, 0.084656, 0.066844, 0.035906, 0.009656
#*# 	0.007781, -0.008156, -0.002531, 0.026531, 0.034031, 0.041531, 0.022781, -0.005344, -0.024094
#*# 	-0.054094, -0.095344, -0.084094, -0.060656, -0.054094, -0.055031, -0.062531, -0.078469, -0.087844
#*# 	-0.057844, -0.100969, -0.107531, -0.095344, -0.086906, -0.092531, -0.100969, -0.110344, -0.099094
#*# x_count = 9
#*# y_count = 9
#*# mesh_x_pps = 3
#*# mesh_y_pps = 3
#*# algo = bicubic
#*# tension = 0.4
#*# min_x = 10.0
#*# max_x = 332.96
#*# min_y = 10.0
#*# max_y = 340.0
#*#
#*# [input_shaper]
#*# shaper_type_x = 3hump_ei
#*# shaper_freq_x = 64.2
#*# shaper_type_y = zv
#*# shaper_freq_y = 38.8
```

**Step 2: Verify**

Check:
- `damping_ratio_x: 0.1` and `damping_ratio_y: 0.1` (not 0.001)
- `[probe]` has only one `speed:` line (5.0)
- `[gcode_arcs] resolution: 0.1` (not 1.0)
- `resonance_tester` min_freq: 5, max_freq: 133
- `PRINT_START` has `QUAD_GANTRY_LEVEL` and `BED_MESH_CALIBRATE`
- SAVE_CONFIG block matches original exactly

**Step 3: Commit**
```bash
git add "Printer_Backup/Sv08_optimized/printer.cfg"
git commit -m "feat: SV08 optimized printer.cfg — fixed probe dupe, damping ratio, gcode_arcs, macros"
```

---

### Task 4: SV07+ OrcaSlicer printer profile

**Files:**
- Read: `Printer_Backup/Sv07+/Sovol SV07 Plus 0.4 nozzle - Copy.json`
- Create: `Printer_Backup/Sv07+_optimized/printer_profile.json`

**Step 1:** Copy the original file, then change exactly these fields:

```json
"name": "Sovol SV07 Plus 0.4 nozzle Optimized",
"printer_settings_id": "Sovol SV07 Plus 0.4 nozzle Optimized",
"machine_start_gcode": "PRINT_START EXTRUDER_TEMP=[nozzle_temperature_initial_layer] BED_TEMP=[bed_temperature_initial_layer_single]",
"machine_end_gcode": "PRINT_END",
"default_print_profile": "0.20mm Balanced @SovolSV07Plus"
```

Everything else (bed size 300×300, retraction, machine limits, inherits, etc.) stays identical to the original.

**Step 2: Commit**
```bash
git add "Printer_Backup/Sv07+_optimized/printer_profile.json"
git commit -m "feat: SV07+ OrcaSlicer printer profile — delegate to PRINT_START/PRINT_END macros"
```

---

### Task 5: SV07+ process profiles

**Files:**
- Create: `Printer_Backup/Sv07+_optimized/quality.json`
- Create: `Printer_Backup/Sv07+_optimized/balanced.json`
- Create: `Printer_Backup/Sv07+_optimized/speed.json`
- Create: `Printer_Backup/Sv07+_optimized/print_in_place.json`

**Step 1: Write quality.json**

Input shaper MZV@32.0Hz (Y limiting) → quality accel: outer 1200, default 1500.

```json
{
    "bottom_shell_layers": "5",
    "bridge_flow": "0.95",
    "bridge_speed": "25",
    "brim_type": "no_brim",
    "default_acceleration": "1500",
    "elefant_foot_compensation": "0.1",
    "from": "User",
    "inherits": "0.20mm Standard @Sovol SV07 Plus",
    "initial_layer_infill_speed": "50",
    "initial_layer_line_width": "0.5",
    "inner_wall_acceleration": "1500",
    "inner_wall_line_width": "0.45",
    "inner_wall_speed": "60",
    "internal_solid_infill_line_width": "0.4",
    "line_width": "0.4",
    "minimum_sparse_infill_area": "10",
    "name": "0.20mm Quality @SovolSV07Plus",
    "only_one_wall_top": "1",
    "outer_wall_acceleration": "1200",
    "outer_wall_line_width": "0.4",
    "outer_wall_speed": "40",
    "overhang_2_4_speed": "20",
    "overhang_3_4_speed": "15",
    "print_settings_id": "0.20mm Quality @SovolSV07Plus",
    "reduce_crossing_wall": "1",
    "sparse_infill_line_width": "0.45",
    "sparse_infill_pattern": "honeycomb",
    "sparse_infill_speed": "100",
    "support_line_width": "0.4",
    "support_on_build_plate_only": "1",
    "support_speed": "60",
    "support_style": "organic",
    "support_top_z_distance": "0.25",
    "support_type": "tree(auto)",
    "top_shell_layers": "5",
    "top_surface_acceleration": "1200",
    "top_surface_line_width": "0.4",
    "top_surface_speed": "40",
    "travel_acceleration": "1500",
    "travel_speed": "150",
    "version": "2.3.1.10",
    "wall_loops": "3",
    "wall_sequence": "outer wall/inner wall"
}
```

**Step 2: Write balanced.json**

```json
{
    "bridge_flow": "0.95",
    "default_acceleration": "3000",
    "elefant_foot_compensation": "0.1",
    "from": "User",
    "inherits": "0.20mm Standard @Sovol SV07 Plus",
    "initial_layer_line_width": "0.5",
    "inner_wall_acceleration": "2500",
    "inner_wall_line_width": "0.45",
    "internal_solid_infill_line_width": "0.4",
    "line_width": "0.4",
    "minimum_sparse_infill_area": "10",
    "name": "0.20mm Balanced @SovolSV07Plus",
    "only_one_wall_top": "1",
    "outer_wall_acceleration": "2500",
    "outer_wall_line_width": "0.4",
    "print_settings_id": "0.20mm Balanced @SovolSV07Plus",
    "reduce_crossing_wall": "1",
    "sparse_infill_line_width": "0.45",
    "top_surface_acceleration": "2500",
    "top_surface_line_width": "0.4",
    "travel_acceleration": "3000",
    "travel_speed": "200",
    "version": "2.3.1.10",
    "wall_loops": "3",
    "wall_sequence": "outer wall/inner wall"
}
```

**Step 3: Write speed.json**

```json
{
    "bottom_shell_layers": "3",
    "bridge_flow": "0.95",
    "default_acceleration": "3400",
    "elefant_foot_compensation": "0.1",
    "from": "User",
    "inherits": "0.20mm Standard @Sovol SV07 Plus",
    "initial_layer_line_width": "0.5",
    "inner_wall_acceleration": "3400",
    "inner_wall_line_width": "0.45",
    "internal_solid_infill_line_width": "0.45",
    "line_width": "0.45",
    "minimum_sparse_infill_area": "10",
    "name": "0.20mm Speed @SovolSV07Plus",
    "only_one_wall_top": "1",
    "outer_wall_acceleration": "3000",
    "outer_wall_line_width": "0.4",
    "print_settings_id": "0.20mm Speed @SovolSV07Plus",
    "reduce_crossing_wall": "1",
    "sparse_infill_line_width": "0.5",
    "top_shell_layers": "4",
    "top_surface_acceleration": "3000",
    "top_surface_line_width": "0.4",
    "travel_acceleration": "3400",
    "travel_speed": "250",
    "version": "2.3.1.10",
    "wall_loops": "2",
    "wall_sequence": "outer wall/inner wall"
}
```

**Step 4: Write print_in_place.json**

Self-contained (no inherits) — conservative accel, precise outer wall, seam slopes, brim.

```json
{
    "bottom_shell_layers": "5",
    "bridge_density": "100%",
    "bridge_flow": "0.95",
    "bridge_speed": "25",
    "brim_object_gap": "0.22",
    "brim_type": "outer_only",
    "brim_width": "3",
    "default_acceleration": "800",
    "detect_thin_wall": "1",
    "elefant_foot_compensation": "0.15",
    "enable_arc_fitting": "1",
    "enable_overhang_bridge_fan": "1",
    "fan_max_speed": "100",
    "fan_min_speed": "30",
    "from": "User",
    "gap_infill_speed": "50",
    "infill_jerk": "5",
    "inherits": "",
    "initial_layer_acceleration": "300",
    "initial_layer_infill_speed": "30",
    "initial_layer_jerk": "5",
    "initial_layer_line_width": "0.5",
    "initial_layer_print_height": "0.2",
    "initial_layer_speed": "20",
    "inner_wall_acceleration": "800",
    "inner_wall_jerk": "5",
    "inner_wall_line_width": "0.42",
    "inner_wall_speed": "40",
    "internal_solid_infill_line_width": "0.42",
    "internal_solid_infill_pattern": "rectilinear",
    "internal_solid_infill_speed": "60",
    "is_custom_defined": "0",
    "layer_height": "0.20",
    "line_width": "0.42",
    "name": "0.20mm PRINT-IN-PLACE @SovolSV07Plus",
    "only_one_wall_top": "0",
    "outer_wall_acceleration": "600",
    "outer_wall_jerk": "3",
    "outer_wall_line_width": "0.40",
    "outer_wall_speed": "30",
    "overhang_2_4_speed": "25",
    "overhang_3_4_speed": "15",
    "overhang_4_4_speed": "10",
    "precise_outer_wall": "1",
    "print_settings_id": "0.20mm PRINT-IN-PLACE @SovolSV07Plus",
    "reduce_crossing_wall": "1",
    "seam_position": "aligned",
    "seam_slope_conditional": "1",
    "seam_slope_entire_loop": "1",
    "seam_slope_inner_walls": "1",
    "skirt_loops": "2",
    "slow_down_for_layer_cooling": "1",
    "slow_down_min_speed": "20",
    "sparse_infill_density": "15%",
    "sparse_infill_pattern": "grid",
    "sparse_infill_speed": "70",
    "support_bottom_z_distance": "0.25",
    "support_line_width": "0.38",
    "support_on_build_plate_only": "0",
    "support_speed": "60",
    "support_style": "organic",
    "support_top_z_distance": "0.3",
    "support_type": "tree(auto)",
    "top_shell_layers": "5",
    "top_shell_thickness": "1.0",
    "top_surface_acceleration": "600",
    "top_surface_jerk": "3",
    "top_surface_line_width": "0.42",
    "top_surface_pattern": "monotoniclines",
    "top_surface_speed": "30",
    "travel_acceleration": "1500",
    "travel_jerk": "7",
    "travel_speed": "150",
    "version": "2.3.1.10",
    "wall_loops": "4",
    "wall_sequence": "inner wall/outer wall"
}
```

**Step 5: Commit**
```bash
git add "Printer_Backup/Sv07+_optimized/"
git commit -m "feat: SV07+ OrcaSlicer process profiles — quality/balanced/speed/print-in-place"
```

---

### Task 6: SV08 OrcaSlicer printer profile

**Files:**
- Read: `Printer_Backup/Sv08/Sovol SV08 0.4 nozzle - Copy.json`
- Create: `Printer_Backup/Sv08_optimized/printer_profile.json`

**Step 1:** Copy the original file, then change exactly these fields:

```json
"name": "Sovol SV08 0.4 nozzle Optimized",
"printer_settings_id": "Sovol SV08 0.4 nozzle Optimized",
"machine_start_gcode": "PRINT_START EXTRUDER_TEMP=[nozzle_temperature_initial_layer] BED_TEMP=[bed_temperature_initial_layer_single]",
"machine_end_gcode": "PRINT_END",
"before_layer_change_gcode": "G92 E0",
"default_print_profile": "0.20mm Balanced @SovolSV08"
```

Note: `before_layer_change_gcode` is changed from `TIMELAPSE_TAKE_FRAME\nG92 E0` to just `G92 E0` — removes timelapse dependency (if they want timelapse they can re-add it).

Everything else (bed size 350×350, retraction, machine limits, etc.) stays identical.

**Step 2: Commit**
```bash
git add "Printer_Backup/Sv08_optimized/printer_profile.json"
git commit -m "feat: SV08 OrcaSlicer printer profile — delegate to PRINT_START/PRINT_END macros"
```

---

### Task 7: SV08 process profiles

**Files:**
- Create: `Printer_Backup/Sv08_optimized/quality.json`
- Create: `Printer_Backup/Sv08_optimized/balanced.json`
- Create: `Printer_Backup/Sv08_optimized/speed.json`
- Create: `Printer_Backup/Sv08_optimized/print_in_place.json`

Input shaper ZV@38.8Hz (Y, limiting) → practical accel ceiling ~8,000 mm/s².
Speeds are higher than SV07+ because CoreXY handles them better.

**Step 1: Write quality.json**

```json
{
    "bottom_shell_layers": "5",
    "bridge_flow": "0.95",
    "bridge_speed": "25",
    "brim_type": "no_brim",
    "default_acceleration": "3000",
    "elefant_foot_compensation": "0.1",
    "from": "User",
    "inherits": "0.20mm Standard @Sovol SV08",
    "initial_layer_infill_speed": "50",
    "initial_layer_line_width": "0.5",
    "inner_wall_acceleration": "3000",
    "inner_wall_line_width": "0.45",
    "inner_wall_speed": "80",
    "internal_solid_infill_line_width": "0.4",
    "line_width": "0.4",
    "minimum_sparse_infill_area": "10",
    "name": "0.20mm Quality @SovolSV08",
    "only_one_wall_top": "1",
    "outer_wall_acceleration": "2500",
    "outer_wall_line_width": "0.4",
    "outer_wall_speed": "60",
    "overhang_2_4_speed": "30",
    "overhang_3_4_speed": "20",
    "print_settings_id": "0.20mm Quality @SovolSV08",
    "reduce_crossing_wall": "1",
    "sparse_infill_line_width": "0.45",
    "sparse_infill_pattern": "honeycomb",
    "sparse_infill_speed": "200",
    "support_line_width": "0.4",
    "support_on_build_plate_only": "1",
    "support_speed": "80",
    "support_style": "organic",
    "support_top_z_distance": "0.25",
    "support_type": "tree(auto)",
    "top_shell_layers": "5",
    "top_surface_acceleration": "2500",
    "top_surface_line_width": "0.4",
    "top_surface_speed": "60",
    "travel_acceleration": "3000",
    "travel_speed": "300",
    "version": "2.3.1.10",
    "wall_loops": "3",
    "wall_sequence": "outer wall/inner wall"
}
```

**Step 2: Write balanced.json**

```json
{
    "bridge_flow": "0.95",
    "default_acceleration": "6000",
    "elefant_foot_compensation": "0.1",
    "from": "User",
    "inherits": "0.20mm Standard @Sovol SV08",
    "initial_layer_line_width": "0.5",
    "inner_wall_acceleration": "5000",
    "inner_wall_line_width": "0.45",
    "internal_solid_infill_line_width": "0.4",
    "line_width": "0.4",
    "minimum_sparse_infill_area": "10",
    "name": "0.20mm Balanced @SovolSV08",
    "only_one_wall_top": "1",
    "outer_wall_acceleration": "5000",
    "outer_wall_line_width": "0.4",
    "print_settings_id": "0.20mm Balanced @SovolSV08",
    "reduce_crossing_wall": "1",
    "sparse_infill_line_width": "0.45",
    "top_surface_acceleration": "5000",
    "top_surface_line_width": "0.4",
    "travel_acceleration": "6000",
    "travel_speed": "400",
    "version": "2.3.1.10",
    "wall_loops": "3",
    "wall_sequence": "outer wall/inner wall"
}
```

**Step 3: Write speed.json**

```json
{
    "bottom_shell_layers": "3",
    "bridge_flow": "0.95",
    "default_acceleration": "8000",
    "elefant_foot_compensation": "0.1",
    "from": "User",
    "inherits": "0.20mm Standard @Sovol SV08",
    "initial_layer_line_width": "0.5",
    "inner_wall_acceleration": "8000",
    "inner_wall_line_width": "0.45",
    "internal_solid_infill_line_width": "0.45",
    "line_width": "0.45",
    "minimum_sparse_infill_area": "10",
    "name": "0.20mm Speed @SovolSV08",
    "only_one_wall_top": "1",
    "outer_wall_acceleration": "7000",
    "outer_wall_line_width": "0.4",
    "print_settings_id": "0.20mm Speed @SovolSV08",
    "reduce_crossing_wall": "1",
    "sparse_infill_line_width": "0.5",
    "top_shell_layers": "4",
    "top_surface_acceleration": "7000",
    "top_surface_line_width": "0.4",
    "travel_acceleration": "8000",
    "travel_speed": "500",
    "version": "2.3.1.10",
    "wall_loops": "2",
    "wall_sequence": "outer wall/inner wall"
}
```

**Step 4: Write print_in_place.json**

```json
{
    "bottom_shell_layers": "5",
    "bridge_density": "100%",
    "bridge_flow": "0.95",
    "bridge_speed": "30",
    "brim_object_gap": "0.22",
    "brim_type": "outer_only",
    "brim_width": "3",
    "default_acceleration": "1500",
    "detect_thin_wall": "1",
    "elefant_foot_compensation": "0.15",
    "enable_arc_fitting": "1",
    "enable_overhang_bridge_fan": "1",
    "fan_max_speed": "100",
    "fan_min_speed": "30",
    "from": "User",
    "gap_infill_speed": "60",
    "infill_jerk": "5",
    "inherits": "",
    "initial_layer_acceleration": "500",
    "initial_layer_infill_speed": "40",
    "initial_layer_jerk": "5",
    "initial_layer_line_width": "0.5",
    "initial_layer_print_height": "0.2",
    "initial_layer_speed": "25",
    "inner_wall_acceleration": "1500",
    "inner_wall_jerk": "5",
    "inner_wall_line_width": "0.42",
    "inner_wall_speed": "50",
    "internal_solid_infill_line_width": "0.42",
    "internal_solid_infill_pattern": "rectilinear",
    "internal_solid_infill_speed": "80",
    "is_custom_defined": "0",
    "layer_height": "0.20",
    "line_width": "0.42",
    "name": "0.20mm PRINT-IN-PLACE @SovolSV08",
    "only_one_wall_top": "0",
    "outer_wall_acceleration": "1000",
    "outer_wall_jerk": "3",
    "outer_wall_line_width": "0.40",
    "outer_wall_speed": "40",
    "overhang_2_4_speed": "30",
    "overhang_3_4_speed": "20",
    "overhang_4_4_speed": "10",
    "precise_outer_wall": "1",
    "print_settings_id": "0.20mm PRINT-IN-PLACE @SovolSV08",
    "reduce_crossing_wall": "1",
    "seam_position": "aligned",
    "seam_slope_conditional": "1",
    "seam_slope_entire_loop": "1",
    "seam_slope_inner_walls": "1",
    "skirt_loops": "2",
    "slow_down_for_layer_cooling": "1",
    "slow_down_min_speed": "25",
    "sparse_infill_density": "15%",
    "sparse_infill_pattern": "grid",
    "sparse_infill_speed": "100",
    "support_bottom_z_distance": "0.25",
    "support_line_width": "0.38",
    "support_on_build_plate_only": "0",
    "support_speed": "80",
    "support_style": "organic",
    "support_top_z_distance": "0.3",
    "support_type": "tree(auto)",
    "top_shell_layers": "5",
    "top_shell_thickness": "1.0",
    "top_surface_acceleration": "1000",
    "top_surface_jerk": "3",
    "top_surface_line_width": "0.42",
    "top_surface_pattern": "monotoniclines",
    "top_surface_speed": "40",
    "travel_acceleration": "3000",
    "travel_jerk": "7",
    "travel_speed": "300",
    "version": "2.3.1.10",
    "wall_loops": "4",
    "wall_sequence": "inner wall/outer wall"
}
```

**Step 5: Commit**
```bash
git add "Printer_Backup/Sv08_optimized/"
git commit -m "feat: SV08 OrcaSlicer process profiles — quality/balanced/speed/print-in-place"
```

---

### Task 8: Final verification

**Step 1: Confirm output structure**
```bash
find Printer_Backup/Sv07+_optimized Printer_Backup/Sv08_optimized -type f | sort
```

Expected output:
```
Printer_Backup/Sv07+_optimized/balanced.json
Printer_Backup/Sv07+_optimized/print_in_place.json
Printer_Backup/Sv07+_optimized/printer.cfg
Printer_Backup/Sv07+_optimized/printer_profile.json
Printer_Backup/Sv07+_optimized/quality.json
Printer_Backup/Sv07+_optimized/speed.json
Printer_Backup/Sv08_optimized/balanced.json
Printer_Backup/Sv08_optimized/print_in_place.json
Printer_Backup/Sv08_optimized/printer.cfg
Printer_Backup/Sv08_optimized/printer_profile.json
Printer_Backup/Sv08_optimized/quality.json
Printer_Backup/Sv08_optimized/speed.json
```

**Step 2: Spot-check key values**

SV07+ printer.cfg — must NOT contain:
```bash
grep -n "max_accel_to_decel\|M205\|speed: 15" "Printer_Backup/Sv07+_optimized/printer.cfg"
```
Expected: no output.

SV08 printer.cfg — must NOT contain:
```bash
grep -n "damping_ratio.*0\.001\|speed: 15\|resolution: 1\.0" "Printer_Backup/Sv08_optimized/printer.cfg"
```
Expected: no output.

Check SAVE_CONFIG blocks survived intact:
```bash
grep "z_offset" "Printer_Backup/Sv07+_optimized/printer.cfg"
grep "z_offset" "Printer_Backup/Sv08_optimized/printer.cfg"
```
Expected: `z_offset = 1.800` and `z_offset = 1.755` respectively.

**Step 3: Final commit**
```bash
git add -A
git commit -m "feat: complete SV07+ and SV08 optimized configs and OrcaSlicer profiles"
```

---

## Handoff Notes for Recipient

Include these notes when sending the files:

**SV07+:**
- OrcaSlicer: import `printer_profile.json`, then all four process profiles. Set printer to "Sovol SV07 Plus 0.4 nozzle Optimized". The process profiles inherit from "0.20mm Standard @Sovol SV07 Plus" — make sure Sovol profiles are installed in OrcaSlicer first.
- Klipper: replace `printer.cfg` on the Pi. The PLR (power loss recovery) include is preserved — ensure `plr.cfg` is still present.
- First print: run `SCREWS_TILT_ADJUST` via Mainsail console to check bed level, then run a test print with the Balanced profile.

**SV08:**
- OrcaSlicer: same import process. Profiles inherit from "0.20mm Standard @Sovol SV08".
- Klipper: replace `printer.cfg`. The `Macro.cfg`, `GP3D_Macro.cfg`, and other includes must still be present on the Pi — we only replaced `printer.cfg`.
- ⚠️ If Klipper errors on startup with "PRINT_START already defined", open `printer.cfg`, find the `[gcode_macro PRINT_START]` section, and add `rename_existing: PRINT_START_ORIG` on the line after `[gcode_macro PRINT_START]`. This means `Macro.cfg` also defines it.
- First print: run `QUAD_GANTRY_LEVEL` via Mainsail to verify gantry leveling works, then test with Balanced profile.
