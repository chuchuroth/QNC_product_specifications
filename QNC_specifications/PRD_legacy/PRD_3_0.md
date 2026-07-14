# QNC Product Requirements & System Definition Specification

**Document Number:** QNC-PRD-001  
**Version:** 3.0 (v1 product baseline)  
**Status:** Implementation-ready baseline — supersedes PRS v2.3 for v1 scope  
**Classification:** Internal / Project Controlled  

---

## Document Change Log (v2.3 → v3.0)

| Area | Change |
|------|--------|
| Product definition | Reframed as **software-first industrial edge connectivity gateway** for NEURA cells; explicit **non-goals** reinforced |
| Architecture | Strict **three-tier** product model: **Core**, **Industrial Extensions**, **DDS Pack** |
| DDS | FastDDS is **optional QNC DDS Pack** only; never mandatory platform infrastructure |
| Profiles / config | **YAML** authoritative for authoring/governance; JSON only where runtime requires |
| Semantics | **Single canonical model** — externalized to **QNC-SEMANTIC-CORE-v1** |
| Discovery | Removed universal discovery requirement; **protocol-specific capability matrices** |
| v1 scope | **Normative removal** of Discovery Server, fleet aggregation, SCADA/MES/ERP, EtherCAT/PROFINET v1 claims, broad DDS topologies, generic vendor plugin systems |
| Portfolio | Primary alignment: **MAiRA, LARA, MAV**; MiPA / 4NE1 **secondary / future** |
| New | Packaging, capacity limits, security operations, **dual API versioning**, platform compatibility matrix, compatibility/deprecation policy summaries |

---

## 1. Purpose

This specification defines **product purpose**, **v1 scope**, **architecture**, **functional and non-functional requirements**, **governance**, **packaging**, **capacity**, **security operations**, **compatibility policy**, and **validation expectations** for **QNC (Quick Network Connector)**.

Companion normative artifacts:

| Document | Role |
|----------|------|
| **QNC-SEMANTIC-CORE-v1** | Canonical Command / Response / Telemetry / Fault / Lifecycle semantics |
| **ICD** | Interface contracts, OpenAPI/WS bindings |
| **DPS** | YAML device profile schema and governance |
| **PSM** | Protocol modules, DDS Pack isolation, capability matrices |
| **QNC-IDL** | DDS Pack IDL registry and governance (extension only) |
| **ORG** | Operations, recovery, commissioning |

---

## 2. Product Definition (v1)

### 2.1 Formal Definition

**QNC is a modular, software-first industrial edge connectivity gateway** for **NEURA** robot cells and adjacent automation environments. It provides **southbound** industrial/device integration, **YAML-governed device profiles**, **semantic normalization** to the canonical core, **robot-facing** and **northbound** APIs (REST, WebSocket, structured logging), **Safe Mode** with **fault isolation**, and **atomic configuration activation with rollback**.

### 2.2 Explicit Non-Goals

QNC **is not** and v1 **must not** be positioned as:

- a robot motion **controller**,
- a **ROS/ROS2 replacement** or universal robotics **middleware**,
- a generic **plant/cloud/MES/SCADA** platform,
- a **universal protocol translation** layer,
- a **functional safety** controller or safety-rated device.

### 2.3 Portfolio Alignment

**Primary integration references (v1):** **MAiRA**, **LARA**, **MAV** — reuse patterns for fieldbus/tooling, ROS2-related integration, Python (NeuraPy) workflows, and peripheral models.

**Secondary / future:** **MiPA**, **4NE1** — integration targets only after interface maturity and validation artifacts exist.

**v1 standardization focus:** EOAT and industrial peripherals; **mobile / ROS2 integration pathways** as **documented patterns**, not an unbounded middleware commitment.

### 2.4 Reuse Principles

Prioritize **NeuraPy**, existing **YAML** configuration approaches, commissioning UX, tool/gripper profile models, and established ROS2 integration pathways. **Do not** introduce parallel SDKs, duplicate profile ecosystems, or competing operational tooling.

---

## 3. Three-Tier Product Model (Normative)

### 3.1 QNC Core (Mandatory v1 Product)

Contains **only**:

| Capability | Description |
|------------|-------------|
| Southbound integration | Released industrial/device protocols for v1 baseline (see §5) |
| Profile governance | YAML profiles validated, signed, lifecycle-managed |
| Semantic normalization | Maps southbound data **to** QNC-SEMANTIC-CORE-v1 |
| Robot-facing API | REST/WebSocket (versioned independently) |
| Northbound API | REST/WebSocket/logging for observability (versioned independently) |
| Safe Mode & fault isolation | Operational inhibit; southbound vs. northbound vs. service domains |
| Configuration management | **Atomic** activate, **rollback**, audit |

**Core MUST NOT depend on** DDS, OPC UA, or MQTT runtime presence.

### 3.2 QNC Industrial Extensions (Optional Packages)

Optional, separately released modules, **isolated** from Core compile/runtime where feasible:

| Extension (illustrative) | Note |
|---------------------------|------|
| Modbus TCP | Extension package |
| CANopen | Extension package |
| OPC UA Server | Extension — **not** v1 Core normative requirement |
| MQTT Client | Extension — **not** v1 Core normative requirement |

Plant/MES/ERP/SCADA integration **is out of v1 normative scope** even when OPC UA/MQTT exist as extensions (no mandated enterprise semantics).

### 3.3 QNC DDS Pack (Optional Package)

**Optional** package containing:

- FastDDS participation (DomainParticipant),
- Governed IDL and QoS artifacts (**separate** governance from device profiles),
- Validated, **narrow** DDS bridges if explicitly released,
- **Isolated DDS fault domain** (`fault_domain = dds_pack` per semantic core).

DDS Pack **must** be load-fault- and upgrade-isolated from Core southbound operation.

---

## 4. Problem Statement (Abbreviated)

Industrial EOAT and peripherals expose heterogeneous protocols and semantics. Robot application teams need **fast, validated integration** without owning every low-level stack. QNC provides a **governed boundary** and **normalized semantics** aligned with NEURA ecosystem tooling.

---

## 5. v1 Baseline Scope

### 5.1 v1 Southbound (Core Baseline)

Normative **v1 baseline** southbound protocols:

- **IO-Link** (v1.1 master),
- **Modbus RTU** (serial),
- **EtherNet/IP** (adapter, Class 1 / Class 3 as released),
- **Discrete digital I/O**.

### 5.2 v1 Northbound (Core Baseline)

- **REST** (HTTPS, JSON),
- **WebSocket** (JSON),
- **Structured logging** (e.g., JSON lines; syslog mapping as deployment option).

### 5.3 Explicitly Non-Normative for v1 (Roadmap Only)

The following **MUST NOT** appear as v1 mandatory requirements:

| Item | Classification |
|------|----------------|
| DDS Discovery Server | Roadmap / future architecture note |
| Fleet-level aggregation | Roadmap |
| SCADA / MES / ERP integration | Roadmap / out of product scope for v1 |
| EtherCAT, PROFINET | Roadmap; hardware-dependent |
| Broad multi-hop DDS bridge topologies | Roadmap |
| Universal vendor extension/plugin runtime | Roadmap |
| “Universal” plant/cloud connectivity assumptions | Non-v1 |

They **MAY** appear in **Appendix — Future architecture** in companion docs; they **MUST NOT** be baseline acceptance criteria for v1.

### 5.4 Protocol-Specific Discovery / Identification

**There is no universal “discovery” requirement.**

Each southbound protocol **SHALL** document a **capability matrix** in the PSM:

| Capability | IO-Link | Modbus RTU | EtherNet/IP | DIO |
|------------|---------|------------|-------------|-----|
| Auto-discovery / enumeration | Per PSM | N/A — addressing required | Per PSM | N/A |
| Identification / binding | Yes | Unit ID + optional ID registers | Per CIP identity | Channel map |
| Hot-plug tolerance | Per PSM | Per PSM | Per PSM | Per PSM |

*(Illustrative — authoritative detail in PSM.)*

---

## 6. Functional Requirements (v1 Core Highlights)

Requirements use **REQ-CORE-xxx** for Core, **REQ-EXT-xxx** for extensions (non-v1 baseline unless package installed).

| ID | Statement | Priority |
|----|-----------|----------|
| REQ-CORE-010 | Load and validate **YAML** device profiles; reject invalid; support rollback | High |
| REQ-CORE-011 | Map southbound devices to **QNC-SEMANTIC-CORE-v1** command/telemetry/fault | High |
| REQ-CORE-020 | Expose **robot-facing** REST/WS per ICD; **versioned** independently | High |
| REQ-CORE-021 | Expose **northbound** REST/WS/logs per ICD; **versioned** independently | High |
| REQ-CORE-030 | **Atomic** configuration activation; failed activation **MUST NOT** partial apply | High |
| REQ-CORE-031 | Persist validated config; restore after restart within **≤ 30 s** cold-start budget (see §8) | High |
| REQ-CORE-040 | Enter **Safe Mode** on defined critical faults; **inhibit actuation**; publish faults | High |
| REQ-CORE-041 | Southbound faults **SHALL NOT** be conflated with DDS Pack faults | High |

**DDS Pack** (when installed): optional REQ-DDS-* set as defined in PSM and QNC-IDL — **not** part of Core conformance.

---

## 7. Non-Functional Requirements & Capacity (v1)

### 7.1 Default v1 Capacity Targets (Tunable per SKU)

| Parameter | v1 default target | Notes |
|-----------|-------------------|--------|
| Max southbound device endpoints | **8** active | Across all protocol instances |
| Max concurrent REST sessions | **16** total | Split policy robot vs northbound in deployment guide |
| Max WebSocket client connections | **8** | Configurable lower |
| DDS Pack subscribed topics | **≤ 20** | Only when DDS Pack installed |
| Stored device profiles (YAML) | **≥ 50** | On-disk governance store |
| Fault history retained | **≥ 1000** events or **7 days** whichever first | Configurable |
| Log rotation | **100 MB** default file set | **≥ 7 days** retention typical |

### 7.2 Performance (Design Targets — Evidence per V&V)

| ID | Target |
|----|--------|
| NFR-001 | REST command → device: **≤ 50 ms** p95 (profiled scenarios) |
| NFR-002 | Telemetry **≥ 10 Hz** per device default class |
| NFR-003 | Status query p95 **≤ 10 ms** (local network) |
| NFR-004 | Safe Mode entry **≤ 1 s** from critical fault |

### 7.3 Degradation

Under overload, QNC **SHOULD**: shed lower-priority northbound traffic, preserve fault/Safe Mode paths, preserve actuation inhibit semantics. Exact behavior **SHALL** be documented in PSM per module.

---

## 8. Packaging Definition (v1)

QNC v1 **SHALL** support at least one of the following deployment classes (product packaging may combine):

| Class | Description |
|-------|-------------|
| **A — Software-only** | Container or bare-metal package on approved industrial Linux |
| **B — Appliance image** | Verified OS + QNC + extensions as a **reference stack** |
| **C — Embedded runtime** | BSP-integrated binary for approved embedded target |

**Certified reference stack:** A **B** image with SBOM, pinned versions, and release notes **MAY** be positioned as the default enterprise deliverable.

Hardware SKU (DIN rail device) **MAY** wrap **B** or **C**; mechanical/power **SHALL** be specified in hardware ICD supplement.

---

## 9. Security Operations (v1 Requirements)

Extends interface security (TLS 1.2+, tokens, RBAC) with **operational** requirements:

| ID | Requirement |
|----|-------------|
| SEC-OPS-001 | **Certificate provisioning** model documented (customer PKI vs factory) |
| SEC-OPS-002 | **Token lifecycle**: issuance, refresh, revocation; max lifetime policy |
| SEC-OPS-003 | **Secrets rotation** procedure for API keys and TLS private keys |
| SEC-OPS-004 | **SBOM** (SPDX or CycloneDX) per release artifact |
| SEC-OPS-005 | **CVE response**: triage SLA, patching cadence, customer notification path |
| SEC-OPS-006 | **Third-party dependency governance**: allowlist registries, license scan gate |

---

## 10. Compatibility & Deprecation Policy (Summary)

**Full matrix:** **QNC-ENGINEERING-DELIVERABLES-v1** §Policies.

| Surface | Policy |
|---------|--------|
| Robot-facing REST/WS | MAJOR = breaking; MINOR = backward compatible additions |
| Northbound REST/WS | Stricter external notice; minimum **N-1** client support documented per release |
| YAML device profiles | Semantic versioning; migration tooling for **major** |
| DDS Pack IDL | Per QNC-IDL; **no** reliance on middleware evolution safety |
| QoS catalogs | Additive MINOR; incompatible negotiation **blocked** with fault |

---

## 11. Platform Compatibility Matrix (Governance)

Maintained as **controlled artifact** (spreadsheet or YAML registry):

| Dimension | Rows |
|-----------|------|
| Robot family | MAiRA, LARA, MAV (+ tested firmware) |
| QNC Core version | x.y.z |
| Installed extension packages | Industrial / DDS Pack modules + versions |
| Profile bundle id | Validated combination |
| Network / OS | Approved OS kernel, container runtime |

**Rule:** No marketing claim of compatibility without a matrix row + test evidence reference.

---

## 12. Configuration Governance Domains (Separated)

| Domain | Artifact type | MUST NOT |
|--------|---------------|----------|
| Device profiles | YAML per DPS | Embed DDS IDL or QoS |
| System configuration | YAML system config schema | Encode device semantic mappings |
| DDS Pack IDL | IDL + registry manifest | Encode Modbus register maps |
| DDS QoS catalog | YAML/JSON as approved | Mix device profile validation rules |
| Bridge mappings | Separate governed bundle | Imply Core without bridge package |
| OpenAPI contracts | Robot vs Northbound split | Single undifferentiated “API” |

---

## 13. Safety Position

Unchanged: **QNC is not safety-rated.** Safe Mode is **operational inhibit**, not a safety function.

---

## 14. Validation (v1)

Core validation **SHALL** include: profile YAML validation; atomic config rollback tests; southbound conformance per protocol; fault/Safe Mode; **dual API** contract tests; capacity soak **per §7**.

DDS Pack validation **only** when package is in scope.

---

## 15. Risks & Open Decisions

See **QNC-ENGINEERING-DELIVERABLES-v1** §Risks for the consolidated register.

---

## 16. Governance

Changes to **v1 baseline scope**, **three-tier model**, **semantic core linkage**, **safety position**, or **compatibility policy** require formal CCB per project rules.

---

## 17. Glossary

| Term | Definition |
|------|------------|
| **QNC Core** | Mandatory gateway capabilities without optional packages |
| **Industrial Extensions** | Optional southbound/northbound protocol packages |
| **DDS Pack** | Optional FastDDS extension package |
| **NEURA cell** | Deployed robot + peripherals + QNC integration boundary |

---

## 18. v1 Executive Summary

QNC v1 is a **focused industrial edge gateway**: validated **YAML** profiles, **canonical semantics**, **REST/WebSocket/logging**, **Safe Mode**, **atomic config**, aligned with **MAiRA/LARA/MAV**. **DDS Pack**, **OPC UA**, **MQTT**, and additional fieldbuses are **optional**, **isolated**, and **non-foundational** to Core.

---

## Related Documents

| ID | Title |
|----|--------|
| QNC-SEMANTIC-CORE-v1 | Canonical semantic model |
| ICD_v3.0 | Interface Control Document |
| DPS_v2.0 | Device Profile Specification |
| QNC-PSM_v1.0 | Protocol & Service Module Specification |
| QNC-IDL_v2.0 | DDS Pack IDL registry & governance |
| QNC-ORG_v1.1 | Operations & Recovery Guide |
| QNC-ENGINEERING-DELIVERABLES-v1 | Changelogs, matrix, policies, risks |
