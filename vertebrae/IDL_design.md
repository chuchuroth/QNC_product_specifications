
## Historical Detailed Notes (Archived Reference)

The content below is retained from the old backup summary for historical context.

### Legacy Phase 1 Deliverables (Detailed)

#### 1. Clean IDL Design (`idl/ModbusRTUBridge.idl`)

Decision: use simple, focused `ModbusRTUBridge.idl` instead of comprehensive `ModbusRTU.idl`.

Created protocol-level message types:

```idl
module qnc {
    module modbus {
        struct ReadCommand { ... };         // Read registers
        struct WriteCommand { ... };         // Write single register
        struct WriteMultipleCommand { ... }; // Write multiple
        struct Response { ... };             // Raw data response
        struct BridgeStats { ... };          // Performance metrics
    }
};
```

#### 2. Generic Modbus Bridge (`modbus_rtu/examples/modbus_bridge.cpp`)

Pure pass-through layer with no device semantics:

- Polls common registers (512-514 input, 256-259 holding)
- Displays raw hex data to console
- Reports success/failure for each transaction

Usage:

```bash
./qnc.sh bridge /dev/ttyAMA0 1 115200
```

Example output:

```
[READ] Slave=1 Reg=512 Count=3 ✓
  Data: 0000 0001 03e8
[READ] Slave=1 Reg=256 Count=4 ✓
  Data: 0000 0064 0064 03e8
```

#### 3. Generic Dashboard (`modbus_rtu/examples/modbus_dashboard.cpp`)

Displays raw register data without interpretation:

- Slave ID, register addresses
- Values in hex, decimal, binary
- TX/RX counters, error statistics

#### 4. Unified Script (`modbus_rtu/scripts/qnc.sh`)

```bash
./qnc.sh build [--neurasync|--clean|--debug]
./qnc.sh scan [port] [--verbose]
./qnc.sh bridge <port> <slave> <baudrate>
./qnc.sh dashboard
```

#### 5. Configuration (`config/modbus_bridge.yaml`)

Generic UART and Modbus settings, no device-specific parameters.

### Legacy Architecture Diagram

```
┌─────────────────────────────────────────────┐
│  Robot Platform (External - NOT QNC)        │
│  ├── Device descriptors (JSON)              │
│  ├── Data parsing logic                     │
│  ├── High-level commands (open/close/grip)  │
│  └── Sends: WriteCommand(slave, reg, val)   │
└──────────────────┬──────────────────────────┘
                   │ DDS / NeuraSync
                   │ (Generic commands)
                   │
┌──────────────────▼──────────────────────────┐
│  QNC Device (This Codebase)                 │
│  ├── modbus_bridge.cpp                      │
│  │   - Polls Modbus registers               │
│  │   - Prints raw hex data                  │
│  ├── modbus_dashboard.cpp                   │
│  │   - Renders generic bridge data          │
│  └── No device semantics                    │
└──────────────────┬──────────────────────────┘
                   │ RS485 / ModbusRTU
                   ▼
        Any Modbus Device
        (gripper, sensor, motor, PLC, etc.)
```

### QNC vs External Platform Responsibility

**QNC does (generic protocol layer):**

- Raw register read/write
- RS485 communication
- Protocol error reporting
- TX/RX statistics

**QNC does not do (device semantics):**

- Interpret register values into application meaning
- Own high-level commands (`open`, `close`, etc.)
- Store/own robot-side descriptor semantics

### Legacy Build and Usage Reference

```bash
# Via unified script
cd modbus_rtu/scripts
./qnc.sh build --neurasync

# Manual
cd build
cmake .. -DBUILD_WITH_DDS=ON -DBUILD_WITH_NEURASYNC=ON -DBUILD_EXAMPLES=ON
make -j$(nproc)
```

### Philosophy

**QNC is a protocol bridge, not a device driver.**

- Keep it generic
- Keep it simple
- Keep it focused

Separation of concerns:

- **QNC**: protocol layer (RS485 ↔ DDS)
- **Robot Platform**: application layer (semantics, control logic)

---

**Updated:** February 17, 2026
**Status:** Consolidated reference file (current summary + historical archive)

