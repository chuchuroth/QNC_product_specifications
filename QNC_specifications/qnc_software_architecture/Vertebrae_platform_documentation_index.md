# Vertebrae platform documentation index

| Document | Description |
|----------|-------------|
| [VERTEBRAE-QNC-PLATFORM-ARCHITECTURE-v1.md](./VERTEBRAE-QNC-PLATFORM-ARCHITECTURE-v1.md) | QNC analysis, VRP layer model, repo layout, migration, roadmap, diagrams |
| [VERTEBRAE-INTERFACE-STANDARD-v0.1.md](./VERTEBRAE-INTERFACE-STANDARD-v0.1.md) | VIS: VMM, identity, discovery, skills, lifecycle, sandbox, versioning |

Upstream QNC v1 baseline specs remain in the parent `document_refinement/` directory (`PRS_v3.0.md`, `ICD_v3.0.md`, etc.).



---
---


# Vertebrae Interface Standard (VIS)

**Document:** VRT-IF-001  
**Version:** 0.1 (draft for CCB / RFC)  
**Status:** Pre-normative — **does not** alter QNC-PRD-001 v3.0 until merged via governance  
**Tagline:** *Physical attachment meets declared capability — USB-C semantics for robot modules, without claiming functional safety.*

---

## 1. Purpose

VIS defines the **cross-vendor contract** for attaching **hardware modules** (end-effectors, sensors, tools) and loading **software capabilities** (skills, autonomy helpers, workflows) onto a **Vertebrae-compliant cell**, using **QNC** as the **mandatory southbound normalization layer** where field devices are involved.

**Goals:**

- **Capability discovery** without universal bus discovery (inherits PSM philosophy).  
- **Version negotiation** across hardware, firmware, profiles, and APIs.  
- **Lifecycle hooks** from bind → active → safe → decommission.  
- **Telemetry and command symmetry** aligned with **QNC-SEMANTIC-CORE-v1** (extend, do not fork).  
- **Sandboxed** execution for third-party logic.  
- **Interoperability** with simulation and CI.

**Non-goals:**

- Functional safety certification of VIS itself.  
- Replacing robot motion controllers.  
- Mandating a single mechanical connector geometry (VIS is **logical**; mechanical ICD is a **separate supplement** per coupling class).

---

## 2. Normative artifacts

| Artifact | Format | Authority |
|----------|--------|-----------|
| **VMM** — Vertebrae Module Manifest | YAML (authoritative), signed | Vendor / integrator |
| **QNC Device Profile** | YAML per **DPS v2.0** | Required when module has southbound devices |
| **VRT** — Vertebrae Runtime Token | JWT or signed JSON (implementation) | Issued by registry after validation |
| **VIS Schema** | JSON Schema / CDDL | `contracts/` in platform repo |

---

## 3. Identifiers

| Field | Description |
|-------|-------------|
| `vertebrae_module_id` | Globally unique string, e.g. `vrt.vendor.product` |
| `module_version` | Semver of **manifest + firmware bundle** |
| `hardware_revision` | Opaque HW rev |
| `qnc_profile_ids[]` | Bound DPS profiles (optional if module is software-only) |
| `qnc_instance_id` | As semantic core — target gateway |
| `schema_version` | VIS binding version |

---

## 4. Hardware identity

### 4.1 Identity sources (priority)

1. Cryptographic attestation (TPM / secure element) **when present**  
2. Protocol identity (IO-Link PD, CIP Identity, Modbus registers) per **PSM matrix**  
3. Operator binding (serial, port map) — **lowest trust tier**

### 4.2 `hardware_identity` block (normative shape)

```yaml
hardware_identity:
  module_id: "vrt.acme.gripper-x100"
  module_version: "1.2.0"
  hardware_revision: "R3"
  serial_number: "SN-optional"
  attestation:
    tier: "none"   # none | basic | tpm_signed
    certificate_id: null
  coupling_class: "WRT-1"   # mechanical ICD supplement id; not VIS scope to define geometry
```

---

## 5. Capability discovery

### 5.1 Declared capabilities

Each module **SHALL** declare **capabilities** as typed entries:

| `capability_type` | Meaning |
|-------------------|---------|
| `device` | Southbound device represented via QNC profile |
| `sensor_stream` | High-rate data path (may map to telemetry keys) |
| `skill` | Executable bundle reference |
| `autonomy_hint` | Non-actuating advisory (maps to sandbox class) |
| `workflow_template` | Predefined execution graph fragment |

### 5.2 Example fragment

```yaml
capabilities:
  - capability_id: "grip"
    capability_type: "device"
    qnc_profile_id: "qnc.gripper.acme.x100"
    required_protocols: ["MODBUS_RTU"]
  - capability_id: "slip_detect"
    capability_type: "skill"
    skill_package_digest: "sha256:…"
    resource_class: "S1"
```

---

## 6. Skill declaration

Skills **SHALL** ship as **immutable bundles** containing:

| Component | Required |
|-----------|----------|
| `manifest.yaml` (skill section of VMM pattern) | Yes |
| Compiled bytecode / container image digest | Yes |
| **Resource manifest** (CPU, RAM, fds, network allowlist) | Yes |
| **Action budget** (max concurrent commands, rate Hz) | Yes |
| Self-test vectors (optional, Tier A partners) | Recommended |

**Skill → QNC:** Skills call **only** the **robot-facing API** or an **in-process adapter** provided by VRP that forwards to QNC Command Broker — **no raw socket** access to southbound.

---

## 7. Lifecycle hooks

Hooks are **observed events** and **allowed transitions**, not free-form callbacks into vendor code at arbitrary points in QNC Core.

| Hook | Emitter | Consumer |
|------|---------|----------|
| `on_bind_started` | Registry | Orchestrator |
| `on_profile_validated` | QNC | Registry |
| `on_ready` | QNC lifecycle | Orchestrator |
| `on_active` | Orchestrator | Skills |
| `on_degraded` | Fault manager | All |
| `on_safe_mode` | Fault manager | All |
| `on_unbind` | Operator / policy | Registry |

**Wire encoding:** extend **`qnc.lifecycle`** channel with **`vertebrae_context`** object (VRP extension RFC — fields must be additive MINOR).

---

## 8. Telemetry

- **Device telemetry** remains **QNC canonical telemetry** per profile mapping.  
- **Module-level telemetry** (e.g., skill CPU, graph position) uses **`capability_type: sensor_stream`** with keys under `vrt.telemetry.*` namespace — **registered** in capability registry, not ad hoc.

---

## 9. Command contracts

### 9.1 Extension to semantic core

New **`command_category` values** (proposed — require CCB):

- `skill` — invoke declared skill step  
- `workflow` — control execution graph  
- `registry` — publish, revoke, query manifests  

**Rule:** VRP command extensions **MUST** map outcomes to existing **`completion`** and **`result_code`** patterns in **QNC-SEMANTIC-CORE-v1 §2.2–2.3**.

### 9.2 `skill` command (illustrative JSON)

```json
{
  "header": { "schema_version": "vrt.v0", "message_id": "…", "timestamp_utc": "…", "qnc_instance_id": "qnc-001" },
  "command_category": "skill",
  "command_id": "vrt.skill.invoke",
  "target": { "module_id": "vrt.acme.gripper-x100", "capability_id": "slip_detect" },
  "parameters": { "sensitivity": 0.7 },
  "timeout_ms": 500
}
```

---

## 10. Safety constraints (operational, not SIL)

Each VMM **SHALL** include:

```yaml
safety_constraints:
  operational_inhibit_on_fault: true
  workspace_clip_ids: ["cell.kitchen.zone_a"]
  max_end_effector_speed_m_s: 0.5
  forbidden_command_ids: []
  requires_human_gate_for: ["vrt.commands.irreversible_cut"]
```

**Enforcement:** Orchestrator + robot controller policy; QNC Safe Mode remains **inhibit actuation** per PRD.

---

## 11. State synchronization

- **Authoritative device state:** QNC telemetry stream.  
- **Authoritative graph state:** Orchestrator.  
- **Registry:** manifests and digests — **not** real-time motion truth.

**Clocks:** All published records carry `timestamp_utc`; cross-stream ordering uses **`sequence`** per source (VRP adds monotonic sequence for graph events).

---

## 12. Version negotiation

**Handshake order:**

1. `min_vrp_version` / `max_tested_vrp_version` in VMM  
2. `min_qnc_core_version` (inherit DPS compatibility pattern)  
3. `required_robot_api_schema`  
4. Media pack presence if `requires_dds: true`

**Failure:** `rejected` with `result_code` **`vrt.version_mismatch`** (stable code table — TBD in ICD annex).

---

## 13. Compatibility management

| Change type | Rule |
|-------------|------|
| Additive telemetry keys | MINOR VMM + MINOR registry |
| New required capability | MAJOR VMM |
| Breaking command semantics | Semantic core MAJOR + coordinated ICD |

---

## 14. Sandbox & security isolation

| Class | Network | FS | Devices |
|-------|---------|-----|---------|
| `S0` | none | read-only bundle | QNC API only |
| `S1` | egress allowlist | tmp rw | QNC API |
| `S2` | **disallowed default** | extended | human review |

**Supply chain:** Bundle digest verified; **SBOM** SPDX per PRD SEC-OPS.

---

## 15. Hot-swap semantics

| Mode | Definition |
|------|------------|
| **Logical hot-swap** | VMM deactivate → Safe Mode if needed → new VMM atomic apply (QNC rollback semantics) |
| **Physical hot-swap** | Mechanical + electrical procedure documented in hardware ICD; QNC **identify_and_bind** per PSM |

**MUST NOT** claim uninterrupted end-effector torque during physical swap.

---

## 16. Relationship to Kubernetes / ROS analogies

| Concept | VIS realization |
|---------|-----------------|
| CRD | **VMM** types versioned in registry |
| Admission webhook | **Signature + matrix validation** before `Active` |
| Pod | **Skill bundle** in sandbox class |
| CNI | **Network policy** per class S* |

ROS: **optional** `vrt_msgs` or IDL in Media Pack — **not** part of VIS core conformance.

---

## 17. Conformance classes

| Class | Description |
|-------|-------------|
| **VIS-A** | QNC device only — VMM + profiles, no skills |
| **VIS-B** | A + declared skills S0 |
| **VIS-C** | B + workflows + federation |

---

## 18. Open items (v0.2 targets)

- [ ] Stable `result_code` table for VRP layer  
- [ ] JSON Schema publication for VMM  
- [ ] `vertebrae_context` ICD appendix  
- [ ] ROS 2 message mapping (informational)  
- [ ] Liability & tiering appendix (legal review)

---

## Document control

| Version | Date | Summary |
|---------|------|---------|
| 0.1 | 2026-05-13 | Initial VIS draft aligned to QNC v1 family |
