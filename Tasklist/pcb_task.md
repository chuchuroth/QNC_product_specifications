# Translation of Jira Task List: QNC Project

The existing task list should **not only be expanded but also technically corrected in several places**: The current Jira backlog is still heavily based on an older architecture with "netX 90 + Domestic main CPU," 6-layer PCB, and an EtherCAT/PROFINET-first roadmap. However, the PCB guidelines additionally contain a newer architecture update with **Qualcomm-SoM + Supervisor MCU + optional netX 90**, **8-layer carrier**, and a **Phase 1 focus on Modbus RTU, IO-Link, DIO, and CAN/CANopen**, while the BOM, in turn, lists **CompuLab/Variscite** as the primary SoM path and Toradex only as a risk note. Therefore, the Jira backlog should first clear up architecture conflicts, open specifications, and BOM gaps before proceeding with layout, prototyping, and verification. [PCB.md](https://www.genspark.ai/api/files/s/Nkv8GrFm) [BOM.xlsx](https://www.genspark.ai/api/files/s/qhO8CHVp) [QNC_Project_TaskList_v3.xlsx](https://www.genspark.ai/api/files/s/ziri37Ov)

**Recommended Jira Issues (import-/copy-ready):**

---

* **Issue Type:** Epic  
* **Summary:** Update Architecture and Component Freeze for QNC  
* **Description:** Central Epic for all decisions regarding target architecture, product variants, main components, open specifications, and Jira backlog cleanup. This Epic replaces the previous implicit assumption of "Domestic main CPU + netX 90" with a formal freeze process featuring documented gates.  
* **Acceptance Criteria:** Architecture decision is released; Variant Matrix is documented; all blocker-relevant open points are assigned to an owner and deadline.  
* **Priority:** Highest  
* **Dependencies / Links:** Supersedes parts of P0-02, P0-04; blocks P1-H01, P1-H02, P1-S01.  
* **Epic (if applicable):** —

---

* **Issue Type:** Bug  
* **Summary:** Resolve Architecture Conflict between BOM, PCB Guideline, and Existing Task List  
* **Description:** BOM, PCB document, and existing task list are inconsistent regarding main CPU/SoM, Industrial Ethernet strategy, layer count, and protocol delivery scope. This inconsistency is a direct freeze blocker and can otherwise lead to incorrect schematic scope, unusable footprints, and PCB respins.  
* **Acceptance Criteria:** Written released decision on a) Main CPU/SoM, b) netX-90 usage in Phase 1/2, c) 6-layer vs. 8-layer, d) Phase 1 and Phase 2 protocol scope; Jira task list updated accordingly.  
* **Priority:** Highest  
* **Dependencies / Links:** Links to P0-02, P0-04, P1-H01, P1-H02; blocks all layout and sourcing topics.  
* **Epic (if applicable):** Update Architecture and Component Freeze for QNC

---

* **Issue Type:** Story  
* **Summary:** Select Primary Main CPU SoM and Fallback Path  
* **Description:** Comparison and selection of the primary compute module including Qualcomm SoM candidates from the PCB guideline and CompuLab/Variscite paths from the BOM. Lifecycle, Linux BSP maturity, available SPI/UART/I2C/GPIO/Ethernet pins, temperature range, long-term supply, and carrier documentation must be evaluated.  
* **Acceptance Criteria:** Primary SoM and 1 released fallback are named; decision matrix is filed; lifecycle/supply risk is assessed; required carrier interfaces are confirmed.  
* **Priority:** Highest  
* **Dependencies / Links:** Depends on "Resolve Architecture Conflict"; blocks P1-H01, P1-S01, footprint and stack-up tasks.  
* **Epic (if applicable):** Update Architecture and Component Freeze for QNC

---

* **Issue Type:** Story  
* **Summary:** Define netX-90 Strategy, License Scope, and SPI Host Connection  
* **Description:** Decide whether the Hilscher netX 90 will be populated in Phase 1, provided as DNP (Do Not Populate), or only activated in Phase 2. Additionally, the host interface, IRQ/Reset/Boot-straps, license model, PHY selection, and required Industrial Ethernet protocols must be bindingly defined.  
* **Acceptance Criteria:** Phase 1/Phase 2 decision documented; exact netX-90 variant and license assumptions named; SPI host signals and pin budget confirmed; DNP strategy noted in BOM.  
* **Priority:** High  
* **Dependencies / Links:** Dependent on Main CPU decision; blocks Ethernet/Co-Processor schematic and protocol roadmap; replaces parts of P1-S02, P1-S03, P1-S04.  
* **Epic (if applicable):** Update Architecture and Component Freeze for QNC

---

* **Issue Type:** Task  
* **Summary:** Freeze Variant Matrix, Port Count, and Protocol Roadmap  
* **Description:** Define product variants with exact port counts for RS-485, IO-Link, DIO, CAN, and Ethernet, and bindingly map Phase 1 and Phase 2 protocols. The goal is for hardware, software, and purchasing to work from the same Variant Matrix.  
* **Acceptance Criteria:** Variant Matrix with port count, DNP options, and protocol releases is approved; Phase 1 and Phase 2 scope mapped in Jira; outdated assumptions removed.  
* **Priority:** Highest  
* **Dependencies / Links:** Links to P0-04, P2-04, P4-01; blocks schematic, BOM freeze, and test planning.  
* **Epic (if applicable):** Update Architecture and Component Freeze for QNC

---

* **Issue Type:** Task  
* **Summary:** Close Open System Requirements for Power Supply, Isolation, DIO Topology, and Service Concept  
* **Description:** The PCB guidelines list several freeze blockers: exact power supply specification, brownout/transients, isolation target/HiPot, DIO sourcing/sinking, service interface, Ethernet topology, and regulatory objectives. These points must be finalized as a formal Requirement Addendum.  
* **Acceptance Criteria:** Signed-off Requirement Addendum exists; Min/Max/Transients for 24 V are defined; isolation and creepage targets are set; DIO concept and service concept are clearly described.  
* **Priority:** Highest  
* **Dependencies / Links:** Blocks power, isolation, DIO, connector, and compliance topics.  
* **Epic (if applicable):** Update Architecture and Component Freeze for QNC

---

* **Issue Type:** Task  
* **Summary:** Finalize Connector, Enclosure, and IP Target Definition  
* **Description:** Ethernet, IO-Link, RS-485, CAN, and DIO connectors are not yet final according to BOM/Guideline. Decision between RJ45 and M12 as well as terminal/circular concept, shielding connection, tool clearance, and enclosure/IP requirements must be completed before mechanical freeze.  
* **Acceptance Criteria:** A connector family is released for each external interface; pinout, shielding strategy, and mechanical constraints are documented; impact on PCB edge placement checked.  
* **Priority:** High  
* **Dependencies / Links:** Blocks PCB layout, BOM, DFM/DFA, and EMC concept; linked to P1-T03.  
* **Epic (if applicable):** Update Architecture and Component Freeze for QNC

---

* **Issue Type:** Bug  
* **Summary:** Update or Close Outdated Jira Tasks Regarding Legacy CPU, Dev Boards, and Protocol Scope  
* **Description:** Several existing tasks no longer reflect the current hardware and protocol strategy, e.g., "Domestic main CPU," dev boards GD32H75E/AX58101, EtherCAT/PROFINET-first, and later protocol plans. These topics must either be replaced, renamed, or explicitly closed to avoid duplicate work.  
* **Acceptance Criteria:** Status "Closed/Obsolete" or updated description is stored for all affected old tickets; no legacy architecture tasks remain as an active planning basis.  
* **Priority:** High  
* **Dependencies / Links:** Check/update: P0-02, P0-03, P0-04, P1-S02, P1-S03, P1-S04, P1-S06, P2-04, P4-01.  
* **Epic (if applicable):** Update Architecture and Component Freeze for QNC

---

* **Issue Type:** Story  
* **Summary:** Perform BOM Gap Analysis and Add Missing Series Components  
* **Description:** The BOM covers the active core system but leaves several elements necessary for series production open or inconsistent: connectors, magnetics, potentially netX-90/PHY set for Phase 2, 3.3V regulators, eFuse/protected high-side distribution, boot/config EEPROM, CMC options, and potentially additional protection parts.  
* **Acceptance Criteria:** Delta list between current BOM and required series BOM is created; every missing item has status "Add / Optional / DNP / Not needed"; open MPNs and owners are documented.  
* **Priority:** Highest  
* **Dependencies / Links:** Dependent on Architecture, Connector, and Variant Matrix freeze; blocks purchasing and layout release.  
* **Epic (if applicable):** Update Architecture and Component Freeze for QNC

---

* **Issue Type:** Task  
* **Summary:** Secure AVL, Lifecycle, and Second-Source Strategy for A-Parts  
* **Description:** Critical single-source or long-lead parts such as SoM, STM32G474, TPM, MAX14819, ISO1410, PHYs, DO drivers, and power front-end must be transferred into a resilient AVL (Approved Vendor List) with lifecycle, lead-time, and second-source assessment. Toradex remains only as a documented risk note, not as a primary path.  
* **Acceptance Criteria:** For each A-part, lifecycle status, procurement risk, authorized distributor, and second-source decision exist; safety components have restricted source release; buffer/LTB decisions are documented.  
* **Priority:** High  
* **Dependencies / Links:** Follows primary SoM decision; linked with P3-01.  
* **Epic (if applicable):** Update Architecture and Component Freeze for QNC

---

* **Issue Type:** Task  
* **Summary:** Complete Symbol, Footprint, and Land Pattern Verification for all BOM Components  
* **Description:** Verification of all library elements for U1/U2/U3/U10/U11/U20/U21/U30/U31/U40-U43/U50-U53/U60/U70/U71/U72/U73/U80/U90/U91/D100-D103/R200-R203 as well as central passive families. Special attention to QFN/UQFN, Wide-SOIC, isolated modules, current shunts, and thermally relevant pads.  
* **Acceptance Criteria:** Released symbols and footprints exist for all BOM lines; 3D models/keep-outs checked; Pin 1, exposed pads, thermal vias, and courtyard rules are documented.  
* **Priority:** High  
* **Dependencies / Links:** Requires architecture and BOM gap analysis; blocks schematic and layout.  
* **Epic (if applicable):** Update Architecture and Component Freeze for QNC

---

* **Issue Type:** Epic  
* **Summary:** Update PCB Design and Electrical Implementation  
* **Description:** Epic for schematic, stack-up, placement, routing, protection concepts, PDN, thermal management, testability, and interface implementation of the updated QNC hardware.  
* **Acceptance Criteria:** Schematic and PCB are ready for freeze; all layout rules are documented; review and sign-off checklists are completed.  
* **Priority:** Highest  
* **Dependencies / Links:** Starts after architecture freeze.  
* **Epic (if applicable):** —

---

* **Issue Type:** Story  
* **Summary:** Restructure Schematic Hierarchy According to Functional Domains  
* **Description:** Divide schematic into clearly separated functional blocks: Power Entry, Main CPU SoM, Supervisor MCU, Ethernet/Co-Processor, RS-485/RS-232, IO-Link, CAN, DIO, Security/NVM/Service, and Test/Production. Goal is fault containment, better reviewability, and consistent net names.  
* **Acceptance Criteria:** Sheet hierarchy is released; all external interfaces and power domains are clearly assigned to a sheet group; cross-probing and naming convention are documented.  
* **Priority:** High  
* **Dependencies / Links:** Requires Architecture and Variant Matrix freeze; updates P1-H01.  
* **Epic (if applicable):** Update PCB Design and Electrical Implementation

---

* **Issue Type:** Story  
* **Summary:** Define PCB Stack-up and Controlled Impedance Plan  
* **Description:** Decision and manufacturing release for the final stack-up. Due to the architecture update, the 6-layer vs. 8-layer conflict specifically needs to be resolved; additionally, 50 Ω single-ended, 100 Ω diff, potentially USB 90 Ω diff, and manufacturing coupons must be defined.  
* **Acceptance Criteria:** Final stack-up is approved by the board house; impedance calculations and tolerances are documented; stack-up is stored in the PCB tool.  
* **Priority:** High  
* **Dependencies / Links:** Blocks P1-H02, PHY/Ethernet routing, and power layout; depends on architecture conflict.  
* **Epic (if applicable):** Update PCB Design and Electrical Implementation

---

* **Issue Type:** Story  
* **Summary:** Implement Power Entry, PDN, and Protected Field Supply  
* **Description:** Implementation of 24V input protection, hot-swap/inrush, TVS, EMI filter, 5V_MAIN, 3V3_LOGIC, isolated rails, and protected 24V_FIELD banks. Including current sense, supervisor monitoring, and star distribution of the supply.  
* **Acceptance Criteria:** Circuit and layout for LM5069, LM76005, secondary regulators, NXJ2S0505, field protection, and measurement paths are review-free; load budget and 30% electrical reserve are demonstrated; return current path checked.  
* **Priority:** Highest  
* **Dependencies / Links:** Depends on Requirement Addendum and BOM gap analysis; blocks bring-up and thermal/EMC tests.  
* **Epic (if applicable):** Update PCB Design and Electrical Implementation

---

* **Issue Type:** Story  
* **Summary:** Implement Compute, Supervisor, Security, and Service Domain  
* **Description:** Integration of Main SoM, STM32G474, TPM, RTC, optional EEPROM, recovery pins, console/debug, watchdog, and reset chain. The circuit must support boot, signed config, recovery, fault logging, and secure diagnostic access.  
* **Acceptance Criteria:** All control/service signals are defined; reset/watchdog/boot-mode sequences are documented; TPM/RTC/NVM connection is electrically verified; debug and recovery access is reachable.  
* **Priority:** High  
* **Dependencies / Links:** Dependent on SoM selection; blocks P1-S01 as well as security/recovery tests.  
* **Epic (if applicable):** Update PCB Design and Electrical Implementation

---

* **Issue Type:** Story  
* **Summary:** Implement Field Interfaces for IO-Link, RS-485, CAN, and DIO  
* **Description:** Implementation of 4x IO-Link ports, 2x isolated RS-485 channels, optional/isolated CAN interface, and 8 DI + 8 DO with a clear protection and isolation strategy. Symmetry, port protection, fuse/eFuse concept, and thermal decoupling are to be explicitly considered.  
* **Acceptance Criteria:** Full schematic/layout implementation including protection chain exists for each field interface; termination/biasing/port symmetry checked; DIO topology corresponds to released specification.  
* **Priority:** High  
* **Dependencies / Links:** Depends on Variant Matrix, DIO decision, and Connector freeze; updates P1-H01.  
* **Epic (if applicable):** Update PCB Design and Electrical Implementation

---

* **Issue Type:** Story  
* **Summary:** Integrate Northbound Ethernet and Optional Industrial Ethernet Area DNP-ready  
* **Description:** Realization of the CPU-proximate Ethernet path as well as the optional netX-90 and PHY area for Phase 2. MDI/diff-pair rules, magnetics/shielding, DNP variants, and symmetrical port geometry must be bindingly implemented.  
* **Acceptance Criteria:** CPU Ethernet path is released; optional netX-90 block incl. PHY/connector footprints is layout-ready; DNP marking and population variants are included in the design.  
* **Priority:** High  
* **Dependencies / Links:** Dependent on SoM and netX-90 decision; replaces parts of P1-S03/P1-S04 as hardware preliminary work.  
* **Epic (if applicable):** Update PCB Design and Electrical Implementation

---

* **Issue Type:** Task  
* **Summary:** Embed Grounding, Shielding, Creepage/Clearance, and Connector Protection Rules in Layout  
* **Description:** Implementation of rules for DGND/FGND/CHASSIS, connector-side ESD/TVS/CMC, visible isolation barriers, shield-to-chassis coupling, and sufficient creepage/clearance in isolated areas.  
* **Acceptance Criteria:** Review checklist for protection and isolation zones passed; all external lines have defined protection chain; CHASSIS/DGND coupling documented; clearance rules stored in PCB tool.  
* **Priority:** High  
* **Dependencies / Links:** Blocks EMC, safety margin, and mechanical integration.  
* **Epic (if applicable):** Update PCB Design and Electrical Implementation

---

* **Issue Type:** Task  
* **Summary:** Fully Provide Test Points, Boundary/Bed-of-Nails Access, and Debug Accesses  
* **Description:** Provide mandatory test points for rails, reset/boot, fault signals, RS-485, CAN, IO-Link, DIO, UART, SWD/JTAG, and shield-to-chassis paths. Additionally prepare manufacturing test access, loopback possibilities, and fault localization paths.  
* **Acceptance Criteria:** Test point matrix is complete; ICT/FCT access coordinated with manufacturing; no critical bring-up signal is difficult to reach; debug accesses usable without full disassembly.  
* **Priority:** High  
* **Dependencies / Links:** Dependent on schematic hierarchy; blocks EVT/PVT and manufacturing release.  
* **Epic (if applicable):** Update PCB Design and Electrical Implementation

---

* **Issue Type:** Task  
* **Summary:** Transfer Thermal Zone Concept and Power Dissipation Budget into Layout  
* **Description:** Optimize placement and heat dissipation for SoM, netX 90, PHYs, buck regulators, IO-Link ICs, and DO drivers. Thermal vias, copper planes, distance to connectors, and possible heat spreader to mechanics must be considered early.  
* **Acceptance Criteria:** Power dissipation budget per zone is documented; hotspots and critical components identified; thermal via and heat spreader concept implemented in layout.  
* **Priority:** Medium  
* **Dependencies / Links:** Depends on placement and enclosure decision; blocks 50°C worst-case verification.  
* **Epic (if applicable):** Update PCB Design and Electrical Implementation

---

* **Issue Type:** Epic  
* **Summary:** Secure Design Verification, Compliance, and Product Maturity  
* **Description:** Epic for EVT/DVT/PVT, security and recovery tests, protocol validation, thermal/EMC testing, performance criteria, and release gates.  
* **Acceptance Criteria:** Full verification plan approved; all gate tests have owner, test environment, and completion criteria.  
* **Priority:** High  
* **Dependencies / Links:** Starts after first functional prototype.  
* **Epic (if applicable):** —

---

* **Issue Type:** Story  
* **Summary:** Create EVT/DVT/PVT Test Plan with Measurable QNC Acceptance Criteria  
* **Description:** Derive a structured test plan from PCB guidelines, incl. power-up, brownout, inrush, link recovery, safe mode, rollback, security, burn-in, and pre-compliance. QNC-specific limits such as safe mode ≤1 s, restart ≤30 s, and REST-to-device latency ≤50 ms must be formally adopted.  
* **Acceptance Criteria:** Released test plan with EVT/DVT/PVT structure exists; each test has measurement method and limit; go/no-go criteria defined.  
* **Priority:** High  
* **Dependencies / Links:** Dependent on architecture and hardware freeze; updates P1-H04, P1-T02.  
* **Epic (if applicable):** Secure Design Verification, Compliance, and Product Maturity

---

* **Issue Type:** Task  
* **Summary:** Validate Power, Reset, Brownout, Fault, and Recovery Chain  
* **Description:** Validation of power sequencing, watchdog, supervisor MCU, brownout behavior, inrush faults, safe-mode transition, and restart times. Goal is to handle errors in isolation and enable recovery without a special jig.  
* **Acceptance Criteria:** Measurement logs for power-up/brownout/reset exist; safe mode ≤1 s demonstrated; restart ≤30 s demonstrated; fault log/recovery path works reproducibly.  
* **Priority:** Highest  
* **Dependencies / Links:** Depends on power/supervisor implementation; supplements P1-H04.  
* **Epic (if applicable):** Secure Design Verification, Compliance, and Product Maturity

---

* **Issue Type:** Task  
* **Summary:** Validate Baseline Interfaces and Communication Functions  
* **Description:** Electrical and functional validation of RS-485/Modbus RTU, IO-Link, DIO, and optionally CAN/CANopen in Phase 1. If netX 90 is populated, additionally check link-up, port symmetry, and basic function of the Industrial Ethernet area.  
* **Acceptance Criteria:** Proof of function exists for each released Phase 1 interface; long-term test for Modbus RTU defined/completed; port diagnostics working.  
* **Priority:** High  
* **Dependencies / Links:** Depends on field interface implementation; updates P1-T01 and parts of P1-S05.  
* **Epic (if applicable):** Secure Design Verification, Compliance, and Product Maturity

---

* **Issue Type:** Task  
* **Summary:** Verify Security, TPM, Signed Config, and Rollback Functions  
* **Description:** Testing of TPM connection, board identity, secure boot path, signed configuration, recovery, and rollback behavior. Security BOM and procurement restrictions for TPM-related parts are to be included in the test scope.  
* **Acceptance Criteria:** TPM correctly recognized; signature/rollback paths testable; recovery after misconfiguration works; security test report released.  
* **Priority:** High  
* **Dependencies / Links:** Depends on compute/security domain; new compared to existing task list.  
* **Epic (if applicable):** Secure Design Verification, Compliance, and Product Maturity

---

* **Issue Type:** Task  
* **Summary:** Secure Thermal Performance at 50°C Ambient and Worst-Case Load  
* **Description:** Measurement under 24V, full field load, Ethernet/protocol traffic, and maximum compute load. Validation of SoM, power stages, IO-Link, and DO drivers against defined hotspot and junction limits.  
* **Acceptance Criteria:** Test setup and load profile documented; temperature limits maintained or remedial measures recorded; heat spreader/layout adjustments derived.  
* **Priority:** Medium  
* **Dependencies / Links:** Requires EVT prototype and thermal zone concept.  
* **Epic (if applicable):** Secure Design Verification, Compliance, and Product Maturity

---

* **Issue Type:** Task  
* **Summary:** Perform EMC Pre-Compliance and Immunity for 24V Industrial Environment  
* **Description:** Prepare and perform pre-compliance plan for EN 55032/CISPR 32 Class A, EN 61000-6-2, as well as ESD/EFT/Surge/Conducted RF. Focus on Ethernet emissions, 24V input, IO-Link/DIO burst robustness, and shielding/CMC tuning.  
* **Acceptance Criteria:** Test matrix and setup defined; pre-compliance performed; documented findings with tuning measures exist.  
* **Priority:** High  
* **Dependencies / Links:** Dependent on protection, grounding, and enclosure concept; updates P2-05 and P3-03.  
* **Epic (if applicable):** Secure Design Verification, Compliance, and Product Maturity

---

* **Issue Type:** Story  
* **Summary:** Prepare Manufacturing Test Strategy, ICT/FCT, and Traceability  
* **Description:** Define manufacturing test points, FCT scripts, series calibration, board ID/BOM variant storage, and traceability concept. Goal is a reproducible transition from EVT/DVT to PVT.  
* **Acceptance Criteria:** ICT/FCT coverage documented; board ID/calibration/manufacturing data storage specified; PVT test flow released.  
* **Priority:** Medium  
* **Dependencies / Links:** Depends on test point and NVM concept; supplements P3-02.  
* **Epic (if applicable):** Secure Design Verification, Compliance, and Product Maturity

---

* **Issue Type:** Epic  
* **Summary:** Secure Manufacturing Preparation, Supply Chain, and Release Management  
* **Description:** Epic for DFM/DFA, prototype procurement, assembly readiness, AVL release, manufacturing documentation, and series preparation.  
* **Acceptance Criteria:** Released manufacturing documentation and procurable AVL exist; prototype and PVT builds are plannable.  
* **Priority:** High  
* **Dependencies / Links:** Starts with stable BOM and first PCB status.  
* **Epic (if applicable):** —

---

* **Issue Type:** Story  
* **Summary:** Create DFM/DFA and Fabrication Notes Package for PCB Release  
* **Description:** Finalize IPC class, min design rules, warpage control, solder mask reliefs, impedance coupons, fiducials, coating decision, and assembly accessibility. Special attention to QFN/UQFN, isolated modules, edge connectors, and service accessibility.  
* **Acceptance Criteria:** Manufacturing and assembly notes released; PCB manufacturer feedback incorporated; assembly constraints for critical packages documented.  
* **Priority:** High  
* **Dependencies / Links:** Requires final stack-up and connector selection; updates P1-H03.  
* **Epic (if applicable):** Secure Manufacturing Preparation, Supply Chain, and Release Management

---

* **Issue Type:** Task  
* **Summary:** Prepare Prototype Build, Bring-up Plan, and Critical Assembly Risks  
* **Description:** EVT build with quantity, assembly instructions, and bring-up sequence. Identification of critical assembly steps (e.g., LGA/SoM, fine-pitch connectors) and definition of the bring-up hierarchy (Power -> MCU -> SoM -> Interfaces).  
* **Acceptance Criteria:** Assembly package sent to EMS; bring-up checklist exists; critical risks identified and mitigation (e.g., X-ray for LGA) defined.  
* **Priority:** High  
* **Dependencies / Links:** Depends on DFM/DFA and AVL; updates P1-H03.  
* **Epic (if applicable):** Secure Manufacturing Preparation, Supply Chain, and Release Management

---

* **Issue Type:** Task  
* **Summary:** Define Security and Procurement Handling for TPM/Security BOM  
* **Description:** Security-relevant parts such as TPM and board identity memory require separate procurement and revision control. Authorized sources, handling rules, and change management must be formally established.  
* **Acceptance Criteria:** Security BOM versioned separately; authorized supply chain documented; ECN/PCN process for security parts defined.  
* **Priority:** Medium  
* **Dependencies / Links:** Linked with AVL strategy and security verification.  
* **Epic (if applicable):** Secure Manufacturing Preparation, Supply Chain, and Release Management

---

* **Issue Type:** Task  
* **Summary:** Map Phase 1/Phase 2 Population Variants and DNP Rules in BOM and Manufacturing  
* **Description:** The variant without activated Industrial Ethernet co-processor and the Phase 2 population with netX 90, PHYs, protection, and connectors must be mapped as clean manufacturing variants.  
* **Acceptance Criteria:** Variant BOMs with DNP rules exist; manufacturing can clearly distinguish Phase 1 and Phase 2; cost and risk impacts assessed.  
* **Priority:** Medium  
* **Dependencies / Links:** Depends on netX-90 strategy and BOM gap analysis; replaces vague later protocol tasks.  
* **Epic (if applicable):** Secure Manufacturing Preparation, Supply Chain, and Release Management

---

### Recommended Direct Changes to Existing Jira Tickets
I would **not continue the following existing tasks unchanged**:

- **P0-02** → **replace/rewrite** to Architecture Freeze incl. SoM selection, netX-90 strategy, and Variant Matrix  
- **P0-03** → **close or replace**, as the mentioned dev boards no longer fit cleanly with the new architecture  
- **P0-04** → **rewrite** for Phase 1/Phase 2 protocol model  
- **P1-S02 / P1-S03 / P1-S04** → **move to Phase 2/Optional scope** or link to new netX-90/Industrial Ethernet strategy  
- **P2-04 / P4-01** → **re-stagger**, as the protocol roadmap no longer technically aligns with current documents  
- **P1-H01 / P1-H02 / P1-H03 / P1-H04** → **retain, but update to new architecture, stack-up decision, and BOM** [QNC_Project_TaskList_v3.xlsx](https://www.genspark.ai/api/files/s/ziri37Ov) [PCB.md](https://www.genspark.ai/api/files/s/Nkv8GrFm) [BOM.xlsx](https://www.genspark.ai/api/files/s/qhO8CHVp)

If you wish, I can generate a **CSV/Excel-importable Jira template** for you in the next step with columns such as `Issue Type, Summary, Description, Priority, Epic Link, Depends On`.
