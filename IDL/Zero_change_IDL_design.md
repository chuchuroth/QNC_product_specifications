# Generic Gripper Interface Layer (Zero-IDL-Change Design)

## Assumptions

1. IDL contracts `qnc::gripper::GripperCommand`, `qnc::gripper::GripperState`, and `qnc::gripper::GripperCapabilities` are fixed and cannot change.
2. Existing DDS topics and upper-layer behavior stay unchanged.
3. The QNC bridge consumes `qnc::gripper::GripperCommand` directly and emits `qnc::gripper::GripperState`, using the raw transport fields when descriptor-free Modbus access is needed.
4. One JSON descriptor is loaded per `device_id`.

## 1. Architecture Design

### Component Diagram (Text)

- `Upper Application`
- `DDS Subscriber (GripperCommand)`
- `GenericGripperAdapter`
- `DescriptorCache (JSON schema + per-device config)`
- `ModbusCommandBuilder (WriteMultipleCommand)`
- `QNC Modbus Bridge` (already existing)
- `DDS Publisher (GripperState)`

### Data Flow

1. DDS receives `qnc::gripper::GripperCommand`.
2. Adapter parses command normalized fields (`position`, `max_force`, `max_speed`) plus optional JSON payload conventions.
3. Adapter reads device config by `device_id`.
4. Adapter maps normalized values to raw register values using scaling rules.
5. Adapter translates normalized commands into direct Modbus RTU write/read operations inside the bridge.
6. Adapter reconstructs `qnc::gripper::GripperState` from transport snapshots and normalized descriptor reads.
7. DDS publishes normalized generic state on unchanged topics.

## 2. DDS Type Generation

### Preferred path (NeuraSync toolchain)

Use the same generator version used for your committed Modbus types (FastDDS-Gen v4.x).

```bash
cd neurasync-robot-client
mkdir -p neurasync/generated/gripper
fastddsgen -replace -d neurasync/generated/gripper neurasync/idl/GripperCapabilities.idl
fastddsgen -replace -d neurasync/generated/gripper neurasync/idl/GripperCommand.idl
fastddsgen -replace -d neurasync/generated/gripper neurasync/idl/GripperState.idl
```

### Note from this workspace

`GripperCapabilities.idl` generates, but local `/usr/bin/fastddsgen` fails parsing `GripperCommand.idl` enum assignments. This indicates a generator-version mismatch, not a runtime design issue. Use the same FastDDS-Gen baseline as your NeuraSync CI image or NeuraSync IDL tools container.

## 3. JSON Schema

Primary schema file:

- `device_descriptors/GenericGripperDescriptor_Schema.json`

Device examples:

- `device_descriptors/generic/DH-Robotics_CGC-80_generic_gripper.json`
- `device_descriptors/generic/DH-Robotics_DH-5-6_generic_gripper.json`

## 4. Mapping Table

| Generic IDL Field | JSON Config Path | Modbus Mapping |
|---|---|---|
| `GripperCommand.device_id` | `device.device_id` | Select slave/config route |
| `GripperCommand.position` | `scaling.position` + `register_map.position.write` | normalized [0..1] -> raw -> FC06/FC16 write |
| `GripperCommand.max_force` | `scaling.force` + `register_map.force.write` | normalized [0..1] -> raw -> FC06/FC16 write |
| `GripperCommand.max_speed` | `scaling.speed` + `register_map.speed.write` | normalized [0..1] -> raw -> FC06/FC16 write |
| `GripperState.position` | `register_map.position.read` + `scaling.position` | raw read -> normalized [0..1] |
| `GripperState.force` | `register_map.force.read` + `scaling.force` | raw read -> normalized [0..1] |
| `GripperState.speed` | `register_map.speed.read` + `scaling.speed` | raw read -> normalized [0..1] |
| `GripperState.status` | `register_map.status.motion` + `state_map.motion_codes` | status register code -> enum |
| `GripperState.grasp_result` | `register_map.status.grasp` + `state_map.grasp_codes` | code -> grasp enum |
| `GripperState.error_code` | `register_map.status.fault` | QNC code or device code |

## 5. Adaptation Layer Pseudocode (C++)

Reference file:

- `deploy_on_qnc/generic_gripper_adapter_pseudocode.cpp`

Coverage:

- DDS subscription handling (`on_command`)
- JSON-driven mapping of normalized values -> registers
- `WriteMultipleCommand` creation
- reverse mapping from `Response` + `BridgeStats` -> `GripperState`

## 6. Multi-Joint Handling (DH-5-6)

When `capabilities.has_multi_joint = true`, adapter behavior is configured by `mode_strategy`:

- `aggregate`: apply one normalized value to all DOFs.
- `preset_trigger`: map normalized `position` to a preset register trigger.
- `per_joint`: read advanced JSON payload keys (`dof_position`, `dof_force`, `dof_speed`) and map per DOF registers.

Advanced users can pass per-device conventions in command JSON payload while still using unchanged generic IDL and topic names.

## 7. Device-Specific Examples

### CGC-80 (single-axis)

- `position=0.8` -> raw `800` -> register `259`
- `force=0.6` -> raw `68` (20..100 scaling) -> register `257`
- `speed=0.6` -> raw `60` (1..100 scaling) -> register `260`

### DH-5-6 (multi-DOF)

- `position=0.8` with `preset_trigger` -> `power_grasp` register value `800`
- `force=0.6` -> 68% for each configured DOF force register
- `speed=0.6` -> 60% for each configured DOF speed register

## 8. Extensibility (No IDL / Topic / App Changes)

To add a new gripper:

1. Copy a generic descriptor example JSON.
2. Update register addresses, function codes, scaling, and state code maps.
3. Set capability flags (`supports_force_control`, `supports_speed_control`, `has_multi_joint`).
4. Deploy descriptor and reload adapter config.

No IDL changes, no topic changes, no upper-layer logic changes, and no per-device C++ code changes are required.
