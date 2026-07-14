# QNC Protocol & Service Module Specification

**Document Number:** QNC-PSM-001  
**Version:** 1.0 (v1 baseline)  
**Status:** Normative companion to **QNC-PRD-001 v3.0**  
**Classification:** Internal / Project Controlled  

---

## Document Change Log (Draft 0.1 → v1.0)

| Area | Change |
|------|--------|
| Product tiers | Reorganized into **Core**, **Industrial Extensions**, **QNC DDS Pack** with **fault isolation** |
| Semantics | All internal buses align to **QNC-SEMANTIC-CORE-v1** |
| Discovery | Per-protocol **capability matrices**; removed universal `discover()` contract |
| DDS | Participant module **only inside DDS Pack**; Discovery Server **roadmap**, not v1 |
| Deployment | Removed "Advanced DDS Multi-Network" as normative v1 mode |
| Northbound | OPC UA/MQTT **extension-only**; Core validates REST/WS/log only |

---

## 1. Purpose & Scope

Defines **module architecture**, **runtime responsibilities**, **protocol adaptation rules**, **fault domains**, **configuration artifacts**, and **validation hooks** realizing PRD v3.0 and ICD v3.0.

**Subordinate to:** PRD v3.0. **Semantic source:** QNC-SEMANTIC-CORE-v1.

---

## 2. Three-Tier Runtime Composition

| Tier | Modules |
|------|---------|
| **QNC Core** | Southbound protocol services (baseline list), Profile Resolver, Semantic Mapper, Command Broker, Lifecycle Manager, Fault Manager, State Aggregators, Northbound **REST/WS/Log** server, Config & Artifact Manager |
| **Industrial Extensions** | Modbus TCP, CANopen, OPC UA, MQTT, etc. — **separate** loadable packages |
| **QNC DDS Pack** | FastDDS Participant Service, IDL adapter, QoS resolver, DDS fault monitor, optional **approved** bridge staging — **zero hard dependency from Core** |

---

## 3. Layered Architecture (Core)

Same four-layer intent as prior draft, with **DDS Pack** shown **adjacent**, not below Core:

1. Physical & Protocol Adaptation (southbound)  
2. Profile & Semantic Normalization  
3. Integration & Control  
4. Northbound Edge Services (Core: REST, WebSocket, logging)

**DDS Pack** attaches at layer 1–3 boundary **only** via **isolated adapter** IPC/process boundary **SHOULD** for fault containment.

---

## 4. Module Table (Core Abbreviated)

| Module | Responsibility |
|--------|----------------|
| Transport Adapter | Link bring-up, retries |
| Protocol Service | State machines, transactions |
| Profile Resolver | YAML load, validate, bind |
| Semantic Mapper | Device → canonical telemetry/command/fault |
| Command Broker | Correlation, completion, inhibit |
| Fault Manager | fault_domain routing, Safe Mode |
| Northbound Server | Robot vs northbound route tables |
| Config Manager | Atomic apply, rollback generation ids |

**DDS Pack:** Participant Controller, Discovery Manager (**SIMPLE only v1**), Type Registry Adapter, QoS Resolver, Sample Router, DDS Fault Monitor.

---

## 5. Protocol-Specific Capability Matrices (Normative Pattern)

Each protocol **SHALL** publish:

| Column | Description |
|--------|-------------|
| `protocol_id` | Identifier |
| `auto_discovery` | yes / no / partial |
| `identification` | none / register-based / IO-Link PD / CIP Identity / … |
| `hot_plug` | policy |
| `max_bind_time_s` | upper target for identification phase |
| `addressing_required` | yes/no |

### 5.1 v1 Illustrative Matrix (Authoritative numbers in validation)

| Protocol | auto_discovery | identification | addressing_required |
|----------|----------------|------------------|----------------------|
| IO_LINK | partial (master-dependent) | Vendor/device PD | port map |
| MODBUS_RTU | no | optional ID registers | unit ID, serial params |
| ETHERNET_IP_ADAPTER | partial | CIP Identity objects | IP + connection path |
| DISCRETE_DIGITAL_IO | no | electrical map | channel map |

**Core SHALL NOT** implement a generic `discover()` on all protocols. **MAY** implement `identify_or_bind()` per protocol.

---

## 6. Southbound Service Internal Contract (Revised)

Replace universal `discover()` with:

- `init(adapter_config, protocol_config, profile_binding)`  
- `identify_and_bind(context)` → result: `bound` \| `unresolved` \| `unsupported`  
- `connect()`  
- `execute(...)`  
- `read_telemetry(...)`  
- `read_faults(...)`  
- `recover(policy)`  
- `shutdown()`  

---

## 7. QNC DDS Pack (Optional)

### 7.1 v1 Normative DDS Deployment

- **SIMPLE** discovery only for released v1 DDS Pack.  
- **Discovery Server:** **prohibited** as v1 mandatory feature; roadmap only.  
- **Topic budget:** ≤ 20 subscriptions unless engineering change approved.  
- **Bridge:** No normative multi-hop DDS bridge graph in v1; only **explicitly listed** bridge templates per release.

### 7.2 Fault Isolation

DDS Fault Monitor **SHALL** emit faults only with `fault_domain = dds_pack`. **SHALL NOT** transition southbound protocol instances to FAULTED solely due to DDS discovery loss.

### 7.3 Safe Mode

Subscribe-only allowed if policy enabled; writes blocked including bridge egress.

---

## 8. QoS Catalog

QoS profiles **remain** in PSM for Industrial/DDS extensions. **Core-only build** **SHALL NOT** require QoS catalog presence.

Tables from draft 0.1 **MAY** be retained as **DDS Pack appendix** — copy maintained in implementation repo; PSM references by ID.

---

## 9. Configuration Artifacts

| Artifact | Format | Owner |
|----------|--------|-------|
| System config | YAML | Platform |
| Device profile | YAML | Device integration |
| Extension manifest | YAML | Product |
| DDS Pack bundle | IDL + qos.yaml + bridge.yaml | Robotics platform |

Example skeleton **SHALL** separate `dds_pack:` block **only** in DDS-enabled deployment bundles.

---

## 10. Security & Integrity

Enforces signing, TLS, RBAC — aligns PRD SEC and SEC-OPS.

---

## 11. Validation

Core matrix: each southbound protocol row tested for identification column. DDS Pack: separate test suite.

---

## 12. Boundary with DPS

DPS owns YAML schema; PSM owns runtime interpretation and protocol matrices.

---

## Source Documents

PRD v3.0, ICD v3.0, DPS v2.0, QNC-SEMANTIC-CORE-v1, QNC-IDL_v2.0.

---

## Document Positioning

PSM v1.0 is the **implementation architecture** for modules and **eliminates universal discovery** and **Core-embedded DDS** assumptions.
