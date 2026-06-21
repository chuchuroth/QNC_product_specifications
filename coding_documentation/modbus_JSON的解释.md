

---
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
---
---
# ModbusRTU JSON Descriptor Explained

This document expands on the previous analysis of the `ModbusRTU.idl` file, focusing on the `ModbusRTU_DeviceDescriptor_Schema.json` file. This JSON schema is the key to the system's flexibility, allowing it to support a wide variety of Modbus devices without changing the core IDL.

## 1. The Role of the JSON Descriptor

As mentioned in the previous document, the `DeviceDescriptor` IDL struct contains a `json_descriptor` string field. This field holds a JSON object that conforms to the `ModbusRTU_DeviceDescriptor_Schema.json` schema. The robot platform parses this JSON to understand how to interpret the raw Modbus data it receives from the QNC.

## 2. JSON Schema Structure

The JSON schema is organized into several sections, each defining a different aspect of the device's configuration and capabilities. The following diagram provides a high-level overview of the schema's structure:

![JSON Schema Structure](https://private-us-east-1.manuscdn.com/sessionFile/WPf6tchysqjZFY3vDViqpM/sandbox/gtmLh35240byoPU5DeN52W-images_1769183027347_na1fn_L2hvbWUvdWJ1bnR1L2pzb25fc2NoZW1hX3N0cnVjdHVyZQ.png?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvV1BmNnRjaHlzcWpaRlkzdkRWaXFwTS9zYW5kYm94L2d0bUxoMzUyNDBieW9QVTVEZU41MlctaW1hZ2VzXzE3NjkxODMwMjczNDdfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwycHpiMjVmYzJOb1pXMWhYM04wY25WamRIVnlaUS5wbmciLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE3OTg3NjE2MDB9fX1dfQ__&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=V6-QH5HWu8z31eIts94BjPUGhOMZJ5UXKTqJ88rWUwP~Kl1ZLh67gPbfCr2ubjkY6i4pGAfFfsrClLJF7bYmVndM91J-iEkVe4lAD2a4Kxke31c9m7t9M~rJ14YsNAXn3SuUsqlHtHyZT0tLw9ZsPVMy7T-cWrmU4RZcF~~0gkZZ1-pFzKYgSbTuKHqPr-S7QiA8zx8FIcn1541krsxJWRXrOHS7ekMiDEv-0lJgdhv0GPeCpo4mCzItHLpebXCY16qe52f~LGBn8YsPpZrkuXZY1WB~LNmzrVsbr0N5MrQVUSgbYVg1COMN5eTmVphLjlizhEJUCJlrotWPcZwHOQ__)

### Key Sections

*   **`device`**: Contains metadata about the device, such as its type, manufacturer, and model.
*   **`communication`**: Defines the default serial communication settings, such as baud rate and slave ID.
*   **`registers`**: This is the core of the schema, where the Modbus registers and coils are mapped to human-readable names and data types.
*   **`computed_fields`**: Allows for the creation of new data fields by performing calculations on one or more register values.
*   **`commands`**: Defines high-level commands that the robot can execute, which are translated by the QNC into a sequence of register writes.
*   **`state_machine`**: An optional section that can be used to define a state machine for the device.
*   **`capabilities`**: A flexible section for defining the device's capabilities, such as maximum force or speed.
*   **`polling`**: Provides recommendations for how frequently the QNC should poll the device for its status.

## 3. IDL and JSON Integration

The IDL and the JSON schema work together to provide a complete solution for Modbus communication. The IDL defines the communication protocol and the high-level data structures, while the JSON schema provides the device-specific details that are needed to interpret the data.

The following diagram illustrates how the IDL and JSON schema are integrated:

![IDL and JSON Integration](https://private-us-east-1.manuscdn.com/sessionFile/WPf6tchysqjZFY3vDViqpM/sandbox/gtmLh35240byoPU5DeN52W-images_1769183027348_na1fn_L2hvbWUvdWJ1bnR1L2lkbF9qc29uX2ludGVncmF0aW9u.png?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvV1BmNnRjaHlzcWpaRlkzdkRWaXFwTS9zYW5kYm94L2d0bUxoMzUyNDBieW9QVTVEZU41MlctaW1hZ2VzXzE3NjkxODMwMjczNDhfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwybGtiRjlxYzI5dVgybHVkR1ZuY21GMGFXOXUucG5nIiwiQ29uZGl0aW9uIjp7IkRhdGVMZXNzVGhhbiI6eyJBV1M6RXBvY2hUaW1lIjoxNzk4NzYxNjAwfX19XX0_&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=fVMApD3RiKYnBtlwDV5IQW3yL2wW~vm-u9C3RMO8ka-2Jmm5g6L0G4rg4xK4WMGwKI4VFAhD10cgZ6vCygvr1qA3Sb24cc6nNwe52ewGdEzqAQMzPtAhjXYzr-Bg7XJLwfMpyBIlGXFYQhSpWQtds1MheXDprgmRnnWamxBW60LX62ZOLWMm5b7N8l~pZTYALV05PcdpyApmrf67q2uewleaul7b55J3WURLq2Wq1hrb~kQx6XiE95ZLxs2ivXjRSMC4nBAKtk0hRozewqmRFHQafTWAw7cLtXGDFRV32YZ0Ut-gy3hd848ZkQtbSUx3bHGdKBUspmFOxcXLB0HNsQ__)

As the diagram shows, the `DeviceDescriptor` struct from the IDL carries the JSON payload. The robot platform parses this JSON to understand the structure of the `DeviceState` message and to build `DeviceCommand` and `NamedCommand` messages.

## 4. Register Mapping and Data Transformation

The `registers` section of the JSON schema is where the magic happens. It allows you to define how raw register values are transformed into meaningful data. This includes scaling, offsets, enumerations, and bit fields.

The diagram below shows how a raw register value is transformed into a semantic output using the information from the JSON schema:

![Register Mapping and Data Transformation](https://private-us-east-1.manuscdn.com/sessionFile/WPf6tchysqjZFY3vDViqpM/sandbox/gtmLh35240byoPU5DeN52W-images_1769183027349_na1fn_L2hvbWUvdWJ1bnR1L3JlZ2lzdGVyX21hcHBpbmdfZmxvdw.png?Policy=eyJTdGF0ZW1lbnQiOlt7IlJlc291cmNlIjoiaHR0cHM6Ly9wcml2YXRlLXVzLWVhc3QtMS5tYW51c2Nkbi5jb20vc2Vzc2lvbkZpbGUvV1BmNnRjaHlzcWpaRlkzdkRWaXFwTS9zYW5kYm94L2d0bUxoMzUyNDBieW9QVTVEZU41MlctaW1hZ2VzXzE3NjkxODMwMjczNDlfbmExZm5fTDJodmJXVXZkV0oxYm5SMUwzSmxaMmx6ZEdWeVgyMWhjSEJwYm1kZlpteHZkdy5wbmciLCJDb25kaXRpb24iOnsiRGF0ZUxlc3NUaGFuIjp7IkFXUzpFcG9jaFRpbWUiOjE3OTg3NjE2MDB9fX1dfQ__&Key-Pair-Id=K2HSFNDJXOU9YS&Signature=pORUJhYaH88HO8YUQkyfIEK0~4S1e48iUCEVwV5EbWczI5IbyGhKOOVy97RKHnw5hDXUnW6krZzJ0z~2qCzPnnJpfE-sLaQfjm8UpyBF9AE3VRTVMfv5njwwZ8KGIEG-7DvqVEjsNMLrXe-VkpAoIkd0IBXnclJ1qFa2svNGSwCiXkO28a1jJYMpeNsspBTDsXucegBmvbPWIExwOFEN5x~JFKewA9z9vzqYFFADjBTlGbHIYRtQb7RAVT597P6mIlztLdV4jWTdQPkCub3C9lVPQnUGsuFlTU8d6t3ysex2v48fY00H2jCKu1sLx~2ZYBYzFgtk1q8X0UbCFcxmHA__)

### JSON Snippets

Here are some snippets from the JSON schema that illustrate how these transformations are defined.

**Register Definition with Scaling:**

```json
{
  "address": 259,
  "name": "target_position",
  "type": "uint16",
  "unit": "permille",
  "scale": 0.001
}
```

**Register Definition with Enumeration:**

```json
{
  "address": 512,
  "name": "init_state",
  "type": "uint16",
  "enum": {
    "not_init": 0,
    "initializing": 1,
    "ready": 2
  }
}
```

**High-Level Command Definition:**

```json
"commands": {
  "grip": {
    "description": "Close the gripper to a specified position and force.",
    "parameters": [
      { "name": "position", "type": "float", "unit": "mm" },
      { "name": "force", "type": "float", "unit": "N" }
    ],
    "sequence": [
      { "action": "write_register", "register": "target_force", "value": "${force}" },
      { "action": "write_register", "register": "target_position", "value": "${position}" },
      { "action": "wait_for_state", "state_register": "grip_state", "expected_value": 3, "timeout_ms": 2000 }
    ]
  }
}
```



## 5. Conclusion

The JSON device descriptor is a powerful mechanism that provides a high degree of flexibility to the ModbusRTU communication system. By separating the device-specific logic from the core communication protocol, it allows the system to be easily adapted to new devices without requiring any changes to the IDL or the robot platform's code.
