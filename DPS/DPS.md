# [QNC](https://www.genspark.ai/api/files/s/MQZoL9fC) Device Profile Specification  
**Document Number:** QNC-DPS-001  
**Version:** 1.0  
**Status:** Proposed Baseline Standard  
**Classification:** Internal / Project Controlled  

---

## Document Control

| Version | Date | Author | Change Summary |
|---|---:|---|---|
| 1.0 | 2026-03-25 | Genspark Draft Rewrite | Initial companion Device Profile Specification aligned to revised PRD/SDS and ICD |

### Document Approvals

| Role | Name | Status |
|---|---|---|
| Product Manager | TBD | Pending |
| Systems Architect | TBD | Pending |
| Firmware Lead | TBD | Pending |
| Robotics Software Lead | TBD | Pending |
| QA / Validation Lead | TBD | Pending |

---

## Table of Contents

1. Purpose and Scope  
2. Normative Language  
3. Related Documents  
4. Profile Definition and Role  
5. Profile Architecture and Boundary  
6. Profile Classes and Support Model  
7. Device Profile Artifact Structure  
8. Metadata and Identity Requirements  
9. Protocol Binding Requirements  
10. Capability Model  
11. Command Definition Model  
12. Parameter Definition Model  
13. Telemetry Mapping Model  
14. Fault Mapping Model  
15. Lifecycle Sequences  
16. Compatibility and Versioning  
17. Validation and Approval Requirements  
18. Security and Integrity Requirements  
19. Operational Lifecycle of Profiles  
20. Schema Rules and Constraints  
21. Conformance Requirements  
Appendix A. Normative YAML Template  
Appendix B. Example Gripper Profile  
Appendix C. Validation Checklist  
Appendix D. Field Dictionary  

---

# 1. Purpose and Scope

## 1.1 Purpose

This Device Profile Specification defines the **structure, content, validation rules, compatibility rules, lifecycle rules, and governance requirements** for all device profiles used by [QNC](https://www.genspark.ai/api/files/s/MQZoL9fC).

A device profile is the controlled artifact that allows QNC to support a device within an approved protocol and device-class boundary without requiring ad hoc device-specific firmware behavior. This directly supports the original project intent of using declarative descriptors rather than uncontrolled device-specific code, while correcting the ambiguity and under-specification present in the uploaded source. [Source](https://www.genspark.ai/api/files/s/MQZoL9fC)

## 1.2 Scope

This specification governs:
- the format of QNC device profiles,
- the required and optional fields in each profile,
- how profiles define supported commands and telemetry,
- how profiles map device-specific protocol details into normalized QNC behavior,
- how profiles are validated, versioned, approved, deployed, and retired.

## 1.3 Out of Scope

This document does not define:
- the product strategy or scope of QNC,
- the robot-facing message transport contract,
- low-level protocol frame syntax,
- hardware connector drawings,
- certified safety logic,
- unsupported device semantics.

Those are defined in the companion PRD/SDS and ICD.

---

# 2. Normative Language

The key words **SHALL**, **SHOULD**, **MAY**, and **MUST NOT** in this document are to be interpreted as follows:

- **SHALL**: mandatory requirement.
- **SHOULD**: recommended unless exception is formally approved.
- **MAY**: optional.
- **MUST NOT**: prohibited.

---

# 3. Related Documents

This specification SHALL be used together with:
- **QNC Product Requirements & System Definition Specification (PRD/SDS), v2.0**
- **QNC Interface Control Document (ICD), v2.0**
- **QNC Verification & Validation Plan (to be issued)**
- **QNC Operations & Recovery Guide (to be issued)**

Where conflict exists:
1. the PRD/SDS governs product boundary and release scope,
2. the ICD governs system interfaces and behaviors,
3. this Device Profile Specification governs the profile artifact and its semantics.

---

# 4. Profile Definition and Role

## 4.1 Formal Definition

A **QNC Device Profile** is a **versioned, validated, controlled artifact** that defines how a specific supported device, or a bounded family of devices, is represented and operated within QNC.

A profile SHALL define:
- device identity and support scope,
- protocol binding and communication assumptions,
- initialization behavior,
- supported capabilities,
- command mappings,
- parameter constraints,
- telemetry mappings,
- fault mappings,
- compatibility metadata,
- validation status.

## 4.2 Role in the QNC Architecture

The device profile is the formal bridge between:
- **device-specific protocol details**, and
- **QNC’s normalized integration model**.

The profile SHALL allow QNC to expose approved device-class semantics without claiming universal semantics across unrelated devices. This is important because the original source blurred the line between “protocol-only” behavior and device-aware behavior; the profile system is where that device-aware mapping must be made explicit and controlled. [Source](https://www.genspark.ai/api/files/s/MQZoL9fC)

## 4.3 Architectural Boundary

A profile SHALL:
- describe and constrain supported device behavior,
- define how QNC interacts with a supported device,
- define mappings into QNC’s integration model.

A profile SHALL NOT:
- redefine QNC system-level lifecycle states,
- change robot-facing interface contracts,
- introduce unsupported protocol families,
- override product safety boundary,
- claim support for uncertified or unvalidated devices.

---

# 5. Profile Architecture and Boundary

## 5.1 Layered Model

Each device profile SHALL be structured across five logical layers:

1. **Identity Layer**  
   Defines what device or device family the profile applies to.

2. **Binding Layer**  
   Defines which protocol and communication assumptions apply.

3. **Capability Layer**  
   Defines what functional behaviors the device exposes in QNC terms.

4. **Mapping Layer**  
   Defines how commands, telemetry, and faults map between device protocol details and QNC abstractions.

5. **Governance Layer**  
   Defines validation, compatibility, signing, and lifecycle information.

## 5.2 Separation of Concerns

The profile SHALL not contain arbitrary application logic.  
The profile SHALL contain only:
- declarative mappings,
- bounded sequences,
- constrained metadata,
- enumerated rules supported by the QNC runtime.

This prevents the profile from becoming an uncontrolled scripting mechanism.

---

# 6. Profile Classes and Support Model

## 6.1 Device Class Requirement

Every profile SHALL declare exactly one **device class**. Baseline supported classes are:

- `GRIPPER_ELECTRIC`
- `ACTUATOR_SIMPLE`
- `SENSOR_INDUSTRIAL`

Additional classes require formal change approval.

## 6.2 Support Granularity

Profiles MAY be one of the following:
- **Device-Specific Profile**: supports exactly one model/variant.
- **Device-Family Profile**: supports multiple variants within a bounded compatibility range.
- **Vendor-Profile Template**: reusable internal basis for approved derivative profiles; not deployable as a final production profile by itself.

## 6.3 Support Declaration

A device SHALL be considered supported only if:
- a profile exists,
- the profile is valid,
- the profile is approved,
- compatibility requirements are satisfied,
- release status is published.

A device SHALL NOT be considered supported based solely on protocol reachability.

---

# 7. Device Profile Artifact Structure

## 7.1 Artifact Format

The normative profile artifact format SHALL be **YAML** for authoring and review.  
QNC MAY internally convert profiles into another validated representation for runtime use.

## 7.2 Artifact Sections

Every profile SHALL contain the following top-level sections:

```yaml
profile_metadata:
device_identity:
protocol_binding:
capabilities:
commands:
parameters:
telemetry:
faults:
lifecycle_sequences:
compatibility:
validation:
security:
```

## 7.3 Required Top-Level Fields

| Section | Required | Purpose |
|---|---:|---|
| `profile_metadata` | Yes | Identity, ownership, versioning |
| `device_identity` | Yes | Supported device/model scope |
| `protocol_binding` | Yes | Protocol-level assumptions |
| `capabilities` | Yes | Supported device-class functions |
| `commands` | Yes | Command catalog and mappings |
| `parameters` | Yes | Allowed parameter definitions |
| `telemetry` | Yes | State/measurement mappings |
| `faults` | Yes | Fault and warning mappings |
| `lifecycle_sequences` | Yes | Init/enable/disable/recovery sequences |
| `compatibility` | Yes | Version and platform constraints |
| `validation` | Yes | Approval and test metadata |
| `security` | Yes | Integrity and signature policy |

---

# 8. Metadata and Identity Requirements

## 8.1 Metadata Fields

`profile_metadata` SHALL contain at minimum:

| Field | Type | Required | Description |
|---|---|---:|---|
| `profile_id` | string | Yes | Unique immutable profile identifier |
| `profile_name` | string | Yes | Human-readable profile name |
| `profile_version` | string | Yes | Semantic version of profile |
| `profile_status` | enum | Yes | Draft / Validated / Released / Deprecated / Retired |
| `owner_team` | string | Yes | Responsible team |
| `created_utc` | string | Yes | ISO 8601 creation timestamp |
| `updated_utc` | string | Yes | ISO 8601 last update timestamp |
| `change_summary` | string | Yes | Short change description |
| `document_ref` | string | No | Reference to supporting design/test docs |

## 8.2 Device Identity Fields

`device_identity` SHALL contain at minimum:

| Field | Type | Required | Description |
|---|---|---:|---|
| `vendor` | string | Yes | Device vendor name |
| `model` | string | Yes | Device model identifier |
| `device_class` | enum | Yes | QNC device class |
| `variants` | list | No | Supported variants |
| `hardware_revisions` | list | No | Supported hardware revisions |
| `firmware_revisions` | list | No | Supported vendor firmware revisions |
| `device_ids` | list | No | Device identifiers detectable on protocol |
| `identity_notes` | string | No | Clarifying notes |

## 8.3 Identity Match Policy

A profile SHALL define how identity is established:
- exact match,
- bounded family match,
- protocol signature match,
- operator-approved manual binding.

The match policy SHALL be explicit.  
No implicit support claim is permitted.

---

# 9. Protocol Binding Requirements

## 9.1 Protocol Declaration

Each profile SHALL declare exactly one baseline protocol binding:
- `MODBUS_RTU`
- `IO_LINK`

## 9.2 Protocol Binding Fields

`protocol_binding` SHALL contain:

| Field | Type | Required | Description |
|---|---|---:|---|
| `protocol` | enum | Yes | Bound protocol |
| `transport_profile` | string | Yes | Named transport profile |
| `addressing_mode` | string | Yes | How target addressing is handled |
| `session_requirements` | object | Yes | Session and timing assumptions |
| `communication_defaults` | object | Yes | Default protocol parameters |
| `discovery_hints` | object | No | Detection hints if allowed |
| `protocol_notes` | string | No | Additional protocol considerations |

## 9.3 Binding Rules

A profile SHALL NOT:
- declare unsupported protocol families,
- assume protocol behavior not represented in the ICD,
- require runtime scripting to complete normal protocol operations.

## 9.4 Communication Defaults

Communication defaults MAY include:
- serial settings,
- timeout classes,
- polling intervals,
- retry policy classes,
- data packing rules.

Defaults SHALL be explicit and bounded.

---

# 10. Capability Model

## 10.1 Purpose

The capability model defines what the device can do in **QNC terms**, independent of raw register or frame details.

## 10.2 Capability Requirements

Each profile SHALL define:
- supported capability set,
- whether each capability is mandatory or optional,
- whether each capability is blocking or asynchronous,
- whether each capability is safe in degraded mode.

## 10.3 Example Capability Categories

For `GRIPPER_ELECTRIC`, capability categories MAY include:
- `enable_device`
- `disable_device`
- `move_to_position`
- `set_force`
- `set_speed`
- `stop_motion`
- `read_position`
- `read_status`
- `clear_recoverable_fault`

For `SENSOR_INDUSTRIAL`, capability categories MAY include:
- `read_measurement`
- `read_device_status`
- `read_diagnostics`
- `set_sampling_mode`

## 10.4 Capability Declaration Rules

Capabilities SHALL be declared as:
- normalized capability name,
- device-native mapping reference,
- constraints,
- availability conditions.

Capabilities SHALL NOT imply functions that are only available through undocumented vendor-specific operations.

---

# 11. Command Definition Model

## 11.1 Purpose

The `commands` section defines how normalized QNC commands map to device-native protocol interactions.

## 11.2 Command Entry Structure

Each command SHALL define:

| Field | Type | Required | Description |
|---|---|---:|---|
| `command_name` | string | Yes | Normalized command name |
| `command_type` | enum | Yes | Profile / transport / management |
| `capability_ref` | string | Yes | Linked capability |
| `description` | string | Yes | Human-readable behavior |
| `execution_mode` | enum | Yes | Sync / Async |
| `allowed_states` | list | Yes | QNC states where command is valid |
| `parameter_refs` | list | Yes | Parameter definitions used |
| `device_mapping` | object | Yes | Protocol-native mapping |
| `completion_criteria` | object | Yes | Success/failure determination |
| `timeout_class` | string | Yes | Timeout class |
| `retry_class` | string | Yes | Retry behavior |
| `safety_behavior` | object | No | Restrictions under safe/degraded modes |

## 11.3 Device Mapping Rules

The `device_mapping` object SHALL be declarative and MAY define:
- register addresses,
- indices/subindices,
- bit fields,
- write/read sequences,
- expected acknowledgments,
- response parse rules.

The mapping SHALL be deterministic and finite.

## 11.4 Atomicity Rules

A profile SHALL explicitly state whether a command is:
- single-transaction,
- multi-step but atomic from QNC’s perspective,
- multi-step and non-atomic.

If a command is non-atomic, the profile SHALL define:
- intermediate failure behavior,
- rollback or compensation policy if applicable,
- status reporting rules.

This is necessary because the original source described combined operations without clearly defining atomicity. [Source](https://www.genspark.ai/api/files/s/MQZoL9fC)

---

# 12. Parameter Definition Model

## 12.1 Purpose

The `parameters` section defines all allowable command and telemetry parameters referenced elsewhere in the profile.

## 12.2 Parameter Fields

Each parameter SHALL define:

| Field | Type | Required | Description |
|---|---|---:|---|
| `parameter_name` | string | Yes | Unique parameter identifier |
| `data_type` | enum | Yes | Integer / float / bool / enum / string |
| `unit` | string | No | Engineering unit |
| `required` | boolean | Yes | Whether required in command context |
| `default` | any | No | Default value |
| `minimum` | number | No | Lower bound |
| `maximum` | number | No | Upper bound |
| `allowed_values` | list | No | Enumerated legal values |
| `precision` | number | No | Required resolution |
| `scaling` | object | No | Scaling/transformation rule |
| `validation_rules` | list | No | Additional constraints |
| `description` | string | Yes | Meaning and usage |

## 12.3 Parameter Constraint Rules

Parameters SHALL be validated by QNC prior to execution.  
Profiles SHALL NOT define unbounded parameters for actuation commands.  
Units SHALL be explicit where applicable.

## 12.4 Engineering Units

Engineering units SHOULD use standard SI-based notation where feasible.  
If vendor-native units differ, the profile SHALL define:
- normalized unit,
- device-native unit,
- transformation rule.

---

# 13. Telemetry Mapping Model

## 13.1 Purpose

The `telemetry` section defines how device-native status and measurement data map into QNC telemetry.

## 13.2 Telemetry Categories

Profiles SHALL classify telemetry into:
- `core_status`
- `measurements`
- `device_class_state`
- `vendor_specific_state`
- `diagnostic_state`

## 13.3 Telemetry Entry Structure

Each telemetry item SHALL define:

| Field | Type | Required | Description |
|---|---|---:|---|
| `field_name` | string | Yes | QNC-visible field name |
| `category` | enum | Yes | Telemetry category |
| `data_type` | enum | Yes | Value type |
| `unit` | string | No | Engineering unit |
| `source_mapping` | object | Yes | Protocol-native source |
| `normalization` | object | No | Transformation into normalized value |
| `validity_rules` | object | No | Conditions for trusting the value |
| `publication_policy` | object | Yes | Event-driven or periodic behavior |
| `description` | string | Yes | Meaning and interpretation |

## 13.4 State Semantics Rules

Profiles SHALL distinguish clearly between:
- normalized class-level semantics,
- raw vendor-specific fields.

A field like `object_secured` MAY exist in a gripper profile if clearly defined as gripper-class semantics.  
It SHALL NOT be inserted into the universal QNC core state model without class scoping.

## 13.5 Quality Flags

Telemetry items MAY define quality flags such as:
- valid,
- stale,
- estimated,
- unsupported,
- faulted.

Where quality flags exist, the profile SHALL define when each flag applies.

---

# 14. Fault Mapping Model

## 14.1 Purpose

The `faults` section defines how device-reported or protocol-observed abnormal conditions map into QNC warnings and faults.

## 14.2 Fault Mapping Entry Structure

Each fault mapping SHALL define:

| Field | Type | Required | Description |
|---|---|---:|---|
| `source_fault_id` | string | Yes | Vendor/native fault identifier |
| `qnc_fault_code` | string | Yes | Mapped QNC fault code |
| `severity` | enum | Yes | Warning / Error / Critical |
| `latched` | boolean | Yes | Whether latching behavior applies |
| `source_layer` | enum | Yes | Protocol / device / profile |
| `summary` | string | Yes | Human-readable summary |
| `recovery_guidance` | string | No | Recommended action |
| `safe_mode_required` | boolean | Yes | Whether QNC must enter safe mode |
| `notes` | string | No | Clarifying notes |

## 14.3 Fault Mapping Rules

A profile SHALL:
- map all known vendor/device faults relevant to supported operations,
- distinguish recoverable and non-recoverable faults,
- define behavior for unknown/unmapped device faults.

Unknown vendor faults SHALL NOT be silently ignored.

## 14.4 Safety-Relevant Faults

If a profile identifies a device condition as safety-relevant, it SHALL:
- mark it explicitly,
- define QNC local inhibit behavior,
- not redefine system-level safety ownership.

---

# 15. Lifecycle Sequences

## 15.1 Purpose

The `lifecycle_sequences` section defines bounded, declarative sequences that QNC may execute for a profile.

## 15.2 Required Sequence Types

Each profile SHALL define or explicitly declare not applicable for:
- `initialization_sequence`
- `enable_sequence`
- `disable_sequence`
- `recover_sequence`
- `shutdown_sequence`

## 15.3 Sequence Entry Structure

Each sequence step SHALL define:

| Field | Type | Required | Description |
|---|---|---:|---|
| `step_id` | string | Yes | Unique step identifier |
| `action_type` | enum | Yes | Read / Write / Wait / Verify / Delay |
| `mapping` | object | Yes | Protocol-native action details |
| `expected_result` | object | No | Required response or state |
| `timeout_class` | string | Yes | Timeout policy |
| `on_failure` | enum | Yes | Abort / Retry / Degrade / SafeMode |
| `description` | string | Yes | Purpose of step |

## 15.4 Sequence Constraints

Sequences SHALL be:
- finite,
- deterministic,
- bounded in time,
- supported by the runtime.

Profiles SHALL NOT include arbitrary code execution or unbounded loops.

---

# 16. Compatibility and Versioning

## 16.1 Compatibility Requirements

The `compatibility` section SHALL define:
- minimum QNC firmware version,
- maximum validated QNC firmware version if bounded,
- required interface schema version,
- protocol adapter version constraints,
- supported device revisions,
- incompatibility notes.

## 16.2 Versioning Policy

Profile versions SHALL use **semantic versioning**:
- major = breaking mapping or behavior change,
- minor = backward-compatible capability or mapping update,
- patch = non-breaking correction or metadata update.

## 16.3 Compatibility States

A profile MAY be in one of the following compatibility states relative to a QNC installation:
- compatible,
- conditionally compatible,
- incompatible,
- unknown.

QNC SHALL refuse activation of an incompatible profile.

---

# 17. Validation and Approval Requirements

## 17.1 Validation Fields

The `validation` section SHALL include:

| Field | Type | Required | Description |
|---|---|---:|---|
| `validation_status` | enum | Yes | Draft / InReview / Validated / Released / Deprecated |
| `test_evidence_ref` | list | Yes | References to test evidence |
| `review_approvals` | list | Yes | Approval metadata |
| `known_limitations` | list | No | Published limitations |
| `last_validated_utc` | string | Yes | ISO timestamp |
| `validated_by` | string | Yes | Responsible validation authority |

## 17.2 Approval Policy

A profile SHALL NOT be marked `Released` unless:
- schema validation passed,
- integration tests passed,
- fault handling tests passed,
- compatibility checks passed,
- required stakeholders approved.

## 17.3 Limitation Disclosure

Profiles SHALL declare known limitations, including:
- unsupported device modes,
- reduced capability variants,
- vendor firmware constraints,
- performance caveats.

---

# 18. Security and Integrity Requirements

## 18.1 Security Section

The `security` section SHALL define:
- signature requirement,
- artifact hash,
- signing authority identifier,
- integrity policy,
- tamper response policy.

## 18.2 Signature Policy

Production profiles SHALL be signed.  
Unsigned profiles MAY be allowed only in explicitly authorized development environments.

## 18.3 Integrity Failure Behavior

If profile integrity verification fails, QNC SHALL:
- reject activation,
- log the event,
- emit a security/integrity fault,
- retain the last known valid active configuration where possible.

---

# 19. Operational Lifecycle of Profiles

## 19.1 Lifecycle States

Profiles SHALL move through the following lifecycle states:

- `DRAFT`
- `UNDER_REVIEW`
- `VALIDATED`
- `RELEASED`
- `DEPRECATED`
- `RETIRED`

## 19.2 Allowed State Transitions

| From | To | Allowed |
|---|---|---|
| Draft | Under_Review | Yes |
| Under_Review | Validated | Yes |
| Validated | Released | Yes |
| Released | Deprecated | Yes |
| Deprecated | Retired | Yes |
| Released | Draft | No |

## 19.3 Deprecation Policy

Deprecated profiles SHALL:
- remain installable only per policy,
- include deprecation notice,
- identify successor profile if one exists,
- define support sunset expectation.

---

# 20. Schema Rules and Constraints

## 20.1 General Schema Rules

All fields SHALL have:
- explicit type,
- explicit cardinality,
- explicit meaning.

Unknown top-level fields SHALL be rejected unless the schema explicitly allows extension namespaces.

## 20.2 Extension Policy

Vendor or project-specific extensions MAY be allowed only under:
- a namespaced extension block,
- documented semantics,
- non-conflict with core schema,
- explicit runtime support declaration.

## 20.3 Determinism Rule

A valid profile SHALL be fully evaluable without:
- network lookups,
- external script execution,
- ambiguous fallbacks,
- hidden dependencies.

---

# 21. Conformance Requirements

A profile SHALL be considered conformant to this specification only if:
- all mandatory sections are present,
- all schema rules pass,
- protocol binding is supported,
- command and telemetry mappings are internally consistent,
- compatibility metadata is complete,
- validation metadata is complete,
- integrity/signature policy is satisfied.

Conformance to this specification does not by itself imply release support; it only establishes that the artifact is structurally valid and governable.

---

# Appendix A. Normative YAML Template

```yaml
profile_metadata:
  profile_id: "qnc.gripper.vendor_model"
  profile_name: "Vendor Model Electric Gripper"
  profile_version: "1.0.0"
  profile_status: "Validated"
  owner_team: "QNC Integration"
  created_utc: "2026-03-25T00:00:00Z"
  updated_utc: "2026-03-25T00:00:00Z"
  change_summary: "Initial validated release"

device_identity:
  vendor: "VendorName"
  model: "ModelName"
  device_class: "GRIPPER_ELECTRIC"
  variants:
    - "ModelName-24V"
  hardware_revisions:
    - "RevA"
  firmware_revisions:
    - ">=1.2.0,<2.0.0"
  device_ids:
    - "vendor:model:revA"

protocol_binding:
  protocol: "MODBUS_RTU"
  transport_profile: "modbus_rtu_default"
  addressing_mode: "slave_id"
  session_requirements:
    startup_timeout_ms: 2000
    command_timeout_class_default: "standard_motion"
  communication_defaults:
    baudrate: 115200
    parity: "N"
    stop_bits: 1
    retry_class: "default"
  discovery_hints:
    preferred_slave_ids: [9, 10]

capabilities:
  - capability_name: "enable_device"
    mandatory: true
    availability: "normal"
  - capability_name: "move_to_position"
    mandatory: true
    availability: "normal"
  - capability_name: "set_force"
    mandatory: true
    availability: "normal"
  - capability_name: "stop_motion"
    mandatory: true
    availability: "normal"

parameters:
  - parameter_name: "target_position"
    data_type: "float"
    unit: "mm"
    required: true
    minimum: 0.0
    maximum: 95.0
    precision: 0.1
    description: "Target opening position"
  - parameter_name: "target_force"
    data_type: "float"
    unit: "N"
    required: false
    minimum: 20.0
    maximum: 140.0
    precision: 1.0
    description: "Target gripping force"

commands:
  - command_name: "move_to_position"
    command_type: "profile"
    capability_ref: "move_to_position"
    description: "Move gripper to target opening position"
    execution_mode: "async"
    allowed_states: ["READY", "ACTIVE"]
    parameter_refs: ["target_position", "target_force"]
    device_mapping:
      write_sequence:
        - register: 0x0100
          source_parameter: "target_position"
        - register: 0x0101
          source_parameter: "target_force"
        - register: 0x0001
          literal_value: 1
      completion_check:
        status_field: "motion_complete"
        expected: true
    completion_criteria:
      type: "telemetry_flag"
      field_name: "motion_complete"
      expected_value: true
    timeout_class: "standard_motion"
    retry_class: "no_retry"

telemetry:
  - field_name: "position_actual"
    category: "measurements"
    data_type: "float"
    unit: "mm"
    source_mapping:
      register: 0x0200
    publication_policy:
      mode: "periodic"
      interval_ms: 100
    description: "Measured gripper position"
  - field_name: "object_secured"
    category: "device_class_state"
    data_type: "bool"
    source_mapping:
      register: 0x0201
      bit: 2
    publication_policy:
      mode: "event_and_periodic"
      interval_ms: 100
    description: "Gripper-class semantic state indicating secured object"

faults:
  - source_fault_id: "OVERCURRENT"
    qnc_fault_code: "DEV-002"
    severity: "Error"
    latched: true
    source_layer: "device"
    summary: "Device overcurrent fault"
    recovery_guidance: "Inspect load and clear recoverable fault if allowed"
    safe_mode_required: true

lifecycle_sequences:
  initialization_sequence:
    - step_id: "init_01"
      action_type: "write"
      mapping:
        register: 0x0000
        literal_value: 1
      timeout_class: "startup"
      on_failure: "abort"
      description: "Request device initialization"

compatibility:
  min_qnc_firmware: "2.0.0"
  max_validated_qnc_firmware: "2.x"
  required_interface_schema: "1.0"
  protocol_adapter_version: ">=2.0.0"
  compatibility_state: "compatible"

validation:
  validation_status: "Released"
  test_evidence_ref:
    - "VV-QNC-GRIPPER-001"
  review_approvals:
    - "SystemsApproved"
    - "QAApproved"
  last_validated_utc: "2026-03-25T00:00:00Z"
  validated_by: "QNC Validation Team"

security:
  signature_required: true
  artifact_hash: "SHA256:..."
  signing_authority_id: "QNC-REL"
  integrity_policy: "reject_on_failure"
```

---

# Appendix B. Example Gripper Profile

This appendix provides an illustrative example only. It does not by itself constitute a released profile.

## B.1 Example Intent

A gripper profile SHOULD normalize the following concepts where supported:
- enable state,
- position target,
- force target,
- motion complete,
- recoverable fault state,
- object secured.

## B.2 Example Cautions

A gripper profile MUST NOT assume:
- all grippers expose force identically,
- all grippers detect object presence reliably,
- all vendor “complete” bits mean the same thing.

These assumptions must be validated per device.

---

# Appendix C. Validation Checklist

A profile validation review SHALL confirm at minimum:

| Check | Pass/Fail |
|---|---|
| Metadata complete |
| Device identity bounded and explicit |
| Protocol binding supported |
| Commands internally consistent |
| Parameters bounded |
| Telemetry mappings resolvable |
| Fault mappings complete enough for release |
| Lifecycle sequences deterministic |
| Compatibility metadata complete |
| Validation evidence attached |
| Signature/integrity fields valid |

---

# Appendix D. Field Dictionary

## D.1 `device_class`
QNC-supported class to which the profile belongs.

## D.2 `capability_name`
Normalized functional behavior exposed by the profile.

## D.3 `device_mapping`
Declarative mapping between a normalized QNC command/field and protocol-native operations.

## D.4 `compatibility_state`
Declared compatibility relationship between the profile and the installed QNC runtime.

## D.5 `validation_status`
Governance state of the profile artifact.

---

## Closing Note

This Device Profile Specification is the missing formal layer that makes the [QNC](https://www.genspark.ai/api/files/s/MQZoL9fC) concept operationally credible. The original source relied heavily on descriptor-driven integration as part of its value proposition, but without a sufficiently strict specification, that approach would create ambiguity, support risk, and configuration fragility. This document makes the profile model explicit, governed, and testable. [Source](https://www.genspark.ai/api/files/s/MQZoL9fC)
