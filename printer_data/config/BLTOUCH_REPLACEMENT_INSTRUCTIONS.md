# BLTouch Replacement Instructions

## Current Status
**Your BLTouch has a failed sensor circuit and must be replaced.**

**Diagnosis:**
- Control circuit works (pin deploys/retracts)
- Sensor circuit FAILED (does not detect pin position)
- Tested all configurations and both P0.10 and P1.27 pins
- Confirmed hardware failure - cannot be fixed via software

---

## When Installing New BLTouch

### 1. Physical Installation
- Connect 3-pin servo connector (Brown/Red/Orange)
- Connect 2-pin sensor connector (Black/White)

**Wiring to SKR V1.4 Turbo:**
- White wire → **P0.10** (Z-PROBE signal pin)
- Orange/Yellow wire → **P2.0** (servo control pin)
- Red wire → +5V
- Black/Brown wires → GND

### 2. Test Basic Functionality
Run these commands via Mainsail console or SSH:

```gcode
BLTOUCH_DEBUG COMMAND=reset
BLTOUCH_DEBUG COMMAND=pin_down
BLTOUCH_DEBUG COMMAND=pin_up
BLTOUCH_DEBUG COMMAND=self_test
QUERY_PROBE
```

**Expected results:**
- Pin should deploy and retract smoothly
- `QUERY_PROBE` should show "open" when retracted
- `QUERY_PROBE` should show "TRIGGERED" when you push the pin up

### 3. Calibrate Z Offset
**CRITICAL:** You must calibrate z_offset after installing new probe!

```gcode
G28                          # Home all axes
PROBE_CALIBRATE             # Start calibration
# Follow paper test procedure
TESTZ Z=-0.1                # Move down in small increments
TESTZ Z=+0.01               # Fine adjustments
ACCEPT                      # Accept the offset
SAVE_CONFIG                 # Save to printer.cfg
```

### 4. Test Homing
```gcode
G28                          # Home all axes
G28 Z                        # Home Z only
```

Should complete without errors.

### 5. Create Bed Mesh
```gcode
G28                          # Home first
BED_MESH_CALIBRATE          # Probe 25 points (5x5 grid)
SAVE_CONFIG                 # Save mesh
```

### 6. Verify Probe Accuracy
```gcode
G28
PROBE_ACCURACY              # Test repeatability (should be <0.025mm)
```

---

## Configuration Notes

Current configuration in `printer.cfg`:
- **sensor_pin: ^P0.10** - White wire with pullup enabled
- **control_pin: P2.0** - Orange/Yellow servo wire
- **x_offset: -44** - Probe is 44mm left of nozzle
- **y_offset: -14** - Probe is 14mm forward of nozzle
- **z_offset: 0** - MUST BE CALIBRATED (currently uncalibrated)

### Optional Settings (currently commented out)
Uncomment in printer.cfg if needed:

- `pin_move_time: 0.680` - If probe is slow to deploy
- `stow_on_each_sample: False` - Speeds up multi-point probing
- `probe_with_touch_mode: True` - More accurate but slower

---

## Recommended BLTouch Models

**Genuine (Best):**
- ANTCLABS BLTouch v3.1 (~$40)
- ANTCLABS BLTouch Smart (~$45)

**Reliable Clones:**
- Creality BLTouch (~$25)
- 3DTouch by TriangleLabs (~$20)

**Avoid:** Very cheap clones (<$15) - often have sensor failures like your current one.

---

## Troubleshooting New BLTouch

If new probe also has issues:

1. Check wiring (especially white sensor wire to P0.10)
2. Try alternative sensor_pin configurations:
   - `sensor_pin: ^P0.10` (pullup - default)
   - `sensor_pin: P0.10` (no pullup)
   - `sensor_pin: ^!P0.10` (pullup + inverted)

3. Try output mode settings:
   - Add `set_output_mode: 5V` to [bltouch] section
   - OR `set_output_mode: OD` (open drain)

4. Check for loose Dupont connector crimps
5. Verify probe LED behavior (should flash/change color)

---

## Contact
Configuration prepared by Claude Code after extensive diagnostic testing.
Date: 2025-11-17
