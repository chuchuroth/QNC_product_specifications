
# QNC-IDL-001  
## QNC [Fast DDS](https://fast-dds.docs.eprosima.com/en/latest/fastdds/xtypes/xtypes.html) IDL Type Registry and Governance Standard

**Document Number:** QNC-IDL-001  
**Version:** 1.1  
**Status:** Proposed Project Standard  
**Classification:** Internal / Project Controlled  
**Scope:** QNC FastDDS extension only; not part of baseline unless explicitly enabled. [Source](https://www.genspark.ai/api/files/s/6kl0bhYP)

---

## 1. Purpose

This standard defines the authoritative method by which QNC specifies, versions, validates, approves, releases, and retires DDS topic contracts implemented through the FastDDS extension. It governs the QNC-owned IDL registry, topic/type mappings, compatibility policy, QoS alignment obligations, and release controls required for safe deployment. [Source](https://www.genspark.ai/api/files/s/6kl0bhYP)

This standard exists because the source materials require an IDL Type Registry but do not provide a finalized project-level package. This document formalizes those requirements into a governed specification without expanding scope beyond QNC-owned DDS contracts. [Source](https://www.genspark.ai/api/files/s/6kl0bhYP)

---

## 2. Conformance

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **MAY**, and **RECOMMENDED** are to be interpreted as normative requirements.

A QNC DDS contract, implementation, or release is conformant only if it:

1. uses types registered under this standard,  
2. passes the mandatory validation controls in Section 9,  
3. follows the field evolution rules in Section 8,  
4. completes the approval workflow in Section 10, and  
5. is released with an approved compatibility statement and rollback plan.

---

## 3. Scope and Applicability

This standard applies only when the QNC FastDDS extension is enabled. It covers:

- QNC-owned DDS topic definitions
- QNC-owned IDL modules, enums, structs, and unions
- QNC topic-to-type mappings
- QoS profiles associated with governed topics
- compatibility decisions across releases
- lifecycle governance from proposal through retirement. [Source](https://www.genspark.ai/api/files/s/6kl0bhYP)

This standard does **not** authorize embedding third-party schemas inside QNC-owned IDL packages. External schemas **MUST** be approved, version-pinned, and referenced through a controlled adapter or bridge contract. [Source](https://www.genspark.ai/api/files/s/6kl0bhYP)

---

## 4. Normative References: [DDS-XTypes](https://www.omg.org/spec/DDS-XTypes/1.3/About-DDS-XTypes) and [Fast DDS-Gen](https://fast-dds.docs.eprosima.com/en/2.x/fastddsgen/introduction/introduction.html)

QNC adopts the DDS/XTypes model as the conceptual basis for extensible topic types. Fast DDS documents XTypes support for the DDS type system, type representations including IDL and TypeObject, wire representations, dynamic/plain language binding, and remote type discovery. [Source](https://fast-dds.docs.eprosima.com/en/latest/fastdds/xtypes/xtypes.html)

Fast DDS-Gen generates source code from IDL files by parsing a subset of OMG IDL; unsupported parts of the file are ignored. Therefore, QNC governance **MUST** restrict registry IDL to the approved subset and **MUST** require successful code generation as part of compliance. [Source](https://fast-dds.docs.eprosima.com/en/2.x/fastddsgen/introduction/introduction.html)

Fast DDS also states that type compatibility rules among evolved types are still unsupported in Fast DDS. Therefore, QNC **MUST NOT** rely solely on middleware runtime matching to prove schema evolution safety; compatibility **MUST** be established by registry policy, CI validation, and interoperability testing. [Source](https://fast-dds.docs.eprosima.com/en/latest/fastdds/xtypes/xtypes.html)

---

## 5. Registry Authority and Governance Model

The QNC IDL Type Registry is the sole authoritative catalog for:

- approved topic types
- canonical topic names
- enum definitions
- type ownership
- field evolution records
- compatibility classification
- associated QoS profile references
- lifecycle state. [Source](https://www.genspark.ai/api/files/s/6kl0bhYP)

### 5.1 Lifecycle States

Every registered artifact **MUST** be in exactly one lifecycle state:

**Draft → Under Review → Validated → Released → Deprecated → Retired**

Only artifacts in **Released** state MAY be used in production publishers or subscribers. Deprecated artifacts MAY continue to operate during the published support window. Retired artifacts MUST be blocked from new deployments. [Source](https://www.genspark.ai/api/files/s/6kl0bhYP)

### 5.2 Governance Principles

The following principles are mandatory:

- FastDDS is extension-only, not baseline.
- IDL and QoS are governed artifacts separate from device profiles.
- Only approved registry types may be activated.
- Topics are organized by function, not by implementation convenience.
- No change may create uncontrolled cross-impact failure.
- Safe Mode MUST preserve subscribe-only observability.
- Deployment changes MUST be atomic and rollback-capable. [Source](https://www.genspark.ai/api/files/s/6kl0bhYP)

---

## 6. Layered IDL Specification for [Fast DDS](https://fast-dds.docs.eprosima.com/en/latest/fastdds/xtypes/xtypes.html)

QNC SHALL organize IDL into five logical layers. Dependencies flow downward only.

```text
+------------------------------------------------------+
| L4 Transport Binding Layer                           |
| Topics, QoS profile references, bridge bindings      |
+------------------------------------------------------+
| L3 Service Contract Layer                            |
| Commands, responses, audits, diagnostics             |
+------------------------------------------------------+
| L2 Domain Contract Layer                             |
| Telemetry, faults, lifecycle, domain events          |
+------------------------------------------------------+
| L1 Data Model Layer                                  |
| Shared entities, value objects, enums                |
+------------------------------------------------------+
| L0 Foundation Layer                                  |
| CommonHeader, identifiers, timestamps, version tags  |
+------------------------------------------------------+
```

### 6.1 L0 Foundation Layer

The Foundation Layer defines universally reusable primitives and headers. It **MUST** contain only highly stable, cross-domain elements such as `CommonHeader`, version fields, identifiers, and common enums. It **MUST NOT** contain topic-specific business semantics.

**Rules**
- All transportable top-level messages MUST include `qnc::common::CommonHeader` or an approved successor.
- Foundation types MUST be dependency-free except for scalar and bounded collection types.
- Foundation changes require architecture board approval.

### 6.2 L1 Data Model Layer

The Data Model Layer defines reusable entities, value objects, and enums consumed by multiple domain contracts.

**Rules**
- Types MUST represent domain data, not transport decisions.
- All strings and sequences MUST be bounded unless a waiver is explicitly approved.
- Enums MUST be append-only.

### 6.3 L2 Domain Contract Layer

The Domain Contract Layer defines domain-facing contracts such as telemetry, fault, lifecycle, and state-bearing messages.

**Rules**
- Domain contracts MAY depend on L0 and L1 only.
- A domain contract MUST NOT depend on service/administrative types.
- Domain messages MUST remain normalized and vendor-neutral.

### 6.4 L3 Service Contract Layer

The Service Contract Layer defines request/response and operational service messages such as `CommandMessage`, `ResponseMessage`, `ConfigAuditEvent`, and diagnostic control contracts.

**Rules**
- Service contracts MAY depend on L0, L1, and L2 when necessary.
- Request/response correlation MUST use registry-defined correlation fields.
- Service contracts MUST define deterministic success/failure semantics.

### 6.5 L4 Transport Binding Layer

The Transport Binding Layer binds types to DDS topics, QoS profiles, bridge restrictions, and deployment bundles.

**Rules**
- Topic names MUST follow canonical registry mapping.
- Transport binding metadata MUST be stored separately from core type definitions.
- Bridge-specific staging topics MUST NOT leak bridge payload semantics into core domain contracts.

### 6.6 Dependency Constraints

A layer MAY depend only on lower layers. Reverse imports are forbidden.

| From | Allowed Dependencies |
|---|---|
| L4 Transport | L3, L2, L1, L0 |
| L3 Service | L2, L1, L0 |
| L2 Domain | L1, L0 |
| L1 Data Model | L0 |
| L0 Foundation | none |

Any violation is a release blocker.

---

## 7. Canonical Package, Topic, and Schema Conventions

### 7.1 Module Naming

- Module names MUST be lowercase.
- Type names MUST be PascalCase.
- Enum literals MUST be UPPER_SNAKE_CASE.
- Topics MUST use `qnc.<function>` or `qnc.<function>.<subfunction>`. [Source](https://www.genspark.ai/api/files/s/6kl0bhYP)

### 7.2 Canonical Topics

The following registry topics are normative baseline channels for the extension:

- `qnc.command`
- `qnc.response`
- `qnc.telemetry`
- `qnc.fault`
- `qnc.lifecycle`
- `qnc.service`
- `qnc.config.audit`
- `qnc.diagnostics.bulk`
- `qnc.bridge.opcua`
- `qnc.bridge.mqtt`
- `qnc.vendor.extension.*` [Source](https://www.genspark.ai/api/files/s/6kl0bhYP)

### 7.3 Schema Conventions

Each top-level message type MUST define:

- header
- semantic payload
- version traceability
- stable ownership
- compatibility classification
- associated topic binding(s)

Each field in the registry MUST have metadata:

- field name
- field type
- boundedness
- required/optional status
- introduction version
- deprecation status
- compatibility notes
- owner

---

## 8. Field-Level Versioning and Compatibility Rules

QNC adopts release versioning at the type and bundle level using `MAJOR.MINOR.PATCH`. Major indicates breaking change, Minor indicates backward-compatible extension, Patch indicates non-breaking correction. [Source](https://www.genspark.ai/api/files/s/6kl0bhYP)

### 8.1 Versioning Model

Each released type MUST have:

- **Type version**: schema release version for the type
- **Field introduction version**: first release containing the field
- **Field status**: active, deprecated, reserved, retired
- **Bundle compatibility class**: compatible, partial, incompatible. [Source](https://www.genspark.ai/api/files/s/6kl0bhYP)

### 8.2 General Evolution Rules

1. Fields **MUST NOT** be renumbered conceptually once released.  
2. Fields **MUST NOT** be removed in-place from a released production type.  
3. Deprecated fields **MUST** remain parseable until the declared retirement release.  
4. New fields **SHOULD** be appended, not inserted between existing fields.  
5. Type changes that alter semantic meaning of an existing field are **breaking**, even if the wire representation still compiles.  
6. Enum literals MAY be added only if all impacted consumers are validated against the new value handling policy.  
7. Changes to field bounds are:
   - widening: conditionally compatible
   - narrowing: breaking unless proven unused and all consumers updated
8. Changing a scalar type, key semantics, or collection shape is breaking.

### 8.3 Safe Change Matrix

| Change | Default Classification | Rule |
|---|---|---|
| Add new trailing field | Minor | Allowed if bounded and consumers tolerate absence |
| Add enum literal | Minor/Conditional | Allowed only with default handling tests |
| Clarify description/comment | Patch | No wire impact |
| Increase string/sequence bound | Minor/Conditional | Validate memory and interoperability |
| Deprecate field without removal | Minor | Must document sunset release |
| Remove field | Major | Replace with successor or new type |
| Rename field only | Major | Semantic/API break |
| Change field type | Major | Breaking |
| Change field meaning | Major | Breaking |
| Narrow bound | Major | Breaking |
| Reorder fields | Major | Prohibited in practice |

### 8.4 Deprecation Rules

A deprecated field:

- MUST remain in IDL until retirement
- MUST be documented in release notes
- MUST identify the replacement field or successor type
- MUST NOT be reused for new meaning
- SHOULD trigger warnings in generated validation reports

A retired field name becomes **reserved** and MUST NOT be reassigned.

### 8.5 Edge Cases

**Case A: Add a field needed only by new publishers**  
Allowed as Minor if the new field is bounded, appended, and older subscribers can continue operating safely when the field is absent.

**Case B: Old field was modeled incorrectly**  
Do not mutate meaning in place. Deprecate the old field, add a new field with correct semantics, publish migration notes, and require dual-write or translation during transition.

**Case C: Enum value expansion**  
Allowed only if all consumers handle unknown or newly added values safely. If any consumer maps unknown values to unsafe behavior, the change is Major.

### 8.6 Example

```idl
module qnc {
  module telemetry {
    struct TelemetryMessageV1 {
      qnc::common::CommonHeader header;
      boolean healthy;
      string<32> health_class;
    };

    // V1.1 compatible extension
    struct TelemetryMessageV1_1 {
      qnc::common::CommonHeader header;
      boolean healthy;
      string<32> health_class;
      string<64> active_profile_id; // introduced in 1.1
    };
  };
};
```

In this example, the added field is appended, bounded, and semantically additive; it qualifies as a Minor change under this standard.

---

## 9. Automated Validation and Compliance for [Fast DDS-Gen](https://fast-dds.docs.eprosima.com/en/2.x/fastddsgen/introduction/introduction.html)

Because Fast DDS-Gen parses only a subset of OMG IDL, QNC validation MUST prove that every governed artifact is expressible in the supported subset and that code generation succeeds. Because Fast DDS notes evolved-type compatibility is still unsupported, QNC CI MUST treat compatibility as a registry-governed verification activity rather than an assumed runtime feature. [Source](https://fast-dds.docs.eprosima.com/en/2.x/fastddsgen/introduction/introduction.html) [Source](https://fast-dds.docs.eprosima.com/en/latest/fastdds/xtypes/xtypes.html)

### 9.1 Mandatory Checks

Every change set MUST pass:

1. IDL syntax validation  
2. Fast DDS-Gen code generation  
3. compile and serialization round-trip tests  
4. naming convention checks  
5. boundedness checks  
6. topic/type uniqueness checks  
7. forbidden dependency checks across layers  
8. QoS profile reference validation  
9. compatibility matrix update validation  
10. N/N-1 interoperability test  
11. fault injection and Safe Mode regression  
12. atomic activation rehearsal with rollback. [Source](https://www.genspark.ai/api/files/s/6kl0bhYP)

### 9.2 Enforceable Rule Set

| Rule ID | Requirement | Enforcement |
|---|---|---|
| QNC-IDL-001-LINT-01 | all strings/sequences bounded | static linter |
| QNC-IDL-001-LINT-02 | topic names match canonical pattern | static linter |
| QNC-IDL-001-LINT-03 | no external schema embedding | AST scan |
| QNC-IDL-001-LINT-04 | no upward layer dependency | dependency analyzer |
| QNC-IDL-001-LINT-05 | every top-level message has CommonHeader | AST scan |
| QNC-IDL-001-LINT-06 | every changed type has changelog entry | CI metadata check |
| QNC-IDL-001-LINT-07 | compatibility class declared | registry validation |
| QNC-IDL-001-LINT-08 | deprecated fields have successor or rationale | policy check |
| QNC-IDL-001-LINT-09 | release includes rollback plan | release gate |
| QNC-IDL-001-LINT-10 | Safe Mode topics remain subscribe-safe | integration test |

### 9.3 Recommended Tooling

A reference implementation SHOULD include:

- custom IDL linter over parsed AST
- Fast DDS-Gen build step
- registry manifest in YAML/JSON
- compatibility diff checker
- generated docs for field history
- interoperability harness for publisher/subscriber matrix
- CI policy gates in pull requests

### 9.4 Sample CI Workflow

```text
Commit / PR
  -> IDL parse and lint
  -> Generate Fast DDS code
  -> Compile generated artifacts
  -> Run unit and round-trip serialization tests
  -> Compare registry diff
  -> Enforce compatibility policy
  -> Run N/N-1 interoperability suite
  -> Run Safe Mode and rollback rehearsal
  -> Require approvals
  -> Tag release candidate
```

### 9.5 Sample Validation Pseudocode

```yaml
stages:
  - lint
  - generate
  - build
  - test
  - compat
  - release-gate

lint:
  rules:
    - bounded_types_only
    - canonical_topic_names
    - no_upward_layer_dependencies
    - common_header_required

compat:
  rules:
    - classify_change: compatible|partial|incompatible
    - require_changelog: true
    - require_matrix_update: true
    - require_migration_notes_for_major: true
```

---

## 10. Approval and Release Process

### 10.1 Roles

**Author**  
Creates or modifies the IDL artifact and initial impact statement.

**Domain Owner**  
Confirms business semantics and topic ownership.

**IDL Steward**  
Ensures schema quality, versioning correctness, and registry integrity.

**Architecture Reviewer**  
Approves layer boundaries, compatibility classification, and external reference usage.

**Test/QA Owner**  
Owns interoperability, fault-injection, and Safe Mode evidence.

**Release Manager**  
Owns release packaging, changelog quality, activation scheduling, and rollback readiness.

### 10.2 Workflow

```text
Proposal
  -> Impact Assessment
  -> Draft IDL + Registry Metadata
  -> Technical Review
  -> Validation
  -> Approval
  -> Release Candidate
  -> Atomic Activation
  -> Audit Record
```

This expands the source governance flow into a formal gated process and preserves the required approval-before-activation principle. [Source](https://www.genspark.ai/api/files/s/6kl0bhYP)

### 10.3 Gate Criteria

A proposal MUST NOT advance unless the previous gate is complete.

| Stage | Required Evidence |
|---|---|
| Proposal | use case, owner, affected topics, rationale |
| Review | architecture impact, compatibility class, dependency review |
| Validation | all CI checks pass, test evidence attached |
| Approval | required approvers sign off |
| Release | changelog, compatibility matrix, rollback plan, release notes |
| Activation | atomic rollout plan and success criteria |
| Audit | archived artifacts and traceability links |

### 10.4 Approval Threshold

The minimum approval set MUST include:

- 1 Author
- 1 Domain Owner
- 1 IDL Steward
- 1 Architecture Reviewer
- 1 QA/Test Owner
- 1 Release Manager

No individual MAY approve in more than two of the above roles for the same change.

### 10.5 Release Documentation Requirements

Each release MUST include:

- release identifier
- affected types/topics
- semantic version changes
- compatibility classification
- migration notes
- deprecated/retired items
- rollback instructions
- test summary
- known limitations
- approval record

---

## 11. Compatibility Matrix Policy

The compatibility classes defined in the draft are retained as normative:

- **Compatible**: no breaking change; allowed
- **Partial**: restricted usage; conditional deployment
- **Incompatible**: breaking change; migration required. [Source](https://www.genspark.ai/api/files/s/6kl0bhYP)

Every release bundle MUST publish a matrix covering:

- producer version vs subscriber version
- topic-by-topic compatibility
- Safe Mode behavior
- bridge impact
- migration requirement
- rollback target

Major releases require an explicit migration plan. Minor releases require a compatibility statement. Patch releases require proof of no schema break.

---

## 12. Safe Mode and Operational Constraints

Safe Mode behavior is mandatory:

- subscribe operations MUST remain available
- actuation/command publication MAY be blocked
- fault and health reporting MUST continue
- lifecycle visibility MUST remain available. [Source](https://www.genspark.ai/api/files/s/6kl0bhYP)

Any IDL or topic change that prevents Safe Mode observability is a release blocker.

---

## 13. Submission and Change Templates

### 13.1 Change Proposal Template

| Field | Required |
|---|---|
| Proposal ID | Yes |
| Type name | Yes |
| Topics | Yes |
| Owner | Yes |
| Layer | Yes |
| Use case | Yes |
| QoS profile reference | Yes |
| Compatibility class | Yes |
| Version impact | Yes |
| Rollback plan | Yes |

### 13.2 Field Change Record Template

| Field | Value |
|---|---|
| Type | |
| Field name | |
| Old definition | |
| New definition | |
| Introduced in | |
| Deprecated in | |
| Retirement target | |
| Compatibility impact | |
| Migration action | |

### 13.3 Release Note Template

```text
Release:
Date:
Approvers:
Affected topics/types:
Compatibility class:
Major/Minor/Patch rationale:
Deprecated fields:
Retired fields:
Migration instructions:
Rollback instructions:
Validation evidence links:
```

---

## 14. Recommended Adoption Notes

For practical adoption, I recommend making three artifacts mandatory together:

1. **IDL files** as the authoritative schema source  
2. **Registry manifest** as the governance source of field history and topic mapping  
3. **CI policy pack** as the enforcement mechanism

That combination is especially important because Fast DDS supports XTypes concepts and remote type discovery, but its own documentation warns that compatibility rules among evolved types are still unsupported; project governance must therefore close the gap with explicit policy and testing. [Source](https://fast-dds.docs.eprosima.com/en/latest/fastdds/xtypes/xtypes.html)

---

## 15. Final Standard Statement

No DDS topic, type, or QoS profile within the QNC FastDDS extension SHALL be activated unless it is registered, versioned, validated, approved, release-documented, and compatibility-classified under this standard.

---
---
