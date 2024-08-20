# Software Install: Klipper and Plugins

## Klipper Plugins

I installed Klipper, KlipperScreen, Moonraker, and Mainsail in the last entry but didn't configure any of them, and we'll also need to install some Klipper plugins to use the configs provided in the previous entry as well.

For each of these I followed the install process on their github pages, they're mostly just git checkouts and maybe symlinks, with some moonraker config to check for updates.

For the stock config, we need to add:
* [Cartographer](https://github.com/Cartographer3D/cartographer-klipper) for the leveling probe
* [LED Effect](https://github.com/julianschill/klipper-led_effect) to control LEDs

And then we'll also add in:
* [TMC Autotune](https://github.com/andrewmcgr/klipper_tmc_autotune) to fine-tune our stepper motors
* [Motors Sync](https://github.com/MRX8024/motors-sync) to synchronize the AWD steppers
* [KAMP](https://github.com/kyleisah/Klipper-Adaptive-Meshing-Purging) for some extra goodies (Haven't actually enabled anything in here yet though, Adaptive Meshing is already in Klipper)
* [Shaketune](https://github.com/Frix-x/klippain-shaketune) for input shaping and belt testing

## Klipper Config

I'm not going to go over every section of my config (they're [here](configs) for the whole set) but there's a few sections I'd like to call out.

### Which Accelerometer to Use

There are two ADXL345 chips to choose from: One on the cartographer, and the other on the SB2209 toolhead board.  I chose the toolhead board as its closer to the toolhead's center of gravity, and it was also a less-noisy sensor.

This config is found [here](configs/devices.cfg#L8-L18).

### Steppers

**Note that I'm using TMC5160T Pro stepper drivers for the 4 XY steppers while the stock BOM uses TMC2240's, so make sure to account for that if copying settings.**

Autotune requires motor definitions for the Siboor steppers we're using here (unless they've been merged to main by the time you read this, issue is [here](https://github.com/andrewmcgr/klipper_tmc_autotune/pull/189)).

```
[motor_constants siboor-14sth20-1004a]
#Siboor BOM Voron Trident AWD
resistance: 1.7
inductance: 0.0015
holding_torque: 0.12
max_current: 1.88
steps_per_revolution: 200

[motor_constants siboor-42sth40-2004a]
#Siboor BOM Voron Trident AWD
resistance: 1.1
inductance: 0.0026
holding_torque: 0.45
max_current: 2.0
steps_per_revolution: 200

[motor_constants siboor-42sth48-2504(s45)]
 #Siboor BOM Voron Trident AWD
resistance: 0.9
inductance: 0.0016
holding_torque: 0.6
max_current: 2.5
steps_per_revolution: 200

[autotune_tmc stepper_x]
motor: siboor-42sth48-2504(s45)
[autotune_tmc stepper_y]
motor: siboor-42sth48-2504(s45)
[autotune_tmc stepper_x1]
motor: siboor-42sth48-2504(s45)
[autotune_tmc stepper_y1]
motor: siboor-42sth48-2504(s45)
[autotune_tmc extruder]
motor: siboor-14sth20-1004a
```

Config [here](configs/steppers.cfg#L6-L39).

### Bed Mesh

The settings in the stock config are wrong for the 300mm bed, they'll result in the carto probing off the bed and it looking like your mesh has a steep dropoff.

```
[bed_mesh]
speed: 350                       # Calibration speed
horizontal_move_z: 10            # Z-axis movement speed
mesh_min: 45,25                  # Minimum calibration point coordinates x, y
mesh_max: 255,255                # Maximum calibration point coordinates x, y
probe_count: 10,10               # Number of sampling points (7X7 is 49 points)
mesh_pps: 2,2                    # Additional sampling points
algorithm: bicubic               # Algorithm model
bicubic_tension: 0.2             # Algorithm interpolation, no movement
```

Config [here](configs/calibration.cfg#L12-L20).

### Fans

The stock fan settings have some issues, and as it stands, the stock side skirt fans are too loud for me.  I'll be replacing them with some 24v GDSTIME 6020s.

The stock fan config also has issues, my fan config is [here](config/fans.cfg), as well as below.  Big thanks to **Marci** on the Siboor discord for helping out with these fan settings.

Fans `fan2` (the 12032 side blower) and `fan3` (the fume pack) are labeled as such because that way Orca Slicer can auto-control them.

```
[fan_generic fan2]               # Side blower fan
pin: PA0
kick_start_time: 10               # Startup time
off_below: 0.11                  # Turn off fan below this value
max_power: 1                     # Maximum power
cycle_time: 0.00003              # Cycle time

[gcode_macro M106]
gcode:
    {% set fan = 'fan' + (params.P|int if params.P is defined else 0)|string %}
    {% set speed = (params.S|float / 255 if params.S is defined else 1.0) %}
    SET_FAN_SPEED FAN={fan} SPEED={speed}

[fan_generic fan0]               # Part Cooling
pin:EBBCan:gpio13                # Fan pin
kick_start_time: 1.0             # Startup time
off_below: 0.10                  # Turn off fan below this value
max_power: 1                     # Maximum power

[heater_fan hotend_fan]
pin: EBBCan:gpio14               # Hotend fan pin
heater: extruder                 # Associated heating device
heater_temp: 50.0                # Temperature to start the fan

[fan_generic fan3]               # Fume Pack fan
pin: PF7                         # Nevermore fan pin
max_power: 1.0                   # Maximum power
kick_start_time: 1.0             # Startup time
shutdown_speed: 1.0

#--------------------------------------------------------------------

[temperature_fan skirt_fan]
pin: PF9
max_power: 0.3
shutdown_speed: 0
cycle_time: 0.00003
min_speed: 0.05
max_speed: 1.0
kick_start_time: 0.8
sensor_type: temperature_host    # Sensor type
control: pid                     # Control method
min_temp: 0                      # Minimum temperature
max_temp: 85                     # Maximum temperature
pid_kp: 1.0                      # PID Kp value
pid_ki: 0.5                      # PID Ki value
pid_kd: 2.0                      # PID Kd value
target_temp: 65                  # Target temperature

[controller_fan driver_fan]
pin: PF6
max_power: 0.6                   # Maximum power
shutdown_speed: 0.0              # Shutdown speed
fan_speed: 1.0                   # Fan speed when heater or stepper driver is active (0.0 to 1.0). Default is 1.0.
idle_timeout: 90                 # Time in seconds to keep the fan running after the stepper driver or heater is no longer active. Default is 30 seconds.
idle_speed: 0.5                  # Fan speed after the stepper driver is no longer active and before idle_timeout is reached (0.0 to 1.0). Default is fan_speed.
stepper: stepper_x, stepper_x1, stepper_y, stepper_y1, stepper_z, stepper_z1, stepper_z2
```

### PRINT_START macro
I'm using the following `PRINT_START` macro.  It does a motor sync (if needed, only needed once per machine startup) and an adaptive bed mesh over the printing area.

Config [here](configs/macros.cfg#L22-L67).

```
[gcode_macro PRINT_START]
gcode:
    BED_MESH_CLEAR

    {% set BED_TEMP = params.BED_TEMP|default(50)|float %}
    {% set EXTRUDER_TEMP = params.EXTRUDER_TEMP|default(200)|float %}
    
    M117 Heating bed...
    SET_HEATER_TEMPERATURE HEATER=heater_bed TARGET={BED_TEMP}
    SET_HEATER_TEMPERATURE HEATER=extruder TARGET={150}
    TEMPERATURE_WAIT SENSOR=heater_bed MINIMUM={BED_TEMP}
    
    M117 Homing...
    G28
      {% if not printer.motors_sync.applied %}
      SYNC_MOTORS
      {% endif%}
    G28 Z
    
    M117 Z_tilt_adjusting...
    Z_TILT_ADJUST
    
    M117 Heating nozzle...
    SET_HEATER_TEMPERATURE HEATER=extruder TARGET={EXTRUDER_TEMP}
    
    M117 Rehoming...
    G28 Z

    BED_MESH_CALIBRATE ADAPTIVE=1
    BED_MESH_PROFILE LOAD=default

    G90
    G1 X025  Y000 Z010 F7200
    G90
    
    M117 Heating nozzle...
    TEMPERATURE_WAIT SENSOR=extruder MINIMUM={EXTRUDER_TEMP}

    M117 Drawing lines...
    DRAW_LINES
    
    G91
    G1 Z5
    G90

    M117 # Clearing text messages from console
```

### DRAW_LINES macro

`DRAW_LINES` is a macro for drawing a purge line to clear the nozzle before the print starts.  The default `DRAW_LINES` macro has a couple issues: First, it extrudes 15mm of filament in midair, creating a little filament pile on the plate, and then it immediately smooshes the hotend directly into this pile.  Second, by default it prints at Y=0, which for me is the very edge of the build plate and the line doesn't really end up on the plate as a result.  Mine instead extrudes 5mm of filament (which is likely still overkill to clear oozing) and then moves over before moving close to the bed, and then also prints at Y=10 so it is 1cm into the plate instead of at the very edge.  Adaptive Purge from KAMP would also likely reduce the need for this at all.

Config [here](configs/macros.cfg#L6-L20).

```
[gcode_macro DRAW_LINES]
gcode:
    G90                           # Absolute positioning
    # G92 E0                      # Reset Extruder (commented out for now)
    # G1 Z5.0  F7200              # Move Z Axis up (commented out for now)
    G1 X25  Y10         F7200     # Move to start position
    M83                           # Set extruder to relative mode
    G1 E5 F400                    # Extrude filament
    G1 X50  Y10         F7200
    G1 Z0.28 F7200                # Lower Z axis
    G1 X200 Y10   Z0.28 F1200 E17  # Draw the first line
    G1 X200 Y10.4 Z0.28 F2400      # Move to side a little
    G1 X55  Y10.4 Z0.28 F1200 E34  # Draw the second line
    G92 E0                        # Reset Extruder
    G90                           # Return to absolute positioning
```