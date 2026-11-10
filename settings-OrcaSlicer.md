# OrcaSlicer settings — Ender-3 SKR 3.0

Taken from this repo's `Configuration.h` / `Configuration_adv.h` (branch `ender-bugfix-2.1.x`).
Board BTT SKR Mini E3 v3.0, TMC2209 drivers, BLTouch, 5×5 bilinear mesh, input shaping, linear advance.

> [!WARNING]
> **Settings that will break or slow prints if you leave the Orca defaults**
>
> - **Arc fitting must be OFF.** `ARC_SUPPORT` is off in the firmware, so G2/G3 moves are ignored.
> - **Retraction speed ≤ 25 mm/s.** The firmware's E max feedrate is 25 mm/s, so faster speeds get clamped.
> - **Accelerations ≤ 500 mm/s².** The firmware's max acceleration is 500, so higher values in Orca have no effect.
> - **Use firmware retraction: OFF.** `FWRETRACT` is not enabled, so G10/G11 do nothing.

## Printer settings › Basic information

| Setting | Value | Source / note |
| --- | --- | --- |
| Printable area | 235 × 222, origin 0,0 | `X_BED_SIZE 235`; `Y_MAX_POS = 235 − 13 = 222`. The nozzle can't reach the last 13 mm of Y. |
| Printable height | 250 mm | `Z_MAX_POS 250` |
| G-code flavor | Marlin 2 | Marlin bugfix-2.1.x |
| Use relative E distances | On | Matches the `M83` in the start G-code below |
| Use firmware retraction | Off | `FWRETRACT` disabled |
| Z offset | 0 | Set with BLTouch `M851` / babystep (`BABYSTEP_ZPROBE_OFFSET`), saved to EEPROM |
| Thumbnails | None | The CR10 stock display can't show thumbnails |
| Disable set remaining print time | Either | `SET_PROGRESS_MANUALLY` is on and `SET_REMAINING_TIME` is off, so the firmware ignores `M73 R` |

## Printer settings › Machine limits

Set **Usage of machine limits** to **Use for time estimate only** so the values in EEPROM stay in control. If you pick "Emit to G-code", use exactly these values.

| Setting | Value | Source |
| --- | --- | --- |
| Max speed X / Y | 500 / 500 mm/s | `DEFAULT_MAX_FEEDRATE` |
| Max speed Z | 5 mm/s | |
| Max speed E | 25 mm/s | |
| Max acceleration X / Y | 500 / 500 mm/s² | `DEFAULT_MAX_ACCELERATION` |
| Max acceleration Z | 100 mm/s² | |
| Max acceleration E | 5000 mm/s² | |
| Max acceleration when extruding | 500 mm/s² | `DEFAULT_ACCELERATION` |
| Max acceleration when retracting | 500 mm/s² | `DEFAULT_RETRACT_ACCELERATION` |
| Max acceleration when travelling | 500 mm/s² | `DEFAULT_TRAVEL_ACCELERATION` |
| Maximum junction deviation | 0.08 mm | `JUNCTION_DEVIATION_MM`. `CLASSIC_JERK` is off, so the jerk fields don't matter. |
| Min print / travel speed | 0 mm/s | `DEFAULT_MINIMUMFEEDRATE` |

## Printer settings › Extruder

| Setting | Value | Note |
| --- | --- | --- |
| Nozzle diameter | 0.4 mm | Stock Ender-3 (change it if you fit a different nozzle) |
| Retraction length | 5 mm | Bowden (assumed from the 350 mm fast-load length). Use 0.8–1 mm if direct drive. Fine-tune with Orca's retraction test. |
| Retraction speed | 25 mm/s | Firmware E max feedrate |
| Deretraction speed | 25 mm/s | |
| Z hop | 0.2–0.4 mm | Z hop type: Normal or Slope |
| Retract on layer change | On | |
| Wipe while retracting | On | |

## Filament settings

| Setting | Value | Note |
| --- | --- | --- |
| Diameter | 1.75 mm | `DEFAULT_NOMINAL_FILAMENT_DIA` |
| Nozzle temp (PLA) | 200–210 °C | Firmware PLA preheat is 185 °C; min extrude temp is 170 °C |
| Bed temp (PLA) | 45–60 °C | Firmware PLA preheat is 45 °C |
| Nozzle temp (ABS/PETG) | 230–240 °C | Firmware ABS preheat is 240 / 110 |
| Max nozzle temp | < 260 °C | `HEATER_0_MAXTEMP 275` (thermal cutout is ~15 °C below that) |
| Max bed temp | < 115 °C | `BED_MAXTEMP 125` |
| Pressure advance | Off, or calibrated K | The firmware already uses `ADVANCE_K 0.36`. Turn it on only after you run Orca's PA calibration; Orca then sends `M900 K…`. |

## Process settings

### Speed

With 500 mm/s² acceleration the printer only gets near these speeds on long straight moves. Higher speeds make little difference.

| Setting | Value |
| --- | --- |
| First layer | 25 mm/s |
| Outer wall | 40 mm/s |
| Inner wall | 60 mm/s |
| Sparse infill | 80 mm/s |
| Top surface | 40 mm/s |
| Travel | 150 mm/s |

### Acceleration

| Setting | Value | Note |
| --- | --- | --- |
| Normal printing | 500 mm/s² | Firmware max |
| Outer wall / top surface | 300–500 mm/s² | |
| Travel | 500 mm/s² | |
| Junction deviation / jerk | Leave at default (0) | The firmware value is used |

### Advanced

| Setting | Value | Note |
| --- | --- | --- |
| Arc fitting | Off | `ARC_SUPPORT` disabled |
| Bed mesh / adaptive mesh | Off in Orca | The start G-code runs a full 5×5 `G29` |

## Machine G-code

### Machine start G-code

```gcode
G90                     ; absolute XYZ
M83                     ; relative E
M140 S[bed_temperature_initial_layer_single]
M104 S150               ; warm nozzle without oozing while probing
G28                     ; home (Z safe homing in the bed centre)
M190 S[bed_temperature_initial_layer_single]
G29                     ; probe 5x5 bilinear mesh
; M420 S1 Z10           ; ...or replace G29 with this to load the mesh saved in EEPROM
M109 S[nozzle_temperature_initial_layer]
G92 E0
G1 Z2.0 F3000
G1 X0.1 Y20 Z0.3 F5000
G1 X0.1 Y200 Z0.3 F1500 E15   ; purge line 1
G1 X0.4 Y200 Z0.3 F5000
G1 X0.4 Y20 Z0.3 F1500 E15    ; purge line 2
G92 E0
G1 Z2.0 F3000

G90 ;use absolute coordinates
M83 ;extruder relative mode
M140 S[bed_temperature_initial_layer_single] ;set initial bed temp, non blocking
M104 S150 ;set temporary nozzle temp, non blocking
M190 S[bed_temperature_initial_layer_single] ;set initial bed temp, blocking
G28 ;home all axis
M420 S1                ;load saved mesh (use G29 to re-probe each print)
M109 S[nozzle_temperature_initial_layer]  set temporary nozzle temp, blocking
G92 E0 ;set extruder position to zero
G1 Z2 F600; move print head up
G1 X20 Y5 Z0.3 F5000; move print head to start position
G1 X120 Y5 E15 F1500   ; purge line along the front edge
G1 X120 Y5.5; move print head to start position
G1 X20 Y5.5 E10; purge line along the front edge
G92 E0; set extruder position to zero

```

### Machine end G-code

```gcode
G91                     ; relative
G1 E-2 F1500            ; retract (25 mm/s max)
G1 Z10 F300             ; lift
G90                     ; absolute
G1 X0 Y220 F3000        ; present the part
M106 S0                 ; part fan off
M104 S0                 ; hotend off
M140 S0                 ; bed off
M84 X Y E               ; motors off (Z stays on: DISABLE_IDLE_Z is off)

{if max_layer_z < printable_height}G1 Z{z_offset+min(max_layer_z+2, printable_height)} F600 ; Move print head up{endif}
G1 X5 Y{print_bed_max[1]*0.8} F{travel_speed*60} ; present print
{if max_layer_z < printable_height-10}G1 Z{z_offset+min(max_layer_z+70, printable_height-10)} F600 ; Move print head further up{endif}
{if max_layer_z < max_print_height*0.6}G1 Z{printable_height*0.6} F600 ; Move print head further up{endif}
M140 S0 ; turn off heatbed
M104 S0 ; turn off temperature
M107 ; turn off fan
M84 X Y E ; disable motors, keep Z powered so the gantry stays level
```

### Other G-code

| Field | Value | Note |
| --- | --- | --- |
| Change filament G-code | `M600` | `ADVANCED_PAUSE_FEATURE` + `PARK_HEAD_ON_PAUSE`: parks the head, unloads 400 mm, loads 350 mm |
| Pause G-code | `M600` or `M0` | `M601` is not supported in Marlin. `M0` waits for a click but doesn't park. |
| Layer change G-code | (empty) | Optional: `M117 Layer {layer_num+1}` |

## Connection

- USB on the SKR Mini E3 v3 is native USB (`SERIAL_PORT_2 -1`), so the baud rate doesn't matter. Use 115200 if a host asks for one.
- The TFT/UART port (`SERIAL_PORT 2`) runs at 115200.
- Host prompts and action commands are enabled (good with OctoPrint).

## Calibration to run in Orca

- **Temperature tower**, then **flow rate**, then **pressure advance**. Compare the result with the firmware's K = 0.36.
- **Input shaping**: the firmware defaults are X/Y 40 Hz, ζ 0.15. Orca's input-shaping test sends `M593`. Save the result with `M500`.
- **Retraction test**: keep the speed at or below 25 mm/s.
- E-steps are 147 (custom extruder). Check them with a 100 mm extrude before you calibrate flow.
