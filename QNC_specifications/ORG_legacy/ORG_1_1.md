# QNC Operations & Recovery Guide

**Document Number:** QNC-ORG-001  
**Version:** 1.1 (v1 baseline)  
**Status:** Operational companion to **QNC-PRD-001 v3.0**  
**Classification:** Internal / Project Controlled  

---

## Document Change Log (Draft 1.0 → v1.1)

| Area | Change |
|------|--------|
| Tiers | Operations text aligned to **Core / Industrial Extensions / DDS Pack** |
| Discovery | **No** universal discovery commissioning steps; per-protocol matrix referenced |
| DDS | **Discovery Server** guidance → **future appendix**; v1 SIMPLE only when DDS Pack used |
| APIs | Split **robot-facing** vs **northbound** commissioning checks |
| Security ops | Added **SEC-OPS** checklist (certs, tokens, SBOM, CVE) |
| Capacity | Linked to PRD §7 default limits |

---

## 0. Purpose & Scope

Procedures for **deployment**, **YAML artifact handling**, **monitoring**, **fault/recovery**, **restart**, **Safe Mode**, and **DDS Pack** (optional). Does **not** define OpenAPI paths (use generated runbooks).

---

## 0.1 Operating Principles

1. **No partial configuration activation**  
2. **YAML** source artifacts are authoritative; verify hashes  
3. **Fault domain isolation** in diagnosis (southbound vs `dds_pack` vs northbound)  
4. **Safe Mode ≤ 1 s** for critical classes (per PRD)  
5. **Automatic rollback** on failed activation  

---

## 1. Configuration Procedures

### 1.1 Artifact Separation (Mandatory)

| Artifact | Format | Owner | Notes |
|----------|--------|-------|-------|
| system-config | YAML | Platform / Ops | Network, TLS, enabled packages, API tiers |
| device-profile | YAML | Device Integration | No DDS fields |
| dds-pack-bundle | IDL + yaml | Robotics | **Not** loaded if package absent |
| audit-policy | YAML | Security | Retention, signing |

### 1.2 Commissioning Steps (Core)

1. **Verify SKU**: Core vs extension packages installed.  
2. **Hardware/network**: Power, Ethernet/serial, IO-Link master ports, DIO maps.  
3. **Load system YAML**: atomic apply.  
4. **Load device YAML profiles**: per device; verify `semantic_mapping`.  
5. **Validate**: schema, signature, compatibility (`robot` + `northbound` API schema ids).  
6. **Activate**: single generation id; rollback on failure.  
7. **Lifecycle smoke**: BOOTING→ACTIVE path.  
8. **Functional test**: command/response, telemetry, fault inject to Safe Mode, recovery.

### 1.3 DDS Pack Add-On Steps (Optional)

9. Load **dds-pack-bundle** **separately** from profiles.  
10. Confirm **SIMPLE** discovery feasibility (multicast policy).  
11. Topic/QoS matrix check vs **QNC-IDL_v2.0** registry revision.  
12. Verify **dds_pack** faults do not alter southbound sessions (isolation test).

**Do not** plan Discovery Server for v1 baseline.

---

## 1.4 SEC-OPS Checklist (Field)

- [ ] TLS server chain installed + **expiry calendar**  
- [ ] API token issuance/rotation owners documented  
- [ ] Private key rotation drill **≤ annual** minimum  
- [ ] SBOM archived per release  
- [ ] CVE watchlist subscription for FastDDS + base OS  

---

## 2. DDS Pack Operations (Abbreviated)

Domain ID strategy from v1.0 draft **MAY** be reused as **recommendation** — not duplicated here; maintain in platform playbook.

**v1:** If multicast blocked → **STOP** or architecture exception (Discovery Server **not** v1 default path).

---

## 3. Deployment Modes (Simplified)

| Mode | Contents |
|------|----------|
| **Core Standalone** | Core only, single cell |
| **Core + Industrial** | Adds Modbus TCP / CANopen / OPC UA / MQTT as licensed |
| **Core + DDS Pack** | Adds DDS participant |
| **Full Stack** | All licensed packages |

Remove **Advanced Multi-Network DDS** from v1 go-live checklist.

---

## 4. Fault Recovery

### 4.1 Domains

Operators **MUST** classify: `southbound_*`, `dds_pack`, `configuration`, `northbound`, `robot_api`, `security`.

### 4.2 DDS-Specific

Loss of DDS discovery: treat as **`dds_pack`** fault; **do not** power-cycle fieldbus hardware as first action.

---

## 4.3 Restart Ladder

Service restart → adapter restart → DDS participant restart (if used) → controlled system restart → rollback generation.

---

## 5. Safe Mode

Same behavior: actuation lock; diagnostics open; DDS subscribe-only optional.

---

## 6. Monitoring

Minimum alerts: Safe Mode, critical faults, rollback events, signature failures, **dds_pack** discovery/QoS faults, API auth failures.

---

## 7. Capacity Operations

Monitor device count, WS fan-out, log disk, CPU — compare to PRD §7; escalate before limits.

---

## 8. Limits

Runbook SHALL be generated per deployment with **actual URLs**, **CLI**, and **topic names**.

---

## 9. Executive Summary

Operate QNC v1 with **strict artifact separation**, **YAML authority**, **Core vs DDS Pack isolation**, **robot vs northbound API awareness**, and **protocol-specific binding** instead of assumed discovery.
