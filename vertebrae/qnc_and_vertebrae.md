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


---
---
# Vertebrae × QNC — Platform Architecture & Evolution (v1)

**Document:** VRT-ARCH-001  
**Status:** Architecture directive — extends QNC-PRD-001 v3.0 family  
**Audience:** Platform architects, TPgM, integrators, ecosystem partners  
**Classification:** Internal / Project Controlled  

---

## Executive framing

**QNC v1** (per `PRS_v3.0.md`) is intentionally a **narrow, defensible industrial edge gateway**: southbound field protocols, YAML device profiles, canonical semantics (`QNC-SEMANTIC-CORE-v1`), dual REST/WebSocket APIs, Safe Mode, atomic config.

**Vertebrae** is a **broader physical–digital ecosystem thesis**: modular end-effectors, sensors, skills, autonomy packages, and workflows from **multiple vendors**, with **capability discovery**, **sandboxed execution**, and **lifecycle governance**.

This document **does not** collapse that distinction. It defines how **QNC becomes the mandatory southbound and connectivity spine** inside a larger **Vertebrae Runtime Platform (VRP)**, while preserving QNC’s explicit **non-goals** (not a motion controller, not safety-rated, not ROS replacement).

---

## Part 1 — Analysis of the existing QNC “project”

### 1.1 Repository / artifact reality

| Expectation | Actual (workspace) |
|-------------|-------------------|
| Code monorepo | **None present** — normative **markdown specifications** + archive |
| Runtime implementation | **Out of band** — must be bootstrapped per this architecture |
| API artifacts | **Referenced** (`qnc-robot-api-vMAJOR.yaml`, `qnc-northbound-api-vMAJOR.yaml`) — not checked in here |

**Implication:** “Refactor” applies first to **specification layering and product boundaries**, then to **implementation repo** once code exists.

### 1.2 Strengths (reusable for Vertebrae)

| Asset | Reuse |
|-------|--------|
| **QNC-SEMANTIC-CORE-v1** | Becomes **Tier-1 wire semantics** for device/skill commands, responses, telemetry, faults — extend, do not fork |
| **DPS v2.0** YAML profiles | Direct model for **Vertebrae Module Manifest (VMM)** — same governance patterns (signing, semver, compatibility) |
| **Three-tier model** (Core / Industrial Ext / DDS Pack) | Template for **Vertebrae Core / Vertebrae Media Pack / Partner Packs** with **fault_domain** isolation |
| **Dual API** (robot-facing vs northbound) | Maps cleanly to **cell integrator API** vs **fleet/observability API** |
| **Atomic config + rollback** | Required for **hot-swap** and **A/B skill bundles** |
| **PSM protocol matrices** | Pattern for **no universal discovery** — replace with **attestation + binding + capability registry** |
| **PRD explicit non-goals** | **Retain** — Vertebrae must not silently re-scope QNC into motion control or functional safety |

### 1.3 Architectural bottlenecks vs Vertebrae vision

| QNC v1 constraint | Vertebrae tension |
|-------------------|-------------------|
| **Southbound-centric** — profiles map **devices**, not **skills** or **AI graphs** | Need **execution plane** above QNC |
| **No universal plugin runtime** (PRD §5.3) | Ecosystem **requires** a **governed** plugin/skill runtime — **outside** QNC Core scope but **adjacent** in VRP |
| **Capacity targets** (8 devices, bounded WS) | Multi-vendor **sensor + EE** density may require **horizontal scale** (satellite QNC instances) |
| **Command categories** limited (`profile`, `transport`, `management`) | Need **`skill`**, **`workflow`**, **`autonomy`** categories in **VRP semantic extension**, mapped to QNC where applicable |
| **DDS Pack optional** | Distributed robots may **require** DDS or Zenoh — position as **Vertebrae Media Pack**, not “optional afterthought” for AMR use cases |
| **Single QNC lifecycle** | Vertebrae needs **per-module lifecycle** + **cell-level orchestration** state machine |

### 1.4 Missing abstractions (gaps)

1. **Hardware identity & attestation** beyond protocol binding (vendor/model/serial, cryptographic identity where available).  
2. **Capability registry** — discoverable **skills**, **resources**, **interfaces**, not only device telemetry.  
3. **Execution graph** — DAG/state-machine for multi-step workflows with **budgets** (time, energy, workspace).  
4. **Version negotiation** across **robot API**, **VMM**, **skill bundle**, and **QNC core** simultaneously.  
5. **Sandbox / isolation model** for third-party code (process, cgroup, seccomp, optional micro-VM).  
6. **Simulation-first** contract (same manifests drive sim + hardware).  

### 1.5 Technical debt (spec-level)

| Issue | Remediation |
|-------|-------------|
| **Naming:** “Quick Network Connector” undersells platform role | **Product:** retain QNC as subsystem brand; **platform:** market **Vertebrae Runtime** |
| **Dual OpenAPI sync** (EDP R-02) | Single **contract-first** schema (Protobuf/JSON Schema) → codegen OpenAPI + DDS IDL slices |
| **`discovery_hints` → `binding_hints`** (DPS §8.1) | Complete rename in schema + examples |

### 1.6 Conflicts to resolve explicitly

- **Do not** position Vertebrae as “QNC + app store” without **safety and liability** articulation — PRD safety position **must remain** on all QNC-branded deliverables.  
- **Hot-swappable modules** in **physics** are not the same as **hot-reload profiles** — document **mechanical vs logical** swap semantics separately.

---

## Part 2 — Target platform architecture (VRP)

### 2.1 Layer model

```mermaid
flowchart TB
  subgraph VRP["Vertebrae Runtime Platform (VRP)"]
    subgraph Experience["Experience & workflow plane"]
      WF[Workflow engine]
      UI[Operator / fleet UI adapters]
    end
    subgraph Cognitive["Cognitive plane (optional, sandboxed)"]
      SK[Skill runtime]
      AG[Autonomy / policy plugins]
      AI[AI capability bundles]
    end
    subgraph Orchestration["Orchestration & registry"]
      CR[Capability registry]
      EG[Execution graph scheduler]
      VN[Version negotiation]
    end
    subgraph Connectivity["Connectivity spine"]
      QNC[QNC Core + extensions]
      MP[Vertebrae Media Pack - DDS / Zenoh]
    end
    subgraph South["Southbound / physics adapters"]
      HAL[HAL - buses, clocks, sync]
      DEV[Devices - EE, sensors, IO]
    end
  end
  Robot[Robot controller / MAiRA stack] -->|robot-facing API| QNC
  WF --> EG
  EG --> SK
  SK -->|bounded commands| QNC
  CR --> SK
  QNC --> HAL
  HAL --> DEV
  MP <--> QNC
  Fleet[Fleet / cloud observers] -->|northbound API| QNC
```

**Rule:** QNC remains the **only** component that speaks Modbus/IO-Link/EIP/DIO to devices in v1 alignment; skills **never** open raw southbound sockets directly.

### 2.2 Core platform layers (normative names)

| Layer | Name | Responsibility |
|-------|------|----------------|
| L0 | **Physics & HAL** | Mechanical coupling awareness, timestamps, PTP if present — **not** QNC today; **robot BSP or co-processor** |
| L1 | **QNC Core** | As PRS v3.0 — profiles, semantic normalization, APIs, Safe Mode |
| L2 | **Vertebrae Registry** | Module identity, capability manifests, compatibility, attestation results |
| L3 | **Vertebrae Orchestrator** | Execution graphs, resource locking, conflict detection |
| L4 | **Skill runtime** | Sandboxed processes, quotas, QNC command adapter |
| L5 | **Workflow / autonomy** | Multi-skill compositions, human-in-the-loop |

### 2.3 Vertebrae Interface (conceptual stack)

Formal detail: **`VERTEBRAE-INTERFACE-STANDARD-v0.1.md`**.  
Summary: **VMM** (YAML/JSON) + **VRT** (runtime token) + **VSA** (sandbox contract).

### 2.4 Plugin architecture

| Plugin class | Host | Isolation |
|--------------|------|-----------|
| **Southbound adapter** | QNC Industrial Extension | process + fault_domain |
| **DDS / Zenoh bridge** | Media Pack | existing DDS isolation |
| **Skill** | VRP Skill Runtime | cgroup + seccomp + no network default |
| **Autonomy policy** | VRP | read-only world model APIs + action budget |

### 2.5 Capability registry

Backed by: **OCI-style digest** of VMM + **semantic core version** + **robot API schema**.  
APIs: CRUD for integrators; **read-only catalog** for robot; **signed publish** for vendors.

### 2.6 Skill runtime → QNC path

Skills emit **canonical Commands** (extended schema) → **Command Broker** (existing PSM concept) → southbound.  
**Precondition:** Skill declares `requires_devices: [...]` and `max_command_rate_hz`; orchestrator enforces.

### 2.7 Execution graph model

- **Nodes:** skill step, QNC command batch, wait-for-telemetry, human gate.  
- **Edges:** conditions on `completion` + telemetry predicates.  
- **Global:** deadline, Safe Mode escalation on `fault_domain`.

### 2.8 Device / module lifecycle

```mermaid
stateDiagram-v2
  [*] --> Unbound
  Unbound --> Identified: attestation_or_binding
  Identified --> Validated: VMM_signature_OK
  Validated --> Commissioned: matrix_row_present
  Commissioned --> Active: orchestrator_enable
  Active --> Degraded: non_critical_fault
  Active --> SafeMode: critical_fault_or_policy
  Degraded --> Active: recover
  SafeMode --> Commissioned: operator_clear
```

### 2.9 Interoperability standards

| Axis | Choice |
|------|--------|
| Wire semantics | **Extend** QNC semantic core (new command categories in **VRP extension document**, not duplicated in ICD) |
| Robotics middleware | **Optional bridges** — ROS 2 topics/services **via Media Pack** only where tested |
| Fleet | **gRPC or REST** for registry — v1 can be internal-only |

### 2.10 Safety boundaries

- **Invariant:** Same as PRD — QNC Safe Mode is **operational inhibit**, not functional safety.  
- **Vertebrae adds:** **skill sandbox resource limits**, **workspace clips** (declared in VMM), **human confirmation nodes** for irreversible actions.  
- **Certification narrative:** “Platform enables **integration of certified subsystems**” — each tier has its own evidence package.

### 2.11 Distributed robotic components

- **Pattern:** One **VRP coordinator** per cell; **multiple QNC instances** (e.g., wrist-attached vs base) as **federated peers** with shared registry.  
- **Clock:** coordinated `timestamp_utc` + monotonic `sequence` per device stream.

### 2.12 Simulation-first

- **VMM** includes `simulation:` block: inertia, contact proxy, noise model IDs.  
- **CI:** profile + manifest validation + **digital twin smoke** (containerized) before `Released`.

---

## Part 3 — Production repository structure (proposal)

### 3.1 Strategy: **monorepo with strict dependency tiers**

**Rationale:** Single semver for platform releases; shared codegen from one schema repo; CI enforces DAG (`qnc-core` must not import `skill-runtime`).

```
vertebrae-platform/                 # root monorepo
  docs/
    specs/                          # normative MD + PDF exports
      qnc/                          # migrated QNC-* specs
      vertebrae/
        VERTEBRAE-INTERFACE-STANDARD.md
    adr/                            # architecture decision records
    governance/
  contracts/                        # single source of truth
    jsonschema/
    protobuf/                       # optional: internal RPC
    openapi/                        # generated qnc-robot-api, qnc-northbound-api
  packages/
    qnc-core/                       # southbound, broker, config (current QNC scope)
    qnc-industrial-ext/
    qnc-dds-pack/
    vrp-registry/                   # capability registry service
    vrp-orchestrator/
    vrp-skill-runtime/
    vrp-workflow/
    hal-adapters/                   # robot-specific; thin
  sdk/
    python/neura-vertebrae-sdk/
    cpp/vertebrae-cpp/
  sim/
    assets/
    scenarios/
  test/
    contract/
    integration/
    soak/
  deployment/
    helm/
    systemd/
    yocto/                          # optional embedded
  ci/
    github-actions/ or gitlab-ci/
    policies/                       # OPA, license allowlist
  tools/
    vmm-cli/
    profile-linter/
```

### 3.2 Multirepo alternative

Split only when: **separate release cadence** for `qnc-core` vs `vrp-*` becomes operationally painful **and** contract semver is automated. Until then, **monorepo** reduces drift (addresses EDP R-02, R-05).

### 3.3 Module ownership

| Package | Owner |
|---------|-------|
| `qnc-core` | Connectivity / fieldbus team |
| `vrp-*` | Platform / runtime team |
| `contracts/*` | Architecture board + codegen bot |
| `sdk/*` | DevEx |

### 3.4 Versioning

- **Platform release:** `VRP x.y.z` bundles pinned `qnc-core a.b.c`.  
- **Semantic core:** MAJOR bumps are **CCB-gated** (per PRS §16).  
- **VMM:** semver independent; **compatibility matrix** row = (`VRP`, `QNC`, `robot_api`, `vmm`).

### 3.5 Testing strategy

| Layer | Tests |
|-------|-------|
| Contracts | schemathesis / openapi diff in CI |
| QNC | protocol simulators, fault injection |
| VRP | sandbox escape attempts, resource exhaustion |
| E2E | hardware-in-loop matrix rows only |

### 3.6 CI/CD topology

```mermaid
flowchart LR
  subgraph CI["CI pipeline"]
    Lint[Lint + license]
    Schema[Schema validate]
    Unit[Unit tests]
    Sim[Sim smoke]
    HIL[HIL nightly]
  end
  Schema --> Unit
  Unit --> Sim
  Sim --> HIL
  HIL --> Release[Signed artifacts + SBOM]
```

### 3.7 Deployment topology

- **Edge:** VRP + QNC on cell gateway (Class B appliance per PRD §8).  
- **Satellite:** optional second QNC on mobile base — same registry, **distinct** `qnc_instance_id`.

---

## Part 4 — Documentation suite (create / migrate)

| Document | Action |
|----------|--------|
| `PRS_v3.0.md` | Add **§2.5 Vertebrae positioning** — QNC as subsystem; link VRT-ARCH-001 |
| `QNC-SEMANTIC-CORE-v1.md` | Add **Annex VRP** — new command categories (normative process: RFC) |
| New `VERTEBRAE-PLATFORM-VISION.md` | Vision, philosophy, ecosystem incentives |
| New `VERTEBRAE-DEVELOPER-GUIDE.md` | Onboarding, VMM authoring, SDK |
| New `VERTEBRAE-ECOSYSTEM-GOVERNANCE.md` | Tiers, certification, liability split |
| `QNC-ORG_v1.1.md` | Add **multi-QNC** and **VRP recovery** runbooks |

---

## Part 5 — Module dependency map

```mermaid
flowchart TD
  workflow --> orchestrator
  orchestrator --> skill_rt
  skill_rt --> registry
  skill_rt --> qnc_core
  orchestrator --> qnc_core
  qnc_core --> industrial_ext
  qnc_core --> dds_pack
  registry --> contracts
  qnc_core --> contracts
  skill_rt --> contracts
```

**Forbidden edge:** `qnc_core --> skill_rt` (no upward dependency).

---

## Part 6 — Migration strategy (current QNC architecture → VRP)

### Phase 0 — Spec alignment (4–6 weeks)

1. Freeze **QNC v1 baseline** as **L1-only** product for shipping gateway.  
2. Approve **VRP extension RFC** for semantic core command categories.  
3. Publish **VMM schema v0.1** alongside DPS (shared validation tooling).

### Phase 1 — Parallel implementation (8–12 weeks)

1. Implement **registry** + read APIs (northbound).  
2. **Skill runtime** stub: one signed “noop skill” → QNC management command.  
3. **Execution graph** MVP: linear scripts only.

### Phase 2 — Ecosystem beta (ongoing)

1. Partner **Tier B** manifests (unsigned dev / signed prod per EDP R-04).  
2. ROS 2 **shim** topic pack in Media Pack **only** with explicit matrix rows.

### Phase 3 — Scale

1. Federated QNC + registry replication.  
2. Workflow marketplace **legal + technical** gates.

### Compatibility guarantees

- **Existing YAML DPS profiles** remain valid without modification for **QNC-only** deployments.  
- **VMM** **references** `profile_id` — optional binding until skill needs device.

---

## Part 7 — Prioritized implementation roadmap

| Priority | Deliverable | Rationale |
|----------|-------------|-----------|
| P0 | **Contract-first** codegen pipeline | Eliminates dual-OpenAPI drift |
| P0 | **VMM schema + linter** | Unblocks vendor mental model |
| P0 | **Registry MVP** | Central discovery |
| P1 | **Skill runtime sandbox** | Ecosystem trust |
| P1 | **Execution graph MVP** | Workflow differentiation |
| P2 | **Second QNC federation** | Scale / AMR |
| P2 | **Simulation CI** | Velocity |
| P3 | **Public partner program** | GTM |

---

## Part 8 — MVP recommendation

**MVP:** **VRP Lite** — QNC Core unchanged + **Registry** + **linear execution** + **one** certified third-party skill path + **sandbox defaults**.  
**Non-MVP (explicit defer):** full marketplace payments, fleet aggregation, universal ROS parity.

---

## Part 9 — Long-term evolution

- **Semantic federation:** optional **industry profile registries** (e.g., gripper ontology) mapped into canonical IDs.  
- **Hardware attestation:** TPM on module → registry **trust score**.  
- **Policy bundles:** insurer-friendly **packaged risk configs**.

---

## Part 10 — Ecosystem scaling considerations

- **N×M test matrix** — automate matrix-driven CI from **single YAML registry of rows**.  
- **Regional compliance** — data residency for northbound; skill **export control** flags.  
- **Support tiers** — community vs certified vs co-engineered.

---

## Related artifacts

| ID | File |
|----|------|
| VRT-IF-001 | `VERTEBRAE-INTERFACE-STANDARD-v0.1.md` |
| QNC baseline | `PRS_v3.0.md`, `QNC-PSM_v1.0.md`, `ICD_v3.0.md`, `DPS_v2.0.md`, `QNC-SEMANTIC-CORE-v1.md` |

---

## Document control

| Version | Date | Summary |
|---------|------|---------|
| 1.0 | 2026-05-13 | Initial Vertebrae–QNC platform architecture |
