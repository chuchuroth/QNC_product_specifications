# QNC Interface Control Document (ICD)

**Document Number**: QNC-ICD-001  
**Version**: 1.0  
**Date**: February 18, 2026  
**Status**: Official Release  
**Classification**: Public

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-18 | NeuraSync Engineering | Initial release |

### Document Approvals

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Engineering Lead | | | |
| System Architect | | | |
| Quality Assurance | | | |

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Scope](#2-scope)
3. [Reference Documents](#3-reference-documents)
4. [System Overview](#4-system-overview)
5. [Interface Definitions](#5-interface-definitions)
6. [NeuraSync Communication Interface](#6-neurasync-communication-interface)
7. [Protocol-Specific Interfaces](#7-protocol-specific-interfaces)
8. [Hardware Interfaces](#8-hardware-interfaces)
9. [Data Structures](#9-data-structures)
10. [Message Sequences](#10-message-sequences)
11. [Timing Requirements](#11-timing-requirements)
12. [Error Handling](#12-error-handling)
13. [System Handshaking](#13-system-handshaking)
14. [Automated Device Discovery](#14-automated-device-discovery)
15. [Firmware Update Mechanism](#15-firmware-update-mechanism)
16. [Configuration Interface](#16-configuration-interface)
17. [Integration Guidelines](#17-integration-guidelines)
18. [Appendices](#18-appendices)

---

## 1. Introduction

### 1.1 Purpose

This Interface Control Document (ICD) defines the external interfaces for the QNC (Quick Network Connector) multi-protocol bridge system. QNC provides protocol-level communication bridges between robotic platforms and industrial end-effectors without device-specific semantics.

### 1.2 Audience

This document is intended for:
- Robot control system developers
- System integrators
- Application engineers
- Third-party hardware vendors
- Quality assurance and test engineers

**Related Documents**:
- **QNC-PRD-001**: Product Requirements Document - For product vision, business case, and functional requirements
- **QNC-ARCH-001**: Firmware Architecture - For internal implementation details

### 1.3 Document Organization

This ICD is organized into sections covering different interface types:
- **Section 5-6**: Communication interfaces (NeuraSync)
- **Section 7**: Protocol-specific interfaces
- **Section 8**: Hardware interfaces
- **Section 9-12**: Data formats, sequences, and requirements

### 1.4 Design Philosophy and Value Proposition

#### Core Principle

QNC provides a **protocol-level bridge** that separates robot platform concerns from device communication complexity. The fundamental design principle is:

> **QNC handles protocols, NOT devices.** Device-specific semantics remain in the robot platform as declarative JSON descriptors, not code.

#### Architecture Approach

```
Robot Platform              QNC Bridge                Field Devices
─────────────────────────────────────────────────────────────────
• Device descriptors       • Protocol translation    • Grippers
  (JSON - no code!)         (ModbusRTU, IO-Link)     • Sensors  
• High-level commands      • Hardware abstraction    • Motors
• Application logic        • NeuraSync ↔ RS485/SPI/Eth    • PLCs
• Data interpretation      • No device semantics     • Any device
```

**Key Benefits**:
- **Reduced Integration Time**: New device support in minutes vs. days
- **Minimal Robot Firmware**: Devices handled via JSON descriptors, not code
- **Zero Deployment Risk**: JSON updates instead of firmware rebuilds
- **Protocol Abstraction**: Robot developers focus on robotics, not fieldbus details

**For detailed business case and quantified benefits**, see QNC Product Requirements Document (QNC-PRD-001) Section 2.

---

## 2. Scope

### 2.1 Included Interfaces

This ICD covers the following QNC interfaces:

1. **NeuraSync Communication Interface**
   - Generic protocol bridge (unified interface)
   - ModbusRTU-specific interface
   - IO-Link-specific interface (future)

2. **Hardware Interfaces**
   - RS485 (ModbusRTU)
   - SPI/UART (IO-Link)
   - Ethernet (EtherCAT, EtherNet/IP - future)
   - GPIO (detection pins)

3. **Configuration Interface**
   - JSON device descriptors
   - YAML configuration files
   - Detection configuration

### 2.2 Out of Scope

The following are NOT covered in this ICD:
- Internal QNC firmware implementation details
- Device-specific semantics (handled by robot platform)
- Physical connector specifications (separate document)
- Safety systems and emergency stop protocols

---

## 3. Reference Documents

| Document ID | Title | Version |
|-------------|-------|---------|
| QNC-ARCH-001 | QNC Firmware Architecture | 1.0 |
| QNC-DET-001 | Protocol Detection Methods Analysis | 1.0 |
| QNC-README | QNC Project README | Latest |
| NeuraSync-SPEC | NeuraSync Interface Specification (in-house, FastDDS-based) | 1.4 |
| MODBUS-SPEC | Modbus Application Protocol Specification | V1.1b3 |
| IOLINK-SPEC | IO-Link Interface and System Specification | V1.1.3 |

---

## 4. System Overview

### 4.1 System Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    ROBOT CONTROL SYSTEM                      │
│                                                              │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │   Motion   │  │Application │  │   Safety   │            │
│  │  Planning  │  │   Logic    │  │   System   │            │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘            │
│        └────────────────┴────────────────┘                  │
│                         │                                    │
│                  NeuraSync Middleware                              │
└─────────────────────────┼────────────────────────────────────┘
                          │
                          │ Network (Ethernet/WiFi)
                          │
┌─────────────────────────┼────────────────────────────────────┐
│                         │           QNC BRIDGE               │
│                  NeuraSync Interface                               │
│         ┌───────────────┴────────────────┐                  │
│         │  Unified Protocol Bridge       │                  │
│         │  - Protocol Detection          │                  │
│         │  - Protocol Translation        │                  │
│         │  - Protocol Switching          │                  │
│         └───────────────┬────────────────┘                  │
│                         │                                    │
│         ┌───────────────┴────────────────┐                  │
│         │     Protocol Adapters          │                  │
│         │  [ModbusRTU] [IO-Link] [...]   │                  │
│         └───────────────┬────────────────┘                  │
│                         │                                    │
│         ┌───────────────┴────────────────┐                  │
│         │    Hardware Interfaces         │                  │
│         │  [RS485] [SPI] [Ethernet]      │                  │
│         └───────────────┬────────────────┘                  │
└─────────────────────────┼────────────────────────────────────┘
                          │
                          │ Physical Connection
                          │
┌─────────────────────────┼────────────────────────────────────┐
│                         │                                    │
│              END-EFFECTOR (Gripper/Tool)                     │
│                                                              │
│  Supported Protocols:                                        │
│  • ModbusRTU (RS485)                                        │
│  • IO-Link (SPI/UART)                                       │
│  • EtherCAT (Ethernet)                                      │
│  • EtherNet/IP (Ethernet)                                   │
└──────────────────────────────────────────────────────────────┘
```

### 4.2 Interface Hierarchy

```
Level 1: Application Interface
  └─ NeuraSync Topics (Generic Commands/Responses)
  
Level 2: Protocol Translation Layer
  └─ Protocol-agnostic message routing
  
Level 3: Protocol Management Layer
  └─ Protocol detection, switching, lifecycle
  
Level 4: Protocol Adapters
  └─ ModbusRTU, IO-Link, EtherCAT, etc.
  
Level 5: Hardware Abstraction
  └─ RS485, SPI, Ethernet drivers
```

---

## 5. Interface Definitions

### 5.1 Interface Identification

| Interface ID | Name | Type | Direction | Protocol |
|--------------|------|------|-----------|----------|
| IF-001 | Generic Command | NeuraSync | Robot → QNC | NeuraSync |
| IF-002 | Generic Response | NeuraSync | QNC → Robot | NeuraSync |
| IF-003 | Bridge Status | NeuraSync | QNC → Robot | NeuraSync |
| IF-004 | Device Change Event | NeuraSync | QNC → Robot | NeuraSync |
| IF-005 | ModbusRTU Write Command | NeuraSync | Robot → QNC | NeuraSync |
| IF-006 | ModbusRTU Read Command | NeuraSync | Robot → QNC | NeuraSync |
| IF-007 | ModbusRTU Response | NeuraSync | QNC → Robot | NeuraSync |
| IF-008 | ModbusRTU Statistics | NeuraSync | QNC → Robot | NeuraSync |
| IF-009 | RS485 Serial | Hardware | QNC ↔ Device | ModbusRTU |
| IF-010 | SPI/UART | Hardware | QNC ↔ Device | IO-Link |
| IF-011 | Hardware ID Pin | Hardware | Device → QNC | Analog |
| IF-012 | Configuration File | File | User → QNC | JSON/YAML |

### 5.2 Interface Classification

**Mandatory Interfaces** (must be implemented):
- IF-001: Generic Command
- IF-002: Generic Response
- IF-003: Bridge Status

**Optional Interfaces** (protocol-specific):
- IF-005 to IF-008: ModbusRTU-specific
- IF-009: RS485 hardware (if using ModbusRTU)
- IF-010: SPI/UART (if using IO-Link)

**Configuration Interfaces**:
- IF-011: Hardware ID detection (optional)
- IF-012: Configuration files (recommended)

---

## 6. NeuraSync Communication Interface

### 6.1 Overview

QNC provides both **generic** (protocol-agnostic) and **protocol-specific** NeuraSync interfaces. Robot applications should preferably use the generic interface for maximum flexibility.

### 6.2 NeuraSync Domain and Partition

| Parameter | Value | Notes |
|-----------|-------|-------|
| NeuraSync Domain ID | 0 (default) | Configurable via environment |
| NeuraSync QoS Profile | DEFAULT | Reliable, keep-last |
| History Depth | 10 | For command/response topics |
| Reliability | RELIABLE | All topics |
| Durability | VOLATILE | Status topics may use TRANSIENT_LOCAL |

### 6.3 Generic Interface Topics

#### 6.3.1 Topic: `qnc/bridge/command` (IF-001)

**Purpose**: Send protocol-agnostic commands to any connected device.

**Direction**: Robot → QNC

**Data Type**: `qnc::bridge::GenericCommand`

**Message Structure**:
```cpp
struct GenericCommand {
    CommandType command_type;      // READ, WRITE, EXECUTE, etc.
    uint32 device_id;              // Device/slave address (e.g., 1)
    uint32 address;                // Register/parameter address
    sequence<octet> data;          // Command data payload
    DataType data_type;            // UINT8, UINT16, FLOAT, etc.
    uint32 timeout_ms;             // Command timeout (default: 500ms)
    string tag;                    // Optional tracking identifier
    string timestamp;              // ISO 8601 timestamp
};
```

**Field Descriptions**:

| Field | Type | Range | Description |
|-------|------|-------|-------------|
| command_type | enum | 0-8 | Command operation type |
| device_id | uint32 | 1-247 | Target device address |
| address | uint32 | 0-65535 | Register/parameter address |
| data | byte[] | 0-252 bytes | Command payload |
| data_type | enum | 0-10 | Data type hint for translation |
| timeout_ms | uint32 | 100-5000 | Maximum wait time |
| tag | string | 0-64 chars | User-defined identifier |
| timestamp | string | ISO 8601 | Command creation time |

**Example Usage**:
```cpp
// Write position to gripper
GenericCommand cmd;
cmd.command_type = CommandType::WRITE;
cmd.device_id = 1;
cmd.address = 259;              // Position register
cmd.data = {0x01, 0xF4};        // 500 (50.0mm)
cmd.data_type = DataType::UINT16;
cmd.timeout_ms = 500;
cmd.tag = "set_position_50mm";
cmd.timestamp = "2026-02-18T10:30:00.123Z";

publisher.publish(cmd);
```

**Publishing Rate**: As needed (event-driven)

**QoS Requirements**:
- Reliability: RELIABLE
- History: KEEP_LAST (depth: 10)
- Deadline: 1 second
- Liveliness: 3 seconds

#### 6.3.2 Topic: `qnc/bridge/response` (IF-002)

**Purpose**: Receive responses from device commands.

**Direction**: QNC → Robot

**Data Type**: `qnc::bridge::GenericResponse`

**Message Structure**:
```cpp
struct GenericResponse {
    int32 error_code;              // 0 = success, negative = error
    string error_message;          // Human-readable error description
    sequence<octet> data;          // Response data payload
    DataType data_type;            // Data type of response
    uint32 latency_us;             // Round-trip latency in microseconds
    string tag;                    // Matches command tag
    string timestamp;              // ISO 8601 timestamp
};
```

**Field Descriptions**:

| Field | Type | Range | Description |
|-------|------|-------|-------------|
| error_code | int32 | -99 to 0 | 0=success, negative=error code |
| error_message | string | 0-256 chars | Error description (empty if success) |
| data | byte[] | 0-252 bytes | Response payload |
| data_type | enum | 0-10 | Data type of payload |
| latency_us | uint32 | 0-1000000 | Command execution time |
| tag | string | 0-64 chars | Matches request tag |
| timestamp | string | ISO 8601 | Response creation time |

**Error Codes**:

| Code | Name | Description |
|------|------|-------------|
| 0 | SUCCESS | Command executed successfully |
| -1 | GENERIC_ERROR | Unspecified error |
| -2 | TIMEOUT | Device did not respond in time |
| -3 | DISCONNECTED | Device not connected |
| -4 | INVALID_COMMAND | Command format invalid |
| -5 | INVALID_ADDRESS | Register address invalid |
| -6 | PERMISSION_DENIED | Operation not allowed |
| -7 | DEVICE_BUSY | Device busy with other operation |
| -8 | CRC_ERROR | Communication checksum error |
| -9 | PROTOCOL_ERROR | Protocol-level error |
| -10 | TRANSLATION_ERROR | Failed to translate command |

**Publishing Rate**: Response to each command (event-driven)

**QoS Requirements**: Same as command topic

#### 6.3.3 Topic: `qnc/bridge/status` (IF-003)

**Purpose**: Monitor QNC bridge status and active protocol.

**Direction**: QNC → Robot

**Data Type**: `qnc::bridge::BridgeStatus`

**Message Structure**:
```cpp
struct BridgeStatus {
    boolean initialized;           // Bridge initialized flag
    boolean device_connected;      // Device connection status
    ProtocolType active_protocol;  // Currently active protocol
    DeviceInfo device_info;        // Connected device information
    uint32 uptime_seconds;         // Bridge uptime
    uint64 total_commands;         // Total commands processed
    uint64 failed_commands;        // Failed command count
    uint32 avg_latency_us;         // Average command latency
    string timestamp;              // ISO 8601 timestamp
};
```

**Publishing Rate**: 1 Hz (periodic)

**QoS Requirements**:
- Reliability: RELIABLE
- History: KEEP_LAST (depth: 1)
- Durability: TRANSIENT_LOCAL
- Deadline: 5 seconds
- Liveliness: 10 seconds

#### 6.3.4 Topic: `qnc/bridge/device_event` (IF-004)

**Purpose**: Notify of device connection changes (hot-swap).

**Direction**: QNC → Robot

**Data Type**: `qnc::bridge::DeviceChangeEvent`

**Message Structure**:
```cpp
struct DeviceChangeEvent {
    DeviceInfo old_device;         // Previous device info
    DeviceInfo new_device;         // New device info
    string reason;                 // Change reason
    string timestamp;              // ISO 8601 timestamp
};
```

**Reason Values**:
- `"connected"` - Initial device connection
- `"disconnected"` - Device disconnected
- `"hot_swap"` - Device changed while running
- `"protocol_switch"` - Protocol changed
- `"error"` - Connection error occurred

**Publishing Rate**: Event-driven (on device changes)

**QoS Requirements**:
- Reliability: RELIABLE
- History: KEEP_ALL
- Durability: TRANSIENT_LOCAL

### 6.4 Command Types

```cpp
enum CommandType {
    UNKNOWN = 0,
    READ,              // Read data from device
    WRITE,             // Write data to device
    READ_WRITE,        // Combined read-modify-write
    EXECUTE,           // Execute function/action
    CONFIGURE,         // Configure device parameters
    STATUS_REQUEST,    // Request device status
    RESET,             // Reset device
    EMERGENCY_STOP     // Emergency stop (safety-critical)
};
```

### 6.5 Data Types

```cpp
enum DataType {
    UINT8 = 0,         // Unsigned 8-bit integer
    INT8,              // Signed 8-bit integer
    UINT16,            // Unsigned 16-bit integer
    INT16,             // Signed 16-bit integer
    UINT32,            // Unsigned 32-bit integer
    INT32,             // Signed 32-bit integer
    FLOAT,             // 32-bit floating point
    DOUBLE,            // 64-bit floating point
    BOOL,              // Boolean (true/false)
    STRING,            // UTF-8 string
    BYTE_ARRAY         // Raw byte array
};
```

### 6.6 Protocol Types

```cpp
enum ProtocolType {
    PROTOCOL_UNKNOWN = 0,
    MODBUS_RTU,        // Modbus RTU over RS485
    IOLINK,            // IO-Link (SPI/UART)
    ETHERCAT,          // EtherCAT (Ethernet)
    ETHERNET_IP,       // EtherNet/IP (Ethernet)
    PROFINET,          // PROFINET (Ethernet)
    CANOPEN,           // CANopen (CAN bus)
    CUSTOM             // Custom protocol
};
```

---

## 7. Protocol-Specific Interfaces

### 7.1 ModbusRTU Interface

For applications requiring direct ModbusRTU control, QNC provides protocol-specific topics.

#### 7.1.1 Topic: `qnc/modbus/write_cmd` (IF-005)

**Purpose**: Write ModbusRTU holding registers.

**Data Type**: `qnc::modbus::WriteCommand`

**Message Structure**:
```cpp
struct WriteCommand {
    uint8 slave_id;                // Modbus slave address (1-247)
    uint16 register_address;       // Register address (0-65535)
    uint16 value;                  // Register value (0-65535)
    string timestamp;              // ISO 8601 timestamp
};
```

**Example**:
```cpp
WriteCommand cmd;
cmd.slave_id = 1;
cmd.register_address = 259;  // Position register
cmd.value = 500;             // 50.0mm
cmd.timestamp = "2026-02-18T10:30:00.123Z";
```

#### 7.1.2 Topic: `qnc/modbus/read_cmd` (IF-006)

**Purpose**: Read ModbusRTU holding registers.

**Data Type**: `qnc::modbus::ReadCommand`

**Message Structure**:
```cpp
struct ReadCommand {
    uint8 slave_id;                // Modbus slave address (1-247)
    uint16 start_register;         // Starting register (0-65535)
    uint16 count;                  // Number of registers (1-125)
    string timestamp;              // ISO 8601 timestamp
};
```

#### 7.1.3 Topic: `qnc/modbus/response` (IF-007)

**Purpose**: ModbusRTU command responses.

**Data Type**: `qnc::modbus::Response`

**Message Structure**:
```cpp
struct Response {
    uint8 slave_id;                // Modbus slave address
    uint16 start_register;         // Starting register
    sequence<uint16> data;         // Register values
    int32 error_code;              // Error code (0 = success)
    string error_message;          // Error description
    string timestamp;              // ISO 8601 timestamp
};
```

#### 7.1.4 Topic: `qnc/modbus/stats` (IF-008)

**Purpose**: ModbusRTU bridge statistics.

**Data Type**: `qnc::modbus::BridgeStats`

**Message Structure**:
```cpp
struct BridgeStats {
    string device_id;              // Device identifier
    uint64 tx_count;               // Transmitted frames
    uint64 rx_count;               // Received frames
    uint64 error_count;            // Error count
    uint32 last_error_code;        // Last error code
    string last_error_message;     // Last error message
    double uptime_seconds;         // Bridge uptime
    string timestamp;              // ISO 8601 timestamp
};
```

**Publishing Rate**: 5 Hz (periodic)

---

## 8. Hardware Interfaces

### 8.1 RS485 Interface (IF-009)

**Purpose**: ModbusRTU communication with industrial devices.

**Physical Layer**:
- Standard: TIA/EIA-485
- Topology: Multi-drop bus (up to 32 devices)
- Cable: Twisted pair, shielded
- Maximum distance: 1200 meters
- Termination: 120Ω at both ends

**Electrical Characteristics**:

| Parameter | Min | Typical | Max | Unit |
|-----------|-----|---------|-----|------|
| Differential voltage (output) | 1.5 | 3.0 | 5.0 | V |
| Common-mode voltage | -7 | 0 | +12 | V |
| Input impedance | - | 12 | - | kΩ |

**Pin Assignment** (standard connector):

| Pin | Signal | Direction | Description |
|-----|--------|-----------|-------------|
| 1 | RS485 A (D+) | I/O | Data positive |
| 2 | RS485 B (D-) | I/O | Data negative |
| 3 | GND | - | Signal ground |
| 4 | VCC | O | Power output (optional) |

**Communication Parameters**:

| Parameter | Value | Notes |
|-----------|-------|-------|
| Baudrate | 9600, 19200, 38400, 57600, 115200 | Auto-detected or configured |
| Data bits | 8 | Fixed |
| Stop bits | 1 | Standard |
| Parity | None, Even, Odd | Device-dependent |
| Flow control | None | Not used |

### 8.2 SPI/UART Interface (IF-010)

**Purpose**: IO-Link communication via MAX22515 transceiver.

**UART Configuration**:

| Parameter | COM1 | COM2 | COM3 |
|-----------|------|------|------|
| Baudrate | 4800 | 38400 | 230400 |
| Data bits | 8 | 8 | 8 |
| Parity | Even | Even | Even |
| Stop bits | 1 | 1 | 1 |

**GPIO Signals**:

| Signal | GPIO Pin | Direction | Description |
|--------|----------|-----------|-------------|
| TXEN | 17 | Output | Transmit enable (C/Q driver) |
| IRQ | 27 | Input | Wake-up/interrupt signal |
| SDA_WU | 2 | Input | Wake-up detection |

**Pin Assignment** (M12 connector):

| Pin | Signal | Direction | Description |
|-----|--------|-----------|-------------|
| 1 | L+ | I | Power supply positive (24V) |
| 2 | Not connected | - | - |
| 3 | L- | I | Power supply negative (0V) |
| 4 | C/Q | I/O | IO-Link data line |

### 8.3 Hardware ID Pin (IF-011)

**Purpose**: Resistor-based device identification.

**Configuration**:
- Pull-up resistor: 10kΩ (on QNC board)
- ID resistor: Device-specific (on gripper)
- ADC resolution: 12-bit (0-4095)
- Reference voltage: 3.3V

**Device ID Mapping**:

| Device | R_ID | ADC Value | Tolerance | Protocol |
|--------|------|-----------|-----------|----------|
| DH-Robotics AG95 | 2.2kΩ | 1862 | ±50 | ModbusRTU |
| Robotiq 2F-85 | 3.3kΩ | 2275 | ±50 | ModbusRTU |
| Zimmer GEP5000IL | 4.7kΩ | 2559 | ±50 | IO-Link |
| OnRobot RG2 | 6.8kΩ | 2856 | ±50 | ModbusRTU |
| Schunk EGP64 | 10kΩ | 3072 | ±50 | IO-Link |
| Unknown/Custom | Open | 4095 | - | Auto-detect |

---

## 9. Data Structures

### 9.1 DeviceInfo Structure

```cpp
struct DeviceInfo {
    string manufacturer;           // e.g., "DH-Robotics"
    string model;                  // e.g., "AG-95"
    string serial_number;          // Device serial number
    string firmware_version;       // Device firmware version
    ProtocolType protocol;         // Communication protocol
    string descriptor_file;        // Path to JSON descriptor
    uint32 vendor_id;              // Vendor identification
    uint32 device_id;              // Device identification
};
```

### 9.2 DeviceInfo Field Constraints

| Field | Type | Max Length | Required | Example |
|-------|------|------------|----------|---------|
| manufacturer | string | 64 chars | Yes | "DH-Robotics" |
| model | string | 64 chars | Yes | "AG-95" |
| serial_number | string | 32 chars | No | "AG95-2024-001234" |
| firmware_version | string | 16 chars | No | "v2.3.1" |
| protocol | enum | - | Yes | MODBUS_RTU |
| descriptor_file | string | 256 chars | No | "/etc/qnc/devices/dh_ag95.json" |
| vendor_id | uint32 | - | No | 0x1234 |
| device_id | uint32 | - | No | 0x5678 |

### 9.3 Timestamp Format

All timestamps must follow **ISO 8601** format:

```
Format: YYYY-MM-DDTHH:MM:SS.sssZ
Example: 2026-02-18T10:30:45.123Z

Where:
  YYYY = 4-digit year
  MM   = 2-digit month (01-12)
  DD   = 2-digit day (01-31)
  HH   = 2-digit hour (00-23)
  MM   = 2-digit minute (00-59)
  SS   = 2-digit second (00-59)
  sss  = 3-digit millisecond (000-999)
  Z    = UTC timezone indicator
```

---

## 10. Message Sequences

### 10.1 Normal Operation Sequence

```
Robot                  QNC Bridge              Device
  |                         |                     |
  |--- GenericCommand ----->|                     |
  |    (WRITE, addr=259)    |                     |
  |                         |                     |
  |                         |--- ModbusRTU ------>|
  |                         |    Write Register   |
  |                         |                     |
  |                         |<--- Response -------|
  |                         |    (success)        |
  |                         |                     |
  |<-- GenericResponse -----|                     |
  |    (error_code=0)       |                     |
  |                         |                     |
```

**Timing**:
1. Robot publishes command: T0
2. QNC receives via NeuraSync: T0 + 1ms
3. QNC translates and sends to device: T0 + 2ms
4. Device processes and responds: T0 + 5ms
5. QNC receives device response: T0 + 6ms
6. QNC publishes NeuraSync response: T0 + 7ms
7. Robot receives response: T0 + 8ms

**Total latency**: 8ms typical

### 10.2 Device Detection Sequence

```
QNC Bridge             Device
  |                       |
  |--- Power On --------->|
  |                       |
  |--- Probe ModbusRTU -->|
  |    (Read register 0)  |
  |                       |
  |<--- Response ---------|
  |    (success/fail)     |
  |                       |
  |--- If success ------->|
  |    Read more regs     |
  |    to verify          |
  |                       |
  |<--- Responses --------|
  |                       |
  |--- DETECTED ----------|
  |    (ModbusRTU)        |
  |                       |
```

**Detection Time**:
- Hardware ID: < 1ms
- Descriptor load: < 10ms
- Protocol probe: 200-1000ms (depends on attempts)

### 10.3 Hot-Swap Sequence

```
Robot             QNC Bridge          Old Device      New Device
  |                   |                    |              |
  |                   |--- Health Check -->|              |
  |                   |    (periodic)      |              |
  |                   |<--- OK ------------|              |
  |                   |                    |              |
  |                   |                    X Disconnect   |
  |                   |                    |              |
  |                   |--- Health Check -->|              |
  |                   |    (TIMEOUT)       |              |
  |                   |                    |              |
  |<-- DeviceChange --|                    |              |
  |    (disconnected) |                    |              |
  |                   |                    |              |
  |                   |                    |    Connect   |
  |                   |                    |       |<-----|
  |                   |                    |              |
  |                   |--- Detect ----------------------->|
  |                   |    (probe)                        |
  |                   |<--- Response ---------------------|
  |                   |    (protocol detected)            |
  |                   |                                   |
  |                   |--- Connect --------------------- >|
  |                   |<--- Ready ------------------------|
  |                   |                                   |
  |<-- DeviceChange --|                                   |
  |    (hot_swap)     |                                   |
  |                   |                                   |
```

**Recovery Time**: 1-3 seconds

### 10.4 Error Recovery Sequence

```
Robot                  QNC Bridge              Device
  |                         |                     |
  |--- GenericCommand ----->|                     |
  |                         |                     |
  |                         |--- Request -------->|
  |                         |                     |
  |                         |    (TIMEOUT)        X
  |                         |                     |
  |<-- GenericResponse -----|                     |
  |    (error_code=-2)      |                     |
  |                         |                     |
  |--- Retry Command ------>|                     |
  |                         |                     |
  |                         |--- Reconnect ------>|
  |                         |<--- Success --------|
  |                         |                     |
  |                         |--- Retry Request -->|
  |                         |<--- Response -------|
  |                         |                     |
  |<-- GenericResponse -----|                     |
  |    (error_code=0)       |                     |
  |                         |                     |
```

---

## 11. Timing Requirements

### 11.1 Command Execution Timing

| Operation | Typical | Maximum | Notes |
|-----------|---------|---------|-------|
| NeuraSync message propagation | 0.5 ms | 2 ms | Local network |
| Command translation | 0.1 ms | 1 ms | Lookup and transform |
| ModbusRTU transaction | 2 ms | 50 ms | @ 115200 baud |
| IO-Link transaction | 5 ms | 100 ms | @ COM2 mode |
| Total end-to-end | 8 ms | 200 ms | Robot to device |

### 11.2 Detection Timing

| Method | Typical | Maximum | Conditions |
|--------|---------|---------|------------|
| Hardware ID read | 0.5 ms | 1 ms | ADC sampling |
| Descriptor load | 5 ms | 50 ms | File I/O |
| ModbusRTU probe | 100 ms | 500 ms | Per attempt |
| IO-Link probe | 200 ms | 1000 ms | Wake-up sequence |
| Full auto-detect | 500 ms | 3000 ms | All protocols |

### 11.3 Periodic Operations

| Operation | Rate | Period | Jitter |
|-----------|------|--------|--------|
| Bridge status publish | 1 Hz | 1000 ms | ±50 ms |
| Health check | 0.2 Hz | 5000 ms | ±100 ms |
| Statistics publish | 5 Hz | 200 ms | ±20 ms |
| Hot-swap monitoring | 1 Hz | 1000 ms | ±50 ms |

### 11.4 Timeout Values

| Timeout Type | Default | Range | Description |
|--------------|---------|-------|-------------|
| Command timeout | 500 ms | 100-5000 ms | Per-command configurable |
| Connection timeout | 2000 ms | 1000-10000 ms | Initial connection |
| Health check timeout | 1000 ms | 500-5000 ms | Device health probe |
| Detection timeout | 3000 ms | 1000-10000 ms | Auto-detection |

---

## 12. Error Handling

### 12.1 Error Code Categories

**Communication Errors** (-1 to -10):
- -1: GENERIC_ERROR
- -2: TIMEOUT
- -3: DISCONNECTED
- -8: CRC_ERROR
- -9: PROTOCOL_ERROR

**Command Errors** (-11 to -20):
- -4: INVALID_COMMAND
- -5: INVALID_ADDRESS
- -6: PERMISSION_DENIED
- -7: DEVICE_BUSY
- -10: TRANSLATION_ERROR

**System Errors** (-21 to -30):
- -21: NOT_INITIALIZED
- -22: RESOURCE_UNAVAILABLE
- -23: INTERNAL_ERROR

### 12.2 Error Recovery Strategies

| Error Code | Recovery Strategy | Retry? | User Action Required? |
|------------|-------------------|--------|------------------------|
| -1 | Log and report | No | Investigate |
| -2 | Retry with exponential backoff | Yes (3x) | Check connection |
| -3 | Attempt reconnection | Yes (auto) | Check physical connection |
| -4 | Validate and resend | No | Fix command format |
| -5 | Check device descriptor | No | Verify address |
| -6 | Check permissions | No | Reconfigure |
| -7 | Wait and retry | Yes (after delay) | None |
| -8 | Retry with error correction | Yes (2x) | Check cabling |
| -9 | Verify protocol match | No | Reconfigure protocol |
| -10 | Check mapping rules | No | Update translation rules |

### 12.3 Fault Conditions

| Fault | Detection | Response | Recovery |
|-------|-----------|----------|----------|
| Device disconnect | Health check timeout | Publish DeviceChangeEvent | Auto-reconnect when detected |
| Protocol mismatch | Probe failure | Report error | Re-run detection |
| Communication timeout | No response | Return error code -2 | Retry with timeout |
| Invalid data | CRC/checksum fail | Return error code -8 | Request retransmit |
| Buffer overflow | Data > max size | Return error code -4 | Split into multiple commands |

### 12.4 Error Logging

All errors SHALL be logged with the following information:
- Timestamp (ISO 8601)
- Error code
- Error message
- Context (command that failed)
- Device information
- Attempted recovery action

**Log Format**:
```
[2026-02-18T10:30:45.123Z] ERROR: Code=-2, Message="Device timeout", Device=ModbusRTU:1, Command=READ:514, Recovery=RETRY
```

### 12.5 Device Disconnection During Operation

**Critical Error Scenario**: Device unplugged while robot is moving

When a device disconnection is detected during robot motion, QNC SHALL:

1. **Immediate Detection** (< 100ms):
   - Health check timeout on active command
   - No response from device within command timeout
   - Physical disconnection detected (if hardware monitoring available)

2. **Emergency Response**:
   - Publish `GenericResponse` with error code `-103` (DEVICE_DISCONNECT_DURING_MOTION)
   - Set `error_message` to include motion state information
   - Recommend robot controller to trigger E-stop
   - Enter safe state (stop accepting new commands)

3. **Robot Controller Actions** (RECOMMENDED):
   - Trigger emergency stop (E-stop)
   - Halt all motion immediately
   - Log event with timestamp and context
   - Alert operator
   - Do not attempt to use end-effector until reconnected and verified

**Error Response Example**:
```cpp
GenericResponse error_response;
error_response.error_code = -103;  // DEVICE_DISCONNECT_DURING_MOTION
error_response.error_message = "CRITICAL: Device disconnected during motion - E-stop recommended";
error_response.data = {};  // Empty
error_response.tag = original_command.tag;
error_response.timestamp = current_iso_timestamp();
```

**Recovery Sequence**:
1. Robot confirms E-stop triggered
2. Device physically reconnected
3. QNC detects reconnection
4. QNC re-initializes device communication
5. QNC verifies device ready state
6. Robot performs homing/calibration sequence
7. Normal operations resume

**Timeout Configuration**:
- **Connection health check**: 1000ms (configurable 500-5000ms)
- **Command timeout**: 500ms default (per-command configurable)
- **Disconnection detection**: 3 consecutive failed health checks
- **Reconnection attempt interval**: 2000ms

---

## 13. System Handshaking

### 13.1 Purpose

System handshaking verifies that QNC is properly mounted, powered, and ready to communicate with the robot controller before normal operations begin. This ensures a reliable startup sequence and prevents operation in degraded states.

### 13.2 Handshake Sequence

**Startup Handshake Flow**:

```
QNC Startup             Robot Controller
     |                         |
     |-- Power On ------------>|
     |   (24V + 5V rails)      |
     |                         |
     |-- Self-Test ------------|
     |   (1-2 seconds)         |
     |                         |
     |-- NeuraSync Discovery ------->|
     |   (network init)        |
     |<-- NeuraSync Ack -------------|
     |                         |
     |-- HandshakeRequest ---->|
     |   [Topic: qnc/system/handshake]
     |                         |
     |<-- HandshakeAck --------|
     |   (robot ready)         |
     |                         |
     |-- StatusReady ---------->|
     |   [Bridge status]       |
     |                         |
     |<-- BeginOperations -----|
     |                         |
    READY                    READY
```

### 13.3 Handshake Message Structure

**Topic**: `qnc/system/handshake_request` (IF-013)

**Direction**: QNC → Robot

**Data Type**: `qnc::system::HandshakeRequest`

```cpp
struct HandshakeRequest {
    string qnc_serial_number;      // Unique QNC identifier
    string firmware_version;       // e.g., "1.0.0"
    uint32 voltage_24v_mv;         // 24V rail voltage (millivolts)
    uint32 voltage_5v_mv;          // 5V rail voltage (millivolts)
    float temperature_celsius;     // CPU temperature
    boolean mount_verified;        // Mechanical mount sensor status
    boolean self_test_passed;      // Self-test result
    uint64 uptime_seconds;         // Time since boot
    string timestamp;              // ISO 8601 timestamp
};
```

**Topic**: `qnc/system/handshake_ack` (IF-014)

**Direction**: Robot → QNC

**Data Type**: `qnc::system::HandshakeAck`

```cpp
struct HandshakeAck {
    boolean acknowledge;           // true = accept, false = reject
    string robot_id;              // Robot controller identifier
    string robot_version;         // Robot firmware version
    boolean ready_for_operations; // Robot ready to send commands
    string reason;                // Reason if rejected
    string timestamp;             // ISO 8601 timestamp
};
```

### 13.4 Power Supply Verification

QNC SHALL verify power supply during handshake:

| Rail | Nominal | Minimum | Maximum | Action if Out of Range |
|------|---------|---------|---------|------------------------|
| 24V | 24.0V | 22.0V | 26.0V | Fail handshake, report error |
| 5V | 5.0V | 4.75V | 5.25V | Fail handshake, report error |

**Voltage Reading Accuracy**: ±2%

**Measurement Method**: ADC sampling from voltage divider circuit

### 13.5 Mechanical Mount Verification

QNC SHALL verify mechanical mounting through:

1. **Connector engagement sensor** (if available)
   - Digital input indicating full connector insertion
   - Expected state: HIGH when properly mounted

2. **Vibration/acceleration monitoring**
   - Detect excessive movement indicating loose mount
   - Threshold: < 0.5g RMS during idle

3. **Continuity check**
   - Verify ground connection integrity
   - Resistance < 1Ω expected

### 13.6 Heartbeat Protocol

After successful handshake, continuous heartbeat monitoring:

**QNC Heartbeat** (IF-015):
- **Topic**: `qnc/system/heartbeat`
- **Rate**: 1 Hz (configurable 0.5-10 Hz)
- **Timeout**: 3 seconds (considered disconnected if no heartbeat)

**Message Structure**:
```cpp
struct QNCHeartbeat {
    uint64 sequence_number;        // Incrementing counter
    boolean operational;           // true = normal, false = degraded
    string status_message;         // Status description
    string timestamp;              // ISO 8601 timestamp
};
```

**Robot Controller SHALL**:
- Monitor heartbeat messages
- Trigger warning if heartbeat missed for 3 consecutive cycles
- Trigger safe state if heartbeat missed for 5 consecutive cycles

### 13.7 Handshake Timeout Values

| Phase | Timeout | Description |
|-------|---------|-------------|
| Power stabilization | 2000ms | Wait for power rails to stabilize |
| Self-test | 5000ms | Internal self-test completion |
| NeuraSync discovery | 10000ms | Network participant discovery |
| Robot acknowledgment | 5000ms | Wait for robot ACK response |
| Total handshake | 30000ms | Maximum time for complete handshake |

### 13.8 Failed Handshake Recovery

**If handshake fails**:
1. QNC enters safe state
2. Error logged with failure reason
3. LED indicator shows error pattern (if available)
4. Retry handshake after 5 seconds
5. After 3 failed attempts, manual intervention required

**Failure Reasons**:
- Power supply out of range
- Mount not verified
- NeuraSync communication failure
- Robot controller not responding
- Self-test failure

---

## 14. Automated Protocol Discovery

### 14.1 Overview

QNC provides automated protocol discovery that identifies communication protocols and connection parameters without requiring manual configuration. The discovery system performs hardware handshaking and protocol detection at the **protocol level only**, without any device-specific semantics. All device-specific logic remains in the robot platform using JSON descriptors (see Section 16).

**Design Principle**:
> QNC discovers **protocols**, NOT **devices**. Device identification, initialization sequences, and operational semantics are handled by the robot platform using declarative JSON descriptors.

**Key Capabilities**:
1. **Hardware Handshaking** - Verifies power supply (24V/5V) and mechanical mounting
2. **Protocol Detection** - Automatically identifies Modbus RTU, IO-Link, or other field bus protocols
3. **Connection Parameter Discovery** - Reports slave ID, baudrate, port path

**What Discovery Does NOT Do** (By Design):
- ❌ Device identification (manufacturer/model)
- ❌ Device-specific initialization sequences
- ❌ Register map inference
- ❌ Vendor-specific configuration

**Benefits**:
- **Protocol-Level Abstraction**: QNC requires zero device-specific code
- **Descriptor Integration**: Works seamlessly with Section 16 device descriptors
- **Reduced Integration Time**: Protocol detection in seconds, not minutes
- **Zero Firmware Updates**: Adding new devices never requires QNC firmware changes

### 14.2 Discovery Architecture

**Two-Stage Process**:

```
┌─────────────────────────────────────────┐
│ Stage 1: QNC Protocol Discovery        │
│ (No device-specific knowledge)          │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ 1. Hardware Handshaking                 │
│   • Check 24V power supply              │
│   • Check 5V power supply               │
│   • Verify mechanical mounting          │
│   • Read system temperature             │
└──────────────┬──────────────────────────┘
               ▼
┌─────────────────────────────────────────┐
│ 2. Protocol Detection                   │
│   ┌─────────────────────────────────┐   │
│   │ Try Modbus RTU                  │   │
│   │  • Scan slave IDs 1-247         │   │
│   │  • Test baudrates               │   │
│   │  • Detect response patterns     │   │
│   └─────────────────────────────────┘   │
│   ┌─────────────────────────────────┐   │
│   │ Try IO-Link                     │   │
│   │  • Initialize GPIO pins         │   │
│   │  • Test COM1/COM2/COM3          │   │
│   │  • Detect protocol handshake    │   │
│   └─────────────────────────────────┘   │
└──────────────┬──────────────────────────┘
               ▼
       ✓ PROTOCOL DETECTED
       
       Returns:
       • Protocol type (MODBUS_RTU, IOLINK, etc.)
       • Connection parameters (port, baudrate, slave_id)
       • Power/mount status

┌─────────────────────────────────────────┐
│ Stage 2: Robot Platform (Not QNC)      │
│ (Device semantics via descriptors)      │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ 3. Device Identification                │
│   • Robot queries device via QNC bridge │
│   • Reads vendor ID, product code, etc. │
│   • Matches to descriptor database      │
│   • Selects appropriate JSON descriptor │
└──────────────┬──────────────────────────┘
               ▼
┌─────────────────────────────────────────┐
│ 4. Initialization via Descriptor        │
│   • Robot provides descriptor to QNC    │
│   • QNC reads init sequence from JSON   │
│   • QNC executes generic protocol ops   │
│   • No vendor-specific logic in QNC     │
└──────────────┬──────────────────────────┘
               ▼
         ✓ DEVICE READY
```

**Key Architectural Points**:

1. **QNC Responsibility**: Protocol detection and communication bridge
2. **Robot Platform Responsibility**: Device identification and semantic interpretation
3. **Descriptor Role**: Contains all device-specific knowledge as declarative JSON
4. **No Hardcoded Devices**: QNC firmware contains zero device-specific code

### 14.3 Discovery API

#### 14.3.1 C++ API

**Include Files**:
```cpp
#include "qnc_system_manager.h"
#include "protocol_discovery.h"  // Renamed from unified_device_discovery.h
```

**Basic Usage - Protocol Discovery Only**:
```cpp
using namespace qnc;

// Create system manager
system::SystemManager sys_mgr;

// Configure protocol discovery
discovery::ProtocolDiscoveryConfig config;
config.verbose = true;
config.perform_handshake = true;
config.try_modbus = true;
config.try_iolink = true;

// Run protocol discovery (NO device-specific logic)
discovery::ProtocolDiscoveryResult result = 
    sys_mgr.discover_protocol("/dev/ttyAMA0", config);

if (result.success) {
    // Check handshake results
    std::cout << "Power 24V: " 
              << result.handshake.voltage_24v_mv / 1000.0f << "V\n";
    std::cout << "Power 5V: " 
              << result.handshake.voltage_5v_mv / 1000.0f << "V\n";
    
    // Check protocol detection results
    for (const auto& conn : result.detected_connections) {
        std::cout << "Protocol: " 
                  << protocol_type_to_string(conn.protocol) << "\n";
        std::cout << "Port: " << conn.port << "\n";
        std::cout << "Baudrate: " << conn.baudrate << "\n";
        
        if (conn.protocol == ProtocolType::MODBUS_RTU) {
            std::cout << "Slave ID: " << (int)conn.slave_id << "\n";
        }
        
        // Now robot platform can:
        // 1. Query device via QNC bridge to identify it
        // 2. Match to descriptor database
        // 3. Load appropriate descriptor
        // 4. Initialize via descriptor (see example below)
    }
} else {
    std::cerr << "Protocol discovery failed: " 
              << result.error_message << "\n";
}
```

**Complete Integration with Descriptor System**:
```cpp
using namespace qnc;

// Step 1: Discover protocol
auto discovery_result = sys_mgr.discover_protocol("/dev/ttyAMA0", config);

if (discovery_result.success && !discovery_result.detected_connections.empty()) {
    auto& conn = discovery_result.detected_connections[0];
    
    // Step 2: Robot platform identifies device (using QNC as bridge)
    std::string descriptor_path;
    if (conn.protocol == ProtocolType::MODBUS_RTU) {
        // Read vendor-specific ID registers via QNC modbus bridge
        uint16_t product_code = sys_mgr.read_modbus_register(
            conn.port, conn.slave_id, 1000);  // Example register
        
        // Match to descriptor database (robot platform logic)
        descriptor_path = match_device_descriptor(product_code);
        // e.g., returns "/etc/qnc/devices/dh_ag95.json"
    }
    
    // Step 3: Initialize device using descriptor
    if (!descriptor_path.empty()) {
        auto init_result = sys_mgr.initialize_from_descriptor(
            conn, descriptor_path);
        
        if (init_result.success) {
            std::cout << "Device initialized successfully\n";
        }
    }
}
```

#### 14.3.2 Discovery Configuration

**Structure**: `ProtocolDiscoveryConfig`

```cpp
struct ProtocolDiscoveryConfig {
    // Hardware handshaking
    bool perform_handshake;          // Enable power/mount verification
    
    // Protocol detection
    bool try_modbus;                 // Attempt Modbus RTU detection
    bool try_iolink;                 // Attempt IO-Link detection
    bool try_all_protocols;          // Test all protocols (vs. first match)
    
    // Modbus-specific settings
    modbus::ScanConfig modbus_config;
    
    // Output control
    bool verbose;                    // Detailed progress logging
};
```

**Modbus Scan Configuration** (Protocol-Level Only):
```cpp
struct ScanConfig {
    uint8_t min_slave_id;            // Minimum slave ID to scan (1-247)
    uint8_t max_slave_id;            // Maximum slave ID to scan (1-247)
    uint32_t response_timeout_ms;    // Response timeout (100-5000ms)
    bool scan_all_baudrates;         // Test multiple baudrates
    bool verbose;                    // Detailed logging
};
```

**Note**: No `auto_initialize` flag - initialization is controlled by robot platform using descriptors (Section 16).

#### 14.3.3 Discovery Result

**Structure**: `ProtocolDiscoveryResult`

```cpp
struct ProtocolDiscoveryResult {
    bool success;                      // Overall success flag
    string error_message;              // Error description (if failed)
    
    HandshakeResult handshake;         // Hardware handshake results
    vector<ProtocolConnection> detected_connections;  // Detected protocols
    
    uint32_t total_duration_ms;        // Total discovery time
};
```

**Handshake Result** (Hardware Infrastructure Status):
```cpp
struct HandshakeResult {
    HandshakeStatus status;          // COMPLETED, FAILED, NOT_STARTED
    bool power_ok;                   // Power supply valid
    bool mount_ok;                   // Mechanical mount verified
    uint32_t voltage_24v_mv;         // 24V rail in millivolts
    uint32_t voltage_5v_mv;          // 5V rail in millivolts
    float temperature_c;             // System temperature (°C)
    string error_message;            // Error if failed
};
```

**Protocol Connection Info** (Protocol-Level Only):
```cpp
struct ProtocolConnection {
    // Protocol information (NO device semantics)
    ProtocolType protocol;           // MODBUS_RTU, IOLINK, ETHERNET_IP, etc.
    string port;                     // Serial port or network interface
    uint32_t baudrate;               // Communication baudrate (serial only)
    
    // Protocol-specific parameters
    uint8_t slave_id;                // Modbus slave ID (if Modbus)
    uint8_t com_mode;                // IO-Link COM mode (if IO-Link)
    
    // Connection quality
    uint32_t response_time_ms;       // Average response time
    bool communication_verified;      // Protocol handshake successful
    
    // NO device identification fields:
    // - No manufacturer field
    // - No model field
    // - No register_map
    // - No is_ready/needs_initialization
    // All device semantics handled by robot platform via descriptors
};
```

**Key Design Point**: Discovery returns **protocol connection parameters** only. Device identification and initialization are handled separately using descriptors (Section 16).

### 14.4 Command-Line Tools

#### 14.4.1 Protocol Discovery Tool

**Purpose**: Automated protocol detection without device-specific logic.

**Location**: `examples/protocol_discovery_example.cpp`

**Binary**: `build/protocol_discovery`

**Usage**:
```bash
./protocol_discovery <serial_port> [options]
```

**Options**:

| Option | Description | Default |
|--------|-------------|---------|
| `--quick` | Quick test: scan only slave ID 1 | Off |
| `--no-handshake` | Skip power/mounting verification | Off |
| `--modbus-only` | Only try Modbus RTU protocol | Off |
| `--iolink-only` | Only try IO-Link protocol | Off |
| `--all-protocols` | Try all protocols | Off |
| `--min-id <id>` | Minimum Modbus slave ID | 1 |
| `--max-id <id>` | Maximum Modbus slave ID | 10 |
| `--fast` | Fast scan (200ms timeout) | Off |
| `--verbose` | Detailed progress output | Off |
| `--quiet` | Minimal output | Off |

**Examples**:
```bash
# Protocol discovery (recommended)
./protocol_discovery /dev/ttyAMA0

# Quick test with verbose output
./protocol_discovery /dev/ttyAMA0 --quick --verbose

# Modbus only, limited range
./protocol_discovery /dev/ttyAMA0 --modbus-only --min-id 1 --max-id 5

# Fast scan
./protocol_discovery /dev/ttyAMA0 --fast
```

**Example Output**:
```
╔══════════════════════════════════════════════════════╗
║  QNC Protocol Discovery                              ║
╚══════════════════════════════════════════════════════╝

Port:         /dev/ttyAMA0
Mode:         Quick Test (ID 1 only)
Handshake:    Yes (QNC-to-device)

Discovering protocols...

┌─ Hardware Handshake Results ────────────────────────┐
│ Status:       ✓ PASSED                              │
│ Power 24V:    24.00V                                │
│ Power 5V:     5.00V                                 │
│ Temperature:  42.5°C                                │
│ Mounting:     ✓ OK                                  │
└─────────────────────────────────────────────────────┘

Found 1 connection(s)

┌─ Connection #1 ─────────────────────────────────────┐
│ Protocol:     Modbus RTU                            │
│ Port:         /dev/ttyAMA0                          │
│ Baudrate:     115200                                │
│ Slave ID:     1                                     │
│ Response Time:  45ms                                │
│ Status:       ✓ PROTOCOL VERIFIED                   │
└─────────────────────────────────────────────────────┘

╔══════════════════════════════════════════════════════╗
║  Next Steps (Robot Platform)                        ║
╚══════════════════════════════════════════════════════╝

1. Query device via QNC Modbus bridge to identify it
2. Match device to descriptor database
3. Load appropriate descriptor JSON
4. Initialize device using descriptor

Example:
  ./qnc_modbus_read /dev/ttyAMA0 1 1000  # Read product ID
  # Match to descriptor database
  ./qnc_init_from_descriptor dh_ag95.json
```

#### 14.4.2 Modbus Scan Tool (Protocol-Level Only)

**Purpose**: Quick Modbus RTU protocol scan without handshaking.

**Location**: `examples/modbus_scan_example.cpp`

**Binary**: `build/modbus_scan`

**Usage**:
```bash
./modbus_scan <serial_port> [min_id] [max_id] [options]
```

**Options**:

| Option | Description |
|--------|-------------|
| `--quick` | Quick test: only test slave ID 1 |
| `--verbose` | Show detailed scanning progress |
| `--all-ids` | Scan all IDs 1-247 (slow) |
| `--fast` | Faster scan with reduced timeout (200ms) |

**Example**:
```bash
# Quick test
./modbus_scan /dev/ttyAMA0 --quick

# Scan range with verbose output
./modbus_scan /dev/ttyAMA0 1 20 --verbose
```

**Output**: Reports slave ID, baudrate, and response time only. No device identification.

### 14.5 Integration with Descriptor System

#### 14.5.1 Two-Phase Workflow

QNC protocol discovery works in conjunction with Section 16 device descriptors:

**Phase 1: QNC Protocol Discovery** (Automated, No Device Knowledge)
```bash
# QNC discovers protocol and connection parameters
./protocol_discovery /dev/ttyAMA0

# Output:  Protocol: Modbus RTU
#          Slave ID: 1
#          Baudrate: 115200
#          Port: /dev/ttyAMA0
```

**Phase 2: Robot Platform Device Management** (Using Descriptors)
```bash
# Robot platform identifies device via QNC bridge
qnc_read_register /dev/ttyAMA0 1 1000  # Read product code
# Returns: 0x1234 (example)

# Robot platform matches to descriptor
descriptor=$(match_device 0x1234)
# Returns: "/etc/qnc/devices/dh_ag95.json"

# Robot platform initializes via descriptor
qnc_init_from_descriptor /dev/ttyAMA0 1 $descriptor
```

#### 14.5.2 Supported Protocols

| Protocol | Detection | Connection Params Reported | Device Identification |
|----------|-----------|---------------------------|------------------------|
| Modbus RTU | ✓ | Slave ID, baudrate, port | Via descriptor (Section 16) |
| IO-Link | ✓ | COM mode, port | Via descriptor (Section 16) |
| EtherNet/IP | Future | IP address, port | Via descriptor (Section 16) |
| EtherCAT | Future | Station address | Via descriptor (Section 16) |

**Key Point**: QNC detects **protocols**, not devices. All device semantics are in descriptors.

#### 14.5.3 Descriptor Matching Strategies

Robot platforms can identify devices using various strategies:

**Strategy 1: Standard Register Query**
```cpp
// Read standard Modbus identification registers
uint16_t vendor_id = qnc.read_register(port, slave_id, 0x0000);
uint16_t product_code = qnc.read_register(port, slave_id, 0x0001);

// Match to descriptor database
std::string descriptor = descriptor_db.find(vendor_id, product_code);
```

**Strategy 2: Register Signature Matching**
```cpp
// Test for specific register patterns
bool has_dh_pattern = qnc.test_register_exists(port, slave_id, 256) &&
                      qnc.test_register_exists(port, slave_id, 512);
                      
if (has_dh_pattern) {
    descriptor = "/etc/qnc/devices/dh_robotics_generic.json";
}
```

**Strategy 3: User Configuration**
```cpp
// User explicitly specifies device type
std::string device_type = config.get("gripper_type");  // e.g., "dh_ag95"
descriptor = "/etc/qnc/devices/" + device_type + ".json";
```

**All strategies are robot platform logic - NOT in QNC firmware.**

### 14.6 Discovery Timing

#### 14.6.1 Typical Protocol Discovery Times

| Operation | Quick Mode | Normal Mode | Full Scan |
|-----------|------------|-------------|-----------|
| Handshake between QNC & end-effector | 50-100ms | 50-100ms | 50-100ms |
| Modbus protocol scan (1 ID) | 50-100ms | 100-200ms | 100-200ms |
| Modbus protocol scan (10 IDs) | - | 500-1000ms | 500-1000ms |
| Modbus protocol scan (all 247 IDs) | - | - | 30-60s |
| Protocol detection (Modbus + IO-Link) | 200-500ms | 500-1000ms | 1-3s |
| **Total (quick, protocol only)** | **300-700ms** | - | - |
| **Total (normal, protocol only)** | - | **650-1200ms** | - |
| **Total (full scan, protocol only)** | - | - | **30-61s** |

**Note**: Device initialization timing depends on descriptor-specific sequences and is handled by the robot platform (not QNC). Typical initialization adds 100-2000ms depending on device.

#### 14.6.2 Timeout Configuration

| Parameter | Default | Range | Description |
|-----------|---------|-------|-------------|
| Response timeout | 500ms | 100-5000ms | Per-command protocol timeout |
| Handshake timeout (QNC-to-device) | 2000ms | 1000-10000ms | Power/mount verification |
| Early termination | 20 attempts | 10-100 | Stop after N consecutive failures |

**Early Termination Logic**:
- If 20 consecutive slave IDs don't respond, assume no more devices on bus
- Prevents unnecessary scanning of empty address ranges
- Significantly reduces total scan time when devices are sparse

**Removed**: Init timeout (device initialization now handled via descriptors, timing varies by device)

### 14.7 Error Handling

#### 14.7.1 Discovery Errors

| Error | Code | Recovery |
|-------|------|----------|
| No protocol detected | -101 | Check connections, try --all-protocols |
| Handshake failed (QNC-to-device) | -102 | Verify power supply, mounting |
| Timeout during scan | -2 | Check device power, wiring |
| Protocol detection failed | -9 | Device may use unsupported protocol |

**Removed**: Initialization errors (now handled by robot platform using descriptors)

#### 14.7.2 Troubleshooting Guide

**No Protocols Detected**:
1. Verify device power supply (24V and 5V)
2. Check serial port wiring (A+, B-, GND for RS-485)
3. Confirm device is powered on
4. Verify correct serial port (`/dev/ttyAMA0` vs `/dev/ttyUSB0`)
5. Try `--all-protocols` option
6. Use `--verbose` for detailed diagnostics

**Handshake Failed (QNC-to-Device) - Power Supply**:
```
Error: Power supply voltages out of range
```
- Check 24V rail (acceptable: 22-26V)
- Check 5V rail (acceptable: 4.75-5.25V)
- Verify power connections and fuses

**Handshake Failed (QNC-to-Device) - Mounting**:
```
Error: QNC not properly mounted
```
- Ensure QNC is securely attached
- Check mounting sensors/switches
- Verify connector engagement

**Protocol Detected But Device Won't Respond to Queries**:
- Protocol detection successful but device queries fail
- Device may require initialization before normal operation
- Robot platform should load device descriptor and follow initialization sequence
- See Section 16 for descriptor-based initialization

### 14.8 Integration Example

**Complete Robot System Integration with Descriptor-Based Initialization**:

```cpp
#include "qnc_system_manager.h"
#include "descriptor_manager.h"
#include <iostream>

int main() {
    using namespace qnc;
    
    // Initialize QNC system
    system::SystemManager qnc;
    
    //=================================================================
    // PHASE 1: QNC PROTOCOL DISCOVERY (No device-specific logic)
    //=================================================================
    
    discovery::ProtocolDiscoveryConfig config;
    config.perform_handshake = true;      // Verify hardware
    config.try_modbus = true;             // Enable Modbus
    config.try_iolink = true;             // Enable IO-Link
    config.modbus_config.min_slave_id = 1;
    config.modbus_config.max_slave_id = 10;  // Limit scan range
    config.verbose = false;               // Production: quiet mode
    
    std::cout << "Phase 1: Discovering protocol...\n";
    auto discovery_result = qnc.discover_protocol("/dev/ttyAMA0", config);
    
    // Check discovery results
    if (!discovery_result.success) {
        std::cerr << "ERROR: Protocol discovery failed - " 
                  << discovery_result.error_message << "\n";
        return 1;
    }
    
    // Verify handshake
    if (!discovery_result.handshake.power_ok) {
        std::cerr << "ERROR: Power supply problem\n";
        return 1;
    }
    
    // Check protocol detection
    if (discovery_result.detected_connections.empty()) {
        std::cerr << "ERROR: No protocols detected\n";
        return 1;
    }
    
    auto& connection = discovery_result.detected_connections[0];
    std::cout << "Protocol detected: " 
              << protocol_type_to_string(connection.protocol) << "\n";
    std::cout << "Port: " << connection.port << "\n";
    std::cout << "Baudrate: " << connection.baudrate << "\n";
    
    if (connection.protocol == ProtocolType::MODBUS_RTU) {
        std::cout << "Slave ID: " << (int)connection.slave_id << "\n";
    }
    
    //=================================================================
    // PHASE 2: ROBOT PLATFORM DEVICE IDENTIFICATION (Via QNC bridge)
    //=================================================================
    
    std::cout << "\nPhase 2: Identifying device...\n";
    std::string descriptor_path;
    
    if (connection.protocol == ProtocolType::MODBUS_RTU) {
        // Use QNC as generic Modbus bridge to query device
        try {
            // Read vendor/product identification registers
            uint16_t product_code = qnc.read_modbus_register(
                connection.port, connection.slave_id, 1000);
            uint16_t device_version = qnc.read_modbus_register(
                connection.port, connection.slave_id, 1001);
            
            std::cout << "Product code: 0x" << std::hex << product_code << "\n";
            
            // Robot platform matches to descriptor database
            descriptor_path = match_device_to_descriptor(
                product_code, device_version);
            
            std::cout << "Matched descriptor: " << descriptor_path << "\n";
            
        } catch (const std::exception& e) {
            std::cerr << "WARNING: Could not read device ID registers\n";
            std::cerr << "Using default descriptor or user configuration\n";
            descriptor_path = "/etc/qnc/devices/generic_modbus.json";
        }
    }
    
    if (descriptor_path.empty()) {
        std::cerr << "ERROR: No descriptor found for device\n";
        return 1;
    }
    
    //=================================================================
    // PHASE 3: DESCRIPTOR-BASED INITIALIZATION (QNC executes from JSON)
    //=================================================================
    
    std::cout << "\nPhase 3: Initializing device from descriptor...\n";
    
    // QNC reads initialization sequence from descriptor JSON
    auto init_result = qnc.initialize_from_descriptor(
        connection, descriptor_path);
    
    if (!init_result.success) {
        std::cerr << "ERROR: Initialization failed - " 
                  << init_result.error_message << "\n";
        return 1;
    }
    
    std::cout << "Device initialized successfully!\n";
    std::cout << "\n╔══════════════════════════════════════╗\n";
    std::cout << "║  System Ready for Operations         ║\n";
    std::cout << "╚══════════════════════════════════════╝\n";
    
    // Now robot can control device via QNC bridge using descriptor register map
    return 0;
}

//=================================================================
// HELPER FUNCTION (Robot platform logic, NOT in QNC)
//=================================================================

std::string match_device_to_descriptor(uint16_t product_code, 
                                       uint16_t version) {
    // Robot platform's device database (example)
    struct DeviceMatch {
        uint16_t code;
        std::string descriptor;
    };
    
    static const DeviceMatch database[] = {
        {0x1234, "/etc/qnc/devices/dh_ag95.json"},
        {0x1235, "/etc/qnc/devices/dh_cgc80.json"},
        {0x2000, "/etc/qnc/devices/robotiq_2f85.json"},
        {0x2001, "/etc/qnc/devices/robotiq_2f140.json"},
    };
    
    for (const auto& entry : database) {
        if (entry.code == product_code) {
            return entry.descriptor;
        }
    }
    
    return "";  // Not found
}
```

**Key Architectural Points**:

1. **Phase 1 (QNC)**: Protocol discovery - zero device knowledge
2. **Phase 2 (Robot)**: Device identification using QNC as bridge
3. **Phase 3 (QNC + Robot)**: Initialization via descriptor - QNC executes generic operations from JSON
4. **Device semantics**: Never hardcoded in QNC firmware
5. **Extensibility**: Adding new devices = adding JSON file, no firmware changes

### 14.9 Files and Implementation

**Header Files**:
- `include/protocol_discovery.h` - Protocol-level discovery API (renamed from unified_device_discovery.h)
- `include/modbus_protocol_scan.h` - Modbus protocol scanning functions (renamed from modbus_device_discovery.h)
- `include/qnc_system_manager.h` - System manager with protocol discovery integration
- `include/descriptor_manager.h` - Descriptor loading and initialization (NEW)

**Implementation Files**:
- `src/protocol_discovery.cpp` - Protocol discovery implementation (no device semantics)
- `src/modbus_protocol_scan.cpp` - Modbus protocol scanning (no device semantics)
- `src/qnc_system_manager.cpp` - System manager implementation
- `src/descriptor_manager.cpp` - Descriptor-based device initialization (NEW)

**Example Applications**:
- `examples/protocol_discovery_example.cpp` - Protocol discovery only (renamed from unified_discovery_example.cpp)
- `examples/modbus_scan_example.cpp` - Modbus protocol scan (renamed from device_discovery_example.cpp)
- `examples/descriptor_init_example.cpp` - Complete workflow with descriptor integration (NEW)

**Configuration Files**:
- `/etc/qnc/devices/*.json` - Device descriptor files (see Section 16)
- `/etc/qnc/config.yaml` - QNC configuration

**Build Targets**:
```bash
# Build protocol discovery tools
cd build
cmake ..
make protocol_discovery modbus_scan descriptor_init_example
```

**Architecture Summary**:

| Component | Responsibility | Device Knowledge |
|-----------|---------------|------------------|
| QNC Protocol Discovery | Detect protocols, report connection params | **Zero** |
| Robot Platform | Device identification, descriptor selection | **All** (via JSON) |
| QNC Descriptor Manager | Execute init sequences from JSON | **Zero** (reads JSON) |
| Device Descriptors | Store all device-specific semantics | **All** (declarative) |

**Design Compliance**: This architecture maintains QNC's core principle: "QNC handles protocols, NOT devices."

---

## 15. Firmware Update Mechanism

### 14.1 Overview

QNC supports firmware updates through multiple mechanisms to accommodate different deployment scenarios. Updates can be delivered over-the-air (OTA) via network or through physical media (USB).

### 14.2 Supported Update Methods

| Method | Interface | Use Case | Priority |
|--------|-----------|----------|----------|
| OTA Network | Ethernet/WiFi | Remote updates, fleet management | Primary |
| USB Device | USB-A port | Field updates, no network | Secondary |
| NeuraSync Transfer | NeuraSync middleware | Robot-initiated updates | Tertiary |
| SD Card | MicroSD slot | Recovery, factory updates | Emergency |

### 14.3 OTA Update Process

**Update Flow**:

```
QNC                    Update Server           Robot Controller
 |                          |                         |
 |-- Check for update ----->|                         |
 |<-- Update available -----|                         |
 |    (version, size, hash) |                         |
 |                          |                         |
 |-- Notify robot ----------------------->|           |
 |    [Update available]                  |           |
 |                          |              |           |
 |<-- Approve update ---------------------|           |
 |                          |                         |
 |-- Download update ------>|                         |
 |<-- Transfer chunks ------|                         |
 |    (progress updates)    |                         |
 |                          |                         |
 |-- Verify checksum -------|                         |
 |   (SHA256 hash)          |                         |
 |                          |                         |
 |-- Create backup ---------|                         |
 |                          |                         |
 |-- Install update --------|                         |
 |                          |                         |
 |-- Reboot ----------------|                         |
 |                          |                         |
 |-- Verify new version ----|                         |
 |                          |                         |
 |-- Report success ---------------------------->|    |
 |                          |                         |
```

### 14.4 Update Message Structures

**Topic**: `qnc/system/update/available` (IF-016)

**Direction**: QNC → Robot (notification)

```cpp
struct UpdateAvailable {
    string current_version;        // Currently running version
    string available_version;      // New version available
    string release_notes;          // Update description
    uint64 update_size_bytes;      // Download size
    string checksum_sha256;        // SHA256 hash for verification
    boolean is_critical;           // Security/critical bug fix
    boolean compatible;            // Compatible with current hardware
    string timestamp;              // ISO 8601 timestamp
};
```

**Topic**: `qnc/system/update/command` (IF-017)

**Direction**: Robot → QNC (control)

```cpp
enum UpdateCommand {
    START_UPDATE,
    CANCEL_UPDATE,
    ROLLBACK
};

struct UpdateControl {
    UpdateCommand command;
    boolean create_backup;         // Create backup before update
    string timestamp;              // ISO 8601 timestamp
};
```

**Topic**: `qnc/system/update/progress` (IF-018)

**Direction**: QNC → Robot (status)

```cpp
enum UpdateStatus {
    IDLE,
    CHECKING,
    DOWNLOADING,
    VERIFYING,
    INSTALLING,
    COMPLETED,
    FAILED,
    ROLLBACK_IN_PROGRESS
};

struct UpdateProgress {
    UpdateStatus status;
    uint32 progress_percent;       // 0-100
    uint64 bytes_downloaded;
    uint64 total_bytes;
    string current_step;           // Human-readable step description
    string error_message;          // Error if status=FAILED
    string timestamp;              // ISO 8601 timestamp
};
```

### 14.5 OTA Update Configuration

**Update Server Configuration** (`/etc/qnc/config.yaml`):

```yaml
qnc:
  updates:
    ota:
      enabled: true
      server_url: "https://updates.neura-sync.com/qnc"
      check_interval_hours: 24
      auto_install: false           # Require approval
      verify_signature: true
      backup_before_update: true
      
    usb:
      enabled: true
      mount_path: "/media/usb0"
      auto_detect: true
      file_pattern: "qnc-firmware-*.img"
      
    security:
      require_signature: true
      trusted_keys_path: "/etc/qnc/trusted-keys/"
      allow_downgrade: false
```

### 14.6 USB Update Process

**Prerequisites**:
- USB-A port available on QNC carrier board
- Formatted USB drive (FAT32 or ext4)
- Update file named: `qnc-firmware-<version>.img`

**Manual USB Update**:

1. **Preparation**:
   ```bash
   # Download update file
   wget https://updates.neura-sync.com/qnc/qnc-firmware-1.1.0.img
   
   # Copy to USB drive
   cp qnc-firmware-1.1.0.img /media/usb/
   
   # Safely eject
   sync && umount /media/usb
   ```

2. **Installation**:
   - Insert USB drive into QNC
   - QNC auto-detects update file
   - Publishes `UpdateAvailable` message to robot
   - Robot approves update
   - QNC extracts and installs update
   - System reboots automatically

3. **Verification**:
   - QNC boots with new firmware
   - Performs self-test
   - Reports new version to robot

**LED Indicators During USB Update**:
- **Slow blink**: Reading USB drive
- **Fast blink**: Installing update
- **Solid**: Update complete, reboot pending
- **Flashing error pattern**: Update failed

### 14.7 Update Security

**Digital Signature Verification**:

1. All official updates signed with NeuraSync private key
2. QNC verifies signature using embedded public key
3. Update rejected if signature invalid
4. Prevents installation of unauthorized firmware

**Checksum Verification**:
- SHA256 hash computed for downloaded file
- Compared against published hash
- Installation aborted if mismatch detected

**Version Compatibility**:
- Update package contains compatibility matrix
- QNC hardware revision checked
- Update rejected if incompatible

### 14.8 Backup and Rollback

**Automatic Backup** (if enabled):
- Current firmware backed up before update
- Stored in separate partition
- Retained for one generation

**Rollback Triggers**:
- Boot failure after update (automatic after 3 attempts)
- Self-test failure
- Robot controller command
- Manual intervention (button press during boot)

**Rollback Process**:
1. Boot into recovery mode
2. Restore previous firmware from backup
3. Verify restored firmware
4. Boot with previous version
5. Report rollback event to robot

### 14.9 Update Timing and Scheduling

**Robot Controller Responsibilities**:

- Schedule updates during maintenance windows
- Ensure robot is idle before approving update
- Monitor update progress
- Verify successful update completion

**Recommended Update Windows**:
- During scheduled maintenance
- After shift completion
- When robot is not in production
- With operator supervision

**Update Duration Estimates**:

| Update Type | Size | Download | Install | Total |
|-------------|------|----------|---------|-------|
| Patch update | < 50 MB | 30s | 2 min | ~3 min |
| Minor update | 50-200 MB | 2 min | 5 min | ~8 min |
| Major update | > 200 MB | 5 min | 10 min | ~15 min |

*Note: Download times assume 10 Mbps network connection*

### 14.10 Update Error Handling

**Common Update Errors**:

| Error Code | Description | Recovery |
|------------|-------------|----------|
| -200 | Download failed | Retry download |
| -201 | Verification failed (checksum) | Re-download |
| -202 | Installation failed | Rollback to previous |
| -203 | Signature invalid | Reject update |
| -204 | Incompatible version | Manually upgrade |
| -205 | Insufficient storage | Free space, retry |

**Failed Update Response**:
- Automatic rollback initiated
- Error logged with details
- Robot notified of failure
- Previous firmware restored
- System returns to operational state

---

## 16. Configuration Interface

### 16.1 Device Descriptor Format (JSON)

**File Location**: `/etc/qnc/devices/<device_name>.json`

**Structure**:
```json
{
  "device": {
    "manufacturer": "DH-Robotics",
    "model": "AG-95",
    "protocol": "modbus_rtu",
    "description": "Collaborative electric gripper"
  },
  "connection": {
    "interface": "rs485",
    "port": "/dev/ttyAMA0",
    "baudrate": 115200,
    "data_bits": 8,
    "stop_bits": 1,
    "parity": "none",
    "slave_id": 1,
    "timeout_ms": 500
  },
  "capabilities": {
    "min_position_mm": 0,
    "max_position_mm": 95,
    "min_force_percent": 20,
    "max_force_percent": 100,
    "max_speed_mm_s": 150
  },
  "registers": {
    "control": {
      "position_target": {
        "address": 259,
        "type": "uint16",
        "unit": "mm",
        "scaling": 10,
        "description": "Target position (value * 0.1mm)"
      },
      "force_target": {
        "address": 257,
        "type": "uint16",
        "unit": "percent",
        "range": [20, 100],
        "description": "Gripping force percentage"
      }
    },
    "status": {
      "position_current": {
        "address": 514,
        "type": "uint16",
        "unit": "mm",
        "scaling": 10,
        "description": "Current position (value * 0.1mm)"
      },
      "status_word": {
        "address": 513,
        "type": "uint16",
        "description": "Status bits"
      }
    }
  },
  "initialization": {
    "sequence": [
      {"register": 256, "value": 1, "description": "Start initialization"},
      {"register": 256, "value": 165, "description": "Full initialization (0xA5)"}
    ]
  }
}
```

**Field Constraints**:

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| device.manufacturer | string | Yes | Max 64 chars |
| device.model | string | Yes | Max 64 chars |
| device.protocol | enum | Yes | Must match ProtocolType |
| connection.port | string | Yes | Valid device path |
| connection.baudrate | integer | Yes | Valid baudrate |
| connection.slave_id | integer | Yes | 1-247 range |
| registers | object | No | Valid register map |

### 16.2 QNC Configuration File (YAML)

**File Location**: `/etc/qnc/config.yaml`

**Structure**:
```yaml
qnc:
  detection:
    enable_hardware_id: true
    enable_protocol_probe: true
    enable_descriptor: true
    default_descriptor: "/etc/qnc/devices/default.json"
    probe_order:
      - modbus_rtu
      - iolink
    probe_timeout_ms: 500
    cache_detection: true
    
  protocols:
    modbus_rtu:
      default_port: "/dev/ttyAMA0"
      default_baudrate: 115200
      timeout_ms: 500
      retry_count: 3
      
    iolink:
      uart_device: "/dev/ttyAMA0"
      default_mode: COM2
      gpio_txen: 17
      gpio_irq: 27
      
  hot_swap:
    enable: true
    monitoring_interval_ms: 1000
    health_check_interval_ms: 5000
    
    neurasync:
    domain_id: 0
    enable_generic_interface: true
    enable_protocol_specific: true
    command_topic: "qnc/bridge/command"
    response_topic: "qnc/bridge/response"
    status_topic: "qnc/bridge/status"
    
  logging:
    level: INFO
    file: "/var/log/qnc/qnc.log"
    max_size_mb: 100
    max_files: 10
```

---

## 17. Integration Guidelines

### 17.1 Robot Application Integration

**Step 1**: Initialize NeuraSync communication
```cpp
#include <dds/dds.hpp>
#include "UnifiedProtocolBridge.hpp"

// Create NeuraSync participant
dds::domain::DomainParticipant participant(0);

// Create publisher and subscriber
dds::pub::Publisher publisher(participant);
dds::sub::Subscriber subscriber(participant);
```

**Step 2**: Create topics and readers/writers
```cpp
// Command topic
dds::topic::Topic<qnc::bridge::GenericCommand> cmd_topic(
    participant, "qnc/bridge/command");
dds::pub::DataWriter<qnc::bridge::GenericCommand> cmd_writer(
    publisher, cmd_topic);

// Response topic
dds::topic::Topic<qnc::bridge::GenericResponse> resp_topic(
    participant, "qnc/bridge/response");
dds::sub::DataReader<qnc::bridge::GenericResponse> resp_reader(
    subscriber, resp_topic);
```

**Step 3**: Send commands
```cpp
qnc::bridge::GenericCommand cmd;
cmd.command_type(qnc::bridge::CommandType::WRITE);
cmd.device_id(1);
cmd.address(259);
cmd.data({0x01, 0xF4});  // 500 (50.0mm)
cmd.data_type(qnc::bridge::DataType::UINT16);
cmd.timeout_ms(500);
cmd.tag("set_position");

cmd_writer.write(cmd);
```

**Step 4**: Receive responses
```cpp
// Create listener for responses
class ResponseListener : public dds::sub::NoOpDataReaderListener<qnc::bridge::GenericResponse> {
    void on_data_available(dds::sub::DataReader<qnc::bridge::GenericResponse>& reader) override {
        auto samples = reader.take();
        for (const auto& sample : samples) {
            if (sample.info().valid()) {
                auto resp = sample.data();
                if (resp.error_code() == 0) {
                    // Success
                    std::cout << "Command succeeded, latency: " << resp.latency_us() << " us\n";
                } else {
                    // Error
                    std::cerr << "Command failed: " << resp.error_message() << "\n";
                }
            }
        }
    }
};

auto listener = std::make_shared<ResponseListener>();
resp_reader.listener(listener.get());
```

### 17.2 Device Descriptor Creation

**Step 1**: Obtain device manual/datasheet

**Step 2**: Create descriptor JSON file
- Copy template from `/etc/qnc/devices/template.json`
- Fill in device information
- Map all registers from datasheet
- Specify initialization sequence

**Step 3**: Validate descriptor
```bash
qnc-validate-descriptor /path/to/device.json
```

**Step 4**: Install descriptor
```bash
sudo cp device.json /etc/qnc/devices/
```

### 17.3 Protocol Probing Configuration

To optimize detection speed, configure probe order based on expected devices:

```yaml
qnc:
  detection:
    probe_order:
      - modbus_rtu      # Try ModbusRTU first (most common)
      - iolink          # Then IO-Link
    probe_timeout_ms: 200  # Reduce timeout for faster detection
```

### 17.4 Performance Optimization

**For Low-Latency Applications**:
1. Use protocol-specific interface (e.g., ModbusRTU direct)
2. Set NeuraSync QoS to BEST_EFFORT for non-critical data
3. Reduce health check interval
4. Use descriptor-based detection (skip probing)

**For Reliability**:
1. Use generic interface with automatic translation
2. Enable all detection methods
3. Configure retry logic
4. Use RELIABLE NeuraSync QoS
5. Enable comprehensive logging

---

## 18. Appendices

### Appendix A: Complete IDL Definitions

**File**: `UnifiedProtocolBridge.idl`

```idl
module qnc {
    module bridge {
        
        enum CommandType {
            UNKNOWN, READ, WRITE, READ_WRITE, EXECUTE,
            CONFIGURE, STATUS_REQUEST, RESET, EMERGENCY_STOP
        };
        
        enum ProtocolType {
            PROTOCOL_UNKNOWN, MODBUS_RTU, IOLINK, ETHERCAT,
            ETHERNET_IP, PROFINET, CANOPEN, CUSTOM
        };
        
        enum DataType {
            UINT8, INT8, UINT16, INT16, UINT32, INT32,
            FLOAT, DOUBLE, BOOL, STRING, BYTE_ARRAY
        };
        
        struct GenericCommand {
            CommandType command_type;
            uint32 device_id;
            uint32 address;
            sequence<octet> data;
            DataType data_type;
            uint32 timeout_ms;
            string tag;
            string timestamp;
        };
        
        struct GenericResponse {
            int32 error_code;
            string error_message;
            sequence<octet> data;
            DataType data_type;
            uint32 latency_us;
            string tag;
            string timestamp;
        };
        
        struct DeviceInfo {
            string manufacturer;
            string model;
            string serial_number;
            string firmware_version;
            ProtocolType protocol;
            string descriptor_file;
            uint32 vendor_id;
            uint32 device_id;
        };
        
        struct BridgeStatus {
            boolean initialized;
            boolean device_connected;
            ProtocolType active_protocol;
            DeviceInfo device_info;
            uint32 uptime_seconds;
            uint64 total_commands;
            uint64 failed_commands;
            uint32 avg_latency_us;
            string timestamp;
        };
        
        struct DeviceChangeEvent {
            DeviceInfo old_device;
            DeviceInfo new_device;
            string reason;
            string timestamp;
        };
        
        struct TranslationRule {
            string rule_id;
            ProtocolType source_protocol;
            uint32 source_command;
            ProtocolType target_protocol;
            uint32 target_address;
            boolean bidirectional;
            string description;
        };
        
        // System handshaking and management
        struct HandshakeRequest {
            string qnc_serial_number;
            string firmware_version;
            uint32 voltage_24v_mv;
            uint32 voltage_5v_mv;
            float temperature_celsius;
            boolean mount_verified;
            boolean self_test_passed;
            uint64 uptime_seconds;
            string timestamp;
        };
        
        struct HandshakeAck {
            boolean acknowledge;
            string robot_id;
            string robot_version;
            boolean ready_for_operations;
            string reason;
            string timestamp;
        };
        
        struct QNCHeartbeat {
            uint64 sequence_number;
            boolean operational;
            string status_message;
            string timestamp;
        };
        
        // Firmware update
        enum UpdateCommand {
            START_UPDATE, CANCEL_UPDATE, ROLLBACK
        };
        
        enum UpdateStatus {
            IDLE, CHECKING, DOWNLOADING, VERIFYING,
            INSTALLING, COMPLETED, FAILED, ROLLBACK_IN_PROGRESS
        };
        
        struct UpdateAvailable {
            string current_version;
            string available_version;
            string release_notes;
            uint64 update_size_bytes;
            string checksum_sha256;
            boolean is_critical;
            boolean compatible;
            string timestamp;
        };
        
        struct UpdateControl {
            UpdateCommand command;
            boolean create_backup;
            string timestamp;
        };
        
        struct UpdateProgress {
            UpdateStatus status;
            uint32 progress_percent;
            uint64 bytes_downloaded;
            uint64 total_bytes;
            string current_step;
            string error_message;
            string timestamp;
        };
    };
};
```

### Appendix B: Error Code Reference

Complete list of error codes with descriptions and recovery actions.

| Code | Name | Category | Description | Recovery |
|------|------|----------|-------------|----------|
| 0 | SUCCESS | - | Operation successful | None |
| -1 | GENERIC_ERROR | Comm | Unspecified error | Investigate logs |
| -2 | TIMEOUT | Comm | Device timeout | Retry, check connection |
| -3 | DISCONNECTED | Comm | Device not connected | Reconnect |
| -4 | INVALID_COMMAND | Cmd | Invalid command format | Fix command |
| -5 | INVALID_ADDRESS | Cmd | Invalid register address | Check descriptor |
| -6 | PERMISSION_DENIED | Cmd | Operation not allowed | Check permissions |
| -7 | DEVICE_BUSY | Cmd | Device busy | Wait and retry |
| -8 | CRC_ERROR | Comm | Checksum error | Retry, check cabling |
| -9 | PROTOCOL_ERROR | Comm | Protocol-level error | Verify protocol |
| -10 | TRANSLATION_ERROR | Cmd | Translation failed | Check mapping rules |
| -21 | NOT_INITIALIZED | Sys | System not initialized | Initialize first |
| -22 | RESOURCE_UNAVAILABLE | Sys | Resource not available | Check resources |
| -23 | INTERNAL_ERROR | Sys | Internal system error | Contact support |
| -100 | POWER_FAILURE | Sys | Power supply out of range | Check 24V/5V rails |
| -101 | MOUNT_FAILURE | Sys | Mechanical mount failure | Verify mounting |
| -102 | CONNECTION_TIMEOUT | Sys | NeuraSync connection timeout | Check network |
| -103 | DEVICE_DISCONNECT | Comm | Device disconnected | Reconnect device |
| -104 | DEVICE_DISCONNECT_DURING_MOTION | Critical | Device lost during motion | **TRIGGER E-STOP** |
| -200 | UPDATE_DOWNLOAD_FAILED | Update | Firmware download failed | Retry download |
| -201 | UPDATE_VERIFY_FAILED | Update | Update verification failed | Re-download |
| -202 | UPDATE_INSTALL_FAILED | Update | Installation failed | Rollback firmware |
| -203 | UPDATE_SIGNATURE_INVALID | Update | Invalid digital signature | Reject update |
| -204 | UPDATE_INCOMPATIBLE | Update | Incompatible version | Manual upgrade |
| -205 | UPDATE_INSUFFICIENT_STORAGE | Update | Not enough storage | Free space |

### Appendix C: Compliance Matrix

| Requirement | Interface | Status | Notes |
|-------------|-----------|--------|-------|
| Generic command/response | IF-001, IF-002 | Mandatory | ✅ Defined |
| Bridge status | IF-003 | Mandatory | ✅ Defined |
| Device events | IF-004 | Recommended | ✅ Defined |
| ModbusRTU interface | IF-005 to IF-008 | Optional | ✅ Defined |
| RS485 hardware | IF-009 | Protocol-specific | ✅ Defined |
| SPI/UART (IO-Link) | IF-010 | Protocol-specific | ✅ Defined |
| Hardware ID | IF-011 | Optional | ✅ Defined |
| Configuration | IF-012 | Recommended | ✅ Defined |
| Handshake request | IF-013 | Mandatory | ✅ Defined |
| Handshake ACK | IF-014 | Mandatory | ✅ Defined |
| QNC Heartbeat | IF-015 | Mandatory | ✅ Defined |
| Update available | IF-016 | Recommended | ✅ Defined |
| Update command | IF-017 | Recommended | ✅ Defined |
| Update progress | IF-018 | Recommended | ✅ Defined |

### Appendix D: Glossary

| Term | Definition |
|------|------------|
| **Bridge** | QNC firmware component that translates between protocols |
| **NeuraSync** | In-house communication framework based on FastDDS |
| **Descriptor** | JSON file containing device register maps and configuration |
| **Hot-swap** | Ability to change devices without system restart |
| **ICD** | Interface Control Document - this document |
| **Protocol Adapter** | Software module implementing specific protocol (ModbusRTU, IO-Link, etc.) |
| **QNC** | Quick Network Connector - multi-protocol bridge system |
| **Translation** | Converting commands from one protocol format to another |

### Appendix E: Change History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 0.1 | 2026-02-01 | Engineering | Initial draft |
| 0.5 | 2026-02-10 | Engineering | Added protocol-specific interfaces |
| 0.9 | 2026-02-15 | Engineering | Review comments incorporated |
| 1.0 | 2026-02-18 | Engineering | Official release |
| 1.1 | 2026-02-19 | Engineering | Added system handshaking, enhanced error handling, firmware update mechanism |

### Appendix F: Consolidated Architecture & Implementation Snapshot

This appendix consolidates architecture/implementation context previously split across the following legacy documents (now removed to avoid duplication):

- `ARCHITECTURE_DIAGRAMS.md`
- `ARCHITECTURE.md`
- `FIRMWARE_ARCHITECTURE.md`
- `PROTOCOL_BRIDGE_README.md`
- `IMPLEMENTATION_SUMMARY.md`

#### F.1 Layered Firmware View (Normative Summary)

| Layer | Primary Components | Responsibility |
|------|---------------------|----------------|
| L5 | Unified protocol bridge + NeuraSync interface | Command ingress/egress and bridge-facing API |
| L4 | Protocol translator | Mapping, data transforms, cross-protocol translation |
| L3 | Protocol manager | Protocol registry, switching, hot-swap lifecycle |
| L2 | Protocol detector | Hardware ID, protocol probing, descriptor-assisted detection |
| L1 | Protocol adapters (`IProtocol`) | Protocol-specific read/write/connect implementation |
| L0 | Hardware interfaces | RS485, SPI/UART, Ethernet transport |

#### F.2 Responsibility Boundary (Normative)

| Domain | Owner | Scope |
|--------|-------|-------|
| Protocol framing, transport timing, retries, bus access | QNC firmware | Protocol operations only |
| Device register semantics, unit conversion, capability modeling | Robot platform | Descriptor-driven device intelligence |

This boundary enforces the core principle: QNC handles protocols, not device semantics.

#### F.3 Component-to-File Mapping (Implementation Reference)

| Component | Headers | Sources |
|-----------|---------|---------|
| Protocol detector | `include/protocol_detector.h` | `src/modbus_device_discovery.cpp`, `src/unified_device_discovery.cpp` |
| Protocol manager | `include/protocol_manager.h` | `src/qnc_system_manager.cpp` |
| Protocol translation | `include/protocol_translator.h` | `examples/protocol_translator_example.cpp` (reference) |
| Modbus adapter | `include/modbus_rtu_adapter.h` | `src/modbus_rtu_adapter.cpp` |
| Unified discovery bridge | `include/unified_device_discovery.h` | `src/unified_device_discovery.cpp` |

#### F.4 Implementation Status Snapshot

| Work Item | Status |
|-----------|--------|
| Interface contracts and message definitions | Defined in this ICD |
| Modbus transport bridge path | Implemented |
| Unified discovery and system manager integration | Implemented |
| Additional protocol adapters (beyond Modbus path) | Incremental rollout |

#### F.5 Build and Validation Quick Commands

```bash
cd /home/pi/Documents/qnc-cm5
mkdir -p build && cd build
cmake ..
make
```

```bash
./modbus_bridge
./protocol_translator_example
```

#### F.6 Single Source of Truth Rule

Architecture definitions, protocol flow descriptions, and interface/timing values must be maintained in this ICD. External architecture markdown files are informational pointers only.

---

## Document End

**For questions or clarifications regarding this ICD, contact:**

**NeuraSync Engineering Team**  
Email: engineering@neura-sync.com  
Document Ref: QNC-ICD-001 v1.0  

**Document Generated**: February 18, 2026  
**Next Review Date**: August 18, 2026 (6 months)

---

*This document is subject to change. Always refer to the latest version available in the official documentation repository.*


---
---
# Product Requirements Document: QNC Multi-Protocol Bridge System

**Document Number**: QNC-PRD-001  
**Version**: 2.1  
**Date**: February 19, 2026  
**Status**: Draft  
**Author**: Manus AI

**Revision History**:
- Version 1.0: Initial software-focused requirements
- Version 1.1: Added hardware specifications and roadmap
- Version 2.0: Comprehensive merge of all requirements (software + hardware)
- Version 2.1: Added system handshaking, enhanced error handling for device disconnection during motion, firmware update mechanism

---

## 1. Introduction

### 1.1 Purpose

This Product Requirements Document (PRD) outlines the essential features, functionalities, and technical specifications for the Quick Network Connector (QNC) multi-protocol bridge system. It serves as a foundational document for development, testing, and deployment, ensuring a shared understanding among all stakeholders regarding the product's objectives and capabilities. The information contained herein is derived from the QNC Interface Control Document (ICD) [1], the QNC Board Concept Planning and Roadmap [2], and supplemented by planned features.

### 1.2 Scope

The QNC system aims to revolutionize robotic platform integration with industrial end-effectors by providing a universal protocol-level communication bridge. This document details the requirements for the QNC bridge, covering its core functionalities, hardware architecture, supported interfaces, performance characteristics, and error handling mechanisms. It also incorporates planned features and a development roadmap. Out of scope are internal QNC firmware implementation details, device-specific semantics (which remain with the robot platform), and safety systems.

**Companion Documents**:
- **QNC-ICD-001**: Interface Control Document - For detailed technical interface specifications, data structures, message formats, timing diagrams, error codes, and protocol-level implementation details
- **QNC-ARCH-001**: Firmware Architecture - For internal implementation architecture and design patterns

### 1.3 Audience

This document is intended for a diverse audience, including:
*   **Product Management**: To guide product strategy and roadmap decisions.
*   **Engineering Teams**: To develop and implement the QNC system.
*   **Quality Assurance**: To design and execute comprehensive test plans.
*   **System Integrators**: To understand how to integrate QNC into broader robotic ecosystems.
*   **Technical Writers**: To create user manuals and technical documentation.
*   **Sales and Marketing**: To articulate the product's value proposition and features.

### 1.4 Document Conventions

Key terms and definitions are provided in the Glossary (Appendix A). Technical specifications and interface details are presented in tables and code snippets for clarity. All timing values are approximate and subject to further refinement during development and testing.

---

## 2. Product Overview

### 2.1 Product Vision

The vision for the QNC multi-protocol bridge system is to simplify and accelerate the integration of diverse industrial end-effectors and devices with robotic platforms. By abstracting away complex, device-specific protocol handling, QNC enables robot developers to focus on high-level application logic, thereby reducing development time, firmware complexity, and maintenance burden.

### 2.2 Problem Statement

Traditionally, integrating robotic platforms with various industrial end-effectors necessitates the development of numerous device-specific drivers and extensive protocol stack code. This approach leads to significant challenges:
*   **High Development Effort**: Each new device requires substantial coding (e.g., 500-600 lines per driver, 3000+ lines of protocol stack code) [1].
*   **Time-Consuming Integration**: Adding support for a new device can take up to a week, and debugging hardware/protocol issues can consume 2-3 days [1].
*   **Firmware Bloat and Maintenance**: Robot firmware becomes large (15,000+ lines of device-specific code) and requires continuous maintenance for protocol updates, leading to high deployment risk with every rebuild [1].
*   **Duplication of Effort**: Similar code is often duplicated across different robot variants, increasing inefficiency.

### 2.3 QNC Solution and Value Proposition

QNC addresses these challenges by acting as a **protocol-level bridge**, separating the concerns of robot control from device communication. It translates high-level commands from the robot platform into device-specific protocol messages and vice-versa. The core principle is that QNC handles **protocols**, not **devices**, with device-specific semantics managed by declarative JSON descriptors on the robot platform, not embedded code [1].

This approach offers significant benefits:
*   **Accelerated Device Support**: Adding new device support is reduced from 1 week to approximately 30 minutes, representing a 97% faster integration [1].
*   **Reduced Development Effort**: Effort to support 10 devices drops from 10 weeks to 5 hours, a 99% reduction [1].
*   **Faster Debugging**: Protocol debugging time is cut by 95%, from 2-3 days to 1 hour [1].
*   **Minimized Firmware Footprint**: Robot firmware size is drastically reduced from 15,000+ lines to around 500 lines, a 97% reduction [1].
*   **Eliminated Deployment Risk**: Updates become a simple JSON modification, eliminating the need for full robot firmware rebuilds [1].

**Universality as Core Product Value**:
The unique value of QNC lies in its **universality** as an interface, meaning it must enable data exchange between heterogeneous systems and remain compatible with most robotic arms and end-effectors. If QNC only supports a narrow subset of robot arms or tools, its bridging functionality could be integrated directly into the robot arm during development, making a separate component unjustified. Therefore, QNC SHALL prioritize broad mechanical, electrical, and protocol compatibility to avoid adding unnecessary assembly complexity and system cost.

### 2.4 High-Level Architecture

The QNC system integrates between the Robot Control System and various Field Devices (e.g., grippers, sensors, motors, PLCs). It utilizes NeuraSync, an in-house communication framework based on FastDDS, for communication with the robot platform. Internally, QNC features a Unified Protocol Bridge for detection, translation, and switching between protocols, supported by various Protocol Adapters (e.g., ModbusRTU, IO-Link) and Hardware Interfaces (e.g., RS485, SPI, Ethernet) [1].

The QNC Module is a **24 V powered assembly** featuring an **embedded Linux system** based on a high-performance **NXP i.MX 8M PLUS MCU** with multiple peripheral interfaces [2]. A single standardized connector supports various end modules.

```
┌──────────────────────────────────────────────────────────────┐
│                    ROBOT CONTROL SYSTEM                      │
│                                                              │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │   Motion   │  │Application │  │   Safety   │            │
│  │  Planning  │  │   Logic    │  │   System   │            │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘            │
│        └────────────────┴────────────────┘                  │
│                         │                                    │
│                  NeuraSync Middleware                              │
└─────────────────────────┼────────────────────────────────────┘
                          │
                          │ Network (Ethernet/WiFi)
                          │
┌─────────────────────────┼────────────────────────────────────┐
│                         │           QNC BRIDGE               │
│                  NeuraSync Interface                               │
│         ┌───────────────┴────────────────┐                  │
│         │  Unified Protocol Bridge       │                  │
│         │  - Protocol Detection          │                  │
│         │  - Protocol Translation        │                  │
│         │  - Protocol Switching          │                  │
│         └───────────────┬────────────────┘                  │
│                         │                                    │
│         ┌───────────────┴────────────────┐                  │
│         │     Protocol Adapters          │                  │
│         │  [ModbusRTU] [IO-Link] [...]   │                  │
│         └───────────────┬────────────────┘                  │
│                         │                                    │
│         ┌───────────────┴────────────────┐                  │
│         │    Hardware Interfaces         │                  │
│         │  [RS485] [SPI] [Ethernet]      │                  │
│         └───────────────┬────────────────┘                  │
└─────────────────────────┼────────────────────────────────────┘
                          │
                          │ Physical Connection
                          │
┌─────────────────────────┼────────────────────────────────────┐
│                         │                                    │
│              END-EFFECTOR (Gripper/Tool)                     │
│                                                              │
│  Supported Protocols:                                        │
│  • ModbusRTU (RS485)                                        │
│  • IO-Link (SPI/UART)                                       │
│  • EtherCAT (Ethernet)                                      │
│  • EtherNet/IP (Ethernet)                                   │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Functional Requirements

### 3.1 Core Communication Functionality

**FR-001: Generic Command/Response Interface**
The QNC system SHALL provide a generic NeuraSync-based interface for sending commands to and receiving responses from connected end-effectors. This interface SHALL support various command types (READ, WRITE, EXECUTE, CONFIGURE, etc.) and data types (UINT8, INT16, FLOAT, etc.) [1].

**FR-002: Protocol Translation**
The QNC system SHALL automatically translate generic commands from the robot platform into the appropriate protocol-specific messages (e.g., ModbusRTU, IO-Link) for the connected end-effector [1].

**FR-003: Protocol Switching**
The QNC system SHALL be capable of dynamically switching between different communication protocols based on the detected or configured end-effector [1].

**FR-004: Bridge Status Reporting**
The QNC system SHALL periodically publish its operational status, including initialization state, device connection status, active protocol, and device information, via a NeuraSync topic [1].

**FR-005: Device Change Event Notification**
The QNC system SHALL publish events to the robot platform upon detection of device connection, disconnection, or hot-swap events [1].

### 3.2 Protocol-Specific Interfaces

**FR-006: ModbusRTU Interface**
The QNC system SHALL provide specific NeuraSync topics for direct ModbusRTU write commands (`qnc/modbus/write_cmd`), read commands (`qnc/modbus/read_cmd`), and responses (`qnc/modbus/response`) [1].

**FR-007: ModbusRTU Statistics Reporting**
The QNC system SHALL periodically publish ModbusRTU bridge statistics, including transmitted/received frame counts, error counts, and uptime, via a NeuraSync topic (`qnc/modbus/stats`) [1].

**FR-008: IO-Link Interface**
The QNC system SHALL support IO-Link communication, including specific NeuraSync topics for IO-Link commands and responses [1].

**FR-009: Additional Protocol Support**
The QNC system SHALL support the following interfaces: EtherCAT (Class B only), WIFI 5G, ModbusTCP, RS485, RS232, SPI, I2C, Ethernet, CAN-FD, and a 24 V relay output [2].

### 3.3 Device Detection and Configuration

**FR-010: Hardware ID-based Device Detection**
The QNC system SHALL be able to detect connected devices using a resistor-based hardware ID pin, mapping specific resistance values to known devices and their protocols [1].

**FR-011: Protocol Probing for Device Detection**
The QNC system SHALL be able to probe connected devices using various protocols (e.g., ModbusRTU, IO-Link) to automatically identify the device and its communication protocol [1].

**FR-012: JSON Device Descriptor Support**
The QNC system SHALL utilize JSON files as device descriptors to define device-specific semantics, including manufacturer, model, protocol, connection parameters, capabilities, and register mappings [1].

**FR-013: YAML Configuration Management**
The QNC system SHALL allow configuration of its operational parameters, such as detection methods, protocol-specific settings, hot-swap behavior, NeuraSync parameters, and logging, via a YAML configuration file [1].

**FR-014: Device Descriptor Validation**
The QNC system SHALL provide a mechanism (e.g., `qnc-validate-descriptor` tool) to validate the syntax and structure of device descriptor JSON files [1].

### 3.4 Power Management

**FR-015: Powering Connected Devices**
The QNC system SHALL provide power to the connected end-effector while attached to the robot flange. The module requires approximately **1 A at 24 V** for the gripper, plus its own power consumption [2].

**FR-016: Battery Mode Operation**
When detached from the robot, the QNC system SHALL operate in a battery-powered mode (USV-Battery Mode). It SHALL be able to charge at 24 V and continue operating for **at least 10 minutes** when disconnected [2].

**FR-017: Battery Life Indication**
The QNC system SHALL include a display to show the state of charge of its internal battery [Planned Features].

**FR-018: Battery Charging**
The QNC system SHALL charge its internal battery when attached to the robot flange [Planned Features].

### 3.5 User Interface and Control

**FR-019: Internal Screen/External App Control**
The QNC system SHALL provide control and status monitoring via an internal screen or an external application [Planned Features].

**FR-020: Device Mode Switching**
The QNC system SHALL allow switching between different device modes via its internal screen or external app [Planned Features].

**FR-021: Motion Switching**
The QNC system SHALL allow switching between different motions via its internal screen or external app [Planned Features].

**FR-022: Gripper Function Selection**
The QNC system SHALL enable selection of various gripper functions via its internal screen or external app [Planned Features].

**FR-023: Pairing with External Devices**
The QNC system SHALL support pairing with external devices via NFC or Bluetooth for control and configuration [Planned Features].

### 3.6 Advanced Features

**FR-024: Pose Tracking**
The QNC system SHALL support pose tracking functionality [Planned Features].

**FR-025: Object Tracking with AI**
The QNC system SHALL incorporate AI-driven object tracking capabilities, utilizing the high-performance **i.MX 8M PLUS MCU** [2].

**FR-026: Camera Integration (D435i)**
The QNC system SHALL interface with a Camera D435i, supporting start/stop capture and start/stop pose tracking functionalities [Planned Features].

**FR-027: Drive Various Peripherals**
The QNC module SHALL be able to drive various peripherals, such as a **3D camera, gripper, and vacuum pumps** [2].

**FR-028: General AI Features**
The QNC system SHALL support general AI features, including vision and language processing, facilitated by the embedded Linux system [2].

---

## 4. Non-Functional Requirements

### 4.1 Performance Requirements

**NFR-001: End-to-End Latency**
The total end-to-end latency for command execution (Robot to Device) SHALL be typically 8 ms, with a maximum of 200 ms [1].

**NFR-002: NeuraSync Message Propagation Latency**
NeuraSync message propagation on the local network SHALL have a typical latency of 0.5 ms and a maximum of 2 ms [1].

**NFR-003: Command Translation Latency**
Command translation within the QNC system SHALL have a typical latency of 0.1 ms and a maximum of 1 ms [1].

**NFR-004: ModbusRTU Transaction Time**
ModbusRTU transactions SHALL typically complete within 2 ms, with a maximum of 50 ms (at 115200 baud) [1].

**NFR-005: IO-Link Transaction Time**
IO-Link transactions SHALL typically complete within 5 ms, with a maximum of 100 ms (in COM2 mode) [1].

**NFR-006: Hardware ID Detection Time**
Hardware ID reading for device detection SHALL take typically 0.5 ms, with a maximum of 1 ms [1].

**NFR-007: Descriptor Load Time**
Loading a device descriptor SHALL typically take 5 ms, with a maximum of 50 ms [1].

**NFR-008: Protocol Probe Time**
ModbusRTU probing SHALL take typically 100 ms per attempt (maximum 500 ms), and IO-Link probing SHALL take typically 200 ms (maximum 1000 ms) [1].

**NFR-009: Full Auto-Detection Time**
Full auto-detection of a device (across all protocols) SHALL typically complete within 500 ms, with a maximum of 3000 ms [1].

**NFR-010: Hot-Swap Recovery Time**
The QNC system SHALL recover from a hot-swap event (device disconnection and reconnection) within 1-3 seconds [1].

**NFR-011: Periodic Operation Rates**
*   Bridge status SHALL be published at 1 Hz (1000 ms period, ±50 ms jitter) [1].
*   Health checks SHALL be performed at 0.2 Hz (5000 ms period, ±100 ms jitter) [1].
*   Statistics SHALL be published at 5 Hz (200 ms period, ±20 ms jitter) [1].
*   Hot-swap monitoring SHALL occur at 1 Hz (1000 ms period, ±50 ms jitter) [1].

**NFR-012: Software Flexibility**
The QNC module MUST be **software-flexible**, allowing easy implementation of different applications [2].

### 4.2 Reliability and Error Handling

**NFR-013: Error Logging**
All errors SHALL be logged with timestamp, error code, error message, context (command that failed), device information, and attempted recovery action [1].

**NFR-014: Command Timeout**
The QNC system SHALL support configurable command timeouts, with a default of 500 ms and a range of 100-5000 ms [1].

**NFR-015: Connection Timeout**
Initial connection attempts SHALL have a timeout of 2000 ms, with a configurable range of 1000-10000 ms [1].

**NFR-016: Health Check Timeout**
Device health probes SHALL have a timeout of 1000 ms, with a configurable range of 500-5000 ms [1].

**NFR-017: Detection Timeout**
Auto-detection processes SHALL have a timeout of 3000 ms, with a configurable range of 1000-10000 ms [1].

**NFR-018: Error Recovery Mechanisms**
The QNC system SHALL implement specific recovery strategies for various error codes, including retries with exponential backoff for timeouts, automatic reconnection for disconnections, and validation for invalid commands/addresses [1].

**NFR-019: Fault Detection and Response**
The QNC system SHALL detect and respond to fault conditions such as device disconnects (via health check timeout), protocol mismatches (via probe failure), communication timeouts, invalid data (CRC/checksum fail), and buffer overflows [1].

**NFR-020: Fault Isolation**
Module failures MUST NOT propagate. The battery module uses **ideal diodes** to handle input power supply, allowing the QNC system to remain powered even if the battery module is removed [2].

### 4.3 Usability and Cost

**NFR-021: Ease of Device Integration**
The QNC system SHALL enable new device support with minimal effort, primarily through declarative JSON descriptors rather than extensive code changes [1].

**NFR-022: Configuration Simplicity**
Configuration of QNC operational parameters SHALL be straightforward via a YAML file [1].

**NFR-023: Target Cost**
The PCB MUST cost **less than €40** [2].

### 4.4 Physical Characteristics

**NFR-024: Weight Target**
The target weight for the QNC system SHALL be between 0.5 kg and 1.0 kg [Planned Features].

**NFR-025: Form Factor**
The QNC module MUST be smaller than **5 cm x 3 cm** [2].

**NFR-026: Touchable Surface Temperature**
The touchable surface temperature of the QNC system SHALL remain below 40 °C [Planned Features].

**NFR-027: Battery Life**
The internal battery SHALL provide at least 10 minutes of operation when detached [2].

### 4.5 Security Requirements

**NFR-028: Secure Communication (Future)**
The QNC system MAY incorporate secure communication protocols for NeuraSync and other interfaces as a future enhancement.

### 4.6 Maintainability Requirements

**NFR-029: Firmware Portability**
The QNC firmware SHALL be designed with portability in mind to facilitate porting to successor/next-generation chips, considering lifecycle support for components like NetX90 [Planned Features].

---

## 5. Technical Specifications

### 5.1 Hardware Architecture

The QNC Board is centered around the **i.MX 8M PLUS MCU** (4x Cortex-A53).

**Key Components** [2]:
*   **MCU**: i.MX 8M PLUS (Toradex module suggested for development).
*   **Memory**: LPDDR4 (1 GByte, 32-bit interface), eMMC 5.1 (16 GByte).
*   **Wireless**: WLAN-Modul (2.4 / 5 GHz, Bluetooth).
*   **Sensors**: IMU (Lagesensor).
*   **Display**: OLED Display (128x128 Pixel).
*   **Power Management**: PCA9450CHN PMIC, eFuse for 24 V protection.

### 5.2 Interface Definitions

#### 5.2.1 NeuraSync Communication Interfaces

The QNC system utilizes NeuraSync (in-house, FastDDS-based) for communication with the robot control system. The following interfaces are defined [1]:

| Interface ID | Name | Type | Direction | Protocol | Purpose |
|--------------|------|------|-----------|----------|---------|
| IF-001 | Generic Command | NeuraSync | Robot → QNC | NeuraSync | Send high-level commands to QNC |
| IF-002 | Generic Response | NeuraSync | QNC → Robot | NeuraSync | Receive responses to generic commands |
| IF-003 | Bridge Status | NeuraSync | QNC → Robot | NeuraSync | Report QNC operational status |
| IF-004 | Device Change Event | NeuraSync | QNC → Robot | NeuraSync | Notify of device connection/disconnection |
| IF-005 | ModbusRTU Write Command | NeuraSync | Robot → QNC | NeuraSync | Send direct ModbusRTU write commands |
| IF-006 | ModbusRTU Read Command | NeuraSync | Robot → QNC | NeuraSync | Send direct ModbusRTU read commands |
| IF-007 | ModbusRTU Response | NeuraSync | QNC → Robot | NeuraSync | Receive responses to ModbusRTU commands |
| IF-008 | ModbusRTU Statistics | NeuraSync | QNC → Robot | NeuraSync | Report ModbusRTU bridge statistics |

#### 5.2.2 Hardware Interfaces

The QNC system supports various hardware interfaces for communication with end-effectors [1]:

| Interface ID | Name | Purpose | Physical Layer | Protocols |
|--------------|------|---------|----------------|-----------|
| IF-009 | RS485 | ModbusRTU communication | TIA/EIA-485 | ModbusRTU |
| IF-010 | SPI/UART | IO-Link communication | SPI/UART | IO-Link |
| IF-011 | Hardware ID Pin | Resistor-based device identification | ADC | N/A |
| N/A | Ethernet (Future) | EtherCAT, EtherNet/IP | Ethernet | EtherCAT, EtherNet/IP |

**RS485 Interface Parameters** [1]:

| Parameter | Value |
|-----------|-------|
| Standard | TIA/EIA-485 |
| Topology | Multi-drop bus (up to 32 devices) |
| Cable | Twisted pair, shielded |
| Max Distance | 1200 meters |
| Termination | 120Ω at both ends |
| Baudrate | 9600, 19200, 38400, 57600, 115200 (Auto-detected or configured) |
| Data bits | 8 |
| Stop bits | 1 |
| Parity | None, Even, Odd (Device-dependent) |
| Flow control | None |

**SPI/UART Interface Parameters (IO-Link)** [1]:

| Parameter | COM1 | COM2 | COM3 |
|-----------|------|------|------|
| Baudrate | 4800 | 38400 | 230400 |
| Data bits | 8 | 8 | 8 |
| Parity | Even | Even | Even |
| Stop bits | 1 | 1 | 1 |

#### 5.2.3 Peripheral Interfaces

**Peripheral Interfaces** [2]:
*   **GPIO**: 4 outputs and 4 inputs at 3.3 V.
*   **Industrial Bus**: CAN-FD, RS485 (Half-duplex), RS232, EtherCAT.
*   **General Purpose**: USB-C (USB 3.0), Ethernet (1000 Base-TX), I2C, SPI.

#### 5.2.4 Configuration Interfaces

| Interface ID | Name | Purpose | Format | Location |
|--------------|------|---------|--------|----------|
| IF-012 | Device Descriptor | Define device-specific semantics | JSON | `/etc/qnc/devices/<device_name>.json` |
| N/A | QNC Configuration | Configure QNC operational parameters | YAML | `/etc/qnc/config.yaml` |

### 5.3 Data Structures

**Note**: Detailed data structure definitions are provided in the QNC Interface Control Document (QNC-ICD-001) Section 9. This section provides a summary of key structures for reference.

#### 5.3.1 `DeviceInfo` Structure

The `DeviceInfo` structure provides detailed information about a connected device. See ICD Section 9.1 for complete definition.

**Key Fields**: manufacturer, model, serial_number, firmware_version, protocol, descriptor_file, vendor_id, device_id

#### 5.3.2 `GenericCommand` Structure

The `GenericCommand` structure defines the generic command format sent from the robot to QNC. See ICD Section 9.2 for complete definition.

**Key Fields**: command_type, device_id, address, data, data_type, timeout_ms, tag, timestamp

#### 5.3.3 `GenericResponse` Structure

The `GenericResponse` structure defines the generic response format sent from QNC to the robot. See ICD Section 9.3 for complete definition.

**Key Fields**: error_code, error_message, data, data_type, latency_us, tag, timestamp

#### 5.3.4 Timestamp Format

All timestamps within the QNC system SHALL adhere to the ISO 8601 format: `YYYY-MM-DDTHH:MM:SS.sssZ` (e.g., `2026-02-18T10:30:45.123Z`) [1].

### 5.4 Communication Protocols

**NeuraSync**: In-house communication framework based on FastDDS, used for high-performance, scalable, and reliable communication between the robot control system and QNC [1].

**ModbusRTU**: A serial communication protocol commonly used in industrial automation for communication between devices over RS485 [1].

**IO-Link**: A standardized IO technology (IEC 61131-9) used for communicating with sensors and actuators, typically over SPI/UART [1].

**EtherCAT (Future)**: An Ethernet-based fieldbus system for real-time control applications [1].

**EtherNet/IP (Future)**: An industrial network protocol that adapts the Common Industrial Protocol (CIP) to standard Ethernet [1].

### 5.5 Device Detection Mechanisms

**Hardware ID Detection**: Utilizes a resistor-based identification system where different resistance values on a dedicated pin correspond to specific device types and their associated protocols [1].

**Protocol Probing**: Involves sending test messages across various supported protocols to identify the active protocol and device type [1].

**Descriptor-based Detection**: Relies on pre-configured JSON device descriptors to identify and configure devices [1].

### 5.6 Error Codes

**Requirement**: The QNC system SHALL define a comprehensive set of error codes categorized by Communication Errors, Command Errors, System Errors, and Update Errors.

**Note**: The complete error code reference table is maintained in QNC-ICD-001 Section 12. Key error codes for requirements validation:

**Critical Error Codes**:
- `0`: SUCCESS - Operation successful
- `-2`: TIMEOUT - Device/network timeout (NFR-014)
- `-3`: DISCONNECTED - Device not connected
- `-104`: DEVICE_DISCONNECT_DURING_MOTION - Critical: Device lost during motion (SHALL trigger E-stop recommendation)
- `-100`: POWER_FAILURE - Power supply out of range (NFR-013)
- `-200` to `-205`: Update-related errors (Section 5.9)

**Error Categories** (See ICD Section 12 for complete list):
- **Communication Errors** (-1 to -9, -103): Protocol, timeout, CRC, disconnection errors  - **Command Errors** (-4 to -7, -10): Invalid commands, addresses, permissions
- **System Errors** (-21 to -23, -100 to -104): Initialization, resources, power, mounting
- **Update Errors** (-200 to -205): Firmware update failures

**Error Handling Requirements** (NFR-013, NFR-018):
- All errors SHALL be logged with timestamp, error code, message, context, and recovery action
- Error recovery strategies SHALL include retries with exponential backoff, automatic reconnection, and validation
- Refer to ICD Section 12 for detailed error recovery matrix

### 5.7 System Handshaking

**Requirement**: QNC SHALL implement a comprehensive handshaking protocol to verify proper mounting, power, and readiness before normal operations.

**Note**: Detailed handshaking protocol specifications including message structures, timing, and sequence diagrams are provided in QNC-ICD-001 Section 13. This section defines the functional requirements.

**Handshake Requirements**:

1. **Power Supply Verification** (NFR-013):
   - SHALL verify 24V rail voltage within acceptable range (22.0V - 26.0V)
   - SHALL verify 5V rail voltage within acceptable range (4.75V - 5.25V)
   - Measurement accuracy: ±2%
   - Failure action: Report error (-100 POWER_FAILURE), fail handshake

2. **Mechanical Mount Verification**:
   - SHALL check connector engagement sensor
   - SHALL monitor vibration/acceleration (< 0.5g RMS during idle)
   - SHALL verify ground connection (< 1Ω resistance)
   - Failure action: Report mount failure (-101 MOUNT_FAILURE)

3. **Network Connection Verification** (NFR-015):
   - SHALL establish NeuraSync participant discovery
   - Timeout: 10 seconds (configurable 1000-10000ms per NFR-015)
   - SHALL confirm bidirectional communication with robot controller
   - Failure action: Report network error (-102 CONNECTION_TIMEOUT)

4. **System Information Exchange**:
   - SHALL send handshake request message containing: QNC serial number, firmware version, power status, mount status, self-test result
   - SHALL receive handshake acknowledgment from robot controller
   - Timeout: 5 seconds
   - SHALL exchange capability information

**Heartbeat Protocol** (NFR-011):
- After successful handshake, SHALL maintain continuous heartbeat at 1 Hz (configurable 0.5-10 Hz)
- SHALL include: sequence number, operational status, timestamp
- Heartbeat timeout: 3 consecutive missed beats trigger warning, 5 trigger safe state

**Failed Handshake Recovery**:
- SHALL enter safe state on handshake failure
- SHALL log failure reason with full context
- SHALL provide visual indicator (LED pattern) of failure state
- SHALL retry handshake after 5 seconds
- SHALL require manual intervention after 3 failed attempts

**Timing Requirements** (See ICD Section 13.7 for details):
- Power stabilization: ≤ 2000 ms
- Self-test: ≤ 5000 ms
- NeuraSync discovery: ≤ 10000 ms
- Total handshake: ≤ 30000 ms

**See**: QNC-ICD-001 Section 13 for complete handshake protocol specification, message structures (`HandshakeRequest`, `HandshakeAck`), and sequence diagrams.

### 5.8 Enhanced Error Handling and Recovery

**Requirement**: QNC SHALL implement robust error handling with specific actions for critical scenarios, particularly device disconnection during robot motion.

**Critical Error Scenario: Device Disconnection During Motion** [NEW]:

When a device is physically unplugged while the robot is moving, QNC SHALL:

1. **Immediate Detection** (< 100 ms):
   - Health check timeout on active command
   - No response within command timeout (default 500ms)
   - Physical disconnection detection (if hardware monitoring available)

2. **Emergency Response Actions**:
   - Publish `GenericResponse` with error code `-104` (DEVICE_DISCONNECT_DURING_MOTION)
   - Include motion state information in error message
   - **Recommend robot controller trigger E-stop**
   - Enter safe state immediately (reject new commands)
   - Log critical event with full context

3. **Robot Controller Actions** (RECOMMENDED):
   - **Trigger emergency stop (E-stop) immediately**
   - Halt all motion axes
   - Log event with timestamp and context
   - Alert operator via HMI/alarm system
   - Do not attempt end-effector commands until reconnected and verified

**Timeout Configuration** [NEW]:

| Timeout Type | Default | Range | Purpose |
|--------------|---------|-------|---------|
| Connection health check | 1000 ms | 500-5000 ms | Periodic device ping |
| Command timeout | 500 ms | 100-5000 ms | Per-command response wait |
| Disconnection detection | 3 checks | 2-5 checks | Consecutive failures to confirm disconnect |
| Reconnection attempt | 2000 ms | 1000-10000 ms | Retry interval |

**Recovery Sequence** [NEW]:

1. Robot confirms E-stop triggered
2. Operator reconnects device physically
3. QNC detects device reconnection (automatic monitoring)
4. QNC re-initializes device communication
5. QNC verifies device ready state
6. Robot performs homing/calibration sequence
7. Operator acknowledges ready to resume
8. Normal operations resume

**Error Recovery Strategies** [NEW]:

| Error Severity | Automatic Recovery | Operator Action Required | E-stop Trigger |
|----------------|-------------------|--------------------------|----------------|
| INFO | Yes | No | No |
| WARNING | Yes (retry) | No | No |
| ERROR | Yes (reconnect) | Monitor | No |
| CRITICAL | Partial (safe state) | Yes (acknowledge) | If during motion |
| FATAL | No | Yes (full intervention) | Yes |

**Safe State Definition** [NEW]:

When QNC enters safe state:
- Reject all new commands (return error)
- Maintain power to QNC
- Continue heartbeat transmission
- Log all attempted commands
- Publish safe state status to robot
- Require explicit exit command from robot to resume

### 5.9 Firmware Update Mechanism

**Requirement**: QNC SHALL support multiple firmware update mechanisms to accommodate different deployment scenarios and operational requirements.

**Supported Update Methods** [NEW]:

1. **OTA (Over-The-Air) Network Update** - PRIMARY:
   - Via Ethernet/WiFi network connection
   - Connect to update server (configurable URL)
   - Automatic update checking (configurable interval, default 24 hours)
   - Manual update checking via robot command
   - Use case: Remote fleet management, production updates

2. **USB Device Update** - SECONDARY:
   - Via USB-A port on QNC carrier board
   - Auto-detection of USB drive insertion
   - Search for firmware file pattern: `qnc-firmware-*.img`
   - Use case: Field updates, no network available

3. **NeuraSync Transfer Update** - TERTIARY:
   - Firmware transferred from robot controller via NeuraSync
   - Robot acts as update source
   - Useful when QNC isolated but robot has network
   - Use case: Robot-managed updates

4. **SD Card Update** - EMERGENCY:
   - Via microSD card slot
   - Manual insertion and reboot
   - Use case: Recovery, factory restore

**Update Process Flow** [NEW]:

1. **Check Phase**:
   - QNC queries update server or checks USB/SD
   - Retrieves update metadata (version, size, checksum, release notes)
   - Determines compatibility with current hardware
   - Notifies robot controller of available update

2. **Approval Phase**:
   - QNC publishes `UpdateAvailable` message to robot
   - Includes: current version, new version, size, criticality flag
   - Robot controller decides whether to proceed
   - Robot sends `UpdateControl` command (START/CANCEL)

3. **Download Phase**:
   - Download firmware package from source
   - Progress tracking (bytes downloaded, percentage)
   - Publish `UpdateProgress` messages to robot
   - Estimated time: 30s - 5min (depending on size and connection)

4. **Verification Phase**:
   - Compute SHA256 checksum of downloaded file
   - Compare with published checksum
   - Verify digital signature (if enabled)
   - Abort if verification fails

5. **Backup Phase** (if enabled):
   - Create backup of current firmware
   - Store in backup partition
   - Allows rollback if update fails

6. **Installation Phase**:
   - Extract update package
   - Write new firmware to boot partition
   - Update bootloader configuration
   - Update version information

7. **Reboot Phase**:
   - Notify robot of pending reboot
   - System reboots automatically
   - Boot with new firmware

8. **Verification Phase**:
   - Perform self-test with new firmware
   - Report new version to robot
   - Confirm successful update

**Update Configuration** [NEW]:

```yaml
qnc:
  updates:
    ota:
      enabled: true
      server_url: "https://updates.neura-sync.com/qnc"
      check_interval_hours: 24
      auto_install: false           # Require robot approval
      verify_signature: true
      backup_before_update: true
    usb:
      enabled: true
      mount_path: "/media/usb0"
      auto_detect: true
    security:
      require_signature: true
      allow_downgrade: false
```

**Update Security** [NEW]:

1. **Digital Signature Verification**:
   - All official updates signed with NeuraSync private key
   - QNC verifies using embedded public key
   - Prevents unauthorized firmware installation

2. **Checksum Verification**:
   - SHA256 hash validation
   - Ensures file integrity during transfer

3. **Version Compatibility Check**:
   - Hardware revision verification
   - Prevent installation of incompatible firmware

**Backup and Rollback** [NEW]:

*   **Automatic Backup**: Current firmware backed up before update (if enabled)
*   **Rollback Triggers**:
    *   Boot failure after update (automatic after 3 boot attempts)
    *   Self-test failure with new firmware
    *   Robot controller rollback command
    *   Manual intervention (button press during boot)
*   **Rollback Process**:
    *   Boot into recovery mode
    *   Restore previous firmware from backup partition
    *   Verify restored firmware integrity
    *   Boot with previous version
    *   Report rollback event to robot

**Update Timing** [NEW]:

| Update Type | Size | Download | Install | Total |
|-------------|------|----------|---------|-------|
| Patch | < 50 MB | 30s | 2 min | ~3 min |
| Minor | 50-200 MB | 2 min | 5 min | ~8 min |
| Major | > 200 MB | 5 min | 10 min | ~15 min |

*Assumes 10 Mbps network connection*

**Update Scheduling Recommendations** [NEW]:

- Schedule during maintenance windows
- Ensure robot is idle (not in production)
- Operator supervision required
- Avoid updates during shifts
- Allow time for post-update verification

**Update Error Codes** [NEW]:

- `-200`: Download failed → Retry download
- `-201`: Verification failed → Re-download
- `-202`: Installation failed → Rollback
- `-203`: Invalid signature → Reject update
- `-204`: Incompatible version → Contact support
- `-205`: Insufficient storage → Free space

### 5.10 Battery Pack Architecture

The battery module is split into an **electronics assembly** and a **battery pack** [2]:
*   **Cells**: Lithium-Ion (4.2 V cells).
*   **Management**: Balancer and protection electronics, temperature measurement on cells.
*   **Monitoring**: I2C-bus interface for monitoring state of charge, charging status, and temperatures.

### 5.11 Mechanical Design and Electrical Design Considerations

**Mechanical Design Vision**:
QNC mechanical integration SHOULD function as a soft connector concept (e.g., analogous to a rubber wristband), capable of adapting to different flange/tool shapes and dimensions while maintaining sufficient structural strength and rigidity for load-bearing operation.

**Electrical Design Vision**:
The electrical connection concept SHOULD maintain broad compatibility across commonly used industrial connector and port standards to preserve QNC's universality objective.

**Market Research Requirement**:
Before finalizing mechanical/electrical architecture, the team SHALL perform market research to identify existing materials and comparable concept products that can satisfy adaptability, rigidity, and durability requirements. Considering cost constraints, initial supplier and technology scouting SHOULD prioritize Chinese manufacturers.



---

## 6. Appendices

### Appendix A: Glossary

| Term | Definition |
|------|------------|
| **Bridge** | QNC firmware component that translates between protocols [1] |
| **NeuraSync** | In-house communication framework based on FastDDS [1] |
| **Descriptor** | JSON file containing device register maps and configuration [1] |
| **Hot-swap** | Ability to change devices without system restart [1] |
| **ICD** | Interface Control Document [1] |
| **Protocol Adapter** | Software module implementing specific protocol (ModbusRTU, IO-Link, etc.) [1] |
| **QNC** | Quick Network Connector - multi-protocol bridge system [1] |
| **Translation** | Converting commands from one protocol format to another [1] |

### Appendix B: Planned Features (Summary)

This section summarizes additional planned features not fully detailed in the ICD but identified as future enhancements or considerations:

*   **Battery Management**: Internal battery for USV-Battery Mode (10-15 min life), display for state of charge, and charging when connected to the robot flange.
*   **User Interface**: Internal screen or external application for control, including switching device modes, motions, and gripper functions. NFC or Bluetooth pairing for external apps.
*   **Advanced Capabilities**: Pose tracking, AI-driven object tracking (utilizing i.MX 8M PLUS MCU [2]), Camera D435i integration (start/stop capture, pose tracking).
*   **Future Considerations**: Camera passthrough, wireless communication, microphone integration via external app, Automotive Ethernet.
*   **Physical Design**: Target weight 0.5-1.0 kg, compact form factor (5 cm x 3 cm [2]), touchable surface temperature below 40 °C.
*   **Connectivity**: Standardized cable pin connector between Robot and QNC Flange (e.g., Neura 8Pin/12Pin, IO-Link 4 Pin M12), spring-loaded pins between QNC-Flange and QNC-Tool for short-circuit and twist-proof connection, full mounting solution of device manufacturer for QNC-Tool to actual Tool (e.g., M8 8Pin of OnRobot).
*   **Maintainability**: Consideration for component End-of-Life (EOL) and ease of firmware porting for future chip generations (e.g., NetX90 lifecycle support).

---

## References

[1]: QNC-ICD-001 v1.0 - QNC Interface Control Document  
[2]: N-QNCBoard-ConceptPlanningandRoadmap-130226-115752 - QNC Board Concept Planning and Roadmap (file:///home/ubuntu/upload/N-QNCBoard-ConceptPlanningandRoadmap-130226-115752.pdf)


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


---
---
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
