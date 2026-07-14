# QNC Canonical Semantic Core (v1)

**Document Number:** QNC-CSC-001  
**Version:** 1.0  
**Status:** Normative — single source of truth for cross-interface semantics  
**Classification:** Internal / Project Controlled  

---

## 1. Purpose

This document defines the **canonical semantic model** for QNC v1. All external contracts SHALL derive from this core:

- REST/OpenAPI (robot-facing and northbound API families)
- WebSocket event and stream schemas
- Structured logging and audit events
- **QNC DDS Pack** IDL types and topic payloads (when the DDS Pack is installed)
- Device profile normalization targets (southbound mapping **into** these semantics)

**Rule:** Other specifications SHALL NOT redefine competing command, response, telemetry, fault, or lifecycle semantics. They SHALL only bind transport, encoding, layering, and governance.

---

## 2. Semantic Primitives

### 2.1 Identifiers and Correlation

| Field | Meaning |
|--------|---------|
| `schema_version` | Version of the message schema binding (API or DDS Pack) |
| `message_id` | Unique id for this message instance |
| `correlation_id` | Links response, async completion, and related telemetry to a command |
| `timestamp_utc` | ISO 8601 UTC |
| `qnc_instance_id` | Deployed QNC instance identifier |

### 2.2 Command (canonical)

**Intent:** Request a supported operation within active profile, mode, and policy.

| Field | Required | Description |
|--------|----------|-------------|
| `header` | Yes | Common header (§2.1) |
| `command_category` | Yes | `profile` \| `transport` \| `management` |
| `command_id` | Yes | Normalized command identifier |
| `target` | Yes | Target device/slot/port binding reference |
| `parameters` | Yes | Bounded key-value per command schema |
| `timeout_ms` | No | Upper bound within profile/policy limits |
| `caller_context` | No | Opaque, bounded metadata from integrator |

**Preconditions:** Evaluated by runtime; violations yield `rejected` without side effects.

### 2.3 Response (canonical)

**Intent:** Deterministic outcome for a command (sync or async completion).

| Field | Required | Description |
|--------|----------|-------------|
| `header` | Yes | Common header |
| `request_message_id` | Yes | Original command `message_id` |
| `completion` | Yes | Enumeration below |
| `result_code` | Yes | Stable machine-readable code |
| `details` | No | Structured payload |

**Completion enumeration:**  
`accepted` \| `rejected` \| `completed` \| `completed_with_warning` \| `failed` \| `timed_out` \| `canceled` \| `inhibited_by_mode`

### 2.4 Telemetry (canonical)

**Intent:** Time-series and event-sampled operational data.

| Field | Required | Description |
|--------|----------|-------------|
| `header` | Yes | Common header |
| `lifecycle_state` | Yes | Canonical lifecycle (§2.6) |
| `connection_state` | Yes | Southbound link state |
| `protocol_state` | Yes | Session/discovery/transaction state **for that protocol instance** |
| `profile_state` | Yes | Profile load/validity |
| `device_class_state` | No | Class-scoped normalized state |
| `vendor_state` | No | Raw vendor fields; clearly labeled |
| `health` | Yes | Summary object (e.g., `ok`, `degraded`, `faulted`) |
| `telemetry_quality` | No | `valid` \| `stale` \| `estimated` \| `unsupported` \| `faulted` |

**Note:** DDS Pack may mirror a subset as DDS samples; fields SHALL remain semantically aligned with this structure.

### 2.5 Fault (canonical)

**Intent:** Structured fault reporting for integration and operations.

| Field | Required | Description |
|--------|----------|-------------|
| `header` | Yes | Common header |
| `fault_code` | Yes | Stable fault code |
| `severity` | Yes | `info` \| `warning` \| `error` \| `critical` |
| `fault_domain` | Yes | `configuration` \| `profile` \| `southbound_connection` \| `southbound_protocol` \| `southbound_device` \| `command` \| `service` \| `security` \| `northbound` \| `robot_api` \| **`dds_pack`** |
| `latched` | Yes | Whether explicit recovery required |
| `summary` | Yes | Human-readable |
| `context` | No | Structured diagnostic context |
| `recommended_action` | No | Operator guidance |

**Isolation rule:** Faults in `dds_pack` SHALL NOT imply southbound `southbound_*` fault semantics and vice versa, except where a single documented shared resource failure is explicitly specified in the PSM.

### 2.6 Lifecycle (canonical)

**Intent:** Single lifecycle state machine for QNC runtime (not robot application lifecycle).

**States:**  
`BOOTING` → `INITIALIZING` → `READY` → `ACTIVE` → (`DEGRADED` ↔ `ACTIVE`) → `SAFE_MODE` | `SERVICE_MODE` → (`FAULT_LOCKED` from critical latched conditions)

| State | Meaning |
|--------|---------|
| `BOOTING` | Power-on / process start |
| `INITIALIZING` | Artifact validation, transport bring-up |
| `READY` | Valid config; actuation not yet enabled per policy |
| `ACTIVE` | Normal operation |
| `DEGRADED` | Partial restrictions; non-critical faults |
| `SAFE_MODE` | Operational inhibit: **no new actuation**; diagnostics/telemetry per policy |
| `SERVICE_MODE` | Maintenance; actuation inhibited by default |
| `FAULT_LOCKED` | Critical latched; explicit recovery |

**Safe Mode** is **not** a functional safety mode (see PRS safety position).

---

## 3. API Binding Namespaces (versioning)

| API family | Purpose | Version surface |
|------------|---------|-----------------|
| **Robot-facing API** | Commands, high-rate integration from robot controller / cell software | Independent MAJOR.MINOR; compatibility policy: PRS |
| **Northbound API** | Plant/cloud/IT consumers, observability | Independent MAJOR.MINOR; stricter deprecation for external consumers |

Same canonical semantics; different route prefixes, auth policies, and lifecycle cadence.

---

## 4. Encoding Policy

- **Authoritative configuration and profile artifacts:** YAML at rest and in CI.  
- **Wire formats:** JSON for REST/WebSocket/logs as specified in ICD/OpenAPI.  
- **Internal runtime:** Implementations MAY compile YAML to optimized internal representation (e.g., JSON binary, compiled structs).  
- **DDS Pack:** IDL types SHALL be generated/mapped from the canonical fields above, not divergent parallel models.

---

## 5. Traceability

| Companion | How it uses this core |
|-----------|------------------------|
| PRS | Product scope, NFRs, packaging, capacity |
| ICD | REST/WS/log field binding to this core |
| DPS | Profile maps device native → Command/Telemetry/Fault subsets |
| PSM | Module responsibilities, fault domains, protocol matrices |
| QNC-IDL | DDS Pack IDL layers align to L2/L3 in IDL spec |
| ORG | Runbooks reference lifecycle and fault domains |

---

## Document Control

| Version | Date | Summary |
|---------|------|---------|
| 1.0 | 2026-05-12 | Initial consolidated canonical semantic core for QNC v1 |
