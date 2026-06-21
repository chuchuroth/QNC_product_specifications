
## Issue Type: Epic

**Summary:** Update architecture and component freeze for QNC

**Description:**
Central epic for all decisions regarding target architecture, product variants, main components, open specifications, and Jira backlog cleanup. This epic replaces the previous implicit assumption “Domestic main CPU + netX 90” with a formal freeze process including documented gates.

**Acceptance Criteria:**

* Architecture decision is approved
* Variant matrix is documented
* All blocker-relevant open items are assigned to an owner and deadline

**Priority:** Highest

**Dependencies / Links:** Supersedes parts of P0-02, P0-04; blocks P1-H01, P1-H02, P1-S01

---

## Issue Type: Bug

**Summary:** Resolve architectural conflict between BOM, PCB guideline, and existing task list

**Description:**
BOM, PCB document, and current task list are inconsistent regarding main CPU/SoM, industrial Ethernet strategy, layer count, and protocol scope. This inconsistency is a direct freeze blocker and may otherwise lead to incorrect schematic scope, unusable footprints, and PCB respins.

**Acceptance Criteria:**
Approved written decision on:
a) Main CPU/SoM
b) netX-90 usage in Phase 1/2
c) 6-layer vs. 8-layer
d) Phase-1 and Phase-2 protocol scope
Jira task list updated accordingly

**Priority:** Highest

---

## Issue Type: Story

**Summary:** Select primary main CPU SoM and fallback path

**Description:**
Compare and select the primary compute module, including Qualcomm SoM candidates from PCB guidelines and CompuLab/Variscite options from the BOM. Evaluate lifecycle, Linux BSP maturity, available interfaces (SPI/UART/I2C/GPIO/Ethernet), temperature range, long-term supply, and carrier documentation.

**Acceptance Criteria:**

* Primary SoM and one approved fallback defined
* Decision matrix documented
* Lifecycle/supply risks assessed
* Required carrier interfaces confirmed

**Priority:** Highest

---

## Issue Type: Story

**Summary:** Define netX-90 strategy, licensing scope, and SPI host connection

**Description:**
Decide whether the Hilscher netX 90 is populated in Phase 1, marked as DNP, or activated in Phase 2. Additionally define host interface, IRQ/reset/boot straps, licensing model, PHY selection, and required industrial Ethernet protocols.

**Acceptance Criteria:**

* Phase 1/2 decision documented
* Specific netX-90 variant and licensing assumptions defined
* SPI host signals and pin budget confirmed
* DNP strategy reflected in BOM

**Priority:** High

---

## Issue Type: Task

**Summary:** Freeze variant matrix, port count, and protocol roadmap

**Description:**
Define product variants with exact port counts for RS-485, IO-Link, DIO, CAN, and Ethernet, and map Phase 1 and Phase 2 protocols. Ensure hardware, software, and procurement align on the same variant matrix.

**Acceptance Criteria:**

* Variant matrix approved (including DNP options and protocols)
* Phase scopes reflected in Jira
* Outdated assumptions removed

**Priority:** Highest

---

## Issue Type: Task

**Summary:** Close open system requirements (power, isolation, DIO, service concept)

**Description:**
PCB guidelines list several freeze blockers: power specifications, brownout/transients, isolation/HiPot targets, DIO sourcing/sinking, service interface, Ethernet topology, and regulatory goals. These must be finalized in a formal requirement addendum.

**Acceptance Criteria:**

* Signed requirement addendum available
* 24 V min/max/transients defined
* Isolation and creepage targets defined
* DIO and service concepts clearly specified

**Priority:** Highest

---

## Issue Type: Task

**Summary:** Finalize connector, enclosure, and IP rating definition

**Description:**
Ethernet, IO-Link, RS-485, CAN, and DIO connectors are not finalized. Decide between RJ45 vs. M12, terminal vs. circular, shielding, tool clearance, and enclosure/IP requirements before mechanical freeze.

**Acceptance Criteria:**

* Connector family defined per interface
* Pinout, shielding, and mechanical constraints documented
* PCB edge placement impact reviewed

**Priority:** High

---

## Issue Type: Bug

**Summary:** Update or close outdated Jira tasks (legacy CPU, dev boards, protocols)

**Description:**
Several tasks no longer reflect current hardware/protocol strategy (e.g., “Domestic CPU”, old dev boards, EtherCAT-first approach). These must be replaced, renamed, or closed to avoid duplication.

**Acceptance Criteria:**

* All legacy tickets marked “Closed/Obsolete” or updated
* No outdated architecture tasks remain active

**Priority:** High

---

## Issue Type: Story

**Summary:** Perform BOM gap analysis and complete missing components

**Description:**
The BOM covers the core system but lacks several production-relevant items: connectors, magnetics, optional netX-90/PHY, 3.3V regulators, eFuse/high-side protection, EEPROM, CMC options, and protection parts.

**Acceptance Criteria:**

* Delta list between current and required BOM created
* Each missing item categorized (Add / Optional / DNP / Not needed)
* Missing MPNs and owners documented

**Priority:** Highest

---

## Issue Type: Task

**Summary:** Secure AVL, lifecycle, and second-source strategy

**Description:**
Critical parts (SoM, MCU, TPM, PHYs, power components, etc.) must be evaluated for lifecycle, lead time, and second sourcing.

**Acceptance Criteria:**

* Lifecycle and sourcing risk documented for each A-part
* Authorized distributors defined
* Second-source decisions documented

**Priority:** High

---

## Issue Type: Task

**Summary:** Verify symbols, footprints, and land patterns

**Description:**
Verify all library elements and critical packages (QFN, SOIC, isolated modules, thermal pads).

**Acceptance Criteria:**

* Approved symbols/footprints for all BOM items
* 3D models and keep-outs verified
* Pin orientation, thermal vias, and rules documented

**Priority:** High

---

# Next Epic: PCB Design and Electrical Implementation

## Summary: Update PCB design and implementation

**Description:**
Covers schematic, stack-up, routing, protection, PDN, thermal, testability, and interface implementation.

**Acceptance Criteria:**

* Schematic and PCB ready for freeze
* Layout rules documented
* Reviews completed

---

## (Selected key items from this section)

### Schematic restructuring

Split into domains: power, CPU, supervisor MCU, Ethernet, field interfaces, security, test.

### Stack-up definition

Resolve 6 vs. 8 layers, define impedance (50Ω, 100Ω, etc.), confirm with manufacturer.

### Power design

Implement 24V input protection, regulators, isolated rails, monitoring, and current sensing.

### Compute & security domain

Integrate SoM, MCU, TPM, RTC, EEPROM, debug, watchdog, and recovery.

### Field interfaces

Implement IO-Link, RS-485, CAN, DIO with protection and isolation strategy.

### Ethernet

Implement CPU Ethernet and optional netX-90 section (DNP-ready).

### Layout rules

Grounding, shielding, creepage, ESD protection, connector protection.

### Testability

Define test points, ICT/FCT access, debug interfaces.

### Thermal

Define thermal zones, power dissipation, and heat management.

---

# Next Epic: Design Verification & Compliance

Includes:

* EVT/DVT/PVT testing
* Power/reset validation
* Interface validation
* Security testing (TPM, signed config, rollback)
* Thermal validation (50°C worst case)
* EMC pre-compliance

---

# Final Epic: Manufacturing & Release

Includes:

* DFM/DFA documentation
* Prototype build & bring-up
* Security BOM handling
* Variant/DNP manufacturing rules


