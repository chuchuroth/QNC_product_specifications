# Testing Documentation

Test logs, validation reports, and testing procedures.

## Contents

- **[gripper_test_logs.md](gripper_test_logs.md)** - DH Robotics gripper testing logs and results
- **[IDL_Code_Generation_Test.md](IDL_Code_Generation_Test.md)** - IDL code generation test results

## Overview

This directory contains:
- Hardware test logs and results
- Integration test documentation
- Validation reports
- Test procedures and protocols

## Test Coverage

### Gripper Testing
- DH Robotics gripper Modbus RTU communication
- Socket-based command interface testing
- FastDDS/NeuraSync integration tests
- Dashboard visualization validation
- Physical hardware test results

## Related Documentation

- [Protocol Specifications](../protocols/) - Protocol details being tested
- [Hardware Specifications](../hardware/) - Hardware being tested

---
---
## Testing

### IDL Code Generation Test

```bash
cd modbus-rtu-test/build

cmake ../generated \
  -DCMAKE_PREFIX_PATH="/path/to/fastdds/install"

make -j$(nproc)

# Run message type tests
./test_modbus_rtu_messages

# Run pub/sub communication test
./ModbusRTU subscriber &
./ModbusRTU publisher
```

### Expected Output

```
============================================
    ModbusRTU IDL Message Type Tests
============================================

=== Test 1: DeviceDescriptor ===
  Device ID: gripper_01
  Device Type: DH_AG95
  JSON Descriptor Length: 635 bytes
  [PASS] DeviceDescriptor created successfully

=== Test 2: DeviceState ===
  Connection State: CONNECTED
  Sequence Number: 42
  Input Registers: 1 blocks
  [PASS] DeviceState created successfully

... (all 8 tests pass)

============================================
    Test Summary
============================================
  Messages Created: 10
  All tests PASSED!
============================================
```


---
---
# QNC Gripper Integration – Clean Documentation

This document consolidates build steps, runtime commands, architecture, test status, and integration notes for the QNC gripper system on Raspberry Pi CM5.

---

# 1. DH Gripper Bridge (Python + Socket + DDS)

## Run the Bridge

### Daemon mode

```bash
python3 dh_gripper_bridge.py
```

### Interactive mode

```bash
python3 dh_gripper_bridge.py -i
```

### Test via Socket Client

```bash
echo '{"cmd": "grip"}' | nc -q 1 localhost 9876
```

The bridge exposes:

* **Socket server** (port `9876`, JSON protocol)
* Optional **FastDDS publisher**
* 10 Hz status publishing thread

This allows NeuraSync or any TCP client to trigger:

* `grip`
* `release`
* `move`
* `stop`

---

## Manual Command-Line Control (Direct Modbus Test)

```bash
sudo chmod 666 /dev/ttyUSB0
source venv/bin/activate
python qnc/tools/protocol_simulator/test_dh_gripper.py
```

---

# 2. System Services Status

## Running Services

| Service                    | PID    | Status                           |
| -------------------------- | ------ | -------------------------------- |
| DH Gripper Bridge (Python) | 325007 | Running (port 9876)              |
| NeuraSync Dashboard        | 327591 | Running (GUI visible)            |
| FastDDS Publisher (C++)    | 328064 | Publishing to `gripper_position` |

---

## Test Results

* ✅ Initial status: 0mm (open)
* ✅ Move to 47.5mm
* ✅ Grip (95mm)
* ✅ Release (0mm)
* ✅ DDS publisher captured all position changes

---

## DH Gripper Bridge Status

* Status: Running
* Socket Server: Listening on port 9876
* Log: `/tmp/gripper_bridge.log`

---

## Dashboard Configuration Note

The dashboard uses **Shared Memory (SHM)** by default, while the publisher uses **UDP multicast**.

To see topics:

1. Open dashboard
2. Go to Connection Settings
3. Change transport to **UDP**
4. Click **Refresh**
5. Discover `gripper_position`

---

# 3. Dashboard & Publisher Commands

## Run Dashboard

```bash
run_dashboard.sh
```

or manually:

```bash
cd neurasync-dashboard/build
./neurasync-dashboard
```

---

## Run Gripper Bridge Test Script

```bash
cd gripper-bridge
python3 test_bridge.py
```

---

# 4. Gripper Test Log (Modbus Parameters)

| Parameter | Value        |
| --------- | ------------ |
| Device    | /dev/ttyUSB0 |
| Baudrate  | 115200       |
| Parity    | None (8N1)   |
| Slave ID  | 1            |
| Protocol  | Modbus RTU   |

---

## Socket Fallback Mode

If DDS libraries (CycloneDDS/FastDDS) are not installed:

* Socket server still works
* Gripper can be fully controlled via JSON TCP interface
* Status can be monitored via socket protocol

---

# 5. Project Structure

## `gripper-bridge/`

* `dh_gripper_bridge.py`

  * Modbus RTU driver
  * Socket server (9876)
  * Optional FastDDS integration
  * 10 Hz status publisher

* `gripper_fastdds_publisher.cpp` (legacy)

* `gripper_neura_publisher.cpp` (NeuraSync version)

* `test_bridge.py`

* `idl/GripperMessages.idl`

---

## `gripper-configs/examples/`

* `dh_robotics_gripper.json`

  * Control registers: `0x0100–0x0103`
  * Status registers: `0x0200–0x0202`
  * 115200 baud, 8N1, slave ID 1
  * Commands: initialize, grip, release, move, stop

---

## `protocols/`

* `modbus.h`
* `tools/protocol_simulator/`

---

# 6. QNC Deployment on Raspberry Pi CM5

## Build Status

Built successfully in Release mode:

* `qnc-cm5-app`
* `test_modbus_handler`
* `test_iolink_handler`
* `test_neurasync_integration`

---

## Test Results

| Component             | Status                            |
| --------------------- | --------------------------------- |
| Modbus Handler        | Hardware not connected (expected) |
| IO-Link Handler       | ✅ Passed (mock)                   |
| NeuraSync Integration | ✅ Passed (mock DDS)               |
| Main App              | ✅ Running                         |

---

## Current Runtime Mode

* Modbus: Fallback (no `/dev/ttyAMA1`)
* IO-Link: Mock
* DDS: Mock
* Configs: Hardcoded

---

# 7. Physical Testing Readiness

## Modbus Gripper (Robotiq 2F-85)

**~85% Ready**

### Requirements

```bash
sudo apt install libmodbus-dev
```

### Test

```bash
./build/test_modbus_handler
./build/qnc-cm5-app --modbus-device /dev/ttyAMA1 --modbus-baud 115200
```

---

## IO-Link

**~10% Ready**

Missing:

* Vendor SDK
* Process data mapping
* ISDU layer
* Config example

---

# 8. FastDDS Integration Status

## Current State

* ❌ FastDDS not installed
* ❌ IDL files missing
* ⚠️ Code prepared with `#ifdef USE_REAL_FASTDDS`
* Currently using mock DDS

---

## To Enable Real FastDDS

### Install

```bash
sudo apt install libfastdds-dev libfastcdr-dev fastdds-tools
```

### Generate Code

```bash
fastddsgen -replace GripperCommand.idl
fastddsgen -replace GripperStatus.idl
```

### Rebuild

```bash
rm -rf build
cmake -B build
cmake --build build
```

---

## Recommendation

For local CM5 testing:

➡ Mock DDS is sufficient
➡ Focus on Modbus hardware validation first

---

# 9. NeuraSync Integration (Final Architecture)

## Files Created

| File                          | Purpose             |
| ----------------------------- | ------------------- |
| `idl/GripperMessages.idl`     | Message definitions |
| `gripper_neura_publisher.cpp` | NeuraSync publisher |
| `CMakeLists.txt`              | Build config        |

---

## Topics

* `qnc/gripper/status`
* `qnc/gripper/position`
* Partition: `"gripper"`

---

## Dashboard Usage

1. Click **Discover**
2. Subscribe to `qnc/gripper/position`
3. Plot `position_mm`
4. Observe 10-cycle movement test:

   * 50mm → 0mm → 95mm → stop

Publishing frequency: **10 Hz**

---

# 10. Core Modbus Driver Overview

## DH-Specific Files

| File                    | Notes                          |
| ----------------------- | ------------------------------ |
| `dh_gripper_modbus.cpp` | DH protocol implementation     |
| `dh_gripper_modbus.hpp` | Register map, states, commands |

Specific:

* Registers `0x0100–0x0202`
* Stroke 95mm
* Position encoding (permille)
* Init byte `0xA5`
* States: moving/reached/caught/dropped

---

## Reusable Components

| File                  | Reusable              |
| --------------------- | --------------------- |
| `serial_port.cpp/hpp` | POSIX serial wrapper  |
| `calcCRC16()`         | Standard Modbus CRC16 |

---

## Alternative Approach

Instead of raw Modbus implementation:

* Use `libmodbus`
* Reduces maintenance
* Handles CRC/framing automatically

---

# 11. Running Full Test with Dashboard

## Terminal 1 – Publisher

```bash
export LD_LIBRARY_PATH="..."

cd gripper-bridge/build
./gripper_neura_publisher -p /dev/ttyUSB0 -t
```

Options:

* `-p` Serial port
* `-t` 10-cycle test
* `-d` DDS domain
* `-s` Modbus slave ID

---

## Terminal 2 – Dashboard

```bash
cd neurasync-dashboard/build
./neurasync-dashboard
```

Then:

* Discover topics
* Subscribe
* Plot `position_mm`

---

# 12. Final Status Summary

| Component                | Status                          |
| ------------------------ | ------------------------------- |
| Mock Simulation          | ✅ 95% Complete                  |
| Physical Modbus          | ⚠️ Ready (verify slave address) |
| Physical IO-Link         | ❌ Not ready                     |
| JSON Config Loader       | ❌ Stub                          |
| FastDDS Real Integration | ❌ Optional                      |

---

# Conclusion

The QNC gripper system:

* Is production-quality in architecture
* Fully functional in mock mode
* Ready for physical Modbus testing
* Supports both DDS and socket control
* Cleanly separates hardware, middleware, and visualization layers

Immediate next step:
➡ Test with physical Modbus gripper and verify slave address.

---
---
# neurasync — Shared IDL Submodule

This directory is a **git submodule** pointing to `qnc-neurasync-idl`,
the single source of truth for all DDS interface definitions shared between
`qnc-firmware` and `neurasync-robot-client`.

## Contents

```
neurasync/
├── idl/
│   ├── QncBridge.idl          ← GenericCommand, GenericResponse, BridgeStatus,
│   │                             DeviceChangeEvent, CommandType, DataType
│   └── ModbusRTUBridge.idl    ← WriteCommand, ReadCommand, Response, BridgeStats
└── generated/                 ← fastddsgen output (committed — do not edit manually)
    ├── ModbusRTUBridge.hpp
    ├── ModbusRTUBridgeCdrAux.hpp / .ipp
    ├── ModbusRTUBridgePubSubTypes.hpp / .cxx
    └── ModbusRTUBridgeTypeObjectSupport.hpp / .cxx
```

## Versioning rule

Any IDL change **must bump the package version** and be coordinated across
**both workspaces** before either workspace is rebuilt.

## Initialising the submodule (after clone)

```bash
git submodule update --init --recursive
```

## Regenerating C++ types

```bash
cd neurasync/idl
fastddsgen -d ../generated -typeros2 ModbusRTUBridge.idl
# fastddsgen -d ../generated -typeros2 QncBridge.idl
```

Commit the updated `generated/` files so CI and the other workspace stay in sync.
