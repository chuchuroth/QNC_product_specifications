# QNC Interface Control Document  
**Document Number:** QNC-ICD-001  
**Version:** 2.3  
**Status:** Companion to QNC Product Requirements & System Definition Specification (PRS) v2.3  
**Classification:** Internal / Project Controlled  

---

## Revision History

| Version | Date       | Author(s)            | Changes |
|---------|------------|----------------------|---------|
| 2.0     | 2026-03-25 | Systems Engineering  | Companion ICD aligned with revised QNC product definition |
| 2.3     | 2026-04    | Systems Engineering  | Aligned structure, terminology, and scope tiers with QNC-PRD-001 v2.3; baseline protocols, northbound interfaces, four-layer architecture, configuration governance, DDS extension boundaries, and safety/security cross-references |

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
5. Release Scope and Supported Interfaces  
6. Interface Architecture  
7. Physical and Electrical Interfaces  
8. Logical Communication Interfaces  
9. Command Model  
10. Telemetry and State Model  
11. Fault and Error Model  
12. Discovery and Identification Behavior  
13. Configuration and Profile Interfaces  
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

This Interface Control Document (ICD) defines the **external interfaces, message contracts, state behaviors, fault handling rules, and configuration artifact interfaces** for the **Quick Network Connector (QNC)**. It operationalizes interface-level detail that the **QNC Product Requirements & System Definition Specification (PRS), QNC-PRD-001**, reserves for companion specifications.

This document is intended to be used by:
- systems engineers,
- embedded/firmware developers,
- robotics software developers,
- integration engineers,
- validation teams,
- project and release managers.

## 1.2 Scope

This ICD covers the interfaces between:
1. the **robot platform** (robot controller, robot software stack, or local automation application) and QNC,
2. QNC and **southbound** industrial end-effectors and adjacent field devices,
3. QNC and **northbound** clients and services (baseline and, where released, extension interfaces),
4. QNC and **configuration and profile artifacts** (including DDS governance artifacts when the FastDDS extension is in scope),
5. QNC and **service, update, and diagnostic tooling**.

Normative baseline interface contracts are defined herein. **Approved extension** and **advanced gateway direction** capabilities described in the PRS SHALL appear in this ICD as binding interface commitments only after the capability has completed the release validation and sign-off process defined in the PRS. Until then, such capabilities are roadmap-governed and SHALL NOT be treated as baseline ICD requirements.

## 1.3 Exclusions

This ICD does **not** define:
- product strategy, scope boundaries, or non-goals (governed by the PRS),
- detailed internal source-code module decomposition beyond interface-relevant layering,
- certified safety design or safety case evidence,
- protocols or northbound services not formally released for the applicable QNC build,
- manufacturing drawings or connector CAD (may reside in hardware specifications),
- arbitrary third-party device semantics outside approved device profiles.

This ICD SHALL be maintained so that interface claims remain consistent with the PRS principle of **unified integration, not native universality**: no protocol or service SHALL be implied as universally supported without validated release scope.

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

- **QNC Product Requirements & System Definition Specification (PRS), QNC-PRD-001, v2.3**
- **QNC Device Profile Specification** (to be issued)
- **QNC Protocol & Service Module Specification** (to be issued), including FastDDS Participant module design, QoS profile catalog, and deployment mode configuration where applicable
- **QNC Verification & Validation Plan** (to be issued)
- **QNC Operations & Recovery Guide** (to be issued)
- **QNC FastDDS IDL Type Registry** (to be issued), governing approved DDS topic IDL types and compatibility matrices where the FastDDS extension is released

Where conflicts arise, **QNC-PRD-001** governs product purpose, scope tiers, architecture boundaries, functional and non-functional requirements, safety position, security requirements, configuration governance, and validation expectations. This ICD SHALL be updated to remain consistent with the PRS.

---

# 4. System Context and Interface Boundary

## 4.1 System Context

QNC operates as a **configurable industrial end-effector connectivity gateway and software-first edge integration platform** (see PRS Section 2). It forms a controlled boundary among:

- **Southbound**: supported industrial end-effectors and adjacent field devices,
- **Robot-facing integration**: robot controller, robot software stack (including DDS- or ROS 2-based systems where applicable), or local automation applications,
- **Northbound** (baseline and extension): higher-level monitoring, orchestration, plant, or cloud-facing consumers via approved interfaces.

The logical interface chain is:

**Robot / automation client ↔ Northbound & integration interfaces ↔ QNC ↔ Southbound device link ↔ End-effector or field device**

## 4.2 Boundary of Control

### QNC SHALL own:
- southbound protocol session establishment and state machines for released protocols,
- supported transport framing and parsing,
- device profile and (where released) DDS IDL / QoS artifact loading and validation,
- command validation and normalization per approved profiles,
- normalized telemetry, status, health, and fault publication to approved northbound paths,
- connection and protocol state management,
- configuration integrity enforcement and atomic configuration application.

### Robot Platform SHALL own:
- application intent,
- process sequencing,
- robot motion and control decisions,
- system-level orchestration,
- safety controller and safety-rated system behavior,
- selection and deployment of compatible supported profiles and released extension configurations.

### End-Effector or Field Device SHALL own:
- device-internal actuation logic,
- vendor-specific firmware behavior,
- device-specific electrical behavior not abstracted by the approved profile.

## 4.3 Layered Architecture (PRS-Aligned)

QNC interface behavior SHALL map to the four architectural layers in PRS Section 7.1:

1. **Physical & Protocol Adaptation Layer**  
   Physical interface management, southbound protocol transport, and the FastDDS peer interface where that extension is released.

2. **Profile & Semantic Normalization Layer**  
   Mapping of device-specific and (where enabled) DDS-subscribed data into QNC internal abstractions using validated profiles and IDL type mappings.

3. **Integration & Control Layer**  
   Normalized commands, telemetry, state, and lifecycle exposure to robot-facing systems, including DDS-to-internal routing where released.

4. **Northbound Edge Services Layer**  
   Approved data publication and event routing (e.g., REST, WebSocket, logging; OPC UA, MQTT, and DDS-to-northbound bridging where released).

Each layer SHALL have defined external interface touchpoints, dependencies, and fault isolation expectations consistent with the PRS. The **Integration Interface Layer**, **Profile Interface Layer**, and **Protocol Interface Layer** descriptions elsewhere in this ICD SHALL be interpreted as views onto the above stack, not as a conflicting model.

---

# 5. Release Scope and Supported Interfaces

## 5.1 Relationship to the PRS Scope Model

QNC releases SHALL follow the tiered scope model in PRS Section 5:

- **Baseline Release Scope** — validated and documented in the current approved release.
- **Approved Extension Scope** — under design, implementation, or validation for a near-term release.
- **Advanced Gateway Direction** — roadmap direction without release commitment.

This ICD SHALL state baseline interface requirements normatively. Extension and advanced capabilities SHALL be documented here as normative ICD content only when explicitly released; otherwise they MAY be referenced summary-only and detailed in companion specifications (OpenAPI supplements, OPC UA NodeSets, MQTT topic conventions, IDL registry, protocol module specs) per PRS Sections 13.2 and 13.3.

## 5.2 Baseline Southbound Protocols

For the current baseline release, southbound protocol interfaces SHALL be limited to:

- **IO-Link** (v1.1 master)
- **Modbus RTU** (serial)
- **EtherNet/IP** (adapter mode, Class 1 and Class 3)
- **Discrete digital I/O** (sourcing/sinking configurable)

No other southbound protocol SHALL be claimed as baseline capability in this ICD.

## 5.3 Baseline Northbound Interfaces

Baseline northbound interfaces SHALL include:

- **REST API** (JSON over HTTP/HTTPS) for command, status query, and configuration
- **WebSocket** (JSON streaming for telemetry and events)
- **Logging and fault reporting** (syslog, structured JSON)

These SHALL conform to published schemas as defined in companion artifacts (OpenAPI 3.0 for REST, defined JSON message types for WebSocket, logging format rules) and SHALL support API versioning per PRS Section 13.2.

## 5.4 Approved Extension Interfaces (Summary)

The following are **not** baseline ICD commitments until released per PRS Section 5.3. When released, their normative contracts SHALL be added or referenced from this ICD:

- **Southbound extensions**: Modbus TCP, CANopen, additional serial/CAN-based protocols as approved
- **Northbound extensions**: OPC UA Server, MQTT Client, normalized multi-device aggregation and event routing
- **DDS / ROS 2 extensions**: FastDDS DomainParticipant integration, DDS topic subscription and publication per validated IDL and QoS artifacts, DDS-to-northbound bridging for validated paths (e.g., DDS-to-OPC UA, DDS-to-MQTT), DDS-aware relay patterns for approved deployment classes

FastDDS integration SHALL be governed per PRS Sections 5.5.1, 12.1, and 13.3 (including SIMPLE discovery as baseline DDS deployment mode and Discovery Server mode as an advanced tier requiring additional validation).

## 5.5 Advanced Gateway Direction (Summary)

The following remain **roadmap** items per PRS Section 5.4 until formally released. They SHALL NOT be represented as current ICD baseline requirements:

- Industrial Ethernet protocols requiring hardware assistance (e.g., EtherCAT, PROFINET), subject to approved coprocessor or FPGA architecture
- Plant-level supervisory integration (SCADA, MES, ERP connectivity)
- FastDDS Discovery Server mode for approved large-scale or cross-network deployments
- Fleet-level robot data aggregation and normalized cluster telemetry

## 5.6 Supported Device Classes

Baseline device classes include, non-exclusively:

- electric grippers,
- simple actuators,
- selected industrial sensors.

Support is valid only when an **approved, versioned, validated device profile** exists (PRS Section 7.2).

## 5.7 Unsupported Conditions

The following SHALL be treated as unsupported:

- devices without approved profiles,
- devices requiring protocols or northbound services not in the installed build’s released scope,
- devices with ambiguous or conflicting identification,
- devices whose declared profile version is incompatible with installed QNC firmware,
- DDS topics or QoS combinations not approved in the IDL Type Registry and QoS Profile Catalog when the FastDDS extension is enabled.

---

# 6. Interface Architecture

## 6.1 Interface Classes

The QNC external interface model consists of the following classes:

| Interface Class | Direction | Purpose |
|---|---|---|
| Integration Command Interface | Client → QNC | Command issuance (typically via northbound REST or equivalent) |
| Integration Telemetry Interface | QNC → Client | State and telemetry publication (REST, WebSocket, or equivalent) |
| Integration Fault Interface | QNC → Client | Fault and warning publication |
| Southbound Protocol Link Interface | QNC ↔ Device | Southbound transport/session communication |
| Northbound Client Interface | QNC → Northbound consumers | Baseline REST, WebSocket, logging; extension OPC UA, MQTT where released |
| DDS Peer Interface (extension) | QNC ↔ Robot DDS domain | FastDDS DomainParticipant subscription/publication where released |
| Configuration Interface | Tooling / authorized client → QNC | Load and manage configuration, profile, and DDS governance artifacts |
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

QNC SHALL reject operation or enter an operational inhibit consistent with **Safe Mode (Operational Inhibit Mode)** policy when electrical preconditions required by the active interface are violated. Such behavior is **not** a certified safety function (PRS Section 10).

Examples include:
- supply out of declared operating range,
- protocol signal integrity failure,
- short/open detection where hardware supports detection.

## 7.5 Physical Layer Notes

This ICD does not assume that one wiring rule applies universally across all supported southbound protocols. All protocol-side wiring guidance SHALL be protocol-specific and SHALL avoid ambiguous generic physical-layer statements that do not match the actual medium (e.g., Ethernet, IO-Link, or discrete I/O) in use.

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
Used to publish mode transitions such as initialization, ready, degraded, Safe Mode (Operational Inhibit Mode), and service mode.

### 8.4 Baseline Northbound Realization

Baseline northbound interfaces SHALL conform to PRS Sections 5.2 and 13.2:

- **REST API**: OpenAPI 3.0 specification defining endpoints, request/response schemas, and error codes; TLS 1.2 or higher for HTTPS where required by security policy (PRS SEC-001).
- **WebSocket**: JSON-based telemetry and event schema with defined message types.
- **Logging**: syslog and structured JSON fault and event reporting (PRS FR-032).

The logical channels listed in Section 8.2 SHALL be realized through these northbound services with **semantically equivalent** behavior. Endpoint naming MAY vary by binding; versioning SHALL follow PRS northbound API versioning rules.

### 8.5 DDS and FastDDS Peer Interface (Approved Extension, Where Released)

When the FastDDS extension is included in a validated release, QNC MAY create and manage a **FastDDS DomainParticipant** per PRS Sections 5.5.1 and 8.2. In that case:

- Subscribed and published topics SHALL be limited to **approved** topics with validated **IDL** schemas and **QoS** policies from the governed catalogs (PRS Sections 12.1, 13.3).
- **DDS-to-northbound bridging** SHALL be implemented only for explicitly validated and released paths (e.g., DDS-to-OPC UA, DDS-to-MQTT).
- **DDS domain faults** (disconnection, QoS mismatch, schema incompatibility) SHALL be **isolated** from unrelated southbound protocol operation and logged as structured faults (PRS FR-055, NFR-011).
- In **Safe Mode (Operational Inhibit Mode)**, DDS participation MAY continue in **subscribe-only** mode where permitted by configuration (PRS Section 6.2).

Until the FastDDS extension is released for a given build, the DDS Peer Interface SHALL NOT be claimed for that build.

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
- require authorization policy if security policy is enabled (PRS Section 11),
- not be presented as the primary integration abstraction.

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

For example, a gripper-oriented state like “object secured” MAY exist for the gripper class, but SHALL NOT be treated as a universal state across all device classes.

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
- integrity/security fault,
- **DDS domain fault** (where the FastDDS extension is released): domain disconnection, participant failure, **QoS negotiation mismatch**, **IDL schema incompatibility**, or bridging failure on an approved path.

DDS-related faults SHALL NOT degrade unrelated southbound protocol services except where a shared critical resource failure is explicitly documented.

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

QNC SHALL report and locally react to communication and integration faults within its scope.  
QNC SHALL NOT claim authority over system-level emergency stop or other **safety-rated** functions (PRS Section 10).

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

For baseline releases, discovery and enumeration SHALL satisfy PRS FR-001 (and related requirements) for each **released** southbound protocol; evidence SHALL be produced per the QNC Verification & Validation Plan under defined test conditions.

---

# 13. Configuration and Profile Interfaces

## 13.1 Artifact Types

QNC configuration SHALL be governed through artifact types consistent with PRS Section 12.1. The following controlled artifact types apply:

- **System configuration** — network settings, northbound interface configuration, logging levels, security policies
- **Device profiles** — validated per-device-class command, parameter, status, and fault mappings
- **DDS IDL schemas** — versioned IDL type definitions for subscribed and published DDS topics (when FastDDS extension is in scope)
- **DDS QoS profiles** — QoS policy definitions for DDS subscriptions and publications (when in scope)
- **Protocol configurations** — per-protocol settings (baud rates, timeouts, retry policies, etc.)
- **Update package** — signed/verified firmware or software update artifacts
- **Diagnostic export bundle** — controlled export for field diagnostics

Validation rules SHALL align with PRS Section 12.2 (e.g., atomic application, schema validation, rollback support).

## 13.2 System Configuration Interface

System configuration SHALL contain at minimum:
- QNC identity/configuration metadata,
- enabled northbound and integration interfaces,
- allowed southbound protocol and transport settings,
- security policy settings (aligned with PRS Section 11 where applicable),
- logging and service policy,
- profile compatibility policy,
- when the FastDDS extension is released: governed DDS domain, discovery mode, and participant QoS settings per validated deployment profiles.

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

Where quantitative targets apply, ICD timing claims SHALL remain consistent with PRS non-functional requirements (NFR-001 through NFR-005) and SHALL be evidenced per the Verification & Validation Plan.

---

# 15. Safety-Related Operational Behavior

## 15.1 Safety Position

**QNC is not a safety-rated system.** QNC does not provide functional safety capabilities as defined by IEC 61508, ISO 13849, or similar standards. Safety functions SHALL be implemented using separate, certified safety controllers and safety-rated I/O per applicable standards (PRS Section 10).

## 15.2 Safe Mode (Operational Inhibit Mode)

`SAFE_MODE` in this ICD corresponds to **Safe Mode (Operational Inhibit Mode)** in the PRS. In Safe Mode, QNC SHALL:

- reject new **actuation** commands,
- MAY continue southbound communication for **status and health queries** only, per PRS Section 6.2,
- limit northbound publication to **status and fault reporting** where applicable,
- MAY continue **DDS participation in subscribe-only mode** where permitted by configuration, when the FastDDS extension is released,
- log Safe Mode entry event and reason,
- require **explicit recovery** (manual reset or validated recovery protocol) to return to Normal Mode.

Safe Mode is a **fault management** feature, not a safety function.

## 15.3 Safety-Relevant Interface Behavior

QNC SHALL:
- detect and report communication and integrity failures within its integration scope,
- inhibit new actuation commands in Safe Mode,
- publish critical faults with appropriate interface priority on approved northbound paths.

QNC SHALL NOT:
- claim certified emergency stop authority or safety interlock functions,
- be positioned as a replacement for the robot or system safety controller.

## 15.4 Safe Mode Entry Conditions

Safe Mode SHALL be entered under defined critical fault conditions (see PRS FR-062 and fault policy), including examples such as:

- unrecoverable communication fault during active operation,
- critical integrity or configuration fault,
- active profile invalidation during control,
- explicit service or operator Safe Mode request.

## 15.5 Recovery from Safe Mode

Recovery SHALL require:
1. underlying critical condition cleared,
2. profile and protocol state revalidated,
3. explicit recovery command or operator-approved workflow (PRS FR-064).

---

# 16. Security and Integrity Interfaces

## 16.1 Security Scope

Security interfaces SHALL apply to:
- firmware,
- profiles,
- configuration and DDS governance artifacts,
- northbound API and WebSocket access,
- service access,
- diagnostic export,
- DDS domain access policy where the FastDDS extension is released.

Detailed security requirements (TLS, authentication, RBAC, encryption at rest, audit logging, secure update, firewall rules, DDS domain policy, DDS-Security evaluation) are specified in **PRS Section 11**. This ICD SHALL not contradict SEC-001 through SEC-010 for released builds.

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
- protocol adapter version,
- DDS IDL artifact versions and QoS profile catalog versions (when the FastDDS extension is released),
- robot software compatibility matrix revision for DDS interfaces (when applicable).

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
- command/response conformance across baseline northbound interfaces (REST, WebSocket as applicable),
- telemetry and event streaming conformance,
- profile validation behavior,
- fault handling and latching behavior,
- discovery and enumeration behavior for each **baseline** southbound protocol,
- Safe Mode (Operational Inhibit Mode) entry and exit,
- update and rollback behavior,
- logging and diagnostic behavior,
- protocol conformance and fault injection for each released southbound stack (PRS Section 15).

When the FastDDS extension is in release scope, verification SHALL additionally include DomainParticipant integration tests, DDS QoS negotiation, schema validation, DDS fault isolation, and DDS-to-northbound bridging tests for **released** paths, per PRS Section 15.

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
| `source_layer` | enum | Yes | Integration / profile / protocol / service / security / dds (when extension released) |
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
| `OPS-001` | Operations | Critical | Safe Mode (Operational Inhibit Mode) entered |
| `DDS-001` | DDS (extension) | Error | Domain or participant fault |
| `DDS-002` | DDS (extension) | Warning | QoS negotiation mismatch |
| `DDS-003` | DDS (extension) | Error | IDL schema validation failure |
| `DDS-004` | DDS (extension) | Error | DDS-to-northbound bridging fault on approved path |

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

A QNC implementation claiming conformity to this ICD for a **baseline** release SHALL demonstrate:

- support only for **declared baseline** southbound protocols and northbound interfaces (PRS Section 5.2),
- valid command/response correlation on released northbound bindings,
- layered state separation consistent with PRS Section 7.1,
- structured fault publication to approved northbound logging and client interfaces,
- deterministic **Normal Mode** and **Safe Mode (Operational Inhibit Mode)** behavior per PRS Section 6,
- profile validation enforcement per PRS Section 7.2,
- audit logging of controlled changes consistent with PRS configuration governance,
- clear unsupported-device and out-of-scope protocol handling,
- actuation command inhibit behavior in Safe Mode,
- version compatibility enforcement for interface schemas and profiles,
- no claim of unreleased extension or advanced-gateway capabilities as baseline ICD conformance.

Implementations including the **FastDDS extension** SHALL additionally demonstrate DDS catalog governance, fault isolation (PRS FR-055), and released bridging paths only.

---

## Document Positioning

This Interface Control Document is the **normative interface companion** to **QNC-PRD-001 v2.3**. It separates **southbound protocol behavior**, **profile and semantic normalization**, **robot- and northbound-facing integration behavior**, and **explicit non-safety positioning** so that external contracts can be validated independently of product strategy. Changes to interface commitments SHALL follow PRS governance (QNC-PRD-001 Section 18) and SHALL preserve alignment with the PRS scope model and terminology.
