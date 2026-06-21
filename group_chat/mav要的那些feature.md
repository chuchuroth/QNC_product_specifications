# Reconciled Addendum: New Customer Requirements vs QNC PRD/ICD

**Date**: 2026-02-23  
**Status**: Proposed (for review and approval)  
**Base Documents**:
- `docs/new_customer_requirements.md`
- `docs/QNC_Product_Requirements_Document_v2.md`
- `docs/QNC_INTERFACE_CONTROL_DOCUMENT.md`

---

## 1) Purpose

This addendum reconciles customer work packages with the original QNC product ideology and interface contracts:

- QNC remains a **protocol-level bridge**.
- Device-specific semantics and robot application logic remain on the **robot platform** (descriptor-driven).
- Existing NeuraSync topic contracts and timing/QoS requirements remain normative unless explicitly versioned.

---

## 2) Reconciliation Rules (Normative)

1. **Boundary Rule**
   - QNC firmware scope: protocol framing, bus access, timing/retry/error behavior, protocol discovery, bridge interfaces.
   - Robot platform scope: device meaning, `RobotCommand` semantics, UI/business state machines, capability modeling.

2. **Topic Contract Rule**
   - Keep IF-001..IF-008 and system topics as primary external contract.
   - Any new topic family (e.g., video/imu streaming) must be declared as a **versioned extension profile** and must not break existing interfaces.

3. **QoS Compatibility Rule**
   - Existing bridge topics keep current ICD QoS defaults.
   - New high-bandwidth topics may define separate QoS profiles but cannot weaken reliability/timing guarantees of bridge command/response paths.

4. **Performance Rule**
   - New capabilities must not violate command/translation/protocol timing budgets in PRD/ICD.
   - Resource-heavy functions (camera/audio pipelines) require isolation and explicit CPU/memory budgets.

5. **Safety/Control Rule**
   - Emergency or safety semantics are not defined in this ICD scope.
   - Physical stop-button events from QNC are exported as neutral events/status; robot safety controller decides action.

---

## 3) Disposition of Original Customer Work Packages

### 3.1 Keep in QNC Scope (Aligned with Protocol Bridge)

- **WP-A02 GPIO Interrupt Driver**
  - Keep as hardware event input layer.
  - Output only neutral event notifications to robot platform; no device/business semantics in QNC.

- **WP-A06 Hardware Watchdog**
  - Keep as platform reliability hardening.

- **WP-B01 IDL Code Generation**
  - Keep, but generation should include existing bridge/interface IDL as baseline compatibility set.

- **WP-B05 Topic Liveliness Monitoring**
  - Keep for transport health monitoring.

- **WP-C01 Modbus TCP/RTU Polling**
  - Keep protocol transport/polling behavior in QNC.
  - Data interpretation beyond protocol validity remains robot-side via descriptors.

- **WP-C02 IO-Link Data Parsing**
  - Keep protocol-level parsing/transport packaging in QNC.
  - Device meaning/unit conversion remains robot-side.

### 3.2 Keep as Optional/Planned Extension (Do Not Redefine Core Contract)

- **WP-A03 RealSense Baseline Pipeline**
- **WP-A05 Audio ALSA Integration**
- **WP-C03 HMI Display Driver**

These are acceptable as product extensions or board capabilities, but not required for protocol-bridge MVP compliance.

### 3.3 Re-scope to Robot Platform (Contradiction if Kept in QNC Core)

- **WP-C04 State Machine Logic + `RobotCommand` / `CMD_STOP` semantics**
  - Move command semantics and mode logic to robot platform application/safety stack.
  - QNC may publish `button_event`/`state_event`; robot platform maps to `RobotCommand`.

### 3.4 Conditional, with Interface Governance

- **WP-A04 / WP-B03 / WP-B04 Zero-Copy video path (`loan_sample`)**
  - Allowed only as an extension profile (`qnc/ext/video/*`) with explicit version and compatibility note.
  - Must not alter IF-001..IF-008 behavior.

---

## 4) Dependency Corrections

Original dependency cycle is invalid:
- WP-A04 depends on WP-B03
- WP-B03 depends on WP-A04

### Corrected order
1. **B01** IDL generation and base types
2. **B02** QoS profile baseline
3. **B03** Publisher loan API scaffolding (without camera allocator hook)
4. **A04** Camera allocator integration to loaned buffer
5. **B04** Subscriber + `return_loan()` validation

This removes circular dependency and enables incremental verification.

---

## 5) Rewritten Aligned Work Packages (Normative Candidate)

## A. Embedded Linux & Hardware Integration

- **A01 (Optional hardening)**: OS shared-memory/swap tuning for deterministic behavior.
- **A02 (Required)**: GPIO interrupt capture with debounce; publish neutral `button_event`.
- **A03 (Extension)**: RealSense baseline pipeline as optional media feature.
- **A04 (Extension)**: Zero-copy camera bridge integration with extension topic profile.
- **A05 (Extension)**: ALSA capture pipeline as optional media feature.
- **A06 (Required)**: Hardware watchdog keepalive and reset validation.

## B. DDS & Data Architecture

- **B01 (Required)**: IDL generation for bridge contracts + extension contracts.
- **B02 (Required)**: QoS profile file; preserve bridge topic QoS; define extension QoS separately.
- **B03 (Extension)**: Zero-copy publisher for media extension topics.
- **B04 (Extension)**: Zero-copy subscriber and memory lifecycle validation.
- **B05 (Required)**: Liveliness monitoring for bridge health and fault reporting.

## C. Industrial Protocol Integration

- **C01 (Required)**: Modbus RTU/TCP transport polling at configured rate with transport-quality status.
- **C02 (Required)**: IO-Link process-data transport extraction and forwarding.
- **C03 (Extension)**: Local HMI text UI only for diagnostics/config (no device semantics).
- **C04 (Re-scoped)**: Robot platform owns command/state machine semantics; QNC exports events only.
- **C05 (Required)**: End-to-end stress test with separate metrics for bridge path vs extension path.

---

## 6) Acceptance Criteria Alignment

To stay compliant with PRD/ICD ideology, all new work should satisfy:

1. **Bridge compatibility**: IF-001..IF-008 unchanged and passing compatibility tests.
2. **Boundary compliance**: No device register meaning or command semantics hard-coded in QNC core.
3. **Timing non-regression**: Existing command/translation/protocol timing envelopes preserved.
4. **Extension isolation**: Video/audio/HMI features can be disabled without affecting bridge operation.
5. **Fault behavior**: Liveliness/watchdog faults are observable and recoverable without contract breakage.

---

## 7) Change-Control Recommendation

Adopt this as `QNC-ADD-NEWCUST-001` and review under architecture control board with:
- PRD owner
- ICD owner
- QNC firmware lead
- Robot platform lead

Approval outcome should mark each WP as: **Core**, **Extension**, or **Robot-side**.


---
---
---
# New Customer Requirements (Aligned Version)

**Date**: 2026-02-23  
**Purpose**: Preserve customer intent while aligning with QNC architecture principle: protocol bridge responsibilities in QNC, device semantics and application logic on robot platform.

## Developer A: Embedded Linux & Hardware Integration

### WP-A01: OS Memory Partitioning *(Optional hardening)*
- **Action**: Configure deterministic runtime memory (`/dev/shm` and swap policy) without introducing regressions to bridge processes.
- **Exit Criteria**: Shared memory and swap settings are validated at boot and documented in deployment profile.
- **Dependencies**: None.

### WP-A02: GPIO Interrupt Driver *(Core)*
- **Action**: Implement `libgpiod` event listener for 4 hardware buttons with 200 ms debounce. Publish neutral `button_event` notifications.
- **Exit Criteria**: Interrupt-to-event latency < 10 ms. No double-trigger over 100 presses.
- **Dependencies**: None.

### WP-A03: RealSense Baseline Pipeline *(Extension)*
- **Action**: Initialize `librealsense2` RGB + Depth capture at 640x480@30 FPS as optional media pipeline.
- **Exit Criteria**: Stable 30 Hz capture loop when extension is enabled.
- **Dependencies**: None.

### WP-A04: Zero-Copy Camera Memory Mapping *(Extension)*
- **Action**: Integrate camera callback with DDS loaned buffer API from WP-B03 under extension topic namespace.
- **Exit Criteria**: No user-space frame copy in enabled extension path; bridge command/response timing remains within baseline limits.
- **Dependencies**: WP-B03.

### WP-A05: Audio ALSA Integration *(Extension)*
- **Action**: Configure ALSA capture (16 kHz, 16-bit mono) with bounded chunking policy for optional audio stream profile.
- **Exit Criteria**: Continuous bounded capture loop without overrun in extension mode.
- **Dependencies**: None.

### WP-A06: Hardware Watchdog *(Core)*
- **Action**: Integrate `/dev/watchdog` keepalive thread at 2 s interval.
- **Exit Criteria**: Forced keepalive suspension causes reboot within configured watchdog timeout.
- **Dependencies**: None.

## Developer B: Fast DDS & Data Architecture

### WP-B01: IDL Code Generation *(Core)*
- **Action**: Run `fastddsgen` for bridge IDL contracts and extension contracts with `-typeobject`.
- **Exit Criteria**: Generated sources compile cleanly in CMake build.
- **Dependencies**: None.

### WP-B02: QoS XML Implementation *(Core)*
- **Action**: Define `qos_profiles.xml` with separate profiles for bridge topics and extension media topics.
- **Exit Criteria**: Bridge endpoints match and preserve existing reliability/history guarantees; extension endpoints match in SHM mode.
- **Dependencies**: WP-A01.

### WP-B03: Zero-Copy Publisher Pipeline *(Extension)*
- **Action**: Implement `loan_sample(void*& sample)` publisher pipeline for media extension topics.
- **Exit Criteria**: Extension publisher runs at 30 Hz target without affecting bridge path performance budgets.
- **Dependencies**: WP-B02.

### WP-B04: Zero-Copy Subscriber Pipeline *(Extension)*
- **Action**: Implement `take_next_sample()` + `return_loan()` and validate memory lifecycle.
- **Exit Criteria**: Stable 30 FPS subscriber throughput and no memory growth over soak test.
- **Dependencies**: WP-B03.

### WP-B05: Topic Liveliness Monitoring *(Core)*
- **Action**: Configure LIVELINESS monitoring and callbacks for transport health/fault events.
- **Exit Criteria**: Liveliness loss detection meets configured timeout and is reported on diagnostic/status channels.
- **Dependencies**: WP-B02.

## Developer C: Industrial Protocol & System Integration

### WP-C01: Modbus TCP/RTU Polling *(Core)*
- **Action**: Implement configurable polling loop for Modbus transports and expose protocol-level data/quality status.
- **Exit Criteria**: Valid transport data observed; quality transitions to BAD on disconnect.
- **Dependencies**: WP-B01.

### WP-C02: IO-Link Data Parsing *(Core)*
- **Action**: Extract IO-Link process data from SPI/UART path and publish transport-level payload/status.
- **Exit Criteria**: IO-Link events propagate within 100 ms target envelope.
- **Dependencies**: WP-B01.

### WP-C03: HMI Display Driver *(Extension)*
- **Action**: Provide text-based local diagnostics/configuration UI with constrained footprint.
- **Exit Criteria**: Refresh > 15 Hz and binary footprint < 20 MB in extension build.
- **Dependencies**: None.

### WP-C04: State Machine Logic *(Re-scoped to Robot Platform)*
- **Action**: QNC publishes neutral button/state events only. Robot platform owns `RobotCommand` mapping (including stop semantics).
- **Exit Criteria**: End-to-end button-event propagation < 50 ms; command/safety semantics verified on robot platform.
- **Dependencies**: WP-A02, WP-B01.

### WP-C05: System-Level Stress Test *(Core validation)*
- **Action**: Run integrated soak test for 8 hours, recording CPU, RAM, dropped DDS frames, and bridge timing.
- **Exit Criteria**: No crash; bridge timing contracts maintained; extension workloads do not regress bridge SLA.
- **Dependencies**: All required core WPs; extension WPs tested separately when enabled.

---

## Dependency Correction Note

The original cycle `WP-A04 -> WP-B03 -> WP-A04` is replaced by:
1. `WP-B01` -> 2. `WP-B02` -> 3. `WP-B03` -> 4. `WP-A04` -> 5. `WP-B04`

This removes circular dependency and supports staged verification.
