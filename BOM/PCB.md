






## 1) System interpretation and design assumptions

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

## 2) Schematic partitioning and hierarchy

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

This partitioning reflects the separation required in the documents between physical/protocol adaptation, normalization, control, diagnostics, and service—mapped cleanly at the hardware level.  

### Functional blocks at board level

- **Compute domain**: Linux/QNC runtime, REST/WebSocket, logging, DDS optional.
- **Supervisor domain**: deterministic board/power/fault supervision.
- **Fieldbus domain**: Modbus RTU, CANopen optional, IO-Link, DIO.
- **Service domain**: update, diagnostics, recovery.
- **Power domain**: separate rails for logic, interface, isolated field side.

Fault containment is thus physically supported as well: an RS-485 or IO-Link fault must not disturb the compute domain or other ports; that is explicitly built into the QNC fault model.  

---

## 3) Detailed BOM proposal

Important: this is a **recommended reference BOM** to implement the documented function. It is **not** specified in the QNC documents. Connectors, some power stages, and the final DIO topology remain open until interface sign-off.

### Critical active parts

| Block | Recommended part | Function | Key specs | Sourcing note |
|---|---|---|---|---|
| Compute SoM | **Toradex Verdin iMX8M Plus Quad 8GB WB IT** | Linux/edge/DDS/REST/WebSocket | Industrial SoM, reduced DDR risk | Check manufacturer/distributor, prefer long-term availability [Toradex](https://www.toradex.com/computer-on-modules/verdin-arm-family/nxp-imx-8m-plus?srsltid=AfmBOoqKgcokkZBOHE2eLQYVikifUyZdY12uMnYiCEbgnh8hNAinrX0U) |
| Alternative SoM | **UCM-iMX8M-Plus** | Alternative compute platform | i.MX 8M Plus SoM | Qualify only as backup source [CompuLab](https://www.compulab.com/products/computer-on-modules/ucm-imx8m-plus-nxp-i-mx-8m-plus-som-system-on-module-computer/) |
| Supervisor MCU | **STM32G474RET6** | Power/reset/watchdog/health supervisor | Industrial grade, enough ADC/timer/I/O | No direct second source; check lifecycle |
| TPM | **OPTIGA-TPM-SLB-9672-FW15** | Secure boot / keys / attestation | SPI TPM 2.0 | Version security BOM separately [Infineon](https://www.infineon.com/part/OPTIGA-TPM-SLB-9672-FW15) |
| IO-Link master | **MAX14819ATM+** ×2 | 4 IO-Link master ports (2 chips × 2 ports) | Integrated framer/L+ control | Critical long-life part; stock early [ADI](https://www.analog.com/en/products/max14819.html) |
| RS-485 | **ISO1410DWR** ×2 | 2 isolated Modbus RTU ports | 5 kVrms isolation, robust EMC | Good choice for industrial front end [TI](https://www.ti.com/product/ISO1410) |
| CAN (extension) | **TCAN1042HVDRQ1** | CAN/CANopen-ready | Fault-protected CAN transceiver | Populate only if extension is approved [TI](https://www.ti.com/product/TCAN1042HV-Q1/part-details/TCAN1042HVDRQ1) |
| Digital input | **ISO1212DBQR** ×4 | 8 isolated 24 V DI channels | 2 channels/IC, industrial input receiver | IEC 61131-like input topology [TI](https://www.ti.com/product/ISO1212) |
| Digital output (sourcing) | **TPS272C45CRHFR** ×4 | 8 high-side DO channels | 36 V, 3 A, smart high-side | Only if sourcing is finally approved [TI](https://www.ti.com/product/TPS272C45) |
| DIO alternative | **MAX14900EAGM+CKT** ×2 | Flexible DIO topology | Per-channel flexible, industrial DO/DIO | Check whether logic model fits ICD [ADI](https://www.analog.com/en/products/max14900e.html) |
| Hot-swap/inrush | **LM5069MM-2/NOPB** | 24 V input protection | Hot-swap / inrush / fault cutoff | Required for robust industrial power-in [TI DS](https://www.ti.com/lit/ds/symlink/lm5069.pdf) |
| Main buck | **LM76005RNPR** | 24 V → 5 V main supply | 60 V input tolerant, 5 A | Good robustness on 24 V bus [TI](https://www.ti.com/product/LM76005/part-details/LM76005RNPR) |
| Isolated DC/DC | **NXJ2S0505MC-R13** ×2…4 | Isolated auxiliary supplies for field side | 5 V / 2 W isolated | Count per channel grouping [Murata](https://pim.murata.com/en-us/pim/details/?partNum=NXJ2S0505MC) |
| RTC | **RV-3028-C7** | Timestamps / audit / recovery | Ultra-low-power RTC | Optional but recommended |
| ESD array | **TPD4E05U06QDRQ1** | ESD protection for service/data lines | Low-capacitance ESD | Ethernet needs a separate protection strategy |
| TVS 24 V input | **SMBJ33A** | Surge/transient clamp | 600 W class | Select vendor by surge level |
| Common-mode chokes | Würth / TDK industrial families | Line emissions | Differential per interface | Tune in pre-compliance |
| Ethernet connector | **TBD by enclosure** | RJ45 shielded or M12 X-coded | Mechanically open | Not finalizable without mechanical sign-off |
| Field connectors | **TBD by mechanics** | IO-Link / RS-485 / DIO / CAN | Screw terminal or M8/M12 | Depends on mechanics and IP |

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

- **A-parts**: SoM, IO-Link, isolated RS-485, TPM, power front end—secure LTB/LTS capability early.
- **B-parts**: Protection devices, isolation modules, DIO drivers—dual source.
- **C-parts**: Standardize passives to 0402/0603/0805, allow manufacturer pool.
- **No-go**: NCNR parts without lifecycle approval, unauthorized brokers for security parts.

---

## 4) PCB stack-up

Because the compute side is implemented as an SoM, a **6-layer stack-up** with controlled impedance is sufficient for the carrier board. That is the best compromise of cost, manufacturability, and SI/EMI margin.

### Recommended stack-up

| Layer | Function | Copper | Note |
|---|---|---:|---|
| L1 | Components + critical signals | 35 µm | Short service/mgmt/diff pairs |
| L2 | Solid GND plane | 35 µm | Reference for L1/L3 |
| L3 | Power + slow signals | 18 µm | 24 V / 5 V / 3.3 V islands |
| L4 | Controlled signals / diff pairs | 18 µm | Ethernet/USB/MCU interconnect |
| L5 | Solid GND / chassis coupling | 35 µm | Shield and return current routing |
| L6 | Components + field signals | 35 µm | Transceivers, connectors, protection |

### Material and manufacturing target

- **FR-4, Tg ≥ 170 °C**
- Total thickness **1.6 mm**
- Outer layers **1 oz**, inner layers **0.5 oz**  
- Controlled impedance coupons:
  - 50 Ω single-ended
  - 90 Ω diff for USB 2.0 if used
  - 100 Ω diff for Ethernet/high-speed pairs
  - 120 Ω nominal for CAN/RS-485 line model on board not strictly required, but controlled symmetry is sensible

This fits an industrial gateway with limited but present high-speed content and supports clean return paths plus EMC headroom.  

---

## 5) High-speed and sensitive routing guidelines

### General routing rules

- No split-GND return paths under high-speed signals.
- Shield/chassis bonding at the connector, **not** deep inside the board.
- Field interfaces at the edge; compute/TPM/service toward the center.
- Isolation barriers visibly executed as keep-out + creepage channel.
- Every external line first through **ESD/surge/filter**, then to the IC.

### Ethernet

- If the SoM integrates PHYs: MDI pairs short, symmetric, no stubs to the magnetics module.
- 100 Ω diff, small pair length skew, same layer preferred.
- No sharp 90° corners; 45°/arc.
- Bob Smith / shield net exactly per magnetics/connector app note.
- Chassis shield near port with 360° shield bond in the enclosure concept.

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

These rules follow directly from the documented need for fault isolation, diagnosability, update security, and robust field interfaces.  

---

## 6) PDN design

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
- Each field port with local current limit or fuse.
- Current sense on 24V_IN, 5V_MAIN, and at least one field bank.
- Supervisor MCU monitors UV/OV, inrush fault, thermal fault, and port overcurrent.
- Distribute rails star-style from the power center, not as a daisy chain across the board.

### Sizing proposal

- 24V_IN budget: **2 A continuous** as a starting point
- 5V_MAIN: **4–5 A**
- 24V_FIELD: depending on port configuration **0.5–1 A per bank**
- 3V3_LOGIC: **1–2 A**
- Reserve margin: **30 % electrical**, **20 % thermal**

This supports the documented goals for recovery, fault handling, diagnostics, and multi-device operation.  

---

## 7) Grounding and shielding strategy

There should be three reference systems:

1. **DGND / logic GND**  
   SoM, MCU, TPM, internal logic.

2. **FGND / field-isolated GND**  
   Isolated RS-485/CAN and optionally DIO subsections.

3. **CHASSIS / PE**  
   Shields, connector shells, surge diversion.

### Rules

- Cable shield always to **CHASSIS** first, not directly to DGND.
- CHASSIS to DGND via defined RC/HF coupling, not a hard low-impedance short everywhere.
- Isolation barriers clearly separated with adequate creepage/clearance.
- Route TVS return current to the appropriate reference; no uncontrolled jump to DGND.
- Do not run LED/debug/service GND through field current paths.

This effectively supports the fault containment required in the documents against conducted disturbances as well.  

---

## 8) Thermal analysis and heat removal

### Rough loss estimate for Freeze A

| Block | Rough loss |
|---|---:|
| SoM | 6…10 W |
| Power front end + buck | 1.5…3 W |
| IO-Link (4 ports, no load) | 1…2 W |
| RS-485 / CAN / DIO logic | 1…2 W |
| Load-dependent DO losses | highly variable, 0…6 W |

### Thermal concept

- Couple SoM with **heat spreader** to enclosure or metal carrier.
- Dense thermal vias under power ICs, IO-Link drivers, and high-side switches.
- Large copper areas on L1/L6 plus internal copper tie-in.
- Do not place field connectors directly next to SoM heat source.
- If DIN-rail enclosure is closed: support convection vertically.

### Thermal validation

- CFD-light/simulation estimate before layout freeze.
- Measurement at 24 V, full load, 50 °C ambient.
- Hot-spot limit internally < 105 °C for passive magnetics/ESD clusters, < 125 °C junction for active power parts.
- Worst case: all DIO high-side channels active + DDS/Ethernet traffic + IO-Link telemetry.

The 0…50 °C temperature range is stated in the QNC materials; the board should be designed with at least 10 °C margin. 

---

## 9) EMI/EMC compliance

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

### EMC test plan

- First **pre-compliance** on bare board and in target enclosure.
- Then targeted tuning for CMC, RC snubber, shield bonding.
- Critical: Ethernet emissions, 24 V input conducted, IO-Link/DIO burst robustness.

Because QNC demands secure communication, diagnostics, and availability, an EMC-resilient front end is not optional but central.  

---

## 10) DFM/DFA and manufacturing notes

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

The documents do not prescribe manufacturing standards; these points are a pragmatic industrial DFM baseline.  

---

## 11) Test points, debugging, and validation

### Mandatory test points

- 24V_IN, 24V_FIELD, 5V_MAIN, 3V3_LOGIC, all isolated 5 V rails
- Reset, boot mode, watchdog kick, fault summary
- RS-485 A/B per channel
- CANH/CANL
- IO-Link C/Q + L+ per port
- DIO bank inputs/outputs
- UART console, SWD/JTAG, TPM SPI header optionally not populated
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

**DVT**
- IO-Link master tests
- Modbus RTU long-run test
- EtherNet/IP throughput / link recovery
- Safe mode reaction chain
- Rollback/update recovery
- Security: signature/TPM/boot paths

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

These criteria follow directly from the documented NFR/fault/lifecycle requirements.   

---

## 12) Risks, trade-offs, and still-missing information

### Largest project risks

| Risk | Impact | Recommendation |
|---|---|---|
| Missing connector/pinout sign-off | PCB cannot be finalized | Follow up mechanics + ICD hardware annex |
| Unclear input voltage / current budget | Power design possibly wrong size | Sign off 24 V system budget in writing |
| DIO “sourcing/sinking” not precisely defined | Wrong output topology | Decide per-bank or per-channel configuration immediately |
| Unclear isolation requirement | Safety/EMC/cost risk | Define minimum working voltage + HiPot target |
| Unclear port count | Area and thermal risk | Define product variant matrix |
| Unclear connector standard (RJ45 vs M12, terminal vs circular) | Rework of mechanical/EMC design | Fix early |
| FastDDS port sharing unclear | Network topology unclear | Decide dedicated port vs VLAN vs shared NIC |

### Important trade-offs

- **SoM vs. discrete MPU**  
  SoM is faster, lower risk, more expensive per piece; discrete is cheaper at volume but much riskier in NRE and SI.

- **Full isolation vs. partial isolation**  
  More isolation improves robustness/fault containment but costs area, BOM, and heat.

- **2 Ethernet vs. 3 Ethernet ports**  
  2 ports usually suffice for baseline; 3 ports decouple DDS/service/field better but increase cost and enclosure complexity.

- **4 IO-Link vs. 8 IO-Link ports**  
  4 ports are realistic for Freeze A; 8 ports drive current, heat, and PCB area strongly upward.

- **RJ45 vs. M12**  
  RJ45 cheaper and lab-friendly; M12 more robust industrially but more demanding mechanically and in cost.

### Missing information for finalization

Before the board can be signed off, at least these items are still missing in my view:

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

These gaps are not a side issue; they are exactly the points that the QNC documents deliberately place outside the hardware-detail scope.   

---

## 13) Clear recommendation

If I had to take this project into production, I would set the next step as follows:

- **Fix architecture**: confirm SoM-based carrier board  
- **Close open points**: supply, connectors, isolation target, DIO topology  
- **Create Design Freeze A**: 6-layer carrier, 2× GbE, 2× RS-485, 4× IO-Link, 8×DI/8×DO, optional CAN  
- **Lay out EVT board** with EMC/thermal margin  
- then **DVT** with final enclosure and pre-compliance




---
---
# Industrial Edge Gateway BOM and PCB Design Update

## 1. Architecture Update

This update replaces the earlier "single Linux compute + field interfaces" assumption with a **dual-processor architecture** that is better aligned with phased industrial protocol deployment:

- **Main CPU**: Industrial Linux-capable ARM Cortex-A platform, `>= 1 GHz`, `>= 1 GB RAM`
- **Communication co-processor**: Dedicated industrial Ethernet multi-protocol controller for deterministic real-time Ethernet
- **Supervisor MCU**: Independent low-power microcontroller for sequencing, watchdog, recovery, and health monitoring

### Recommended partitioning

- **Phase 1**
  - Main CPU handles application logic, web/API stack, logging, configuration, Modbus RTU, CANopen, IO-Link master management, and digital I/O control.
  - Industrial Ethernet co-processor section is designed into the PCB but may be `DNP` or firmware-disabled in low-cost variants.

- **Phase 2**
  - Industrial Ethernet co-processor is populated and enabled for EtherCAT, PROFINET, EtherNet/IP, or other licensed real-time Ethernet stacks.
  - Main CPU remains the system controller, UI/API gateway, data concentrator, and OTA/update endpoint.

## 2. Recommended Dual-Processor Topology

### Preferred implementation

- **Main CPU module**
  - **Preferred direction**: Qualcomm-based industrial SoM, leveraging existing company cooperation
  - Recommended performance target: Cortex-A class application processor, `>= 1 GHz`, `>= 2 GB RAM`
  - Use a vendor-supported SoM rather than a raw Qualcomm SoC to reduce PMIC, DDR, and high-speed bring-up risk

- **Industrial Ethernet co-processor**
  - `Hilscher netX 90` family, host-controlled industrial communication controller
  - Select exact orderable variant according to required protocol license bundle and memory option

- **Host interconnect**
  - **Recommended for this design**: `SPI`
  - Reason: lower PCB complexity than PCIe, broad SoM support, easier isolation of deterministic industrial Ethernet domain, and realistic integration path for a first production carrier board
  - Additional control lines:
    - `COPROC_SPI_CLK`
    - `COPROC_SPI_MOSI`
    - `COPROC_SPI_MISO`
    - `COPROC_SPI_CS_N`
    - `COPROC_IRQ_N`
    - `COPROC_RESET_N`
    - `COPROC_BOOTMODE[1:0]`
    - `COPROC_SYNC` (optional)

### SPI vs PCIe trade-off

| Option | Recommendation | Why |
|---|---|---|
| SPI | **Recommended now** | Lower risk, easier routing, available on nearly all industrial ARM SoMs, good fit for staged deployment |
| PCIe | Reserve only if needed | Better bandwidth, but more layers, tighter SI constraints, more validation effort, and not the most natural fit for netX 90 integration |

If later process-image size or cyclic update bandwidth exceeds SPI comfort margins, the roadmap should move to a higher-performance communication controller or a PCIe-capable real-time Ethernet architecture in a future board revision.

### Recommended Qualcomm SoM candidates

| Candidate | Vendor | Why it fits | Main caution |
|---|---|---|---|
| `Open-Q 6490CS SOM` | Lantronix | Strong vendor ecosystem, Qualcomm QCS6490, long-life embedded positioning, good fit for Linux edge gateway and future AI/HMI variants | Confirm industrial temp grade, carrier access, and exact Ethernet/SPI availability on exported pins |
| `TurboX C6490 SOM` | Thundercomm | Direct Qualcomm ecosystem alignment and strong platform familiarity for Qualcomm-based development | Verify long-term industrial availability and carrier-board design documentation depth |
| `SOM-SMARC-QCS6490` / `SOM-SMARC-QCS5430` | SECO | Industrial/embedded focus, SMARC form factor, easier serviceability and vendor-managed BSP path | Check exact variant temperature grade and whether the required field I/O can be mapped cleanly on the chosen SMARC carrier |

### Qualcomm selection criteria

Before freezing a Qualcomm SoM, verify:

- industrial or extended-temperature availability
- Yocto/Linux BSP maturity and update policy
- long-term supply commitment
- exported `SPI`, `UART`, `I2C`, `GPIO`, and Ethernet interfaces needed by the carrier board
- secure boot and TPM integration path
- recovery and boot-strap behavior for factory service

## 3. Updated PCB Block Diagram

```text
24V_IN / PE
    |
    v
+------------------------------------------------------+
| Power Entry, Protection, EMI, Hot-Swap, Buck Rails   |
| 24V -> 5V_MAIN / 3V3_LOGIC / isolated field rails    |
+------------------+-------------------+---------------+
                   |                   |
                   v                   v
        +------------------+   +----------------------+
        | Supervisor MCU   |   | Security / NVM / RTC |
        | Reset / WD / PM  |   | TPM / EEPROM / RTC   |
        +--------+---------+   +----------------------+
                 |
                 v
        +---------------------------+
        | Main CPU SoM              |
        | Linux / App / API / OTA   |
        | Modbus / CANopen / UI     |
        +-----+---------+-----------+
              |         |
              | SPI     | Native CPU I/O / UART / SPI / GPIO
              v         v
   +----------------+   +-----------------------------------+
   | netX 90        |   | RS-485 / optional RS-232          |
   | Ind. Ethernet  |   | IO-Link Master                    |
   | Co-processor   |   | CAN / CANopen                     |
   +---+--------+---+   | Digital Inputs / Outputs          |
       |        |       +-----------------------------------+
       |        |
       v        v
  Real-time  Real-time
  ETH PHY 1  ETH PHY 2
       |        |
       v        v
   M12/RJ45  M12/RJ45

Additional northbound Ethernet from CPU SoM:
   CPU GbE -> Magnetics -> RJ45/M12
```

## 4. Updated BOM

### 4.1 Core processing and control

| Category | Recommended Part | Qty | Rationale | Notes |
|---|---|---:|---|---|
| Main CPU SoM (preferred) | `Lantronix Open-Q 6490CS SOM` | 1 | Qualcomm QCS6490 platform, strong embedded positioning, good fit where company already has Qualcomm collaboration | Verify industrial temp option, exported interfaces, and Linux BSP terms before freeze |
| Main CPU SoM (alternative A) | `Thundercomm TurboX C6490 SOM` | 1 | Qualcomm ecosystem alignment and good candidate for cooperation-driven roadmap | Confirm lifecycle, pin accessibility, and carrier design collateral |
| Main CPU SoM (alternative B) | `SECO SOM-SMARC-QCS6490` or `SOM-SMARC-QCS5430` | 1 | Industrial SMARC route with easier modular service strategy | Good if the project wants a standards-based module footprint |
| Communication co-processor | `Hilscher netX 90` host-controlled variant | 1 | Dedicated real-time industrial Ethernet offload for Phase 2 | Exact commercial SKU depends on protocol bundle |
| Supervisor MCU | `STM32G474RET6` | 1 | Strong ADC/timer set for sequencing, health monitoring, watchdog, fault capture | Keep independent from Linux domain |
| TPM | `Infineon OPTIGA TPM SLB 9672 FW15` | 1 | Secure boot, key storage, device identity | SPI-connected to main CPU |
| RTC | `Micro Crystal RV-3028-C7` | 1 | Low-power RTC for audit trails and offline timestamp continuity | Optional but recommended |
| Boot / config EEPROM | `24AA256T-I/ST` | 1 | Stores board ID, BOM variant, calibration, manufacturing data | Simple and proven industrial support device |

### 4.2 Industrial Ethernet and network support

| Category | Recommended Part | Qty | Rationale | Notes |
|---|---|---:|---|---|
| Real-time Ethernet co-processor | `Hilscher netX 90` | 1 | Enables EtherCAT/PROFINET/EtherNet/IP migration in Phase 2 | Place on PCB from rev A, DNP if not populated |
| 100BASE-TX PHY for co-processor | `TI DP83822IFRHBR` | 2 | Industrial Ethernet PHY with wide temp support and strong EMC reputation | Good fit for real-time Ethernet ports |
| Northbound GbE port(s) | Provided by selected Qualcomm SoM carrier design | 1-2 | Use SoM-exported MAC/PHY path where available to reduce risk | Confirm actual exposed interface on chosen module before PCB freeze |
| Ethernet magnetics / connector | `Bel Fuse / HALO / Würth industrial RJ45` or `M12 X-coded/Y-coded` | 2-4 | Connector style depends on enclosure and IP target | Use shield strategy tied to chassis near connector |
| Ethernet ESD | `Semtech RClamp0524P` or `TI TPD4E05U06-Q1` | per port | Protect cable-side differential pairs and LED/control pins | Use low-capacitance parts only |
| Common-mode choke | `Würth 7499011121A` family or equivalent | per port | Helps pre-compliance tuning for conducted/radiated emissions | DNI option may be useful during tuning |

### 4.3 CAN / CANopen

| Category | Recommended Part | Qty | Rationale | Notes |
|---|---|---:|---|---|
| Isolated CAN transceiver | `TI ISO1042BDW` | 1-2 | Robust isolated CAN FD transceiver; fully suitable for CANopen at classic CAN rates | Preferred for harsh industrial field cabling |
| Lower-cost non-isolated alternative | `TI TCAN1042HVDRQ1` | 1-2 | Good EMC and fault protection | Use only where system-level isolation is not required |
| CAN TVS | `Nexperia PESD1CAN` or `SM24CANB` | per port | ESD and surge protection on CANH/CANL | Place directly at connector |

### 4.4 RS-485 / RS-232

| Category | Recommended Part | Qty | Rationale | Notes |
|---|---|---:|---|---|
| Isolated RS-485 transceiver | `TI ISO1410DWR` | 2 | Industrial isolated Modbus RTU front-end with strong EMC performance | Preferred default |
| Optional isolated RS-232 | `Analog Devices ADM3252EABCZ` | 1 | Integrated isolated dual RS-232 for legacy commissioning/service | Optional service feature |
| RS-485 termination / bias | 120 ohm + bias network | per port | Required for stable field bus behavior | One bias source per segment only |
| RS-485 TVS | `SM712-02HTG` | per port | Specifically targeted for RS-485 line protection | Place at connector entry |

### 4.5 IO-Link

| Category | Recommended Part | Qty | Rationale | Notes |
|---|---|---:|---|---|
| IO-Link master IC | `Analog Devices MAX14819ATM+` | 2 | Proven dual-port IO-Link master with integrated framing and L+ control | Gives 4 ports total |
| Port protection / power switch | Per-port high-side protected switch, e.g. `TPS274160ARHARQ1` | 4 | Improves port fault isolation and current limiting | Use if IO-Link power budget per port is high |
| IO-Link connector | M8 or M12 A-coded | 4 | Industry-standard field connector choice | Decide with enclosure and IP target |

### 4.6 Digital I/O

| Category | Recommended Part | Qty | Rationale | Notes |
|---|---|---:|---|---|
| Digital input receiver | `TI ISO1212DBQR` | 4 | 24 V industrial digital input front-end, 2 channels per IC | Good match for 8 DI |
| Digital output driver | `TI TPS272C45ARHFR` | 4 | Smart high-side 24 V output driver, 2 channels per IC | Good fit for 8 DO sourcing outputs |
| Flexible DIO alternative | `ADI MAX14900EAGM+` | 2 | Use only if per-channel configurable DIO becomes a hard requirement | Higher integration, more firmware complexity |

### 4.7 Power, isolation, and protection

| Category | Recommended Part | Qty | Rationale | Notes |
|---|---|---:|---|---|
| Hot-swap / surge control | `TI LM5069MM-2/NOPB` | 1 | Strong 24 V industrial input protection and inrush control | Keep close to input connector |
| Main 24 V to 5 V buck | `TI LM76005RNPR` | 1 | 60 V-tolerant, robust industrial main rail converter | Good baseline for 5V_MAIN |
| 5 V to 3.3 V regulator | `TI TPS62130RGTR` or equivalent | 1-2 | Efficient secondary point-of-load regulation | Use local rails per domain where needed |
| Isolated DC/DC for field side | `Murata NXJ2S0505MC-R7` or equivalent | 2-4 | Power isolated RS-485/CAN and other field domains | Count depends on isolation partitioning |
| eFuse / protected high-side distribution | `TI TPS26624DRCR` | 2-4 | Per-bank field supply protection and fault reporting | Recommended for IO-Link and DIO banks |
| 24 V TVS | `SMBJ33A` | 1-2 | Input surge clamp | Validate against target surge standard |
| USB / service ESD | `TPD4E05U06QDRQ1` | as needed | Low-capacitance service-port protection | Keep near connector |

### 4.8 Passive and mechanical rules

| Category | Recommendation | Rationale |
|---|---|---|
| Bulk capacitors on 24 V input | Polymer + electrolytic combination | Better transient energy absorption |
| Ceramic decoupling | 100 nF + 1 uF + 10 uF staged close to each IC | Broadband decoupling strategy |
| Sense resistors | 0.1 percent / 25 ppm where current monitoring matters | Stable telemetry and fault thresholds |
| Bus resistors | 120 ohm, 1 percent for CAN/RS-485 | Stable differential bus behavior |
| Creepage / clearance | Dimension for target working voltage and pollution degree | Do not finalize without compliance target |
| Connector family | Prefer M12 for field ports in harsh environments | Higher robustness and IP compatibility than RJ45 |

## 5. Updated PCB Partitioning

Recommended schematic / layout partitioning:

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

Place the **main CPU SoM**, **netX 90**, and **northbound Ethernet** in the central/high-speed zone. Put **field transceivers and field connectors** on outer edges. Keep **isolation barriers** physically obvious and do not run high-speed digital traces across those boundaries.

## 6. PCB Design Recommendations

### 6.1 High-speed routing

#### SPI between main CPU and netX 90

- Keep SPI link on a continuous reference plane.
- Route as a tightly grouped bus with matched topology rather than exact high-speed length tuning.
- Add `22-33 ohm` series damping resistors at the driver side for `CLK`, `MOSI`, and optional `CS`.
- Keep the CPU-to-coprocessor SPI trace length ideally below `50 mm` on the carrier board.
- Route `COPROC_IRQ_N` away from noisy switching nodes.
- On Qualcomm SoM carriers, reserve additional test pads for SPI bring-up because pinmux and BSP setup often become the first integration bottleneck.

#### Northbound Gigabit Ethernet

- Use `100 ohm differential impedance`.
- Avoid layer changes unless necessary.
- Keep pair-to-pair skew controlled.
- Do not cross plane splits.
- Place ESD and common-mode choke on the connector side, magnetics as close as mechanically practical.

#### Real-time Ethernet ports from netX 90

- Prefer short PHY-to-magnetics traces.
- Keep both real-time Ethernet ports geometrically symmetric.
- Place PHY clock sources away from power magnetics and hot-switch nodes.

### 6.2 Signal integrity and EMI/EMC

- Upgrade from the earlier 6-layer recommendation to an **8-layer carrier board** for the dual-processor variant:
  - `L1`: components / critical routing
  - `L2`: solid GND
  - `L3`: high-speed signals
  - `L4`: power planes
  - `L5`: solid GND
  - `L6`: medium-speed signals / control buses
  - `L7`: field power / low-speed routing
  - `L8`: field-side components and connectors
- Keep CPU, co-processor, PHYs, and clocks over uninterrupted return planes.
- Separate chassis-coupled connector region from quiet digital ground region with a controlled stitching strategy.
- Add optional footprints for common-mode chokes, RC snubbers, and split termination tuning where compliance risk is high.

### 6.3 Power supply and isolation

- Use a **single protected 24 V input** with hot-swap, reverse protection, surge clamp, and EMI filtering.
- Generate:
  - `5V_MAIN` for SoM and central logic, or the SoM vendor's required primary rail if different
  - `3V3_LOGIC` for CPU I/O, supervisor, control logic
  - isolated 5 V or 3.3 V rails for RS-485 and CAN where isolation is required
  - protected `24V_FIELD` rails for IO-Link and digital outputs
- Do not share noisy field return currents with CPU return paths.
- Use per-bank or per-port protected high-side switching for IO-Link and DIO power domains.
- Qualcomm SoMs often have stricter rail sequencing and peak-current behavior than simpler industrial MPU modules; follow the exact module power-up timing and PMIC guidance from the module vendor.

### 6.4 Thermal management

- Budget thermal load for:
  - CPU SoM
  - netX 90
  - Ethernet PHYs
  - Buck regulators
  - IO-Link master ICs
  - High-side digital output drivers
- Provide thermal vias under power ICs and PHYs.
- Tie SoM heat spreader to chassis or metal frame where possible.
- Keep high-loss field output drivers away from the CPU and RTC/TPM zone.
- Validate worst-case ambient operation at `50 C` with all field channels active and real-time Ethernet traffic enabled.
- If a Qualcomm SoM with AI acceleration is selected, validate thermal headroom with CPU, NPU, and Ethernet load active simultaneously; do not assume gateway workloads remain CPU-only over product lifetime.

### 6.5 Industrial protection

- Protect every external interface with a connector-side protection chain:
  - ESD
  - surge / TVS
  - optional common-mode choke
  - then transceiver / PHY
- Use isolation on:
  - RS-485 by default
  - CAN when field grounding conditions are not controlled
  - optional service interfaces only where required
- Use chassis-first shield termination for Ethernet and shielded serial connectors.
- Keep creepage and clearance visible in the layout around isolated sections.

## 7. Interface Coverage Summary

| Interface | Implementation | Phase 1 | Phase 2 |
|---|---|---|---|
| CAN / CANopen | Isolated CAN transceiver + CPU stack | Yes | Yes |
| RS-485 / Modbus RTU | Isolated RS-485 ports + CPU UART | Yes | Yes |
| Optional RS-232 | Isolated service / legacy serial | Optional | Optional |
| IO-Link | 4-port IO-Link master on CPU-controlled SPI/GPIO domain | Yes | Yes |
| Industrial Ethernet | netX 90 + dual PHY + protected ports | PCB-ready, optional BOM | Enabled |
| Digital I/O | 8 DI + 8 DO industrial 24 V front-end | Yes | Yes |

## 8. Scalability Strategy

### Phase 1 BOM strategy

Populate:

- Qualcomm Main CPU SoM
- Supervisor MCU
- RS-485
- CAN
- IO-Link
- DIO
- Security and service sections

Optionally leave `DNP`:

- `Hilscher netX 90`
- real-time Ethernet PHYs
- selected real-time Ethernet magnetics/connectors

This allows a lower-cost early production variant without a PCB redesign.

### Phase 2 BOM strategy

Populate and qualify:

- `Hilscher netX 90`
- dual real-time Ethernet PHYs
- port protection and connector set
- protocol-specific firmware / licensing

## 9. Key Trade-offs

| Topic | Recommendation | Trade-off |
|---|---|---|
| Main CPU implementation | Use Qualcomm industrial/embedded SoM | Benefits from company cooperation and higher compute headroom, but BSP and lifecycle validation become vendor-dependent |
| CPU-to-coprocessor link | SPI | Lower complexity, but lower bandwidth than PCIe |
| CAN isolation | Prefer isolated CAN | Higher BOM and power cost, better field robustness |
| RS-232 support | Keep optional | Useful for legacy service, but not required in all variants |
| Board stack-up | Move to 8 layers | Higher PCB cost, better SI/EMC margin |
| Connector choice | Prefer M12 on field side | More expensive, but better IP and vibration performance |

## 10. Final Recommendation

For a production-oriented industrial edge gateway, the best balance of risk, scalability, and maintainability is:

- **Main CPU**: Qualcomm-based SoM, with `Lantronix Open-Q 6490CS SOM` as the current preferred candidate
- **Communication co-processor**: `Hilscher netX 90`
- **Host link**: `SPI + IRQ + reset + boot straps`
- **RS-485**: `TI ISO1410DWR`
- **CAN**: `TI ISO1042BDW`
- **IO-Link**: `ADI MAX14819ATM+`
- **Digital I/O**: `TI ISO1212DBQR` + `TI TPS272C45ARHFR`
- **Power front-end**: `LM5069MM-2/NOPB` + `LM76005RNPR`
- **PCB**: `8-layer carrier board`, isolated field domains, connector-side protection, and DNP-ready real-time Ethernet section for Phase 1

This architecture supports immediate software-based protocol deployment while keeping the hardware ready for deterministic industrial Ethernet in the next release without a full board respin.

### Qualcomm-specific freeze gate

Do not lock the PCB before the selected Qualcomm SoM vendor confirms:

- exported SPI host port for `netX 90`
- at least one suitable northbound Ethernet path
- Linux BSP support window
- industrial temperature grade or validated environmental limits
- carrier-board design guide availability
- long-term supply commitment aligned with the product lifecycle


---
---
# Industrial Edge Gateway BOM and PCB Design Update

## 1. Architecture Update

This update replaces the earlier "single Linux compute + field interfaces" assumption with a **dual-processor architecture** that is better aligned with phased industrial protocol deployment:

- **Main CPU**: Industrial Linux-capable ARM Cortex-A platform, `>= 1 GHz`, `>= 1 GB RAM`
- **Communication co-processor**: Dedicated industrial Ethernet multi-protocol controller for deterministic real-time Ethernet
- **Supervisor MCU**: Independent low-power microcontroller for sequencing, watchdog, recovery, and health monitoring

### Recommended partitioning

- **Phase 1**
  - Main CPU handles application logic, web/API stack, logging, configuration, Modbus RTU, CANopen, IO-Link master management, and digital I/O control.
  - Industrial Ethernet co-processor section is designed into the PCB but may be `DNP` or firmware-disabled in low-cost variants.

- **Phase 2**
  - Industrial Ethernet co-processor is populated and enabled for EtherCAT, PROFINET, EtherNet/IP, or other licensed real-time Ethernet stacks.
  - Main CPU remains the system controller, UI/API gateway, data concentrator, and OTA/update endpoint.

## 2. Recommended Dual-Processor Topology

### Preferred implementation

- **Main CPU module**
  - `Toradex Verdin iMX8M Plus Quad 2GB WB IT`
  - NXP i.MX8M Plus, quad Cortex-A53 up to 1.8 GHz
  - 2 GB LPDDR4, industrial temperature support, mature Linux BSP, long lifecycle

- **Industrial Ethernet co-processor**
  - `Hilscher netX 90` family, host-controlled industrial communication controller
  - Select exact orderable variant according to required protocol license bundle and memory option

- **Host interconnect**
  - **Recommended for this design**: `SPI`
  - Reason: lower PCB complexity than PCIe, broad SoM support, easier isolation of deterministic industrial Ethernet domain, and realistic integration path for a first production carrier board
  - Additional control lines:
    - `COPROC_SPI_CLK`
    - `COPROC_SPI_MOSI`
    - `COPROC_SPI_MISO`
    - `COPROC_SPI_CS_N`
    - `COPROC_IRQ_N`
    - `COPROC_RESET_N`
    - `COPROC_BOOTMODE[1:0]`
    - `COPROC_SYNC` (optional)

### SPI vs PCIe trade-off

| Option | Recommendation | Why |
|---|---|---|
| SPI | **Recommended now** | Lower risk, easier routing, available on nearly all industrial ARM SoMs, good fit for staged deployment |
| PCIe | Reserve only if needed | Better bandwidth, but more layers, tighter SI constraints, more validation effort, and not the most natural fit for netX 90 integration |

If later process-image size or cyclic update bandwidth exceeds SPI comfort margins, the roadmap should move to a higher-performance communication controller or a PCIe-capable real-time Ethernet architecture in a future board revision.

## 3. Updated PCB Block Diagram

```text
24V_IN / PE
    |
    v
+------------------------------------------------------+
| Power Entry, Protection, EMI, Hot-Swap, Buck Rails   |
| 24V -> 5V_MAIN / 3V3_LOGIC / isolated field rails    |
+------------------+-------------------+---------------+
                   |                   |
                   v                   v
        +------------------+   +----------------------+
        | Supervisor MCU   |   | Security / NVM / RTC |
        | Reset / WD / PM  |   | TPM / EEPROM / RTC   |
        +--------+---------+   +----------------------+
                 |
                 v
        +---------------------------+
        | Main CPU SoM              |
        | Linux / App / API / OTA   |
        | Modbus / CANopen / UI     |
        +-----+---------+-----------+
              |         |
              | SPI     | Native CPU I/O / UART / SPI / GPIO
              v         v
   +----------------+   +-----------------------------------+
   | netX 90        |   | RS-485 / optional RS-232          |
   | Ind. Ethernet  |   | IO-Link Master                    |
   | Co-processor   |   | CAN / CANopen                     |
   +---+--------+---+   | Digital Inputs / Outputs          |
       |        |       +-----------------------------------+
       |        |
       v        v
  Real-time  Real-time
  ETH PHY 1  ETH PHY 2
       |        |
       v        v
   M12/RJ45  M12/RJ45

Additional northbound Ethernet from CPU SoM:
   CPU GbE -> Magnetics -> RJ45/M12
```

## 4. Updated BOM

### 4.1 Core processing and control

| Category | Recommended Part | Qty | Rationale | Notes |
|---|---|---:|---|---|
| Main CPU SoM | `Toradex Verdin iMX8M Plus Quad 2GB WB IT` | 1 | Production-ready industrial ARM platform, Linux maturity, integrated RAM reduces DDR design risk | Good default for carrier-board architecture |
| Main CPU alternative | `Variscite DART-MX8M-PLUS 2GB Industrial` | 1 | Alternate industrial SoM source | Use only if supply-chain strategy requires second platform |
| Communication co-processor | `Hilscher netX 90` host-controlled variant | 1 | Dedicated real-time industrial Ethernet offload for Phase 2 | Exact commercial SKU depends on protocol bundle |
| Supervisor MCU | `STM32G474RET6` | 1 | Strong ADC/timer set for sequencing, health monitoring, watchdog, fault capture | Keep independent from Linux domain |
| TPM | `Infineon OPTIGA TPM SLB 9672 FW15` | 1 | Secure boot, key storage, device identity | SPI-connected to main CPU |
| RTC | `Micro Crystal RV-3028-C7` | 1 | Low-power RTC for audit trails and offline timestamp continuity | Optional but recommended |
| Boot / config EEPROM | `24AA256T-I/ST` | 1 | Stores board ID, BOM variant, calibration, manufacturing data | Simple and proven industrial support device |

### 4.2 Industrial Ethernet and network support

| Category | Recommended Part | Qty | Rationale | Notes |
|---|---|---:|---|---|
| Real-time Ethernet co-processor | `Hilscher netX 90` | 1 | Enables EtherCAT/PROFINET/EtherNet/IP migration in Phase 2 | Place on PCB from rev A, DNP if not populated |
| 100BASE-TX PHY for co-processor | `TI DP83822IFRHBR` | 2 | Industrial Ethernet PHY with wide temp support and strong EMC reputation | Good fit for real-time Ethernet ports |
| Northbound GbE port(s) | Provided by CPU SoM carrier design | 1-2 | Use SoM MAC/PHY path if available to reduce risk | Match selected SoM reference design |
| Ethernet magnetics / connector | `Bel Fuse / HALO / Würth industrial RJ45` or `M12 X-coded/Y-coded` | 2-4 | Connector style depends on enclosure and IP target | Use shield strategy tied to chassis near connector |
| Ethernet ESD | `Semtech RClamp0524P` or `TI TPD4E05U06-Q1` | per port | Protect cable-side differential pairs and LED/control pins | Use low-capacitance parts only |
| Common-mode choke | `Würth 7499011121A` family or equivalent | per port | Helps pre-compliance tuning for conducted/radiated emissions | DNI option may be useful during tuning |

### 4.3 CAN / CANopen

| Category | Recommended Part | Qty | Rationale | Notes |
|---|---|---:|---|---|
| Isolated CAN transceiver | `TI ISO1042BDW` | 1-2 | Robust isolated CAN FD transceiver; fully suitable for CANopen at classic CAN rates | Preferred for harsh industrial field cabling |
| Lower-cost non-isolated alternative | `TI TCAN1042HVDRQ1` | 1-2 | Good EMC and fault protection | Use only where system-level isolation is not required |
| CAN TVS | `Nexperia PESD1CAN` or `SM24CANB` | per port | ESD and surge protection on CANH/CANL | Place directly at connector |

### 4.4 RS-485 / RS-232

| Category | Recommended Part | Qty | Rationale | Notes |
|---|---|---:|---|---|
| Isolated RS-485 transceiver | `TI ISO1410DWR` | 2 | Industrial isolated Modbus RTU front-end with strong EMC performance | Preferred default |
| Optional isolated RS-232 | `Analog Devices ADM3252EABCZ` | 1 | Integrated isolated dual RS-232 for legacy commissioning/service | Optional service feature |
| RS-485 termination / bias | 120 ohm + bias network | per port | Required for stable field bus behavior | One bias source per segment only |
| RS-485 TVS | `SM712-02HTG` | per port | Specifically targeted for RS-485 line protection | Place at connector entry |

### 4.5 IO-Link

| Category | Recommended Part | Qty | Rationale | Notes |
|---|---|---:|---|---|
| IO-Link master IC | `Analog Devices MAX14819ATM+` | 2 | Proven dual-port IO-Link master with integrated framing and L+ control | Gives 4 ports total |
| Port protection / power switch | Per-port high-side protected switch, e.g. `TPS274160ARHARQ1` | 4 | Improves port fault isolation and current limiting | Use if IO-Link power budget per port is high |
| IO-Link connector | M8 or M12 A-coded | 4 | Industry-standard field connector choice | Decide with enclosure and IP target |

### 4.6 Digital I/O

| Category | Recommended Part | Qty | Rationale | Notes |
|---|---|---:|---|---|
| Digital input receiver | `TI ISO1212DBQR` | 4 | 24 V industrial digital input front-end, 2 channels per IC | Good match for 8 DI |
| Digital output driver | `TI TPS272C45ARHFR` | 4 | Smart high-side 24 V output driver, 2 channels per IC | Good fit for 8 DO sourcing outputs |
| Flexible DIO alternative | `ADI MAX14900EAGM+` | 2 | Use only if per-channel configurable DIO becomes a hard requirement | Higher integration, more firmware complexity |

### 4.7 Power, isolation, and protection

| Category | Recommended Part | Qty | Rationale | Notes |
|---|---|---:|---|---|
| Hot-swap / surge control | `TI LM5069MM-2/NOPB` | 1 | Strong 24 V industrial input protection and inrush control | Keep close to input connector |
| Main 24 V to 5 V buck | `TI LM76005RNPR` | 1 | 60 V-tolerant, robust industrial main rail converter | Good baseline for 5V_MAIN |
| 5 V to 3.3 V regulator | `TI TPS62130RGTR` or equivalent | 1-2 | Efficient secondary point-of-load regulation | Use local rails per domain where needed |
| Isolated DC/DC for field side | `Murata NXJ2S0505MC-R7` or equivalent | 2-4 | Power isolated RS-485/CAN and other field domains | Count depends on isolation partitioning |
| eFuse / protected high-side distribution | `TI TPS26624DRCR` | 2-4 | Per-bank field supply protection and fault reporting | Recommended for IO-Link and DIO banks |
| 24 V TVS | `SMBJ33A` | 1-2 | Input surge clamp | Validate against target surge standard |
| USB / service ESD | `TPD4E05U06QDRQ1` | as needed | Low-capacitance service-port protection | Keep near connector |

### 4.8 Passive and mechanical rules

| Category | Recommendation | Rationale |
|---|---|---|
| Bulk capacitors on 24 V input | Polymer + electrolytic combination | Better transient energy absorption |
| Ceramic decoupling | 100 nF + 1 uF + 10 uF staged close to each IC | Broadband decoupling strategy |
| Sense resistors | 0.1 percent / 25 ppm where current monitoring matters | Stable telemetry and fault thresholds |
| Bus resistors | 120 ohm, 1 percent for CAN/RS-485 | Stable differential bus behavior |
| Creepage / clearance | Dimension for target working voltage and pollution degree | Do not finalize without compliance target |
| Connector family | Prefer M12 for field ports in harsh environments | Higher robustness and IP compatibility than RJ45 |

## 5. Updated PCB Partitioning

Recommended schematic / layout partitioning:

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

Place the **main CPU SoM**, **netX 90**, and **northbound Ethernet** in the central/high-speed zone. Put **field transceivers and field connectors** on outer edges. Keep **isolation barriers** physically obvious and do not run high-speed digital traces across those boundaries.

## 6. PCB Design Recommendations

### 6.1 High-speed routing

#### SPI between main CPU and netX 90

- Keep SPI link on a continuous reference plane.
- Route as a tightly grouped bus with matched topology rather than exact high-speed length tuning.
- Add `22-33 ohm` series damping resistors at the driver side for `CLK`, `MOSI`, and optional `CS`.
- Keep the CPU-to-coprocessor SPI trace length ideally below `50 mm` on the carrier board.
- Route `COPROC_IRQ_N` away from noisy switching nodes.

#### Northbound Gigabit Ethernet

- Use `100 ohm differential impedance`.
- Avoid layer changes unless necessary.
- Keep pair-to-pair skew controlled.
- Do not cross plane splits.
- Place ESD and common-mode choke on the connector side, magnetics as close as mechanically practical.

#### Real-time Ethernet ports from netX 90

- Prefer short PHY-to-magnetics traces.
- Keep both real-time Ethernet ports geometrically symmetric.
- Place PHY clock sources away from power magnetics and hot-switch nodes.

### 6.2 Signal integrity and EMI/EMC

- Upgrade from the earlier 6-layer recommendation to an **8-layer carrier board** for the dual-processor variant:
  - `L1`: components / critical routing
  - `L2`: solid GND
  - `L3`: high-speed signals
  - `L4`: power planes
  - `L5`: solid GND
  - `L6`: medium-speed signals / control buses
  - `L7`: field power / low-speed routing
  - `L8`: field-side components and connectors
- Keep CPU, co-processor, PHYs, and clocks over uninterrupted return planes.
- Separate chassis-coupled connector region from quiet digital ground region with a controlled stitching strategy.
- Add optional footprints for common-mode chokes, RC snubbers, and split termination tuning where compliance risk is high.

### 6.3 Power supply and isolation

- Use a **single protected 24 V input** with hot-swap, reverse protection, surge clamp, and EMI filtering.
- Generate:
  - `5V_MAIN` for SoM and central logic
  - `3V3_LOGIC` for CPU I/O, supervisor, control logic
  - isolated 5 V or 3.3 V rails for RS-485 and CAN where isolation is required
  - protected `24V_FIELD` rails for IO-Link and digital outputs
- Do not share noisy field return currents with CPU return paths.
- Use per-bank or per-port protected high-side switching for IO-Link and DIO power domains.

### 6.4 Thermal management

- Budget thermal load for:
  - CPU SoM
  - netX 90
  - Ethernet PHYs
  - Buck regulators
  - IO-Link master ICs
  - High-side digital output drivers
- Provide thermal vias under power ICs and PHYs.
- Tie SoM heat spreader to chassis or metal frame where possible.
- Keep high-loss field output drivers away from the CPU and RTC/TPM zone.
- Validate worst-case ambient operation at `50 C` with all field channels active and real-time Ethernet traffic enabled.

### 6.5 Industrial protection

- Protect every external interface with a connector-side protection chain:
  - ESD
  - surge / TVS
  - optional common-mode choke
  - then transceiver / PHY
- Use isolation on:
  - RS-485 by default
  - CAN when field grounding conditions are not controlled
  - optional service interfaces only where required
- Use chassis-first shield termination for Ethernet and shielded serial connectors.
- Keep creepage and clearance visible in the layout around isolated sections.

## 7. Interface Coverage Summary

| Interface | Implementation | Phase 1 | Phase 2 |
|---|---|---|---|
| CAN / CANopen | Isolated CAN transceiver + CPU stack | Yes | Yes |
| RS-485 / Modbus RTU | Isolated RS-485 ports + CPU UART | Yes | Yes |
| Optional RS-232 | Isolated service / legacy serial | Optional | Optional |
| IO-Link | 4-port IO-Link master on CPU-controlled SPI/GPIO domain | Yes | Yes |
| Industrial Ethernet | netX 90 + dual PHY + protected ports | PCB-ready, optional BOM | Enabled |
| Digital I/O | 8 DI + 8 DO industrial 24 V front-end | Yes | Yes |

## 8. Scalability Strategy

### Phase 1 BOM strategy

Populate:

- Main CPU SoM
- Supervisor MCU
- RS-485
- CAN
- IO-Link
- DIO
- Security and service sections

Optionally leave `DNP`:

- `Hilscher netX 90`
- real-time Ethernet PHYs
- selected real-time Ethernet magnetics/connectors

This allows a lower-cost early production variant without a PCB redesign.

### Phase 2 BOM strategy

Populate and qualify:

- `Hilscher netX 90`
- dual real-time Ethernet PHYs
- port protection and connector set
- protocol-specific firmware / licensing

## 9. Key Trade-offs

| Topic | Recommendation | Trade-off |
|---|---|---|
| Main CPU implementation | Use industrial SoM | Slightly higher BOM cost, much lower DDR/bring-up risk |
| CPU-to-coprocessor link | SPI | Lower complexity, but lower bandwidth than PCIe |
| CAN isolation | Prefer isolated CAN | Higher BOM and power cost, better field robustness |
| RS-232 support | Keep optional | Useful for legacy service, but not required in all variants |
| Board stack-up | Move to 8 layers | Higher PCB cost, better SI/EMC margin |
| Connector choice | Prefer M12 on field side | More expensive, but better IP and vibration performance |

## 10. Final Recommendation

For a production-oriented industrial edge gateway, the best balance of risk, scalability, and maintainability is:

- **Main CPU**: `Toradex Verdin iMX8M Plus Quad 2GB WB IT`
- **Communication co-processor**: `Hilscher netX 90`
- **Host link**: `SPI + IRQ + reset + boot straps`
- **RS-485**: `TI ISO1410DWR`
- **CAN**: `TI ISO1042BDW`
- **IO-Link**: `ADI MAX14819ATM+`
- **Digital I/O**: `TI ISO1212DBQR` + `TI TPS272C45ARHFR`
- **Power front-end**: `LM5069MM-2/NOPB` + `LM76005RNPR`
- **PCB**: `8-layer carrier board`, isolated field domains, connector-side protection, and DNP-ready real-time Ethernet section for Phase 1

This architecture supports immediate software-based protocol deployment while keeping the hardware ready for deterministic industrial Ethernet in the next release without a full board respin.
