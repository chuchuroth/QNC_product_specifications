_Please note that this demo may contain content that contradicts the product guidelines. For the authoritative reference, please consult the formal product documentation on Confluence, which serves as the single source of truth._


# QNC Bridge — Deployment Memo

**Document ref:** QNC-ICD-001  
**Date:** 2026-03-25  
**Branch:** `PROTOTYPE_WORKSPACE_PLAN`  
**Platform:** Raspberry Pi CM5

---

## 1. System Architecture

### 1.1 High-Level Overview

```
┌─────────────────────────────┐      LAN (UDP multicast)      ┌───────────────────────────────────────┐
│   Robot Platform Computer   │ ◄── FastDDS domain 0 ──────► │       Raspberry Pi CM5  (QNC)         │
│                             │                               │                                       │
│  dds_command_publisher      │  PUB qnc/gripper/command ───►│  qnc_bridge                           │
│  ( DDS gripper command run) │                               │    subscribes gripper commands        │
│                             │◄── qnc/gripper/state          │    executes Modbus RTU on RS485       │
│                             │                               │                                       │
│                             │                               │  /dev/gripper_ag160 ──► AG-160        │
└─────────────────────────────┘                               │  /dev/gripper_cgc80 ──► CGC-80        │
                                                              │  /dev/gripper_dh56  ──► DH-5-6        │
                                                              └───────────────────────────────────────┘
```

| Machine | Role | Binary | Serial hardware |
|---------|------|--------|-----------------|
| Robot Platform Computer | Publishes DDS commands | `dds_command_publisher` | None |
| Raspberry Pi CM5 (QNC) | Bridge — DDS → Modbus RTU → RS485 | `qnc_bridge` | 3 × FTDI USB-RS485 |

### 1.2 Two-Workspace Architecture

In the prototype phase, the single monorepo splits into **two independent workspaces** running on separate physical machines. The architectural boundary between them is the **NeuraSync Communication Interface** defined in the ICD (Section 6).

```
┌────────────────────────────────────┐          ┌─────────────────────────────────────┐
│  WORKSPACE 1: qnc-firmware         │          │  WORKSPACE 2: neurasync-robot-client│
│  Hardware: Pi CM5 + RS485/IO-Link  │          │  Hardware: separate computer        │
│  Role: QNC bridge                  │          │  Role: Robot controller             │
│                                    │          │                                     │
│  • Protocol translation            │          │  • Device descriptors (JSON)        │
│  • Hardware abstraction            │          │  • Application logic                │
│  • NeuraSync DDS subscriber        │          │  • NeuraSync DDS publisher          │
│    (receives commands)             │          │    (sends commands)                 │
│  • NeuraSync DDS publisher         │    LAN   │  • NeuraSync DDS subscriber         │
│    (sends responses/status)        │◄────────►│    (receives responses/status)      │
│                                    │          │                                     │
│  NO device semantics               │          │  ALL device semantics               │
│  NO JSON descriptors               │          │  ALL device-specific code           │
└────────────────────────────────────┘          └─────────────────────────────────────┘
              ▲                                                   ▲
              │       Shared interface contract (git submodule)   │
              └──────────────── qnc-neurasync-idl ───────────────┘
                          (IDL files + generated DDS types)
```

The **only** coupling between the two workspaces is the NeuraSync DDS topic definitions. These are kept as a standalone shared package (`qnc-neurasync-idl`) so neither workspace can silently break the other.

### 1.3 Pi Dev Setup vs. Production Architecture

The Pi is currently **collapsing two roles** into one machine for hardware bringup:

| Layer | Production owner | Pi dev setup |
|---|---|---|
| High-level commands, JSON descriptors | Robot ECU (separate machine) | Pi (same) |
| NeuraSync DDS pub/sub | Inter-machine network | Loopback / single process |
| `modbus_rtu` stack, `qnc_system_manager` | **QNC firmware on CM5** | **Pi — this IS the QNC role** |
| RS485 electrical conversion | XY-485 inside QNC enclosure | XY-485 on Pi header |
| Gripper | Field device | DH gripper |

Running both roles on one Pi is the correct approach for bringup and integration testing.

### 1.4 QNC as a Combined Hardware + Software Product

```
QNC Module (the shipping product)
├── CM5 compute module     ← runs qnc_system_manager, modbus_rtu stack, future stacks
├── XY-485 / MAX485        ← IF-009: RS485 hardware interface (Modbus RTU)
├── [MAX22515]             ← IF-010: IO-Link hardware interface (planned — chip researched in iolink/)
└── Ethernet PHY           ← IF-011: EtherCAT / EtherNet/IP (future)

Robot ECU (customer's machine)
└── publishes/subscribes NeuraSync DDS topics only — the only interface boundary
```

The **NeuraSync DDS interface (IF-001 through IF-008) remains identical** regardless of which physical protocol is active. The robot platform never needs to know which transceiver chip is on the QNC board — this is the core value proposition of QNC.

#### Adding Protocols to QNC

| Protocol | Software addition | Hardware addition |
|---|---|---|
| Modbus RTU (current) | `modbus_rtu` stack, `ModbusRTUAdapter` | XY-485 / MAX485 |
| IO-Link (next) | IO-Link SDI/SDO telegram stack | MAX22515 (already in `iolink/docs/`) |
| EtherCAT (future) | SOEM master stack | ET1100 ASIC + Ethernet PHY |
| EtherNet/IP (future) | CIP stack | Same Ethernet PHY |

---

## 2. Interface Contract (DDS Topics)

### 2.1 Mandatory Topics (IF-001 to IF-003)

| Topic | Type | Direction | Description |
|-------|------|-----------|-------------|
| `qnc/bridge/command` | `GenericCommand` | Robot → QNC | Send a protocol-agnostic command |
| `qnc/bridge/response` | `GenericResponse` | QNC → Robot | Receive the result |
| `qnc/bridge/status` | `BridgeStatus` | QNC → Robot | Heartbeat / bridge health (1 Hz) |

### 2.2 Gripper Topics — Currently Implemented

| Topic | Direction | Publisher | Subscriber | QoS |
|-------|-----------|-----------|------------|-----|
| `qnc/gripper/command` | Robot → QNC | `dds_command_publisher` | `qnc_bridge` | KEEP_LAST(10) |
| `qnc/gripper/state`   | QNC → Robot | `qnc_bridge` | `dds_command_publisher` | KEEP_LAST(10) |

### 2.3 Device Change Notification (IF-004)

| Topic | Type | Direction | Description |
|-------|------|-----------|-------------|
| `qnc/bridge/device_event` | `DeviceChangeEvent` | QNC → Robot | Hot-swap / connect / error |

### 2.4 DDS QoS (from ICD Section 6.2)

- Domain ID: `0`
- Reliability: `RELIABLE`
- History: `KEEP_LAST (depth: 10)` for command/response; `KEEP_LAST (depth: 1)` for status

### 2.5 Shared IDL Package — `qnc-neurasync-idl`

This standalone git repository is the **single source of truth** for the DDS interface. Both workspaces reference it as a git submodule at `neurasync/`.

```
qnc-neurasync-idl/
├── README.md
├── CMakeLists.txt
├── idl/
│   ├── GripperCommand.idl         ← generic gripper command contract
│   ├── GripperState.idl           ← generic gripper state contract
│   └── GripperCapabilities.idl    ← transport + capability metadata
└── generated/                     ← committed auto-generated files (fastddsgen output)
  └── gripper/                   ← committed Fast DDS generated files for gripper IDLs
```

**Versioning rule:** Any change to an IDL file **must bump the package version** and be coordinated across both workspaces before either workspace is rebuilt.

**Regenerating IDL types:** The generated files in `neurasync/generated/` are committed. If the IDL changes, regenerate manually:

```bash
cd neurasync-robot-client
cmake --build build --target gripper_idl_gen
```

The project wires regeneration through the CMake target above; use it after editing the gripper IDLs.

---

## 3. Hardware Setup

### 3.1 GPIO Wiring: Pi CM5 → XY-485 → Modbus RTU Gripper

The XY-485 is a **half-duplex UART ↔ RS485 transceiver** (MAX485/SP3485 compatible). Only **three Pi signals** are needed: UART TX, UART RX, and a direction control pin (DE/RE).

| CM5 Pin | GPIO (BCM) | Signal | XY-485 Pin |
|---------|-----------|--------|------------|
| Pin 8   | GPIO14    | UART0 TX | DI (Driver Input) |
| Pin 10  | GPIO15    | UART0 RX | RO (Receiver Output) |
| Pin 11  | GPIO17    | DE/RE direction | DE + RE (tied together) |
| Pin 1   | —         | 3.3 V supply | VCC |
| Pin 6   | —         | GND | GND |

XY-485 field side:
- **A (D+)** and **B (D−)** → gripper RS485 terminals
- **120 Ω termination resistor** across A–B at the gripper end (required at ≥19200 baud)

### 3.2 Boot Configuration

```ini
# /boot/config.txt
enable_uart=1
dtoverlay=disable-bt   # frees UART0 from Bluetooth, assigns it to GPIO14/15
```

Verify: `ls -la /dev/serial0` should symlink to `ttyAMA0`.

### 3.3 Linux RS485 Kernel Mode (Preferred)

GPIO17 doubles as the `RTS0` alt-function pin for UART0. With `TIOCSRS485`, the kernel asserts DE/RE HIGH before transmit and LOW after the last stop bit — no userspace GPIO toggling required.

```cpp
#include <linux/serial.h>
struct serial_rs485 rs485conf = {};
rs485conf.flags |= SER_RS485_ENABLED;
rs485conf.flags |= SER_RS485_RTS_ON_SEND;
rs485conf.flags &= ~SER_RS485_RTS_AFTER_SEND;
ioctl(fd, TIOCSRS485, &rs485conf);
```

Add this immediately after `open("/dev/ttyAMA0", ...)` in `modbus_rtu::connect()`.

### 3.4 UART Settings (from DH Device Descriptors)

| Parameter | Value |
|---|---|
| Baud rate | 115200 |
| Data bits | 8 |
| Parity | None |
| Stop bits | 1 |
| Flow control | None |
| Response timeout | 100 ms |
| Inter-frame delay | 50 ms |

### 3.5 End-to-End Data Flow

```
Pi userspace
  modbus_rtu::write_register(addr, val)
    └─ builds RTU frame: [SlaveID][FC][AddrHi][AddrLo][ValHi][ValLo][CRC]
    └─ write(fd, frame, N) → /dev/ttyAMA0

UART0 hardware (GPIO14, 3.3 V logic)
    └─ serial bits out

XY-485 board
  DE/RE ← GPIO17 HIGH (kernel asserts before TX)
  DI → [MAX485 driver] → A/B differential on RS485 bus
  DE/RE ← GPIO17 LOW  (kernel de-asserts after last stop bit)
  RO ← [MAX485 receiver] ← A/B differential (gripper response)

RS485 bus (twisted pair, ≤100 m @ 115200 baud)
    └─ gripper receives frame, validates CRC, executes FC06/FC04
    └─ gripper sends response

Pi userspace
  read(fd, buf, max) → modbus_rtu::receive_frame()
    └─ parse_read_response() / parse_write_response()
    └─ CRC check → return data to caller
```

---

## 4. DH Gripper Register Map & Command Flow

### 4.1 Register Map (CGC-80 / AG-95 / AG-160)

QNC's entire job at the DDS interface level is to accept raw `{register_address, value}` pairs and execute the corresponding Modbus transaction. This is by design (QNC-ICD-001 §1.4):

> *QNC handles protocols, NOT devices. All device-specific semantics reside in JSON descriptors on the robot platform side.*

| Register (dec) | Register (hex) | FC | Value range | Meaning |
|---|---|---|---|---|
| 256 | `0x0100` | FC06 write | `0x0001` | Initialize / activate |
| 257 | `0x0101` | FC06 write | 20–100 | Closing force % |
| 258 | `0x0102` | FC06 write | 1–100 | Speed % (AG-160-95) |
| 259 | `0x0103` | FC06 write | 0–1000 | Target position (0=open, 1000=closed) |
| 260 | `0x0104` | FC06 write | 1–100 | Speed % (CGC-80) |
| 512 | `0x0200` | FC04 read | — | Init state |
| 513 | `0x0201` | FC04 read | — | Gripper state |
| 514 | `0x0202` | FC04 read | — | Current position |

> **Note:** AG-160-95 uses register 258 for speed; CGC-80 uses register 260. Select the correct model via `QNC_GRIPPER_MODEL`.

### 4.2 Production Command Flow

```
Robot ECU
  reads JSON descriptor → resolves "initialize" → reg=256, val=1
  publishes: WriteCommand { slave_id=1, register_address=256, value=1 }
      │
      ▼  [NeuraSync DDS]
QNC bridge (CM5)
  modbus_rtu::write_register(0x0100, 0x0001)
      │
      ▼  [RS485 via XY-485]
Gripper activates
      │
      ▼  [RS485 response frame]
QNC publishes: Response { slave_id=1, error_code=0 }
      │
      ▼  [NeuraSync DDS]
Robot ECU receives confirmation
```

---

## 5. Prerequisites

| Package | Version | Install |
|---------|---------|---------|
| CMake | ≥ 3.16 | `sudo apt install cmake` |
| FastDDS | 3.4.1 | `sudo apt install fastdds-dev` |
| fastcdr | 2.3.4 | installed with FastDDS |
| fastddsgen | 2.3.0 | `sudo apt install fastddsgen` |
| g++ (C++17) | ≥ 9 | `sudo apt install build-essential` |
| nlohmann-json3-dev | 3.11.3 | `sudo apt install nlohmann-json3-dev` |

Both machines must be on the **same LAN segment**. UDP multicast must not be blocked. If multicast is unavailable, use a unicast peer config:

```bash
export FASTRTPS_DEFAULT_PROFILES_FILE=~/fastdds_unicast.xml
```

No `CMAKE_PREFIX_PATH` is required — FastDDS 3.4.1 cmake config is at the system location (`/usr/share/fastdds/cmake/`).

---

## 6. Build

### 6.1 Pi CM5 — `qnc_bridge`

```bash
cmake -S . -B build -DBUILD_WITH_DDS=ON
cmake --build build --target qnc_bridge
```

### 6.2 Robot Platform Computer — `dds_command_publisher`

```bash
cmake -S . -B build -DBUILD_WITH_DDS=ON
cmake --build build --target dds_command_publisher
```

### 6.3 Test Binaries (Robot Computer)

```bash
cmake -B build -DBUILD_WITH_DDS=ON -DBUILD_TESTS=ON
cmake --build build -j$(nproc)
```

### 6.4 Build Outputs

| Binary | Size | Build flags | Purpose |
|---|---|---|---|
| `build/qnc_bridge` | — | `BUILD_WITH_DDS=ON` | QNC bridge daemon (Pi CM5) |
| `build/dds_command_publisher` | — | `BUILD_WITH_DDS=ON` | DDS command runner (Robot Computer) |
| `build/libneurasync_types.so` | ~871 KB | shared lib | Pre-generated DDS types (`WriteCommand` etc.) |

### 6.5 `libneurasync_types.so` Runtime Dependency

`dds_command_publisher` and `qnc_bridge` both link against the shared library `libneurasync_types.so`. The dynamic loader needs it at runtime:

```bash
# Required if not installing to a system lib path:
LD_LIBRARY_PATH=build ./build/dds_command_publisher demo_all --cycles 1 --delay-ms 500
```

Alternatively, install: `sudo cmake --install build` copies `libneurasync_types.so` to `/usr/local/lib`.

### 6.6 `QNC_PROJECT_ROOT` CMake Define

`test_neurasync_idl` is compiled with `-DQNC_PROJECT_ROOT="<cmake source dir>"`. This is used in Suite 5 to:
1. Locate the device descriptor JSON — e.g., `$QNC_PROJECT_ROOT/device_descriptors/DH-Robotics_AG160-95_RTU.json`; selected at runtime by `QNC_GRIPPER_MODEL`
2. Locate the subprocess binary: `$QNC_PROJECT_ROOT/build/gripper_control_dds`

The value is set at cmake time to `${CMAKE_SOURCE_DIR}`, so the test always finds files relative to the repo regardless of where the binary is invoked from.

---

## 7. Demo — Startup Procedure

> **Start order:** launch `qnc_bridge` first, then `dds_command_publisher`.

```bash
# Terminal on Pi CM5
LD_LIBRARY_PATH=/home/pi/Documents/robot_platform/neurasync-robot-client/build:$LD_LIBRARY_PATH QNC_DE_RE_GPIO=none ./build/qnc_bridge
```

Wait for `[Bridge] Waiting for commands (Ctrl-C to stop)...`, then:

```bash
# Terminal on Robot Platform Computer
cd build && ./dds_command_publisher demo_all
```

This initialises all three grippers, waits for calibration, then runs synchronized open/close cycles across AG-160 (slave 1), CGC-80 (slave 2), and DH-5-6 (slave 3).

### dds_command_publisher Usage

```bash
dds_command_publisher demo_all [cycles] [--cycles N] [--delay-ms N] [--init-wait-ms N] [--domain N]
dds_command_publisher --gripper <ag160|cgc80|dh56> <init|open|close|status|demo> [cycles] [--delay-ms N] [--domain N]
```

---

## 8. Pi CM5 Configuration

### 8.1 Environment Variables

With udev symlinks installed, **no environment variables are required** beyond `QNC_DE_RE_GPIO=none`.

| Variable | Default | Description |
|----------|---------|-------------|
| `QNC_SERIAL_PORT` | `/dev/gripper_ag160` | Primary RS485 port (AG-160) |
| `QNC_SERIAL_PORT_2` | `/dev/gripper_cgc80` | Secondary RS485 port (CGC-80) |
| `QNC_SERIAL_PORT_3` | `/dev/gripper_dh56` | Tertiary RS485 port (DH-5-6) |
| `QNC_SLAVE_ID_1` | `1` | DDS slave_id routed to port 1 |
| `QNC_SLAVE_ID_2` | `2` | DDS slave_id routed to port 2 |
| `QNC_SLAVE_ID_3` | `3` | DDS slave_id routed to port 3 |
| `QNC_MODBUS_ADDR_1` | `1` | Modbus wire address for port 1 |
| `QNC_MODBUS_ADDR_2` | `1` | Modbus wire address for port 2 |
| `QNC_MODBUS_ADDR_3` | `1` | Modbus wire address for port 3 |
| `QNC_DE_RE_GPIO` | `none` | sysfs GPIO for DE/RE; `none` = auto-direction HAT |
| `QNC_DOMAIN_ID` | `0` | FastDDS domain ID (must match robot computer) |
| `QNC_STATS_INTERVAL` | `10` | Publish stats every N write operations |

### 8.2 udev Device Symlinks

Install once on the Pi CM5:

```bash
sudo cp scripts/99-qnc-grippers.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules && sudo udevadm trigger
```

| Symlink | FTDI serial | Gripper |
|---------|------------|--------|
| `/dev/gripper_ag160` | `A10LSRFJ` | DH AG-160 |
| `/dev/gripper_cgc80` | `B001PRSH` | DH CGC-80 |
| `/dev/gripper_dh56`  | `BG00SMUT` | DH-5-6 |

---

## 9. Validation Notes (Merged from former `tests/README.md`) Content expired; provided for reference only.



### 9.1 Overview

**Source history:** Former in-repo `tests/README.md` and `test_neurasync_idl` validation suite (now removed from this production workspace).  
**ICD reference:** QNC-ICD-001 §5 / §7.1 (IF-005..IF-008)  
**Total assertions (historical):** 59

Validation covered five progressive suites:

| Suite | Assertions | Hardware | Network | Description |
|---|---|---|---|---|
| 1 — Type Instantiation | 15 | No | No | IDL type construction and field access |
| 2 — CDR Serialisation | 18 | No | No | Serialize/deserialize round-trip for all 5 types |
| 3 — DDS Loopback | 5 | No | No | In-process pub/sub on domain 99 |
| 4 — Dashboard Launch | 2 | No¹ | DDS | Verifies `neurasync-dashboard` launches and stays alive briefly |
| 5 — Physical Gripper Cycles | 14 | Yes | DDS + RS485 | Descriptor parse + command publication + observed topic counts and hardware actuation |

¹ Requires display (`$DISPLAY`) and dashboard binary availability.

### 9.2 Two-Machine Test Architecture

Suites 1–3 run entirely on the Robot Computer. Suite 5 runs the Robot Computer as **DDS publisher** while the Pi CM5 acts as **QNC bridge** (subscribes to DDS → RS485 → gripper):

```
┌──────────────────────────────────────┐      DDS Domain 0 (UDP multicast)      ┌──────────────────────────────────────┐
│  Robot Computer (172.28.3.58)        │◄──────────────────────────────────────►│  Pi CM5 (QNC bridge)                 │
│                                      │                                         │                                      │
│  ./build/gripper_validation_test     │  qnc/gripper/command ────────────────► │  ./build/qnc_bridge                  │
│                                      │  qnc/gripper/state   ◄─────────────── │  QNC_SERIAL_PORT=/dev/ttyAMA0        │
│  Suite 5: drives validator traffic   │                                         │  QNC_DE_RE_GPIO=none                 │
│  (DDS + RS485 runtime path)          │                                         │                                      │
│  Counts all DDS messages             │                                         │  RS485 → XY-485 → gripper            │
│  Asserts expected totals             │                                         │  /dev/ttyAMA0 (BCM UART GPIO14/15)   │
└──────────────────────────────────────┘                                         └──────────────────────────────────────┘
```

> **Single-machine note**: Without a dashboard and RS485 hardware, only the in-process suites (1–3) are expected to pass.

### 9.3 Machine Prerequisites

| Item | Robot Computer | Pi CM5 |
|---|---|---|
| FastDDS 3.4.1 | System package (`sudo apt install fastdds-dev`) | System package or built from source |
| nlohmann-json3-dev | `sudo apt install nlohmann-json3-dev` | Required for descriptor parsing |
| `LD_LIBRARY_PATH` | Set to `build/` for `libneurasync_types.so` | Not needed for `qnc_bridge` |
| `neurasync-dashboard` | Run separately in background (`DISPLAY=:0 neurasync-dashboard`) | Not needed |
| RS485 gripper | Not needed — DDS-only mode | AG-160-95 or CGC-80 on serial port; select with `QNC_GRIPPER_MODEL` |
| Serial group | Not needed | `pi` user must be in `dialout` |

### 9.4 Running the Tests

#### Step 1 — Pi CM5: start the QNC bridge

```bash
QNC_SERIAL_PORT=/dev/ttyAMA0 QNC_DE_RE_GPIO=none ./build/qnc_bridge
```

#### Step 2 — Robot Computer: launch the dashboard (background)

```bash
DISPLAY=:0 neurasync-dashboard &
```

#### Step 3 — Robot Computer: run the test

```bash
cd neurasync-robot-client

# Current test bench: DH Robotics CGC-80
QNC_GRIPPER_MODEL=CGC-80 QNC_REMOTE_GRIPPER=1 LD_LIBRARY_PATH=build ./build/test_neurasync_idl

# Alternate model (AG-160-95, default):
# QNC_REMOTE_GRIPPER=1 LD_LIBRARY_PATH=build ./build/test_neurasync_idl
```

> **`QNC_GRIPPER_MODEL=CGC-80` is required when the physical gripper is a CGC-80.** Without it, `gripper_control_dds` defaults to `AG160-95` and publishes speed to register 258 (0x0102). The CGC-80 expects speed at register 260 (0x0104) — the wrong register means speed is never set and the gripper will not actuate.

#### Execution flow

```
main()
 ├── suite1_type_instantiation()   ~1 s    (pure C++, no DDS)
 ├── suite2_serialisation()        ~1 s    (CDR, no DDS middleware)
 ├── suite3_dds_loopback()         ~5 s    (domain 99, in-process)
 └── suite5_gripper_30cycles()     ~35 s   (domain 0, spawns gripper_control_dds; N=QNC_DEMO_CYCLES)
```

#### Expected output

```
  Received: wc=180  rc=60  rsp=60  bs=4
========================================================
 Results: 51 passed, 0 failed
========================================================
```

With Pi CM5 active on LAN, `rsp` will exceed the synthetic baseline (e.g. `rsp=72`) as real RS485 replies arrive.

### 9.5 Suite Details

#### Suite 1 — Type Instantiation (15 assertions)

Verifies the five IDL-generated C++ types are constructible and have correct field accessors. No DDS middleware involved.

| Assertion | What is checked |
|---|---|
| 1.1 | Default construction of all 5 types without exception |
| 1.2a–1.2j | Field assignment + getter round-trip for `WriteCommand` (slave_id, register_address, value, timeout_ms, tag, timestamp) and `ReadCommand` (slave_id, start_register, count, function_code) |
| 1.3 | `WriteMultipleCommand.values` bounded-sequence holds max 123 elements |
| 1.4a–1.4c | `Response.data` accepts 3 register values; `error_code == 0` (EC_SUCCESS) |

#### Suite 2 — CDR Serialisation (18 assertions)

Uses `PubSubType::serialize` / `deserialize` directly — no DDS middleware, no network. A `SerializedPayload_t(4096)` buffer is used for all types.

| Assertion | What is checked |
|---|---|
| 2.1a–2.1e | `WriteCommand` CDR round-trip; slave_id, register_address, value, tag preserved |
| 2.2a–2.2d | `ReadCommand` CDR round-trip; start_register, count, function_code preserved |
| 2.3a–2.3e | `Response` CDR round-trip; data size, data[2]=750, error_code=0, latency_us=3500 preserved |
| 2.4a–2.4d | `BridgeStats` CDR round-trip; tx_count=42, device_connected=true, last_error_code=-2 (EC_TIMEOUT) preserved |

#### Suite 3 — DDS Loopback (5 assertions)

In-process publisher + subscriber on **domain 99** (avoids cross-talk with Suite 5's domain 0). The helper `dds_loopback<T, PubSubType>()` creates a topic, publisher, subscriber, writer, and reader, publishes one sample, then polls for 2 seconds for receipt. All DDS entities are cleaned up before returning.

| Assertion | What is checked |
|---|---|
| 3.1 | `DomainParticipant` created on domain 99 |
| 3.2–3.5 | `WriteCommand`, `ReadCommand`, `Response`, `BridgeStats` each loopback on their canonical topic names |

#### Suite 5 — DDS-Only Gripper N Cycles (13 assertions)

**5.0 — Device Descriptor (6 assertions)**

`DeviceDescriptorParser` loads the device descriptor at runtime. The active model is selected by `QNC_GRIPPER_MODEL`:

| `QNC_GRIPPER_MODEL` | Descriptor file | Current use |
|---|---|---|
| unset or `AG160-95` | `DH-Robotics_AG160-95_RTU.json` | **default — current test bench** |
| `CGC-80` | `DH-Robotics_CGC-80_RTU.json` | alternate model |

Register layout differences:

| Assertion | Register name | AG160-95 | CGC-80 | Modbus address |
|---|---|---|---|---|
| 5.0c | `setPositionGroup` | 259 | 259 | 0x0103 |
| 5.0d | `setSpeedGroup` | **258** | **260** | 0x0102 / 0x0104 |
| 5.0e | `getInitStateGroup` | 512 | 512 | 0x0200 |
| 5.0f | Status register contiguity | 512–514 | 512–514 | 0x0200–0x0202 |

**5.1–5.2 — DDS Infrastructure (2 assertions)**

Creates a `DomainParticipant` on domain 0 (production domain). Registers all 4 types and creates one `DataReader` per topic with a `CountingListener`. `CountingListener<T>` uses `std::atomic<int> count` — thread-safe because DDS middleware calls `on_data_available()` from an internal receive thread.

**5.3–5.7 — Subprocess execution and message counts (5 assertions)**

The test invokes `system("<QNC_PROJECT_ROOT>/build/gripper_control_dds demo N")` where N = `QNC_DEMO_CYCLES` (default 30).

**Timing inside `gripper_control_dds demo N`:**
- `sleep_ms(200)` after init write cmd for DDS peer-discovery
- 100 ms pacing between the 3 write cmds per half-cycle, then 300 ms before the status read
- Per full cycle ≈ 1 s; 30-cycle run ≈ 35 s total

**Message count math (N cycles):**

| Topic | Formula | N=30 example |
|---|---|---|
| `write_cmd` | 1 init + N × (3 open + 3 close) | 181 |
| `read_cmd` | N × (1 open + 1 close) | 60 |
| `response` | N × (1 open + 1 close) | 60 |
| `stats` | every 10 cycles + 1 post-loop | 4 |

| Assertion | Expected (N=30) | Notes |
|---|---|---|
| 5.3 | exit code 0 | `gripper_control_dds demo N` exited cleanly |
| 5.4 | `write_cmd` ≥ N×6 = 180 | tolerates 1 pre-discovery init message loss |
| 5.5 | `read_cmd` == N×2 = 60 | synthetic `ReadCommand` per half-cycle |
| 5.6 | `response` ≥ N×2 = 60 | 60 synthetic + any real RS485 replies from Pi CM5 |
| 5.7 | `stats` ≥ N÷10 = 3 | 4 expected; ≥ N÷10 tolerates delivery variance |

> Assertions scale automatically with `QNC_DEMO_CYCLES`. For example, `QNC_DEMO_CYCLES=100` requires `write_cmd ≥ 600`, `read_cmd == 200`, `response ≥ 200`, `stats ≥ 10`.

### 9.6 Verified Test Results

#### Two-Machine LAN Run (2026-03-05)

**Robot Computer:** Ubuntu 24.04 Noble, IP `172.28.3.58`  
**Pi CM5:** `172.28.3.48` — `qnc_bridge` active  
**FastDDS:** 3.4.1, Domain 0, UDP multicast `239.255.0.1:7400`  
**Gripper model:** AG160-95 (default)

```bash
QNC_REMOTE_GRIPPER=1 LD_LIBRARY_PATH=build ./build/test_neurasync_idl
```

```
  Received: wc=180  rc=60  rsp=72  bs=4
========================================================
 Results: 51 passed, 0 failed
========================================================
```

`rsp=72` (> 60 synthetic baseline) confirms the Pi CM5 bridge is actively forwarding RS485 replies over DDS.

#### Two-Machine LAN — Physical Gripper Movement Confirmed (2026-03-05)

**Gripper:** DH CGC-80, slave_id=1, `/dev/ttyAMA0`, XY-485 HAT

| Metric | Result |
|---|---|
| Physical gripper movement | ✅ 30 open/close cycles, gripper physically actuated |
| DDS `write_cmd` relayed to RS485 | ✅ All 181 frames forwarded fire-and-forget |
| Init calibration wait | ✅ `qnc_bridge` held 3.5 s after reg=0x0100 before forwarding cycle commands |
| RS485 RX (status reads) | ⚠️ XY-485 RO pin not yet wired to GPIO15 — FC04 reads time out |

#### Single-Machine Run (2026-03-05)

All 51 assertions pass in single-machine DDS-only mode — no Pi CM5 or RS485 gripper required. Typical run time: ~43 seconds total (Suites 1–3 ~8 s, Suite 5 ~35 s).

---

## 10. Workspace Directory Layouts

### 10.1 Workspace 1 — `qnc-firmware` (Pi CM5)

> **Responsibility**: Protocol-level bridge — no device semantics, no JSON descriptors

```
qnc-firmware/
├── CMakeLists.txt
├── include/                       ← Protocol-layer headers
│   ├── protocol_types.h           •  shared enums/structs
│   ├── protocol_interface.h       •  abstract base for protocol adapters
│   ├── protocol_manager.h         •  adapter lifecycle management
│   ├── protocol_translator.h      •  GenericCommand → adapter-specific command
│   ├── protocol_detector.h        •  hardware ID-pin / software detection
│   ├── modbus_rtu.h               •  low-level RTU framing (build/CRC/parse)
│   ├── modbus_rtu_adapter.h       •  ModbusRTU ProtocolInterface implementation
│   ├── modbus_device_discovery.h  •  Modbus-side scan / auto-detect
│   ├── iolink_adapter.h           •  IO-Link ProtocolInterface implementation
│   ├── unified_device_discovery.h •  multi-protocol discovery orchestrator
│   ├── unified_protocol_bridge.h  •  top-level bridge (DDS ↔ adapter)
│   └── qnc_system_manager.h       •  init, lifecycle, watchdog
├── src/                           ← Implementations
├── neurasync/                     ← Interface contract (git submodule → qnc-neurasync-idl)
├── iolink/                        ← IO-Link hardware support
├── config/                        ← qnc_default.yaml, qos_profiles.xml
├── scripts/
└── docs/
```

**Excluded:** Device JSON descriptors, device-specific headers (`brainco_revo1.hpp`), application logic (`gripper_control.cpp`).

### 10.2 Workspace 2 — `neurasync-robot-client` (Robot Computer)

> **Responsibility**: Device semantics, application logic, NeuraSync publisher

```
neurasync-robot-client/
├── CMakeLists.txt
├── neurasync/                     ← Same git submodule as qnc-firmware
├── device_descriptors/            ← Device semantic database (JSON)
│   ├── ModbusRTU_DeviceDescriptor_Schema.json
│   ├── DH-Robotics_AG160-95_RTU.json
│   ├── DH-Robotics_CGC-80_RTU.json
│   └── DH-Robotics_DH-5-6_Hand_RTU.json
├── include/
│   └── modbus_rtu_common.hpp
├── src/
│   └── gripper_control_dds.cpp
├── deploy_on_qnc/
│   └── qnc_bridge.cpp
├── scripts/
│   └── 99-qnc-grippers.rules
└── docs/
```

**Excluded:** `protocol_*.h` headers, `modbus_rtu*.h`, `unified_*.h`, `iolink/` — all protocol internals are QNC-side only.

---

## 11. LAN Configuration Checklist

- Both machines on the same subnet (or multicast-routable network)
- FastDDS uses **UDP multicast** on `239.255.0.1:7400` by default
- Robot computer DDS domain ID = 0 (must match QNC)
- If multicast is blocked, configure FastDDS unicast locators in `qos_profiles.xml` pointing each machine at the other's IP
- Validate multicast: `ping 239.255.0.1` from both machines

---

## 12. Prototype Development Roadmap

```
Phase 1 — Stub QNC (robot computer only)
  • Bring up neurasync-robot-client with a loopback DDS echo process
  • Validate IDL types compile and topics flow correctly
  • No Pi hardware needed

Phase 2 — QNC firmware baseline (Pi only)
  • Port existing include/ headers into qnc-firmware
  • Implement main.cpp: subscribe to commands, stub-respond (no RS485 yet)
  • Validate DDS comms across LAN between two machines

Phase 3 — RS485 integration (Pi + gripper)
  • Wire CM5 → XY-485 → gripper (see Section 3: Hardware Setup)
  • Enable modbus_rtu_adapter with TIOCSRS485 kernel mode
  • Run gripper_control on robot computer, observe gripper move

Phase 4 — IO-Link (Pi + IO-Link device)
  • Wire CM5 SPI/UART → MAX22515 → IO-Link device
  • Implement iolink_adapter
  • GenericCommand WRITE/READ dispatches to IO-Link

Phase 5 — Protocol detection & hot-swap
  • Enable protocol_detector (hardware ID pin ADC + software scan)
  • Test device_event topic on device swap
```

---

## 13. Migration Mapping from Monorepo

| Monorepo path | Destination workspace | Notes |
|---------------|-----------------------|-------|
| `include/` | `qnc-firmware/include/` | Unchanged |
| `iolink/` | `qnc-firmware/iolink/` | Unchanged |
| `scripts/99-qnc-grippers.rules` | `qnc-firmware/scripts/` | Retained for deployment |
| `robot_side/src/device_descriptor_parser.hpp` | `neurasync-robot-client/include/` | Move from src/ to include/ |
| `robot_side/src/brainco_revo1.hpp` | `neurasync-robot-client/include/` | Move from src/ to include/ |
| `robot_side/device_descriptors/` | `neurasync-robot-client/device_descriptors/` | Unchanged |
| `robot_side/tests/GripperCommand.idl` | `qnc-neurasync-idl/idl/` | Seeds the shared gripper command contract |
| `robot_side/tests/generated/` | `qnc-neurasync-idl/generated/` | Seeds the shared IDL package |
| `docs/QNC_INTERFACE_CONTROL_DOCUMENT.md` | Both workspaces `docs/` | Read-only copy in each |
| `docs/QNC_Hardware_Setup_*.md` | `qnc-firmware/docs/` | Pi-specific → firmware side |
| `docs/DETECTION_METHODS_ANALYSIS.md` | `qnc-firmware/docs/` | Protocol detection is firmware concern |
| `CMakeLists.txt` | Split into two separate CMakeLists | One per workspace |

---

## 14. ICD Invariants to Preserve

| ICD Section | Invariant |
|-------------|-----------|
| 1.4 | QNC handles protocols, NOT devices — no device-specific code in `qnc-firmware` |
| 5.2 | IF-001, IF-002, IF-003 are mandatory — both workspaces must implement these topics |
| 6.2 | DDS Domain ID = 0 on both machines |
| 9.3 | All timestamps in ISO 8601 (`YYYY-MM-DDTHH:MM:SS.sssZ`) |
| 10.x | Message sequence: command → translate → field device → response; never skip the response |
| 11.x | Round-trip latency budget ≤ 50 ms at 115200 baud for Modbus RTU commands |

---

## 15. Relevant Source Files

| File | Role |
|---|---|
| `include/modbus_rtu_common.hpp` | Shared Modbus RTU serial helpers used by bridge implementation |
| `neurasync/idl/GripperCommand.idl` | DDS command definitions for the bridge ingress |
| `neurasync/idl/GripperState.idl` | DDS state definitions for the bridge egress |
| `src/gripper_control_dds.cpp` | Authoritative DDS command publisher implementation |
| `deploy_on_qnc/qnc_bridge.cpp` | Authoritative DDS↔RS485 bridge implementation |
| `device_descriptors/DH-Robotics_AG160-95_RTU.json` | Register map, baud, slave ID for DH AG-160-95 |
| `device_descriptors/DH-Robotics_CGC-80_RTU.json` | Register map for DH CGC-80 |
| `iolink/docs/MAX22515.txt` | IO-Link transceiver datasheet research (next protocol) |

---

## 16. Notes

- A missing RS485 device is **non-fatal** — the bridge returns `EC_DISCONNECTED` in every `Response` so the robot can detect the fault over DDS without hanging.
- DDS domain ID on both machines **must match** (default: `0`).
- Send `SIGINT` (`Ctrl-C`) to stop — the bridge publishes a final `BridgeStats` before exiting.
- The **XY-485 board** is a passive electrical transceiver — a component *inside* QNC, not QNC itself. The `modbus_rtu` / `qnc_system_manager` code running on the CM5 is the QNC bridge. The application code publishing `WriteCommand` / `ReadCommand` is the robot platform role.
