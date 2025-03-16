# Klipper automatic offset calibration for tool changers  
This is a set of macros and files that will allow you to calibrate
the X, Y, and Z offsets between all of your tool heads using a 3 axis probe.  

These macros require a 3 axis probe, such as https://github.com/zruncho3d/nudge or https://github.com/viesturz/NozzleAlign

They have been built around using the klipper-toolchanger add on available at https://github.com/viesturz/klipper-toolchanger

## Usage:

Upload the offset_save_file.cfg and tc_offset_calibration_macros.cfg to your printer.

## Printer.cfg Modifications:
In your printer.cfg add the line:
```
[include tc_offset_calibration_macros.cfg]
```

## Offset_save_file.cfg Modifications:
Add as many offset variable blocks as you have tool heads, and number them accordingly. You do not need to change the nudge_position
values, they will be automatically updated by running a macro later.
```
current_tool = 0
nudge_position_x = -1000
nudge_position_y = -1000
nudge_position_z = -1000
t0_offset_x = 0.0
t0_offset_y = 0.0
t0_offset_z = 0.0
t1_offset_x = 0.0
t1_offset_y = 0.0
t1_offset_z = 0.0
t2_offset_x = 0.0
t2_offset_y = 0.0
t2_offset_z = 0.0
```

## Tool Head cfg file Modifications:
In each of your tool head configuration files, copy the macro in tool_file_modifications.cfg and replace your
[gcode_macro Tx] with it. Rename the macro to the number of your tool, T0, T1, T2, etc..

Set the variable_tool_number to match the name of the gcode_macro.

It should look similar to this:
```
[gcode_macro T2]
variable_color: ""
variable_tool_number: 2
gcode:
```

Nothing else in the macro needs to be changed for the offset functionality. Though fan and LED macros
can be added to the macro, that is beyond the scope of this repo.

Additionally in the tool head configuration files, make sure that all of your gcode offsets under 
the [tool Tx] are set to 0.
```
[tool T2]
gcode_x_offset: 0
gcode_y_offset: 0
gcode_z_offset: 0
```

## Probe setup and testing:
In the tc_offset_calibration_macros.cfg set the proper pin for your probe. Test your probe with:
```
TOOL_CALIBRATE_QUERY_PROBE
```

## Running the macros:
Once the probe has been verified, home your printer, QGL or Z_TILT_ADJUST. Then position T0 over the probe.

Run the command: 
```
TC_GET_PROBE_POSITION
```

This will find and save the position of the probe to the save file. This will only need to be run once, unless you move the
probe or edit your endstops.

Then to find the offsets of all of your tools run:
```
TC_FIND_ALL_TOOL_OFFSETS
```
To find the offsets for specific tools run:
```
TC_FIND_TOOL_OFFSETS TOOL=1,2,13,6
```
This can be run with any number of tools as long as the numbers are separated by a comma. So if you rebuild T3 then you can run:
```
TC_FIND_TOOL_OFFSETS TOOL=3
```

## What should be happening when the macros are run?
The process starts by heating up T0 to a probing temperature, this is set as a variable at the top of the TC_FIND_ALL_TOOL_OFFSETS
and TC_FIND_TOOL_OFFSETS macros. It is currently set to 150C, if you would like it different, then it can be changed there. Once
T0 is heated up, it will locate the probe and get a baseline to test the other tools to. For each tool to be tested, it will be heated
up to the probing temperature and the probing will begin. At the end of the probing, if the printer is not printing, the tools will be
cooled down. If the offset calibrations are run during a print_start, then the tools will be left at 150C and it is expected that the
print_start will heat them to printing temperatures at the correct time.

## Running TC_ADJUST_OFFSET
If during a print, you notice that an offset is not quite correct, you can adjust the offset while printing. This will adjust and save
the offset. For instance, if you notice that the Y offset for tool 3 is slightly too far to the rear of where it should be, run the
following in the console:
```
TC_ADJUST_OFFSET TOOL=3 AXIS=Y AMOUNT=-0.10
```
This will shift the Y offset towards the left by 0.10mm. If the tool is currently printing, the offset will be adjusted immediately.

## G-code Offset Adjustment tips:

<li>A good rule I use to remember how to adjust the offsets, a positive number moves the offset towards the axis maximum, a negative number
moves the offset towards the axis minimum.</li>
<li>I recommend adjusting in 0.1 mm increments for X/Y to start, you can do this repeatedly in the Klipper console by pressing the up arrow. I
generally do not adjust X or Y under 0.050 increments.</li>
<li>I use increments of 0.05mm or 0.025mm to start when adjusting the gcode offset for Z. I generally don't adjust Z under 0.010 increments. I
also do not usually adjust Z after the first layer.</li>
<li>If you get too far off and cannot get the adjustments back into line; clean the nozzle then re-run the offset calibration for that tool.</li>
