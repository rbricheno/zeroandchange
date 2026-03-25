# 🚀 Print Start Sequence: zeroandchange

This document outlines the logic used in the `PRINT_START` macro to handle multi-tool preheating, smart homing, and conditional priming.

## 1. Sequence Overview

The `PRINT_START` macro (found in `printer.cfg`) follows these steps:

1.  **Safety Home:** Calls `SMART_HOMING` to ensure the gantry is positioned safely.
2.  **Bed Pre-heat:** Sets the bed to target temperature (`M190`) and waits.
3.  **Tool Pre-heat (Ooze Prevention):** 
    *   Heats T0 (the homing/probing tool) and T1 (if used) to 150°C.
    *   This temperature is high enough for quick final heating but low enough to prevent most filaments from oozing during probing.
4.  **Final Heat-up:** 
    *   Heats only the tools flagged as `TRUE` in the `IST0` and `IST1` parameters (sent by the slicer).
    *   Unused tools are turned off (`M104 S0`).
5.  **Sequential Priming:** 
    *   Determines which tool is the `INITIAL_TOOL`.
    *   Primes the *other* used tool first, then switches to the `INITIAL_TOOL` and primes it last, leaving it ready to print.

## 2. Temperature Handling

### The "Ooze-Free" Window
We use 150°C as a standby/probing temperature. 
*   **Probing:** `TC_FIND_TOOL_OFFSETS` heats to this temperature.
*   **Waiting:** During `PRINT_START`, we hold tools at 150°C while the bed finishes heating.

### Smart Waiting (`_WAIT_FOR_TEMP_WITHIN_TOLERANCE`)
The printer uses a non-blocking initial heat-up (`M104`) followed by a "Ready Check" during tool pickups. 
*   **Logic:** The system only blocks if the tool is **too cold**. 
*   **Overshoot:** If a tool overshoots the target by a few degrees, the system **does not wait** for it to cool down, preventing unnecessary delays.

## 3. Best Practices & Safety

### Bed Meshing
*   **Always use T0:** Bed meshes must be generated while Tool 0 is active.
*   **Clean Slate:** Ensure no `User Tweak` is active when running `BED_MESH_CALIBRATE`. The safest way to do this is to run `G28` right before meshing, as homing resets tweaks to 0.0.

### Pin Sharing
*   This machine shares `EBBCan0: PB6` for X-homing, Z-homing (virtual endstop), and Tool Detection. 
*   Because of this, `[duplicate_pin_override]` is active. Always ensure the toolhead is clear of the dock before homing to prevent signal noise from the dock magnets/switches.

### Fan Management
*   The `print_end` macro uses a custom `M107` command. This ensures that even if a tool is parked and inactive, its part cooling fan is explicitly turned off at the end of a job.

## 4. Slicer Integration
To use this macro, your slicer's "Start G-code" should look like this:
```gcode
PRINT_START TOOL_0_START=[first_layer_temperature_0] TOOL_1_START=[first_layer_temperature_1] BED=[first_layer_bed_temperature] INITIAL_TOOL=[initial_tool] IST0=[is_extruder_used_0] IST1=[is_extruder_used_1]
```
