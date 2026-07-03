
# [SmartWrist](https://www.genspark.ai/api/files/s/ECwQmhjB) Interface Specification v2.0  
**Proposed Formal Interface Specification for compatibility with [NEURA Robotics](https://neura-robotics.com/) platforms ([MiPA](https://neura-robotics.com/products/mipa/), [LARA](https://neura-robotics.com/products/lara/), [MAiRA](https://neura-robotics.com/products/maira/))**

## Document Control

| Item | Value |
|---|---|
| Document title | SmartWrist Interface Specification v2.0 |
| Status | Proposed Draft |
| Supersedes | QNC SmartWrist CN Product Specification Pack Rev 1.0 |
| Intended use | Mechanical, electrical, software, and controls integration |
| Primary target platforms | NEURA MiPA / LARA / MAiRA, plus generic cobots |
| Conformance language | SHALL, SHOULD, MAY |

This v2.0 specification converts your draft SmartWrist concept into a formal interface definition optimized for modular tooling, robot portability, and higher-fidelity integration with NEURA-style controller architectures. It keeps the strong parts of your existing draft—24 V industrial power, ISO-style robot adapters, tool changing, RS485 baseline, and diagnostics—while adding a clearer fieldbus structure, tool-ID behavior, safety states, startup/shutdown rules, and a sealed production interface model. The direction is consistent with the SmartWrist draft, NEURA’s public interface positioning, and benchmark patterns from Zimmer, DH-Robotics, Inspire, and OnRobot. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://neura-robotics.com/products/lara/) [Source](https://d1hqi5mu4fov5n.cloudfront.net/pdfs/general/NEURA_model_overview_specs_EN_2.pdf) [Source](https://www.zimmer-group.com/en-us/products/components/industrial-communication/scm-master-gateway-for-simple-control) [Source](https://en.dh-robotics.com/product/rgi) [Source](https://onrobot.com/en/products/quick-changer)

---

# 1. Scope

This document defines the SmartWrist external and internal interface requirements, including mechanical mounting, connector interfaces, signal behavior, power budget, communication profiles, safety states, and startup/shutdown behavior. It is intended to serve as the baseline for EVT/DVT hardware design, firmware implementation, robot-side driver development, and partner integration documentation. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB)

This is a **proposed engineering specification**, not a NEURA-approved interface manual. Public NEURA materials confirm modular interfaces and industrial communications such as EtherCAT, IO-Link, Modbus, CAN bus, GPIO, and TCP/IP, but they do not publicly disclose full wrist-side mechanical drawings or tool connector pinouts for all models. Therefore, any NEURA-specific adapter or fieldbus profile defined here SHALL be validated against OEM drawings before release to production. [Source](https://neura-robotics.com/products/lara/) [Source](https://d1hqi5mu4fov5n.cloudfront.net/pdfs/general/NEURA_model_overview_specs_EN_2.pdf) [Source](https://neura-robotics.com/products/mipa/)

---

# 2. Design Intent and Conformance Philosophy

SmartWrist v2.0 SHALL be implemented as a **3-layer modular platform**:

```text
[Robot-side adapter]
        |
        v
+----------------------+
|   SmartWrist Core    |
| power / comm / safety|
| motion / diagnostics |
+----------------------+
        |
        v
+----------------------+
| Quick-change coupler |
| blind-mate media bus |
+----------------------+
        |
        v
+----------------------+
| Interchangeable tool |
| gripper / vacuum /   |
| screwdriver / custom |
+----------------------+
```

This layered structure follows the strongest pattern visible across your draft and market benchmarks: ISO-style robot adaptation, 24 V industrial power, simple fallback control such as RS485/DI/O, and richer smart-device integration using fieldbus and intelligent tool modules. Zimmer emphasizes intelligent endpoint integration and gateway-style communication bridging; DH and Inspire show the market reality of RS485/Modbus + DI/O as a practical baseline; OnRobot validates the value of fast modular tool exchange. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://www.zimmer-group.com/en/products/components/handling-technology/gripper-series-gep2000) [Source](https://www.zimmer-group.com/en-us/products/components/industrial-communication/scm-master-gateway-for-simple-control) [Source](https://en.dh-robotics.com/product/rgi) [Source](https://en.inspire-robots.com/wp-content/uploads/2024/02/INSPIRE-ROBOTS-ELECTRIC-GRIPPER-USER-MANUAL.pdf) [Source](https://onrobot.com/en/products/quick-changer)

The following conformance terms apply:

- **SHALL** = mandatory requirement  
- **SHOULD** = recommended unless justified otherwise  
- **MAY** = optional feature

---

# 3. System Architecture

## 3.1 Functional Blocks

```text
Robot Controller / App Layer
   |  EtherCAT / CAN / Modbus / IO-Link / GPIO
   v
Robot-side SmartWrist Driver
   |  capability API + profile manager
   v
SmartWrist Communication Layer
   |  uplink profile selection
   v
SmartWrist Motion & Safety Core
   |  motor, encoder, current, temperature, latch sensors
   v
Quick-change Interface
   |  power + comm + tool ID + latch detect
   v
Tool Module
```

The SmartWrist controller SHALL abstract transport from capability. Robot-side software SHOULD interact with high-level functions such as `home`, `grip`, `release`, `move_to_position`, `move_to_force`, `load_profile`, `identify_tool`, and `read_health`, rather than relying only on low-level register writes. This is consistent with the Tool Profile concept in your draft and with the broader trend toward easier robot-side integration. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://neura-robotics.com/products/lara/)

## 3.2 Supported Uplink Profiles

| Profile | Transport | Intended use | Status |
|---|---|---|---|
| U | RS485 Modbus RTU + DI/O | universal baseline, fastest deployment | SHALL |
| N | EtherCAT slave over 100BASE-TX | NEURA-native / premium integration | SHOULD |
| C | CAN or CAN-FD / CANopen-style object model | mid-tier industrial integration | SHOULD |
| I | IO-Link device mode | smart endpoint integration | MAY |
| B | BLE 5.0 | commissioning only | SHALL for setup, SHALL NOT be used as primary safety-critical control |

This profile structure reflects your draft architecture while aligning more closely with NEURA’s public interface stack and industry integration practice. LARA and NEURA public materials reference EtherCAT, IO-Link, Modbus, CAN bus, GPIO, and TCP/IP. DH-Robotics shows the common commercial pattern of RS485 baseline with optional EtherCAT/CAN/TCP/IP/PROFINET. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://neura-robotics.com/products/lara/) [Source](https://d1hqi5mu4fov5n.cloudfront.net/pdfs/general/NEURA_model_overview_specs_EN_2.pdf) [Source](https://en.dh-robotics.com/product/rgi)

---

# 4. Mechanical Interface

## 4.1 Robot-side Mounting

SmartWrist SHALL use an interchangeable robot-side adapter concept based on the **ISO 9409-1 family** wherever possible. Adapter kits SHALL be used for robot compatibility rather than redesigning the SmartWrist core housing per robot brand. This is consistent with your draft and with standard robot-to-tool interface practice. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://www.graspmonkey.de/en/blogs/news/iso-9409-1-die-schnittstelle-zwischen-roboter-und-werkzeug)

### Supported baseline flange families

| Family | Typical designation | Typical use |
|---|---|---|
| ISO-50 class | ISO 9409-1-40-4-M6 | compact cobots |
| ISO-63 class | ISO 9409-1-50-4-M6 | common collaborative robots |
| ISO-80 class | ISO 9409-1-63-6-M6 | larger wrists / higher moment margin |

**Source note:** The exact dimensions and naming pattern come from ISO 9409-1 flange practice; your draft already targets Ø50/63/80 compatibility through adapter kits. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://www.graspmonkey.de/en/blogs/news/iso-9409-1-die-schnittstelle-zwischen-roboter-und-werkzeug)

## 4.2 SmartWrist Core Envelope

| Parameter | Requirement |
|---|---:|
| Wrist housing nominal diameter | 80–85 mm |
| Core height, robot flange to coupler | ≤ 65 mm target |
| Total assembly height to fingertip | application-dependent |
| Maximum nominal payload | 5 kg including tool |
| Core ingress rating | IP65 minimum target |
| Operating temperature | -10 °C to +60 °C |
| Storage temperature | -20 °C to +70 °C |

The v1 draft lists approximately Ø80 mm wrist geometry, ~65 mm module height, 5 kg payload, and the same environmental temperature range. v2.0 raises the default target ingress from optional/variant IP65 to **core-IP65 by design** because that is more appropriate for industrial robot integration, especially against the backdrop of NEURA’s public industrial positioning. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://neura-robotics.com/products/lara/) [Source](https://d1hqi5mu4fov5n.cloudfront.net/pdfs/general/NEURA_model_overview_specs_EN_2.pdf)

## 4.3 Quick-change Coupler

| Parameter | Requirement |
|---|---:|
| Tool change type | manual quick-change, blind-mate ready |
| Target change time | < 5 s |
| Minimum rated cycle life | > 50,000 changes |
| Blind-mate features | electrical contacts + latch detect |
| Reserved media | optional pneumatic pass-through |

The quick-change coupler SHALL support electrical blind mating at minimum. Pneumatic pass-through MAY be reserved for future vacuum tooling. This extends your draft in the same direction as OnRobot-style fast-change tooling while preserving your current manufacturability approach. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://onrobot.com/en/products/quick-changer)

---

# 5. Electrical Interface

## 5.1 Electrical Summary

| Parameter | Requirement |
|---|---:|
| Main supply | 24 VDC nominal |
| Allowed supply range | 21.6 V to 26.4 V |
| Peak input current | 4.0 A max |
| Peak motor current | 3.0 A peak |
| Standby current | < 0.2 A |
| Internal rails | 5 V logic, 12 V auxiliary |
| Reverse polarity protection | required |
| Hold-up energy | ~500 ms safe-state preservation minimum |

These values are inherited from the SmartWrist draft and aligned with the BOM’s power architecture, including 24 V input protection, 24→5 V conversion, 24→12 V auxiliary rail, and supercapacitor hold-up. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://www.genspark.ai/api/files/s/neaB0t0R)

## 5.2 Power Budget

### 5.2.1 System power budget

| Load group | Rail | Typical | Peak | Notes |
|---|---|---:|---:|---|
| MCU + logic | 5 V | 0.20 A | 0.45 A | control core |
| BLE / comms | 3.3/5 V | 0.08 A | 0.20 A | commissioning + diagnostics |
| Sensor stack | 3.3/5 V | 0.10 A | 0.20 A | encoder, temp, latch, IMU optional |
| Digital I/O frontend | 24 V / 5 V | 0.03 A | 0.10 A | excludes external DO loads |
| Motor drive idle/hold | 24 V | 0.20 A | 0.80 A | static hold or low-load state |
| Motor active motion | 24 V | 1.00–1.80 A | 3.00 A | grip ramp / motion peak |
| Tool-side auxiliary budget | 24 V | 0 A default | 0.50 A shared reserve | optional, profile-specific |

### 5.2.2 Power management rules

- Total SmartWrist input current SHALL NOT exceed **4.0 A peak**.  
- Tool-side auxiliary power, when enabled, SHALL be budgeted within the same system input envelope.  
- DO outputs SHALL be current-limited and protected.  
- Brownout handling SHALL prioritize safe motion stop, state retention, and fault logging over continued motion.  

These rules preserve the draft electrical limits while making the allocation explicit for system integrators. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://www.genspark.ai/api/files/s/neaB0t0R)

---

# 6. Connector Definitions

## 6.1 Connector Overview

| ID | Name | Type | Required | Function |
|---|---|---|---|---|
| J1 | Main Robot Uplink | M12 A-coded 8-pin male | SHALL | power + serial/CAN + DI/O |
| J2 | Service Port | M8 4-pin female | SHALL | service, debug, firmware |
| J3 | Fieldbus Port | M12 D-coded 4-pin female | SHOULD | EtherCAT / 100BASE-TX |
| J4 | Tool-side Blind-Mate Interface | custom 10-pin pogo/block | SHALL | tool power + comm + ID + latch |
| P1/P2 | Pneumatic Pass-through | optional media ports | MAY | vacuum/tool air reserve |

This consolidates the interface set into a sealed production architecture. v1 referenced M12 robot-side, M8 service, optional IO-Link, and RJ45/Ethernet; v2.0 replaces exposed RJ45 with industrial sealed fieldbus connection and formalizes the tool-side blind-mate bus. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://www.genspark.ai/api/files/s/neaB0t0R)

---

## 6.2 J1 — Main Robot Uplink Connector

**Connector type:** M12 A-coded 8-pin male  
**Primary use:** 24 V power, baseline communication, discrete I/O

### Pin table

| Pin | Signal | Direction | Type | Description |
|---|---|---|---|---|
| 1 | +24V_MAIN | In | Power | Main supply input |
| 2 | 0V_MAIN | In | Power return | Supply return |
| 3 | COM_A | I/O | Differential | RS485_A or CAN_H, profile-selected |
| 4 | COM_B | I/O | Differential | RS485_B or CAN_L, profile-selected |
| 5 | DI1 | In | 24 V digital | general input / safety input A (profile-defined) |
| 6 | DI2 | In | 24 V digital | general input / safety input B (profile-defined) |
| 7 | DO1 | Out | 24 V open drain | status / interlock output |
| 8 | DO2 | Out | 24 V open drain | status / interlock output |
| Shield | PE_CHASSIS | — | Shield | chassis/protective earth |

### Electrical limits

| Signal class | Requirement |
|---|---:|
| DI high threshold | > 15 V |
| DI low threshold | < 5 V |
| DO type | open drain, PNP/NPN-compatible system use |
| DO max current | 500 mA per channel |
| COM isolation | recommended on RS485/CAN path |

This pin structure preserves the strongest part of your original draft while allowing the same connector to serve universal RS485 or optional CAN-based profiles. The DI/DO thresholds and current limits come directly from your draft. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB)

---

## 6.3 J2 — Service Port

**Connector type:** M8 4-pin female  
**Primary use:** commissioning, firmware update, diagnostics, manufacturing test

### Pin table

| Pin | Signal | Direction | Type | Description |
|---|---|---|---|---|
| 1 | +5V_SVC | Out | Power | service power, 500 mA max |
| 2 | USB_D- | I/O | USB 2.0 | service data |
| 3 | GND_SVC | — | Ground | service ground |
| 4 | USB_D+ | I/O | USB 2.0 | service data |

The service port SHALL be non-production-critical and SHALL NOT be required for runtime operation. v2.0 uses one sealed service strategy to avoid ambiguity between M8 service and exposed USB-C. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://www.genspark.ai/api/files/s/neaB0t0R)

---

## 6.4 J3 — Optional Fieldbus Port

**Connector type:** M12 D-coded 4-pin female  
**Primary use:** EtherCAT / 100BASE-TX industrial network

### Pin table

| Pin | Signal | Direction | Type | Description |
|---|---|---|---|---|
| 1 | TX+ | I/O | Differential | Ethernet pair |
| 2 | RX+ | I/O | Differential | Ethernet pair |
| 3 | TX- | I/O | Differential | Ethernet pair |
| 4 | RX- | I/O | Differential | Ethernet pair |
| Shield | PE_CHASSIS | — | Shield | chassis/shield |

### Rules

- J3 SHALL be used for **Profile N**.  
- J3 MAY be hidden behind a sealed cap or harness tail depending on robot integration package.  
- Exposed RJ45 SHOULD NOT be used on the wrist in production configurations.  

This is a proposed v2.0 addition to better fit premium industrial integration and to avoid the reliability issues of exposed RJ45 on moving EOAT. The need for industrial fieldbus compatibility is supported by NEURA’s public communications and benchmark products. [Source](https://neura-robotics.com/products/lara/) [Source](https://d1hqi5mu4fov5n.cloudfront.net/pdfs/general/NEURA_model_overview_specs_EN_2.pdf) [Source](https://en.dh-robotics.com/product/rgi)

---

## 6.5 J4 — Tool-side Blind-Mate Interface

**Connector type:** custom 10-pin blind-mate pogo/block with mechanical datum pins  
**Primary use:** power, tool comms, tool ID, coupler state

### Pin table

| Pin | Signal | Direction | Type | Description |
|---|---|---|---|---|
| 1 | +24V_TOOL | Out | Power | switched tool supply |
| 2 | 0V_TOOL | — | Power return | tool return |
| 3 | TOOL_COM_A | I/O | Differential | RS485_A / private tool bus |
| 4 | TOOL_COM_B | I/O | Differential | RS485_B / private tool bus |
| 5 | TOOL_ID_1W | I/O | 1-wire | tool EEPROM / identification |
| 6 | TOOL_PRESENT | In | Logic | tool seated detect |
| 7 | TOOL_LATCH_OK | In | Logic | coupler lock confirmation |
| 8 | AUX_IN | In | Logic/analog | reserved sensor input |
| 9 | AUX_OUT | Out | Logic | reserved actuator output |
| 10 | PE_TOOL | — | Shield/chassis | shield / bond |

### Media reservation

| Port | Function | Status |
|---|---|---|
| P1 | pneumatic / vacuum channel A | optional |
| P2 | pneumatic / vacuum channel B | optional |

The tool-side interface SHALL support tool identification and latch sensing as core platform features. Your BOM already includes a tool ID EEPROM, and the system concept depends on automatic profile loading to make the “many robots, many tools” value proposition real. [Source](https://www.genspark.ai/api/files/s/neaB0t0R) [Source](https://www.genspark.ai/api/files/s/ECwQmhjB)

---

# 7. Signal Definitions

## 7.1 Power Signals

| Signal | Nominal | Limits | Notes |
|---|---:|---:|---|
| +24V_MAIN | 24 VDC | 21.6–26.4 V | main system supply |
| +24V_TOOL | 24 VDC switched | shared system budget | disabled during unsafe states unless configured |
| 0V_MAIN / 0V_TOOL | 0 V | — | common return |
| +5V_SVC | 5 VDC | 500 mA max | service only |

## 7.2 Discrete Signals

| Signal | Type | Definition |
|---|---|---|
| DI1 / DI2 | 24 V input | user-configurable; may be mapped to interlock/safety channels |
| DO1 / DO2 | open-drain output | status, ready, gripping, fault, or tool-present indication |
| TOOL_PRESENT | logic input | high when tool face fully seated |
| TOOL_LATCH_OK | logic input | high when mechanical latch locked |

## 7.3 Serial / Fieldbus Signals

| Signal | Type | Definition |
|---|---|---|
| COM_A / COM_B | differential pair | RS485 or CAN physical layer depending on profile |
| TOOL_COM_A / B | differential pair | private tool bus |
| TX/RX pairs on J3 | Ethernet | EtherCAT / 100BASE-TX |

## 7.4 Identification and Commissioning Signals

| Signal | Type | Definition |
|---|---|---|
| TOOL_ID_1W | 1-wire | reads tool EEPROM, serial, profile, limits |
| BLE commissioning | wireless | setup, logs, non-safety diagnostics only |

Zimmer and comparable smart gripper ecosystems make heavy use of teachable profiles, integrated diagnostics, and easy control abstraction. The SmartWrist signal model SHOULD therefore expose identity, status, and health—not just motion. [Source](https://www.zimmer-group.com/en/products/components/handling-technology/gripper-series-gep2000) [Source](https://www.zimmer-group.com/en-us/products/components/industrial-communication/scm-master-gateway-for-simple-control)

---

# 8. Communication Protocol Specification

## 8.1 Profile U — Modbus RTU over RS485

### Physical layer
- Half-duplex RS485
- Default baud rate: **115200 bps**
- Format: **8N1**
- Multi-drop support: optional, not required for baseline

### Required commands

| Object | Access | Function |
|---|---|---|
| DEVICE_ENABLE | W | arm controller for operation |
| HOME_CMD | W | perform homing |
| GRIP_CMD | W | close/grip command |
| RELEASE_CMD | W | open/release command |
| TARGET_POS | W | target position |
| TARGET_FORCE | W | target grip force |
| TARGET_SPEED | W | target speed |
| STOP_CMD | W | controlled stop |
| CLEAR_FAULT | W | clear clearable faults |
| LOAD_TOOL_PROFILE | W | load profile by tool ID or profile slot |

### Required status objects

| Object | Access | Function |
|---|---|---|
| ACT_POS | R | actual position |
| EST_FORCE | R | estimated grip force |
| MOTOR_CURRENT | R | actual motor current |
| STATUS_WORD | R | state bitmap |
| FAULT_CODE | R | active fault |
| TEMP_MOTOR | R | motor temperature |
| TEMP_PCB | R | board temperature |
| TOOL_ID | R | tool identity |
| TOOL_PRESENT | R | seated/not seated |
| TOOL_LATCH_OK | R | latch status |
| CYCLE_COUNT | R | life diagnostics |

The baseline Modbus architecture is directly compatible with your draft and follows a common commercial pattern used by DH-Robotics and Inspire-style grippers. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://en.dh-robotics.com/product/rgi) [Source](https://en.inspire-robots.com/wp-content/uploads/2024/02/INSPIRE-ROBOTS-ELECTRIC-GRIPPER-USER-MANUAL.pdf)

### Proposed status word bit map

| Bit | Name | Meaning |
|---|---|---|
| 0 | BOOTING | device initializing |
| 1 | NOT_HOMED | homing required |
| 2 | READY | safe idle, commands accepted |
| 3 | MOVING | active motion |
| 4 | GRIPPED | part contact / hold state |
| 5 | TOOL_PRESENT | tool seated |
| 6 | LATCH_OK | coupler locked |
| 7 | WARNING | non-fatal warning |
| 8 | FAULT | fault active |
| 9 | SAFE_STATE | safety/estop state active |
| 10 | COMMISSIONING | BLE/service mode active |
| 11 | PROFILE_VALID | tool profile loaded |

---

## 8.2 Profile N — EtherCAT Slave

Profile N SHALL expose the same application object model as Profile U but over EtherCAT slave transport. For robot integration, the object dictionary SHOULD preserve identical semantic meaning for position, force, state, fault, and profile handling so that robot-side software can share the same capability abstraction across transport layers. This is recommended because NEURA publicly emphasizes EtherCAT-based standardized interfaces, and premium industrial integrations increasingly expect deterministic fieldbus support. [Source](https://neura-robotics.com/products/lara/) [Source](https://d1hqi5mu4fov5n.cloudfront.net/pdfs/general/NEURA_model_overview_specs_EN_2.pdf)

### EtherCAT minimum process data set

| PDO group | Required items |
|---|---|
| Control PDO | enable, stop, home, move cmd, target pos, target force, target speed |
| Status PDO | state, actual position, estimated force, current, tool present, fault |
| Diagnostics PDO | temperatures, cycle count, warning bits, tool ID hash |

---

## 8.3 Profile C — CAN / CAN-FD

CAN/CAN-FD MAY be implemented as a secondary industrial profile for customers who need more robust integration than RS485 but do not require EtherCAT. The object set SHOULD remain transport-consistent with Profiles U and N. NEURA public material references CAN bus among supported interfaces, making this a useful optional bridge profile. [Source](https://d1hqi5mu4fov5n.cloudfront.net/pdfs/general/NEURA_model_overview_specs_EN_2.pdf)

---

## 8.4 Profile I — IO-Link Device Mode

IO-Link support in SmartWrist v2.0 SHOULD be implemented first as **device mode**, not only as a local master/gateway concept. This better matches how intelligent grippers are commonly integrated into automation systems. Zimmer’s smart gripper and SCM examples show the value of endpoint intelligence and gateway translation for simpler systems; SmartWrist should therefore behave as a smart endpoint with optional bridging logic, not only as a bus host. [Source](https://www.zimmer-group.com/en-us/products/components/industrial-communication/scm-master-gateway-for-simple-control) [Source](https://www.zimmer-group.com/en/products/components/handling-technology/gripper-series-gep2000)

---

## 8.5 BLE Commissioning Rules

BLE SHALL be limited to:
- profile setup
- log retrieval
- commissioning
- firmware transfer
- non-safety diagnostics

BLE SHALL NOT be the sole channel for:
- active production motion control
- safety-critical stop functions
- coupler lock authorization

This follows the spirit of your draft, which positions BLE for commissioning and diagnostics rather than primary control. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB)

---

# 9. Safety and Fault Model

## 9.1 Safety Design Principles

SmartWrist SHALL fail safe to the greatest extent possible within its certification scope. The end-effector SHALL not rely only on wireless connectivity or software state for safe behavior. Safety handling SHALL prioritize:
1. motion stop,
2. grip retention where mechanically available,
3. fault annunciation,
4. deterministic recovery procedure.

This emphasis is necessary because collaborative robot ecosystems such as NEURA place strong value on safety architecture and sensing, while benchmark grippers such as Zimmer and Inspire highlight self-locking or power-loss retention behavior. [Source](https://neura-robotics.com/products/lara/) [Source](https://d1hqi5mu4fov5n.cloudfront.net/pdfs/general/NEURA_model_overview_specs_EN_2.pdf) [Source](https://www.zimmer-group.com/en/products/components/handling-technology/gripper-series-gep2000) [Source](https://en.inspire-robots.com/wp-content/uploads/2023/12/Inspire-Robots-Products-Selection-Guide-V15.pdf)

## 9.2 Safety States

| State | Name | Description |
|---|---|---|
| S0 | DE_ENERGIZED | no valid main power |
| S1 | BOOT_SELF_TEST | power valid, self-test active |
| S2 | SAFE_IDLE | powered, outputs in safe defaults, no motion |
| S3 | NOT_HOMED | ready for homing only |
| S4 | READY | homed and command-capable |
| S5 | MOTION_ACTIVE | executing controlled move |
| S6 | HOLDING | gripping/holding object |
| S7 | TOOL_CHANGE_MODE | motion inhibited, coupler unlock permitted |
| S8 | FAULT_SAFE | fault active, motion disabled |
| S9 | EMERGENCY_STOP_HOLD | emergency stop/interlock state |

## 9.3 State transition overview

```text
S0 -> S1 -> S2 -> S3 -> S4 -> S5 -> S6
               |         |         |
               |         +-------> S7
               |                   |
               +-----------------> S8
Any active state --------safety----> S9
S8/S9 --recovery sequence--> S2 or S3
```

## 9.4 Mandatory safety behaviors

| Event | Required behavior |
|---|---|
| Power applied | enter S1, outputs inactive |
| Self-test fail | enter S8 |
| Tool missing | inhibit motion-to-grip commands |
| Latch not confirmed | inhibit tool-change exit and production motion |
| E-stop / safety interlock | transition to S9 |
| Communication watchdog timeout | controlled stop, then S8 or S9 per profile |
| Brownout | preserve state, controlled stop if possible, log event |
| Overcurrent / locked rotor | stop motion, set fault |
| Overtemperature | derate, then fault if threshold persists |

---

# 10. Startup Behavior

## 10.1 Normal startup sequence

1. **Power validation**  
   SmartWrist SHALL verify supply voltage is within range before enabling motion subsystems.

2. **Self-test**  
   MCU, memory, encoder, current sensing, latch sensing, and communication interfaces SHALL be tested.

3. **Safe defaults**  
   DO outputs SHALL default inactive; tool power SHALL remain off until startup checks pass.

4. **Tool presence detection**  
   The controller SHALL read `TOOL_PRESENT` and `TOOL_LATCH_OK`.

5. **Tool identification**  
   If a tool is present, the controller SHALL read `TOOL_ID_1W` and load the matching profile.

6. **Health check**  
   Temperature, current offsets, and previous fault log SHALL be checked.

7. **Homing decision**  
   If absolute position is valid and last shutdown was clean, homing MAY be skipped by policy; otherwise the unit SHALL enter `NOT_HOMED`.

8. **Transition to SAFE_IDLE or READY**  
   The system SHALL not execute motion until the applicable ready conditions are met.

This startup flow operationalizes the Tool Profile approach already present in your draft and uses the tool ID concept present in the BOM. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://www.genspark.ai/api/files/s/neaB0t0R)

## 10.2 Startup timing targets

| Phase | Target |
|---|---:|
| power qualification | < 100 ms |
| core self-test | < 500 ms |
| tool ID + profile load | < 300 ms |
| ready-to-home indication | < 1.5 s total |
| ready-to-run indication | < 3.0 s typical |

---

# 11. Shutdown Behavior

## 11.1 Controlled shutdown

On commanded shutdown, SmartWrist SHALL:

1. reject new motion commands,  
2. finish or abort active motion safely,  
3. disable tool auxiliary power if configured,  
4. store last state, counters, and fault history,  
5. return to SAFE_IDLE, then de-energize motion stage.

## 11.2 Brownout / power loss shutdown

On brownout or sudden power loss, SmartWrist SHALL use hold-up energy to preserve a safe state for approximately **500 ms minimum**. During this interval, the controller SHALL:
- stop issuing new motion,
- preserve state data,
- record fault cause,
- maintain grip state only to the degree supported by mechanism and available energy.

The 500 ms hold-up requirement comes from your draft and BOM. Benchmark products also suggest that power-loss object retention is commercially important; in later revisions, SmartWrist SHOULD add a more explicit mechanical self-lock or brake function for premium applications. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://www.genspark.ai/api/files/s/neaB0t0R) [Source](https://www.zimmer-group.com/en/products/components/handling-technology/gripper-series-gep2000) [Source](https://en.inspire-robots.com/wp-content/uploads/2023/12/Inspire-Robots-Products-Selection-Guide-V15.pdf)

---

# 12. Diagnostics and Health Monitoring

## 12.1 Required monitored values

| Parameter | Source |
|---|---|
| actual position | encoder |
| estimated force | current model / force estimator |
| motor current | current sensing stage |
| motor temperature | temperature sensor |
| PCB temperature | temperature sensor |
| tool present | coupler sensor |
| latch status | coupler sensor |
| cycle count | firmware-maintained counter |
| communication health | watchdog |
| supply voltage | power monitor |

Your draft already includes position, force estimate, current monitoring, fault codes, and diagnostics tooling. The BOM expands this with temperature sensing, IMU, and tool ID support, so v2.0 formalizes them as first-class diagnostics. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://www.genspark.ai/api/files/s/neaB0t0R)

## 12.2 Warning thresholds

| Condition | Warning | Fault |
|---|---:|---:|
| motor temp | 75 °C | 85 °C |
| pcb temp | 70 °C | 80 °C |
| input undervoltage | < 22.0 V | < 21.0 V |
| overcurrent | — | instantaneous driver limit |
| repeated homing failure | after 2 retries | after 3 retries |

---

# 13. Tool Identification and Profile Handling

Each tool module SHALL contain a non-volatile tool identity element accessible through `TOOL_ID_1W`. The identity record SHOULD include:
- tool type,
- serial number,
- firmware compatibility range,
- stroke or media capability,
- recommended force/speed limits,
- maintenance counters,
- calibration version.

Automatic profile loading is one of the most important differentiators SmartWrist can implement. It turns a modular mechanical coupler into a true smart-tool platform and is directly supported by the BOM’s tool ID element. [Source](https://www.genspark.ai/api/files/s/neaB0t0R)

### Minimum tool profile fields

| Field | Required |
|---|---|
| tool class | yes |
| serial number | yes |
| max position | yes |
| max force | yes |
| recommended speed range | yes |
| maintenance interval | yes |
| calibration checksum | yes |

---

# 14. Minimum Robot-side Software API

| Function | Description |
|---|---|
| `get_device_info()` | read model, FW, tool ID |
| `get_state()` | read state + warnings + faults |
| `home()` | perform homing |
| `move_to_position(pos, speed)` | position move |
| `grip(force, speed)` | force-limited grip |
| `release(speed)` | open/release |
| `stop()` | controlled stop |
| `enter_tool_change_mode()` | safe tool change state |
| `exit_tool_change_mode()` | validate latch + profile before ready |
| `clear_fault()` | recover clearable faults |
| `read_health()` | temperatures, current, counters |
| `load_profile(id)` | profile select |

---

# 15. Compliance and Environmental Targets

| Item | Target |
|---|---|
| EMC / CE / LVD | planned / required for productization |
| RoHS / REACH | required |
| IEC 60529 | IP65 target minimum for core |
| IO-Link standard conformance | if Profile I implemented |
| EtherCAT conformance | if Profile N implemented |
| vibration/shock | equal or better than v1 draft targets |

Your original draft already lists CE, RoHS, REACH, IEC 60529, and related compliance targets, with IP54/IP65 differentiation and vibration/shock requirements. v2.0 keeps the compliance direction but hardens the ingress requirement for the core. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB)

---

# 16. Open Engineering Actions Before EVT Freeze

| Priority | Action |
|---|---|
| High | confirm NEURA wrist geometry and connector packaging with OEM drawings |
| High | resolve final service-port strategy in CAD and harness design |
| High | define whether CAN is populated in rev A hardware |
| High | freeze tool-side blind-mate contact stack and wear test plan |
| High | finalize latch sensing and unlock interlock logic |
| Medium | decide whether EtherCAT is on-baseboard or mezzanine |
| Medium | add mechanical hold/brake strategy for power-loss grip retention |
| Medium | define tool EEPROM data structure and versioning |
| Medium | complete register/object dictionary mapping for all profiles |

---

# 17. Reference Visuals

These reference visuals from the benchmark material are useful for industrial design, interface review, and presentation prep:

- [OnRobot Quick Changer product image](https://onrobot.com/storage/products/quick-changer.png)  
- [Zimmer SCM communication module image](https://www.zimmer-group.com/fileadmin/daten/Kommunikation/SCM/SOM_BIL_ART_SCM-C-00-00-A__SALL__APR__V2.png)  
- [DH-Robotics RGI overview image](https://www.dh-robotics.com/wp-content/uploads/2021/11/RGI-Vedio.jpg)  
- [ISO 9409-1 flange explainer image](https://www.graspmonkey.de/cdn/shop/articles/Screenshot_2025-04-08_135148_54330fa3-69c4-4363-8c1c-de040cfbd9f5.png?v=1744113407&width=1100)

---

# 18. Approval Summary

**Recommended release basis:** use this v2.0 document as the interface-control baseline for hardware revision planning, firmware object-dictionary definition, and NEURA partner discussion.  
**Main architectural decision:** keep RS485/DI/O as the universal baseline, but design the hardware now so EtherCAT, CAN, tool-ID, and blind-mate tool modules are native citizens of the platform.  
**Most important change versus v1:** SmartWrist is now specified as a **modular smart wrist platform**, not only a gripper assembly. [Source](https://www.genspark.ai/api/files/s/ECwQmhjB) [Source](https://neura-robotics.com/products/lara/)

If you want, I can next turn this into either:
- a **one-page ICD diagram pack** with cleaner block diagrams and pinout drawings, or
- a **formal verification matrix** listing each requirement, test method, pass criteria, and responsible team.
