
## Concrete Example: Robot Sends "Close Gripper" Command

### ✅ CORRECT: Using ModbusRTUBridge.idl (Pure Bridge)

```
┌─────────────────────────────────────────────────────────┐
│ ROBOT SIDE (Has Device Intelligence)                   │
│─────────────────────────────────────────────────────────│
│ 1. Robot loads device descriptor:                      │
│    dh_robotics_gripper.json                            │
│                                                         │
│ 2. High-level command:                                 │
│    close_gripper(position=50mm, force=80%)             │
│                                                         │
│ 3. Robot translates using descriptor knowledge:        │
│    - position 50mm → register 259 = 0x01F4 (500)       │
│    - force 80%    → register 257 = 0x0050 (80)         │
│                                                         │
│ 4. Robot publishes DDS messages:                       │
│    WriteCommand{slave=1, reg=257, value=0x0050}        │
│    WriteCommand{slave=1, reg=259, value=0x01F4}        │
└─────────────────────────────────────────────────────────┘
                         ↓
            NeuraSync (DDS Transport)
         qnc::modbus::WriteCommand
                         ↓
┌─────────────────────────────────────────────────────────┐
│ QNC BRIDGE (Device Agnostic)                           │
│─────────────────────────────────────────────────────────│
│ 5. Receives: WriteCommand{slave=1, reg=257, val=80}    │
│    Executes: Modbus function 0x06 (write register)     │
│    Returns: Response{error_code=0}                     │
│                                                         │
│ 6. Receives: WriteCommand{slave=1, reg=259, val=500}   │
│    Executes: Modbus function 0x06 (write register)     │
│    Returns: Response{error_code=0}                     │
└─────────────────────────────────────────────────────────┘
                         ↓
                  RS485 / Modbus RTU
                         ↓
┌─────────────────────────────────────────────────────────┐
│ PHYSICAL DEVICE (DH Gripper)                           │
└─────────────────────────────────────────────────────────┘
```

**Key Benefits:**
- ✅ QNC has ZERO knowledge of grippers
- ✅ Same QNC works with ANY ModbusRTU device (motors, sensors, IO modules)
- ✅ Robot side handles all device-specific logic
- ✅ Device descriptors live where they belong (robot platform)

### ❌ WRONG: Using rich ModbusRTU.idl (Violates Pure Bridge)

```
┌─────────────────────────────────────────────────────────┐
│ ROBOT SIDE                                              │
│─────────────────────────────────────────────────────────│
│ 1. High-level command:                                 │
│    close_gripper(position=50mm, force=80%)             │
│                                                         │
│ 2. Robot publishes:                                    │
│    GripperCommand{                                     │
│       action=CLOSE,                                    │
│       position_mm=50,                                  │
│       force_percent=80                                 │
│    }                                                   │
└─────────────────────────────────────────────────────────┘
                         ↓
            NeuraSync (DDS Transport)
         ModbusRTU::GripperCommand  ← Device-specific!
                         ↓
┌─────────────────────────────────────────────────────────┐
│ QNC BRIDGE (Now has device knowledge!)  ❌             │
│─────────────────────────────────────────────────────────│
│ 3. Receives: GripperCommand{action=CLOSE, ...}         │
│                                                         │
│ 4. QNC must know gripper semantics:  ← WRONG!         │
│    - CLOSE → write reg 259 = (position_mm * 10)        │
│    - force_percent → write reg 257 = force_percent     │
│                                                         │
│ 5. What if you add a motor? Now QNC needs motor logic! │
│ 6. What if you add IO module? QNC needs more logic!    │
│                                                         │
│ Result: QNC becomes device-specific mess  ❌           │
└─────────────────────────────────────────────────────────┘
```

## Answer: **ModbusRTUBridge.idl is Rich Enough AND Generic**

Looking at your current ModbusRTUBridge.idl:

```idl
// ✅ Complete protocol coverage, zero device semantics
struct ReadCommand { uint8 slave_id; uint16 start_register; uint16 count; }
struct WriteCommand { uint8 slave_id; uint16 register_address; uint16 value; }
struct WriteMultipleCommand { uint8 slave_id; uint16 start_register; sequence<uint16> values; }
struct Response { uint8 slave_id; sequence<uint16> data; int32 error_code; ... }
struct BridgeStats { uint64 tx_count; uint64 rx_count; ... }
```

**This is the PERFECT level of "richness":**
- ✅ Rich in **protocol operations** (read, write, write-multiple, responses, stats)
- ✅ Generic in **device semantics** (no GripperCommand, MotorCommand, etc.)
- ✅ Supports ALL ModbusRTU devices (grippers, motors, sensors, IO modules)

## Verification: Your Device Descriptors Prove This Works

Your existing device_descriptors_external/ show exactly how robots should use this:

```python
# Robot side code (not in QNC!)
descriptor = load_json("dh_robotics_gripper.json")

# Translate high-level command using descriptor
def close_gripper(force_percent):
    force_reg = descriptor['registers']['control']['force']['address']  # 257
    
    # Publish generic command via ModbusRTUBridge.idl
    publish(WriteCommand(slave_id=1, register=force_reg, value=force_percent))
```

## Final Answer

**ModbusRTUBridge.idl is correct** because:

1. **Rich enough:** Covers all Modbus protocol operations needed
2. **Generic enough:** Zero device-specific types (no GripperCommand, MotorState, etc.)
3. **Scalable:** Works with ANY ModbusRTU device without changing QNC
4. **Separation of concerns:** Device intelligence lives in robot platform with JSON descriptors

**ModbusRTU.idl violates architecture** because:
- Contains device-specific types (GripperCommand, DeviceState with init_state/gripper_state fields)
- Forces QNC to know about grippers, motors, etc.
- Breaks when you add new device types

**Your current architecture is perfect!** Keep ModbusRTUBridge.idl. The "richness" comes from covering the protocol completely, NOT from adding device semantics.


---
---

# ModbusRTU.idl Explained: A Visual Guide

This document provides a comprehensive overview of the `ModbusRTU.idl` file, a NeuraSync Interface Definition Language (IDL) file designed for general-purpose Modbus RTU communication. We will explore its architecture, communication patterns, and data structures, supplemented by visual diagrams to illustrate the key concepts.

## 1. Introduction

The `ModbusRTU.idl` file defines the data contract for communication between a robotic platform and an external device interface, known as the QNC (QNC-Nano-Controller). The design philosophy is to enable support for any Modbus RTU device without requiring modifications to the IDL itself. This is achieved by having the QNC publish raw Modbus data in a device-agnostic format, while the robot platform uses a JSON device descriptor to parse and interpret that data.

## 2. Data Flow Architecture

The system consists of three main components: the Robot Platform, the QNC (acting as a Modbus gateway), and the External Modbus RTU Device. The Robot Platform communicates with the QNC using the NeuraSync protocol, and the QNC, in turn, communicates with the external device using the Modbus RTU protocol.

The following diagram illustrates the high-level architecture and the primary message types exchanged between the components:

![Data Flow Architecture](https://private-us-east-1.manuscdn.com/sessionFile/WPf6tchysqjZFY3vDViqpM/sandbox/dITm0JN5m3g6xyWI1cAsY0-images_1769180823619_na1fn_L2hvbWUvdWJ1bnR1L21vZGJ1c3J0dV9hcmNoaXRlY3R1cmU.png?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvV1BmNnRjaHlzcWpaRlkzdkRWaXFwTS9zYW5kYm94L2RJVG0wSk41bTNnNnh5V0kxY0FzWTAtaW1hZ2VzXzE3NjkxODA4MjM2MTlfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwyMXZaR0oxYzNKMGRWOWhjbU5vYVhSbFkzUjFjbVUucG5nIiwiQ29uZGl0aW9uIjp7IkRhdGVMZXNzVGhhbiI6eyJBV1M6RXBvY2hUaW1lIjoxNzk4NzYxNjAwfX19XX0_&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=p4NEmfG~-ZHUkKMg2D26ojc0SHKgoS-xKsDDP94rK7u760-0RjksC4R8sFkbcR5IwD8j1WXSiWEQaujXhlCxH4eKqAlmhBE9egnQgp3cunBR2pJAF-5Ht2QUKHfOqGIPNrsmcf~xr7i~zcJ801Wq39k3t65ro4r0f4iGQlPOauh1m3eEMcfoyxqJ0yAhZXddJM9Mtn~UVdbv3-fIN9kIMXRiYgzpzAs9LDL7EQuSl8~x06ogRX3ew~SgTVn2Y35SSVhXtK8LEgb0s8AUb-CP6jm-ptmSNFP1dL6ipzsB8-jrbeT~cX9jtdNZZtSa8bkLJ7tgBTiTgNiIWxrARSdAWQ__)

As shown, the Robot Platform can send commands and configuration to the QNC, which then translates them into Modbus RTU commands for the device. The QNC periodically sends the device's state back to the Robot Platform and provides responses to on-demand commands.

## 3. Communication Sequence

The interaction between the components follows a well-defined sequence, which can be broken down into several phases:

*   **Connection Phase:** The Robot Platform initiates the connection by sending a `DeviceConfig` message to the QNC. The QNC establishes a connection with the device and, upon success, sends a `DeviceDescriptor` to the robot. This descriptor contains a JSON schema that the robot uses to interpret the raw Modbus data.
*   **Periodic Polling:** The QNC periodically polls the device for its state based on the configuration provided by the robot. The raw data is then sent to the robot in a `DeviceState` message.
*   **On-Demand Commands:** The robot can send specific read or write commands to the device via the QNC using a `DeviceCommand` message. The QNC executes the command and returns the result in a `TransactionResponse`.
*   **High-Level Commands:** For more complex devices, the robot can use `NamedCommand` messages, which are translated by the QNC into a sequence of register writes based on the JSON descriptor.

The sequence diagram below visualizes this communication flow:

![Communication Sequence](https://private-us-east-1.manuscdn.com/sessionFile/WPf6tchysqjZFY3vDViqpM/sandbox/dITm0JN5m3g6xyWI1cAsY0-images_1769180823620_na1fn_L2hvbWUvdWJ1bnR1L21vZGJ1c3J0dV9zZXF1ZW5jZQ.png?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvV1BmNnRjaHlzcWpaRlkzdkRWaXFwTS9zYW5kYm94L2RJVG0wSk41bTNnNnh5V0kxY0FzWTAtaW1hZ2VzXzE3NjkxODA4MjM2MjBfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwyMXZaR0oxYzNKMGRWOXpaWEYxWlc1alpRLnBuZyIsIkNvbmRpdGlvbiI6eyJEYXRlTGVzc1RoYW4iOnsiQVdTOkVwb2NoVGltZSI6MTc5ODc2MTYwMH19fV19&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=SSM3yauKmElMPB7tNrUq-rCoxwhI7vew5Sm320XMWilFu6KZyhUnonjHodr5EozWCVW2OP8MXM5VucEMI~xgp9lr5faOFv1rAq3vSWvukqoDOeX9Iu03DzHMmziEnwFiSgmor811WA~zIwBvf4K-BTP3TjAQykHNCaPPQkI7H4wr~5Mxp0GW8g-87GM14XT4iIVEAcQPKrxQQ0XW5XORCsnOScnw45zN459qpaTcUA0a8hluO~~D-PFm4gl4zyjvIC0o8QPDRg1V~Pao5eVnWpiM2FM27Z2Wy8JZxZuxYqVjF3tmLU7q7nbBeLdTe-HrXmI9l5rjzNDYoAi5HKzsrg__)

## 4. Data Structures

The IDL defines a rich set of data structures to represent the state, commands, and configuration of the Modbus device. These structures are organized hierarchically, with high-level messages composed of more fundamental data types.

The diagram below shows the relationships between the main data structures defined in the IDL:

![Data Structures](https://private-us-east-1.manuscdn.com/sessionFile/WPf6tchysqjZFY3vDViqpM/sandbox/dITm0JN5m3g6xyWI1cAsY0-images_1769180823620_na1fn_L2hvbWUvdWJ1bnR1L21vZGJ1c3J0dV9zdHJ1Y3R1cmVz.png?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvV1BmNnRjaHlzcWpaRlkzdkRWaXFwTS9zYW5kYm94L2RJVG0wSk41bTNnNnh5V0kxY0FzWTAtaW1hZ2VzXzE3NjkxODA4MjM2MjBfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwyMXZaR0oxYzNKMGRWOXpkSEoxWTNSMWNtVnoucG5nIiwiQ29uZGl0aW9uIjp7IkRhdGVMZXNzVGhhbiI6eyJBV1M6RXBvY2hUaW1lIjoxNzk4NzYxNjAwfX19XX0_&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=rYZAK6bIx8j6003SPB4zM1egGeOMhxe6xu0viZ3Yu1VnEPKFOdUMxdEchcPoMS9TGqNPZ~kGj41tAf9VKAwQp-EqarhwFuOXGuWahR45CvBEX5sEGI7vvGg-W~~UTuv7HLOdAYlqqyaREc0B-pXgsAykN7zlSzZdUM0rL8vNmWN9LWE9igRrOUnqXehiaT76tVQdRMsyIL5r3grBVp1ky20tyZSm-WU0YdfIyWWgC5mr1-ThT37LWmNhZVlQ4cz252O3zsUPg7BjI824QaR9JsP2qNIahM5HUbzxrNu1XzXmRmvemrJ-7gf9RNAxLVtp8CMc5l4KQEwnoc3zrfMQ0g__)

### Key Data Structures

Here are some of the most important data structures and their purpose:

| Structure             | Description                                                                                             | Fields                                                                                                                              |
| --------------------- | ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| `DeviceDescriptor`    | Contains the JSON schema that describes how to interpret the device's data.                               | `device`, `timestamp`, `json_descriptor`                                                                                            |
| `DeviceState`         | Represents the complete state of the device, including raw register and coil values.                      | `device`, `timestamp`, `connection_state`, `holding_registers`, `input_registers`, `coils`, `discrete_inputs`, `sequence_num`, `error_message` |
| `DeviceCommand`       | A generic command message for performing Modbus read/write operations.                                    | `device`, `timestamp`, `command_id`, `function_code`, and various command data fields.                                              |
| `NamedCommand`        | A high-level command that allows the robot to use semantic names instead of raw register operations.      | `device`, `timestamp`, `command_id`, `command_name`, `params`                                                                       |
| `TransactionResponse` | The response to a `DeviceCommand` or `NamedCommand`, indicating the status and any data read from the device. | `device`, `timestamp`, `command_id`, `status`, `register_data`, `coil_data`, `error_message`                                        |
| `DeviceConfig`        | Used to configure the QNC's polling behavior and serial port settings.                                    | `device`, `timestamp`, serial port settings, `register_polls`, `coil_polls`, and timeouts.                                          |

### Code Snippets

To provide a more concrete understanding, here are some snippets from the IDL file that define some of these key structures.

**Enumerations for Function Codes and Status:**

```idl
enum FunctionCode {
    FC_READ_COILS,              // 0x01
    FC_READ_DISCRETE_INPUTS,    // 0x02
    FC_READ_HOLDING_REGISTERS,  // 0x03
    FC_READ_INPUT_REGISTERS,    // 0x04
    FC_WRITE_SINGLE_COIL,       // 0x05
    FC_WRITE_SINGLE_REGISTER,   // 0x06
    FC_WRITE_MULTIPLE_COILS,    // 0x0F
    FC_WRITE_MULTIPLE_REGISTERS // 0x10
};

enum TransactionStatus {
    TX_SUCCESS,
    TX_PENDING,
    TX_ERROR_ILLEGAL_FUNCTION,
    TX_ERROR_ILLEGAL_ADDRESS,
    TX_ERROR_ILLEGAL_VALUE,
    TX_ERROR_DEVICE_FAILURE,
    TX_ERROR_TIMEOUT,
    TX_ERROR_CRC,
    TX_ERROR_CONNECTION
};
```

**DeviceState Structure:**

```idl
struct DeviceState {
    DeviceInfo device;
    Timestamp timestamp;
    ConnectionState connection_state;

    // Raw register data (as read from device)
    sequence<RegisterBlock, 8> holding_registers;   // Function code 0x03
    sequence<RegisterBlock, 8> input_registers;     // Function code 0x04

    // Raw coil/discrete input data
    sequence<CoilBlock, 4> coils;                   // Function code 0x01
    sequence<CoilBlock, 4> discrete_inputs;         // Function code 0x02

    // Sequence number for detecting missed messages
    unsigned long sequence_num;

    // Error information (if connection_state == ERROR)
    string<MAX_ERROR_MSG_LEN> error_message;
};
```

**DeviceCommand Structure:**

```idl
struct DeviceCommand {
    DeviceInfo device;              // Target device
    Timestamp timestamp;
    unsigned long command_id;       // Unique command ID for tracking
    FunctionCode function_code;     // Modbus function to execute

    // Command data (use appropriate field based on function_code)
    WriteRegisterCmd write_register;      // For FC_WRITE_SINGLE_REGISTER
    WriteRegistersCmd write_registers;    // For FC_WRITE_MULTIPLE_REGISTERS
    WriteCoilCmd write_coil;              // For FC_WRITE_SINGLE_COIL
    WriteCoilsCmd write_coils;            // For FC_WRITE_MULTIPLE_COILS
    ReadRegisterCmd read_registers;       // For FC_READ_HOLDING_REGISTERS, FC_READ_INPUT_REGISTERS
    ReadCoilCmd read_coils;               // For FC_READ_COILS, FC_READ_DISCRETE_INPUTS
};
```

## 5. Conclusion

The `ModbusRTU.idl` file provides a flexible and powerful framework for integrating Modbus RTU devices into a robotic system. By decoupling the raw data transmission from its interpretation, it allows for a high degree of adaptability and reuse. The use of a JSON-based device descriptor is a key feature that enables this flexibility, allowing the system to support a wide range of devices without changing the underlying communication protocol.

This document has provided a visual and descriptive overview of the IDL's architecture, communication patterns, and data structures. The diagrams and code snippets should serve as a useful reference for anyone working with this interface.


---
---
---
# ModbusRTU IDL Integration for QNC

## Overview

This document describes the general-purpose ModbusRTU IDL (Interface Definition Language) for NeuraSync communication between robotic platforms and QNC (Quantum Negative Clutch). The QNC device serves as a communication interface between robotic platforms and external devices supporting various protocols, with ModbusRTU being the primary focus.

### Design Philosophy

The ModbusRTU IDL follows a **device-agnostic** design:

1. **QNC publishes raw Modbus data** - Register/coil values without interpretation
2. **Robot platform interprets data** - Using JSON device descriptors
3. **Zero code changes for new devices** - Only a JSON descriptor file is needed
4. **Self-describing devices** - Device descriptors contain all parsing information

### Data Flow

```
Robot Platform <──NeuraSync DDS──> QNC <──ModbusRTU──> External Device
       │                            │
       │ DeviceCommand              │ Raw Modbus frames
       │ NamedCommand               │ (FC 0x03, 0x06, etc.)
       │ DeviceConfig               │
       │                            │
       │ DeviceState                │
       │ TransactionResponse        │
       │ DeviceDescriptor           │
       │ QNCHeartbeat               │
       ▼                            ▼
```

---

## Architecture

### Component Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                      Robot Platform                              │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │  JSON Parser (uses device descriptor to interpret data)  │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                              │
                    NeuraSync DDS Topics
                              │
    ┌─────────────────────────┼─────────────────────────┐
    │                         │                         │
    ▼                         ▼                         ▼
qnc/modbus/           qnc/modbus/              qnc/modbus/
device_command        named_command            device_config
    │                         │                         │
    └─────────────────────────┼─────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                       QNC Device                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   ModbusRTUBridge                        │    │
│  │  ┌────────────────┐  ┌────────────────┐  ┌───────────┐  │    │
│  │  │ Subscriber     │  │ Publisher      │  │ Device    │  │    │
│  │  │ (commands)     │  │ (state/resp)   │  │ Manager   │  │    │
│  │  └────────────────┘  └────────────────┘  └───────────┘  │    │
│  │                              │                           │    │
│  │              ┌───────────────┴───────────────┐          │    │
│  │              │    TransactionHandler         │          │    │
│  │              │    (Modbus RTU execution)     │          │    │
│  │              └───────────────────────────────┘          │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                              │
                        ModbusRTU
                      (RS-485 Serial)
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    External Device                               │
│              (Gripper, Sensor, Actuator, etc.)                  │
└─────────────────────────────────────────────────────────────────┘
```

### DDS Topics

| Topic | Direction | Message Type | Purpose |
|-------|-----------|--------------|---------|
| `qnc/modbus/device_command` | Robot → QNC | `DeviceCommand` | Raw Modbus read/write commands |
| `qnc/modbus/named_command` | Robot → QNC | `NamedCommand` | High-level semantic commands |
| `qnc/modbus/device_config` | Robot → QNC | `DeviceConfig` | Serial port and polling configuration |
| `qnc/modbus/device_state` | QNC → Robot | `DeviceState` | Periodic raw register/coil values |
| `qnc/modbus/transaction_response` | QNC → Robot | `TransactionResponse` | Response to commands |
| `qnc/modbus/device_descriptor` | QNC → Robot | `DeviceDescriptor` | Device self-description (JSON) |
| `qnc/modbus/heartbeat` | QNC → Robot | `QNCHeartbeat` | Connection status |

---

## IDL Message Types

### File Location

```
# In the separate neura-sync repository:
neura_sync_core_msgs/idl/ModbusRTU.idl
```

> **Note:** neura-sync is maintained as a separate repository:
> `git@gitlab.hrg.systems:adv_dev/robot_communications/neura-sync.git`

### Constants

```idl
const unsigned short MAX_REGISTERS = 125;      // Max registers per transaction
const unsigned short MAX_COILS = 2000;         // Max coils per transaction
const unsigned short MAX_DEVICE_NAME_LEN = 64;
const unsigned short MAX_ERROR_MSG_LEN = 128;
const unsigned short MAX_DESCRIPTOR_LEN = 8192; // JSON descriptor max length
```

### Enumerations

#### FunctionCode
Standard Modbus function codes:
```idl
enum FunctionCode {
    FC_READ_COILS,              // 0x01
    FC_READ_DISCRETE_INPUTS,    // 0x02
    FC_READ_HOLDING_REGISTERS,  // 0x03
    FC_READ_INPUT_REGISTERS,    // 0x04
    FC_WRITE_SINGLE_COIL,       // 0x05
    FC_WRITE_SINGLE_REGISTER,   // 0x06
    FC_WRITE_MULTIPLE_COILS,    // 0x0F
    FC_WRITE_MULTIPLE_REGISTERS // 0x10
};
```

#### ConnectionState
Device connection states:
```idl
enum ConnectionState {
    DISCONNECTED,    // Device not connected
    CONNECTING,      // Connection in progress
    CONNECTED,       // Device connected and responding
    ERROR,           // Communication error
    TIMEOUT          // Device not responding
};
```

#### TransactionStatus
Modbus transaction results:
```idl
enum TransactionStatus {
    TX_SUCCESS,                 // Operation completed successfully
    TX_PENDING,                 // Operation in progress
    TX_ERROR_ILLEGAL_FUNCTION,  // Modbus exception 0x01
    TX_ERROR_ILLEGAL_ADDRESS,   // Modbus exception 0x02
    TX_ERROR_ILLEGAL_VALUE,     // Modbus exception 0x03
    TX_ERROR_DEVICE_FAILURE,    // Modbus exception 0x04
    TX_ERROR_TIMEOUT,           // No response from device
    TX_ERROR_CRC,               // CRC mismatch
    TX_ERROR_CONNECTION         // Serial/connection error
};
```

### Core Message Types

#### DeviceInfo
Device identification used in all messages:
```idl
struct DeviceInfo {
    string<64> device_id;      // Unique identifier (e.g., "gripper_01")
    string<64> device_type;    // Device type (e.g., "DH_AG95")
    string<64> manufacturer;   // Manufacturer name
    string<64> model;          // Model number
    octet slave_id;            // Modbus slave address (1-247)
};
```

#### DeviceDescriptor
Self-describing device with JSON schema:
```idl
struct DeviceDescriptor {
    DeviceInfo device;
    Timestamp timestamp;
    string<8192> json_descriptor;  // JSON with register map, commands, capabilities
};
```

#### DeviceState
Periodic raw register/coil data from QNC:
```idl
struct DeviceState {
    DeviceInfo device;
    Timestamp timestamp;
    ConnectionState connection_state;
    sequence<RegisterBlock, 8> holding_registers;   // FC 0x03
    sequence<RegisterBlock, 8> input_registers;     // FC 0x04
    sequence<CoilBlock, 4> coils;                   // FC 0x01
    sequence<CoilBlock, 4> discrete_inputs;         // FC 0x02
    unsigned long sequence_num;
    string<128> error_message;
};
```

#### DeviceCommand
Raw Modbus command from robot platform:
```idl
struct DeviceCommand {
    DeviceInfo device;
    Timestamp timestamp;
    unsigned long command_id;
    FunctionCode function_code;

    WriteRegisterCmd write_register;      // For FC 0x06
    WriteRegistersCmd write_registers;    // For FC 0x10
    WriteCoilCmd write_coil;              // For FC 0x05
    WriteCoilsCmd write_coils;            // For FC 0x0F
    ReadRegisterCmd read_registers;       // For FC 0x03, 0x04
    ReadCoilCmd read_coils;               // For FC 0x01, 0x02
};
```

#### NamedCommand
High-level semantic command:
```idl
struct NamedCommand {
    DeviceInfo device;
    Timestamp timestamp;
    unsigned long command_id;
    string<32> command_name;              // e.g., "grip", "release", "move_to"
    sequence<CommandParam, 8> params;     // e.g., force=80, speed=50
};
```

#### TransactionResponse
Response after command execution:
```idl
struct TransactionResponse {
    DeviceInfo device;
    Timestamp timestamp;
    unsigned long command_id;
    TransactionStatus status;
    sequence<unsigned short, 125> register_data;  // For read operations
    sequence<boolean, 2000> coil_data;
    string<128> error_message;
};
```

---

## JSON Device Descriptor Schema

### File Location

```
schemas/ModbusRTU_DeviceDescriptor_Schema.json
```

### Schema Structure

```json
{
  "schema_version": "1.0",

  "device": {
    "device_type": "DH_AG95",
    "manufacturer": "DH-Robotics",
    "model": "AG-95"
  },

  "communication": {
    "default_baudrate": 115200,
    "default_slave_id": 1,
    "parity": "none",
    "response_timeout_ms": 100
  },

  "registers": {
    "holding": [
      {
        "address": 256,
        "name": "init_command",
        "type": "uint16",
        "access": "wo",
        "enum": { "init": 1, "full_init": 165 }
      },
      {
        "address": 259,
        "name": "target_position",
        "type": "uint16",
        "unit": "permille",
        "scale": 1.0,
        "min": 0,
        "max": 1000
      }
    ],
    "input": [
      {
        "address": 512,
        "name": "init_state",
        "type": "uint16",
        "enum": { "not_initialized": 0, "initializing": 1, "initialized": 2 }
      }
    ]
  },

  "computed_fields": [
    {
      "name": "position_mm",
      "type": "float",
      "unit": "mm",
      "formula": "(1000 - current_position) * 0.095"
    }
  ],

  "commands": {
    "grip": {
      "parameters": [
        { "name": "force", "type": "float", "unit": "percent", "min": 20, "max": 100 }
      ],
      "sequence": [
        { "action": "write_register", "register": "force_percent", "value": "${force}" },
        { "action": "write_register", "register": "target_position", "value": 1000 }
      ]
    }
  },

  "capabilities": {
    "max_force_n": 140,
    "stroke_mm": 95,
    "has_position_feedback": true
  }
}
```

### Example Device Descriptor

See: `gripper-configs/examples/DH_AG95_descriptor.json`

---

## QNC Integration

### Repository Structure

The project is organized into separate repositories:

| Repository | Location | Description |
|------------|----------|-------------|
| **qnc** | `git@gitlab.hrg.systems:adv_dev/innovation/qnc.git` | Main QNC application |
| **neura-sync** | `git@gitlab.hrg.systems:adv_dev/robot_communications/neura-sync.git` | NeuraSync DDS middleware (separate repo) |
| **neurasync-dashboard** | `git@gitlab.hrg.systems:adv_dev/innovation/neurasync-dashboard.git` | Dashboard application (separate repo) |

### Files Created/Modified

**In qnc repository:**

| File | Type | Description |
|------|------|-------------|
| `schemas/ModbusRTU_DeviceDescriptor_Schema.json` | Created | JSON Schema for device descriptors |
| `gripper-configs/examples/DH_AG95_descriptor.json` | Created | Example device descriptor |
| `rpi-cm5/include/modbus_rtu_bridge.hpp` | Created | Bridge header with all classes |
| `rpi-cm5/src/modbus_rtu/modbus_rtu_bridge.cpp` | Created | Bridge implementation |
| `rpi-cm5/tests/test_modbus_rtu_bridge.cpp` | Created | Test application |
| `rpi-cm5/CMakeLists.txt` | Modified | Added ModbusRTU bridge to build |

**In neura-sync repository:**

| File | Type | Description |
|------|------|-------------|
| `neura_sync_core_msgs/idl/ModbusRTU.idl` | Created | Main IDL definition (~480 lines) |
| `neura_sync_core_msgs/CMakeLists.txt` | Modified | Added ModbusRTU.idl to build |

### Bridge Classes

#### ModbusRTUPublisher
Publishes messages from QNC to robot platform:
- `publishDeviceState()` - Periodic device state
- `publishTransactionResponse()` - Command responses
- `publishDeviceDescriptor()` - Device self-description
- `publishHeartbeat()` - Connection status

#### ModbusRTUSubscriber
Receives commands from robot platform:
- `registerCommandCallback()` - For `DeviceCommand` messages
- `registerNamedCommandCallback()` - For `NamedCommand` messages
- `registerConfigCallback()` - For `DeviceConfig` messages

#### ModbusRTUDeviceManager
Manages connected devices and their descriptors:
- `loadDeviceDescriptor()` - Load JSON descriptor from file
- `getDescriptor()` - Get descriptor by device ID
- `setDeviceState()` / `getDeviceState()` - Connection state management
- `buildHeartbeat()` - Generate heartbeat message

#### ModbusRTUTransactionHandler
Executes Modbus transactions:
- `executeCommand()` - Execute raw Modbus command
- `executeNamedCommand()` - Execute named command using descriptor
- `readRegisters()` / `writeRegister()` - Low-level Modbus operations

#### ModbusRTUBridge
Main integration class:
- `initialize()` - Initialize DDS and serial connection
- `start()` / `stop()` - Control bridge operation
- Automatically handles command routing and state publishing

---

## Build Instructions

### Prerequisites

- CMake 3.16+
- C++17 compiler
- FastDDS 3.x (optional, uses mock if not available)
- libmodbus (optional, uses mock if not available)

### Clone Repositories

```bash
# Clone all repositories into the same workspace
mkdir -p ~/workspace && cd ~/workspace

git clone git@gitlab.hrg.systems:adv_dev/innovation/qnc.git
git clone git@gitlab.hrg.systems:adv_dev/robot_communications/neura-sync.git
git clone git@gitlab.hrg.systems:adv_dev/innovation/neurasync-dashboard.git
```

### Build neura_sync_core_msgs

```bash
cd ~/workspace/neura-sync
mkdir -p build && cd build

cmake .. \
  -DCMAKE_PREFIX_PATH="/path/to/fastdds/install" \
  -DCMAKE_INSTALL_PREFIX="~/workspace/local_install"

make -j$(nproc)
make install
```

This generates C++ code from `ModbusRTU.idl`:
- `ModbusRTU.hpp` - Type definitions
- `ModbusRTUPubSubTypes.cxx` - Serialization
- `ModbusRTUTypeObjectSupport.cxx` - Type registration

### Build rpi-cm5 Application

```bash
cd ~/workspace/qnc/rpi-cm5
mkdir -p build && cd build

cmake .. \
  -DCMAKE_PREFIX_PATH="/path/to/fastdds/install;~/workspace/local_install" \
  -DCMAKE_BUILD_TYPE=Release

make -j$(nproc)
```

### Build Configuration

The build system supports two modes:

| Mode | FastDDS | libmodbus | Use Case |
|------|---------|-----------|----------|
| Real | Found | Found | Production deployment |
| Mock | Not found | Not found | Development/testing |

Preprocessor definitions:
- `USE_REAL_FASTDDS` - Use real FastDDS implementation
- `USE_REAL_MODBUS` - Use real libmodbus implementation

---

## Usage Examples

### Running the Test Application

```bash
cd rpi-cm5/build

# Run with defaults
./test_modbus_rtu_bridge

# Run with custom options
./test_modbus_rtu_bridge \
  --domain 0 \
  --config ../../gripper-configs/examples \
  --port /dev/ttyUSB0
```

### Robot Platform: Sending Commands

```cpp
// Low-level command (write single register)
ModbusRTU::DeviceCommand cmd;
cmd.device().device_id("gripper_01");
cmd.device().slave_id(1);
cmd.command_id(1001);
cmd.function_code(ModbusRTU::FunctionCode::FC_WRITE_SINGLE_REGISTER);
cmd.write_register().address(259);  // target_position
cmd.write_register().value(1000);   // fully closed

publisher.publish(cmd);
```

```cpp
// High-level command (named command)
ModbusRTU::NamedCommand cmd;
cmd.device().device_id("gripper_01");
cmd.command_id(2001);
cmd.command_name("grip");

ModbusRTU::CommandParam force_param;
force_param.name("force");
force_param.value(80.0);
cmd.params().push_back(force_param);

publisher.publish(cmd);
```

### Robot Platform: Receiving State

```cpp
// Subscribe to device state
subscriber.registerCallback([](const ModbusRTU::DeviceState& state) {
    std::cout << "Device: " << state.device().device_id() << std::endl;
    std::cout << "Connection: " << (int)state.connection_state() << std::endl;

    // Parse using device descriptor
    for (const auto& block : state.input_registers()) {
        std::cout << "Registers at " << block.start_address() << ": ";
        for (auto val : block.values()) {
            std::cout << val << " ";
        }
        std::cout << std::endl;
    }
});
```

### QNC: Handling Commands

```cpp
ModbusRTUBridge bridge;
bridge.initialize(0, "/path/to/configs");

// Load device descriptors
bridge.getDeviceManager().loadDeviceDescriptor(
    "DH_AG95_descriptor.json", "gripper_01");

// Start processing
bridge.start();

// Commands are automatically handled via callbacks
// Responses are automatically published
```

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

## Comparison with Universal Gripper Library

| Aspect | Universal Gripper Library | ModbusRTU.idl |
|--------|---------------------------|---------------|
| New Device Support | New C++ class + recompile | JSON descriptor only |
| Architecture | Device-specific classes | Device-agnostic |
| IDL Complexity | ~200 lines (minimal) | ~480 lines (comprehensive) |
| Flexibility | Low | High |
| Maintenance | High (per-device code) | Low (JSON changes) |

### Key Advantages

1. **Zero-code device support** - New devices need only a JSON file
2. **Self-describing** - Device descriptors contain all information
3. **Separation of concerns** - QNC handles protocol, robot handles semantics
4. **Future-proof** - Works with unknown device types

---

## File References

**neura-sync repository:**
- IDL Definition: `neura_sync_core_msgs/idl/ModbusRTU.idl`

**qnc repository:**
- JSON Schema: `schemas/ModbusRTU_DeviceDescriptor_Schema.json`
- Example Descriptor: `gripper-configs/examples/DH_AG95_descriptor.json`
- Bridge Header: `rpi-cm5/include/modbus_rtu_bridge.hpp`
- Bridge Implementation: `rpi-cm5/src/modbus_rtu/modbus_rtu_bridge.cpp`
- Test Application: `rpi-cm5/tests/test_modbus_rtu_bridge.cpp`

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-01-26 | Initial implementation |

---

## Authors

- ModbusRTU IDL Design and Implementation
- QNC Integration
