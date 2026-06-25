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
