Below is a Jira-ready backlog derived from the provided QNC documents. If a substantially similar issue already exists in Jira, **update/merge it instead of creating a duplicate**; the backlog includes a specific hygiene task for closing or reclassifying outdated issues. The issue set is organized so baseline scope, approved extension scope, governance gaps, and design risks stay clearly separated. [Source](https://www.genspark.ai/api/files/s/Zz6pnjCF)

---

* **Issue Type:** Epic  
* **Summary:** Build QNC baseline core runtime and southbound protocol foundation  
* **Description:** Deliver the baseline QNC runtime aligned to the four-layer architecture, including physical/protocol adaptation, profile/semantic normalization, integration/control, and northbound readiness for released baseline scope only. This epic covers baseline southbound capabilities: IO-Link, Modbus RTU, EtherNet/IP adapter mode, and discrete digital I/O. [Source](https://www.genspark.ai/api/files/s/Zz6pnjCF)  
* **Acceptance Criteria:**  
  * Baseline runtime architecture is implemented with explicit layer boundaries.  
  * Only baseline protocols are enabled by default.  
  * Runtime exposes separate lifecycle, connection, protocol, and profile state domains.  
  * Extension-only capabilities are feature-gated and not treated as baseline.  
* **Priority:** Highest  
* **Epic (if applicable):** N/A  

---

* **Issue Type:** Story  
* **Summary:** Implement baseline southbound protocol services for released scope  
* **Description:** Implement protocol services for IO-Link v1.1 master, Modbus RTU, EtherNet/IP adapter mode (Class 1/Class 3), and configurable discrete digital I/O. Include connection establishment, state tracking, retry behavior, and conformance boundaries per released scope. [Source](https://www.genspark.ai/api/files/s/Zz6pnjCF)  
* **Acceptance Criteria:**  
  * Each baseline protocol can establish and maintain connections using released configuration models.  
  * Connection state is externally visible and logged.  
  * Reconnection follows configured retry policy.  
  * No non-baseline protocol is advertised as supported.  
* **Priority:** Highest  
* **Epic (if applicable):** Build QNC baseline core runtime and southbound protocol foundation  

---

* **Issue Type:** Story  
* **Summary:** Implement device discovery, enumeration, and normalized command/telemetry pipeline  
* **Description:** Build the baseline capability for discovering devices, loading validated profiles, translating normalized commands into protocol-native interactions, polling/subscribing to telemetry, normalizing status, and publishing faults. This is the core operational path for FR-001 through FR-007. [Source](https://www.genspark.ai/api/files/s/Zz6pnjCF)  
* **Acceptance Criteria:**  
  * Device discovery completes within the required baseline timing target.  
  * Only validated profiles can be loaded for active devices.  
  * Invalid commands are rejected with structured fault information.  
  * Normalized status and telemetry are produced for supported device classes.  
* **Priority:** Highest  
* **Epic (if applicable):** Build QNC baseline core runtime and southbound protocol foundation  

---

* **Issue Type:** Story  
* **Summary:** Implement lifecycle state model and persistent restart recovery  
* **Description:** Implement the required lifecycle states and their transitions, including BOOTING, INITIALIZING, READY, ACTIVE, DEGRADED, SAFE_MODE, SERVICE_MODE, and FAULT_LOCKED, with persistent restoration of valid configuration and prior state after restart. [Source](https://www.genspark.ai/api/files/s/x8A0x0Bn)  
* **Acceptance Criteria:**  
  * All baseline lifecycle states are implemented and externally observable.  
  * Recovery restores last valid configuration after restart within documented target time.  
  * Invalid persisted state is detected and rejected safely.  
  * Transition events are logged with timestamps and reason codes.  
* **Priority:** High  
* **Epic (if applicable):** Build QNC baseline core runtime and southbound protocol foundation  

---

* **Issue Type:** Story  
* **Summary:** Build atomic configuration artifact manager with rollback support  
* **Description:** Implement configuration handling for system config and related controlled artifacts so changes are validated before activation, activated atomically, audited, and rolled back immediately on failure. Mixed old/new configuration state must never be observable. [Source](https://www.genspark.ai/api/files/s/hDVmS7jT)  
* **Acceptance Criteria:**  
  * Invalid artifacts fail validation and do not alter the running system.  
  * Configuration activation is atomic with no partial activation state.  
  * Rollback returns the system to the last valid configuration set.  
  * All configuration changes are logged with actor, timestamp, and change summary.  
* **Priority:** Highest  
* **Epic (if applicable):** Build QNC baseline core runtime and southbound protocol foundation  

---

* **Issue Type:** Epic  
* **Summary:** Deliver QNC interface contracts and northbound integration services  
* **Description:** Define and implement the baseline interface layer for robot-facing and northbound consumers, including REST, WebSocket, logging/fault channels, and the contract-level behaviors for commands, telemetry, faults, lifecycle, and service management. [Source](https://www.genspark.ai/api/files/s/x8A0x0Bn)  
* **Acceptance Criteria:**  
  * Baseline interface contracts are documented and implemented.  
  * Command, response, telemetry, fault, lifecycle, and service semantics are preserved across bindings.  
  * Interface versioning and compatibility rules are defined.  
  * Extension interfaces remain gated until separately released.  
* **Priority:** Highest  
* **Epic (if applicable):** N/A  

---

* **Issue Type:** Story  
* **Summary:** Publish and implement REST API with OpenAPI 3.0 specification  
* **Description:** Create the baseline REST API for command submission, status query, and configuration management, with OpenAPI 3.0 documentation, versioning, and error-code definitions aligned to the ICD and PRS. [Source](https://www.genspark.ai/api/files/s/Zz6pnjCF)  
* **Acceptance Criteria:**  
  * OpenAPI 3.0 spec is published and reviewed.  
  * API supports command, status, and configuration operations in released baseline scope.  
  * Error responses are structured and mapped to documented codes.  
  * HTTPS/TLS requirements are enforced where security policy requires it.  
* **Priority:** Highest  
* **Epic (if applicable):** Deliver QNC interface contracts and northbound integration services  

---

* **Issue Type:** Story  
* **Summary:** Implement WebSocket telemetry and event streaming  
* **Description:** Deliver the baseline WebSocket channel for real-time telemetry, faults, and lifecycle events using documented JSON message types and timing behavior consistent with the northbound contract. [Source](https://www.genspark.ai/api/files/s/x8A0x0Bn)  
* **Acceptance Criteria:**  
  * Telemetry, fault, and lifecycle event message schemas are documented.  
  * Streamed events meet required timing targets for baseline telemetry and event delivery.  
  * Clients can subscribe and recover cleanly from disconnects.  
  * Schema version is included in streamed messages or channel negotiation.  
* **Priority:** High  
* **Epic (if applicable):** Deliver QNC interface contracts and northbound integration services  

---

* **Issue Type:** Story  
* **Summary:** Implement structured logging and fault publication service  
* **Description:** Build structured JSON logging and syslog-compatible fault/event publication with severity, source layer, timestamps, mode, and recommended recovery action. This supports diagnostics, observability, and external integration. [Source](https://www.genspark.ai/api/files/s/x8A0x0Bn)  
* **Acceptance Criteria:**  
  * Fault events contain code, severity, source layer, timestamp, mode, and summary.  
  * Logs are emitted in valid JSON and support configured severity levels.  
  * Critical faults latch as required until clear and recovery conditions are met.  
  * Logging output is consumable by operational monitoring tooling.  
* **Priority:** High  
* **Epic (if applicable):** Deliver QNC interface contracts and northbound integration services  

---

* **Issue Type:** Task  
* **Summary:** Close robot-side and device-side physical/electrical interface specification gaps  
* **Description:** The ICD leaves connector identifiers, signal names, voltage domains, tolerances, shielding, and some hardware-specific interface details open to companion hardware specifications. Create the missing hardware-facing contract so integration and validation teams have an authoritative reference. [Source](https://www.genspark.ai/api/files/s/x8A0x0Bn)  
* **Acceptance Criteria:**  
  * Physical connector and signal definitions exist for robot-side and end-effector-side interfaces.  
  * Electrical preconditions that trigger inhibit or Safe Mode are documented.  
  * Hardware specification is linked from the ICD and review-approved.  
  * Validation team confirms the spec is sufficient for bench integration testing.  
* **Priority:** High  
* **Epic (if applicable):** Deliver QNC interface contracts and northbound integration services  

---

* **Issue Type:** Epic  
* **Summary:** Establish device profile governance, validation, and released profile catalog  
* **Description:** Implement the full device-profile governance model defined by the DPS, including YAML artifact structure, schema validation, lifecycle states, compatibility enforcement, signing/integrity controls, and released-profile onboarding for supported device classes only. [Source](https://www.genspark.ai/api/files/s/I5FfwvKe)  
* **Acceptance Criteria:**  
  * Normative profile authoring and validation workflow is in place.  
  * Profile activation is blocked unless validity, approval, and compatibility checks pass.  
  * Released profile catalog is traceable to firmware/build compatibility.  
  * Unsupported devices are not treated as supported based on protocol reachability alone.  
* **Priority:** Highest  
* **Epic (if applicable):** N/A  

---

* **Issue Type:** Story  
* **Summary:** Implement YAML device profile schema and validator  
* **Description:** Create the normative YAML schema and validation engine for the mandatory top-level sections: metadata, identity, binding, capabilities, commands, parameters, telemetry, faults, lifecycle sequences, compatibility, validation, and security. [Source](https://www.genspark.ai/api/files/s/I5FfwvKe)  
* **Acceptance Criteria:**  
  * Schema validates required sections and required fields.  
  * Validator rejects profiles with missing mandatory fields, illegal protocol bindings, or invalid types.  
  * Validation output is human-readable and machine-consumable.  
  * Schema versioning is explicit and traceable.  
* **Priority:** Highest  
* **Epic (if applicable):** Establish device profile governance, validation, and released profile catalog  

---

* **Issue Type:** Story  
* **Summary:** Enforce profile lifecycle, compatibility, and artifact signing policy  
* **Description:** Implement profile lifecycle transitions from Draft through Retired, semantic version compatibility checks, firmware/build compatibility enforcement, and signature/integrity verification so only approved artifacts can be activated. [Source](https://www.genspark.ai/api/files/s/I5FfwvKe)  
* **Acceptance Criteria:**  
  * Lifecycle states are enforced in tooling and runtime.  
  * Profiles declare minimum/maximum firmware compatibility and activation is refused on mismatch.  
  * Signature/hash verification is required for production activation.  
  * Deprecated and retired profiles are visible but not activatable by default.  
* **Priority:** Highest  
* **Epic (if applicable):** Establish device profile governance, validation, and released profile catalog  

---

* **Issue Type:** Story  
* **Summary:** Publish authoritative device-class catalog and initial released device profiles  
* **Description:** Formalize the project-controlled device-class catalog and create the first approved released profiles for baseline supported device classes and target devices. This resolves the current gap where class examples exist but the authoritative catalog still needs governance. [Source](https://www.genspark.ai/api/files/s/I5FfwvKe)  
* **Acceptance Criteria:**  
  * Device-class catalog is versioned and review-approved.  
  * At least one released profile exists for each targeted baseline device class in scope.  
  * Each released profile includes identity match policy, capability mappings, telemetry mappings, and fault mappings.  
  * Validation evidence is attached to each released profile.  
* **Priority:** High  
* **Epic (if applicable):** Establish device profile governance, validation, and released profile catalog  

---

* **Issue Type:** Epic  
* **Summary:** Deliver governed DDS/FastDDS extension and IDL type registry  
* **Description:** Build the approved extension work needed for FastDDS participation, including governed IDL types, QoS alignment, compatibility validation, schema ownership, topic mapping, and extension-only deployment controls. This epic must remain outside baseline release unless separately validated and signed off. [Source](https://www.genspark.ai/api/files/s/Zz6pnjCF)  
* **Acceptance Criteria:**  
  * DDS capability is feature-gated as extension scope.  
  * IDL registry, QoS catalog, and participant service are all governed artifacts.  
  * DDS faults are isolated from baseline southbound services.  
  * Release sign-off criteria for DDS extension are documented and enforced.  
* **Priority:** Highest  
* **Epic (if applicable):** N/A  

---

* **Issue Type:** Story  
* **Summary:** Build QNC IDL type registry and governance workflow  
* **Description:** Implement the authoritative registry for approved DDS topic types, canonical topic names, type ownership, compatibility class, QoS references, and lifecycle state. Include workflow roles for authoring, review, validation, and release management. [Source](https://www.genspark.ai/api/files/s/bhF1rdhW)  
* **Acceptance Criteria:**  
  * Registry stores approved topics, types, ownership, lifecycle state, and QoS references.  
  * Every registered artifact is in exactly one governed lifecycle state.  
  * Change requests require compatibility classification and approval workflow.  
  * Registry output can be consumed by CI and runtime validation.  
* **Priority:** Highest  
* **Epic (if applicable):** Deliver governed DDS/FastDDS extension and IDL type registry  

---

* **Issue Type:** Story  
* **Summary:** Implement FastDDS DomainParticipant service with SIMPLE discovery baseline  
* **Description:** Deliver the FastDDS participant service for approved extension deployments, with governed configuration of domain ID, discovery mode, participant QoS, approved topic subscription/publication, and isolation of DDS faults from unrelated services. SIMPLE discovery is the baseline DDS mode; Discovery Server remains separately governed. [Source](https://www.genspark.ai/api/files/s/Zz6pnjCF)  
* **Acceptance Criteria:**  
  * DomainParticipant joins the configured DDS domain in extension-enabled builds.  
  * SIMPLE discovery mode works within documented timing expectations.  
  * Subscribed/published topics are limited to approved registry entries.  
  * DDS faults are logged and isolated from unrelated southbound operations.  
* **Priority:** High  
* **Epic (if applicable):** Deliver governed DDS/FastDDS extension and IDL type registry  

---

* **Issue Type:** Story  
* **Summary:** Implement DDS QoS catalog, schema validation, and N/N-1 interoperability CI  
* **Description:** Build the governed QoS catalog and automated validation pipeline for DDS schema/QoS correctness, code generation, compile tests, serialization round-trip tests, compatibility matrix updates, and N/N-1 interoperability checks. [Source](https://www.genspark.ai/api/files/s/bhF1rdhW)  
* **Acceptance Criteria:**  
  * CI validates IDL syntax and code generation on every change set.  
  * Schema/QoS approval is required before runtime activation.  
  * Compatibility matrix is updated as part of change review.  
  * N/N-1 interoperability tests pass before release promotion.  
* **Priority:** Highest  
* **Epic (if applicable):** Deliver governed DDS/FastDDS extension and IDL type registry  

---

* **Issue Type:** Story  
* **Summary:** Implement DDS-to-northbound bridging for validated released paths only  
* **Description:** Design and implement DDS-derived data bridging into approved northbound services such as OPC UA and MQTT, but only for explicitly validated and released mappings. The bridge must use controlled adapter contracts rather than uncontrolled schema embedding. [Source](https://www.genspark.ai/api/files/s/Zz6pnjCF)  
* **Acceptance Criteria:**  
  * Bridge paths are explicitly enumerated and feature-gated.  
  * Each bridge path has approved source topic, IDL type, QoS profile, and target schema.  
  * Bridge latency meets the documented performance target for released paths.  
  * Unsupported or unapproved DDS topics cannot be bridged.  
* **Priority:** Medium  
* **Epic (if applicable):** Deliver governed DDS/FastDDS extension and IDL type registry  

---

* **Issue Type:** Bug  
* **Summary:** Mitigate FastDDS evolved-type compatibility gap with explicit registry and migration strategy  
* **Description:** Fast DDS does not reliably provide the required runtime type-evolution safety for evolved schemas, so the project must not rely on middleware matching alone. Define and implement explicit compatibility policy, version pinning, migration patterns, and translation/dual-write strategy where incompatible correction is required. [Source](https://www.genspark.ai/api/files/s/bhF1rdhW)  
* **Acceptance Criteria:**  
  * Project publishes compatibility classes and required runtime behavior for each class.  
  * Incompatible changes require migration design before release.  
  * Translation or dual-write strategy exists for correction scenarios.  
  * Release checklist blocks schema changes lacking compatibility disposition.  
* **Priority:** Highest  
* **Epic (if applicable):** Deliver governed DDS/FastDDS extension and IDL type registry  

---

* **Issue Type:** Task  
* **Summary:** Define approved IDL linting, AST tooling, and waiver process  
* **Description:** QNC-IDL-001 requires strong validation discipline but leaves the exact linter/tooling and waiver workflow insufficiently specified. Close that governance gap so teams have a consistent enforcement toolchain. [Source](https://www.genspark.ai/api/files/s/bhF1rdhW)  
* **Acceptance Criteria:**  
  * Approved linter/toolchain is documented and adopted in CI.  
  * Waiver process includes owner, reviewer, expiry, and risk rationale.  
  * Unbounded or exceptional schema constructs cannot ship without recorded waiver.  
  * Tooling documentation is linked from the IDL governance standard.  
* **Priority:** Medium  
* **Epic (if applicable):** Deliver governed DDS/FastDDS extension and IDL type registry  

---

* **Issue Type:** Epic  
* **Summary:** Deliver reliability, Safe Mode behavior, monitoring, and operational recovery  
* **Description:** Implement QNC operational resilience, including structured fault handling, Safe Mode, fault latching, state-domain monitoring, restart strategies, commissioning workflow, rollback, and recovery procedures aligned to the Operations & Recovery Guide and PRS fault behavior. [Source](https://www.genspark.ai/api/files/s/hDVmS7jT)  
* **Acceptance Criteria:**  
  * Fault handling and Safe Mode policy are implemented and testable.  
  * Monitoring covers all required state domains.  
  * Recovery and rollback procedures are documented and executable.  
  * Operational controls prevent partially activated or incompatible artifacts.  
* **Priority:** Highest  
* **Epic (if applicable):** N/A  

---

* **Issue Type:** Story  
* **Summary:** Implement fault manager, fault latching, Safe Mode transition, and domain isolation  
* **Description:** Deliver structured fault handling across configuration, profile, connection, protocol, command validation, service/update, integrity/security, and DDS domain faults. Critical faults must trigger Safe Mode within the documented timing, while DDS faults remain isolated from unrelated services. [Source](https://www.genspark.ai/api/files/s/x8A0x0Bn)  
* **Acceptance Criteria:**  
  * Critical faults transition the system to Safe Mode within required timing.  
  * New actuation commands are rejected in Safe Mode.  
  * Critical faults latch until clear and approved recovery action occurs.  
  * DDS domain faults do not take down unrelated southbound protocol services.  
* **Priority:** Highest  
* **Epic (if applicable):** Deliver reliability, Safe Mode behavior, monitoring, and operational recovery  

---

* **Issue Type:** Story  
* **Summary:** Build operational monitoring, diagnostics, and audit coverage across all state domains  
* **Description:** Implement observability for lifecycle, connection, protocol, profile, device-class, vendor-specific, and DDS domain states, with structured logs, retention/rotation policy, and alerting hooks for critical events. [Source](https://www.genspark.ai/api/files/s/hDVmS7jT)  
* **Acceptance Criteria:**  
  * Monitoring surfaces all required state domains.  
  * Critical alerts can be triggered from structured events.  
  * Audit logs cover configuration change, activation, recovery, and rollback events.  
  * Retention and rotation settings are documented and tested.  
* **Priority:** High  
* **Epic (if applicable):** Deliver reliability, Safe Mode behavior, monitoring, and operational recovery  

---

* **Issue Type:** Story  
* **Summary:** Publish commissioning, restart, rollback, and recovery runbooks  
* **Description:** Turn the Operations & Recovery Guide into executable operational runbooks covering commissioning, pre-activation validation, atomic activation, functional verification, fault classification, restart options, rollback, and recovery verification. [Source](https://www.genspark.ai/api/files/s/hDVmS7jT)  
* **Acceptance Criteria:**  
  * Commissioning checklist exists and is review-approved.  
  * Restart procedures cover service, protocol adapter, participant, system, and rollback paths.  
  * Recovery flow includes detect, classify, lock, isolate, diagnose, recover, verify, return, and audit stages.  
  * Operations team can execute a tabletop or lab validation using the runbook.  
* **Priority:** High  
* **Epic (if applicable):** Deliver reliability, Safe Mode behavior, monitoring, and operational recovery  

---

* **Issue Type:** Bug  
* **Summary:** Prevent misuse of Safe Mode as a functional safety substitute  
* **Description:** Multiple documents explicitly state that QNC is not a safety-rated system and that Safe Mode is an operational inhibit, not certified functional safety. This must be handled as a project risk and product/ops documentation bug to prevent unsafe customer assumptions. [Source](https://www.genspark.ai/api/files/s/Zz6pnjCF)  
* **Acceptance Criteria:**  
  * Product, interface, and operations documentation all include consistent safety-boundary language.  
  * UI/API/log messaging does not imply safety certification.  
  * Release checklist includes safety-position verification.  
  * Customer-facing materials are reviewed for overclaiming.  
* **Priority:** Highest  
* **Epic (if applicable):** Deliver reliability, Safe Mode behavior, monitoring, and operational recovery  

---

* **Issue Type:** Bug  
* **Summary:** Close DDS fault-propagation risk across domain boundaries  
* **Description:** The documents require DDS domain faults to be isolated, but this is a high-risk area for robotics extension deployments. Track and resolve the design risk that DDS disconnection, QoS mismatch, or schema incompatibility could degrade unrelated southbound services. [Source](https://www.genspark.ai/api/files/s/Zz6pnjCF)  
* **Acceptance Criteria:**  
  * Failure-mode design identifies shared-resource risks and containment strategy.  
  * Fault-injection tests prove DDS failures do not cascade into unrelated southbound operations.  
  * Recovery behavior is documented for DDS-only fault cases.  
  * Residual risk is reviewed by architecture and QA owners.  
* **Priority:** Highest  
* **Epic (if applicable):** Deliver reliability, Safe Mode behavior, monitoring, and operational recovery  

---

* **Issue Type:** Epic  
* **Summary:** Establish security, validation, and release governance controls  
* **Description:** Implement the security and release-governance controls required to safely ship QNC, including TLS, authorization boundaries, controlled artifact signing, configuration integrity, formal validation evidence, approval ownership, and scope hygiene across baseline vs extension features. [Source](https://www.genspark.ai/api/files/s/Zz6pnjCF)  
* **Acceptance Criteria:**  
  * Security controls for interfaces and artifacts are defined and implemented.  
  * Validation evidence is required for release promotion.  
  * Approval ownership is assigned for each controlled document/artifact.  
  * Scope-control rules are enforceable in backlog and release reviews.  
* **Priority:** Highest  
* **Epic (if applicable):** N/A  

---

* **Issue Type:** Story  
* **Summary:** Implement TLS, RBAC/authorization, and artifact integrity enforcement  
* **Description:** Apply security hardening across northbound interfaces and artifact management, including TLS 1.2+, authorization for advanced/raw operations where applicable, and signature/hash validation for controlled artifacts before activation. [Source](https://www.genspark.ai/api/files/s/LjS08BwN)  
* **Acceptance Criteria:**  
  * HTTPS/TLS is enforced for protected interfaces.  
  * Advanced/raw command paths require authorization controls.  
  * Artifact signature/hash verification blocks unauthorized activation.  
  * Security review findings are tracked and closed before release.  
* **Priority:** Highest  
* **Epic (if applicable):** Establish security, validation, and release governance controls  

---

* **Issue Type:** Story  
* **Summary:** Create end-to-end verification and validation plan with automated test suites  
* **Description:** Produce the missing V&V plan and implement the required automated and system-level tests: protocol conformance, profile semantic validation, DDS discovery/join/leave, QoS negotiation, IDL compatibility, fault injection, Safe Mode transition, rollback/recovery, and performance benchmarking. [Source](https://www.genspark.ai/api/files/s/LjS08BwN)  
* **Acceptance Criteria:**  
  * V&V plan is published and linked to PRS/ICD/DPS/PSM/IDL requirements.  
  * Test suites cover baseline and extension requirements separately.  
  * Release gating requires passing evidence for all in-scope features.  
  * Performance and fault-injection results are archived for release review.  
* **Priority:** Highest  
* **Epic (if applicable):** Establish security, validation, and release governance controls  

---

* **Issue Type:** Task  
* **Summary:** Assign document approvals and artifact ownership for all pending TBD roles  
* **Description:** Several documents still show Product Manager, Systems Architect, Firmware Lead, Robotics Software Lead, and QA/Validation Lead approvals as TBD/Pending. Assign owners and formalize approval responsibilities across product, architecture, validation, and release governance. [Source](https://www.genspark.ai/api/files/s/x8A0x0Bn)  
* **Acceptance Criteria:**  
  * All pending approval roles are assigned named owners.  
  * Approval matrix covers PRS, ICD, DPS, PSM, IDL registry, and ops runbooks.  
  * Release workflow records required approvers and their decision points.  
  * Controlled documents are updated from TBD/Pending to named ownership status.  
* **Priority:** High  
* **Epic (if applicable):** Establish security, validation, and release governance controls  

---

* **Issue Type:** Task  
* **Summary:** Audit backlog and close or reclassify outdated scope-violating tasks  
* **Description:** Review existing Jira issues for scope drift and document-rule violations. In particular, close or reclassify issues that incorrectly treat extension features as baseline, place DDS schema/QoS inside device profiles, overclaim universal protocol support, or imply QNC is a safety controller. [Source](https://www.genspark.ai/api/files/s/I5FfwvKe)  
* **Acceptance Criteria:**  
  * Jira audit identifies duplicate, outdated, and scope-violating issues.  
  * Extension items are moved under the correct extension epic.  
  * Tasks conflicting with DPS/IDL/PRS boundaries are closed or rewritten.  
  * Audit report is attached to the epic and approved by product/architecture owners.  
* **Priority:** High  
* **Epic (if applicable):** Establish security, validation, and release governance controls  

---

* **Issue Type:** Task  
* **Summary:** Define DDS domain ID and discovery topology policy for deployment classes  
* **Description:** The operational guidance references DDS discovery topology and derived domain ID practices, but project-level deployment policy is still not fully hardened. Define approved domain ID ranges, topology selection rules, and when Discovery Server is required. [Source](https://www.genspark.ai/api/files/s/hDVmS7jT)  
* **Acceptance Criteria:**  
  * Domain ID allocation policy is documented per deployment class.  
  * SIMPLE vs Discovery Server usage rules are documented.  
  * Multicast-restricted network behavior is defined.  
  * Policy is referenced from configuration, ops, and DDS extension docs.  
* **Priority:** Medium  
* **Epic (if applicable):** Establish security, validation, and release governance controls  

---

## Recommended import/order of execution

Start with the epics in this order:  
1. **Build QNC baseline core runtime and southbound protocol foundation**  
2. **Deliver QNC interface contracts and northbound integration services**  
3. **Establish device profile governance, validation, and released profile catalog**  
4. **Establish security, validation, and release governance controls**  
5. **Deliver reliability, Safe Mode behavior, monitoring, and operational recovery**  
6. **Deliver governed DDS/FastDDS extension and IDL type registry**  

That order matches the document dependency chain: PRS/ICD/DPS baseline first, then security/validation, then operations hardening, then the FastDDS extension under governed release control. [Source](https://www.genspark.ai/api/files/s/Zz6pnjCF)

If you want, I can do the next step and turn this into one of these formats:
- CSV for Jira import
- JSON issue payloads
- a condensed “create first 15 issues only” MVP backlog
- a dependency map showing which issues block others
