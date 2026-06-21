# QNC Gripper Communication Device

A robot arm flange-mounted communication device for controlling industrial grippers via Modbus RTU and IO-Link protocols, integrated with NeuraSync dashboard.

## Current Development Phase

**Platform**: Raspberry Pi Compute Module 5 (CM5) + In-house Communication Module
**Target**: RT-Labs U-Phy Chip Module01 (future migration)
**Integration**: NeuraSync Dashboard via FastDDS

## Features

### Active Features (Current Phase)
- ✅ **NeuraSync Integration**: FastDDS-based communication with NeuraSync dashboard
- ✅ **Modbus RTU Support**: RS485-based gripper control (Schunk, Robotiq, etc.)
- ✅ **IO-Link Support**: IO-Link master for intelligent devices
- ✅ **Custom Communication Module**: In-house designed PCB with J3 (IO-Link/CQ) and J4 (RS485)
- ✅ **Dual Device Support**: Control up to 2 robot hands simultaneously
- ✅ **JSON Configuration**: Flexible gripper configuration without code changes
- ✅ **Real-time Feedback**: Bidirectional communication with grippers

### Out of Scope (Current Phase)
- ❌ Battery backup and USV functionality
- ❌ Thermal management
- ❌ AI capabilities (vision, NLP)
- ❌ Display and UI components
- ❌ PROFINET and EtherCAT protocols
- ❌ Mobile app integration

## Architecture

```
NeuraSync Dashboard (FastDDS)
         ↓
    CM5 Application
         ↓
  Communication Module
    ↙         ↘
  J3          J4
(IO-Link)   (Modbus RTU)
   ↓            ↓
Robot Hand   Robot Hand
```

## Technology Stack

- **Hardware**: Raspberry Pi Compute Module 5 + Custom Communication Module
- **Communication**: NeuraSync (FastDDS), Modbus RTU, IO-Link
- **Protocols**: RS485 half-duplex, IO-Link Master, CQ (optional)
- **Configuration**: JSON-based GripperConfig files
- **Build System**: CMake for ARM64
- **Integration**: NeuraSync Dashboard (`git@gitlab.hrg.systems:adv_dev/innovation/neurasync-dashboard.git`)

## Quick Start

### Prerequisites

```bash
# Install dependencies on CM5
sudo apt-get update
sudo apt-get install cmake build-essential git

# Install FastDDS
sudo apt-get install fastdds libfastdds-dev

# Install Modbus library
sudo apt-get install libmodbus-dev

# Clone NeuraSync (if needed for development)
git clone git@gitlab.hrg.systems:adv_dev/innovation/neurasync-dashboard.git neurasync-integration/neurasync
```

### Build CM5 Application

```bash
cd rpi-cm5
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
```

### Configure Gripper

```bash
# Copy gripper config
sudo mkdir -p /etc/qnc/grippers
sudo cp gripper-configs/examples/schunk_egu_modbus.json /etc/qnc/grippers/gripper_right.json

# Validate config
python3 tools/config_validator/validate_gripper_config.py /etc/qnc/grippers/gripper_right.json
```

### Run Application

```bash
cd rpi-cm5/build
sudo ./qnc-cm5-app --config /etc/qnc/grippers/

# With debug logging
sudo ./qnc-cm5-app --config /etc/qnc/grippers/ --log-level debug
```

## Documentation

### Architecture
- [Project Structure](PROJECT_STRUCTURE.md) - Updated directory structure
- [NeuraSync Integration](docs/architecture/neurasync-integration.md) - FastDDS integration details
- [Data Flow Logic](docs/architecture/data-flow-logic.md) - Complete data flow documentation
- [Communication Module](docs/hardware/communication-module.md) - Hardware specifications

### Protocols
- [Modbus RTU Implementation](docs/protocols/modbus-rtu-implementation.md)
- [IO-Link Implementation](docs/protocols/iolink-implementation.md)
- [NeuraSync FastDDS Protocol](docs/protocols/neurasync-fastdds.md)

### Hardware
- [Raspberry Pi CM5 Setup](docs/hardware/raspberry-pi-cm5-setup.md)
- [Communication Module Specs](docs/hardware/communication-module.md)
- [Migration to Module01](docs/hardware/migration-to-module01.md)

## Hardware Setup

### Current Development Platform
- **Compute Module**: Raspberry Pi Compute Module 5
- **Carrier Board**: Custom or compatible CM5 carrier
- **Communication Module**: In-house designed PCB
  - **J3**: IO-Link / CQ protocol (Robot Hand 1)
  - **J4**: RS485 / Modbus RTU (Robot Hand 2)
- **Power**: 24V external supply (separate power for each hand recommended)
- **Data Interface**: Ethernet/SPI/UART between CM5 and communication module

### Connections
```
CM5 ←→ Communication Module (DATA interface)
                ↓
        ┌───────┴───────┐
        ↓               ↓
    J3 (IO-Link)    J4 (RS485)
        ↓               ↓
    Robot Hand 1    Robot Hand 2
```

⚠️ **Important**: Use separate 24V power cables for each robot hand. The communication module PCB cannot handle power distribution for both hands.

## Supported Grippers

### Modbus RTU (via J4)
- Schunk EGU series
- Robotiq 2F-85/140
- OnRobot grippers with Modbus support
- Any Modbus RTU compatible gripper

### IO-Link (via J3)
- IO-Link compatible grippers
- Configurable via ISDU parameters

## Testing

### Unit Tests
```bash
cd rpi-cm5/build
./test_modbus_handler
./test_iolink_handler
./test_neurasync_integration
```

### Hardware Tests
```bash
# Modbus loopback test
sudo ./test_rs485_loopback

# NeuraSync communication test
python3 tools/neurasync_tools/neurasync_message_tester.py
```

### Monitor DDS Traffic
```bash
python3 tools/neurasync_tools/monitor_dds_traffic.py
```

## NeuraSync Integration

The device integrates with NeuraSync dashboard via FastDDS:

1. **Subscribe to Commands**: Receives `GripperCommand` messages from NeuraSync
2. **Execute Commands**: Translates to Modbus RTU or IO-Link and sends to gripper
3. **Publish Status**: Sends `GripperStatus` feedback to NeuraSync
4. **Error Reporting**: Reports timeouts, faults, and configuration errors

See [NeuraSync Integration Guide](docs/architecture/neurasync-integration.md) for details.

## Migration Path

1. **Current**: Develop and test on Raspberry Pi CM5
2. **Validate**: Test all protocols and NeuraSync integration
3. **Port**: Migrate code to RT-Labs Module01 U-Phy Chip
4. **Deploy**: Production deployment on Module01

## License

[Specify your license here]

## Contributing

Please read CONTRIBUTING.md for details on our code of conduct and the process for submitting pull requests.

## Contact

For support and inquiries, please contact [your contact information].
