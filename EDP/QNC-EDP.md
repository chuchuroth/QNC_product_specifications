QNC-EDP


By Chuchu Xu

4 min

Listen

2

Add a reaction
App IconApply Workflow




QNC v1 — Engineering Deliverables Package
Document Number: QNC-EDP-001  
Version: 1.0  
Date: 2026-05-12  
Classification: Internal / Project Controlled  


This package satisfies the cross-cutting deliverables requested alongside the six refined specifications. Authoritative specification text lives in the versioned documents listed in §0.

0. Refined Specification Set (File Index)
#

Deliverable

File

1 

Product Requirements & System Definition 

PRS_v3.0.md 

2 

Protocol & Service Module Specification 

QNC-PSM_v1.0.md 

3 

Interface Control Document 

ICD_v3.0.md 

4 

Device Profile Specification 

DPS_v2.0.md 

5 

DDS Pack — IDL Registry & Governance 

QNC-IDL_v2.0.md 

6 

Operations & Recovery Guide 

QNC-ORG_v1.1.md 

— 

Canonical semantic model 

QNC-SEMANTIC-CORE-v1.md 

Superseded for v1 planning: PRS_v2.3.md, ICD_v2.3.md, DPS_v1.1.md, QNC-PSM.md (draft 0.1), QNC-IDL.md (v1.1), QNC-ORG-001.md (draft 1.0) — retain for history; do not use for v1 baseline without gap review.

1. Consolidated Change Log (By Document)
Document

Summary of refinement

PRS v3.0 

Three-tier product; DDS Pack optional; YAML authority; portfolio alignment; v1 scope cuts; packaging, capacity, SEC-OPS, compatibility matrix governance 

ICD v3.0 

Dual API surfaces; semantic core binding; DDS Pack optional; discovery → protocol matrices; fault_domain isolation 

DPS v2.0 

YAML-only normative; semantic_mapping; DDS expunged from profiles; fault source_layer DDS removal 

PSM v1.0 

Tiered modules; per-protocol matrices; revised service API; DDS Pack isolation; no universal discover() 

QNC-IDL v2.0 

DDS Pack framing; SIMPLE-only v1; semantic alignment; vendor extension demotion 

ORG v1.1 

Tier-aware runbooks; SEC-OPS; DDS Pack optional path; no Discovery Server v1 

Semantic Core 

New — single Command/Response/Telemetry/Fault/Lifecycle definition 

2. Cross-Document Consistency Matrix
Topic

PRS v3.0

ICD v3.0

DPS v2.0

PSM v1.0

QNC-IDL v2.0

ORG v1.1

Semantic Core

Command semantics 

Requires REQ alignment 

REST/WS bind 

canonical_command_id 

Broker 

DDS mapping optional 

— 

Owns 

Telemetry semantics 

NFR targets 

REST/WS bind 

canonical_telemetry_key 

Mapper 

DDS L2 

Monitoring 

Owns 

Fault semantics 

Safe Mode policy 

Publication 

Southbound source_layer 

Fault mgr 

dds_pack domain 

Recovery ladder 

Owns domains 

Lifecycle 

Safe Mode 

ICD §15 

Sequences only 

Lifecycle mgr 

Safe DDS 

Safe ops 

Owns states 

YAML authority 

§12 governance 

Config apply 

Profiles 

System YAML 

DDS bundle separate 

Commissioning 

— 

DDS optional 

DDS Pack tier 

Extension only 

Forbidden in profile 

Pack module 

Owns DDS 

Optional steps 

DDS-agnostic 

Discovery 

Not universal 

§12 matrices 

binding_hints 

§5 matrices 

N/A (DDS SIMPLE) 

Per-protocol 

N/A 

API versioning 

Dual policy 

Two OpenAPIs 

compat fields 

— 

DDS semver 

— 

Namespaces 

Non-v1 roadmap 

§5.3 

§5.4 

§6 

§7 appendix 

Appendix A 

§2 DDS future 

— 

Legend: “Owns” = normative semantic source; others reference.

3. Refined Product Architecture Definition
┌─────────────────────────────────────────────────────────────────────────┐
│                         MAiRA / LARA / MAV ecosystem                     │
│              (motion, safety, ROS2 patterns, NeuraPy tooling)           │
└───────────────┬───────────────────────────────┬─────────────────────────┘
                │ Robot-facing API              │ Northbound API
                │ (version A)                   │ (version B)
                ▼                               ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                            QNC Core                                      │
│  Northbound Edge: REST / WebSocket / structured logs                     │
│  Integration & Control: Command broker, lifecycle, fault (semantic core) │
│  Profile layer: YAML validate → semantic normalization                   │
│  Southbound: IO-Link, Modbus RTU, EtherNet/IP adapter, DIO               │
└───────┬─────────────────────────────────────────────────────┬──────────┘
        │ optional packages                                     │
        ▼                                                       ▼
┌───────────────────┐                                 ┌───────────────────┐
│ Industrial Ext.   │                                 │ QNC DDS Pack       │
│ Modbus TCP,       │                                 │ FastDDS, IDL, QoS  │
│ CANopen, OPC UA,  │                                 │ isolated faults    │
│ MQTT, …           │                                 │ SIMPLE v1          │
└───────────────────┘                                 └───────────────────┘
Data plane: Southbound frames → profile mappings → canonical telemetry/fault → API fan-out (+ optional DDS writers).  
Control plane: YAML system config + profile bundle → atomic generation → rollback.  
Fault plane: fault_domain discrimination prevents DDS issues from being mis-reported as gripper faults.

4. Removed or Re-Scoped Features (v1 Normative)
Former / draft concept

v1 disposition

Universal device discovery (FR-001 style) 

Removed; per-protocol matrix 

FastDDS as strategic baseline / differentiator 

Re-scoped to optional DDS Pack 

Discovery Server mode (baseline/advanced) 

Roadmap only 

DDS-to-OPC UA / MQTT bridging as implied baseline 

Extension + explicit release per bridge 

Fleet aggregation & cluster telemetry 

Roadmap 

SCADA / MES / ERP integration 

Out of v1 product scope 

EtherCAT / PROFINET 

Roadmap 

Generic vendor extension DDS topics 

Demoted / project-controlled only 

JSON as co-equal profile authoring 

Demoted; YAML authoritative 

Single undifferentiated northbound “API” 

Split robot vs northbound 

Mixed profile/DDS governance 

Separated 

5. Compatibility & Governance Policies (Consolidated)
5.1 Semantic & ICD Binding
Semantic core MAJOR bump requires coordinated ICD binding version + mapper updates + profile schema_version policy.  

ICD robot vs northbound versions independent; changelog SHALL state consumer impact separately.

5.2 Device Profiles (YAML)
Semantic MAJOR: breaking command/telemetry meaning → new profile major; migration guide.  

MINOR: additive fields; consumers tolerate absence.  

PATCH: documentation/clarifications without behavior change.

5.3 DDS Pack (IDL / QoS)
Follow QNC-IDL_v2.0 evolution rules; assume no middleware automatic safe evolution.  

QoS negotiation failure → dds_pack fault; no silent downgrade to incompatible QoS.

5.4 Configuration Bundles
Generation id monotonic; activation atomic; rollback to N-1 generation mandatory.

5.5 Deprecation Lifecycle
Announce deprecation in release notes  

Maintain ≥ 1 release overlap for MINOR deprecations  

Retire only after telemetry shows zero critical consumers or contractual date  

6. Outstanding Risks & Unresolved Decisions
ID

Risk / decision

Suggested path

R-01 

Multicast policy in customer sites blocks DDS SIMPLE 

Architecture decision: UDP unicast peer list vs delay DDS Pack sale 

R-02 

Dual OpenAPI maintenance cost 

Codegen from single IDL-like schema vs manual sync 

R-03 

Capacity defaults vs embedded SKU reality 

Per-SKU verified matrix row 

R-04 

Third-party gripper profile velocity vs governance gates 

Tiered validation classes (dev vs production signed) 

R-05 

MAiRA/LARA/MAV version skew 

Robot compatibility matrix ownership (robot team vs QNC team) 

R-06 

DDS Pack security (DDS-Security) 

Time-boxed assessment; not v1 blocker if disabled by default 

R-07 

OPC UA / MQTT in Industrial Extensions vs cloud marketing 

Keep out of v1 normative PRD to prevent scope creep 

7. Final Implementation-Ready v1 Product Definition
Name: QNC Core v1  
One-line: A software-first industrial edge gateway for NEURA cells integrating EOAT and peripherals over released southbound protocols, governed by signed YAML profiles, exposing semantically normalized data via versioned robot-facing and northbound REST/WebSocket/logging APIs, with Safe Mode, fault-domain isolation, and atomic configuration rollback.

In scope (normative):

Protocols: IO-Link, Modbus RTU, EtherNet/IP adapter, discrete DIO  

APIs: REST + WebSocket + structured logging (two version lines)  

Profiles: YAML, semantic core traceability  

Ops: SEC-OPS checklist, capacity targets, rollback  

Packaged optionally:

Industrial Extensions (e.g., Modbus TCP, CANopen, OPC UA, MQTT)  

QNC DDS Pack (FastDDS, IDL registry, SIMPLE discovery, bounded topics)

Explicitly out of v1 normative scope:

Robot controller replacement, functional safety, universal middleware, plant/MES/SCADA, fleet platforms, EtherCAT/PROFINET, Discovery Server, universal discovery

Primary portfolio alignment: MAiRA, LARA, MAV — reuse NeuraPy and existing integration patterns.

8. Traceability Note
Requirements in PRS v3.0 map to verification cases in the (separate) V&V plan; this EDP does not replace formal req ↔ test IDs.

Document Control
Version

Date

Authoring

1.0 

2026-05-12 

Consolidated engineering refinement 


