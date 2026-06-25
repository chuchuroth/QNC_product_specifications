# QNC - Protocol Bridge Package

**Generic communication bridge between robotic platforms and industrial devices**

QNC provides **protocol-level bridges** without device-specific semantics. It handles raw communication protocols (ModbusRTU, IO-Link) and acts as a pure pass-through layer between DDS/NeuraSync and industrial fieldbus devices.

## Design Philosophy: Why QNC Exists

### The Problem: Without QNC

**Scenario:** Your robotics company builds collaborative robots with interchangeable end-effectors (grippers, sensors, tools). You need to support:
- 5 different gripper brands (DH-Robotics, Robotiq, Zimmer, Schunk, OnRobot)
- 3 communication protocols (ModbusRTU, IO-Link, EtherCAT)
- Multiple interface types (RS485, SPI, GPIO)

**WITHOUT QNC, your robot development team must:**

1. **❌ Implement 15+ device drivers from scratch:**
   ```
   Robot ECU Code:
   ├── gripper_dh_robotics_modbus.cpp         (500 lines)
   ├── gripper_robotiq_modbus.cpp             (450 lines)
   ├── gripper_zimmer_iolink.cpp              (600 lines)
   ├── gripper_schunk_iolink.cpp              (580 lines)
   ├── sensor_sick_modbus.cpp                 (400 lines)
   ├── sensor_ifm_iolink.cpp                  (550 lines)
   └── ... (10+ more files)
   
   Total: ~6000+ lines of device-specific code IN YOUR ROBOT
   ```

2. **❌ Maintain protocol stacks in robot ECU:**
   - ModbusRTU stack (CRC, timeouts, error handling)
   - IO-Link stack (SIO modes, telegram parsing)
   - Hardware drivers (UART, SPI, GPIO control)
   - **Result:** 3000+ lines of low-level protocol code

3. **❌ Debug hardware issues in robot codebase:**
   - RS485 timing problems? Debug in robot ECU
   - IO-Link telegram errors? Debug in robot ECU
   - New device won't respond? Add device-specific quirks to robot code

4. **❌ Rebuild robot firmware for every new device:**
   ```
   Customer: "We want to use the new Robotiq 2F-85 gripper"
   
   Your team must:
   1. Write new driver code (2-3 days)
   2. Test integration (1-2 days)
   3. Rebuild robot firmware (30 minutes)
   4. Deploy to ALL robot units (hours)
   5. Risk breaking existing devices
   
   Total: 1 week per new device
   ```

5. **❌ Duplicate code across robot models:**
   ```
   robot_model_A/src/gripper_drivers/
   robot_model_B/src/gripper_drivers/    ← Same code copied!
   robot_model_C/src/gripper_drivers/    ← Same code copied!
   ```

**Cost:** ~15,000+ lines of low-level code scattered across your robot platform, weeks of work per new device, constant maintenance burden.

---

### The Solution: With QNC as Universal Interface

**WITH QNC, your robot development is simplified:**

```
┌─────────────────────────────────────────────────────────────────────┐
│ Robot Platform (Your Code)                                         │
│─────────────────────────────────────────────────────────────────────│
│  high_level_control/                                               │
│  ├── gripper_controller.py         ← 150 lines (device-agnostic)  │
│  └── motion_planner.py                                             │
│                                                                     │
│  device_descriptors/  (JSON files - no code!)                      │
│  ├── dh_ag95.json                  ← 200 lines JSON                │
│  ├── robotiq_2f85.json             ← 180 lines JSON                │
│  └── zimmer_gep5000.json           ← 190 lines JSON                │
│                                                                     │
│  Total robot code: ~500 lines (vs 15,000 before!)                  │
└─────────────────────────────────────────────────────────────────────┘
                              ↕
                    NeuraSync DDS (Protocol-agnostic)
                 qnc::modbus::WriteCommand / Response
                              ↕
┌─────────────────────────────────────────────────────────────────────┐
│ QNC Bridge (Runs on CM5, separate from robot)                      │
│─────────────────────────────────────────────────────────────────────│
│  - Handles ALL protocols (ModbusRTU, IO-Link, ...)                │
│  - Handles ALL hardware (RS485, SPI, GPIO)                         │
│  - Generic register operations ONLY                                │
│  - No device-specific knowledge                                    │
│                                                                     │
│  Works with ANY end-effector without modification!                 │
└─────────────────────────────────────────────────────────────────────┘
                              ↕
                    RS485 / IO-Link / EtherCAT
                              ↕
┌─────────────────────────────────────────────────────────────────────┐
│ Any Industrial Device (Gripper, Sensor, Motor, PLC)                │
└─────────────────────────────────────────────────────────────────────┘
```

**Architecture Principle:**
```
Robot Platform (External)     ←→     QNC (This Package)     ←→     Field Devices
- Device descriptors (JSON)           - Protocol bridges            - Grippers
- High-level commands                 - Raw register ops            - Sensors
- Application logic                   - DDS ↔ RS485/IO-Link         - Motors
- Data interpretation                 - No device semantics         - PLCs
```

**QNC handles protocols, NOT devices.**

---

### Concrete Work Saved: Real Examples

#### Example 1: Adding New Gripper Support

**WITHOUT QNC:**
```cpp
// Robot ECU code (robot_controller/src/gripper_xyz_driver.cpp)
// 500+ lines of device-specific code you must write, test, deploy

class GripperXYZ {
    int modbus_fd;
    
    bool initialize() {
        // Setup RS485 hardware
        modbus_fd = open("/dev/ttyUSB0", O_RDWR);
        // Configure UART (baudrate, parity, stop bits)
        // Implement ModbusRTU protocol stack
        // Device-specific init sequence
    }
    
    bool setPosition(int pos_mm) {
        // Translate mm → register value (device-specific formula)
        // Build Modbus RTU frame (CRC calculation, timing)
        // Send via RS485, handle retries
        // Parse response, check errors
    }
    
    // ... 400 more lines ...
};

// Result: 1 week of work, rebuild robot firmware, redeploy to fleet
```

**WITH QNC:**
```json
// Robot platform code (robot_controller/device_descriptors/gripper_xyz.json)
// 200 lines of JSON - copy from manufacturer's manual

{
  "metadata": {
    "name": "GripperXYZ",
    "manufacturer": "ACME Corp",
    "model": "XYZ-2000"
  },
  "communication": {
    "protocol": "modbus_rtu",
    "baudrate": 115200,
    "slave_address": 1
  },
  "registers": {
    "position_target": {
      "address": 1000,
      "type": "uint16",
      "unit": "mm",
      "scaling": 10,
      "description": "Target position (value * 0.1 = mm)"
    }
  }
}
```

```python
# Robot platform code (uses existing generic controller)
from robot_controller import GripperController

gripper = GripperController("dh_ag95.json")  # or "gripper_xyz.json"
gripper.move_to(position_mm=50, force_percent=80)

# GripperController translates using JSON descriptor and publishes DDS:
# DDS: WriteCommand{slave=1, reg=1000, value=500}  # 50mm * 10
```

**Work saved:** 1 week → 30 minutes (copy JSON from manual)

#### Example 2: Supporting 10 Different Devices

**WITHOUT QNC:**
```
Robot firmware size: 15,000+ lines device code
Time to add 10 devices: 10 weeks
Maintenance burden: Every protocol change = 10 drivers to update
```

**WITH QNC:**
```
Robot firmware size: 500 lines generic code
Time to add 10 devices: 5 hours (copy 10 JSON descriptors)
Maintenance burden: Zero - descriptors are data, not code
```

#### Example 3: Hardware Protocol Debugging

**WITHOUT QNC:**
```
Problem: "RS485 communication unstable"

Your robot team must:
1. Debug UART driver code in robot ECU
2. Analyze RS485 timing with oscilloscope
3. Modify robot firmware ModbusRTU stack
4. Rebuild and redeploy robot software
5. Risk breaking other devices

Time: 2-3 days
```

**WITH QNC:**
```
Problem: "RS485 communication unstable"

QNC team handles it:
1. Issue is isolated to QNC bridge
2. Fix once in QNC codebase
3. No robot firmware changes needed
4. Update QNC on CM5 only (systemd restart)

Time: 1 hour, zero robot impact
```

---

### Summary: Development Effort Comparison

| Task | Without QNC | With QNC | Time Saved |
|------|-------------|----------|------------|
| Add new gripper support | 1 week | 30 min | **97% faster** |
| Support 10 devices | 10 weeks | 5 hours | **99% less code** |
| Protocol debugging | 2-3 days | 1 hour | **95% faster** |
| Maintain device drivers | Continuous burden | Zero (JSON data) | **Eliminated** |
| Robot firmware size | 15,000+ lines | 500 lines | **97% smaller** |
| Deployment risk | High (full rebuild) | Zero (JSON change) | **Risk eliminated** |

**QNC Result:** Robot development focuses on robotics, not fieldbus protocols.

> 📖 **For authoritative architecture, interfaces, and timing content, see [docs/QNC_INTERFACE_CONTROL_DOCUMENT.md](docs/QNC_INTERFACE_CONTROL_DOCUMENT.md)**

---

## Features

### ModbusRTU Bridge
- Generic register read/write operations
- RS485 communication with automatic mode switching
- NeuraSync DDS integration (generic commands/responses)
- Raw register dashboard
- Docker support with FastDDS 3.x

### IO-Link Bridge
- MAX22515 IO-Link transceiver drivers
- COM1/COM2/COM3 mode support
- PIN mode and I2C mode control
- Python-based GPIO drivers
- Generic process data exchange

## Directory Structure

```
qnc-cm5/
├── modbus_rtu/                        # ModbusRTU Bridge
│   ├── include/
│   │   └── modbus_rtu.h              # Header-only protocol library
│   ├── examples/
│   │   ├── modbus_bridge.cpp         # **Main: Generic bridge**
│   │   ├── modbus_dashboard.cpp      # **Raw register dashboard**
│   │   ├── modbus_rtu_example.cpp    # Test tool
│   │   └── gripper_control.cpp       # (Legacy, deprecated)
│   └── README.md                      # **Read this first!**
│
├── robot_side/                        # ← NOT part of QNC firmware
│   └── device_descriptors/           # Reference JSON descriptors
│       ├── DH-Robotics_CGC-80_RTU.json
│       ├── DH_AG95_descriptor.json
│       ├── dh_robotics_gripper.json
│       └── robotiq_2f_modbus.json    # Copy these to your robot platform
│
├── iolink/                            # IO-Link Bridge
│   ├── docs/                          # MAX22515 datasheet
│   ├── iodd/                          # Device descriptions (reference)
│   └── README.md
│
├── idl/
│   └── ModbusRTUBridge.idl           # **Generic command types**
├── schemas/
│   └── ModbusRTU_DeviceDescriptor_Schema.json
├── config/
│   └── modbus_bridge.yaml            # Bridge configuration
├── external/                          # Third-party IDL packages
├── build/                             # Build output
└── CMakeLists.txt
```

## Quick Start

### ModbusRTU Gripper (Docker)

Build and run with NeuraSync support:

```bash
# Build DockerBridge

**QNC provides a generic bridge - no device-specific knowledge!**

```bash
# Build with NeuraSync support
mkdir build && cd build
cmake .. -DBUILD_WITH_NEURASYNC=ON -DBUILD_EXAMPLES=ON
make -j$(nproc)

# Start generic Modbus bridge
# (Connects to ANY Modbus device at slave 1, 115200 baud)
./modbus_bridge /dev/ttyAMA0 1 115200

# Monitor raw register data in another terminal
./modbus_dashboard
```

What you'll see:
```
═══════════════════════════════════════
  QNC ModbusRTU Bridge Dashboard
═══════════════════════════════════════
┌─ Slave ID: 1 ────────────────────┐
│ Addr   Hex      Decimal           │
│ 1000   0x03E8   1000              │
│ 1001   0x0032   50                │
└───────────────────────────────────┘
```

**Your robot platform** sends commands via DDS:
```cpp
// Robot platform code (external to QNC)
WriteCommand cmd;
cmd.slave_id = 1;
cmd.register_address = 1000;
cmd.value = 0x0100;
qnc_publisher.publish(cmd);  // QNC executes and returns raw Response
```

### IO-Link Bridge

```bash
# Install dependencies
sudo apt-get install python3-serial python3-libgpiod

# Run generic IO-Link driver (PIN mode)
cd iolink/drivers
sudo python3 zimmer_gripper_gpiod.py  # Works with any IO-Link device

# Or scan with I2C mode
cd iolink/scripts
python3 scant/Import Docker Image (Offline)

On the build machine:
```bash
# Build the image
docker build -t modbus-rtu-protocol:neurasync .

# Export to a tar file
docker save modbus-rtu-protocol:neurasync -o modbus-rtu-protocol.tar

# Compress for transfer (optional)
gzip modbus-rtu-protocol.tar
```

On the target machine:
```bash
# Copy the tar file to target machine, then load it
docker load -i modbus-rtu-protocol.tar
# Or if compressed:
gunzip -c modbus-rtu-protocol.tar.gz | docker load

# Verify the image is loaded
docker images | grep modbus-rtu-protocol

# Run the application
docker run --rm -it --device=/dev/ttyUSB0 --network=host \
  modbus-rtu-protocol:neurasync gripper_control /dev/ttyUSB0 1 115200
```

### Option 2: Docker Registry (Online)

Push to Docker Hub or private registry:
```bash
# Tag for your registry
docker tag modbus-rtu-protocol:neurasync your-registry/modbus-rtu-protocol:neurasync

# Push to registry
docker push your-registry/modbus-rtu-protocol:neurasync
```

On target machine:
```bash
# Pull from registry
docker pull your-registry/modbus-rtu-protocol:neurasync

# Run
docker run --rm -it --device=/dev/ttyUSB0 --network=host \
  your-registry/modbus-rtu-protocol:neurasync gripper_control /dev/ttyUSB0 1 115200
```

### Option 3: Docker Compose

Copy the project to the target machine and use docker-compose:
```bash
# Start gripper control and dashboard
docker-compose up -d gripper-control gripper-dashboard

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

## Hardware Setup (Raspberry Pi / CM5)

### RS485 Wiring to DH Gripper

Connect the RS485 transceiver module to the Pi's UART0 pins:

| Pi Pin | GPIO | Function | RS485 Module |
|--------|------|----------|--------------|
| Pin 8  | GPIO14 (TXD) | Transmit | **RXD / RO** (Receiver Output) |
| Pin 10 | GPIO15 (RXD) | Receive  | **TXD / DI** (Driver Input) |
| Pin 6  | GND          | Ground   | GND |

> **Important:** The TX/RX lines are **crossed** - the Pi's TXD connects to the RS485 module's RXD (RO), and the Pi's RXD connects to the module's TXD (DI). This is because the Pi transmits what the module receives, and vice versa.

### Enable UART on Raspberry Pi

Add to `/boot/firmware/config.txt`:
```
enable_uart=1
```
Then reboot. The gripper will be available at `/dev/ttyAMA0`.

### Serial Port Permissions

```bash
sudo usermod -aG dialout $USER
# Logout/login for changes to take effect
```

### Native Quick Start (without Docker)

```bash
# Terminal 1 - Monitor gripper state
./build/gripper_dashboard

# Terminal 2 - Control gripper
./build/gripper_control /dev/ttyAMA0 1 115200
```

## Target Machine Requirements

For running the Docker container on other machines:

- **Docker Engine** 20.10 or later
- **RS485 USB adapter** (e.g., `/dev/ttyUSB0`) or **UART + RS485 transceiver** (`/dev/ttyAMA0`) for gripper communication
- **Network access** for NeuraSync DDS communication (uses UDP multicast)
- **Linux aarch64 / x86_64**

## Building from Source

### Dependencies

For NeuraSync support, you need:
- FastDDS 3.x
- FastCDR 2.x
- neura-sync library
- neura_sync_idl_tools

### Build Steps

```bash
# Clone dependencies (if not using Docker)
# Copy neura-sync and neura_sync_idl_tools to this directory

mkdir build && cd build
cmake .. -DBUILD_WITH_NEURASYNC=ON -DENABLE_LOGGING=ON
make -j$(nproc)
```

### CMake Options

| Option | Default | Description |
|--------|---------|-------------|
| `BUILD_EXAMPLES` | ON | Build example applications |
| `BUILD_WITH_DDS` | OFF | Build with raw FastDDS support |
| `BUILD_WITH_NEURASYNC` | OFF | Build with NeuraSync library |
| `ENABLE_LOGGING` | OFF | Enable debug logging |

## NeuraSync Integration

When built with `BUILD_WITH_NEURASYNC=ON`, the gripper_control publishes state to:

- **Topic:** `qnc/modbus/device_state`
- **Type:** `DeviceState`

The neurasync-dashboard can subscribe and visualize the data using the `QNC_ModbusRTU` connection profile.

### DeviceState Fields

| Field | Type | Description |
|-------|------|-------------|
| device_id | string | Gripper identifier |
| timestamp_sec | uint32 | Timestamp seconds |
| timestamp_nsec | uint32 | Timestamp nanoseconds |
| init_state | uint16 | 0=not_init, 1=initializing, 2=ready |
| gripper_state | uint16 | 0=in_motion, 1=pos_reached, 2=caught, 3=dropped |
| position | uint16 | Raw position (0-1000) |
| force | uint16 | Force percentage |
| speed | uint16 | Speed percentage |
| online | bool | Connection status |

## Usage Examples

### Basic ModbusRTU

```cpp
#include "modbus_rtu.h"

modbus_rtu mb("/dev/ttyUSB0", 115200, 8, 1, 'N');
mb.set_slave_id(1);
mb.connect();

uint16_t position;
mb.read_input_registers(514, 1, &position);
mb.write_register(259, 500);  // Set target position

mb.close();
```

### Gripper Control

```bash
# With Docker
docker run --rm -it --device=/dev/ttyUSB0 --network=host \
  modbus-rtu-protocol:neurasync gripper_control /dev/ttyUSB0 1 115200 gripper_01

# Arguments: <serial_port> <slave_id> <baudrate> [device_id]
```

## License

Copyright (C) 2025 NeuraSync

---
---
---
# QNC Architecture Refactoring Summary

> **Why these changes?** See [README.md - Design Philosophy](../README.md#design-philosophy-why-qnc-exists) for the complete rationale of QNC as a universal protocol bridge that eliminates 99% of device driver development work.

## Current Consolidated Status (Reference)

This document combines the prior `REFACTORING_SUMMARY.md` and `REFACTORING_SUMMARY.md.backup` into one reference file.

### ✅ Phase 1: Core Architecture (Completed - Feb 17, 2026)

The QNC package was restructured to act as a **pure protocol bridge** with no device-specific semantics.

#### Deliverables

1. **Clean IDL Design** (`idl/ModbusRTUBridge.idl`)
   - Decision: keep focused `ModbusRTUBridge.idl` (50 lines) and retire large legacy `ModbusRTU.idl`.
   - Generated types: `ReadCommand`, `WriteCommand`, `WriteMultipleCommand`, `Response`, `BridgeStats`.

2. **Generic Modbus Bridge** (`modbus_rtu/examples/modbus_bridge.cpp`)
   - Pass-through layer for raw register operations and polling.

3. **Generic Dashboard** (`modbus_rtu/examples/modbus_dashboard.cpp`)
   - Generic raw-register dashboard; device-specific dashboard deprecated.

4. **Unified Script** (`modbus_rtu/scripts/qnc.sh`)
   - One entry point for build, scan, bridge, and dashboard.

### ✅ Phase 2: DDS Integration (Completed - Feb 17, 2026)

- Build system generates IDL types.
- NeuraSync publish/subscribe integration implemented.
- Bridge publishes responses and stats.
- Dashboard subscribes and renders live raw-register telemetry.

### ✅ Phase 3: Cleanup (Completed - Feb 17, 2026)

Deprecated files moved under `deprecated/` where applicable, preserving reference history while keeping active code generic.

### Key Decisions

1. One active IDL for Modbus bridge semantics.
2. One active generic dashboard.
3. Device semantics stay out of QNC runtime path.
4. Keep QNC focused as a protocol bridge.

### Summary

QNC now operates as a clean, generic ModbusRTU protocol bridge with:

- Real-time DDS publishing/subscribing via NeuraSync
- Live raw register visualization
- Zero device-specific semantics in active runtime flow
- Maintainable structure with legacy references preserved in deprecated/archive paths

---
