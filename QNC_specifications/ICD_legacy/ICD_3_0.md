# QNC Interface Control Document

**Document Number:** QNC-ICD-001  
**Version:** 3.0 (v1 baseline)  
**Status:** Normative companion to **QNC-PRD-001 v3.0**  
**Classification:** Internal / Project Controlled  

---

## Document Change Log (v2.3 → v3.0)

| Area | Change |
|------|--------|
| Semantics | All message contracts **normatively reference** **QNC-SEMANTIC-CORE-v1** |
| APIs | **Robot-facing** vs **northbound** APIs: **separate** versioning and OpenAPI artifacts |
| DDS | **QNC DDS Pack** only; never baseline Core ICD commitment |
| Discovery | §12 rewritten: **protocol-specific matrices**; removed PRS FR-001 style universal discovery |
| Roadmap | EtherCAT/PROFINET/Discovery Server/fleet/SCADA explicitly **non-v1** in ICD text |
| Appendices | Aligned field names to semantic core (`fault_domain`, `qnc_instance_id`) |

---

## 1. Purpose & Scope

This ICD defines **external interface contracts** for QNC: message shapes, state behaviors, fault publication, configuration interfaces, and **binding** of logical channels to REST, WebSocket, and logging.

**Normative semantic source:** **QNC-SEMANTIC-CORE-v1** — this ICD defines transport, paths, error codes, and **realization** only.

**Subordinate to:** QNC-PRD-001 v3.0.

---

## 2. Normative Language

SHALL / SHOULD / MAY / MUST NOT per RFC 2119 style usage in PRS.

---

## 3. Related Documents

- QNC-PRD-001 v3.0  
- QNC-SEMANTIC-CORE-v1  
- DPS_v2.0  
- QNC-PSM_v1.0  
- QNC-IDL_v2.0 (DDS Pack)  
- QNC-ORG_v1.1  

---

## 4. System Context & Ownership

### 4.1 Context

**Robot / cell software** ↔ **Robot-facing API** ↔ **QNC Core** ↔ **Southbound devices**  
**Northbound consumers** ↔ **Northbound API** ↔ **QNC** (same semantic core, **different** API surface and policy)

### 4.2 Ownership

| Owner | Responsibility |
|-------|------------------|
| QNC | Southbound sessions, profile execution, normalization, publish/subscribe to **approved** topics, config integrity |
| Robot controller / MAiRA / LARA / MAV stack | Motion, safety, application sequencing |
| Device vendor | Device-internal behavior |
| DDS domain (when DDS Pack used) | Robot middleware ownership; QNC is **guest participant** |

---

## 5. v1 Interface Scope

### 5.1 Core Southbound (v1)

IO-Link, Modbus RTU, EtherNet/IP adapter, discrete DIO — **as PRD §5**.

### 5.2 Core Northbound (v1)

- **Robot-facing:** REST + WebSocket (`/robot/v{MAJOR}` prefix pattern — exact OpenAPI artifact)  
- **Northbound:** REST + WebSocket + structured logs (`/nb/v{MAJOR}` or host-separated deployment)

**SHALL:** distinct OpenAPI documents: `qnc-robot-api-vMAJOR.yaml`, `qnc-northbound-api-vMAJOR.yaml`.

### 5.3 DDS Pack

When **and only when** DDS Pack is installed and enabled: DDS peer interface per **QNC-IDL_v2.0**. **Not** baseline Core conformance.

### 5.4 Non-v1 ICD Claims

ICD **MUST NOT** require: OPC UA, MQTT, Discovery Server, fleet APIs, SCADA adapters, EtherCAT/PROFINET stacks for Core conformance.

---

## 6. Logical Channels → Canonical Core

| Logical channel | Canonical binding |
|-----------------|-------------------|
| `qnc.command` | §2.2 Command |
| `qnc.response` | §2.3 Response |
| `qnc.telemetry` | §2.4 Telemetry |
| `qnc.fault` | §2.5 Fault |
| `qnc.lifecycle` | §2.6 Lifecycle |
| `qnc.service` | Management/diagnostics operations (subset of Command category `management` + read-only resources) |

---

## 7. Physical & Electrical

Unchanged intent from v2.3: hardware supplement defines connector/part details.

---

## 8. REST & WebSocket

### 8.1 JSON Schema Rule

Request/response bodies **SHALL** be JSON objects compatible with semantic core fields; `schema_version` **SHALL** identify ICD binding version (distinct from PRD product version).

### 8.2 Error Model (HTTP)

HTTP status + body **SHALL** include machine `result_code` aligned to **Appendix B** catalog; never contradict `completion` semantics.

---

## 9. Command Model

Command categories and completion set **identical** to semantic core §2.2–2.3. ICD **SHALL NOT** add alternate completion enums.

---

## 10. Telemetry & State

State layers: Lifecycle, Connection, Protocol, Profile, Device-class, Vendor — **as v2.3**, with telemetry payload **subset** of semantic core §2.4.

---

## 11. Fault Model

### 11.1 `fault_domain` (Normative)

Fault records **SHALL** use `fault_domain` from **QNC-SEMANTIC-CORE-v1** §2.5.  
**DDS Pack** faults use **`dds_pack`** only — **MUST NOT** reuse `southbound_device` codes for DDS QoS/IDL failures.

### 11.2 Severity & Latching

Unchanged from v2.3 principles.

---

## 12. Discovery, Identification, Binding

### 12.1 Objective

Establish **supported** operation only when: **binding** (address/port/slot), **identity** (where protocol provides), and **validated profile** align.

### 12.2 Protocol Capability Statement

For each southbound protocol, the **PSM** publishes a matrix: discovery support, identification means, hot-plug, max enumeration time **where applicable**. **Absence** of discovery **SHALL NOT** be treated as defect.

### 12.3 Precedence

1. Operator-configured binding  
2. Protocol identification result (if supported)  
3. Hardware hint  
4. Profile compatibility resolution  

Ambiguity **SHALL** yield `DEGRADED` or `SAFE_MODE` per policy; **MUST NOT** silently assume support.

---

## 13. Configuration Interfaces

### 13.1 Artifact Classes (Isolated)

Per PRD §12 — system YAML, profile YAML, extension bundles — **no cross-embedding**.

### 13.2 Atomic Apply

Configuration apply **SHALL** be transactional; failure **SHALL** restore prior committed generation id.

---

## 14. Timing & Performance

Claims **SHALL** reference PRD §7 test scenarios; ICD does not duplicate numeric NFR tables.

---

## 15. Safe Mode

Matches semantic core + PRD: actuation inhibit; telemetry/fault paths remain; DDS Pack subscribe-only **if** enabled by policy.

---

## 16. Security Interfaces

TLS 1.2+, authentication for write/config, RBAC — **detailed** in OpenAPI security schemes. **SEC-OPS-*** satisfied at system level per PRD.

---

## 17. Diagnostics & Logging

Structured logs **SHALL** embed fault/telemetry subsets per logging schema artifact.

---

## 18. Versioning

- `robot_api_major` / `northbound_api_major` **independent**  
- `icd_binding_version` in each payload  
- Profile `schema_version` per DPS  

---

## 19. Verification

Core: no DDS in test scope unless DDS Pack declared. **Protocol matrix** evidence for identification/discovery columns.

---

# Appendix A — Message Headers (Illustrative JSON)

Aligned to semantic core; final field list in OpenAPI.

```json
{
  "schema_version": "icd.robot.v1",
  "message_id": "uuid",
  "correlation_id": "uuid",
  "timestamp_utc": "2026-05-12T12:00:00Z",
  "qnc_instance_id": "qnc-001"
}
```

---

# Appendix B — Error / Result Code Catalog (Structure)

| Code | fault_domain | Severity | Summary |
|------|----------------|----------|---------|
| CFG-001 | configuration | error | Invalid system or bundle schema |
| PRF-001 | profile | error | Profile YAML invalid |
| LNK-001 | southbound_connection | warning | Link unstable |
| PTL-001 | southbound_protocol | error | Session failure |
| CMD-001 | command | error | Invalid parameters |
| DDS-001 | dds_pack | error | Participant / discovery fault |
| DDS-002 | dds_pack | warning | QoS mismatch |

*Authoritative full catalog project-controlled.*

---

# Appendix C — Lifecycle States

Identical enumeration to **QNC-SEMANTIC-CORE-v1** §2.6.

---

# Appendix D — Profile Artifact Interface

Authoritative structure: **DPS_v2.0** YAML schema. ICD **MUST NOT** duplicate full profile schema.

---

# Appendix E — Conformance Checklist (v1 Core)

- [ ] Robot-facing and northbound OpenAPI split published  
- [ ] Payloads trace to semantic core  
- [ ] No DDS requirement in Core build  
- [ ] Protocol identification matrix satisfied per PSM  
- [ ] Safe Mode actuation inhibit verified  
- [ ] Atomic config rollback verified  

---

## Document Positioning

ICD v3.0 is the **interface contract** layer for v1; semantics are **centralized** in QNC-SEMANTIC-CORE-v1 to eliminate cross-document drift.
