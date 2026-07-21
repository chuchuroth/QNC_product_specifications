# QNC Industrial Protocol Gateway — Repositioned Product Roadmap & Architecture

**Version:** 2.0 — Product Repositioning Update  
**Date:** June 2026  
**Status:** Active — Supersedes all previous AI-Edge-focused roadmap documents  
**Classification:** Internal — Engineering & Product Strategy

---

> **Repositioning Notice**  
> This document supersedes the previous QNC roadmap and all associated architecture, BOM, and PCB guidance documents that referenced AI Edge Computing, Edge AI, Machine Learning, AI Inference, Vision AI, Generative AI, AI Acceleration, or AI-enabled analytics. Those references are removed in their entirety. The product is now defined exclusively as an **Industrial Protocol Gateway / Robotic Communication Gateway** optimised for deterministic fieldbus connectivity, multi-protocol translation, and industrial robot integration.

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Product Repositioning & Value Proposition](#2-product-repositioning--value-proposition)
3. [Updated System Architecture](#3-updated-system-architecture)
4. [TI Processor Evaluation Matrix](#4-ti-processor-evaluation-matrix)
5. [Recommended Hardware Platform & Rationale](#5-recommended-hardware-platform--rationale)
6. [Revised Project Roadmap](#6-revised-project-roadmap)
7. [Migration Plan — Previous to New Roadmap](#7-migration-plan--previous-to-new-roadmap)
8. [Risk Register](#8-risk-register)
9. [Appendix A — Removed Features & Rationale](#appendix-a--removed-features--rationale)
10. [Appendix B — Target Application Matrix](#appendix-b--target-application-matrix)

---

## 1. Executive Summary

### What Changed and Why

The QNC product has been repositioned from a general-purpose Linux edge compute platform with AI acceleration aspirations to a purpose-built **Industrial Protocol Gateway** targeting industrial robots, PLC integration, factory automation, and brownfield equipment connectivity.

This change eliminates hardware complexity, reduces BOM cost, shortens time-to-market, and aligns the product with an underserved industrial connectivity market where determinism, protocol breadth, long-term availability, and ruggedness matter far more than compute headroom.

### New Product Identity

| Attribute | Definition |
|---|---|
| **Product name** | QNC Industrial Protocol Gateway |
| **Primary positioning** | Multi-protocol industrial communication gateway for robotics and factory automation |
| **Alternative positioning** | Robotic Communication Gateway · Industrial Edge Connectivity Platform · Factory Automation Connectivity Gateway |
| **Core value** | Connects heterogeneous industrial devices and robots across incompatible protocols — deterministically, securely, and without requiring custom integration software |
| **What it is NOT** | An AI compute platform, an inference accelerator, a vision processing unit, or a general-purpose Linux edge server |

### Key Outcomes of Repositioning

- **Processor selection recentred** on Texas Instruments Sitara / PRU-ICSS family — purpose-built for industrial Ethernet and real-time fieldbus, with 15+ year longevity commitments
- **Qualcomm SoM candidates removed** — NPU/AI acceleration irrelevant; carrier-board BSP complexity unjustified for a gateway workload
- **iMX8M-Plus retained only as fallback** — viable if TI platform faces supply issue, but not preferred
- **netX 90 role clarified** — remains as the industrial Ethernet multi-protocol co-processor for EtherCAT / PROFINET / EtherNet/IP licensed stacks, unchanged
- **Software stack refocused** — runtime, protocol translation, device profiles, REST/WebSocket northbound retained; FastDDS scoped as extension only; all AI/ML pipeline work cancelled
- **Schedule impact** — architecture simplification recovers approximately 4–6 weeks versus previous plan; revised GA target remains Week 44 with higher confidence

---

## 2. Product Repositioning & Value Proposition

### 2.1 Market Positioning Statement

> **QNC Industrial Protocol Gateway** bridges the communication gap between industrial robots, PLCs, sensors, and factory systems by translating across IO-Link, Modbus RTU, EtherNet/IP, CANopen, PROFINET, and EtherCAT — deterministically, at the edge, without cloud dependency.

### 2.2 Target Applications

| Application | Protocol Challenge Solved | Primary Buyer |
|---|---|---|
| Industrial robot integration | Translate robot controller I/O (EtherNet/IP, PROFINET) to plant Modbus/IO-Link sensors | System integrator, robot OEM |
| PLC connectivity | Bridge legacy Modbus RTU PLCs to modern EtherNet/IP or PROFINET networks | Factory automation engineer |
| Brownfield equipment retrofit | Connect legacy RS-485/CAN devices to IIoT northbound REST/MQTT without ripping infrastructure | Maintenance and OT engineer |
| IO-Link master aggregation | Centralise up to 4 IO-Link device ports with northbound REST telemetry to SCADA/MES | Machine builder, OEM |
| Factory floor protocol gateway | Real-time EtherCAT master-to-Modbus RTU slave bridging for motion control integration | Drive and motion OEM |
| IIoT connectivity hub | Aggregate multi-protocol field data, normalise, and publish to MQTT/OPC UA northbound | IIoT platform vendor |
| Manufacturing interoperability | Enable heterogeneous device communication without PLC program changes | Manufacturing IT |
| Collaborative robot (cobot) integration | IO-Link safety device aggregation and EtherNet/IP adapter for cobot controllers | Cobot OEM, integrator |

### 2.3 Value Proposition by Persona

**System Integrator**
- One gateway replaces 3–5 separate protocol converter modules
- Pre-validated device profiles eliminate custom mapping scripts
- REST/WebSocket northbound enables rapid SCADA and MES integration

**Robot OEM / Cobot Manufacturer**
- Drop-in EtherNet/IP adapter or PROFINET device interface for robot controller connectivity
- IO-Link master ports for integrated tool and sensor management
- Industrial hardening: 24 V, DIN rail, 0–50 °C, IEC 61131 DIO levels

**Factory Automation Engineer**
- Brownfield retrofit without PLC program changes — gateway appears as native fieldbus device
- Secure remote configuration and OTA update via authenticated REST API
- Safe Mode and rollback eliminate unplanned production stops from firmware issues

**IIoT / Digital Twin Platform Vendor**
- Structured JSON telemetry stream via WebSocket — no custom parser required
- Device profile catalog maps physical device semantics to normalized data model
- Open REST API with OpenAPI 3.0 spec for straightforward platform integration

### 2.4 Competitive Differentiation

| Dimension | QNC Industrial Protocol Gateway | Typical Anybus / HMS Competitor | DIY Linux Gateway |
|---|---|---|---|
| Protocol breadth (baseline) | IO-Link + Modbus RTU + EtherNet/IP + DIO + CAN | Usually 2-protocol pair per SKU | Depends on stack sourcing |
| Real-time Ethernet (Phase 2) | EtherCAT / PROFINET / EtherNet/IP via netX 90 | ✅ Yes (licensed) | Requires separate FPGA or ASIC |
| Northbound REST / WebSocket | ✅ Built-in, OpenAPI 3.0 | ❌ Usually absent | Custom implementation |
| Device profile catalog | ✅ YAML-governed, versioned, validated | ❌ Usually absent | Custom implementation |
| Secure OTA + rollback | ✅ TPM-backed, signed artifacts | ❌ Usually absent | Custom implementation |
| Open integration API | ✅ REST + WebSocket + structured logs | ❌ Proprietary tools | Varies |
| Industrial longevity | TI Sitara 15+ yr lifecycle | ✅ Industrial-grade | SoC-dependent |
| Price tier | Mid-range industrial | Mid–premium | Low HW, high NRE |

---

## 3. Updated System Architecture

### 3.1 Architecture Philosophy

The repositioned architecture applies three principles:

1. **Determinism over compute headroom** — real-time protocol translation requires predictable cycle times, not GHz of application CPU. The PRU-ICSS subsystem on TI AM64x / AM243x handles time-critical fieldbus directly in hardware.
2. **Protocol isolation** — each fieldbus domain is fault-contained. A fault on an RS-485 port must not affect EtherNet/IP throughput or IO-Link master operation.
3. **Northbound openness** — the application CPU runs a secure Linux environment for REST, WebSocket, device profile management, OTA, and logging, accessible to any SCADA, MES, or IIoT platform.

### 3.2 Functional Block Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         24 V Industrial Power Entry                         │
│          Hot-swap · TVS · EMI filter · LM5069 · LM76005 buck               │
└──────────────────────────────────┬──────────────────────────────────────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
              ▼                    ▼                    ▼
   ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
   │  Supervisor MCU  │  │ Security / NVM   │  │  Power monitors  │
   │  STM32G0 / G4    │  │  TPM SLB9672     │  │  per-rail ADC    │
   │  Watchdog/Reset  │  │  EEPROM 24AA256  │  │  supervisor loop │
   │  Rail monitor    │  │  RTC RV-3028     │  │                  │
   └────────┬─────────┘  └──────────────────┘  └──────────────────┘
            │
            ▼
┌───────────────────────────────────────────────────────────────────────────┐
│                    Main Application Processor (TI AM64x)                  │
│                                                                           │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │              Linux Application Domain (Cortex-A53)                  │  │
│  │  • QNC runtime: lifecycle, config, rollback, device profiles        │  │
│  │  • REST API (OpenAPI 3.0) · WebSocket telemetry · Structured logs   │  │
│  │  • Protocol normalisation, command routing, fault management        │  │
│  │  • Secure OTA, TPM-signed artifact activation                       │  │
│  │  • MQTT / OPC UA northbound bridge (extension)                      │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
│                                                                           │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │           Real-Time Domain (Cortex-R5F + PRU-ICSS)                  │  │
│  │  • IO-Link master firmware (4 ports via MAX14819)                   │  │
│  │  • Modbus RTU master/slave (2× RS-485 via ISO1410)                  │  │
│  │  • CANopen protocol stack (1× CAN via ISO1042)                      │  │
│  │  • DIO management (8 DI · 8 DO · IEC 61131 levels)                 │  │
│  │  • Deterministic inter-domain message passing to A53                │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
│                                                                           │
│  ┌─────────────────────────────────────────────────────────────────────┐  │
│  │       Industrial Ethernet Domain (PRU-ICSS or netX 90 SPI)          │  │
│  │  Phase 1: EtherNet/IP adapter via PRU-ICSS (AM64x native)           │  │
│  │  Phase 2: EtherCAT / PROFINET / EtherNet/IP via Hilscher netX 90   │  │
│  └─────────────────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────────────┘
            │                    │                      │
            ▼                    ▼                      ▼
  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────────┐
  │  Fieldbus Front  │  │  Ethernet Ports  │  │  Service & Management    │
  │  IO-Link ×4      │  │  GbE northbound  │  │  USB-C console           │
  │  RS-485 ×2       │  │  100M RT Eth ×2  │  │  SWD/JTAG debug          │
  │  CAN ×1          │  │  (netX 90 Ph.2)  │  │  Recovery header         │
  │  8 DI · 8 DO     │  │  M12 / RJ45      │  │  Factory DIP config      │
  └──────────────────┘  └──────────────────┘  └──────────────────────────┘
```

### 3.3 Key Architecture Decisions vs. Previous Version

| Decision Point | Previous Architecture | Repositioned Architecture | Rationale |
|---|---|---|---|
| Main CPU | Qualcomm QCS6490 SoM (NPU, AI workload) | **TI AM64x (Cortex-A53 + R5F + PRU-ICSS)** | PRU-ICSS handles fieldbus natively; no NPU needed; 15-yr lifecycle |
| CPU selection driver | AI/ML inference headroom | **Deterministic real-time Ethernet + fieldbus** | Product mission changed |
| Industrial Ethernet Phase 1 | netX 90 only (DNP) | **AM64x PRU-ICSS native EtherNet/IP adapter** | Reduces Phase 1 cost; netX 90 reserved for PROFINET/EtherCAT in Phase 2 |
| Co-processor (netX 90) | Phase 1 DNP, Phase 2 enabled | **Phase 2 DNP, enabled for PROFINET/EtherCAT** | PRU-ICSS covers EtherNet/IP in Phase 1 without licensing cost |
| Host interface to netX 90 | SPI | **SPI (unchanged)** | Still the correct choice; AM64x SPI available |
| PCB layers | 8-layer (dual high-speed processor) | **6-layer** (single SoC, no DDR complexity) | AM64x with integrated LPDDR4 on SiP variant; reduced stack |
| OS | Linux (Yocto) | **Linux (Yocto / TI SDK)** | Unchanged; TI provides mature Yocto BSP |
| Supervisor MCU | STM32G474 | **STM32G0 series (cost-optimised)** | Simpler supervision task; G474 ADC headroom not needed |
| Removed subsystems | — | NPU, MIPI CSI camera, LPDDR5 high-bandwidth memory, PCIe for AI accelerator | Not applicable to gateway |

### 3.4 Protocol Coverage Map

| Protocol | Direction | Implementation | Phase |
|---|---|---|---|
| IO-Link v1.1 | Master (4 ports) | MAX14819 ×2 + R5F firmware | Phase 1 |
| Modbus RTU | Master + Slave (2 ports) | ISO1410 ×2 + R5F firmware | Phase 1 |
| EtherNet/IP | Adapter (Class 1 + Class 3) | AM64x PRU-ICSS | Phase 1 |
| CANopen | Master + Slave (1 port) | ISO1042 + R5F firmware | Phase 1 |
| Discrete DIO | 8 DI + 8 DO (24 V IEC 61131) | ISO1212 ×4 + TPS272C45 ×4 | Phase 1 |
| EtherCAT | Slave + optional Master | Hilscher netX 90 | Phase 2 |
| PROFINET | Device (IO-Device) | Hilscher netX 90 | Phase 2 |
| EtherNet/IP Scanner | Scanner (initiator) | Hilscher netX 90 | Phase 2 |
| Modbus TCP | Client + Server | Linux TCP stack | Phase 1 extension |
| MQTT | Publisher (northbound) | Linux application | Phase 1 extension |
| OPC UA | Server (northbound) | Linux application | Phase 2 |

### 3.5 Software Stack Architecture

```
┌──────────────────────────────────────────────────────────────────────────┐
│                        NORTHBOUND INTERFACES                             │
│   REST API (OpenAPI 3.0)  ·  WebSocket  ·  MQTT  ·  OPC UA (Phase 2)   │
├──────────────────────────────────────────────────────────────────────────┤
│                         QNC RUNTIME CORE                                 │
│  Lifecycle Manager  ·  Config/Rollback  ·  Fault Manager  ·  Safe Mode  │
├──────────────────────────────────────────────────────────────────────────┤
│                    PROTOCOL NORMALISATION LAYER                          │
│  Device Profile Engine  ·  Command Router  ·  Telemetry Aggregator      │
├──────────────────────────────────────────────────────────────────────────┤
│                     SOUTHBOUND PROTOCOL SERVICES                         │
│  IO-Link  ·  Modbus RTU  ·  EtherNet/IP  ·  CANopen  ·  DIO            │
│  (Phase 2: EtherCAT · PROFINET via netX 90 host driver)                 │
├──────────────────────────────────────────────────────────────────────────┤
│                       SECURITY & MANAGEMENT                              │
│  TPM 2.0  ·  Signed OTA  ·  TLS 1.3  ·  RBAC  ·  Audit Log            │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 4. TI Processor Evaluation Matrix

### 4.1 Evaluation Criteria

Gateway requirements — ranked by priority:

| # | Criterion | Weight | Rationale |
|---|---|---|---|
| 1 | PRU-ICSS / industrial Ethernet support | Critical | Native deterministic fieldbus without external ASIC |
| 2 | Real-time core (R5F or M4F) availability | Critical | Deterministic protocol handling independent of Linux |
| 3 | Industrial temperature grade (-40 to +85 °C) | Critical | DIN rail / control cabinet environment |
| 4 | Long-term availability (≥10 years) | Critical | Industrial product lifecycle requirement |
| 5 | IO-Link master support (via SPI/GPIO) | High | Primary sensor interface |
| 6 | Modbus RTU UART availability | High | Legacy device connectivity |
| 7 | CAN / CAN-FD availability | High | Robot controller and automation bus |
| 8 | GbE MAC for northbound | High | Management and data upload |
| 9 | Sufficient RAM for Linux + protocol stacks | High | ≥512 MB recommended |
| 10 | Low power consumption | Medium | Thermal management in sealed enclosure |
| 11 | TI SDK / Yocto BSP maturity | Medium | Development risk mitigation |
| 12 | Package / PCB complexity | Medium | Carrier board cost and schedule |

### 4.2 Candidate Processor Comparison Matrix

| Criterion | AM64x (AM6442) | AM243x (AM2434) | AM263x (AM2634) | AM261x (AM2612) | LP-AM261 |
|---|---|---|---|---|---|
| **CPU cores** | 2× Cortex-A53 + 2× R5F + PRU-ICSS | 4× R5F + PRU-ICSS (no A-class) | 4× R5F + PRU-ICSS (no A-class) | 2× R5F + PRU-ICSS (no A-class) | 2× R5F + PRU-ICSS |
| **Linux capable** | ✅ Yes (A53) | ❌ No (RTOS only) | ❌ No (RTOS only) | ❌ No (RTOS only) | ❌ No (RTOS only) |
| **PRU-ICSS instances** | 2× PRU-ICSS | 1× PRU-ICSS | 1× PRU-ICSS | 1× PRU-ICSS | 1× PRU-ICSS |
| **Industrial Ethernet** | EtherCAT, PROFINET, EtherNet/IP (PRU) | EtherCAT, EtherNet/IP (PRU) | EtherCAT, EtherNet/IP (PRU) | EtherNet/IP, Modbus TCP (PRU) | EtherNet/IP (PRU) |
| **GbE ports** | 2× GbE (CPSW) | 2× GbE (CPSW) | 2× 100M Ethernet | 2× 100M Ethernet | 2× 100M Ethernet |
| **RAM (on-chip / ext)** | External DDR4/LPDDR4, up to 4 GB | On-chip 1.25 MB + ext EtherRAM | On-chip 1.5 MB SRAM | On-chip 1.5 MB SRAM | On-chip 1 MB SRAM |
| **CAN / CAN-FD** | ✅ CAN-FD ×2 | ✅ CAN-FD ×3 | ✅ CAN-FD ×3 | ✅ CAN-FD ×2 | ✅ CAN-FD ×2 |
| **UART (RS-485)** | ✅ ×6 | ✅ ×6 | ✅ ×6 | ✅ ×4 | ✅ ×4 |
| **SPI (for IO-Link ICs)** | ✅ ×3 | ✅ ×3 | ✅ ×3 | ✅ ×2 | ✅ ×2 |
| **Industrial temp** | ✅ -40 to +85 °C | ✅ -40 to +85 °C | ✅ -40 to +85 °C | ✅ -40 to +85 °C | ✅ -40 to +85 °C |
| **Long-term availability** | ✅ 15+ yr TI commitment | ✅ 15+ yr TI commitment | ✅ 15+ yr TI commitment | ✅ 15+ yr TI commitment | ✅ 15+ yr TI commitment |
| **Typical power (active)** | 2.5–4 W (A53 + R5F active) | 0.8–1.5 W | 0.8–1.5 W | 0.5–1.0 W | 0.4–0.8 W |
| **Package** | 17×17 mm FCBGA-784 | 12×12 mm FCBGA-196 | 12×12 mm FCBGA-196 | 10×10 mm FCBGA-196 | 9×9 mm FCBGA-144 |
| **PCB complexity** | Medium (DDR routing required) | Low (no DDR, on-chip SRAM) | Low | Low | Very low |
| **TI SDK / BSP** | Mature (Yocto + RTOS SDK) | Mature (RTOS only) | Mature (RTOS only) | Mature (RTOS only) | Mature (RTOS only) |
| **TI industrial Eth SDK** | ✅ INDUSTRIAL-SDK | ✅ INDUSTRIAL-SDK | ✅ INDUSTRIAL-SDK | ✅ INDUSTRIAL-SDK | ✅ INDUSTRIAL-SDK |
| **Suitable for this product** | ✅ **PRIMARY RECOMMENDATION** | ⚠️ Needs Linux companion | ⚠️ Needs Linux companion | ⚠️ Limited RAM for northbound | ⚠️ Limited RAM for northbound |

### 4.3 Scoring Summary (1–5 per criterion, weighted)

| Criterion | Weight | AM64x | AM243x | AM263x | AM261x | LP-AM261 |
|---|---|---|---|---|---|---|
| PRU-ICSS / ind. Ethernet | 5× | 5 | 4 | 4 | 3 | 3 |
| Real-time core | 5× | 5 | 5 | 5 | 4 | 4 |
| Industrial temp | 5× | 5 | 5 | 5 | 5 | 5 |
| Long-term availability | 5× | 5 | 5 | 5 | 5 | 5 |
| Linux capability | 4× | 5 | 1 | 1 | 1 | 1 |
| GbE northbound | 4× | 5 | 5 | 3 | 3 | 3 |
| RAM adequacy | 4× | 5 | 2 | 2 | 2 | 1 |
| CAN / UART availability | 3× | 5 | 5 | 5 | 4 | 4 |
| Low power | 3× | 3 | 5 | 5 | 5 | 5 |
| PCB complexity (inverse) | 3× | 3 | 5 | 5 | 5 | 5 |
| TI SDK maturity | 2× | 5 | 5 | 5 | 5 | 5 |
| **Weighted total** | — | **194** | **159** | **155** | **149** | **146** |

**AM64x scores highest** across weighted criteria due to its unique combination of Linux-capable Cortex-A53 cores alongside R5F and PRU-ICSS on a single SoC.

### 4.4 AM243x / AM263x as Co-Processor Option

If a dual-chip architecture is preferred to maximise real-time determinism separation:

- **Main application CPU:** A lightweight Linux SBC (e.g., TI AM62x single-core A53, minimal BOM) for REST/WebSocket/OTA only
- **Real-time gateway CPU:** AM243x or AM263x for all protocol stacks (IO-Link, Modbus, CANopen, EtherNet/IP via PRU-ICSS)
- **Trade-off:** Two SoCs increase PCB area and BOM cost by ~$8–15/unit; inter-processor communication (SPI or dual-port RAM) adds latency; not recommended for the baseline product

This option should be revisited only if the AM64x supply chain encounters problems.

---

## 5. Recommended Hardware Platform & Rationale

### 5.1 Primary Recommendation: TI AM64x (AM6442)

**Part:** `TI AM6442BBBCAAD1` (industrial grade, -40 to +85 °C, FCBGA-784)  
**Alternative:** `TI AM6441` (single A53 core, lower cost, adequate for gateway workload)

#### Why AM64x

| Reason | Detail |
|---|---|
| **Single-SoC gateway** | A53 cores run Linux (northbound, config, OTA, REST/WebSocket); R5F cores run deterministic protocol firmware; PRU-ICSS handles time-critical industrial Ethernet — all on one die |
| **No external DDR complexity** | AM64x SIP (System-in-Package) variants include LPDDR4 on-package, eliminating high-speed DDR routing risk |
| **PRU-ICSS native EtherNet/IP** | TI INDUSTRIAL-SDK ships EtherNet/IP adapter firmware for PRU-ICSS, enabling Phase 1 without netX 90 licensing cost |
| **15+ year supply commitment** | TI Sitara industrial family has documented longevity policy — essential for industrial product |
| **Eliminates AI-compute overspec** | AM64x is correctly sized for gateway; previous Qualcomm QCS6490 was oversized by 10× for this workload and introduced NPU/LPDDR5 complexity with no product benefit |
| **Mature TI SDK** | Yocto layer, Linux 5.10 LTS, RTOS SDK, PRU firmware examples, industrial Ethernet SDK — all production-ready |
| **Cost** | ~$18–25/unit at volume (vs. $45–80 for Qualcomm SoM) |

#### AM64x Resource Allocation

| Resource | Allocation |
|---|---|
| Cortex-A53 core 0 | Linux OS — QNC runtime, REST API, WebSocket, device profile engine |
| Cortex-A53 core 1 | Linux OS — config manager, OTA, secure logging, northbound MQTT/OPC UA |
| Cortex-R5F core 0 | Real-time — IO-Link master FSM, Modbus RTU master/slave |
| Cortex-R5F core 1 | Real-time — CANopen stack, DIO management, supervisor IPC |
| PRU-ICSS 0 | Industrial Ethernet — EtherNet/IP adapter (Phase 1) or netX 90 host driver (Phase 2) |
| PRU-ICSS 1 | Industrial Ethernet — spare / second EtherNet/IP port or EtherCAT (Phase 2) |

### 5.2 Fallback Option: TI AM62x + netX 90 (Dual-chip)

If AM64x sourcing becomes constrained:

- **AM62x (AM6254):** Quad Cortex-A53, no R5F, no PRU-ICSS — Linux-only, lower cost
- **Supplement with:** netX 90 for all real-time industrial Ethernet; separate MCU (STM32G4) for IO-Link/Modbus/CANopen firmware
- **Trade-off:** Three-chip solution; more PCB area; higher NRE; acceptable as risk mitigation only

### 5.3 Updated BOM — Core Processing and Control

| Category | Recommended Part | Qty | Rationale |
|---|---|---|---|
| Main application SoC | `TI AM6442BBBCAAD1` (or AM6441) | 1 | Linux + R5F + PRU-ICSS gateway SoC; 15-yr lifecycle |
| Fallback SoC | `TI AM6254` (AM62x family) | 1 | Linux-only fallback; requires netX 90 + MCU companion |
| Industrial Eth co-processor | `Hilscher netX 90` (host-controlled) | 1 | Phase 2: EtherCAT / PROFINET / EtherNet/IP licensed stacks |
| Supervisor MCU | `STM32G071RBT6` | 1 | Rail monitoring, watchdog, reset sequencing — G0 series adequate |
| TPM | `Infineon OPTIGA TPM SLB 9672 FW15` | 1 | SPI TPM 2.0, secure boot, key storage, device identity |
| RTC | `Micro Crystal RV-3028-C7` | 1 | Timestamps, audit log continuity, offline recovery |
| Config / board-ID EEPROM | `24AA256T-I/ST` | 1 | Board ID, BOM variant, calibration data |

### 5.4 Updated BOM — Field Interfaces (Unchanged from Previous)

| Category | Recommended Part | Qty | Rationale |
|---|---|---|---|
| IO-Link master IC | `Analog Devices MAX14819ATM+` | 2 | Dual-port, 4 ports total; proven IO-Link framer |
| IO-Link port protection | `TPS274160ARHARQ1` | 4 | Per-port high-side current limiting |
| Isolated RS-485 transceiver | `TI ISO1410DWR` | 2 | 5 kVrms isolation; Modbus RTU front-end |
| Isolated CAN transceiver | `TI ISO1042BDW` | 1 | Isolated CAN-FD; CANopen at classic rates |
| Digital input receiver | `TI ISO1212DBQR` | 4 | 24 V isolated DI, 2 ch/IC; 8 DI total |
| Digital output driver | `TI TPS272C45ARHFR` | 4 | Smart high-side 24 V DO, 2 ch/IC; 8 DO total |
| 100BASE-TX PHY (netX 90) | `TI DP83822IFRHBR` | 2 | Industrial PHY for Phase 2 real-time Ethernet ports |

### 5.5 Updated BOM — Power (Unchanged from Previous)

| Category | Recommended Part | Qty | Rationale |
|---|---|---|---|
| Hot-swap / inrush control | `TI LM5069MM-2/NOPB` | 1 | 24 V industrial input protection |
| Main 24 V → 5 V buck | `TI LM76005RNPR` | 1 | 60 V-tolerant, 5 A, robust industrial rail |
| 5 V → 3.3 V regulator | `TI TPS62130RGTR` | 1–2 | Point-of-load regulation per domain |
| Isolated DC/DC (field side) | `Murata NXJ2S0505MC-R7` | 2–4 | Isolated RS-485 / CAN field power |
| eFuse / field supply protection | `TI TPS26624DRCR` | 2–4 | Per-bank IO-Link / DIO overcurrent protection |
| 24 V TVS input clamp | `SMBJ33A` | 1–2 | Surge protection at input |

### 5.6 Updated PCB Specification

| Parameter | Previous Spec | Updated Spec | Change Reason |
|---|---|---|---|
| Layer count | 8 layers | **6 layers** | Single SoC; no high-speed DDR BGA fanout; significant cost saving |
| Stack-up | 8-layer (dual processor complexity) | L1 signals / L2 GND / L3 power / L4 signals / L5 GND / L6 field signals | Adequate for AM64x + field interfaces |
| Impedance targets | 50 Ω SE, 100 Ω diff, 90 Ω USB | **50 Ω SE, 100 Ω diff** (USB 90 Ω retained if used) | No change in critical requirements |
| Board thickness | 1.6 mm | **1.6 mm** | Unchanged |
| Material | FR-4, Tg ≥ 170 °C | **FR-4, Tg ≥ 170 °C** | Unchanged |
| DFM class | IPC Class 2 | **IPC Class 2** | Unchanged |
| Estimated PCB cost | High (8L, complex stack) | **~30–40% reduction** (6L) | Major BOM cost improvement |

---

## 6. Revised Project Roadmap

### 6.1 Timeline Summary

| Phase | Phase Name | Duration | Start | End | Critical Path |
|---|---|---|---|---|---|
| **P0** | Architecture Freeze & Scope Alignment | 3 wks | Wk 1 | Wk 3 | ✅ YES |
| **P1** | Requirements Closure & BOM Finalization | 3 wks | Wk 2 | Wk 4 | ✅ YES |
| **P2** | Schematic Design & Component Verification | 6 wks | Wk 4 | Wk 9 | ✅ YES |
| **P3** | PCB Layout (6-layer), Stack-up & DFM | 5 wks | Wk 8 | Wk 12 | ✅ YES |
| **P4** | EVT Build & Hardware Bring-up | 7 wks | Wk 13 | Wk 19 | ✅ YES |
| **P5** | Core Software Platform *(parallel)* | 11 wks | Wk 8 | Wk 18 | ↔ PARALLEL |
| **P6** | System Integration & Alpha | 6 wks | Wk 19 | Wk 24 | ✅ YES |
| **P7** | DVT & Validation | 8 wks | Wk 25 | Wk 32 | ✅ YES |
| **P8** | Beta Pilot & Manufacturing Prep | 6 wks | Wk 33 | Wk 38 | ✅ YES |
| **P9** | Release Candidate & Launch | 4 wks | Wk 39 | Wk 42 | ✅ YES |
| **Buffer** | 15% contingency | ~3 wks | absorbed | Wk 44 | — |

**Revised GA target: Week 42–44 from kickoff (~May 2027 if starting June 2026)**  
Architecture simplification recovers 2–4 weeks versus the previous plan with higher schedule confidence.

### 6.2 Phase-by-Phase Breakdown

---

#### P0 — Architecture Freeze & Scope Alignment (Wk 1–3)

**Duration:** 3 weeks | Low: 2 wks | Expected: 3 wks | High: 4 wks

**Deliverables**
- Architecture Decision Record: TI AM64x confirmed as primary SoC; Qualcomm and iMX8M-Plus candidates formally closed
- AI/ML references removed from all existing project documents, Jira backlog, and marketing materials
- Updated variant matrix: port counts, Phase 1/2 DNP rules, protocol release map
- Jira backlog hygiene: all AI-edge, dev board (GD32H75E, AX58101), and legacy architecture tasks closed or reclassified
- Confirmed 6-layer PCB target and updated PDN strategy for single-SoC platform

**Dependencies**
- TI AM64x industrial availability confirmed (standard production item — no gating expected)
- Product management sign-off on AI feature removal
- Engineering team briefed on repositioning

**Risks**
- Stakeholder resistance to removing AI features: mitigate with written product strategy decision
- Jira cleanup uncovering orphaned dependencies: allocate dedicated clean-up sprint

---

#### P1 — Requirements Closure & BOM Finalization (Wk 2–4)

**Duration:** 3 weeks (overlapping P0) | Low: 3 wks | Expected: 3 wks | High: 5 wks

**Deliverables**
- Signed Requirements Addendum: 24 V power spec, isolation withstand, DIO topology, IP rating, regulatory scope
- Connector family decision: M12 vs RJ45; field terminal standard
- BOM gap analysis: all missing MPNs resolved; AM64x replaces Qualcomm/iMX8M-Plus throughout
- AVL lifecycle assessment for AM64x, MAX14819, ISO1410, TPM, ISO1042
- Symbol and footprint verification for all updated BOM components (AM64x FCBGA-784 package)
- Procurement advance buffer order for AM64x, MAX14819, ISO1042 (long-lead A-parts)

**Dependencies**
- P0 architecture decision record published
- Mechanical team input on connector and enclosure

**Risks**
- AM64x FCBGA-784 footprint complexity → engage PCB designer early for fanout review
- Connector decision delay remains a direct PCB layout blocker

---

#### P2 — Schematic Design (Wk 4–9)

**Duration:** 6 weeks | Low: 5 wks | Expected: 6 wks | High: 8 wks

**Deliverables — Updated 12-Sheet Hierarchy**

| Sheet | Content |
|---|---|
| `01_PWR_ENTRY_PROTECTION` | 24 V input, LM5069 hot-swap, TVS, EMI filter, LM76005 buck |
| `02_AM64x_SOC` | AM64x pin assignment, DDR/LPDDR interface (if discrete), boot mode, clock |
| `03_SUPERVISOR_MCU` | STM32G071, rail monitoring, watchdog, reset chain |
| `04_ETH_COPROCESSOR_NETX90` | netX 90 (DNP Phase 1), SPI host signals, BOOTMODE straps |
| `05_RT_ETH_PHY_PORTS` | DP83822 ×2, magnetics, ESD, connector (netX 90 Phase 2 ports) |
| `06_NORTHBOUND_ETHERNET` | AM64x CPSW GbE, magnetics, RJ45/M12, Bob Smith termination |
| `07_RS485_MODBUS` | ISO1410 ×2, TVS (SM712), bias/termination, optional RS-232 |
| `08_IOLINK_MASTER` | MAX14819 ×2 (4 ports), L+ high-side, port protection |
| `09_CAN_CANOPEN` | ISO1042 (isolated CAN-FD), TVS, termination, CANopen ready |
| `10_DIGITAL_IO` | ISO1212 ×4 (8 DI), TPS272C45 ×4 (8 DO), per-bank eFuse |
| `11_SECURITY_NVM_SERVICE` | TPM SLB9672, EEPROM, RTC, USB-C console, SWD/JTAG |
| `12_TEST_AND_PRODUCTION` | Test points, bed-of-nails, current shunts, loopback jumpers |

**Dependencies**
- P1 Requirements Addendum signed; BOM finalized; footprint library verified

**Risks**
- AM64x FCBGA-784 power domain decoupling requires careful PDN — budget 1 extra week for PDN review
- PRU-ICSS pin assignment must be co-ordinated with TI industrial SDK pin-mux constraints

---

#### P3 — PCB Layout, 6-Layer Carrier (Wk 8–12)

**Duration:** 5 weeks | Low: 4 wks | Expected: 5 wks | High: 7 wks

**Deliverables**
- 6-layer stack-up confirmed with board house: 50 Ω SE, 100 Ω diff
- Full placement: AM64x central zone, field transceivers at board edges, visible isolation barriers
- High-speed routing: CPSW GbE MDI pairs (100 Ω diff), SPI to netX 90 (≤50 mm, damping resistors)
- PRU-ICSS Ethernet traces symmetric, short PHY-to-magnetics
- DFM/DFA package: IPC Class 2, 6/6 mil rules, QFN solder mask reliefs, fiducials
- Gerber files, drill files, assembly drawings, DRC/ERC pass certificate

**Dependencies**
- P2 schematic and netlist approved
- Board house DFM rules and stack-up pricing confirmed
- Enclosure mechanical outline finalised

**Risks**
- AM64x FCBGA-784 fanout may require blind vias on inner layers — engage fab in Wk 8 DFM review
- 6-layer is sufficient; if AM64x SiP with on-package LPDDR4 is selected, DDR routing eliminated entirely

---

#### P4 — EVT Build & Hardware Bring-up (Wk 13–19)

**Duration:** 7 weeks | Low: 6 wks | Expected: 7 wks | High: 10 wks

**Deliverables**
- EVT board assemblies: 5–10 units
- Power-up checklist: Power → Supervisor MCU → AM64x boot → field interfaces
- AM64x Linux boot confirmed; TI SDK Yocto image running; PRU-ICSS enumerated
- All field ports functional: IO-Link discovery (4 ports), Modbus RTU loopback (2 ports), CAN frame, DIO toggle
- Thermal measurement at room temperature under load
- EVT issue log with triage and rework decisions

**Dependencies**
- P3 Gerbers released to EMS; EMS selected and NDA signed by Wk 10
- TI AM64x evaluation board available for SW bring-up from Wk 8 (parallel to P5)

**Risks**
- EMS assembly lead time 3–4 weeks standard
- PRU-ICSS firmware bring-up (pin-mux, RGMII timing) may require TI FAE support

---

#### P5 — Core Software Platform, Parallel Track (Wk 8–18)

**Duration:** 11 weeks | Low: 9 wks | Expected: 11 wks | High: 14 wks

*Runs in parallel. Uses TI AM64x evaluation board (SK-AM64B or similar) until EVT hardware available.*

**Deliverables**
- TI Yocto BSP ported and Linux booting on AM64x eval board
- PRU-ICSS EtherNet/IP adapter firmware integrated from TI INDUSTRIAL-SDK
- R5F firmware: IO-Link master FSM (MAX14819 SPI driver), Modbus RTU master/slave, CANopen stack
- Inter-processor communication (IPC) between R5F and A53 validated
- QNC runtime: lifecycle states, config/rollback, fault manager, Safe Mode logic
- REST API (OpenAPI 3.0), WebSocket telemetry, structured JSON logging
- Device profile schema + validator; first 3 approved profiles
- TPM integration, TLS 1.3, signed artifact activation, RBAC

**Key software removals vs. previous plan**
- ~~AI/ML inference pipeline~~ — removed entirely
- ~~NPU driver and ONNX runtime~~ — removed entirely
- ~~Vision pipeline / MIPI CSI camera interface~~ — removed entirely
- ~~FastDDS as baseline~~ — scoped to extension only, not required for Phase 1

**Dependencies**
- TI AM64x eval board (SK-AM64B) available from Wk 8
- TI INDUSTRIAL-SDK and PRU-ICSS EtherNet/IP firmware licensed (free under TI SDK terms)
- IO-Link software stack licensed from ADI or third party by Wk 12

**Risks**
- R5F firmware bring-up complexity for Modbus + IO-Link simultaneously → allocate dedicated R5F firmware engineer
- IPC latency between R5F and A53 must be characterised early to ensure ≤50 ms REST-to-device target

---

#### P6 — System Integration & Alpha (Wk 19–24)

**Duration:** 6 weeks | Low: 5 wks | Expected: 6 wks | High: 8 wks

**Deliverables**
- Software running on EVT hardware (not eval board)
- Reference device integration: ≥2 IO-Link devices, ≥1 Modbus RTU slave, ≥1 CAN device
- Safe Mode ≤1 s, restart ≤30 s confirmed on hardware
- REST-to-device round-trip latency ≤50 ms verified on PRU-ICSS EtherNet/IP path
- Fault isolation validated: RS-485 fault does not affect IO-Link or EtherNet/IP
- ≥3 approved device profiles in catalog

---

#### P7 — DVT & Validation (Wk 25–32)

**Duration:** 8 weeks | Low: 7 wks | Expected: 8 wks | High: 11 wks

**Deliverables**
- Protocol conformance: IO-Link v1.1, Modbus RTU, EtherNet/IP (PRU-ICSS), CANopen
- Security test report: TPM, signed boot, RBAC, TLS 1.3, rollback
- Thermal validation: 50 °C ambient, worst-case load — all junction limits met
- EMC pre-compliance: EN 55032 Class A, EN 61000-6-2, IEC 61000-4-2/4/5/6
- 72-hour burn-in report
- Full V&V evidence package

---

#### P8 — Beta Pilot & Manufacturing Prep (Wk 33–38)

**Duration:** 6 weeks | Low: 5 wks | Expected: 6 wks | High: 8 wks

**Deliverables**
- ICT/FCT manufacturing test strategy
- Phase 1 and Phase 2 BOM variant files released
- 3–5 pilot units deployed to reference customer (robot OEM or system integrator)
- Pilot feedback report; critical fixes implemented
- CE/FCC/UL certification filing initiated (parallel path, does not gate GA)

---

#### P9 — Release Candidate & Launch (Wk 39–42)

**Duration:** 4 weeks | Low: 3 wks | Expected: 4 wks | High: 6 wks

**Deliverables**
- RC freeze: final HW rev, SW version, BOM with DNP rules
- Release notes, known-limitations document, installation guide
- Sustaining engineering handoff
- GA announcement; series production order placed

---

### 6.3 Key Milestones

| ID | Target | Milestone | Success Criteria |
|---|---|---|---|
| M1 | Wk 3 | Architecture Freeze | AM64x confirmed; AI features formally removed; ADR published |
| M2 | Wk 4 | Requirements Baseline Signed | All 10 open HW requirements closed; BOM gap analysis complete |
| M3 | Wk 9 | Schematic Review Complete | All 12 sheets approved; netlist ready for layout |
| M4 | Wk 12 | PCB Released to Fabrication | Gerbers, drill files, DFM sign-off sent to board house |
| M5 | Wk 19 | EVT Hardware Qualified | Power, supervisor, AM64x, all field ports functional |
| M6 | Wk 18 | Software Alpha Release | Yocto BSP, R5F firmware, baseline protocols running on eval board |
| M7 | Wk 24 | System Alpha Complete | End-to-end demo: field device → protocol → REST on EVT hardware |
| M8 | Wk 32 | DVT Complete / Pre-compliance Pass | V&V evidence package; EMC report issued |
| M9 | Wk 38 | Beta Pilot Complete | Pilot feedback triaged; RC BOM/SW frozen |
| M10 | Wk 42 | **Production Launch (GA)** | Product released; sustaining handoff complete |

---

## 7. Migration Plan — Previous to New Roadmap

### 7.1 What Is Removed

The following items are **cancelled** and must be removed from all project documents, Jira, marketing materials, and architectural diagrams:

| Removed Item | Previous Location | Reason |
|---|---|---|
| AI Edge Computing positioning | All marketing and product docs | Not applicable to industrial protocol gateway |
| Edge AI / AI Inference references | Architecture documents, BOM rationale | No AI workload in product |
| Machine Learning pipeline | Software architecture | Cancelled — zero gateway relevance |
| Vision AI / MIPI CSI interface | HW BOM, schematic scope | Camera interface not needed |
| Generative AI references | Any | Not applicable |
| AI Acceleration / NPU | CPU selection rationale (Qualcomm QCS6490 NPU) | Qualcomm SoM removed |
| AI-enabled analytics | Product positioning | Replaced by deterministic protocol translation |
| Qualcomm QCS6490 SOM | BOM, PCB documents | AI/NPU-focused SoC; no gateway advantage |
| Lantronix Open-Q 6490CS SOM | BOM table | Qualcomm family; removed |
| Thundercomm TurboX C6490 SOM | BOM table | Qualcomm family; removed |
| SECO SOM-SMARC-QCS6490 | BOM table | Qualcomm family; removed |
| Toradex Verdin iMX8M Plus (primary) | BOM, PCB docs | Demoted to emergency fallback only |
| Variscite DART-MX8M-PLUS (primary) | BOM table | Demoted to emergency fallback only |
| 8-layer PCB spec (AI-compute-driven) | PCB guidelines | Replaced by 6-layer; single-SoC justified |
| LPDDR5 / high-bandwidth memory | HW architecture | Not applicable; AM64x uses LPDDR4 or on-chip SRAM |
| PCIe for AI accelerator | Architecture trade-off section | Removed entirely |

### 7.2 What Is Retained (Unchanged)

| Retained Item | Rationale |
|---|---|
| Hilscher netX 90 co-processor | Phase 2 EtherCAT/PROFINET still requires licensed stack |
| netX 90 SPI host interface | Interface choice unchanged |
| Supervisor MCU (updated to STM32G0) | Supervision function unchanged; G0 adequate |
| TPM SLB9672 | Secure boot and identity unchanged |
| IO-Link: MAX14819 ×2 | Field interface unchanged |
| RS-485: ISO1410 ×2 | Field interface unchanged |
| CAN: ISO1042 | Field interface unchanged |
| DIO: ISO1212 ×4 + TPS272C45 ×4 | Field interface unchanged |
| Power front-end: LM5069 + LM76005 | Power architecture unchanged |
| QNC runtime lifecycle, config, rollback | Core software architecture unchanged |
| REST API (OpenAPI 3.0) + WebSocket | Northbound interface unchanged |
| Device profile governance (YAML) | Data model architecture unchanged |
| Safe Mode (non-safety-rated) | Safety boundary statement unchanged |
| EMC targets: EN 55032 Class A, EN 61000-6-2 | Compliance targets unchanged |

### 7.3 What Is Updated

| Updated Item | Previous State | Updated State |
|---|---|---|
| Main CPU | Qualcomm QCS6490 SoM | TI AM6442 SoC (AM64x family) |
| PCB layer count | 8-layer | 6-layer |
| EtherNet/IP Phase 1 | netX 90 DNP; no native ET/IP | AM64x PRU-ICSS native EtherNet/IP adapter |
| netX 90 role | Phase 1 optional; Phase 2 full | Phase 1 DNP; Phase 2 PROFINET/EtherCAT |
| Supervisor MCU | STM32G474 | STM32G071 (simpler, lower cost) |
| Software alpha platform | Qualcomm or iMX8M-Plus eval | TI SK-AM64B evaluation kit |
| P0 duration | 4 weeks | 3 weeks (TI decision simpler) |
| P3 duration | 6 weeks (8-layer) | 5 weeks (6-layer, less routing complexity) |
| P4 duration | 8 weeks | 7 weeks (single SoC, simpler bring-up) |
| P5 duration | 12 weeks | 11 weeks (no AI stack to remove) |
| Overall schedule | Wk 44 | Wk 42–44 (recovered 2–4 wks) |

### 7.4 Jira Migration Actions

| Action | Tickets Affected | Priority |
|---|---|---|
| **CLOSE** all AI-edge, NPU, ML, vision pipeline tasks | Search: label:AI OR label:NPU OR label:ML | Immediate |
| **CLOSE** all Qualcomm SoM evaluation and BSP tasks | Search: Qualcomm OR QCS6490 OR Thundercomm OR Lantronix | Immediate |
| **CLOSE** 8-layer PCB justification tasks tied to AI payload | Search: 8-layer AND Qualcomm | Immediate |
| **UPDATE** BOM tasks: replace Qualcomm/iMX8M-Plus SoM with AM64x | P1-H01, P1-H02, BOM gap analysis | Week 1 |
| **UPDATE** schematic task scope: Sheet 02 is now AM64x SoC, not SoM | P2 schematic tasks | Week 1 |
| **UPDATE** software bring-up: target platform is SK-AM64B | P5-S01 and all BSP tasks | Week 1 |
| **ADD** new TI PRU-ICSS EtherNet/IP firmware integration task | P5 | Week 1 |
| **ADD** new R5F firmware task: IO-Link + Modbus + CANopen | P5 | Week 1 |
| **RETAIN** netX 90 Phase 2 tasks; update scope to PROFINET/EtherCAT only | P4-ETH, Phase 2 epic | Week 2 |
| **ADD** AM64x processor evaluation and socket verification task | P0 | Immediate |

### 7.5 Document Update Register

| Document | Action | Owner | Deadline |
|---|---|---|---|
| PCB.md / PCB_en.md | Remove Qualcomm references; update SoM → AM64x SoC; update layer count | HW Lead | Wk 1 |
| QNC_BOM_PCB_Update.md | Replace all BOM tables with AM64x-based BOM | HW Lead | Wk 1 |
| QNC_BOM_PCB_Qualcomm_as_main_CPU.md | **Archive / supersede entirely** | PM | Wk 1 |
| qnc_tasklist_final.md | Remove AI/NPU tasks; update Phase assignments | PM | Wk 1 |
| pcb_task.md | Update architecture conflict resolution; align to AM64x | HW Lead | Wk 2 |
| qnc_jira_task_overall.md | Update software epic to remove DDS-first assumption; add PRU-ICSS tasks | PM | Wk 2 |
| Product marketing materials | Remove all AI/ML positioning; replace with industrial gateway messaging | Marketing | Wk 2 |
| Customer-facing datasheet (draft) | Rewrite around protocol coverage, determinism, longevity | PM + Marketing | Wk 3 |

---

## 8. Risk Register

| ID | Phase | Risk | Probability | Impact | Score | Mitigation | Owner | Schedule Impact |
|---|---|---|---|---|---|---|---|---|
| R01 | P0 | AM64x FCBGA-784 PCB footprint complexity delays library sign-off | Low | Medium | 4 | Engage PCB designer in Wk 1 for footprint review; TI WEBENCH provides land pattern | HW Lead | +1 wk |
| R02 | P1 | Connector/enclosure family undecided → edge placement blocked | High | High | 8 | Lock M12 vs RJ45 decision in Wk 3; mech team attends P1 review | Mech + PM | +2–3 wks |
| R03 | P1 | DIO sourcing/sinking topology still ambiguous | Med | High | 6 | Define per-bank sourcing as default; MAX14900 fallback if per-channel needed | HW Lead | +1–2 wks |
| R04 | P2 | AM64x PRU-ICSS pin-mux conflicts with field interface signals | Med | High | 6 | Review TI SYSCONFIG pin-mux tool in P1; reserve alternate UART/SPI assignments | HW + SW | +1–2 wks |
| R05 | P3 | AM64x FCBGA-784 may require blind vias for inner-layer fanout | Med | Med | 5 | DFM review with board house by Wk 8; confirm 6-layer is sufficient | PCB Designer | +1 wk |
| R06 | P4 | EMS assembly lead time exceeds 4 weeks | Med | High | 6 | Select EMS and place NDA by Wk 10; request expedited slot | PM | +2–3 wks |
| R07 | P4 | PRU-ICSS EtherNet/IP firmware bring-up issues | Med | High | 6 | Use TI SK-AM64B eval board for SW from Wk 8; engage TI FAE early | SW + HW | +1–3 wks |
| R08 | P5 | R5F firmware complexity: IO-Link + Modbus + CANopen simultaneously | Med | High | 6 | Allocate dedicated R5F firmware engineer; phase protocol bring-up sequentially | SW Lead | +2–3 wks |
| R09 | P5 | IO-Link software stack license from ADI delayed | Med | Med | 5 | Initiate license request Wk 2; use open-source IO-Link stack as fallback for dev | SW Lead | +1–2 wks |
| R10 | P7 | EMC pre-compliance failure (conducted emissions, EFT) | Med | High | 6 | Book lab slot by Wk 23; 2-week tuning window reserved; CMC footprints in design | HW EE + Lab | +2–4 wks |
| R11 | P7 | Thermal hotspot on DO drivers or AM64x at 50 °C | Low | High | 5 | Thermal simulation in P2; thermal via arrays under power components | HW EE | +2–4 wks |
| R12 | P8 | Pilot hardware defect requiring PCB respin | Low | Critical | 6 | 72-hour burn-in in P7; fault injection testing reduces probability | PM | +6–10 wks |
| R13 | P0–P9 | Residual AI/edge positioning in customer materials creates expectation mismatch | Med | High | 6 | Mandatory document sweep in Wk 1; all materials reviewed by PM before external release | PM + Marketing | Reputation risk |
| R14 | P1 | AM64x, MAX14819, ISO1042 long lead times from distributor | Med | High | 6 | Place advance buffer order by Wk 4; verify TI warehouse stock | Procurement | +4–8 wks |
| R15 | P2 | netX 90 Phase 2 SKU not confirmed → DNP footprint must be generic | Low | Low | 2 | Use generic netX 90 DNP footprint; SKU can be locked post-P1 without schematic change | HW Lead | 0 |

**Risk Score = Probability × Impact (1–3 scale): ≥8 Critical (immediate action) · 5–7 High (tracked weekly) · ≤4 Monitor**

---

## Appendix A — Removed Features & Rationale

The following features and references are removed from the product. This list is definitive for document hygiene purposes.

| Removed Feature / Reference | Found In | Removal Rationale |
|---|---|---|
| AI Edge Computing (product positioning) | All docs | Not a gateway function |
| Edge AI | Marketing, architecture | Not applicable |
| Machine Learning | Architecture, software scope | No ML workload in gateway |
| AI Inference | BOM rationale (Qualcomm NPU), software | No inference required |
| Vision AI | Architecture options | No camera/vision interface |
| Generative AI | Any reference | Not applicable |
| AI Acceleration | BOM (Qualcomm NPU), CPU selection | CPU selection now based on fieldbus, not TFLOPS |
| AI-enabled analytics | Product positioning | Replaced by deterministic protocol telemetry |
| Qualcomm QCS6490 (NPU SoC) | BOM (primary candidate) | Oversized, AI-focused, no PRU-ICSS, lifecycle risk |
| NPU thermal budget | PCB thermal analysis | NPU removed |
| NPU + Ethernet simultaneous thermal scenario | DVT validation plan | NPU removed |
| LPDDR5 high-bandwidth memory | Architecture assumptions | Not needed for gateway |
| PCIe lane for AI accelerator | Architecture trade-off | Not applicable |
| MIPI CSI camera interface | Hardware scope | No vision pipeline |
| "AI/HMI variants" (future roadmap) | BOM rationale for Qualcomm | Removed from roadmap |

---

## Appendix B — Target Application Matrix

| Application | Required Protocols | QNC Support | Phase |
|---|---|---|---|
| Industrial robot integration | EtherNet/IP, PROFINET, IO-Link | EtherNet/IP P1; PROFINET P2 | P1/P2 |
| PLC connectivity (legacy) | Modbus RTU, Modbus TCP | ✅ P1 | P1 |
| PLC connectivity (modern) | EtherNet/IP, PROFINET | EtherNet/IP P1; PROFINET P2 | P1/P2 |
| Brownfield RS-485 retrofit | Modbus RTU → REST/MQTT | ✅ P1 | P1 |
| IO-Link sensor aggregation | IO-Link v1.1 master, REST/WebSocket | ✅ P1 | P1 |
| EtherCAT motion integration | EtherCAT slave/master | P2 via netX 90 | P2 |
| IIoT MQTT gateway | Modbus RTU → MQTT | ✅ P1 extension | P1 ext |
| OPC UA server | Multi-protocol → OPC UA | P2 extension | P2 |
| Cobot IO-Link tool interface | IO-Link master, EtherNet/IP adapter | ✅ P1 | P1 |
| CANopen robot controller | CANopen master/slave | ✅ P1 | P1 |
| Safety device aggregation | IO-Link Safety (future) | Roadmap item post-GA | Post-GA |
| Digital twin data feed | REST/WebSocket → cloud platform | ✅ P1 | P1 |

---

*Document version 2.0 — All AI/ML/Edge-AI references removed. TI AM64x selected as primary SoC. Qualcomm and AI-compute positioning retired. This document supersedes PCB.md, PCB_en.md, QNC_BOM_PCB_Update.md, and QNC_BOM_PCB_Qualcomm_as_main_CPU.md.*
