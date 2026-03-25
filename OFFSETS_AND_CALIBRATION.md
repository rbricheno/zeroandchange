# 🛠 Offset & Calibration Guide: zeroandchange

This document explains the decoupled offset philosophy used in the `zeroandchange` configuration. This system separates physical machine calibration from tool-specific deltas and user-applied "baby-steps."

## 1. The Three-Layer Offset Model

Unlike standard Klipper tool-changing, which often bakes all offsets into a single value, this config uses three layers to calculate the final G-code offset:

**Final Z Offset = [System Baseline] + [Tool Delta] + [User Tweak]**

### A. System Baseline (`t0_probe_z_offset`)
*   **Stored in:** `offset_save_file.cfg`
*   **Definition:** The vertical distance between the T0 nozzle tip and the point where the Z-probe triggers.
*   **Purpose:** This is the "Global Z" for the entire printer. If you change your bed surface or home position, this remains constant. You set this once using a paper test or feeler gauge.

### B. Tool Delta (`tX_offset_x/y/z`)
*   **Stored in:** `offset_save_file.cfg`
*   **Definition:** The difference in length and position between Tool X and Tool 0.
*   **Reference:** **T0 is always 0,0,0**. Its deltas should never be changed from zero.
*   **Calibration:** Found automatically using the `TC_FIND_ALL_TOOL_OFFSETS` macro.

### C. User Tweak (`last_user_tweak`)
*   **Stored in:** Memory (Variable in `homing_variables`)
*   **Definition:** Live baby-stepping applied by the user during a print. These are the values that are modified if you click on the buttons in Mainsail to change the offset during a print, for example.
*   **Persistence:** The macro `_SAVE_CURRENT_TWEAK` captures any live baby-stepping before a tool change and reapplies it to the next tool. This ensures that if you "squish" the first layer for T0, T1 will respect that same squish.

---

## 2. Calibration Neutrality

A core feature of this config is **Calibration Neutrality**. 

When running `TC_FIND_TOOL_OFFSETS`, the system must know the difference between the tools without the math being "polluted" by active G-code offsets or previous baby-steps.

**The Probing Sequence:**
1.  **Pickup Tool:** The tool is loaded.
2.  **Neutralize:** `SET_GCODE_OFFSET` is called to clear X and Y, and set Z specifically to the `System Baseline` (ignoring previous tool offsets or tweaks).
3.  **Probe:** The Nudge probe is hit.
4.  **Calculate:** Because we started from a "neutral" baseline, the result of the probe is the pure Tool Delta.
5.  **Save & Restore:** The delta is saved, and the tool's actual offset (including tweaks) is restored.

---

## 3. Essential Macros

| Macro | Description |
| :--- | :--- |
| `TC_GET_PROBE_POSITION` | Locates the Nudge pin in absolute machine space using T0. |
| `TC_FIND_ALL_TOOL_OFFSETS` | Heats all tools and probes them relative to T0. |
| `APPLY_TOOL_OFFSETS` | The core logic that assembles the three layers (Baseline + Delta + Tweak). |
| `_SAVE_CURRENT_TWEAK` | Calculates how much the user has baby-stepped and buffers it. |

---

## 4. Maintenance Workflow

1.  **If T0 nozzle is replaced:** You must update `t0_probe_z_offset` in `offset_save_file.cfg` and re-run tool calibration for all other tools.
2.  **If a secondary tool (T1, T2...) is replaced:** Simply run `TC_FIND_TOOL_OFFSETS TOOL=X`.
3.  **If the Nudge pin is moved:** Re-run `TC_GET_PROBE_POSITION`.

