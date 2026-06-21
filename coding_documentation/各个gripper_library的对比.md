+ rtt_binzel_app : 这是moritz几年前开发的，all gripper通用

+ https://gitlab.hrg.systems/control_software/robot_communication/libraries/universal_gripper_library


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
