QNC Device Profile Specification

Document Number: QNC-DPS-001  
Version: 2.0 (v1 baseline)  
Status: Normative companion to QNC-PRD-001 v3.0  
Classification: Internal / Project Controlled  

Document Change Log (v1.1 → v2.0)

Area

Change

YAML 

Only normative authoring format; JSON allowed only as compiled/runtime export with checksum 

Semantics 

Commands/telemetry/fault targets explicitly map to QNC-SEMANTIC-CORE-v1 

DDS 

Removed any implication that profiles reference DDS types; strict boundary 

Fault mapping 

source_layer values aligned with fault_domain taxonomy; no dds inside device profile fault source — DDS faults only in DDS Pack 

Compatibility 

References robot API and northbound API schema versions separately 

Conformance 

v1 profiles MUST NOT require roadmap-only protocols 

1. Purpose & Scope

Defines YAML device profile structure, validation, governance, lifecycle, and conformance for southbound devices.

Out of scope: DDS IDL, QoS, bridge maps, OpenAPI — separate artifacts.

2. Normative Language

SHALL / SHOULD / MAY / MUST NOT.

3. Related Documents

PRD v3.0, ICD v3.0, PSM v1.0, QNC-SEMANTIC-CORE-v1, QNC-IDL_v2.0 (informational boundary only).

4. Profile Role

Profiles are the declarative bridge from vendor protocol semantics into canonical Command, Telemetry, and Fault semantics (semantic core). They MUST NOT redefine lifecycle or Safe Mode.

5. Architectural Boundary

Profiles SHALL NOT contain: executable scripts, network-wide discovery assertions, DDS definitions, northbound endpoint definitions.

6. YAML as Authoritative Format

Authoring, review, CI, and release SHALL use YAML.  

Runtime MAY compile to internal JSON or binary; build SHALL record artifact_hash of canonical YAML.  

MUST NOT treat JSON hand-edits as source of truth without YAML reconciliation.

7. Top-Level Sections (Required)

profile_metadata:
device_identity:
protocol_binding:
semantic_mapping:      # NEW v2: explicit binding to canonical core
capabilities:
commands:
parameters:
telemetry:
faults:
lifecycle_sequences:
compatibility:
validation:
security:

7.1 semantic_mapping (v2 Required)

Purpose: Declare how profile fields attach to canonical semantics.

semantic_mapping:
  canonical_command_namespace: "qnc.v1.commands.gripper"
  canonical_telemetry_namespace: "qnc.v1.telemetry.gripper"
  canonical_fault_namespace: "qnc.v1.faults.vendor"

Rule: Every commands[] entry SHALL declare canonical_command_id matching semantic registry row (project-controlled enum table).

8. Protocol Binding

Exactly one southbound protocol per profile; identifiers SHALL match released Core or Industrial Extension for target build.

MUST NOT bind EtherCAT/PROFINET in v1 profile corpus unless separately released (expected: false for v1).

8.1 discovery_hints → binding_hints (Rename Recommended)

Optional hints SHALL NOT be named in a way that implies guaranteed discovery; prefer binding_hints in new profiles.

9. Capability & Command Models

Commands SHALL include:

canonical_command_id (string, registered)  

command_category: profile | transport (transport only if policy allows)  

allowed_lifecycle_states list aligned to semantic core §2.6  

10. Telemetry

Each telemetry field SHOULD declare canonical_telemetry_key for normalized export to REST/WS.

11. Fault Mapping

Each fault SHALL map to:

qnc_fault_code  

severity  

source_layer: protocol | device | profile only (southbound scope)  

safe_mode_required boolean  

MUST NOT use source_layer: dds in device profiles.

12. Lifecycle Sequences

Bounded declarative steps only — same rules as v1.1.

13. Compatibility

compatibility SHALL include:

compatibility:
  min_qnc_core_version: "3.0.0"
  max_validated_qnc_core_version: "3.x"
  required_robot_api_schema: "robot.v1"
  required_northbound_api_schema: "nb.v1"
  protocol_adapter_constraints: []

14. Validation & Approval

Unchanged governance intent: no Released without tests + signatures per PRD.

15. Security

YAML signing SHALL be mandatory for production profiles.

16. DDS & Northbound

Single paragraph normative: Device profiles apply to southbound only. Any relationship to DDS Pack is indirect (normalization may feed internal bus consumed by bridge code) but SHALL NOT be authored inside the profile file.

17. Conformance

Profile is conformant if: YAML validates against schema; semantic_mapping complete; protocol released; no forbidden protocols; signature valid.

Appendix A — Normative YAML Skeleton

profile_metadata:
  profile_id: "qnc.gripper.vendor.model"
  profile_version: "2.0.0"
  profile_status: "Draft"
  owner_team: "Integration"
  created_utc: "2026-05-12T00:00:00Z"
  updated_utc: "2026-05-12T00:00:00Z"
  change_summary: "v2 semantic mapping"

semantic_mapping:
  canonical_command_namespace: "qnc.v1.commands.gripper"
  canonical_telemetry_namespace: "qnc.v1.telemetry.gripper"
  canonical_fault_namespace: "qnc.v1.faults.vendor"

device_identity:
  vendor: "Vendor"
  model: "Model"
  device_class: "GRIPPER_ELECTRIC"

protocol_binding:
  protocol: "MODBUS_RTU"
  transport_profile: "serial_default"
  addressing_mode: "unit_id"
  session_requirements: {}
  communication_defaults: {}

commands: []
parameters: []
telemetry: []
faults: []
lifecycle_sequences: {}
compatibility: {}
validation: {}
security:
  signature_required: true

Document Positioning

DPS v2.0 aligns device governance with YAML authority, semantic core traceability, and strict DDS separation.
