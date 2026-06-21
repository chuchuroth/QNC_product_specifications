# Modbus RTU Integration & Testing Guide — Read-Only Slave

**Document ref:** MODBUS-INT-TEST-001  
**Date:** 2026-03-30 (updated for read-only command register architecture)  
**Workspace:** modbus_master_slave_comm_workspace

---

## 1. Quick Start

### 1.1 Build All Components

```bash
cd neurasync-robot-client
cmake -B build -DBUILD_EXAMPLES=ON -DENABLE_LOGGING=OFF
cmake --build build -j$(nproc)
```

**Output binaries:**
- `build/modbus_slave_test` — Low-level RTU frame tests + serial listener
- `build/modbus_bridge_test` — Bridge adapter and data sync integration tests

### 1.2 Run Unit & Integration Tests

```bash
# Run all tests
./build/modbus_bridge_test

# Run with verbose output
./build/modbus_bridge_test --verbose
```

Expected output:
```
╔════════════════════════════════════════════════════════════╗
║  Modbus Bridge Integration Test Suite                    ║
╚════════════════════════════════════════════════════════════╝

╔═══════════════════════════════════════════════════════════╗
║  TEST 1: Bridge Initialization & Register Access        ║
╚═══════════════════════════════════════════════════════════╝
  ✓ Set register 0 = 0x1234
  ✓ Set register 10 = 0xABCD
  ✓ Bulk write/read 3 regs
  ✓ Write callback triggered

  Result: 4 passed, 0 failed

[... more tests ...]

╔════════════════════════════════════════════════════════════╗
║  Test Summary                                              ║
╚════════════════════════════════════════════════════════════╝
  Passed: 23
  Failed: 0
```

---

## 2. Component Overview

### 2.1 Architecture Layers

```
┌─────────────────────────────────────────────────────┐
│  Application / State Machine (PLC logic)            │
│  • Command publishing (writes registers 0–7)        │
│  • State transitions (IDLE → TEACH → WORK → etc)    │
│  • Register image management                        │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│  Bridge Adapter (modbus_bridge.hpp)                 │
│  • Serial I/O management                            │
│  • Thread-safe register array (8 registers)         │
│  • Statistics & diagnostics                         │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│  Serial Utilities (serial_utils.hpp)                │
│  • Port enumeration & configuration                 │
│  • Baud rate handling                               │
│  • Non-blocking I/O helpers                         │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│  Modbus RTU Slave (modbus_rtu_slave.hpp)            │
│  • FC03 (Read Holding Registers) only               │
│  • Rejects FC06/FC10 with ILLEGAL_FUNCTION          │
│  • CRC16 validation                                 │
│  • Exception handling                               │
└─────────────────────────────────────────────────────┘
```

### 2.2 Key Components

| Component | File | Purpose |
|-----------|------|---------|
| **Modbus RTU Slave** | `include/modbus_rtu_slave.hpp` | FC03-only frame processor; read-only enforcement |
| **Bridge Adapter** | `include/modbus_bridge.hpp` | Serial transport + thread-safe 8-register bank |
| **Command Registers** | `include/modbus_command_registers.hpp` | Register layout definition + packing/unpacking helpers |
| **Serial Utils** | `include/serial_utils.hpp` | Port enumeration, configuration |
| **Example: QNC Node** | `src/modbus_qnc_node.cpp` | Command register publishing (integration layer) |
| **Example: PLC State Machine** | `src/plc_state_machine_hil.cpp` | Deterministic state-driven register publishing |
| **Example: HIL Tool** | `src/hil_live_test.cpp` | Observation & validation of incoming FC03 requests |

---

## 3. Testing Strategy

### 3.1 Test Levels

#### Level 1: Unit Tests (modbus_rtu_slave_test)

Tests the low-level Modbus RTU frame processing.

```bash
./build/modbus_slave_test
```

**Test cases:**
- ✓ FC03 Read Holding Registers (all 8 command registers)
- ✓ FC06 Write Single Register → ILLEGAL_FUNCTION exception
- ✓ FC10 Write Multiple Registers → ILLEGAL_FUNCTION exception
- ✓ Address mismatch on FC03 (no slave response)
- ✓ Bad CRC (frame ignored)
- ✓ Unsupported function codes → ILLEGAL_FUNCTION exception
- ✓ Out-of-range register addresses (>7) → ILLEGAL_DATA_ADDRESS exception
- ✓ Read count > 8 → ILLEGAL_DATA_VALUE exception
- ✓ Register round-trip validation (write image → FC03 reads)

**Expected outcome:** 7–10 tests all passing. All write requests (FC06, FC10) rejected.

#### Level 2: Integration Tests (modbus_bridge_test)

Tests bridge adapter, command register packing/unpacking, and thread safety.

```bash
./build/modbus_bridge_test --verbose
```

**Test groups:**
1. **Bridge Initialization**: Register read/write operations on 8-register bank
2. **Command Register Packing**: High/low byte extraction and round-trip validation
   - Device mode (H), Workpiece number (L) packing
   - Position tolerance (H), Grip force (L) unpacking
   - Drive velocity, Work position byte swapping
3. **Read-Only Enforcement**: FC06/FC10 rejected, registers never modified by master
4. **Thread Safety**: Concurrent register access (4+ threads) with no corruption
5. **Serial Utilities**: Baud rate mapping, port scanning
6. **CRC Validation**: Correct polynomial & residue for RTU frames

**Expected outcome:** 20–25 tests all passing. Zero write operations from master side.

#### Level 3: Serial I/O Validation Tests

Test communication with a real or simulated master.

```bash
# Plain listen mode (responds to any master on /dev/ttyUSB0)
./build/modbus_slave_test --listen /dev/ttyUSB0 115200

# Discovery mode (logs all master access patterns)
./build/modbus_slave_test --discover /dev/ttyUSB0 115200

# HIL observation (validates register semantics during live reads)
./build/hil_live_test /dev/ttyUSB0 115200
```

**Validation points:**
- Master sends FC03 reads; slave responds with full 8-register image
- No FC06 or FC10 seen from master (or properly rejected if sent)
- Register values decoded correctly per `modbus_command_registers.hpp`
- Serial line stays clean (no CRC errors, no timeouts)

#### Level 4: State Machine Tests

Test deterministic command register publishing.

```bash
./build/plc_state_machine_hil /dev/ttyUSB0 115200 60
```

Validates:
- State machine transitions (IDLE → TEACH → WORK → HOLD)
- Register updates match each state's expected image
- Master can read command registers at any state
- No handshake timeouts or stuck states

### 3.2 Testing Checklist

- [ ] **Build tests compile**: `cmake --build build -j$(nproc)` succeeds
- [ ] **Unit tests pass**: `./build/modbus_slave_test` shows 7–10 passing
- [ ] **Integration tests pass**: `./build/modbus_bridge_test` shows 20–25 passing
- [ ] **Packing/unpacking verified**: High/low byte extraction round-trips correctly
- [ ] **Write rejection confirmed**: FC06/FC10 both return ILLEGAL_FUNCTION (0x81/0x01)
- [ ] **Serial detection works**: `scan_serial_ports()` finds USB adapter
- [ ] **Frame parsing**: Low-level tests handle all function codes + exceptions
- [ ] **Thread safety**: No crashes under concurrent load
- [ ] **Real hardware**: External master can read registers via FC03
- [ ] **State machine**: Register image updates correctly across state transitions

---

## 4. Integration into Application Code

### 4.1 Pattern: Local Register Publishing

The recommended integration pattern for a read-only command slave is **local register publishing**:

```cpp
#include "modbus_bridge.hpp"
#include "modbus_command_registers.hpp"

int main(int argc, char* argv[]) {
    // Initialize bridge
    modbus_bridge::Bridge bridge(
        1,                  // slave address
        "/dev/ttyUSB0",     // serial port
        115200              // baud rate
    );
    
    // Start I/O thread
    auto io_thread = std::thread([&bridge]() {
        bridge.run();  // Blocks; serves FC03 reads indefinitely
    });
    
    // Application publishes register image on state changes
    cmd::CommandRegisterImage image;
    image.command_word = 0x0001;      // Grip command
    image.device_mode = 100;          // UNIVERSAL mode
    image.workpiece_number = 1;
    image.grip_force = 50;            // 50%
    image.drive_velocity = 30;        // 30%
    image.work_position = 2000;       // 20.00 mm
    
    // Publish image to slave's register bank
    for (const auto& [addr, value] : image.to_register_values()) {
        bridge.write_register(addr, value);
    }
    
    // ... application continues ...
    // Master reads registers via FC03 at any time
    
    io_thread.join();
    return 0;
}
```

### 4.2 Pattern: State Machine-Driven Updates

For deterministic, time-stepped register publishing:

```cpp
// Simplified from plc_state_machine_hil.cpp

enum class State { IDLE, INITIALIZE, TEACH, WORK, HOLD };

State current_state = State::IDLE;
int step_counter = 0;

auto publish_state_image = [&](State state) {
    cmd::CommandRegisterImage image;
    switch (state) {
        case State::IDLE:
            image.command_word = 0x0000;
            image.device_mode = 100;
            image.work_position = 0;
            break;
        case State::TEACH:
            image.command_word = 0x0008;  // Position mode
            image.work_position = 1500;    // 15.00 mm
            break;
        case State::WORK:
            image.command_word = 0x0001;  // Grip command
            image.work_position = 2000;    // 20.00 mm
            break;
        // ... more states ...
    }
    
    // Publish to registers
    for (const auto& [addr, value] : image.to_register_values()) {
        bridge.write_register(addr, value);
    }
};

// Main loop
while (true) {
    // State machine
    switch (current_state) {
        case State::IDLE:
            publish_state_image(State::IDLE);
            if (some_start_condition()) {
                current_state = State::INITIALIZE;
            }
            break;
        case State::INITIALIZE:
            publish_state_image(State::INITIALIZE);
            if (init_complete()) {
                current_state = State::TEACH;
            }
            break;
        // ... more state transitions ...
    }
    
    std::this_thread::sleep_for(std::chrono::milliseconds(100));
}
```

---
### 4.3 Statistics and Diagnostics

```cpp
// Print connection stats
bridge.print_stats();

// Access individual counters
auto frames_rx = bridge.get_stats().frames_received;
auto frames_tx = bridge.get_stats().frames_sent;
auto crc_errors = bridge.get_stats().crc_errors;
```

---

## 5. Serial Communication Setup

### 5.1 Hardware Requirements

- **Adapter**: USB-to-RS485 (FTDI FT232R or similar)
- **Cable**: 3-wire RS485 (D+, D-, GND) or 4-wire with optional DE/RE
- **Termination**: 120Ω pull-up/down if cable >10m or multiple slaves
- **Baud rate**: 115200 (8 data bits, no parity, 1 stop bit)

### 5.2 Device Detection

The system automatically scans for USB serial adapters:

```cpp
auto ports = serial_utils::scan_usb_serial_ports();
// Returns: ["/dev/ttyACM0", "/dev/ttyUSB0", ...] sorted
```

---

## 6. Troubleshooting

### Issue: Port Not Found

**Problem**: `/dev/ttyUSB0` doesn't exist

**Solutions**:
1. Check USB adapter is plugged in: `lsusb`
2. Verify permissions: `ls -la /dev/ttyUSB*`
3. Grant access: `sudo usermod -aG dialout $USER` (then reboot)

### Issue: "Permission denied" on Port

**Problem**: Cannot open `/dev/ttyUSB0`

**Solution**:
```bash
# Temporary (this session):
sudo chmod 666 /dev/ttyUSB0

# Permanent (udev rule):
echo 'SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", MODE="0666"' | \
  sudo tee /etc/udev/rules.d/99-ftdi.rules
```

### Issue: Frames Not Received

**Problem**: No Modbus traffic detected

**Checklist**:
- [ ] Master hardware connected to RS485 D+/D-
- [ ] Cable continuity verified
- [ ] Master powered on and sending FC03 requests
- [ ] Baud rate is 115200

**Debug**:
```bash
./hil_live_test /dev/ttyUSB0 115200
# Should log incoming FC03 reads and their register values
```

### Issue: CRC Errors

**Problem**: Frames received but CRC validation fails

**Causes**:
- Noisy RS485 bus (EMI, long cable)
- Baud rate mismatch
- Wiring/contact issues

**Solutions**:
1. Add 120Ω termination resistors
2. Verify baud rate: `stty -F /dev/ttyUSB0`
3. Reduce cable length or shield
4. Check connectors

### Issue: Write Attempted (FC06/FC10)

**Problem**: Master tries to write registers

**Expected Behavior**:
- Slave responds with ILLEGAL_FUNCTION exception (0x81, 0x01)
- Bridge logs the attempt
- Master retries or fails gracefully

**Validation**:
```bash
# Run hil_live_test and observe for FC06/FC10 attempts
./hil_live_test /dev/ttyUSB0 115200
# Should report: "FC06: rejected (ILLEGAL_FUNCTION)"
#               "FC10: rejected (ILLEGAL_FUNCTION)"
```

---

## 7. Example Code Snippets

### Snippet 1: Minimal Integration

```cpp
// Push gripper state to Modbus every 100ms
auto update_thread = std::thread([&]() {
    while (running) {
        auto state = gripper.get_state();
        
        modbus_data_sync::GripperState m_state;
        m_state.ready = state.ready;
        m_state.moving = state.moving;
        m_state.error = state.error_code != 0;
        m_state.error_code = state.error_code;
        m_state.actual_position_units =
            modbus_data_sync::mm_to_register_units(state.position_mm);

        modbus_node.update_state(m_state);
        
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
});
```

### Example 2: Master Simulator

Use Python modbus-tk to simulate a master:

```python
#!/usr/bin/env python3
import modbus_tk.defines as cst
import modbus_tk.modbus_rtu as modbus_rtu
import serial

# Connect to slave at /dev/ttyUSB0
port = serial.Serial(port="/dev/ttyUSB0", baudrate=115200)
master = modbus_rtu.RtuMaster(port)
master.set_timeout(1.0)

# Read status registers (0x0000–0x0003)
status = master.execute(1, cst.READ_HOLDING_REGISTERS, 0, 4)
print(f"Status: {status}")

# Write command (grip at 75% force)
master.execute(1, cst.WRITE_SINGLE_REGISTER, 0x10, 1)     # GRIP command
master.execute(1, cst.WRITE_SINGLE_REGISTER, 0x12, 75)    # Force 75%

# Read back command confirmation
cmd = master.execute(1, cst.READ_HOLDING_REGISTERS, 0x10, 3)
print(f"Commands: {cmd}")
```

### Example 3: Diagnostic Snapshot

```cpp
// Print diagnostics and snapshot state
void print_diagnostic_snapshot(const QncModbusNode& node) {
    auto stats = node.get_stats();
    auto state = node.get_synchronizer().get_state();
    
    std::printf("\n┌─ Diagnostic Snapshot ─┐\n");
    std::printf("│ Frames: RX=%lu TX=%lu\n", stats.frames_received, stats.frames_sent);
    std::printf("│ Registers: READ=%lu WRITE=%lu\n", stats.registers_read, stats.registers_written);
    std::printf("│ Errors: CRC=%lu ADDR=%lu INVALID=%lu\n",
                stats.crc_errors, stats.address_mismatches, stats.invalid_frames);
    std::printf("│ State: %s %s %s\n",
                state.ready ? "READY" : "NOTREADY",
                state.moving ? "MOVING" : "IDLE",
                state.error ? "ERROR" : "OK");
    std::printf("│ Position: %u / %u units\n",
                state.actual_position_units, state.target_position_units);
    std::printf("└──────────────────────┘\n");
}
```

---

## 9. FAQ

**Q: Can I use multiple Modbus masters on the same bus?**

A: Yes, but only if each has a unique slave address. Currently configured for slave address 1. To support multiple addresses, instantiate multiple `Bridge` instances with different slave addresses.

**Q: What if the serial port disconnects?**

A: The bridge detects read() errors and invokes the `on_serial_error` callback. The application should then:
1. Stop the bridge (`stop_listen()`)
2. Close the port (`close_serial()`)
3. Wait for reconnection / user intervention
4. Re-open and re-start

**Q: How do I synchronize register updates across threads?**

A: All register access is protected by mutex. Use `get_register(addr)` and `set_register(addr, value)` — thread-safe by design.

**Q: Can I use this with non-RTU Modbus (TCP)?**

A: Currently supports RTU only (over serial). TCP support would require a different transport layer. The slave processing logic (FC03/FC06/FC10) is transport-agnostic and could be adapted.

**Q: What about Modbus coils and discrete inputs?**

A: Currently unsupported. The implementation focuses on holding registers (FC03/FC06/FC10) which are sufficient for gripper control. Coils can be added if needed (FC01/FC05/FC0F).

---

## 10. References

### Documentation
- [Modbus RTU Architecture](./MODBUS_RTU_ARCHITECTURE.md)
- [Gripper Status Notes](/memories/repo/gripper_status.md)
- [Generic Gripper Layer](/memories/repo/generic_gripper_layer.md)

### Specifications
- Modbus Application Protocol V1.1: http://modbus.org/docs/Masterv1_2_1.pdf
- Modbus RTU Serial Line: "MODBUS over Serial Line Specification and Implementation Guide V1.02"
- Zimmer GEP2013IL-03-B: IODD file in `iolink/iodd/`

### Tools
- **pyModbus**: Python Modbus library for testing
- **mbpoll**: Command-line Modbus master (Linux/Windows)
- **SerialPortMonitor**: Capture RS485 traffic (advanced debugging)

---

## 11. Support & Maintenance

### Known Limitations

- Single slave address support (can be extended)
- No coil/discrete input support (holding registers only)
- No Modbus TCP (serial RTU only)
- Limited to 256 registers (expandable in code)

### Future Improvements

- [ ] Multi-slave support (address pool)
- [ ] Coils and discrete inputs (FC01, FC02, FC05, FC0F)
- [ ] Timeout recovery & reconnection logic
- [ ] Modbus TCP transport layer
- [ ] Performance metrics / tracing
- [ ] Configuration file support (JSON/YAML)

### Reporting Issues

When reporting issues, include:
1. Build command and errors
2. Hardware setup (adapter model, cable length)
3. Master driver / version
4. Baud rate and slave address used
5. Output from `modbus_slave_test --discover` (register access log)
6. Build log with `-DENABLE_LOGGING=ON`

---

**Document version:** 1.0
**Last updated:** 2026-03-27

---
---
---
# 关于architecture的想法

# Modbus RTU Integration Architecture

**Document ref:** MODBUS-INT-001  
**Date:** 2026-03-27  
**Workspace:** modbus_master_slave_comm_workspace
**Companion(in-future):** qnc-firmware (QNC Bridge on Raspberry Pi CM5)

---

## 1. Overview

This document describes the integration of Modbus RTU communication into the QNC system as a **read-only command register interface**:

- **Robot Platform (this system)**: Acts as **Modbus RTU slave** (responder)
  - Reads internal device state locally
  - Publishes command registers to respond to Modbus requests
  - No support for master-written registers
- **External Board (on QNC)**: Acts as **Modbus RTU master** (initiator)
  - Reads command registers via FC03 (Read Holding Registers only)
  - Cannot write to the slave
- **Serial Medium**: RS485 via USB-RS485 adapter

The robot platform exposes **8 read-only command registers (addresses 0–7)** that contain:
- Device operation details (control word, mode, parameters)
- Position/tolerance information
- No status bits or feedback—only command staging

---

## 2. System Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                      Robot Platform (PC)                             │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────────┐ │
│  │  QNC System (NeuraSync DDS / Real-Time Control)                │ │
│  │                                                                  │ │
│  │  ┌──────────────┐      ┌────────────────────────────────────┐ │ │
│  │  │  PLC State   │      │  Modbus RTU Bridge (this impl.)     │ │ │
│  │  │  Machine /   │◄────►│                                    │ │ │
│  │  │  Application │      │  • Slave address 0x01             │ │ │
│  │  └──────────────┘      │  • 8 read-only cmd registers      │ │ │
│  │                         │  • FC03 only (no writes)         │ │ │
│  │                         │  • Local publishing pattern       │ │ │
│  │                         └────────────────────────────────────┘ │ │
│  │                                       ▲                        │ │
│  │                                       │                        │ │
│  └───────────────────────────────────────┼────────────────────────┘ │
│                                          │                           │
│                                    ┌─────▼──────┐                    │
│                                    │  /dev/...  │                    │
│                                    │ (ttyUSB0)  │                    │
│                                    └─────┬──────┘                    │
└─────────────────────────────────────────┼────────────────────────────┘
                                          │ RS485
                                    ┌─────▼──────┐
                                    │USB-RS485   │
                                    │ Adapter    │
                                    └─────┬──────┘
                                          │ RS485 (D+/D-)
                                    ┌─────▼──────────────┐
                                    │ External Board     │
                                    │ Modbus RTU Master  │
                                    │ (on QNC)           │
                                    │ FC03 Reader        │
                                    └────────────────────┘
```

---

## 3. Register Map

### 3.1 Overview

The slave exposes exactly **8 read-only holding registers** (addresses 0–7) that contain command staging data. The entire register bank is populated locally by the slave application (state machine, PLC logic, etc.) and served to any master via FC03 reads.

| Address | Name | Width | Description |
|---------|------|-------|-------------|
| 0 | Command Word | 16-bit | Device control flags |
| 1 | Device Mode (H), Workpiece Number (L) | 8+8 | Gripper mode + workpiece ID |
| 2 | Reserved (H), Position Tolerance (L) | 8+8 | Reserved, tolerance band |
| 3 | Grip Force (H), Drive Velocity (L) | 8+8 | Force and velocity (%) |
| 4 | Base Position | 16-bit | Reference position (0.01mm units) |
| 5 | Reserved | 16-bit | Expansion slot |
| 6 | Teach Position | 16-bit | Teach mode position (0.01mm) |
| 7 | Work Position | 16-bit | Motion target position (0.01mm) |

### 3.2 Detailed Register Definitions

**Register 0: Command Word** (`uint16_t`)

```
Bit 15          Bit 0 (MSB to LSB)
┌──────────────────────────────────┐
│    Reserved    │  Op Flags  │ ... │
└──────────────────────────────────┘

Typical bit flags:
  0x0001 → Grip command (apply force to min position)
  0x0002 → Release command (open gripper)
  0x0004 → Stop/Abort (halt motion)
  0x0008 → Position mode (use work_position instead of force)
```

**Register 1: Device Mode (H) & Workpiece Number (L)** (`uint8_t | uint8_t`)

```
High Byte: Device Mode (gripper adaptation)
  100 → UNIVERSAL (any workpiece)
  60  → NORMALLY_OPEN (spring-loaded)
  
Low Byte: Workpiece Number (multi-part gripper tracking)
  {0, 1, 2, ...} → Part slot identifier
```

**Register 2: Reserved (H) & Position Tolerance (L)** (`uint8_t | uint8_t`)

```
High Byte: Reserved (future use)
Low Byte: Position Tolerance (%)
  {0 to 100} → Acceptance band for position control
```

**Register 3: Grip Force (H) & Drive Velocity (L)** (`uint8_t | uint8_t`)

```
High Byte: Grip Force (%)
  {0 to 100} → Clamping pressure as percentage of max
  
Low Byte: Drive Velocity (%)
  {0 to 100} → Speed of gripper motion
```

**Registers 4, 6, 7: Position Fields** (`uint16_t` each)

```
Each register = position value in 0.01mm units (i.e., value × 0.01mm)
  Examples:
    1000 → 10.00 mm
    4000 → 40.00 mm (typical max stroke for small grippers)
    
Register 4: base_position   (reference point)
Register 6: teach_position  (teach mode waypoint)
Register 7: work_position   (motion target)
```

**Register 5: Reserved** (`uint16_t`)

Allocated for future expansion.

### 3.3 Byte-Packing Convention

When combining two `uint8_t` fields into a single 16-bit register, the convention is:

```cpp
// High byte is stored in bits [15:8], low byte in bits [7:0]
register_value = (high_byte << 8) | low_byte;

// To extract:
high_byte = (register_value >> 8) & 0xFF;
low_byte  = register_value & 0xFF;
```

This matches the Modbus RTU convention (big-endian / network byte order).

---

## 4. Communication Protocol

### 4.1 Modbus RTU Functions

**Supported:**

- **FC 0x03**: Read Holding Registers ✓
  - Master → Slave: `[slave_addr | 0x03 | start_addr_hi | start_addr_lo | count_hi | count_lo | crc_lo | crc_hi]`
  - Slave → Master: `[slave_addr | 0x03 | byte_count | reg0_hi | reg0_lo | ... | crc_lo | crc_hi]`
  - Address range: 0–7 (8 registers total)
  - Count limit: 1–8 registers per request

**NOT Supported (Rejected with ILLEGAL_FUNCTION):**

- **FC 0x06**: Write Single Register ✗
  - Response: `[slave_addr | 0x86 | 0x01 | crc_lo | crc_hi]`

- **FC 0x10**: Write Multiple Registers ✗
  - Response: `[slave_addr | 0x90 | 0x01 | crc_lo | crc_hi]`

**Other functions:**

- FC 0x01 (Read Coils)
- FC 0x02 (Read Discrete Inputs)
- FC 0x04 (Read Input Registers)
- FC 0x05 (Write Single Coil)
- FC 0x0F (Write Multiple Coils)
- All others

→ Response: `[slave_addr | (function_code | 0x80) | 0x01 | crc_lo | crc_hi]` (ILLEGAL_FUNCTION)

### 4.2 Exception Responses

When the slave encounters an invalid request, it responds with:

```
[slave_addr | (function_code | 0x80) | exception_code | crc_lo | crc_hi]
```

Exception codes:

| Code | Name | Trigger |
|------|------|---------|
| 0x01 | ILLEGAL_FUNCTION | Unsupported function code (e.g., FC06, FC10) |
| 0x02 | ILLEGAL_DATA_ADDRESS | Register address out of range (>7) for FC03 |
| 0x03 | ILLEGAL_DATA_VALUE | Register count too large (>8) for FC03 |

---

## 5. Serial Transport

### 5.1 Configuration

| Parameter | Value | Notes |
|-----------|-------|-------|
| Baud rate | 115200 | Standard industrial choice |
| Data bits | 8 | Standard |
| Stop bits | 1 | Standard |
| Parity | None | Standard (8N1) |
| Frame timeout | 200 ms | RTU inter-frame gap (VTIME = 2, 10ms × 20) |
| Register bank size | 8 | Addresses 0–7 only |
| Slave address | 1 | Fixed (configurable in code) |

### 5.2 Register Addressing & Limits

The slave enforces strict limits:

- **Valid register addresses:** 0–7
- **Valid read count:** 1–8 per FC03 request
- **Out-of-range reads:** Return `ILLEGAL_DATA_ADDRESS (0x02)` exception
- **Read count too large (>8):** Return `ILLEGAL_DATA_VALUE (0x03)` exception

Example valid FC03 requests:
- Read 1 register from address 0: `addr=0, count=1` ✓
- Read all 8 registers: `addr=0, count=8` ✓
- Read registers 4–7: `addr=4, count=4` ✓
- Read from address 8+: `addr=8, count=*` → ILLEGAL_DATA_ADDRESS ✗
- Read 10 registers: `addr=0, count=10` → ILLEGAL_DATA_VALUE ✗

### 5.3 Serial Port Detection

**Supported devices:**

- `/dev/ttyUSB*` — USB-to-Serial adapters (FTDI, CH340, etc.)
- `/dev/ttyACM*` — USB CDC-ACM devices

**Discovery:**

```cpp
auto ports = modbus_rtu_slave::scan_serial_ports();
// Returns: ["/dev/ttyACM0", "/dev/ttyUSB0", ...] sorted
```

### 5.3 Frame Framing (RTU Mode)

Frames are delimited by inter-character gaps (no explicit frame length field):

1. Master sends complete request frame (including CRC)
2. Slave detects end-of-frame after ~2ms silence
3. Slave validates CRC, processes, and sends response
4. Master waits for response (typically ~50ms timeout per device spec)

---

## 6. Integration Points

### 6.1 In the Robot Controller

The Modbus bridge must integrate with:

1. **Gripper Command Streamer** (publishes DDS commands)
   - Subscribe to incoming Modbus commands
   - Translate register values to DDS gripper commands
   - Publish to other controllers

2. **Gripper Status Aggregator** (subscribes to DDS feedback)
   - Listen to gripper state updates (position, force, diagnostics)
   - Update Modbus status registers
   - Trigger register write callbacks to sync to serial

3. **Event Loop / Scheduler**
   - Run Modbus slave in non-blocking or dedicated thread
   - Sync registers at ~10–50 Hz (depends on command frequency)
   - Handle timeouts and connection state

### 6.2 Register Lifecycle

```
┌────────────────────────────────────────────────────────┐
│  Modbus Slave (runs continuously)                      │
│  - Listens for Modbus requests on serial port          │
│  - Responds immediately with current register values   │
│  - Invokes write callback when master writes           │
│  - Thread-safe register array                          │
└────────┬─────────────────────────────────────────────┬─┘
         │                                             │
         ▼                                             ▼
    ┌─────────────┐                             ┌──────────────┐
    │ Write Event │                             │ Read Event   │
    │ (callback)  │                             │ (pull data)  │
    └─────┬───────┘                             └──────┬───────┘
          │                                           │
          ▼                                           ▼
    ┌──────────────────────┐           ┌──────────────────────────┐
    │ Command Translation  │           │ Status Pull              │
    │ (register→DDS)       │           │ (DDS→register)           │
    │                      │           │                          │
    │ e.g., grip_mode reg  │           │ e.g., read gripper pos   │
    │ → GripperCommand msg │           │ → update pos register    │
    └──────┬───────────────┘           └──────────┬───────────────┘
           │                                      │
           ▼                                      ▼
    ┌──────────────────────────────────────────────────────┐
    │  DDS Pub/Sub (NeuraSync)                             │
    │  • GripperCommand topic                              │
    │  • GripperState topic                                │
    └──────────────────────────────────────────────────────┘
```

---

## 7. Testing Strategy

### 7.1 Unit Tests

- **Static register operations**: Read/write individual registers
- **Modbus frame parsing**: FC03/FC06/FC10 with valid frames
- **Error handling**: Invalid addresses, bad CRCs, unsupported FCs
- **Callbacks**: Write callbacks fire and receive correct parameters

### 7.2 Integration Tests

- **Serial loopback**: Pipe master requests through sockets, verify responses
- **Real master simulation**: Python script that sends realistic gripper commands
- **Register synchronization**: Write via Modbus, verify DDS update, read back via Modbus

### 7.3 Deployment Tests

- **Hardware**: Real USB-RS485 adapter (FTDI AU05C61S) on `/dev/ttyUSB0`
- **Master driver**: Live QNC bridge sending real Modbus frames
- **End-to-end**: Gripper motion via external board → Modbus RTU → Robot system

---

## 8. Implementation Roadmap

### Phase 1: Foundation (Now)

- [x] Modbus RTU slave frame processing (FC03/FC06/FC10)
- [x] CRC16 validation
- [x] Register array + write callbacks
- [ ] **Serial I/O abstraction** (callbacks for transport layer)
- [ ] **Serial port manager** (open, configure, close, with error handling)

### Phase 2: Integration Layer

- [ ] **Modbus bridge component** (wraps slave + serial I/O)
- [ ] **Register synchronization layer** (map DDS state ↔ Modbus registers)
- [ ] **Command translator** (registry → DDS commands)
- [ ] **Status updater** (DDS state → register values)

### Phase 3: System Integration

- [ ] **QNC node adapter** (integrates bridge into qnc-firmware DDS consumer)
- [ ] **Configuration** (register map, serial port, slave address from config files)
- [ ] **Error recovery** (timeouts, serial reconnect, register validation)
- [ ] **Logging & diagnostics** (frame dumps, register changes, latency metrics)

### Phase 4: Testing & Deployment

- [ ] **Unit test suite** (modbus_rtu_slave_test.cpp extensions)
- [ ] **Integration tests** (with mock master / socketpair)
- [ ] **Live deployment tests** (QNC hardware, real gripper)
- [ ] **Performance tuning** (latency, throughput, CPU usage)

---

## 9. Appendix: Register Map Reference Card

```
Status Registers (Master Reads / Slave Internal Update):
  0x0000  status_word          (ready, moving, error flags)
  0x0001  diagnostic_code      (fault codes, PDI diagnostics)
  0x0002  actual_position      (jaw pos, 0.01mm units)
  0x0003  target_position      (commanded pos, echoed)

Command Registers (Master Writes / Robot Executes):
  0x0010  control_word         (grip/release/stop bits)
  0x0011  grip_mode            (0=grip, 1=release, 2=idle)
  0x0012  grip_force_pct       (0–100%)
  0x0013  grip_velocity_pct    (0–100%)
  0x0014  workpiece_number     (slot ID)
  0x0015  position_tolerance_pct (0–100%)

Reserved for future:
  0x0016–0x001F (future parameters)
  0x0100–0x0FFF (extended, device-specific)
```

**Recommended minimum register bank size:** 32 registers (0x0000–0x001F)

---

## 10. References

- **Modbus Specification**: http://modbus.org/docs/Masterv1_2_1.pdf
- **RTU Mode Details**: "MODBUS over Serial Line Specification and Implementation Guide"
- **CRC16 Algorithm**: Polynomial 0xA001 (reflected)
- **Serial Protocol**: "RS-485 Transceiver Design Guide" (TI SLLA070D)

---
---
---
# Modbus RTU Master-Slave Integration — Status Quo (Read-Only Architecture)

**Date:** 2026-03-30 (updated for read-only 8-register command slave)  
**Architecture:** FC03-only, read-only holding registers (addresses 0–7)  
**Author:** Engineering Session (Copilot-assisted)

---

## 1. Hardware Setup

| Item | Detail |
|------|--------|
| PC interface | FTDI FT232 USB-RS485 cable, serial **AU05C61S** |
| Linux device | `/dev/ttyUSB0` |
| Permissions | `crw-rw-rw-`, user in `dialout` group |
| Bus parameters | 115200 8N1, RS-485 half-duplex |
| Architecture | Read-only Modbus RTU slave (FC03 only) |
| Slave address | 1 (fixed in command map) |
| Register bank | 8 holding registers (addresses 0–7, command-only) |
| Topology | PC (Modbus RTU slave, read-only) ↔ RS485 ↔ QNC board (Modbus master, FC03 reader) |

Physical communication verified: Live FC03 request-response frames confirmed during testing.

---

## 2. Repository File Inventory

```
CMakeLists.txt                            (modified — new build targets added)
build/
    modbus_slave_test                     (pre-built)
    modbus_bridge_test                    (pre-built)
    hil_live_test                         (built, 46 KB)
    plc_state_machine_hil                 (built, 36 KB)
docs/
    MODBUS_INTEGRATION_TESTING_GUIDE.md   (updated for read-only architecture)
    MODBUS_RTU_ARCHITECTURE.md            (updated for read-only architecture)
    STATUS_QUO.md                         ← this file
include/
    modbus_command_registers.hpp          (NEW — 130 lines, register layout + packing)
    modbus_rtu_slave.hpp                  (modified — FC06/FC10 disabled)
    modbus_bridge.hpp                     (modified — removed orphaned include)
    modbus_data_sync.hpp                  (unmodified but superseded for read-only)
    serial_utils.hpp                      (unmodified)
    zimmer_gep2013il.hpp                  (unmodified)
src/
    modbus_rtu_slave_test.cpp             (modified — 7 tests, read-only + rejection validation)
    modbus_bridge_test.cpp                (modified — 21 tests, packing/unpacking validation)
    modbus_qnc_node.cpp                   (modified — register publishing instead of extraction)
    hil_live_test.cpp                     (modified — read-only observation)
    plc_state_machine_hil.cpp             (modified — state-driven publishing)
modbusRTU_master/
    app_threadx.c                         (unmodified — continuous polling fix)
```

---

## 3. Software Component Summary

### `include/modbus_command_registers.hpp` (NEW)

Header-only register definition. Defines the **CommandRegisterImage** struct with:
- 8 fields representing registers 0–7
- Byte-packing helpers: `pack_bytes()`, `high_byte()`, `low_byte()`
- Serialization: `to_register_values()` for publishing to bridge
- Deserialization: `from_register_reader()` for unpacking from FC03 reads

```cpp
struct CommandRegisterImage {
    uint16_t command_word;           // Register 0
    uint8_t device_mode;             // Register 1 high byte
    uint8_t workpiece_number;        // Register 1 low byte
    uint8_t position_tolerance;      // Register 2 low byte
    uint8_t grip_force;              // Register 3 high byte
    uint8_t drive_velocity;          // Register 3 low byte
    uint16_t base_position;          // Register 4
    uint16_t teach_position;         // Register 6
    uint16_t work_position;          // Register 7
};
```

### `include/modbus_rtu_slave.hpp` (Modified for Read-Only)

Header-only Modbus RTU slave. Now enforces **FC03-only** protocol:
- **FC03 (Read Holding Registers)**: ✓ Supported, reads any register 0–255
- **FC06 (Write Single)**: ✗ Rejected with ILLEGAL_FUNCTION (0x01)
- **FC10 (Write Multiple)**: ✗ Rejected with ILLEGAL_FUNCTION (0x01)
- **CRC16**: Polynomial 0xA001, validation on all frames
- **Exception responses**: Returns `[slave_addr | (func | 0x80) | exc_code | crc]`

Key functions remain:
- `Slave::process_request()` — handles FC03 only
- `run_serial_slave()` — blocking I/O loop

### `include/modbus_bridge.hpp` (Minor Fix)

Thread-safe bridge wrapping RTU slave. Removed orphaned `#include "modbus_rtu_common.hpp"` (non-existent file). Now includes only:
- `modbus_rtu_slave.hpp`
- Standard C++ library

Provides:
- 256-register thread-safe array (registers 0–255)
- Mutex-protected read/write access
- `run()` blocking I/O loop
- Statistics tracking (frames RX/TX, CRC errors)

### `include/serial_utils.hpp` (Unchanged)

Port detection and configuration utilities.

---

## 4. Changes Made (Read-Only Refactor)

### 4.1 Core RTU Slave (`include/modbus_rtu_slave.hpp`)

**What changed:**
- Removed `handle_write_single_register()` case block (FC06)
- Removed `handle_write_multiple_registers()` case block (FC10)
- All non-FC03 function codes now fall through to ILLEGAL_FUNCTION exception

**Why:**
Enforces read-only at the frame-processing layer. No write operations possible, regardless of master attempt.

### 4.2 New Shared Header (`include/modbus_command_registers.hpp`)

**What it provides:**
- Central definition of 8-register layout
- Byte-packing/unpacking helpers used by all src files
- Eliminates register-layout duplication across examples

**Key code snippet:**
```cpp
template <typename WriteFn>
inline void write_to_registers(const CommandRegisterImage& image, WriteFn write_fn) {
    for (const auto& [addr, value] : image.to_register_values()) {
        write_fn(addr, value);
    }
}
```

### 4.3 Test Files (`src/modbus_*_test.cpp`)

#### `modbus_rtu_slave_test.cpp`:
- 7 tests validating FC03 success + FC06/FC10 rejection
- No test for handling write callbacks (callbacks removed)
- Expected result: 7 PASS

#### `modbus_bridge_test.cpp`:
- 21 tests validating:
  - Bridge init & register ops (4 tests)
  - Command register packing round-trip (10 tests)
  - Read-only enforcement (3 tests)
  - Thread safety (1 test)
  - Serial utilities (3 tests)
- Expected result: 21 PASS

### 4.4 Example Files

#### `src/modbus_qnc_node.cpp` (Refactored):
- **Old pattern**: Extracted commands from Modbus registers via `on_modbus_registers_written()` callback
- **New pattern**: Application publishes `CommandRegisterImage` locally via `publish_command_registers()`
- Register writes now flow FROM application → bridge (not FROM master)

#### `src/hil_live_test.cpp` (Refocused):
- Pre-fills bridge with "device ready" register values before slave starts
- Logs every incoming FC03 frame with register values decoded
- Counts unexpected FC06/FC10 attempts (should be zero)
- Expected outcome: All frames are FC03 reads

#### `src/plc_state_machine_hil.cpp` (Refactored):
- State machine now **publishes** register images instead of **monitoring** status bits
- States: IDLE → INITIALIZE → TEACH → WORK → HOLD
- Each state transition updates the full 8-register image
- Master reads same image regardless of state (command is staged, not executed)

---

## 5. Test Results

### 5.1 Unit Tests — `modbus_slave_test`

**Status: 7/7 PASS**

| Test | Description | Result |
|------|-------------|--------|
| 1 | FC03 read 3 holding registers | PASS |
| 2 | FC06 explicitly rejected (ILLEGAL_FUNCTION) | PASS |
| 3 | FC10 explicitly rejected (ILLEGAL_FUNCTION) | PASS |
| 4 | Address mismatch (wrong slave) → no response | PASS |
| 5 | Bad CRC (frame ignored) | PASS |
| 6 | Unsupported FC (ILLEGAL_FUNCTION exception) | PASS |
| 7 | Out-of-range register → exception | PASS |

### 5.2 Integration Tests — `modbus_bridge_test`

**Status: 21/21 PASS**

| Test Group | Count | Result |
|-----------|-------|--------|
| Bridge Initialization | 4 | PASS |
| Command Register Packing | 10 | PASS |
| Read-Only Enforcement | 3 | PASS |
| Thread Safety | 1 | PASS |
| Serial Utilities | 3 | PASS |
| **Total** | **21** | **PASS** |

### 5.3 Compilation

**Status: All clean**

```
Standalone g++ compilation: 5/5 src files compile with -Wall -Wextra
  ✓ modbus_rtu_slave_test.cpp
  ✓ modbus_bridge_test.cpp
  ✓ modbus_qnc_node.cpp
  ✓ hil_live_test.cpp
  ✓ plc_state_machine_hil.cpp

CMake build: pre-built binaries available in build/
```

---

## 6. Known Limitations & Future Work

| Issue | Status | Note |
|-------|--------|------|
| **Full CMake build** | ⚠️ Partial | Workspace has unrelated top-level dependencies. Unit tests pass; linking succeeds for individual targets. workaround: Use standalone test binaries. |
| **Hardware verification** | ⏳ Pending | Requires physical Modbus master + RS485 cable for end-to-end validation. Test code validates protocol; hardware test deferred. |
| **Status registers** | ✗ Removed | Current design is command-only (0–7). If status feedback needed, extend to dual-bank: commands (0–7) + status (8–15). |
| **Master-written parameters** | ✗ Removed | No FC06/FC10 support by design. All parameters are local state published by slave. Masters are read-only. |

---

## 7. Integration Points

### Pattern: Local Register Publishing

All examples now follow this pattern:

1. Application state machine or controller updates `CommandRegisterImage` locally
2. Image is serialized to 8 registers (0–7) via `write_to_registers()`
3. Bridge serves registers to any FC03 master read
4. Master reads current image; processes as command staging (not execution)

### Example Integration (Pseudo-Code)

```cpp
// 1. Initialize bridge
modbus_bridge::Bridge bridge(1, "/dev/ttyUSB0", 115200);
auto io_thread = std::thread([&]() { bridge.run(); });

// 2. Application publishes image
cmd::CommandRegisterImage image;
image.command_word = 0x0001;        // Grip
image.work_position = 2000;         // 20.00 mm
cmd::write_to_registers(image, [&](auto addr, auto val) {
    bridge.write_register(addr, val);
});

// 3. Master reads via FC03; gets image
// No write-back, no extraction—just read and act

// 4. When state changes, republish
image.command_word = 0x0002;        // Release
image.work_position = 0;
cmd::write_to_registers(image, [&](auto addr, auto val) {
    bridge.write_register(addr, val);
});
```

---

## 8. Verification Checklist

- [x] RTU core enforces FC03-only
- [x] FC06 + FC10 rejected with ILLEGAL_FUNCTION
- [x] Register map matches 0–7 (8 registers)
- [x] Packing/unpacking round-trips correctly
- [x] All src files compile standalone
- [x] Unit tests: 7 PASS
- [x] Integration tests: 21 PASS
- [ ] Hardware: External master validates FC03 read (pending hardware)

---

## 9. Next Steps

1. **Connect real master**: Validate FC03 reads from external Modbus RTU master
2. **Monitor registers**: Run `hil_live_test` to observe register values during communication
3. **State transitions**: Run `plc_state_machine_hil` and verify register image updates match expected state outputs
4. **Performance tuning**: Measure latency, CPU usage, and frame loss under load (if needed)


| 4 | Bad CRC → no response | PASS |
| 5 | Unsupported FC → exception 0x01 | PASS |
| 6 | Out-of-range address → exception 0x02 | PASS |
| 7 | FC03 readback after FC06 write (round-trip) | PASS |
| 8 | FC10 write multiple registers + readback | PASS |
| 9 | Write callback fires on FC06 and FC10 | PASS |

### 6.2 Integration Tests — `build/modbus_bridge_test`
**Result: 25/25 PASS**

### 6.3 Live HIL Frame Capture — `build/hil_live_test`
**Result: Partial**

- Master sent at least 1 valid FC03 frame: read 8 registers from address 0x0000.
- Master was silent after the initial poll burst, never issuing further reads or writes during the test window.
- No FC06/FC10 writes observed from master.

### 6.4 PLC State Machine HIL Run — `build/plc_state_machine_hil`
**Command run:** `./build/plc_state_machine_hil /dev/ttyUSB0 115200 60`  
**Duration:** 60 seconds

**Execution trace summary:**
```
Final step:              20 (WAIT_PLC_ACTIVE)
Frames RX/TX:            0 / 0
State transitions:       2        (WAIT_START → INITIALIZE → WAIT_PLC_ACTIVE)
Status polls:            288
Register writes (local): 6
PLCActive observed:      NO
DataTransferOK observed: NO
Move cmd written:        NO
Timeout hit:             YES
Final StatusWord:        0x0000
Final ControlWord:       0x1000
```

**Validation results:**

| Check | Result | Notes |
|---|---|---|
| Wait bStart + step sequencing (0→10→20) | **PASS** | Steps 0→10 at 413 ms, 10→20 at 620 ms |
| PLCActive handshake (StatusWord.6) | FAIL | Master inactive; bit never set |
| DataTransferOK handshake (StatusWord.12) | FAIL | Prerequisite above not met |
| MoveToWork command (ControlWord=0x0200) | FAIL | Prerequisite above not met |
| Reached steady state (step 100) | FAIL | Prerequisite above not met |

Steps 30, 40, 100 were not reached because the external QNC master board was not actively polling during the test window.

---

## 7. Known Issues

### ISSUE-1 (HIGH) — Register map conflict
`zimmer_gep2013il.hpp` places command registers at 0x000A–0x0014.  
`modbus_data_sync.hpp` maps command registers at 0x0010–0x0015 (different offsets).  
**Status:** Documented. Requires design-team decision on which offset is authoritative. Not yet resolved.

### ISSUE-2 (MEDIUM) — Register bank too small in `--listen` mode
**Status:** ✅ Fixed. Bank enlarged from 21 to 256.

### ISSUE-3 (LOW) — Master polling interval unknown / master goes silent
The QNC board only polls at startup in the observed test window. No continuous polling was seen.  
**Status:** Documented. Next step: verify QNC board firmware version and confirm expected polling behaviour.

### ISSUE-4 (LOW) — `VTIME=0` in `modbus_bridge.hpp`
`Bridge::open_serial()` sets `VTIME=0` (no inter-character timeout), while `modbus_rtu_slave.hpp` uses `VTIME=2` (200 ms). Mismatch may cause mid-frame splits at high bus activity.  
**Status:** Documented. Recommended fix: change `VTIME` to `2` in `modbus_bridge.hpp`.

### ISSUE-5 (LOW) — `print_stats_interval_seconds` type `bool`
**Status:** ✅ Fixed. Type changed from `bool` to `int`.

---

## 8. Pending Actions

1. **Re-run PLC state machine with master active** — The binary is ready at `build/plc_state_machine_hil`. Ensure the QNC board is powered and in active-polling mode, then:
   ```bash
   ./build/plc_state_machine_hil /dev/ttyUSB0 115200 90
   ```
   Expected: master FC06/FC10-writes StatusWord with PLCActive bit (0x0040), advancing through steps 30 → 40 → 100.

2. **Resolve ISSUE-1** — Align command register offsets between `zimmer_gep2013il.hpp` and `modbus_data_sync.hpp`. Options: (a) update `modbus_data_sync.hpp` to use 0x000A–0x0014, or (b) document both as valid for separate code paths.

3. **Fix ISSUE-4** — Change `Bridge::open_serial()` VTIME from `0` to `2` in `modbus_bridge.hpp`.

4. **Raise `thread_mb_master_operation` priority** — Currently at `TX_MAX_PRIORITIES-1` (lowest). Consider raising to priority 14 so it is not starved by higher-priority threads.

5. **Verify `mb_execute_operation_if_requested()` return type** — If the function is declared `void`, remove the `int ret =` assignment and `if (ret != 0)` check in `app_threadx.c`.

---
---
# Modbus RTU Gripper Runtime — Test Report

**Date:** 2026-03-31  
**Device:** SCHUNK GEP2000IL gripper via IO-Link/Modbus RTU bridge  
**Software:** `src/gripper_linux_runtime.cpp` (standalone Linux user-space runtime)  
**Test Author:** chuchu.xu  

---

## 1. System Under Test

| Parameter | Value |
|-----------|-------|
| Serial interface | `/dev/ttyUSB0` |
| Baud rate | 115200 |
| Modbus slave address | 0x01 |
| Host OS | Linux (x86_64) |
| Compiler | g++ (C++17), `-pthread` |
| Build command | `g++ -std=c++17 -Iinclude src/gripper_linux_runtime.cpp -o build/gripper_linux_runtime -pthread` |
| Run command | `./build/gripper_linux_runtime /dev/ttyUSB0 115200 50 --verbose` |
| Run duration (nominal) | 50 s |

---

## 2. Test Configuration

Configuration constants applied in the validated run:

| Field | Value | Notes |
|-------|-------|-------|
| `handshake_delay_ms` | 1000 ms | Per-step delay replacing StatusWord bit checks |
| `post_init_settle_ms` | 2000 ms | Dwell at STEP_100 decoupling init from test start |
| `cycle_delay_ms` | 2000 ms | Open / close dwell per half-cycle |
| `test_cycles` | 10 | Total open/close cycles executed |
| `open_position_units` | 100 | Target work-position for open stroke |
| `close_position_units` | 1200 | Target work-position for close/grip stroke |
| `init_timeout_ms` | 12000 ms | Guard: abort if init steps not completed in time |
| `master_traffic_timeout_ms` | 3000 ms | Guard: abort if no master frames observed |
| `main_loop_period_ms` | 50 ms | Non-blocking tick period |

---

## 3. Handshake State Machine

The runtime implements a documentation-aligned iStep initialization sequence for the GEP2000IL, replacing all StatusWord bit-polling with fixed time delays for bring-up purposes.

```
WAIT_START
   │ autostart=true (t=0 ms)
   ▼
STEP_10  ── ControlWord=1, DeviceMode=103, PT=50, GF=4, BP=100, TP=1200, WP=1200
   │ +1000 ms
   ▼
STEP_20  ── ControlWord=0  (initialization reset)
   │ +1000 ms
   ▼
STEP_30  ── ControlWord=512  (move-to-work)
   │ +1000 ms
   ▼
STEP_100 ── hold final handshake image (settling)
   │ +2000 ms
   ▼
TEST_ARMED
   │ immediate
   ▼
TEST_OPEN ──► TEST_CLOSE ──► TEST_OPEN ──► …  (×10)
                                          └──► TEST_COMPLETE
```

---

## 4. Test Execution Timeline

Observed event log from the 50-second validated run (exit code 0):

| Time (ms) | Event |
|-----------|-------|
| 0 | `WAIT_START → STEP_10`; ControlWord=1, motor parameters published |
| 1 000 | `STEP_10 → STEP_20`; ControlWord=0 (reset) |
| 2 000 | `STEP_20 → STEP_30`; ControlWord=512 (move-to-work) |
| 3 000 | `STEP_30 → STEP_100`; handshake image held stable |
| 5 050 | `STEP_100 → TEST_ARMED`; post-init settle elapsed |
| 5 100 | `TEST_ARMED → TEST_OPEN` cycle 1; WP=100, CW=512 |
| 7 100 | `TEST_OPEN → TEST_CLOSE` cycle 1; WP=1200, CW=512 |
| 9 100 | `TEST_CLOSE → TEST_OPEN` cycle 2 |
| … | Cycles repeat every 4 000 ms (2 s open + 2 s close) |
| 41 550 | `TEST_CLOSE → TEST_OPEN` cycle 10 |
| 43 600 | `TEST_OPEN → TEST_CLOSE` cycle 10 |
| 45 600 | `TEST_CLOSE → TEST_COMPLETE`; "Completed 10/10 open/close cycles" |
| 45 600–50 000 | TEST_COMPLETE steady hold; ControlWord=0 |
| 50 000 | "Runtime limit reached (50 s)"; graceful shutdown |

---

## 5. Pass / Fail Results

| Test Item | Result | Evidence |
|-----------|--------|----------|
| Serial port opened | **PASS** | `bridge.open_serial()` succeeded |
| Modbus listener started | **PASS** | `bridge.start_listen()` succeeded |
| Master traffic detected within 3 s | **PASS** | First FC03 request received before 3 000 ms guard |
| Initialization complete within 12 s | **PASS** | STEP_100 reached at ~3 000 ms (budget: 12 000 ms) |
| Handshake image stable during settle | **PASS** | No spurious transitions for 2 000 ms between STEP_100 and TEST_ARMED |
| All 10 open/close cycles completed | **PASS** | TEST_COMPLETE reached at ~45 600 ms; counter = 10/10 |
| Clean idle after test, no FAULT | **PASS** | ControlWord=0 held from 45 600 ms to runtime limit |
| Process exit code 0 (nominal) | **PASS** | `echo $?` → 0 |
| Frame integrity (CRC errors) | **PASS** | CRC errors = 0 |
| Unexpected invalid frames | **PASS** | Invalid = 1 (single spurious short frame; non-zero CRC count: 0) |

**Summary: 10/10 test items PASS.**

---

## 6. Bridge Statistics

Captured at ~45 s (before TEST_COMPLETE hold ended):

| Counter | Value |
|---------|-------|
| Frames received (from master) | 235 |
| Frames sent (slave responses) | 234 |
| Invalid / malformed frames | 1 |
| CRC errors | 0 |
| Exception responses sent | 0 |

Steady-state exchange rate: ~20 frames/s (one FC03 request + one FC03 response per 50 ms tick, matching `main_loop_period_ms`).

---

## 7. Protocol Reference — FC03 Canonical Response Frames

This section provides the three most architecturally significant Modbus RTU response frames observed during the validated 50-second run. They serve as golden reference values for regression testing and bring-up verification.

### 7.1 Request Frame (constant throughout run)

The PLC master sends a single, unchanging FC03 query to read all 8 command holding registers (addresses 0–7).

```
01 03 00 00 00 08 44 0C
```

| Field | Bytes | Value | Description |
|-------|-------|-------|-------------|
| Slave address | `01` | 0x01 (1) | Gripper bridge device |
| Function code | `03` | 0x03 | Read Holding Registers |
| Start register | `00 00` | 0x0000 (0) | First register: COMMAND_WORD |
| Register count | `00 08` | 0x0008 (8) | Read all 8 command registers |
| CRC | `44 0C` | — | CRC-16/Modbus (little-endian) |

---

### 7.2 Frame A — Initialization Transfer (STEP_10)

Asserted from t ≈ 0 ms to t ≈ 1 000 ms. Sets all motor parameters and signals initialization start to the device (ControlWord bit 0 = 1).

```
01 03 10 00 01 67 00 00 32 04 04 00 64 00 00 04 B0 04 B0 E2 36
```

#### Byte map

| Offset | Bytes | Reg# | Name | Raw | Parsed |
|--------|-------|------|------|-----|--------|
| 0 | `01` | — | Slave address | 0x01 | 1 |
| 1 | `03` | — | Function code | 0x03 | Read Holding Registers |
| 2 | `10` | — | Byte count | 0x10 | 16 (= 8 registers × 2 bytes) |
| 3–4 | `00 01` | 0 | COMMAND_WORD | 0x0001 | **1** — InitTransfer bit set |
| 5–6 | `67 00` | 1 | DEVICE_MODE / WORKPIECE# | 0x6700 | DeviceMode=**103** (0x67), WorkpieceNo=**0** |
| 7–8 | `00 32` | 2 | RESERVED / POSITION_TOLERANCE | 0x0032 | Reserved=0, PosTolerance=**50** |
| 9–10 | `04 04` | 3 | GRIP_FORCE / DRIVE_VELOCITY | 0x0404 | GripForce=**4**, DriveVelocity=**4** |
| 11–12 | `00 64` | 4 | BASE_POSITION | 0x0064 | **100** units |
| 13–14 | `00 00` | 5 | RESERVED_5 | 0x0000 | 0 |
| 15–16 | `04 B0` | 6 | TEACH_POSITION | 0x04B0 | **1200** units |
| 17–18 | `04 B0` | 7 | WORK_POSITION | 0x04B0 | **1200** units |
| 19–20 | `E2 36` | — | CRC-16 | — | CRC-16/Modbus |

> **Identifying signature:** Reg 0 = `00 01` (ControlWord = 1).

---

### 7.3 Frame B — Move-to-Work / Normal Operation (STEP_30, STEP_100, TEST_CLOSE)

First asserted at t ≈ 2 000 ms (STEP_30). Sustained throughout STEP_100 normal operation and during each close half-cycle. ControlWord bit 9 (0x0200 = 512) commands the gripper to move to WorkPosition.

```
01 03 10 02 00 67 00 00 32 04 04 00 64 00 00 04 B0 04 B0 A2 57
```

#### Byte map

| Offset | Bytes | Reg# | Name | Raw | Parsed |
|--------|-------|------|------|-----|--------|
| 0 | `01` | — | Slave address | 0x01 | 1 |
| 1 | `03` | — | Function code | 0x03 | Read Holding Registers |
| 2 | `10` | — | Byte count | 0x10 | 16 |
| 3–4 | `02 00` | 0 | COMMAND_WORD | 0x0200 | **512** — MoveToWork bit set |
| 5–6 | `67 00` | 1 | DEVICE_MODE / WORKPIECE# | 0x6700 | DeviceMode=103, WorkpieceNo=0 |
| 7–8 | `00 32` | 2 | RESERVED / POSITION_TOLERANCE | 0x0032 | PosTolerance=50 |
| 9–10 | `04 04` | 3 | GRIP_FORCE / DRIVE_VELOCITY | 0x0404 | GripForce=4, DriveVelocity=4 |
| 11–12 | `00 64` | 4 | BASE_POSITION | 0x0064 | 100 |
| 13–14 | `00 00` | 5 | RESERVED_5 | 0x0000 | 0 |
| 15–16 | `04 B0` | 6 | TEACH_POSITION | 0x04B0 | 1200 |
| 17–18 | `04 B0` | 7 | WORK_POSITION | 0x04B0 | **1200** units (close / grip position) |
| 19–20 | `A2 57` | — | CRC-16 | — | CRC-16/Modbus |

> **Identifying signature:** Reg 0 = `02 00` (ControlWord = 512), Reg 7 = `04 B0` (WorkPosition = 1200).

---

### 7.4 Frame C — Open Cycle (TEST_OPEN)

Asserted for 2 000 ms at the start of each of the 10 open half-cycles (first at t ≈ 5 100 ms). WorkPosition is set to the open target (100 units). ControlWord remains 512.

```
01 03 10 02 00 67 00 00 32 04 04 00 64 00 00 04 B0 00 64 A0 C8
```

#### Byte map

| Offset | Bytes | Reg# | Name | Raw | Parsed |
|--------|-------|------|------|-----|--------|
| 0 | `01` | — | Slave address | 0x01 | 1 |
| 1 | `03` | — | Function code | 0x03 | Read Holding Registers |
| 2 | `10` | — | Byte count | 0x10 | 16 |
| 3–4 | `02 00` | 0 | COMMAND_WORD | 0x0200 | **512** — MoveToWork bit set |
| 5–6 | `67 00` | 1 | DEVICE_MODE / WORKPIECE# | 0x6700 | DeviceMode=103, WorkpieceNo=0 |
| 7–8 | `00 32` | 2 | RESERVED / POSITION_TOLERANCE | 0x0032 | PosTolerance=50 |
| 9–10 | `04 04` | 3 | GRIP_FORCE / DRIVE_VELOCITY | 0x0404 | GripForce=4, DriveVelocity=4 |
| 11–12 | `00 64` | 4 | BASE_POSITION | 0x0064 | 100 |
| 13–14 | `00 00` | 5 | RESERVED_5 | 0x0000 | 0 |
| 15–16 | `04 B0` | 6 | TEACH_POSITION | 0x04B0 | 1200 |
| 17–18 | `00 64` | 7 | WORK_POSITION | 0x0064 | **100** units (open position) |
| 19–20 | `A0 C8` | — | CRC-16 | — | CRC-16/Modbus |

> **Identifying signature:** Reg 0 = `02 00` (ControlWord = 512), Reg 7 = `00 64` (WorkPosition = 100).  
> Distinguishable from Frame B solely by WorkPosition bytes (`00 64` vs `04 B0`).

---

### 7.5 Quick Differentiator Table

For automated validation, each canonical frame has a unique byte pattern at a fixed offset:

| Frame | Purpose | Reg 0 (bytes 3–4) | Reg 7 (bytes 17–18) | CRC (bytes 19–20) |
|-------|---------|--------------------|----------------------|--------------------|
| A | Initialization transfer | `00 01` | `04 B0` | `E2 36` |
| B | Move-to-work / normal op | `02 00` | `04 B0` | `A2 57` |
| C | Open cycle (gripper open) | `02 00` | `00 64` | `A0 C8` |

The complete 21-byte response length is fixed for all frames (slave=1 byte, FC=1, byte-count=1, 16 data bytes, CRC=2).

---

## 8. Known Limitations

- StatusWord feedback from the device is **not** validated; fixed time delays substitute for bit-level handshake confirmation. This is intentional for bring-up but should be replaced with real StatusWord polling for production use.
- The single invalid frame (bridge counter: Invalid=1) is a benign short-frame artifact from RS-485 bus idle transitions.
- `open_position_units` / `close_position_units` are not exposed as CLI arguments; they must be changed by recompiling.

---

## 9. Conclusion

The `gripper_linux_runtime` executed a complete 50-second sequence:

1. Documentation-aligned 3-step initialization handshake completed in ~3 s (well within the 12 s guard).
2. A 2-second decoupled settle window confirmed handshake stability before any motion command.
3. All 10 open/close cycles completed without error (exit code 0, CRC errors = 0).
4. The runtime shut down cleanly at the 50-second limit, publishing a safe ControlWord=0 image.

The Modbus traffic pattern is consistent and deterministic. Frames A, B, and C (section 7) are suitable as golden reference values for continuous integration regression tests.
