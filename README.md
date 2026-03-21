# zeroandchange
Ambrosia personal current yellow/blue deep zero configs

Will probably serve as the basis for "launch" deep zero config, but then might get more experimental.

Full stack config including homing and r2pdx macros plus extensive use of gcode offsets instead of relying on klipper's internal behaviour.

Follow zruncho instructions to configure geometry.
z_offset for all tool_probes should be set to 0
Home with T0
X Y and Z offsets should be set to 0 for T0. Other tool offsets are relative to T0 at 0,0,0
Follow r2pdx procedure for offset callibration
The variable t0_probe_z_offset in offset_save_file is your new z offset. I do maxwell probing, mine is about 0.2. Set it and forget it using something like the paper test.
My nudge is installed next to my bed, front left. Your geometry will vary. Make sure your nozzles are extremely clean before homing (T0) or doing any offset callibration (T0 and T1!).
