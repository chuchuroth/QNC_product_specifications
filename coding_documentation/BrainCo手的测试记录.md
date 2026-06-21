# BrainCo Bionic Dexterous Hand — Revo 1 Driver

**Hardware:** BrainCo Bionic Dexterous Hand Revo 1 (SKU: MediumRight)  
**Protocol:** Modbus RTU · RS485 · 115200 baud · slave_id=1  
**Adapter:** FTDI FT232R USB-RS485 on `/dev/ttyUSB0`  
**SDK:** `bc_stark_sdk` v1.1.5 (aarch64)

---

## 1. Hardware Setup

### Wiring

| USB-RS485 Adapter | Hand Wrist Connector |
|---|---|
| A (D+) | RS485-A |
| B (D−) | RS485-B |
| GND | GND |

The hand MCU also requires **motor power** (separate from the RS485 bus):

- Connect the LiPo battery via the palm connector, **or** connect the DC charger/supply to the wrist-cuff USB-C port.
- Without motor power the MCU responds to all Modbus queries (ACKs are sent), but no joints move and every register reads back as `0`.
- Indicator: `reg 2024` (battery voltage) reads non-zero when motor power is present (typical range 7200–8400 for a 7.4 V LiPo at normal charge).

> **Symptom of missing motor power:** every FC10 write is ACKed correctly, but `reg 1010` reads back `0` immediately after the write, and no physical movement occurs.

### Power-On Sequence

1. Connect RS485 wiring.
2. Plug in / switch on motor power. Wait ~2 s for the auto-calibration chime.
3. The hand performs a brief auto-calibration stroke on power-up — do **not** send commands during this window.
4. Once calibration is complete, the hand is ready. No explicit `init` command is required.

---

## 2. Register Map (Revo 1)

All registers are 16-bit unsigned unless noted.

### Control — FC10 Write Multiple Registers

| Register | Name | Range | Notes |
|---|---|---|---|
| 1010 | ThumbFlex position | 0–100 | 0 = open, 100 = closed |
| 1011 | ThumbAux position | 0–50 | thumb abduction/adduction |
| 1012 | Index position | 0–100 | |
| 1013 | Middle position | 0–100 | |
| 1014 | Ring position | 0–100 | |
| 1015 | Pinky position | 0–100 | |
| 1016–1021 | Per-finger speed | 0–100 % | **See critical note below** |

### Configuration — FC06 Write Single Register

| Register | Name | Values |
|---|---|---|
| 1002 | Force level | 1 = Small, 2 = Normal, 3 = Full |

### Status — FC03 Read Holding Registers

| Register | Name | Notes |
|---|---|---|
| 2000–2005 | Actual positions | 0–100 %, mirrors set values when at rest |
| 2006–2011 | Actual speeds | Signed, per-finger |
| 2018–2023 | Motor state | 0=idle, 1=running, 2=stall, 3=turbo |
| 2024 | Battery voltage | mV × 10; 0 = motor power absent |

> **FC04 is not supported.** The device accepts FC04 queries but always responds with an FC03 frame. Use FC03 for all reads.

---

## 3. Critical Protocol Notes

### ① Speed register write cancels motion

Writing to speed registers 1016–1021 **immediately after** a position write (reg 1010) causes the firmware to treat the speed write as a new command, cancelling the in-progress motion before the motors start.

**Do not write speeds in the same command sequence as positions.** Set speed once at startup if needed, then only write position registers to trigger motion.

This was confirmed by strace of the official `bc_stark_sdk`, which **only ever writes reg 1010** for `set_finger_positions` — it never touches reg 1016 during normal operation.

```
# SDK on-wire for set_finger_positions([500, 500, 1000, 1000, 1000, 1000]):
TX: 01 10 03 F2 00 06 0C 00 32 00 32 00 64 00 64 00 64 00 64 92 E1
    ── FC10  addr=1010  count=6  data=[50, 50, 100, 100, 100, 100] ──
```

### ② Input scaling: SDK uses 0–1000 externally, 0–100 on wire

The SDK's `set_finger_positions()` accepts 0–1000 and divides by 10 before writing. The register values are always 0–100.

```
SDK input [500, 500, 1000, 1000, 1000, 1000]
→ wire values [50, 50, 100, 100, 100, 100]
→ ThumbFlex/Aux at 50%, all fingers at 100% (fist)
```

### ③ Thumb range is 0–50, not 0–100

ThumbFlex (reg 1010) and ThumbAux (reg 1011) saturate at 50 in normal use. The SDK maps a full 1000 input to 50 for thumbs, matching `input / 20`.

### ④ Position registers read back 0 at rest

At rest all position registers (2000–2005) return 0 regardless of the commanded position. This is normal firmware behaviour — there is no live encoder feedback exposed over Modbus RTU on this variant.

---

## 4. Software Artifacts

### `robot_side/src/brainco_revo1.hpp`

Header-only C++17 driver. Contains the complete `CGC80Gripper` class and all Modbus RTU infrastructure (CRC-16, frame builder, `transact`, `drain_rx`, `validate_response`). Include this file to get a full driver with no external dependencies.

Key public API:

```cpp
CGC80Gripper hand("/dev/ttyUSB0", /*slave_id=*/1);
hand.initialize();                 // set force = Normal (reg 1002)
hand.open();                       // position 0 on all fingers
hand.close();                      // position 1000 mapped to 0–100
hand.set_position(600);            // arbitrary 0–1000
hand.read_status();                // print positions to stdout
auto pos = hand.get_positions();   // returns std::vector<uint16_t> (6 regs)
```

### `robot_side/src/gripper_control.cpp`

Standalone binary, builds as `./build/gripper_control`.

```
Usage:
  ./gripper_control init             [device_id]
  ./gripper_control open             [device_id]
  ./gripper_control close            [device_id]
  ./gripper_control position <0–1000> [device_id]
  ./gripper_control status           [device_id]
  ./gripper_control demo [cycles]    [device_id]
```

Port override: `QNC_SERIAL_PORT=/dev/ttyUSB0 ./build/gripper_control <cmd>`

### `robot_side/tests/revo_cycle_test.cpp`

Endurance test. Builds as `./build/revo_cycle_test`.

```
Usage: QNC_SERIAL_PORT=/dev/ttyUSB0 ./build/revo_cycle_test [cycles] [device_id] [dwell_ms]

Defaults: 30 cycles · slave_id=1 · 2500 ms dwell

Exit codes:
  0 — all cycles passed
  1 — one or more cycles failed
  2 — fatal error (port open / init failed)
```

---

## 5. Build

```bash
# From workspace root
cmake -S . -B build -DBUILD_EXAMPLES=ON
make -C build gripper_control revo_cycle_test
```

No external dependencies. Pure C++17 + POSIX.

---

## 6. Quick Reference — Common Commands

```bash
# Single open/close
QNC_SERIAL_PORT=/dev/ttyUSB0 ./build/gripper_control open
QNC_SERIAL_PORT=/dev/ttyUSB0 ./build/gripper_control close

# Read finger positions
QNC_SERIAL_PORT=/dev/ttyUSB0 ./build/gripper_control status

# 30-cycle endurance test (default)
QNC_SERIAL_PORT=/dev/ttyUSB0 ./build/revo_cycle_test

# 10 cycles, 3 s dwell between positions
QNC_SERIAL_PORT=/dev/ttyUSB0 ./build/revo_cycle_test 10 1 3000

# Custom device ID
QNC_SERIAL_PORT=/dev/ttyUSB0 ./build/gripper_control close 2
```

---

## 7. Debugging Checklist

| Symptom | Likely Cause | Fix |
|---|---|---|
| ACK received but no movement | Motor power absent | Plug in battery / power supply |
| All registers read as `0` | Motor power absent | Same as above |
| `✗ Failed to read status` | Wrong function code | Ensure code uses FC03 not FC04 |
| Commands ACKed but finger stops mid-move | Speed register written after position | Remove write to reg 1016 from motion sequence |
| No ACK at all | Wrong baud / wiring | Verify 115200, check A/B polarity |
| `[diag] validate_response` on console | Response FC mismatch | Expected – device echoes FC10 ACK in some states; drain_rx(50) clears it |

---

## 8. SDK Cross-Reference

```python
# Python SDK equivalent (bc_stark_sdk v1.1.5)
from bc_stark_sdk import main_mod as lib

async def example():
    c = await lib.modbus_open("/dev/ttyUSB0", lib.Baudrate.Baud115200)
    await c.set_hardware_type(1, lib.StarkHardwareType.Revo1Basic)
    await c.set_force_level(1, lib.ForceLevel.Full)          # FC06 reg 1002 = 3
    await c.set_finger_positions(1, [500,500,1000,1000,1000,1000])  # fist
    await c.set_finger_positions(1, [0, 0, 0, 0, 0, 0])            # open
    lib.modbus_close(c)
```

On-wire bytes confirmed via `strace -f -e trace=write`:

```
# set_force_level(Full):
TX: 01 06 03 EA 00 03 E8 7B   (FC06, reg 1002, value 3)

# set_finger_positions([500,500,1000,1000,1000,1000]):
TX: 01 10 03 F2 00 06 0C 00 32 00 32 00 64 00 64 00 64 00 64 92 E1

# set_finger_positions([0,0,0,0,0,0]):
TX: 01 10 03 F2 00 06 0C 00 00 00 00 00 00 00 00 00 00 00 00 3F 02
```
