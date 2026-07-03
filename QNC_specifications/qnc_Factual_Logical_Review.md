# QNC Project — Factual & Logical Review

---

## 1. Factual Accuracy

### 1.1 RS485 Wiring Table Mislabels Pins (README.md)

> "Pin 8 | GPIO14 (TXD) | Transmit | **RXD / RO** (Receiver Output)"
> "Pin 10 | GPIO15 (RXD) | Receive | **TXD / DI** (Driver Input)"

The labels in the "RS485 Module" column are inverted relative to standard RS485 transceiver naming conventions. On a typical MAX485/SN75176-type chip:
- **DI** (Driver Input) is connected to the MCU's **TXD** (the MCU drives the RS485 line via DI).
- **RO** (Receiver Output) is connected to the MCU's **RXD** (the RS485 receiver drives the MCU RXD via RO).

The table header for the RS485 Module column should read **DI** (not RXD/RO) for the TXD row, and **RO** (not TXD/DI) for the RXD row. The inline explanation partially corrects this but introduces a conflation of "RXD" (a UART pin name) with "RO" (an RS485 driver signal name), which are different things. This will confuse any hardware engineer wiring the circuit.

**Suggestion**: Separate the UART names from the RS485 transceiver signal names clearly in the table.

---

### 1.2 `error_code` Range Inconsistency (ICD, Section 6.3.2)

> "error_code | int32 | **-99 to 0** | 0=success, negative=error code"

Appendix B of the same ICD defines error codes that go far outside this stated range: `-100` (POWER_FAILURE), `-101` (MOUNT_FAILURE), `-102`, `-103`, `-104`, and `-200` through `-205`. The declared field constraint of `-99 to 0` is directly contradicted by the specification's own error table.

**Suggestion**: Update the field range to reflect the actual minimum value (e.g., `-205 to 0`), or document that the range is open-ended for extensibility.

---

### 1.3 ICD Version Header Not Updated (ICD, Appendix E)

The ICD cover page reads **Version: 1.0, Status: Official Release**, but Appendix E records a **version 1.1** (dated 2026-02-19) that added system handshaking, enhanced error handling, and the firmware update mechanism — all of which are substantive sections of the document. The cover page was never updated.

**Suggestion**: Increment the cover version to 1.1 to match the change history.

---

### 1.4 Disconnect During Motion Uses Wrong Error Code in ICD Body (ICD, Section 12.5 vs. PRD Section 5.8)

ICD Section 12.5 uses **-103** for `DEVICE_DISCONNECT_DURING_MOTION`:
> `error_response.error_code = -103;  // DEVICE_DISCONNECT_DURING_MOTION`

But Appendix B assigns **-103** to `DEVICE_DISCONNECT` and **-104** to `DEVICE_DISCONNECT_DURING_MOTION`. The PRD (Section 5.8) also correctly uses `-104`. The code example in Section 12.5 is wrong.

**Suggestion**: Fix the example in ICD Section 12.5 to use `-104`.

---

### 1.5 "99% Less Code" Conflates Metrics (README.md — Summary Table)

> "Support 10 devices | 10 weeks | 5 hours | **99% less code**"

The column header is "Time Saved," but the value given is "99% less code" — a code-size metric, not a time metric. Immediately above this the text says "Time to add 10 devices: 5 hours." The table mixes two different measurements (time and lines of code) within the same column, making the comparison misleading.

---

### 1.6 Reference [2] Is a Local Filesystem Path (PRD, References Section)

> "[2]: N-QNCBoard-ConceptPlanningandRoadmap-130226-115752 — **(file:///home/ubuntu/upload/...)**"

A `file:///` path pointing to a local Ubuntu filesystem is not a valid document reference. It implies the PRD was machine-generated from an AI assistant's working session without the reference being properly resolved to a canonical document identifier. This path will be inaccessible to any reader.

**Suggestion**: Replace with a proper document number, version, and storage location (e.g., internal document server, repository path).

---

### 1.7 IO-Link Physical Layer Description Is Imprecise (ICD, Section 5.4 and elsewhere)

> "IO-Link: A standardized IO technology (IEC 61131-9) used for communicating with sensors and actuators, typically over **SPI/UART**."

IO-Link is a point-to-point protocol transmitted over a **single wire (C/Q line)** using UART framing. It does not use SPI. The hardware schematic (Section 8.2) correctly identifies the MAX22515 transceiver and the C/Q line, but the repeated characterization of IO-Link as "SPI/UART" in interface tables, hardware specs, and protocol descriptions is technically imprecise and may mislead integrators. IO-Link uses UART-like serial framing at one of three baud rates, not SPI (which is a 4-wire synchronous protocol).

---

## 2. Common Sense & Logical Consistency

### 2.1 "Pure Protocol Bridge" Claim Contradicts Hardware ID Device Table (ICD, Section 8.3 vs. REFACTORING_SUMMARY)

The refactoring document repeatedly asserts:
> "Zero device-specific semantics in active runtime flow"
> "Device semantics stay out of QNC runtime path"

Yet ICD Section 8.3 contains a resistor-to-device mapping table listing specific commercial products by name (DH-Robotics AG95, Robotiq 2F-85, Zimmer GEP5000IL, etc.) with their exact resistance values and protocols. This is device-specific knowledge baked into the QNC hardware ID detection path. QNC cannot be "purely" protocol-agnostic if it ships with a lookup table of known device models.

This is not necessarily a design flaw, but the documentation should stop claiming zero device semantics when a named-device mapping table is part of the specification. The claim overstates the abstraction.

---

### 2.2 EMERGENCY_STOP CommandType vs. "Safety Systems Out of Scope" (ICD, Sections 2.2 and 6.4)

ICD Section 2.2 explicitly states:
> "Safety systems and emergency stop protocols" are **Not covered** in this ICD.

Yet Section 6.4 defines `EMERGENCY_STOP` as a valid `CommandType` in the generic command interface. You cannot simultaneously exclude emergency stop protocols from scope and define an interface for triggering them. This is a direct internal contradiction.

**Concern**: Providing an E-stop command over a DDS pub/sub channel is architecturally inappropriate for a certified safety function. Industrial safety standards (ISO 10218-1/2, IEC 62061) require hardware-level E-stop circuits — software messages over a network cannot satisfy SIL or PL requirements. The document should explicitly disclaim any safety certification intent for this path.

---

### 2.3 Form Factor Contradiction: 5 cm × 3 cm vs. 0.5–1.0 kg Weight (PRD, NFR-024 and NFR-025)

> NFR-025: "The QNC module MUST be smaller than **5 cm x 3 cm**"
> NFR-024: "Target weight: **0.5 kg to 1.0 kg**"

A device measuring 5 × 3 cm with no specified depth constraint weighing 500–1000 g would have an implausibly high density (comparable to solid steel). Even at 3 cm depth (5 × 3 × 3 cm = 45 cm³), 0.5 kg implies ~11 g/cm³ — denser than iron. For reference, the i.MX 8M PLUS Toradex Verdin SoM alone is 55 × 35 mm. These two requirements as written are mutually inconsistent and appear to come from different source documents with no reconciliation.

**Suggestion**: The form factor requirement likely refers to a 2D footprint constraint (perhaps the flange mount area), not the full 3D volume. Clarify the constraint and verify the weight is achievable given the specified hardware (SoM, battery, OLED display, connectors).

---

### 2.4 Health Check Rate Creates a 5-Second Blind Spot (ICD, Section 11.3)

> "Health check: **0.2 Hz** (5000 ms period)"
> "Disconnection detection: **3 consecutive failed** health checks"

Combined, these mean QNC may take up to **15 seconds** (3 × 5 s) to confirm a device disconnection. This contradicts Section 12.5, which states detection should happen in **< 100 ms**. The < 100 ms target applies only when a command is actively in flight (command timeout fires). During idle periods between commands, disconnection would not be detected for up to 15 seconds. This gap is not documented.

**Suggestion**: Specify behavior clearly: command-timeout-based detection covers active operation; health-check-based detection covers idle periods. Consider increasing the health check rate in safety-relevant deployments.

---

### 2.5 Heartbeat Timeout of 5 Missed Beats Is Too Slow (PRD, Section 5.7)

> "5 consecutive missed beats trigger safe state" at 1 Hz → **5 seconds** to detect lost communication.

During those 5 seconds, the robot platform could be sending commands to QNC, receiving no responses, and making decisions without knowing QNC has lost contact. For a robotics system, this is an unreasonably long communication loss tolerance. The timeout window should be configurable at the low end (the document does say 0.5–10 Hz for heartbeat rate, but the "5 beats" trigger is fixed).

---

### 2.6 JSON Descriptor Location Is Contradictory (README.md vs. ICD and PRD)

README.md describes device descriptors as residing on the **robot platform**:
> "device_descriptors/ (JSON files - no code!) — robot platform code"

But ICD IF-012 specifies their location as:
> `/etc/qnc/devices/<device_name>.json` — *on the QNC device*

And PRD FR-012 says "The QNC system SHALL utilize JSON files as device descriptors." These are three contradictory answers to a fundamental architectural question: who owns and stores the device descriptor? This gap has real integration consequences.

---

### 2.7 Hot-Swap Support vs. Disconnection-During-Motion as Critical Error

The specification simultaneously supports hot-swap (NFR-010: recovery within 1–3 seconds; FR-005: device change event notification) and treats device disconnection during motion as a critical safety event requiring E-stop (Section 5.8). There is no boundary condition defined: when is a disconnect a valid hot-swap versus a safety-critical disconnection? The distinction likely hinges on robot motion state, but this logic is neither specified in QNC (which is supposed to be protocol-only) nor clearly delegated to the robot platform.

---

## 3. Definitions & Conceptual Clarity

### 3.1 PROFINET and CANopen in ProtocolType Enum Without Any Supporting Requirements

The `ProtocolType` enum (ICD, Section 6.6) includes `PROFINET` and `CANOPEN`. Neither protocol appears anywhere else in the ICD, PRD, or README — there are no hardware interfaces, no protocol adapters, no timeline, and no requirements for either. Including them in the production enum without any accompanying specification creates ambiguity about whether these are aspirational, planned, or simply placeholder values.

**Suggestion**: Either add placeholder requirements with a `[Future]` tag, or move undefined protocols to a comment block rather than active enum values.

---

### 3.2 Error Code Category Boundaries Are Internally Inconsistent (ICD, Section 12.1)

Section 12.1 declares:
> "Command Errors: -11 to -20"

But then lists `-4, -5, -6, -7, -10` under that category — all of which fall in the -1 to -10 range declared as "Communication Errors." The category boundaries are defined incorrectly; they appear to be copy-pasted labels that don't match the actual code values.

---

### 3.3 `DeviceChangeEvent` Undefined Semantics for Initial Connect / Final Disconnect

The `DeviceChangeEvent` structure contains `old_device` and `new_device`. For the `"connected"` reason (first ever device attach), `old_device` has no meaningful value. For `"disconnected"`, `new_device` is similarly undefined. The ICD does not specify what value these fields carry in those edge cases (null, empty struct, zeroed fields), which will cause inconsistent behavior across implementations.

---

### 3.4 "QNC Eliminates 99% of Device Driver Work" Is Unverifiable (README.md, Design Philosophy)

> "QNC as a universal protocol bridge that eliminates **99% of device driver development work**"

This is stated as a factual figure in the project's design philosophy heading. The 99% figure appears nowhere with any supporting measurement methodology. In practice, authoring a correct JSON descriptor requires reading and validating device register maps, handling scaling factors, verifying protocol parameters — work that is non-trivial and error-prone. The claim is marketing language presented as engineering fact, which is misleading in a technical reference document.

---

## 4. Product Development Perspective

### 4.1 Severe Scope Creep From Protocol Bridge to AI Edge Device

The PRD's functional requirements include:

- **FR-024**: Pose tracking
- **FR-025**: AI-driven object tracking (utilizing i.MX 8M PLUS NPU)
- **FR-026**: Intel D435i depth camera integration
- **FR-027**: Drive 3D cameras, grippers, vacuum pumps
- **FR-028**: "General AI features including **vision and language processing**"
- **FR-023**: NFC/Bluetooth pairing
- **FR-019 to FR-022**: Internal OLED UI, mode switching, motion switching

These features are architecturally orthogonal to a protocol bridge. A component that does Modbus register pass-through has no inherent reason to also perform language processing or AI object tracking. This scope creep undermines the stated core value proposition ("QNC handles protocols, NOT devices") and threatens the simplicity, maintainability, and cost targets the document itself defines.

**Concern**: At this scope, QNC is no longer a bridge — it is an autonomous robotics compute module. The cost target of **< €40 PCB** (NFR-023) is almost certainly incompatible with an NXP i.MX 8M PLUS with 1 GB LPDDR4, 16 GB eMMC, WLAN, Bluetooth, OLED display, IMU, depth camera, and battery management.

---

### 4.2 NeuraSync Dependency Undermines "Universal" Claim

The PRD states:
> "The unique value of QNC lies in its **universality**... it must remain compatible with most robotic arms and end-effectors."

But QNC's robot-side communication is entirely proprietary: it requires **NeuraSync**, an in-house FastDDS framework. Any robot platform that does not use NeuraSync cannot use QNC. This is not universal compatibility — it is compatibility within a single vendor's ecosystem. An OPC-UA or plain FastDDS interface would be more consistent with the universality claim.

---

### 4.3 Dual Interface Redundancy: Generic vs. Protocol-Specific Topics

QNC defines both a **generic interface** (`qnc/bridge/command`, `qnc/bridge/response`) and **protocol-specific interfaces** (`qnc/modbus/write_cmd`, `qnc/modbus/read_cmd`, etc.) that are explicitly Modbus-only. Section 6.1 says:
> "Robot applications should preferably use the generic interface for maximum flexibility."

If the generic interface is the preferred and sufficient path, the justification for maintaining separate Modbus-specific topics needs to be documented. Otherwise, implementers will split across two interfaces, creating maintenance burden. The ICD describes both as active with no deprecation guidance.

---

### 4.4 OTA Update Mechanism Security Risk: Robot Controller as Update Source

Section 5.9 describes "NeuraSync Transfer Update" where the robot controller can push firmware to QNC via the same DDS channel used for gripper commands. This creates a supply chain attack vector: a compromised robot controller could push malicious firmware to QNC. The update mechanism does include signature verification, but the trust model — specifically whether a robot controller's "push" command is authenticated separately from the signature check — is not addressed.

---

### 4.5 README.md Contains Garbled / Broken Documentation

The README has two rendering errors that suggest copy-paste corruption:

1. The IO-Link bridge section contains `python3 scant/Import Docker Image (Offline)` — a fragment that appears to be the start of a docker section accidentally embedded mid-code-block.
2. The "ModbusRTU Gripper (Docker)" section begins with `# Build DockerBridge` immediately followed by a new, unnested code block, suggesting a missing or broken section header.

These errors will confuse any developer following the quick-start guide.

---

## 5. General Risks & Concerns

### 5.1 PRD Authored by "Manus AI"

> "Author: Manus AI"

The PRD's documented author is an AI agent. While AI tools can accelerate drafting, a requirements document that will govern hardware and software decisions in a safety-adjacent robotics product needs explicit human sign-off. The ICD's document approval table (Engineering Lead, System Architect, QA) is entirely blank. No human has formally approved either foundational document.

**Risk**: Requirements may contain hallucinated figures (line counts, time estimates, cost targets) without engineering validation. The local `file:///` reference for the source hardware roadmap document (noted in §1.6 above) supports this concern.

---

### 5.2 Safety Responsibility Gap

ICD Section 2.2 excludes safety systems and emergency stop from scope. Section 5.8 of the PRD says QNC "recommends" an E-stop but the robot "SHOULD" (not SHALL) trigger it. No document clearly assigns safety responsibility. In industrial robotics (ISO 10218, IEC 62061), the safety function owner must be unambiguously specified. As written, neither QNC nor the robot platform is definitively responsible for the safety response to a mid-motion disconnect.

---

### 5.3 No Security Requirements for Network Communication Until "Future"

NFR-028 defers all secure communication to a future enhancement: "MAY incorporate secure communication protocols." For a device that:
- Accepts OTA firmware updates over a network
- Is physically mounted on a robot and controls end-effectors
- Communicates via WiFi (FR-009 lists WiFi 5G)

...having no baseline security requirement is a significant risk. The OTA update signing (Section 5.9) is good, but DDS communication itself has no authentication or encryption requirement defined.

---

### 5.4 REFACTORING_SUMMARY Indicates Only ModbusRTU Is Implemented

The refactoring document confirms:
> "QNC now operates as a clean, generic **ModbusRTU** protocol bridge"

IO-Link, EtherCAT, EtherNet/IP, CAN-FD, and all other protocols listed in the PRD and ICD are either planned or unimplemented. The PRD treats many of these as SHALL requirements. The gap between "currently implemented" (ModbusRTU only) and "required by PRD" (7+ protocols) is nowhere quantified in a gap analysis or roadmap with dates. This makes the PRD misleading as a current-state description.

---

### 5.5 Battery Life of 10 Minutes Has No Use Case Justification

FR-016 requires 10 minutes of battery operation when disconnected from the robot flange. No use case is documented for why autonomous detached operation is needed or what the device is expected to do for those 10 minutes. Without a use case, it is impossible to validate whether 10 minutes is sufficient, and the requirement adds significant BOM cost (battery, PMIC, charger IC, thermal management) without clear ROI.


