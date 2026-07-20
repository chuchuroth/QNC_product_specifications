
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
