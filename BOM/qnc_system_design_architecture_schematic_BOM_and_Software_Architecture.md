# Industrial Edge Gateway — Hardware Design and Software Architecture

---

## 1) System Interpretation and Design Assumptions

I interpret QNC as a **software-first industrial edge gateway** that in the baseline supports **IO-Link, Modbus RTU, EtherNet/IP adapter, and discrete digital I/O**, provides northbound **REST, WebSocket, and structured logs**, and should be extensible for **FastDDS, Modbus TCP, CANopen, OPC UA, and MQTT**. The system is **not safety-rated**, but must support fault isolation, safe mode, configuration signatures, rollback, diagnostic access, and robust industrial installation. For hardware this means: a Linux-capable compute platform, separated field interfaces, clean fault-containment zones, service-friendly diagnostic paths, and sufficient I/O and network headroom for the baseline plus optional extensions.

### Recommended hardware sizing for Design Freeze A

| Assumption | Proposal |
|---|---|
| Input supply | 24 VDC nominal, 18…36 VDC allowed |
| Compute architecture | Linux compute module + separate real-time / supervisory MCU |
| Ethernet | 2× Gigabit Ethernet, galvanically decoupled from the cable via magnetics |
| Modbus RTU | 2× isolated RS-485 |
| IO-Link | 4× master ports |
| Discrete I/O | 8× DI + 8× DO, configurable per bank |
| Extension-ready | 1× CAN/CANopen, prepared but optionally populated |
| Service | USB-C service/console, isolated debug header, boot recovery |
| Mounting | DIN rail or panel, 0…50 °C operation |
| Target product | Carrier / baseboard with industrial compute module |

These assumptions are consistent with the documented scope, the environmental data 0…50 °C / pollution degree 2, the requirement for at least several simultaneous devices/clients, the baseline protocols, and the extension paths. They do **not** replace the hardware decisions that are still missing.

### Architecture decision

For the first production-capable iteration I recommend **not** a monolithic single board with bare MPU + DDR, but a **baseboard with an industrial SoM**. That reduces DDR / PMIC / BGA risk, shortens layout and SI cycles, and given the missing low-level hardware data is the most robust path to EVT/DVT/PVT. That also fits the document character: the QNC materials specify primarily system and interface behavior, not a fixed CPU platform.

---

## 2) Schematic Partitioning and Hierarchy

### Recommended schematic hierarchy

1. **Sheet 01 – Power entry & protection**
   24 V input, reverse polarity, hot-swap/inrush, TVS, EMI filter, fuses, supply monitoring.

2. **Sheet 02 – Compute core**
   Industrial SoM, boot straps, reset, TPM, RTC, watchdog interface, status LEDs.

3. **Sheet 03 – Supervisory / RT MCU**
   State monitoring, rail monitoring, board ID, recovery, brownout handling, hardware watchdog.

4. **Sheet 04 – Northbound Ethernet**
   2× Ethernet port, magnetics, ESD, shield/chassis bonding, optional link/activity LEDs.

5. **Sheet 05 – Southbound RS-485 / Modbus RTU**
   2× isolated RS-485 channels, termination, biasing, protection circuitry, diagnostics.

6. **Sheet 06 – IO-Link master**
   4× master ports, port power L+, C/Q drivers, short-circuit/thermal protection, port ID.

7. **Sheet 07 – Discrete digital I/O**
   8× DI, 8× DO, optional per-bank sourcing/sinking, isolation/protection layer, status diagnostics.

8. **Sheet 08 – CAN / CANopen extension**
   Optionally populated CAN interface with bus protection and termination.

9. **Sheet 09 – Service, debug, recovery**
   USB-C device/console, UART, SWD/JTAG, recovery header, factory config DIP.

10. **Sheet 10 – Security & nonvolatile state**
    TPM, secure identity/board serial storage, optional secure element, event counter.

11. **Sheet 11 – Test & production support**
    Boundary test points, bed-of-nails access, current sense shunts, loopback options.

This partitioning reflects the separation required in the documents between physical/protocol adaptation, normalization, control, diagnostics, and service — mapped cleanly at the hardware level.

### Functional blocks at board level

- **Compute domain**: Linux/QNC runtime, REST/WebSocket, logging, DDS optional.
- **Supervisor domain**: deterministic board/power/fault supervision.
- **Fieldbus domain**: Modbus RTU, CANopen optional, IO-Link, DIO.
- **Service domain**: update, diagnostics, recovery.
- **Power domain**: separate rails for logic, interface, isolated field side.

Fault containment is thus physically supported: an RS-485 or IO-Link fault must not disturb the compute domain or other ports; that is explicitly built into the QNC fault model.

---

## 3) Detailed BOM Proposal

Important: this is a **recommended reference BOM** to implement the documented function. It is **not** specified in the QNC documents. Connectors, some power stages, and the final DIO topology remain open until interface sign-off.

### Critical active parts

| Block | Recommended part | Function | Key specs | Sourcing note |
|---|---|---|---|---|
| Compute SoM | **Toradex Verdin iMX8M Plus Quad 2GB WB IT** | Linux/edge/DDS/REST/WebSocket | Industrial SoM, reduced DDR risk, mature Linux BSP | Preferred default for carrier-board architecture |
| Alternative SoM | **Variscite DART-MX8M-PLUS 2GB Industrial** | Alternative compute platform | Alternate industrial SoM source | Use only if supply-chain strategy requires second platform |
| Supervisor MCU | **STM32G474RET6** | Power/reset/watchdog/health supervisor | Industrial grade, enough ADC/timer/I/O | No direct second source; check lifecycle |
| TPM | **OPTIGA-TPM-SLB-9672-FW15** | Secure boot / keys / attestation | SPI TPM 2.0 | Version security BOM separately |
| IO-Link master | **MAX14819ATM+** ×2 | 4 IO-Link master ports (2 chips × 2 ports) | Integrated framer/L+ control | Critical long-life part; stock early |
| RS-485 | **ISO1410DWR** ×2 | 2 isolated Modbus RTU ports | 5 kVrms isolation, robust EMC | Good choice for industrial front end |
| CAN (extension) | **ISO1042BDW** | CAN/CANopen — isolated CAN FD transceiver | Fault-protected, suitable for CANopen at classic rates | Preferred for harsh industrial field cabling |
| CAN (lower-cost alternative) | **TCAN1042HVDRQ1** | CAN/CANopen-ready | Fault-protected CAN transceiver | Use only where system-level isolation is not required |
| Digital input | **ISO1212DBQR** ×4 | 8 isolated 24 V DI channels | 2 channels/IC, industrial input receiver | IEC 61131-like input topology |
| Digital output (sourcing) | **TPS272C45ARHFR** ×4 | 8 high-side DO channels | 36 V, 3 A, smart high-side | Only if sourcing is finally approved |
| DIO alternative | **MAX14900EAGM+** ×2 | Flexible DIO topology | Per-channel flexible, industrial DO/DIO | Check whether logic model fits ICD |
| Hot-swap/inrush | **LM5069MM-2/NOPB** | 24 V input protection | Hot-swap / inrush / fault cutoff | Required for robust industrial power-in |
| Main buck | **LM76005RNPR** | 24 V → 5 V main supply | 60 V input tolerant, 5 A | Good robustness on 24 V bus |
| Isolated DC/DC | **NXJ2S0505MC-R7** ×2…4 | Isolated auxiliary supplies for field side | 5 V / 2 W isolated | Count per channel grouping |
| 5 V → 3.3 V regulator | **TPS62130RGTR** or equivalent | Secondary point-of-load regulation | Efficient, local rails per domain | Use as needed |
| RTC | **RV-3028-C7** | Timestamps / audit / recovery | Ultra-low-power RTC | Optional but recommended |
| EEPROM | **24AA256T-I/ST** | Board ID, BOM variant, calibration, manufacturing data | Simple, proven industrial support device | Recommended |
| ESD array | **TPD4E05U06QDRQ1** | ESD protection for service/data lines | Low-capacitance ESD | Ethernet needs a separate protection strategy |
| TVS 24 V input | **SMBJ33A** | Surge/transient clamp | 600 W class | Select vendor by surge level |
| Common-mode chokes | Würth / TDK industrial families | Line emissions | Differential per interface | Tune in pre-compliance |
| Ethernet connector | **TBD by enclosure** | RJ45 shielded or M12 X-coded | Mechanically open | Not finalizable without mechanical sign-off |
| Field connectors | **TBD by mechanics** | IO-Link / RS-485 / DIO / CAN | Screw terminal or M8/M12 | Depends on mechanics and IP |

### Industrial Ethernet co-processor (Phase 2)

| Block | Recommended part | Function | Notes |
|---|---|---|---|
| Real-time Ethernet co-processor | **Hilscher netX 90** (host-controlled variant) | EtherCAT / PROFINET / EtherNet/IP offload | Place on PCB from rev A; DNP in Phase 1 |
| 100BASE-TX PHY | **TI DP83822IFRHBR** ×2 | Industrial Ethernet PHY for co-processor ports | Wide temp, strong EMC reputation |
| Host interconnect | SPI: COPROC_SPI_CLK/MOSI/MISO/CS_N + COPROC_IRQ_N / RESET_N / BOOTMODE[1:0] / SYNC(opt) | CPU-to-netX 90 link | SPI preferred over PCIe for first carrier revision |

### Passive BOM rules with tolerances

| Passive group | Recommendation |
|---|---|
| 24 V path | 50 V or 100 V voltage rating, X7R where MLCC, polymer/electrolytic at bulk |
| 5 V / 3.3 V decoupling | 0.1 µF + 1 µF + 10 µF staggered, X7R/X5R, 10 % sufficient |
| Feedback/sense networks | 1 % standard, 0.1 % / 25 ppm for voltage and current measurement |
| Bus terminations | RS-485/CAN 120 Ω, 1 %, 0.25 W min.; Ethernet per PHY/magnetics app note |
| RC filters for safety-critical diagnostics | 1 % / C0G or stable X7R if trigger thresholds are tight |
| Pull-ups/pull-downs for boot/reset | 1 % or 5 % per strap requirement |
| Shunts for current sense | 1 %, 1 W to 3 W depending on port current, 50 ppm recommended |

### Procurement strategy

- **A-parts**: SoM, IO-Link, isolated RS-485, TPM, power front end — secure LTB/LTS capability early.
- **B-parts**: Protection devices, isolation modules, DIO drivers — dual source.
- **C-parts**: Standardize passives to 0402/0603/0805, allow manufacturer pool.
- **No-go**: NCNR parts without lifecycle approval, unauthorized brokers for security parts.

---

## 4) PCB Stack-Up

Because the compute side is implemented as an SoM with a co-processor, an **8-layer stack-up** with controlled impedance is recommended for the carrier board to provide adequate SI/EMC margin for the dual-processor variant.

### Recommended stack-up

| Layer | Function | Copper | Note |
|---|---|---:|---|
| L1 | Components + critical signals | 35 µm | Short service/mgmt/diff pairs |
| L2 | Solid GND plane | 35 µm | Reference for L1/L3 |
| L3 | High-speed signals | 18 µm | Ethernet/USB/MCU interconnect |
| L4 | Power planes | 18 µm | 24 V / 5 V / 3.3 V islands |
| L5 | Solid GND | 35 µm | Shield and return current routing |
| L6 | Medium-speed signals / control buses | 18 µm | SPI, supervisor, netX 90 link |
| L7 | Field power / low-speed routing | 35 µm | 24V_FIELD, IO-Link L+, DIO power |
| L8 | Field-side components and connectors | 35 µm | Transceivers, connectors, protection |

### Material and manufacturing target

- **FR-4, Tg ≥ 170 °C**
- Total thickness **1.6 mm**
- Outer layers **1 oz**, inner layers **0.5 oz**
- Controlled impedance coupons:
  - 50 Ω single-ended
  - 90 Ω diff for USB 2.0
  - 100 Ω diff for Ethernet/high-speed pairs

> **Note:** A 6-layer stack-up remains viable for a Phase 1 only variant without the netX 90. Moving to 8 layers is recommended when the co-processor is populated to support its additional high-speed routing requirements.

---

## 5) High-Speed and Sensitive Routing Guidelines

### General routing rules

- No split-GND return paths under high-speed signals.
- Shield/chassis bonding at the connector, **not** deep inside the board.
- Field interfaces at the edge; compute/TPM/service toward the center.
- Isolation barriers visibly executed as keep-out + creepage channel.
- Every external line first through **ESD/surge/filter**, then to the IC.

### SPI between main CPU SoM and netX 90

- Keep SPI link on a continuous reference plane.
- Route as a tightly grouped bus with matched topology.
- Add 22–33 Ω series damping resistors at the driver side for CLK, MOSI, and optional CS.
- Keep CPU-to-coprocessor SPI trace length ideally below 50 mm on the carrier.
- Route COPROC_IRQ_N away from noisy switching nodes.
- Reserve test pads on SPI lines for bring-up; pinmux and BSP setup are typically the first integration bottleneck.

### Ethernet

- MDI pairs short, symmetric, no stubs to the magnetics module.
- 100 Ω diff, small pair length skew, same layer preferred.
- No sharp 90° corners; 45°/arc.
- Bob Smith / shield net exactly per magnetics/connector app note.
- Chassis shield near port with 360° shield bond in the enclosure concept.
- Keep both real-time Ethernet ports (netX 90 / PHYs) geometrically symmetric.
- Place PHY clock sources away from power magnetics and hot-switch nodes.

### RS-485 / CAN

- Terminate near the port; bias only once per segment.
- TVS + optional common-mode choke near the port.
- Isolators/isolated transceivers between logic and field sides.
- Do not route bus lines parallel to aggressive switching nodes of the 24 V buck.

### IO-Link

- Build port channels identically, symmetric layout topology.
- L+ current path wide, thermally relieved, individually fused.
- C/Q lines short, low noise, with defined return current path.
- No shared bottlenecks for multiple port return currents.

### Sensitive nets

- Keep TPM SPI, reset, boot straps, supervisor signals away from field cables.
- Route ADC/sense lines in Kelvin style.
- Crystals/RTC close to IC; guarding for very sensitive nets.

---

## 6) PDN Design

### Recommended power tree

| Rail | Source | Typical load | Comment |
|---|---|---:|---|
| 24V_IN | External industrial input | system-wide | protected, monitored |
| 24V_FIELD | fused from 24V_IN | IO-Link/DIO/field | fused per port/bank |
| 5V_MAIN | LM76005 from 24V_IN | SoM + logic | central logic rail |
| 5V_ISO_A | isolated DC/DC | RS-485/CAN group A | field isolation |
| 5V_ISO_B | isolated DC/DC | RS-485/CAN group B | field isolation |
| 3V3_LOGIC | from 5V_MAIN | MCU/TPM/glue logic | locally filtered |
| 1V8_AUX | if needed | level shifter / aux | only if peripherals need it |

### PDN rules

- 24 V input with hot-swap, reverse-polarity protection, TVS, and pi-filter.
- SoM on its own clean 5 V island with bulk buffer near module.
- Follow the SoM vendor's exact rail sequencing and PMIC guidance; Toradex/Verdin modules have defined power-up timing requirements.
- Each field port with local current limit or fuse.
- Current sense on 24V_IN, 5V_MAIN, and at least one field bank.
- Supervisor MCU monitors UV/OV, inrush fault, thermal fault, and port overcurrent.
- Distribute rails star-style from the power center, not as a daisy chain across the board.
- Do not share noisy field return currents with CPU return paths.

### Sizing proposal

- 24V_IN budget: **2 A continuous** as a starting point
- 5V_MAIN: **4–5 A**
- 24V_FIELD: depending on port configuration **0.5–1 A per bank**
- 3V3_LOGIC: **1–2 A**
- Reserve margin: **30 % electrical**, **20 % thermal**

---

## 7) Grounding and Shielding Strategy

There should be three reference systems:

1. **DGND / logic GND** — SoM, MCU, TPM, internal logic.
2. **FGND / field-isolated GND** — Isolated RS-485/CAN and optionally DIO subsections.
3. **CHASSIS / PE** — Shields, connector shells, surge diversion.

### Rules

- Cable shield always to **CHASSIS** first, not directly to DGND.
- CHASSIS to DGND via defined RC/HF coupling, not a hard low-impedance short everywhere.
- Isolation barriers clearly separated with adequate creepage/clearance.
- Route TVS return current to the appropriate reference; no uncontrolled jump to DGND.
- Do not run LED/debug/service GND through field current paths.
- Separate chassis-coupled connector region from quiet digital ground region with a controlled stitching strategy.

---

## 8) Thermal Analysis and Heat Removal

### Rough loss estimate for Freeze A

| Block | Rough loss |
|---|---:|
| SoM | 6…10 W |
| Power front end + buck | 1.5…3 W |
| netX 90 + Ethernet PHYs | 1…2 W |
| IO-Link (4 ports, no load) | 1…2 W |
| RS-485 / CAN / DIO logic | 1…2 W |
| Load-dependent DO losses | highly variable, 0…6 W |

### Thermal concept

- Couple SoM with **heat spreader** to enclosure or metal carrier.
- Dense thermal vias under power ICs, IO-Link drivers, netX 90, PHYs, and high-side switches.
- Large copper areas on L1/L8 plus internal copper tie-in.
- Keep high-loss field output drivers away from the CPU and RTC/TPM zone.
- Do not place field connectors directly next to SoM heat source.
- If DIN-rail enclosure is closed: support convection vertically.

### Thermal validation

- CFD-light/simulation estimate before layout freeze.
- Measurement at 24 V, full load, 50 °C ambient.
- Worst case: all DIO high-side channels active + real-time Ethernet traffic + IO-Link telemetry.
- Hot-spot limit internally < 105 °C for passive magnetics/ESD clusters, < 125 °C junction for active power parts.

---

## 9) EMI/EMC Compliance

The documents do not name specific standards. For an industrial gateway I would therefore target:

- **EN 55032 / CISPR 32 Class A**
- **EN 61000-6-2** (industrial immunity)
- **IEC 61000-4-2** ESD
- **IEC 61000-4-4** EFT
- **IEC 61000-4-5** surge
- **IEC 61000-4-6** conducted RF immunity

### Design measures

- TVS on all external lines.
- Common-mode chokes where pre-compliance justifies them.
- Switching regulators with small hot loop, shield GND return, and snubber footprint.
- Separated field/logic zones.
- Tie enclosure/shield concept to mechanics early.
- Clock sources short and cleanly referenced.
- Optional ferrite beads at domain transitions, but not as a substitute for a clean PDN topology.
- Add optional footprints for common-mode chokes, RC snubbers, and split termination tuning where compliance risk is high.

### EMC test plan

- First **pre-compliance** on bare board and in target enclosure.
- Then targeted tuning for CMC, RC snubber, shield bonding.
- Critical: Ethernet emissions (northbound GbE and real-time Ethernet ports), 24 V input conducted, IO-Link/DIO burst robustness.

---

## 10) DFM/DFA and Manufacturing Notes

### DFM

- Target: **IPC Class 2** for series product; Class 3 only if customer/industry requires.
- Min design rule as starting point: **6/6 mil**, laser vias only if SoM carrier fanout demands.
- Avoid via-in-pad unless the SoM vendor requires it.
- 1.6 mm board, defined copper balance, warpage control.
- Fiducials global + local.
- Test coupon for impedance.
- Optimize solder mask reliefs for fine QFNs.

### DFA

- Connectors only on outer edge.
- Make service/recovery connections reachable without dismounting the full system.
- LEDs readable from the front.
- Consider install orientation for DIN rail and panel together.
- Screw terminals or M12 only with enough tool clearance.

### Fabrication notes

- Conformal coating optional; for pollution degree 2 usually not mandatory, but customer-specific open item.
- No underfill without real need.
- Avoid selective soldering steps unless mechanically mandatory.
- Provide alternate footprints for all protection parts where EMC risk is high.

---

## 11) Test Points, Debugging, and Validation

### Mandatory test points

- 24V_IN, 24V_FIELD, 5V_MAIN, 3V3_LOGIC, all isolated 5 V rails
- Reset, boot mode, watchdog kick, fault summary
- RS-485 A/B per channel
- CANH/CANL
- IO-Link C/Q + L+ per port
- DIO bank inputs/outputs
- UART console, SWD/JTAG, TPM SPI header optionally not populated
- SPI lines (CLK/MOSI/MISO/CS) to netX 90 — reserve pads for bring-up
- Shield-to-chassis measurement point

### Debugging strategy

- Supervisor MCU as **board health monitor** with its own fault log.
- Linux console separate from application network.
- Recovery boot possible without special jig.
- Per-port enable/disable in software + hard mute path.
- Current sense per bank for fast fault localization.

### Validation plan

**EVT**
- Power-up, brownout, inrush, basic ESD tests
- Linux boot, SoM health, supervisor handshake
- Port enumeration and basic communication
- SPI bring-up to netX 90 (even if DNP in Phase 1, verify footprint and signal integrity)

**DVT**
- IO-Link master tests
- Modbus RTU long-run test
- EtherNet/IP throughput / link recovery
- Safe mode reaction chain
- Rollback/update recovery
- Security: signature/TPM/boot paths
- Phase 2: real-time Ethernet protocol stack qualification

**PVT**
- Manufacturing tests
- Temperature cycles
- 72 h burn-in
- Pre-compliance + regression

### QNC-specific acceptance criteria

- Critical fault → **safe mode ≤ 1 s**
- Restart → operational **≤ 30 s**
- REST-to-device latency on target path **≤ 50 ms**
- DDS/extension faults must not affect baseline southbound

---

## 12) Software Architecture for Protocol Gateway

This section defines the software architecture that runs on the hardware described above. Three patterns are applicable at different stages of the product lifecycle; the recommended evolution path moves through them in sequence.

### 12.1 Dual-processor partitioning and software domains

The hardware architecture (main CPU SoM + netX 90 co-processor + supervisor MCU) maps directly onto three software execution domains:

| Domain | Runs on | Responsibilities |
|---|---|---|
| Application / API domain | Main CPU SoM (Linux) | REST/WebSocket, logging, configuration, OTA, protocol stacks for Modbus RTU / CANopen / IO-Link in Phase 1 |
| Real-time Ethernet domain | netX 90 | EtherCAT / PROFINET / EtherNet/IP cyclic process image, deterministic field bus execution (Phase 2) |
| Supervisor domain | STM32G474 | Power sequencing, watchdog, health monitoring, fault capture, safe mode triggering |

### 12.2 Pattern 1 — Protocol Converter (Phase 1 baseline)

#### Core concept
A point-to-point translation model: data enters on one interface, is mapped through a rule table, and exits on another. This is the appropriate pattern for the Phase 1 software baseline where the main CPU handles all protocol adaptation.

#### Data flow

```
┌─────────┐      ┌─────────────────┐      ┌──────────┐
│ CANopen │◄────│  Protocol        │────►│ Modbus   │
│ Devices │      │  Converter       │      │ Devices  │
└─────────┘      └────────┬────────┘      └──────────┘
                          │
                     ┌────▼────┐
                     │ IO-Link │
                     │ Devices │
                     └─────────┘
```

Typical flows:
- CANopen PDO → internal format → northbound REST/WebSocket
- Modbus RTU register → internal format → IO-Link process data
- Emergency stop / safety signals bypass the converter (direct passthrough)

#### Internal data model

No full process image is maintained — only a conversion rule table:

```c
typedef struct {
    uint32_t source_protocol;   // 0=CANopen, 1=Modbus, 2=IO-Link
    uint32_t source_addr;
    uint32_t target_protocol;
    uint32_t target_addr;
    uint8_t  direction;         // 0=unidirectional, 1=bidirectional
    uint8_t  scaling_type;      // linear / lookup / custom
    float    scale_factor;
    float    offset;
    uint32_t deadband;          // only forward if change exceeds this value
    uint32_t last_value;
    uint32_t last_timestamp;
} ConversionRule;
```

#### Task structure (RTOS / Linux scheduling)

| Task | Priority | Period | Responsibility |
|---|---|---|---|
| CANopen RX | High | 1 ms | Receive PDOs, push to raw data queue |
| Modbus RTU RX | High | 1 ms | Receive register data, push to raw data queue |
| IO-Link RX | High | 1 ms | Receive process data, push to raw data queue |
| Conversion engine | Medium | 2 ms | Poll queues, apply conversion rules, generate output commands |
| Northbound TX | Medium | Async | Publish data via REST/WebSocket |
| Config / diagnostics | Low | 100 ms | Rule configuration, diagnostic output |

#### Rule management (double-buffering)

```c
struct {
    ConversionRule rules[MAX_RULES];
    uint32_t version;
    uint32_t crc32;
} rule_set_active, rule_set_shadow;

// Atomic rule update:
memcpy(&rule_set_shadow, &rule_set_active, sizeof(rule_set_active));
// ... modify rule_set_shadow.rules[] ...
rule_set_shadow.version++;
rule_set_shadow.crc32 = calc_crc(&rule_set_shadow);
active_rules = &rule_set_shadow;   // atomic pointer swap
```

#### Deadband filtering

```c
bool should_forward(uint32_t new_value, ConversionRule* rule) {
    uint32_t delta = abs((int32_t)new_value - (int32_t)rule->last_value);
    if (delta > rule->deadband) {
        rule->last_value = new_value;
        return true;
    }
    return false;
}
```

#### Applicability

Best fit for Phase 1 where the protocol set is known and bounded (Modbus RTU, CANopen, IO-Link, discrete I/O). Rule count stays manageable at this scale. The limitation — rule explosion as protocols multiply — is addressed in Phase 2 by migrating to the Data Bus pattern below.

---

### 12.3 Pattern 2 — Data Bus with Process Image (Phase 2 target)

#### Core concept
A publish-subscribe architecture where all protocol stacks read and write a shared **process image**. Adding a new protocol requires only one new adapter; the image is the single source of truth for all field data.

This pattern is the natural target for Phase 2 when the netX 90 is populated, because the co-processor already maintains its own cyclic process image that must be synchronized with the Linux-side application state.

#### Data flow

```
                ┌──────────────────────────────────────┐
                │           Process Image              │
                │  ┌───────────┐ ┌───────────┐         │
                │  │Field Data │ │Diagnostics│  ...    │
                │  └─────▲─────┘ └─────▲─────┘         │
                └────────┼─────────────┼───────────────┘
                         │             │
     ┌──────────┐   ┌────┴─────────────┴────┐   ┌──────────────┐
     │ CANopen  │   │     Data Bus Core     │   │  netX 90     │
     │ Modbus   │◄─►│  • Read/write arb.   │◄─►│  real-time   │
     │ IO-Link  │   │  • QoS management    │   │  Ethernet    │
     └──────────┘   │  • Quality stamps    │   └──────────────┘
                    │  • Persistence (opt) │
     ┌──────────┐   └──────────────────────┘   ┌──────────────┐
     │REST /    │◄─────────────────────────────►│Supervisor   │
     │WebSocket │                               │MCU health   │
     └──────────┘                               └──────────────┘
```

#### Data point model

```c
typedef struct {
    uint32_t id;
    char     name[64];          // e.g. "gateway.port1.process_data"
    uint8_t  type;              // INT8/UINT16/FLOAT32/STRING
    uint8_t  access;            // RO/RW/RW_EXCLUSIVE
    uint32_t size;
    void*    storage;
    uint32_t timestamp_ns;      // monotonic clock
    uint8_t  quality;           // 0=BAD, 1=UNCERTAIN, 2=GOOD
    uint8_t  protocol_source;
    uint32_t subscriber_mask;
} DataPoint;

typedef struct {
    DataPoint*        points;
    uint32_t          point_count;
    SemaphoreHandle_t rw_lock;
    uint32_t          global_version;
} ProcessImage;
```

#### Task structure

| Task | Priority | Period | Responsibility |
|---|---|---|---|
| Field publishers (per protocol) | High | 1 ms | Read field interface → write process image |
| Field subscribers (per protocol) | Medium | 2 ms | Read process image → write field interface |
| netX 90 sync task | High | 1 ms | Exchange process image with co-processor over SPI |
| QoS monitor | Medium | 10 ms | Check update rates, mark STALE/UNCERTAIN/BAD |
| Northbound publisher | Medium | Async | REST/WebSocket from process image |
| Persistence task | Low | 1000 ms | Save critical points to non-volatile storage |
| Diagnostics task | Low | 100 ms | Publish diagnostic data points |

#### Data quality state machine

```
   ┌─────┐  data received regularly  ┌──────┐
   │ BAD │ ─────────────────────────►│ GOOD │
   └──┬──┘                           └──┬───┘
      │                                 │
      │ init / comms lost               │ deadline exceeded,
      │                                 ▼  no new data
      │                           ┌───────────┐
      │                           │ UNCERTAIN │──── degraded data
      │                           └───────────┘     received
      └─────────────────────────────────┘
              manual clear / reconnect
```

The quality field maps directly onto the QNC fault model: UNCERTAIN triggers a diagnostic event; BAD triggers safe mode evaluation in the supervisor domain.

#### QoS policy

```c
typedef enum {
    QOS_BEST_EFFORT,   // deliver if possible
    QOS_RELIABLE,      // confirmed delivery via protocol
    QOS_PERIODIC,      // must update within deadline or degrade quality
    QOS_EXACT_ONCE     // deduplicated delivery
} QoSLevel;

void qos_monitor_task(void* arg) {
    while (1) {
        for each point in process_image {
            if (point.qos == QOS_PERIODIC) {
                uint32_t age = get_monotonic_ns() - point.timestamp_ns;
                if (age > point.expected_period_ns * 3)  point.quality = UNCERTAIN;
                if (age > point.expected_period_ns * 10) point.quality = BAD;
            }
        }
        vTaskDelay(pdMS_TO_TICKS(10));
    }
}
```

The QoS deadline mechanism is the software-side implementation of the hardware fault detection chain: stale field data → quality degrades → supervisor MCU receives health signal → safe mode if threshold exceeded.

#### Read/write locking

```c
void write_data_point(uint32_t id, void* value, uint8_t quality) {
    DataPoint* pt = &process_image.points[id];
    xSemaphoreTake(pt->write_lock, portMAX_DELAY);
    memcpy(pt->storage, value, pt->size);
    pt->timestamp_ns = get_monotonic_ns();
    pt->quality = quality;
    pt->version++;
    xSemaphoreGive(pt->write_lock);
}
```

---

### 12.4 Pattern 3 — Hybrid / Multi-Channel Routing (optional Phase 3)

For applications requiring simultaneous real-time control, high-volume data streaming, and configuration management, a three-channel routing architecture can be layered onto the Data Bus foundation.

#### Channel structure

| Channel | Data type | Path | Latency target |
|---|---|---|---|
| Ch1 — Realtime bus | Cyclic field data (I/O states, process values) | Process image + QoS | 1–2 ms |
| Ch2 — High-speed stream | High-volume aperiodic data (e.g. diagnostics burst) | Zero-copy ring buffer | < 500 µs |
| Ch3 — Config/diagnostics | Parameter reads/writes, event logs | Converter / REST | Not latency-critical |

#### Routing decision

```c
typedef struct {
    uint32_t signal_id;
    uint8_t  max_latency_us;
    uint8_t  route_type;       // 0=realtime image, 1=high-speed stream, 2=converter
    uint8_t  fallback_route;
} RoutingRule;
```

#### Exception bypass (hardware fault isolation in software)

The bypass mechanism is the direct software counterpart to the hardware fault isolation described in Section 2. When the process image becomes stale, critical signals are routed directly through the converter (Ch3) while recovery is attempted:

```c
void health_monitor_task(void* arg) {
    uint32_t last_good_version = 0;
    uint32_t stale_counter = 0;
    while (1) {
        if (realtime_image.version == last_good_version) {
            if (++stale_counter > 10) {  // 10 ms without update
                // Software safe mode: force critical signals to fallback path
                routing_table[CTRLWORD_SIGNAL].route_type = 2;
                start_recovery_timer(5000);
            }
        } else {
            stale_counter = 0;
            last_good_version = realtime_image.version;
        }
        vTaskDelay(pdMS_TO_TICKS(1));
    }
}
```

This ensures the ≤ 1 s safe mode transition requirement (Section 11) is achievable in software without depending exclusively on hardware watchdogs.

---

### 12.5 Architecture comparison and phased evolution

| Dimension | Protocol Converter | Data Bus | Hybrid |
|---|---|---|---|
| Core data flow | Point-to-point mapping | Publish-subscribe | Dynamic routing |
| Data model complexity | Low (rule table) | High (full image) | Very high (layered + rules) |
| RAM requirement | ~256–512 KB | ~1–4 MB | ~2–8 MB |
| Minimum latency | 100–500 µs | 200–1000 µs | 50–500 µs (tunable) |
| Determinism | High (fixed conversion delay) | Medium (lock contention) | High (critical path fixed) |
| Scalability | Poor (new rule per device pair) | Good (one adapter per protocol) | Excellent (auto-routing) |
| Development effort | 2–4 months | 4–8 months | 8–12 months |
| Deployment phase | Phase 1 | Phase 2 | Phase 3 (optional) |

#### Recommended evolution path

```
Phase 1 (months 1–2)
    └── Protocol Converter baseline
        ├── CANopen / Modbus RTU / IO-Link ↔ internal rule table
        ├── REST/WebSocket northbound from conversion output
        └── Validate forwarding correctness and latency budget

Phase 2 (months 3–6)
    └── Data Bus architecture
        ├── Establish unified process image
        ├── Implement QoS, quality stamps, and stale detection
        ├── Integrate netX 90 process image sync over SPI
        ├── Add EtherNet/IP / PROFINET / EtherCAT adapters
        └── Stress test (target: 1000 data points at 1 kHz)

Phase 3 (months 7–10, optional)
    └── Hybrid routing layer
        ├── High-speed diagnostic stream channel (zero-copy)
        ├── Dynamic routing rules with fallback
        ├── Exception bypass tied to hardware safe mode
        └── Performance tuning
```

**If only one pattern can be deployed initially, start with Pattern 2 (Data Bus)**, because:
- The Protocol Converter suffers from rule explosion as the protocol and device count grows.
- Once the Data Bus backbone is in place, adding a new protocol (including the netX 90 real-time Ethernet stack) requires only one new adapter.
- Hybrid features can be layered onto the Data Bus incrementally without a full redesign.

---

## 13) PCB Partitioning (Updated)

Recommended schematic / layout partitioning (updated for dual-processor architecture):

1. `01_PWR_ENTRY_PROTECTION`
2. `02_MAIN_CPU_SOM`
3. `03_SUPERVISOR_MCU`
4. `04_ETH_COPROCESSOR_NETX90`
5. `05_REALTIME_ETH_PHY_PORTS`
6. `06_NORTHBOUND_ETHERNET`
7. `07_RS485_RS232`
8. `08_IOLINK_MASTER`
9. `09_CAN_CANOPEN`
10. `10_DIGITAL_IO`
11. `11_SECURITY_NVM_SERVICE`
12. `12_TEST_AND_PRODUCTION`

### Key placement rule

Place the **main CPU SoM**, **netX 90**, and **northbound Ethernet** in the central/high-speed zone. Put **field transceivers and field connectors** on outer edges. Keep **isolation barriers** physically obvious. Do not run high-speed digital traces across isolation boundaries.

---

## 14) Interface and Phase Coverage Summary

| Interface | Hardware implementation | SW Phase 1 | SW Phase 2 |
|---|---|---|---|
| CAN / CANopen | Isolated CAN transceiver (ISO1042BDW) | Protocol Converter | Data Bus adapter |
| RS-485 / Modbus RTU | Isolated RS-485 (ISO1410DWR) | Protocol Converter | Data Bus adapter |
| IO-Link | 4-port master (MAX14819ATM+ ×2) | Protocol Converter | Data Bus adapter |
| Industrial Ethernet | netX 90 + dual DP83822 PHY | PCB-ready, DNP | Data Bus + real-time sync |
| Digital I/O | ISO1212DBQR (DI) + TPS272C45ARHFR (DO) | Protocol Converter | Data Bus adapter |
| Northbound REST / WS | CPU SoM (Linux) | Converter output | Process image subscriber |
| FastDDS (optional) | CPU SoM (Linux) | Not enabled | Process image subscriber |

---

## 15) Risks, Trade-offs, and Still-Missing Information

### Largest project risks

| Risk | Impact | Recommendation |
|---|---|---|
| Missing connector/pinout sign-off | PCB cannot be finalized | Follow up mechanics + ICD hardware annex |
| Unclear input voltage / current budget | Power design possibly wrong size | Sign off 24 V system budget in writing |
| DIO "sourcing/sinking" not precisely defined | Wrong output topology | Decide per-bank or per-channel configuration immediately |
| Unclear isolation requirement | Safety/EMC/cost risk | Define minimum working voltage + HiPot target |
| Unclear port count | Area and thermal risk | Define product variant matrix |
| Unclear connector standard (RJ45 vs M12, terminal vs circular) | Rework of mechanical/EMC design | Fix early |
| FastDDS port sharing unclear | Network topology unclear | Decide dedicated port vs VLAN vs shared NIC |
| SoM BSP and SPI availability not yet confirmed | netX 90 integration may be blocked | Confirm exported SPI, Linux BSP, and industrial temp grade before PCB freeze |
| Software architecture pattern not selected | Integration planning risk | Commit to Phase 1 pattern (Converter) immediately; plan Phase 2 migration |

### Important trade-offs

- **SoM vs. discrete MPU** — SoM is faster, lower risk, more expensive per piece; discrete is cheaper at volume but much riskier in NRE and SI.
- **Full isolation vs. partial isolation** — more isolation improves robustness/fault containment but costs area, BOM, and heat.
- **2 Ethernet vs. 3 Ethernet ports** — 2 ports usually suffice for baseline; 3 ports decouple DDS/service/field better but increase cost.
- **4 IO-Link vs. 8 IO-Link ports** — 4 ports are realistic for Freeze A; 8 ports drive current, heat, and PCB area strongly upward.
- **RJ45 vs. M12** — RJ45 cheaper and lab-friendly; M12 more robust industrially.
- **SPI vs. PCIe for netX 90** — SPI is recommended for the first carrier revision (lower complexity, adequate bandwidth for Phase 2); PCIe is reserved if process-image size or update bandwidth later exceeds SPI margins.
- **Protocol Converter vs. Data Bus as Phase 1 starting point** — Converter is simpler to implement initially but does not scale; Data Bus has higher upfront cost but is the correct long-term foundation.

### Missing information for finalization

Before the board can be signed off, at least these items are still missing:

1. Exact **supply specification**: nominal/min/max, brownout, transients
2. Final **connector families** and **pinouts**
3. Target **port count** per product variant
4. Required **insulation withstand** and creepage/clearance targets
5. Mechanical data: board dimensions, mounting, enclosure, IP target
6. Regulatory targets: CE/FCC/UL/UKCA, industrial EMC standards
7. DIO electrical: levels, loads, sourcing/sinking concept, protection class
8. Ethernet topology: dedicated ports for northbound/southbound/DDS yes or no
9. Service concept: local USB service, Ethernet only, or both
10. Manufacturing target: IPC class, coating, traceability, series volume
11. **SoM vendor confirmation**: exported SPI for netX 90, northbound Ethernet path, Linux BSP support window, industrial temperature grade, long-term supply commitment
12. **Software architecture decision**: which gateway pattern to deploy in Phase 1, and timeline for Phase 2 migration

---

## 16) Clear Recommendation

If I had to take this project into production, I would set the next steps as follows:

**Hardware**
- Confirm SoM-based carrier board with Toradex Verdin iMX8M Plus as the current preferred candidate
- Close open points: supply specification, connectors, isolation target, DIO topology
- Create Design Freeze A: 8-layer carrier, 2× GbE, 2× RS-485, 4× IO-Link, 8×DI/8×DO, CAN, netX 90 footprint DNP
- Lay out EVT board with EMC/thermal margin
- DVT with final enclosure and pre-compliance

**Software**
- Phase 1: deploy Protocol Converter pattern on Linux main CPU; validate all baseline field interfaces (Modbus RTU, CANopen, IO-Link, DIO) against the ≤ 50 ms latency and ≤ 1 s safe mode NFRs
- Phase 2: migrate to Data Bus pattern; integrate netX 90 process image sync via SPI; qualify real-time Ethernet stack (EtherNet/IP, PROFINET, or EtherCAT)
- Phase 3 (optional): add hybrid routing and zero-copy diagnostic stream if high-volume data use cases materialize

**Gate condition before PCB freeze**: SoM vendor must confirm exported SPI host port for netX 90, northbound Ethernet path, Linux BSP support window, industrial temperature grade, and long-term supply commitment.
