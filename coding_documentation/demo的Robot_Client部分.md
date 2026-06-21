# NeuraSync Robot Client

**Workspace role**: Robot-side application — device semantics, DDS publisher  
**Runs on**: Robot computer (x86-64 or robot ECU)  
**Counterpart**: [`qnc-firmware`](../qnc-firmware) on Raspberry Pi CM5

---

## Architecture

```
neurasync-robot-client (this repo)        qnc-firmware (Pi CM5)
────────────────────────────────          ────────────────────────
  Device descriptors (JSON)               Protocol translation
  Application logic                       Hardware abstraction
  DDS publisher → command topics          DDS subscriber ← command topics
  DDS subscriber ← response topics        DDS publisher → response topics
```

Communication uses **FastDDS over UDP multicast** on domain 0.  
The interface contract is defined in `neurasync/idl/` (shared submodule → `qnc-neurasync-idl`).

---

## Quick Start

### Prerequisites

- CMake ≥ 3.16
- FastDDS 3.x (`/opt/fastdds/install`)
- C++17 compiler

### Build

```bash
cmake -B build \
  -DCMAKE_PREFIX_PATH=/opt/fastdds/install \
  -DBUILD_WITH_DDS=ON
cmake --build build -j$(nproc)
```

### Run gripper demo

```bash
# Against a live QNC on the LAN
./build/dds_command_publisher demo_all --cycles 30 --delay-ms 4000
```

### Run tests (DDS loopback — no QNC hardware needed)

```bash
# No in-repo test suite is retained in the cleaned production workspace.
# Use deployment validation procedure in docs/DEPLOY_MEMO.md.
```

---

## Directory Layout

```
neurasync-robot-client/
├── CMakeLists.txt
├── README.md
├── neurasync/                     ← git submodule: qnc-neurasync-idl
│   ├── idl/
│   │   └── ModbusRTUBridge.idl
│   └── generated/                 ← fastddsgen output (committed)
├── device_descriptors/            ← Device semantic database
│   ├── ModbusRTU_DeviceDescriptor_Schema.json
│   ├── DH_AG95_descriptor.json
│   ├── dh_robotics_gripper.json
│   ├── DH-Robotics_CGC-80_RTU.json
│   └── robotiq_2f_modbus.json
├── include/
│   ├── device_descriptor_parser.hpp
│   └── brainco_revo1.hpp
├── src/
│   └── gripper_control_dds.cpp
├── scripts/
│   └── 99-qnc-grippers.rules
└── docs/
  ├── DEPLOY_MEMO.md
  ├── DETECTION_METHODS_ANALYSIS.md
  └── NEURASYNC_SOURCE_OF_TRUTH.md
```

---

## DDS Topics Published / Subscribed

| Topic | Type | Direction |
|-------|------|-----------|
| `qnc/bridge/command` | `GenericCommand` | **Publish** |
| `qnc/modbus/write_cmd` | `WriteCommand` | **Publish** |
| `qnc/modbus/read_cmd` | `ReadCommand` | **Publish** |
| `qnc/bridge/response` | `GenericResponse` | Subscribe |
| `qnc/bridge/status` | `BridgeStatus` | Subscribe |
| `qnc/modbus/response` | `Response` | Subscribe |
| `qnc/modbus/stats` | `BridgeStats` | Subscribe |
| `qnc/bridge/device_event` | `DeviceChangeEvent` | Subscribe |

DDS Domain ID: **0** | QoS: RELIABLE, KEEP_LAST (depth 10 for cmd/resp, 1 for status)

---

## What this workspace does NOT contain

| Excluded item | Reason |
|--------------|--------|
| `include/protocol_*.h` | Protocol internals — QNC firmware side only |
| `include/modbus_rtu*.h` | Hardware protocol details — QNC side only |
| `include/unified_*.h` | Bridge internals — QNC side only |
| `iolink/` | Hardware driver for IO-Link — QNC side only |

---

## Interface Contract (`neurasync/`)

The `neurasync/` directory is a git submodule pointing to `qnc-neurasync-idl`.  
**Do not edit IDL files directly here** — changes must go through the shared repo and be coordinated with `qnc-firmware` before rebuilding either workspace.

---

## Related

- [QNC Interface Control Document](docs/QNC_INTERFACE_CONTROL_DOCUMENT.md)
- [Product Requirements](docs/QNC_Product_Requirements_Document_v2.md)
- [Gripper Quickstart](docs/QUICKSTART_gripper_control.md)
- [Refactoring Summary](docs/REFACTORING_SUMMARY.md)
