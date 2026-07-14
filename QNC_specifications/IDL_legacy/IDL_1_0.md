# QNC FastDDS IDL Type Registry

**Document Number:** QNC-IDL-001
**Version:** 1.0
**Status:** Proposed authoritative registry for approved DDS IDL topic types
**Classification:** Internal / Project Controlled
**Scope:** QNC FastDDS extension only (not baseline by default)

---

## Important Note

The source materials require an IDL Type Registry but do not provide a finalized IDL package. This document formalizes those requirements into a governed, authoritative draft without altering scope boundaries.

---

# 1. Registry Overview

The QNC FastDDS IDL Type Registry is the authoritative catalog of DDS topic types, including:

* Enums
* Versioning
* Compatibility rules
* Governance lifecycle

### Purpose

Mandated by PRS to govern:

* DDS topics
* QoS profiles
* Compatibility matrix

### Scope Rules

* Applies only when FastDDS extension is enabled
* Covers QNC-owned DDS contracts only
* External schemas must be approved and referenced (not embedded)

---

## 1.1 Governance Model

Lifecycle:

```
Draft → Under Review → Validated → Released → Deprecated → Retired
```

Rules:

* Only approved types can be activated
* Changes must be atomic
* Rollback required on failure

---

## 1.2 Design Principles

| Principle           | Rule                                    |
| ------------------- | --------------------------------------- |
| Scope control       | FastDDS is extension-only               |
| Artifact separation | IDL + QoS separate from device profiles |
| Approved-only       | Only registry types allowed             |
| Organization        | Function-based topics                   |
| Fault isolation     | No cross-impact failures                |
| Safe Mode           | Subscribe-only allowed                  |
| Atomic changes      | All-or-nothing deployment               |

---

# 2. IDL Type Catalog

## 2.1 Catalog Summary

Defined channels:

* `qnc.command`
* `qnc.response`
* `qnc.telemetry`
* `qnc.fault`
* `qnc.lifecycle`
* `qnc.service`

---

## 2.2 Canonical Topic Map

| Topic                  | Type                    | Class            | Bridge  | Notes             |
| ---------------------- | ----------------------- | ---------------- | ------- | ----------------- |
| qnc.command            | CommandMessage          | Command-Ingress  | No      | Robot → QNC       |
| qnc.response           | ResponseMessage         | Response         | No      | Deterministic     |
| qnc.telemetry          | TelemetryMessage        | Telemetry-Egress | Yes     | Normalized        |
| qnc.fault              | FaultMessage            | Fault-Egress     | Limited | Structured faults |
| qnc.lifecycle          | LifecycleEvent          | Status           | Yes     | State changes     |
| qnc.config.audit       | ConfigAuditEvent        | Service          | No      | Audit trail       |
| qnc.diagnostics.bulk   | DiagnosticSample        | Diagnostics      | No      | High volume       |
| qnc.bridge.opcua       | OpcUaBridgeSample       | Bridge           | N/A     | OPC UA staging    |
| qnc.bridge.mqtt        | MqttBridgeSample        | Bridge           | N/A     | MQTT staging      |
| qnc.vendor.extension.* | VendorExtensionEnvelope | Extension        | No      | Controlled        |

---

## 2.3 IDL Package (Excerpt)

```idl
module qnc {
  module common {

    enum ValueKind {
      VK_STRING,
      VK_INT64,
      VK_UINT64,
      VK_FLOAT64,
      VK_BOOL,
      VK_JSON
    };

    struct CommonHeader {
      string<16> schema_version;
      string<64> message_id;
      string<64> correlation_id;
      string<32> timestamp_utc;
      string<64> qnc_id;
    };

    struct HealthSummary {
      boolean healthy;
      boolean degraded;
      string<32> health_class;
      string<64> active_profile_id;
      string<16> qnc_software_version;
      string<128> summary;
    };
  };
};
```

*(Full IDL retained from source; truncated here for readability)*

---

## 2.4 Type Usage Summary

| Type             | Purpose           | Topic         |
| ---------------- | ----------------- | ------------- |
| CommonHeader     | Required metadata | All           |
| CommandMessage   | Command input     | qnc.command   |
| ResponseMessage  | Command result    | qnc.response  |
| TelemetryMessage | State + health    | qnc.telemetry |
| FaultMessage     | Fault reporting   | qnc.fault     |
| LifecycleEvent   | State transitions | qnc.lifecycle |

---

# 3. Versioning Strategy

Uses **Semantic Versioning (SemVer)**:

```
MAJOR.MINOR.PATCH
```

### Rules

| Type  | Meaning             |
| ----- | ------------------- |
| Major | Breaking change     |
| Minor | Backward-compatible |
| Patch | Non-breaking fix    |

---

## 3.2 Compatibility Classes

| Class        | Meaning            | Allowed     |
| ------------ | ------------------ | ----------- |
| Compatible   | No breaking change | Yes         |
| Partial      | Restricted usage   | Conditional |
| Incompatible | Breaking change    | No          |

---

# 4. Compatibility Matrix

## 4.1 Bundle Matrix

| Bundle           | Core  | Device | Service | Bridge | Status       |
| ---------------- | ----- | ------ | ------- | ------ | ------------ |
| RSW-DDS-1.0      | 1.0.x | 1.0.x  | 1.0.x   | 1.0.x  | Compatible   |
| RSW-DDS-1.0-Lite | 1.0.x | 1.0.x  | 1.0.x   | —      | Partial      |
| RSW-DDS-2.x      | 2.x   | 2.x    | 2.x     | 2.x    | Incompatible |

---

## 4.2 Upgrade Rules

| Scenario | Action                 |
| -------- | ---------------------- |
| Patch    | Rolling update allowed |
| Minor    | Allowed if compatible  |
| Major    | Requires migration     |
| Failure  | Automatic rollback     |

---

# 5. Governance Workflow

## 5.1 Process

1. Proposal
2. Impact assessment
3. Draft schema
4. Technical review
5. Validation
6. Approval
7. Atomic activation

---

## 5.2 Approval Criteria

* Valid IDL syntax
* Bounded types
* Approved QoS
* Compatibility updated
* Tests passed
* Safe Mode validated

---

## 5.3 Deprecation Policy

| State               | Meaning           |
| ------------------- | ----------------- |
| Deprecated          | Still usable      |
| Retired             | Blocked           |
| Emergency withdrawn | Immediate removal |

---

# 6. Validation & Compliance

## 6.1 Required Checks

* IDL syntax validation
* Naming compliance
* Boundedness
* QoS alignment
* Compatibility check
* Interoperability testing
* Fault injection testing
* Atomic activation rehearsal

---

## 6.2 Naming Conventions

| Item        | Rule             |
| ----------- | ---------------- |
| Namespace   | lowercase        |
| Type        | PascalCase       |
| Enum values | UPPER_SNAKE_CASE |
| Topics      | `qnc.<function>` |

---

# 7. Usage Guidelines

## Best Practices

* Use core types
* Reuse `CommonHeader`
* Keep vendor data separate
* Bind QoS before activation

## Anti-Patterns

| Anti-pattern           | Reason               |
| ---------------------- | -------------------- |
| IDL in device profiles | Violates separation  |
| Reusing vendor fields  | Breaks normalization |
| Unapproved QoS         | Not allowed          |
| Enum changes           | Break compatibility  |

---

## Safe Mode Behavior

* Subscribe allowed
* Actuation blocked
* Fault/health reporting continues

---

# 8. Appendices

## A. Submission Template

| Field       | Required |
| ----------- | -------- |
| Proposal ID | Yes      |
| Type name   | Yes      |
| Topics      | Yes      |
| Owner       | Yes      |
| Use case    | Yes      |
| QoS         | Yes      |

---

## B. Example

```idl
struct ExampleServiceNotice {
  qnc::common::CommonHeader header;
  string<64> notice_type;
  string<256> summary;
};
```

---

## C. State Reference

**Lifecycle:**
BOOTING, INITIALIZING, READY, ACTIVE, DEGRADED, SAFE_MODE, SERVICE_MODE, FAULT_LOCKED

**Connection:**
DISCONNECTED, DETECTING, CONNECTED_SUPPORTED, FAULTED

---

## D. Governance Flow

```
Proposal → Review → Draft → Validation → Matrix Update → Approval → Activation → Audit
```

---

## E. Release Notes (v1.0.0)

Initial release defining:

* Command, telemetry, fault, lifecycle types
* Service and diagnostic contracts
* Bridge staging types
* Vendor extension framework

All types are **v1.0.0 baseline** and require validation before production use.

---
