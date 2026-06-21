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
