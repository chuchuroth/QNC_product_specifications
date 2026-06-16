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
## qnc_protocol_core Overview

`qnc_protocol_core` is the multi-protocol abstraction layer that sits above the raw `modbus_rtu` header library. Here's what each piece does and its current state:

---

## What it compiles (`src`)

| Source file                     | What it implements                                                                 | Status  |
|--------------------------------|-------------------------------------------------------------------------------------|---------|
| `modbus_rtu_adapter.cpp`       | `ModbusRTUAdapter : IProtocol` — wraps `modbus_rtu` behind the common `IProtocol` interface (read/write/connect/capabilities) | Active  |
| `modbus_device_discovery.cpp`  | `DeviceDiscovery` — probes serial ports across baudrates/slave IDs to auto-detect DH-Robotics, Robotiq, and generic Modbus devices | Active  |
| `unified_device_discovery.cpp` | `UnifiedDeviceDiscovery` — wraps `DeviceDiscovery` + adds voltage sensing, RS485/IO-Link detection, and handshaking | Active  |
| `qnc_system_manager.cpp`       | `QncSystemManager` — top-level orchestrator: system handshake, error handling, firmware update, owns both discovery objects | Active  |

---

## Stubbed (not yet implemented)

These adapters are declared in CMake but not yet written:

```cpp
// src/iolink_adapter.cpp          // TODO: Implement
// src/protocol_detector.cpp       // TODO: Implement
// src/protocol_manager.cpp        // TODO: Implement
// src/protocol_translator.cpp     // TODO: Implement
// src/unified_protocol_bridge.cpp // TODO: Implement
