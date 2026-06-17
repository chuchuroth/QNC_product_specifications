# QNC Interface Control Document

**Document Number:** QNC-ICD-001  
**Version:** 3.0 (v1 baseline)  
**Status:** Normative companion to QNC-PRD-001 v3.0  
**Classification:** Internal / Project Controlled  

---

# Document Change Log (v2.3 → v3.0)

| Area | Change |
|--------|--------|
| Semantics | All message contracts normatively reference QNC-SEMANTIC-CORE-v1 |
| APIs | Robot-facing vs northbound APIs: separate versioning and OpenAPI artifacts |
| DDS | QNC DDS Pack only; never baseline Core ICD commitment |
| Discovery | §12 rewritten: protocol-specific matrices; removed PRS FR-001 style universal discovery |
| Roadmap | EtherCAT/PROFINET/Discovery Server/fleet/SCADA explicitly non-v1 in ICD text |
| Appendices | Aligned field names to semantic core (`fault_domain`, `qnc_instance_id`) |

---

# 1. Purpose & Scope

This ICD defines external interface contracts for QNC: message shapes, state behaviors, fault publication, configuration interfaces, and binding of logical channels to REST, WebSocket, and logging.

**Normative semantic source:** QNC-SEMANTIC-CORE-v1 — this ICD defines transport, paths, error codes, and realization only.

**Subordinate to:** QNC-PRD-001 v3.0.

---

# 2. Normative Language

**SHALL / SHOULD / MAY / MUST NOT** per RFC 2119 style usage in PRS.

---

# 3. Related Documents

- QNC-PRD-001 v3.0
- QNC-SEMANTIC-CORE-v1
- DPS_v2.0
- QNC-PSM_v1.0
- QNC-IDL_v2.0 (DDS Pack)
- QNC-ORG_v1.1

---

# 4. System Context & Ownership

## 4.1 Context

```text
Robot / cell software ↔ Robot-facing API ↔ QNC Core ↔ Southbound devices

Northbound consumers ↔ Northbound API ↔ QNC
(same semantic core, different API surface and policy)
