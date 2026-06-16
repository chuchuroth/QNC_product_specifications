QNC DDS Pack — Fast DDS IDL Type Registry & Governance Standard
Document Number: QNC-IDL-001  
Version: 2.0  
Status: Normative only when QNC DDS Pack is installed  
Classification: Internal / Project Controlled  

Document Change Log (v1.1 → v2.0)
Area

Change

Product framing 

Renamed scope to QNC DDS Pack; explicit not Core 

Semantic alignment 

IDL types MUST map to QNC-SEMANTIC-CORE-v1 fields 

Discovery 

Discovery Server documentation moved to Appendix — Future; v1 SIMPLE only 

Vendor extensions 

qnc.vendor.extension.* topics non-normative for v1; remove universal vendor extension claims 

Governance 

Strengthened separation from device profile YAML governance 

Formatting 

Removed stray "Source" artifacts; consolidated tables 

1. Purpose
Defines how QNC DDS Pack specifies, versions, validates, approves, releases, and retires DDS topic contracts using Fast DDS / Fast DDS-Gen.

Applies only when DDS Pack is enabled. Does not govern device profiles.

2. Conformance
Conformant DDS Pack release MUST: use registry types; pass Section 9 validation; follow evolution rules; complete approval workflow; ship compatibility statement + rollback plan; not claim Core-only QNC compliance.

3. Scope
Covers: QNC-owned DDS topics, IDL modules, topic↔type mappings, QoS references, compatibility decisions, lifecycle states.

Does not: embed third-party IDL without adapter approval; replace YAML device profile governance.

4. Normative References
DDS-XTypes concepts; Fast DDS-Gen subset restrictions; project acknowledges runtime evolved-type compatibility limitations — compatibility is registry + CI enforced, not assumed from middleware.

5. Registry Authority
Sole catalog for: approved types, canonical topic names, ownership, field evolution, compatibility class, QoS profile reference, lifecycle state.

5.1 Lifecycle States
Draft → UnderReview → Validated → Released → Deprecated → Retired

Only Released types in production DDS endpoints.

5.2 Principles
DDS Pack optional  

IDL/QoS separate from device profiles  

Safe Mode: subscribe-safe topics mandatory  

Atomic activation with rollback for DDS configuration bundles  

6. Layered IDL Model
Layers L0–L4 as v1.1 with explicit mapping:

Layer

Maps to semantic core

L2 Domain 

Telemetry, Fault, Lifecycle payloads 

L3 Service 

Command, Response, management 

Rule: Field names SHOULD match JSON ICD fields where serializing to same semantic objects.

7. Canonical Topics (DDS Pack)
Minimum logical alignment (actual names project-controlled, pattern qnc.*):

Topic logical id

Purpose

qnc.command 

Command ingress (if used) 

qnc.response 

Response egress 

qnc.telemetry 

Telemetry egress 

qnc.fault 

Fault egress 

qnc.lifecycle 

Lifecycle 

qnc.service 

Service 

v1: Bridge topics (qnc.bridge.*) SHALL exist only if that bridge is released for the SKU.

Removed from v1 normative: unrestricted qnc.vendor.extension.* trees.

8. Versioning & Evolution
Semantic versioning per type/bundle. Rules: append fields, bounded strings/sequences, enum append-only with consumer policy, no silent semantic redefinition.

9. Mandatory CI Controls
IDL lint, Fast DDS-Gen success, compile, round-trip, boundedness, topic uniqueness, dependency layers, QoS reference validation, compatibility matrix diff, N/N-1 interop, Safe Mode subscribe-only regression, atomic activation drill.

10. Approval Roles
Author, Domain Owner, IDL Steward, Architecture Reviewer, QA Owner, Release Manager — separation of duties (max two hats per change).

11. Compatibility Matrix
Per-release matrix: producer/subscriber versions, topic-level compatibility, Safe Mode behavior, bridge impact, migration, rollback target.

12. Safe Mode
Subscribe operations MUST remain available for health/fault/lifecycle topics; actuation-related writers MUST be blocked.

13. Change Templates
Proposal, field change record, release note templates — retained from v1.1 (implementation detail in repo templates).

14. Final Statement
No DDS topic, type, or QoS profile SHALL ship in DDS Pack without registry registration, validation, approval, and compatibility classification.

Appendix A — Future (Non-v1)
DDS Discovery Server topologies  

Wide-area DDS routing  

Vendor extension marketplace  

Document Positioning
QNC-IDL v2.0 governs DDS Pack only, aligned with QNC-SEMANTIC-CORE-v1 and isolated from DPS YAML lifecycle.
