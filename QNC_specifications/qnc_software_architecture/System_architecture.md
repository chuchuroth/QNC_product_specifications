
# Interface Control

A robot arm flange-mounted device integrating the **ExternalDevice (ToolsV2) pipeline**, **FastDDS communication**, and **Hilscher netX90 protocol handling**.

---

# 1. System Architecture Overview

The system follows a **layered architecture** to ensure modularity between:

* High-level GUI
* Communication middleware
* Low-level hardware abstraction

## Architecture Layers

| Layer                | Component              | Responsibility                                                 |
| -------------------- | ---------------------- | -------------------------------------------------------------- |
| Application          | GUI / External App     | User interface for gripper control and state monitoring        |
| Middleware           | Xdevice (Control Unit) | Forwards FastDDS requests and manages device lifecycle         |
| Pipeline             | ToolsV2 Pipeline       | Maps GUI actions to ExternalDevice commands using JSON configs |
| Communication        | FastDDS                | High-performance publisher/subscriber for protocol requests    |
| Hardware Abstraction | Hilscher netX90        | Multi-protocol translation (RS485, Automotive Ethernet)        |
| Physical             | Power & Battery        | USV-battery mode and charging logic                            |

---

# 2. Codebase Directory Structure

```bash
/robot-flange-device
├── config/
│   ├── grippers/
│   │   ├── schunk_egp40.json
│   │   └── robotiq_2f85.json
│   └── network/
├── src/
│   ├── app/
│   │   ├── ui/
│   │   └── state_manager.py
│   ├── pipeline/
│   │   ├── tool_manager.cpp
│   │   └── json_parser.cpp
│   ├── communication/
│   │   ├── dds_publisher.cpp
│   │   ├── dds_subscriber.cpp
│   │   └── idl/
│   ├── drivers/
│   │   ├── hilscher/
│   │   ├── power/
│   │   └── sensors/
│   └── ai/
│       ├── vision_engine.py
│       └── llm_interface.py
├── firmware/
├── docs/
└── tests/
```

---

# 3. Key Component Specifications and Interfaces

## 3.1 GripperConfig (JSON) Structure

All gripper communication logic is encapsulated in JSON for **flexibility and dynamic integration**.

```json
{
  "device_name": "Gripper_A",
  "protocol": "ModbusRTU",
  "communication_params": {
    "interface": "RS485",
    "baudrate": 115200,
    "duplex": "half",
    "parity": "none",
    "stop_bits": 1
  },
  "registers": [
    {
      "name": "FingerPosition",
      "address": "0x0100",
      "type": "uint16",
      "access": "RW",
      "description": "Current position (0-255)"
    },
    {
      "name": "GripForce",
      "address": "0x0102",
      "type": "uint16",
      "access": "RW",
      "description": "Desired force (0-255)"
    },
    {
      "name": "Status",
      "address": "0x0104",
      "type": "uint8",
      "access": "R",
      "description": "Gripper status"
    }
  ],
  "functions": {
    "open": {
      "sequence": [
        {"register": "FingerPosition", "value": 0, "delay_ms": 50}
      ]
    },
    "close": {
      "sequence": [
        {"register": "FingerPosition", "value": 255, "delay_ms": 50}
      ]
    },
    "set_force": {
      "parameters": [
        {"name": "force_value", "type": "uint16", "min": 0, "max": 255}
      ],
      "sequence": [
        {"register": "GripForce", "value_from_param": "force_value", "delay_ms": 20}
      ]
    }
  }
}
```

---

## 3.2 FastDDS Protocol-Request Pipeline

### IDL Definition

```cpp
struct ProtocolRequest {
    string device_id;
    string command_name;
    sequence<octet> payload;
};
```

### Publisher Example

```cpp
void GripperPublisher::sendRequest(
    const std::string& gripper_id,
    const std::string& command_name,
    const std::vector<uint8_t>& payload) {

    ProtocolRequest request;
    request.device_id(gripper_id);
    request.command_name(command_name);
    request.payload(payload);

    publisher_->write(&request);
}
```

### Subscriber Logic

```cpp
void Xdevice_Subscriber::on_data_available(DataReader* reader) {
    ProtocolRequest request;

    // Deserialize request
    // Lookup gripper config
    // Translate to netX90 commands
    // Send via driver
}
```

---

## 3.3 Power & Thermal Management

### 3.3.1 Power Management

* Input power monitoring (robot flange)
* Battery charging management
* **USV mode (10–15 min runtime)**
* State of Charge (SoC) reporting

### 3.3.2 Thermal Management

* Integrated temperature sensors
* Surface temperature target: **< 40 °C**
* Thermal throttling for safety
* Reporting via `state_manager.py`

---

## 3.4 AI Features (Vision & Language)

### Vision Engine (`ai/vision_engine.py`)

* Camera interface
* Object detection & pose estimation
* Data output for control decisions

### Language Interface (`ai/llm_interface.py`)

* Speech-to-text / text-to-speech
* Natural Language Understanding (NLU)
* Context-aware interaction

---

## 3.5 ExternalDevice (ToolsV2) Pipeline

* **Tool Manager** (`tool_manager.cpp`)
  → Entry point for command execution

* **JSON Parser** (`json_parser.cpp`)
  → Loads and interprets gripper configs

* **Dynamic Register Mapping**
  → Maps GUI actions → hardware registers

---

## 3.6 Hilscher netX90 Integration

* Multi-protocol support:

  * Modbus RTU (RS485)
  * Profinet
  * EtherCAT

* RS485 half-duplex communication

* Automotive Ethernet for high-speed data

---

# 4. Hardware Integration Requirements

| Feature             | Implementation Detail     |
| ------------------- | ------------------------- |
| Weight              | 0.5 – 1.0 kg              |
| Dimensions          | Low-profile flange design |
| Connectivity        | Automotive Ethernet       |
| Display             | OLED or external app      |
| Surface Temperature | < 40 °C                   |

---

# 5. Planned Features Summary

* Power + charging via robot arm
* **USV battery mode (10–15 min)**
* Battery display (internal/external)
* Multi-gripper support via JSON configs
* GUI / app-based control
* AI integration (vision + language)
* Thermal safety management
* Lightweight compact design
* High-speed Ethernet communication





---
---
---

# Xdevice Multi-Protocol Gripper Device - Project Structure

## Overview
Robot arm flange-mounted device supporting multiple gripper communication protocols with battery backup, thermal management, and AI capabilities.

## Technology Stack
- **Communication**: FastDDS, Automotive Ethernet, RS485 (half-duplex)
- **Hardware Interface**: Hilscher netx90
- **Configuration**: JSON-based GripperConfig files
- **GUI Integration**: ExternalDevice (ToolsV2) Pipeline from 5.1.0 Release

## Directory Structure

```
Xdevice-gripper-device/
├── README.md
├── docs/
│   ├── architecture/
│   │   ├── system-overview.md
│   │   ├── communication-flow.md
│   │   ├── hardware-integration.md
│   │   └── thermal-management.md
│   ├── protocols/
│   │   ├── fastdds-integration.md
│   │   ├── rs485-communication.md
│   │   └── ethernet-specs.md
│   └── api/
│       └── gripper-config-schema.md
│
├── firmware/
│   ├── src/
│   │   ├── main.cpp
│   │   ├── core/
│   │   │   ├── device_manager.cpp
│   │   │   ├── power_controller.cpp
│   │   │   └── thermal_monitor.cpp
│   │   ├── communication/
│   │   │   ├── fastdds/
│   │   │   │   ├── dds_publisher.cpp
│   │   │   │   ├── dds_subscriber.cpp
│   │   │   │   └── protocol_request_handler.cpp
│   │   │   ├── rs485/
│   │   │   │   ├── rs485_driver.cpp
│   │   │   │   └── half_duplex_controller.cpp
│   │   │   ├── ethernet/
│   │   │   │   └── automotive_eth_interface.cpp
│   │   │   └── hilscher/
│   │   │       ├── netx90_interface.cpp
│   │   │       └── protocol_forwarder.cpp
│   │   ├── gripper/
│   │   │   ├── gripper_controller.cpp
│   │   │   ├── config_parser.cpp
│   │   │   ├── register_manager.cpp
│   │   │   └── protocol_adapter/
│   │   │       ├── protocol_base.cpp
│   │   │       ├── modbus_adapter.cpp
│   │   │       ├── profinet_adapter.cpp
│   │   │       ├── ethercat_adapter.cpp
│   │   │       └── custom_protocol_adapter.cpp
│   │   ├── power/
│   │   │   ├── battery_manager.cpp
│   │   │   ├── usv_controller.cpp
│   │   │   ├── charge_controller.cpp
│   │   │   └── power_state_machine.cpp
│   │   ├── display/
│   │   │   ├── screen_driver.cpp
│   │   │   ├── ui_renderer.cpp
│   │   │   └── status_display.cpp
│   │   ├── ai/
│   │   │   ├── vision/
│   │   │   │   ├── camera_interface.cpp
│   │   │   │   ├── object_detection.cpp
│   │   │   │   └── gripper_guidance.cpp
│   │   │   └── language/
│   │   │       ├── nlp_processor.cpp
│   │   │       └── command_interpreter.cpp
│   │   └── safety/
│   │       ├── temperature_limiter.cpp
│   │       ├── emergency_shutdown.cpp
│   │       └── diagnostics.cpp
│   ├── include/
│   │   └── [corresponding header files]
│   ├── CMakeLists.txt
│   └── platformio.ini
│
├── Xdevice-daemon/
│   ├── src/
│   │   ├── main.cpp
│   │   ├── Xdevice_server.cpp
│   │   ├── fastdds_receiver.cpp
│   │   ├── hilscher_forwarder.cpp
│   │   └── external_device_bridge.cpp
│   ├── include/
│   └── CMakeLists.txt
│
├── gui-integration/
│   ├── external_device_plugin/
│   │   ├── src/
│   │   │   ├── Xdevice_device_plugin.cpp
│   │   │   ├── device_configurator.cpp
│   │   │   └── gripper_control_panel.cpp
│   │   ├── ui/
│   │   │   ├── device_settings.qml
│   │   │   ├── gripper_manager.qml
│   │   │   └── battery_monitor.qml
│   │   └── CMakeLists.txt
│   └── integration_guide.md
│
├── gripper-configs/
│   ├── schema/
│   │   └── gripper_config_v1.schema.json
│   ├── examples/
│   │   ├── schunk_egu_modbus.json
│   │   ├── robotiq_2f_rs485.json
│   │   ├── zimmer_profinet.json
│   │   └── weiss_ethercat.json
│   └── README.md
│
├── mobile-app/
│   ├── android/
│   │   └── [Android Studio project]
│   ├── ios/
│   │   └── [Xcode project]
│   └── src/
│       ├── screens/
│       │   ├── DeviceControl.tsx
│       │   ├── GripperConfig.tsx
│       │   ├── BatteryStatus.tsx
│       │   └── AIFeatures.tsx
│       ├── services/
│       │   ├── bluetooth_service.ts
│       │   ├── device_api.ts
│       │   └── config_sync.ts
│       └── package.json
│
├── protocol-handlers/
│   ├── fastdds/
│   │   ├── idl/
│   │   │   ├── ProtocolRequest.idl
│   │   │   ├── GripperCommand.idl
│   │   │   └── DeviceStatus.idl
│   │   ├── generated/
│   │   └── CMakeLists.txt
│   └── hilscher/
│       ├── netx90_drivers/
│       └── protocol_converters/
│
├── hardware/
│   ├── schematics/
│   ├── pcb/
│   ├── cad/
│   │   ├── mechanical_design.step
│   │   └── flange_adapter.stl
│   ├── thermal/
│   │   ├── thermal_simulation.xlsx
│   │   └── heatsink_design.pdf
│   └── bom/
│       └── components_list.csv
│
├── tests/
│   ├── unit/
│   │   ├── test_gripper_controller.cpp
│   │   ├── test_battery_manager.cpp
│   │   ├── test_protocol_adapters.cpp
│   │   └── test_config_parser.cpp
│   ├── integration/
│   │   ├── test_fastdds_communication.cpp
│   │   ├── test_Xdevice_integration.cpp
│   │   └── test_hilscher_forwarding.cpp
│   ├── hardware/
│   │   ├── test_rs485_loopback.cpp
│   │   ├── test_thermal_limits.cpp
│   │   └── test_battery_cycles.cpp
│   └── fixtures/
│       └── test_gripper_configs/
│
├── tools/
│   ├── config_validator/
│   │   └── validate_gripper_config.py
│   ├── protocol_simulator/
│   │   └── simulate_gripper.py
│   ├── calibration/
│   │   └── thermal_calibration.py
│   └── deployment/
│       ├── flash_firmware.sh
│       └── update_configs.sh
│
├── dependencies/
│   ├── fastdds/
│   ├── hilscher-sdk/
│   └── external_libs.txt
│
└── build/
    ├── firmware/
    ├── Xdevice-daemon/
    └── gui-plugin/
```

## Key Components Description

### 1. Firmware (Embedded Device)
Main application running on the device hardware, responsible for:
- Power management (battery/charging)
- Protocol handling
- Thermal monitoring
- Display control
- AI processing (vision/language)

### 2. Xdevice Daemon
Linux service running on Xdevice that:
- Receives FastDDS protocol requests
- Forwards to Hilscher netx90
- Manages device state
- Integrates with ExternalDevice pipeline

### 3. GUI Integration
Plugin for the robot control GUI (5.1.0+):
- Device configuration interface
- Gripper selection and control
- Battery monitoring
- Status visualization

### 4. Gripper Configs
JSON-based configuration files containing:
- Register mappings
- Command functions
- Protocol-specific parameters
- Gripper capabilities

### 5. Protocol Handlers
- FastDDS publishers/subscribers
- RS485 half-duplex drivers
- Automotive Ethernet stack
- Hilscher netx90 integration

### 6. Mobile App (Optional)
For external control:
- Bluetooth/WiFi connectivity
- Gripper control interface
- Battery status
- Configuration management

## Communication Flow

```
GUI (ToolsV2)
    ↓ [ExternalDevice Pipeline]
Xdevice Daemon (FastDDS Publisher)
    ↓ [Protocol Request]
Device (FastDDS Subscriber)
    ↓ [Parse GripperConfig]
Gripper Controller
    ↓ [Protocol Adapter]
Hilscher netx90
    ↓ [RS485 Half-Duplex]
Physical Gripper
```

## Build System
- CMake for C++ components
- PlatformIO for firmware
- Gradle/Maven for Android app
- npm/yarn for any web components

## Configuration Management
All gripper-specific logic lives in JSON configs:
- Register addresses
- Function codes
- Command sequences
- Protocol parameters
- Timing specifications

This allows adding new grippers without firmware changes.
