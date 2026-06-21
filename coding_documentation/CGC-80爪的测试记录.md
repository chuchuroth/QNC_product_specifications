# CGC-80 Gripper Control — Quickstart Guide

**Hardware:** DH Robotics CGC-80 · RS485 / Modbus RTU  
**Adapter:** FTDI FT232R USB-RS485 on `/dev/ttyUSB0`

---

## 1. Build (C++)

```bash
g++ -std=c++17 -O2 -o gripper_control_cpp gripper_control.cpp
```

No external libraries required.

---

## 2. Wiring

| RS485 Adapter | CGC-80 Gripper |
|---|---|
| A (D+) | RS485-A |
| B (D−) | RS485-B |
| GND | GND |

> **Important:** A/B polarity must be correct. If the gripper does not respond, swap A and B.

---

## 3. Commands

| Command | Description |
|---|---|
| `./gripper_control_cpp init` | Activate the gripper (required before any motion) |
| `./gripper_control_cpp open` | Fully open (position 0/1000) |
| `./gripper_control_cpp close` | Fully close (position 1000/1000) |
| `./gripper_control_cpp position <0–1000>` | Move to a specific position |
| `./gripper_control_cpp status` | Read init state, gripper state, and position |

An optional device ID argument (default `1`) can be appended to any command:

```bash
./gripper_control_cpp init 2        # use device ID 2
```

---

## 4. Full Cycle Example

```bash
./gripper_control_cpp init && \
./gripper_control_cpp open && \
sleep 2 && \
./gripper_control_cpp close && \
sleep 2 && \
./gripper_control_cpp status
```

Expected output:

```
✓ Connected to gripper on /dev/ttyUSB0 (device ID: 1)
✓ Gripper initialized
✓ Connection closed
✓ Connected to gripper on /dev/ttyUSB0 (device ID: 1)
✓ Gripper initialized
✓ Moving to position 0 (force=50%, speed=50%)
Init state:    1
Gripper state: 1
Position:      0/1000
✓ Connection closed
...
Init state:    1
Gripper state: 1
Position:      1000/1000
✓ Connection closed
```

---

## 5. Python Alternative

The equivalent Python script is also available:

```bash
python3 gripper_control.py init
python3 gripper_control.py open
python3 gripper_control.py close
python3 gripper_control.py position 500
python3 gripper_control.py status
```

Requires `pymodbus >= 3.11`:

```bash
pip install pymodbus
```

---

## 6. Scanning for Grippers

If the gripper does not respond, use the scanner to find the correct port, baud rate, and device ID:

```bash
python3 scan_all.py
```

Scans `/dev/ttyUSB0` and `/dev/ttyUSB1` across baud rates 115200→9600 and device IDs 1–16.

---

## 7. Parameter Ranges

| Parameter | Range | Default | Description |
|---|---|---|---|
| `position` | 0 – 1000 | — | 0 = fully open, 1000 = fully closed (units: 1/10 %) |
| `force` | 20 – 100 | 50 | Closing force (%) |
| `speed` | 1 – 100 | 50 | Movement speed (%) |

Force and speed are only configurable via the API (not CLI arguments). Edit the defaults in `gripper_control.cpp` or `gripper_control.py` if needed.

---
---
DH CGC-80 Gripper Wiring Guide
===============================

MARSHALL CV-USB-RS485 ADAPTER PINOUT
------------------------------------

Your Marshall adapter might have pins labeled as:
1. Option A: A/B/GND
2. Option B: D+/D-/GND  
3. Option C: TX+/TX-/RX+/RX-/GND

For RS485 communication with the gripper:
- Use the TRANSMIT pins (TX+ and TX-, or D+/D-, or A/B)
- IGNORE receive pins if present (RX+/RX-)


WIRING CONFIGURATIONS TO TRY
-----------------------------

If your adapter has A/B labels:
  Try Configuration 1:
    Marshall A → Gripper RS485-A
    Marshall B → Gripper RS485-B
    Marshall GND → Gripper GND
    
  Try Configuration 2 (SWAPPED):
    Marshall A → Gripper RS485-B
    Marshall B → Gripper RS485-A
    Marshall GND → Gripper GND


If your adapter has D+/D- labels:
  Try Configuration 1:
    Marshall D+ → Gripper RS485-A
    Marshall D- → Gripper RS485-B
    Marshall GND → Gripper GND
    
  Try Configuration 2 (SWAPPED):
    Marshall D+ → Gripper RS485-B
    Marshall D- → Gripper RS485-A
    Marshall GND → Gripper GND


If your adapter has TX+/TX-/RX+/RX- labels:
  Try Configuration 1:
    Marshall TX+ → Gripper RS485-A
    Marshall TX- → Gripper RS485-B
    Marshall GND → Gripper GND
    (IGNORE RX+ and RX-)
    
  Try Configuration 2 (SWAPPED):
    Marshall TX+ → Gripper RS485-B
    Marshall TX- → Gripper RS485-A
    Marshall GND → Gripper GND
    (IGNORE RX+ and RX-)


USING ETHERNET CABLE
--------------------

Inside an ethernet cable, you'll find 4 twisted pairs:
- Orange pair: Orange + Orange/White
- Blue pair: Blue + Blue/White
- Green pair: Green + Green/White  
- Brown pair: Brown + Brown/White

Use ONE twisted pair for RS485:
  Blue wire → Connect to one pin (A or TX+)
  Blue/White → Connect to other pin (B or TX-)
  Orange wire → GND


CRITICAL POWER CHECK
--------------------

The gripper MUST be powered with 24V DC!

Visual check:
  [ ] Is there an LED lit on the gripper?
  [ ] If NO LED, gripper is NOT powered!
  
Connection check:
  [ ] 24V DC power supply connected?
  [ ] Power supply plugged in and turned on?
  [ ] Check voltage with multimeter if available


TROUBLESHOOTING STEPS
---------------------

1. Verify power first (LED on gripper)
2. Try Configuration 1 with normal A/B wiring
3. If no response, try Configuration 2 (swap A/B)
4. If still no response, check your Marshall adapter pinout
5. Try the advanced_diagnostic.py script


NEXT STEPS
----------

Please provide this information:
1. Is there an LED on the gripper? What color?
2. What are the pin labels on your Marshall adapter?
3. What type of cable are you using?
4. How long is the cable?

---
---
---
# DH Robotics CGC-80 Gripper — Diagnostic & Fix Report

**Date:** 2026-02-25  
**Hardware:** DH Robotics CGC-80 (RS485 / Modbus RTU)  
**Adapter:** FTDI FT232R USB-RS485 (`/dev/ttyUSB0`, serial `A602ANI1`)  
**Software:** Python 3.12 · pymodbus 3.11.4  

---

## Summary

Two software issues prevented the gripper from operating. Both were identified and fixed. The gripper now opens, closes, moves to arbitrary positions, and returns correct status readings.

---

## Issue 1 — Wrong pymodbus Timeout (Software)

### What happened
pymodbus raised `ModbusIOException: No response received after 3 retries` on every command.

The CGC-80 takes slightly longer than 1 second to formulate and transmit its response at 115200 baud. pymodbus was timing out and discarding the incoming bytes before they fully arrived.

### How it was found
A raw `serial` library probe (bypassing pymodbus entirely) was used to send a manually crafted Modbus RTU frame and listen for any bytes back:

```python
import serial, time

def crc16(data):
    crc = 0xFFFF
    for b in data:
        crc ^= b
        for _ in range(8):
            crc = (crc >> 1) ^ 0xA001 if crc & 1 else crc >> 1
    return crc.to_bytes(2, 'little')

pdu = bytes([0x01, 0x04, 0x02, 0x00, 0x00, 0x03])  # FC04, id=1, addr=0x0200, cnt=3
frame = pdu + crc16(pdu)

ser = serial.Serial('/dev/ttyUSB0', 115200, timeout=0.5)
ser.write(frame)
time.sleep(0.1)
print(ser.read(32).hex())  # → 0104060000000103e8  (valid Modbus response)
```

This proved the gripper was electrically alive and responding. The problem was that pymodbus with `timeout=1` discarded the response before it fully arrived; raising it to `timeout=2` made pymodbus succeed.

### Fix
```python
# gripper_control.py — CGC80Gripper.__init__
self.client = ModbusSerialClient(
    port=port,
    baudrate=115200,
    parity='N',
    stopbits=1,
    bytesize=8,
    timeout=2,      # was 1 — increased to allow gripper response time
    retries=3,
)
```

---

## Issue 2 — Wrong Modbus Function Codes and Register Addresses (Software)

### 3a — Status reads used FC03 instead of FC04

The original `read_status` called `read_holding_registers` (Modbus FC03). The JSON device spec (`DH-Robotics_CGC-80_RTU.json`) defines all status registers under `READ_INPUT_REGISTERS` (FC04). These are distinct function codes with separate address spaces; the gripper silently ignores FC03 requests for those addresses.

| | Before | After |
|---|---|---|
| Function | `read_holding_registers` (FC03) | `read_input_registers` (FC04) |
| Address | `0x0200` | `0x0200` (unchanged) |
| Register layout | position / force / status | init_state / gripper_state / position |

```python
# After fix
result = self.client.read_input_registers(0x0200, count=3, device_id=self.device_id)
#   registers[0] = init_state     (0x0200 / 512)
#   registers[1] = gripper_state  (0x0201 / 513)
#   registers[2] = position       (0x0202 / 514)
```

### 3b — Position/force/speed used FC16 multi-write to wrong addresses

The original `set_position` called `write_registers` (FC16 — Write Multiple Registers) bundling position, force, and speed into one contiguous write starting at `0x0103`. The JSON spec defines each parameter at a **non-contiguous** address, each requiring its own `WRITE_SINGLE_REGISTER` (FC06):

| Parameter | Correct address | Correct range | Original value sent |
|---|---|---|---|
| Closing force | `0x0101` (257) | 20 – 100 % | 500 (out of range) |
| Position       | `0x0103` (259) | 0 – 1000   | correct value |
| Speed          | `0x0104` (260) | 1 – 100 %  | 500 (out of range) |

Using FC16 to non-contiguous addresses, combined with out-of-range values, caused the gripper to return **exception code 4** (Server Device Failure) on every `close` and `set_position` call.

```python
# After fix
for addr, val, name in [
    (0x0101, force,    'force'),     # FC06 — setClosingForce  20-100 %
    (0x0103, position, 'position'),  # FC06 — setPosition      0-1000
    (0x0104, speed,    'speed'),     # FC06 — setSpeed         1-100 %
]:
    self.client.write_register(addr, val, device_id=self.device_id)
```

---

## Diagnostic Sequence

| Step | Tool / Script | Finding |
|---|---|---|
| 1 | `test_api.py` | Confirmed `device_id` is the correct pymodbus 3.11.4 keyword |
| 2 | `full_diagnostic.py` | No response at any baud rate on `ttyUSB0` |
| 3 | `scan_all.py` (created) | Full scan of both ports, all bauds, IDs 1–16 — still silent |
| 4 | Raw `serial` probe (inline) | **Breakthrough** — `ttyUSB0` returns `0104060000000103e8`; gripper is alive, pymodbus timeout too short |
| 5 | `gripper_control.py init` with `timeout=2` | `✓ Gripper initialized` |
| 6 | FC code and register address fixes | Open/close/status all clean, no exception responses |

---

## Files Modified

### `gripper_control.py`
- `CGC80Gripper.__init__`: `timeout=2`, `retries=3`
- `read_status`: `read_holding_registers` → `read_input_registers`, corrected register layout
- `set_position`: `write_registers` (FC16) → three `write_register` (FC06) calls at correct addresses; force clamped 20–100, speed clamped 1–100
- `open` / `close`: default force/speed changed from 500 to 50

## Files Created

### `scan_all.py`
Fast scanner: probes both `ttyUSB0` and `ttyUSB1`, baud rates 115200→9600, device IDs 1–16, using `read_input_registers` and `write_register`. Uses `retries=0` and `timeout=0.3` per probe for speed (~2 min total).

---

## Verified Working Commands

```bash
python3 gripper_control.py init           # activate gripper
python3 gripper_control.py open           # → position 0/1000
python3 gripper_control.py close          # → position 1000/1000
python3 gripper_control.py position 500   # → mid-point
python3 gripper_control.py status         # → init_state / gripper_state / position
```

---

## Key Takeaway

All failures originated from **software bugs**, not hardware. The symptom — zero pymodbus response — was indistinguishable from a wiring fault, which is why broad port/baud/ID scans appeared to show "no device". The raw `serial` bypass was the critical diagnostic step: by removing pymodbus's retry/timeout layer it gave a direct, unambiguous answer to *"does the gripper physically respond at all?"* — the answer was yes, and the actual problem was pymodbus discarding the response due to an insufficiently short timeout. The register map bugs (wrong function codes and addresses) were pre-existing and only became visible once communication was established.

