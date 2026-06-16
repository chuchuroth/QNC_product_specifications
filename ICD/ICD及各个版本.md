# QNC Interface Control Document  
**Document Number:** QNC-ICD-001  
**Version:** 2.0  
**Status:** Proposed Baseline Standard  
**Classification:** Internal / Project Controlled  

---

## Document Control

| Version | Date | Author | Change Summary |
|---|---:|---|---|
| 2.0 | 2026-03-25 | Genspark Draft Rewrite | Companion ICD rewritten for consistency with revised QNC product definition |

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
4. System Context and Interface Boundary  
5. Supported Baseline Scope  
6. Interface Architecture  
7. Physical and Electrical Interfaces  
8. Logical Communication Interfaces  
9. Command Model  
10. Telemetry and State Model  
11. Fault and Error Model  
12. Discovery and Identification Behavior  
13. Configuration and Device Profile Interfaces  
14. Timing, Performance, and Quality of Service  
15. Safety-Related Operational Behavior  
16. Security and Integrity Interfaces  
17. Diagnostics, Logging, and Serviceability  
18. Versioning and Compatibility  
19. Verification Requirements  
Appendix A. Message Schema Definitions  
Appendix B. Error Code Catalog  
Appendix C. State Definitions  
Appendix D. Profile Artifact Structure  
Appendix E. Conformance Checklist  

---

# 1. Purpose and Scope

## 1.1 Purpose

This Interface Control Document (ICD) defines the **external interfaces, message contracts, state behaviors, fault handling rules, and configuration artifact interfaces** for the **QNC (Quick Network Connector)** baseline release.

This document is intended to be used by:
- systems engineers,
- embedded/firmware developers,
- robotics software developers,
- integration engineers,
- validation teams,
- project and release managers.

## 1.2 Scope

This ICD covers the interfaces between:
1. the **robot platform** and QNC,
2. QNC and the **supported end-effector-side communication link**,
3. QNC and **configuration/profile artifacts**,
4. QNC and **service/update/diagnostic tooling**.

This ICD defines only the **baseline release interface scope** and shall not be interpreted as a commitment to unsupported protocols, unsupported devices, or future roadmap capabilities.

## 1.3 Exclusions

This ICD does **not** define:
- business case or market strategy,
- detailed internal source-code architecture,
- certified safety design,
- unsupported protocol families,
- manufacturing drawings or connector CAD,
- arbitrary third-party device semantics outside approved profiles.

The need for this cleaner interface boundary follows directly from inconsistencies in the original QNC source, which mixed product, architecture, protocol, and roadmap concerns into a single document [Source](https://www.genspark.ai/api/files/s/MQZoL9fC).

---

# 2. Normative Language

The key words **SHALL**, **SHOULD**, **MAY**, and **MUST NOT** in this document are to be interpreted as follows:

- **SHALL**: mandatory requirement.
- **SHOULD**: recommended unless a justified exception is approved.
- **MAY**: optional capability.
- **MUST NOT**: prohibited behavior.

---

# 3. Related Documents

This ICD is subordinate to and shall be interpreted with the following companion documents:

- **QNC Product Requirements & System Definition Specification (PRD/SDS), v2.0**
- **QNC Device Profile Specification (to be issued)**
- **QNC Verification & Validation Plan (to be issued)**
- **QNC Operations & Recovery Guide (to be issued)**

Where conflicts arise, the revised PRD/SDS governs product scope and boundary. This companion ICD is designed to operationalize that revised definition [Source](https://www.genspark.ai/api/files/s/MQZoL9fC).

---

# 4. System Context and Interface Boundary

## 4.1 System Context

QNC operates as a **connectivity gateway** between a robot platform and a supported industrial end-effector.

The interface chain is:

**Robot Platform ↔ QNC Integration Interface ↔ QNC Protocol Layer ↔ End-Effector Communication Link ↔ End-Effector**

## 4.2 Boundary of Control

### QNC SHALL own:
- protocol session establishment,
- supported transport framing and parsing,
- profile loading and validation,
- command validation,
- normalized telemetry and fault publication,
- connection state management,
- configuration integrity enforcement.

### Robot Platform SHALL own:
- application intent,
- process sequencing,
- robot motion decisions,
- system-level orchestration,
- safety controller behavior,
- selection and deployment of compatible supported profiles.

### End-Effector SHALL own:
- device-internal actuation logic,
- vendor-specific internal firmware behavior,
- device-specific electrical behavior not abstracted by the approved profile.

## 4.3 Architectural Separation

QNC SHALL expose three distinct interface layers:

1. **Integration Interface Layer**  
   Robot-facing commands, telemetry, health, and faults.

2. **Profile Interface Layer**  
   Validated profile behavior, parameter mapping, capability model.

3. **Protocol Interface Layer**  
   Physical/protocol transport to the end-effector.

This layered separation is intended to eliminate the ambiguity in the original definition, where QNC was described both as protocol-only and as partly device-aware [Source](https://www.genspark.ai/api/files/s/MQZoL9fC).

---

# 5. Supported Baseline Scope

## 5.1 Supported Protocols

The baseline release SHALL support only the following end-effector-side protocols:

- **Modbus RTU**
- **IO-Link**

No other fieldbus or industrial Ethernet protocol shall be treated as baseline capability in this ICD.

## 5.2 Supported Device Classes

Baseline device classes:
- electric grippers,
- simple actuators,
- selected industrial sensors.

Support is valid only when an approved, versioned, validated device profile exists.

## 5.3 Unsupported Conditions

The following SHALL be treated as unsupported:
- devices without approved profiles,
- devices requiring unsupported protocols,
- devices with ambiguous or conflicting identification,
- devices whose declared profile version is incompatible with installed QNC firmware.

---

# 6. Interface Architecture

## 6.1 Interface Classes

The QNC external interface model consists of the following classes:

| Interface Class | Direction | Purpose |
|---|---|---|
| Integration Command Interface | Robot → QNC | Command issuance |
| Integration Telemetry Interface | QNC → Robot | State and telemetry publication |
| Integration Fault Interface | QNC → Robot | Fault and warning publication |
| Protocol Link Interface | QNC ↔ Device | Transport/session communication |
| Configuration Interface | Tooling/Robot → QNC | Load and manage configuration/profile artifacts |
| Service Interface | Tooling → QNC | Diagnostics, logs, update, recovery |

## 6.2 Interface Principles

Each external interface SHALL:
- have a unique interface identifier,
- define allowed message types,
- define state preconditions,
- define failure behavior,
- define version compatibility rules.

## 6.3 Data Ownership Rules

- QNC SHALL be the system of record for **active connection status**, **active protocol session**, and **active loaded profile**.
- The robot platform SHALL be the system of record for **application intent**.
- Vendor-specific raw values MAY be exposed, but SHALL be clearly labeled as vendor-specific and not universal semantics.

---

# 7. Physical and Electrical Interfaces

## 7.1 Physical Interface Policy

Physical interfaces SHALL be documented sufficiently for integration and validation, while detailed connector part numbers and manufacturing drawings MAY be maintained in a separate hardware specification.

## 7.2 Robot-Side Interface

The baseline robot-side physical interface SHALL support the approved system communication and power interface selected by hardware design freeze.

The ICD SHALL define the following at minimum:
- connector identifier,
- signal names,
- voltage domain,
- direction,
- nominal tolerances,
- grounding/shielding expectations.

## 7.3 End-Effector-Side Interface

The end-effector-side physical interface SHALL define:
- transport type,
- power delivery constraints,
- signal directions,
- isolation or reference requirements if applicable,
- supported wiring modes for each protocol family.

## 7.4 Electrical Safety Constraints

QNC SHALL reject operation or enter inhibit mode when electrical preconditions required by the active interface are violated.

Examples include:
- supply out of declared operating range,
- protocol signal integrity failure,
- short/open detection where hardware supports detection.

## 7.5 Physical Layer Notes

This ICD does not assume that one wiring rule applies universally across all supported protocols. All protocol-side wiring guidance SHALL be protocol-specific and SHALL avoid ambiguous statements such as generic TX/RX rules for non-UART physical layers, a source of confusion in the original material [Source](https://www.genspark.ai/api/files/s/MQZoL9fC).

---

# 8. Logical Communication Interfaces

## 8.1 Integration Interface Overview

The robot-facing logical interface SHALL consist of the following logical channels:

- **Command Channel**
- **Response Channel**
- **Telemetry Channel**
- **Fault/Event Channel**
- **Service/Management Channel**

## 8.2 Required Logical Topics / Endpoints

A baseline implementation SHALL provide logical interfaces equivalent to:

| Logical Interface | Purpose |
|---|---|
| `qnc.command` | Robot issues commands |
| `qnc.response` | QNC returns command results |
| `qnc.telemetry` | QNC publishes operational telemetry |
| `qnc.fault` | QNC publishes faults and warnings |
| `qnc.lifecycle` | QNC lifecycle and mode transitions |
| `qnc.service` | Management and service operations |

Endpoint naming MAY vary by middleware implementation, but the interface semantics SHALL remain equivalent.

## 8.3 Interface Semantics

### Command Channel
Used for robot-originated requests to perform supported actions.

### Response Channel
Used for deterministic acknowledgment and completion reporting.

### Telemetry Channel
Used for periodic or event-driven publication of state and measurements.

### Fault/Event Channel
Used for warnings, errors, recovery notices, and important operational events.

### Lifecycle Channel
Used to publish mode transitions such as initialization, ready, degraded, safe mode, service mode.

---

# 9. Command Model

## 9.1 Command Categories

QNC SHALL classify commands into three categories:

1. **Profile-Level Commands**  
   Device-class operations supported by the active profile.

2. **Transport-Level Commands**  
   Protocol-specific or raw access commands, restricted by policy.

3. **Management Commands**  
   Configuration, diagnostics, lifecycle, and service functions.

## 9.2 Command Execution Rules

A command SHALL include:
- command identifier,
- target context,
- command type,
- parameters,
- caller timestamp,
- correlation identifier.

QNC SHALL:
- validate command structure,
- validate command preconditions,
- validate parameter ranges,
- reject commands inconsistent with current mode or active profile.

## 9.3 Allowed Profile-Level Command Types

Baseline profile-level command types MAY include:
- initialize,
- enable,
- disable,
- move/open/close/setpoint commands for supported actuator classes,
- read status,
- clear recoverable fault,
- stop or inhibit.

The exact allowed set SHALL be defined by device class and active profile.

## 9.4 Raw/Advanced Command Access

Raw or register-level commands MAY be supported for engineering or service use, but SHALL:
- be explicitly flagged as advanced,
- require authorization policy if security policy is enabled,
- not be presented as the primary integration abstraction.

This resolves the ambiguity in the original definition, where “generic” commands appeared to expose low-level register details while also implying high-level abstraction [Source](https://www.genspark.ai/api/files/s/MQZoL9fC).

## 9.5 Command Completion Semantics

Each command SHALL result in one of the following response outcomes:
- accepted,
- rejected,
- completed successfully,
- completed with warning,
- failed,
- timed out,
- canceled,
- inhibited by mode.

---

# 10. Telemetry and State Model

## 10.1 Telemetry Classes

QNC SHALL expose telemetry in four classes:

1. **Connectivity Telemetry**
2. **Operational Telemetry**
3. **Device-Class Telemetry**
4. **Vendor-Specific Telemetry**

## 10.2 Required Core Telemetry Fields

Every telemetry publication SHALL include:
- timestamp,
- QNC software version,
- active profile identifier,
- profile version,
- current lifecycle mode,
- connection state,
- protocol state,
- health summary.

## 10.3 State Layer Separation

QNC SHALL keep the following states separate:

| State Layer | Purpose |
|---|---|
| Lifecycle State | Operational mode of QNC |
| Connection State | Physical/protocol connection status |
| Protocol State | Session/transport readiness |
| Profile State | Whether a valid active profile is loaded |
| Device-Class State | Approved normalized state for the active class |
| Vendor-Specific State | Raw or vendor-defined state |

## 10.4 Device-Class State Constraints

Device-class states SHALL be defined only for approved supported classes.  
Vendor-specific meanings SHALL NOT be mislabeled as universal semantics.

For example, a gripper-oriented state like “object secured” MAY exist for the gripper class, but SHALL NOT be treated as a universal state across all devices. This directly addresses conceptual leakage in the original source where gripper semantics appeared inside a supposedly protocol-only abstraction [Source](https://www.genspark.ai/api/files/s/MQZoL9fC).

## 10.5 Lifecycle States

Baseline lifecycle states SHALL include:
- `BOOTING`
- `INITIALIZING`
- `READY`
- `ACTIVE`
- `DEGRADED`
- `SAFE_MODE`
- `SERVICE_MODE`
- `FAULT_LOCKED`

Definitions are provided in Appendix C.

---

# 11. Fault and Error Model

## 11.1 Fault Model Principles

The QNC fault model SHALL:
- be structured,
- be layer-aware,
- be deterministic,
- support diagnosis and recovery,
- distinguish hard faults from warnings.

## 11.2 Fault Categories

Faults SHALL be categorized as:
- configuration fault,
- profile fault,
- connection fault,
- protocol fault,
- command validation fault,
- device-reported fault,
- service/update fault,
- integrity/security fault.

## 11.3 Fault Severity Levels

| Severity | Meaning |
|---|---|
| INFO | Informational event |
| WARNING | Degraded but operable |
| ERROR | Operation failed or blocked |
| CRITICAL | Immediate inhibit / safe mode required |

## 11.4 Fault Publication Requirements

Each fault event SHALL include:
- fault code,
- severity,
- source layer,
- timestamp,
- current mode,
- summary text,
- optional detailed context,
- recommended recovery action.

## 11.5 Fault Latching Rules

Critical faults SHALL latch until:
- the underlying condition clears, and
- an approved recovery action is performed.

Warnings MAY auto-clear if policy permits.

## 11.6 Fault Ownership

QNC SHALL report and locally react to communication/integration faults within its scope.  
QNC SHALL NOT claim authority over system-level emergency stop behavior in baseline release.

This explicitly resolves the unclear safety role in the original material, where QNC both disclaimed safety scope and appeared to participate in E-stop decisions [Source](https://www.genspark.ai/api/files/s/MQZoL9fC).

---

# 12. Discovery and Identification Behavior

## 12.1 Discovery Objective

Discovery SHALL determine one of the following:
- supported connected device identified,
- connected but unsupported device,
- ambiguous identity,
- communication path available but device unresolved,
- no device detected.

## 12.2 Identification Inputs

QNC MAY use one or more of the following identification inputs:
- physical presence signal,
- protocol handshake response,
- profile compatibility metadata,
- optional hardware identification mechanism if approved by hardware design.

## 12.3 Precedence Rules

If multiple identification mechanisms are used, the precedence SHALL be:

1. explicit operator/service configuration,
2. validated protocol identification result,
3. approved hardware identification hint,
4. profile compatibility resolution.

No identification mechanism SHALL silently override a higher-confidence source without logging.

## 12.4 Ambiguity Handling

If device identity remains ambiguous:
- QNC SHALL enter `DEGRADED` or `SAFE_MODE` according to policy,
- QNC SHALL NOT advertise the device as supported,
- QNC SHALL emit a clear ambiguity fault.

## 12.5 Discovery Timing Policy

Discovery time SHALL be specified:
- per protocol,
- per discovery mode,
- under defined test conditions.

This ICD deliberately does not declare a single universal discovery time, because the original document mixed aggressive timing claims with scanning behavior that implied a much larger search space [Source](https://www.genspark.ai/api/files/s/MQZoL9fC).

---

# 13. Configuration and Device Profile Interfaces

## 13.1 Artifact Types

QNC SHALL support the following controlled artifact types:
- system configuration,
- device profile,
- update package,
- diagnostic export bundle.

## 13.2 System Configuration Interface

System configuration SHALL contain at minimum:
- QNC identity/configuration metadata,
- enabled interfaces,
- allowed protocol settings,
- security policy settings,
- logging and service policy,
- profile compatibility policy.

## 13.3 Device Profile Interface

Each device profile SHALL define:
- profile ID,
- profile version,
- supported device class,
- supported device identifiers,
- protocol binding,
- initialization sequence,
- allowed command set,
- parameter limits,
- telemetry mappings,
- fault mappings,
- recovery policy hints,
- compatibility constraints.

## 13.4 Profile Validation Rules

QNC SHALL reject a profile if:
- schema validation fails,
- signature/integrity checks fail,
- required fields are missing,
- referenced command/state mapping is invalid,
- compatibility constraints are not met.

## 13.5 Configuration State Machine

Configuration and profile application SHALL be atomic from the perspective of active operation. QNC SHALL NOT operate under partially applied configuration.

---

# 14. Timing, Performance, and Quality of Service

## 14.1 Performance Specification Principles

Performance SHALL be defined by scenario, not slogan.

Every published performance metric SHALL specify:
- command type,
- protocol,
- active profile,
- payload size class,
- transport conditions,
- system mode,
- whether the metric is typical, maximum observed, or guaranteed.

## 14.2 Timing Classes

QNC SHALL define at minimum:
- command acceptance latency,
- command completion latency,
- telemetry publication interval,
- fault publication latency,
- mode transition latency,
- configuration application time,
- discovery duration per mode.

## 14.3 Quality of Service Requirements

The integration interface SHALL ensure:
- message ordering where required,
- correlation of response to command,
- deterministic timeout behavior,
- bounded retry behavior,
- no silent dropping of critical faults.

## 14.4 Timeout Policy

Timeouts SHALL be:
- command-specific,
- profile-aware where needed,
- explicitly configurable within approved bounds.

No timeout default SHALL contradict the published performance class for the same operation.

This provision is included to prevent the type of inconsistent timing definitions observed in the original source [Source](https://www.genspark.ai/api/files/s/MQZoL9fC).

---

# 15. Safety-Related Operational Behavior

## 15.1 Safety Position

Baseline QNC is **non-safety-rated**.

## 15.2 Safety-Relevant Interface Behavior

QNC SHALL:
- detect and report safety-relevant communication failures within its scope,
- inhibit new actuation commands when entering `SAFE_MODE`,
- publish critical faults with highest available interface priority.

QNC SHALL NOT:
- claim certified emergency stop authority,
- be positioned as a replacement for the robot/system safety controller.

## 15.3 Safe Mode Entry Conditions

Safe mode SHALL be entered under conditions such as:
- unrecoverable communication fault during active operation,
- critical integrity/configuration fault,
- active profile invalidation during control,
- explicit service/operator safe mode request.

## 15.4 Recovery from Safe Mode

Recovery SHALL require:
1. underlying critical condition cleared,
2. profile and protocol state revalidated,
3. explicit recovery command or operator-approved workflow.

---

# 16. Security and Integrity Interfaces

## 16.1 Security Scope

Security interfaces SHALL apply to:
- firmware,
- profiles,
- configuration artifacts,
- service access,
- diagnostic export.

## 16.2 Integrity Verification

QNC SHALL verify integrity before activation of:
- firmware updates,
- profile artifacts,
- configuration artifacts.

## 16.3 Service Access

Service operations SHALL be classified by privilege level.  
At minimum:
- read-only diagnostics,
- maintenance operations,
- update/configuration operations.

## 16.4 Auditability

The following SHALL be audit logged:
- profile load/unload,
- configuration changes,
- update attempts,
- recovery actions,
- entry into and exit from safe mode,
- authentication/authorization failures if implemented.

---

# 17. Diagnostics, Logging, and Serviceability

## 17.1 Diagnostics Interface

QNC SHALL provide a diagnostics interface supporting:
- current state inspection,
- active faults,
- recent event log,
- protocol statistics,
- active profile metadata,
- compatibility status.

## 17.2 Logging Requirements

Logs SHALL distinguish:
- system events,
- connection/protocol events,
- command execution events,
- fault events,
- security/integrity events,
- service actions.

## 17.3 Field Service Behavior

QNC SHALL support a service mode in which:
- actuation commands are inhibited by default,
- diagnostics and controlled maintenance actions are permitted,
- profile/configuration update workflows may be executed safely.

---

# 18. Versioning and Compatibility

## 18.1 Version Domains

The following version domains SHALL be independently tracked:
- QNC firmware version,
- interface schema version,
- profile schema version,
- active profile version,
- protocol adapter version.

## 18.2 Compatibility Rules

A profile SHALL declare:
- minimum supported firmware version,
- maximum validated firmware version if bounded,
- required schema version,
- supported device identifiers.

## 18.3 Backward Compatibility

Interface changes SHALL be classified as:
- backward compatible,
- conditionally compatible,
- breaking.

Breaking changes SHALL require:
- version increment,
- migration notes,
- validation update.

---

# 19. Verification Requirements

## 19.1 Interface Verification

Every interface defined in this ICD SHALL have:
- requirement identifier,
- verification method,
- owner,
- acceptance criteria.

## 19.2 Required Verification Methods

Permitted verification methods:
- inspection,
- analysis,
- test,
- demonstration.

## 19.3 Mandatory Test Areas

The baseline verification plan SHALL include:
- command/response conformance,
- telemetry conformance,
- profile validation behavior,
- fault handling and latching behavior,
- discovery behavior,
- safe mode entry/exit,
- update and rollback behavior,
- logging and diagnostic behavior.

---

# Appendix A. Message Schema Definitions

## A.1 Common Header

All robot-facing logical messages SHALL include:

| Field | Type | Description |
|---|---|---|
| `schema_version` | string | Message schema version |
| `message_id` | string | Unique message ID |
| `correlation_id` | string | Links response/event to request |
| `timestamp_utc` | string | ISO 8601 UTC timestamp |
| `qnc_id` | string | Unique QNC identifier |

## A.2 Command Message

| Field | Type | Required | Description |
|---|---|---:|---|
| `header` | object | Yes | Common header |
| `command_type` | enum | Yes | Profile / transport / management |
| `target_class` | enum | Yes | Device class target |
| `target_id` | string | No | Optional target instance |
| `profile_id` | string | No | Expected active profile |
| `parameters` | object | Yes | Command parameters |
| `timeout_ms` | integer | No | Requested timeout within allowed range |

## A.3 Response Message

| Field | Type | Required | Description |
|---|---|---:|---|
| `header` | object | Yes | Common header |
| `request_message_id` | string | Yes | Original command ID |
| `status` | enum | Yes | Accepted / rejected / completed / failed / timed out |
| `result_code` | string | Yes | Result code identifier |
| `details` | object | No | Structured response details |

## A.4 Telemetry Message

| Field | Type | Required | Description |
|---|---|---:|---|
| `header` | object | Yes | Common header |
| `lifecycle_state` | enum | Yes | QNC lifecycle state |
| `connection_state` | enum | Yes | Physical/protocol connectivity |
| `protocol_state` | enum | Yes | Session state |
| `profile_state` | enum | Yes | Profile validity state |
| `device_class_state` | object | No | Normalized class-level state |
| `vendor_state` | object | No | Vendor-specific raw state |
| `health` | object | Yes | Health summary |

## A.5 Fault Message

| Field | Type | Required | Description |
|---|---|---:|---|
| `header` | object | Yes | Common header |
| `fault_code` | string | Yes | Fault identifier |
| `severity` | enum | Yes | Info / warning / error / critical |
| `source_layer` | enum | Yes | Integration / profile / protocol / service / security |
| `latched` | boolean | Yes | Whether manual recovery required |
| `summary` | string | Yes | Human-readable summary |
| `context` | object | No | Structured fault context |
| `recommended_action` | string | No | Recovery guidance |

---

# Appendix B. Error Code Catalog

The final code set SHALL be project-controlled. A baseline structure is defined below.

| Code | Category | Severity | Meaning |
|---|---|---|---|
| `CFG-001` | Configuration | Error | Invalid configuration schema |
| `CFG-002` | Configuration | Critical | Configuration activation failed |
| `PRF-001` | Profile | Error | Profile schema invalid |
| `PRF-002` | Profile | Critical | Active profile invalidated |
| `LNK-001` | Connection | Warning | Link unstable |
| `LNK-002` | Connection | Error | Link lost |
| `PTL-001` | Protocol | Error | Session initialization failed |
| `PTL-002` | Protocol | Error | Timeout during protocol operation |
| `CMD-001` | Command | Error | Invalid command parameters |
| `CMD-002` | Command | Error | Command rejected in current mode |
| `DEV-001` | Device | Warning | Device-reported warning |
| `DEV-002` | Device | Error | Device-reported fault |
| `SEC-001` | Security | Critical | Integrity verification failed |
| `SRV-001` | Service | Error | Update operation failed |
| `OPS-001` | Operations | Critical | Safe mode entered |

---

# Appendix C. State Definitions

## C.1 Lifecycle States

| State | Meaning |
|---|---|
| `BOOTING` | QNC is starting |
| `INITIALIZING` | Interfaces and profiles being validated |
| `READY` | Ready for activation |
| `ACTIVE` | Active operation allowed |
| `DEGRADED` | Partial operation with restrictions |
| `SAFE_MODE` | New actuation commands inhibited |
| `SERVICE_MODE` | Maintenance mode |
| `FAULT_LOCKED` | Critical latched condition requiring recovery |

## C.2 Connection States

| State | Meaning |
|---|---|
| `DISCONNECTED` | No valid connection detected |
| `DETECTING` | Detection/discovery in progress |
| `CONNECTED_UNRESOLVED` | Link present, device not yet supported/identified |
| `CONNECTED_SUPPORTED` | Link present and supported device/profile active |
| `CONNECTED_UNSUPPORTED` | Link present, device unsupported |
| `FAULTED` | Connection or protocol fault present |

## C.3 Profile States

| State | Meaning |
|---|---|
| `NOT_LOADED` | No profile loaded |
| `VALIDATING` | Profile validation in progress |
| `ACTIVE_VALID` | Valid profile active |
| `ACTIVE_INCOMPATIBLE` | Profile loaded but incompatible |
| `INVALID` | Profile failed validation |

---

# Appendix D. Profile Artifact Structure

A device profile artifact SHALL contain, at minimum:

```yaml
profile_id: string
profile_version: string
device_class: enum
supported_device_ids:
  - string
protocol_binding:
  protocol: enum
  transport_parameters: object
initialization_sequence:
  - step_id: string
    action: string
    parameters: object
commands:
  - name: string
    parameters: object
    limits: object
telemetry_mapping:
  normalized_fields: object
  vendor_fields: object
fault_mapping:
  vendor_faults: object
compatibility:
  min_firmware: string
  schema_version: string
security:
  signature_required: true
```

The exact schema SHALL be maintained in the Device Profile Specification.

---

# Appendix E. Conformance Checklist

A QNC implementation claiming conformity to this ICD SHALL demonstrate:

- support only for declared baseline protocols,
- valid command/response correlation,
- layered state separation,
- structured fault publication,
- deterministic mode behavior,
- profile validation enforcement,
- audit logging of controlled changes,
- clear unsupported-device handling,
- safe-mode inhibit behavior,
- version compatibility enforcement.

---

## Closing Note

This companion ICD is intended to be a **clean operational interface standard** for the revised QNC concept. It deliberately removes the ambiguity of the original document by clearly separating protocol behavior, profile behavior, integration behavior, and safety boundary. That original source provided the underlying product intent, but also demonstrated why a tighter ICD was needed [Source](https://www.genspark.ai/api/files/s/MQZoL9fC).
