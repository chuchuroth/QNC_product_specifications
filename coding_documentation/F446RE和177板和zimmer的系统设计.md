# Smart Sorting and Assembly Demo — System Design

## 1. Overview

This document describes the **Smart Sorting and Assembly Demo**: a closed-loop, 3-layer
automation system that classifies workpieces by magnetic signature and performs
automated grip-and-relocate cycles.

| Layer | Component | Role |
|---|---|---|
| **Command & Control** | Raspberry Pi 4 | Reads sensors, runs sorting logic, commands gripper |
| **IO-Link Bridge** | P-NUCLEO-IOM01M1 (NUCLEO-F446RE + STEVAL-IOM001V1) | IO-Link Master — relays Pi commands to IO-Link devices over SPI3 slave |
| **IO-Link Devices** | IOD02A1, MAXREFDES177, Zimmer LWR50L-02 | Magnetic sensing, motor drive, gripper actuation |

---

## 2. System Architecture

```
┌──────────────────────────┐    SPI3 (70-byte frames)    ┌────────────────────────────────────────┐
│  Raspberry Pi 4          │ ◄─────────────────────────► │  NUCLEO-F446RE  (IO-Link Master)       │
│  Linux / Python 3        │  GPIO8-11 ↔ CN7 pins 24-30  │                                        │
│  raspberry_pi/           │  CE0/SCLK/MOSI/MISO         │  SPI3 slave PA15/PC10/PC11/PC12 (AF6)  │
│  - sorting_demo.py       │                             │  SPI2 → L6360 config registers         │
│  - iolink_spi.py         │                             │  USART1 → L6360 IO-Link C/Q            │
│  scripts/poc_monitor.py  │                             └──────────────┬─────────────────────────┘
└──────────────────────────┘                                            │
                                                         M12 cable │ COM2 (38.4 kBaud)
                                                    ┌───────────────────┼──────────────────────┐
                                                    │                   │                      │
                                          ┌─────────▼──────┐  ┌────────▼───────┐  ┌───────────▼──────┐
                                          │ P-NUCLEO-IOD02A1│  │ MAXREFDES177   │  │ Zimmer LWR50L-02 │
                                          │ magnetic sensor │  │ servo drive    │  │ gripper          │
                                          │ PD_in  19 B     │  │ PD_in/out 4 B  │  │ PD_in   6 B      │
                                          │ VendorID: 0x0001│  │ VendorID:0x014F│  │ PD_out 16 B      │
                                          └─────────────────┘  └────────────────┘  │VendorID: 0x0344  │
                                                                                    └──────────────────┘
```

**Sorting workflow:**
1. Pi reads IOD02A1 magnetometer via `GET_PD` — compute |B| = √(Bx²+By²+Bz²)
2. |B| > 50 µT → metallic workpiece detected
3. Pi commands MAXREFDES177 motor to move to work position
4. Pi commands Zimmer gripper to grip at 50 N → relocate → release
5. Repeat; motor health checked each cycle via MAXREFDES177 statusword

IO-Link is **point-to-point**: only **one** device is active per IO-Link port.
Devices are swapped by changing the M12 cable.

---

## 3. Hardware Wiring

### 3.1 SPI Link — Raspberry Pi ↔ NUCLEO-F446RE (CN7 Morpho Connector)

The Raspberry Pi connects directly to the NUCLEO-F446RE using the 40-pin GPIO
header and four jumper wires to the CN7 ST Morpho connector. **No RS485 HAT or
external transceiver is required.**

| Raspberry Pi GPIO | Signal    | CN7 Pin | STM32F446 Pin | SPI3 Function |
|-------------------|-----------|---------|----------------|---------------|
| GPIO 8  (CE0_N)   | CS (NSS)  | pin 24  | **PA15** AF6   | SPI3_NSS      |
| GPIO 11 (SPI_CLK) | SCLK      | pin 26  | **PC10** AF6   | SPI3_SCK      |
| GPIO 10 (MOSI)    | MOSI      | pin 28  | **PC12** AF6   | SPI3_MOSI     |
| GPIO  9 (MISO)    | MISO      | pin 30  | **PC11** AF6   | SPI3_MISO     |
| GPIO 14 (GND)     | GND       | CN10-20 | GND            | —             |

**SPI configuration:** CPOL=0, CPHA=0 (Mode 0), 8-bit MSB-first,
hardware NSS (active-low), recommended clock ≤ 1 MHz for initial testing.

> **Note on PA15/PB3/PB4:** PA15 is shared with the JTAG JTDI pin. SWD debugging
> (PA13/PA14 only) remains fully functional. The STM32 firmware configures PA15
> as SPI3_NSS (AF6), which automatically disables JTDI without affecting SWD.
> PC10/PC11/PC12 previously drove the RS485 USART3 transceiver; they are repurposed
> here as SPI3 lines. The MAX485 transceiver chip is no longer needed.

### 3.2 IO-Link Device Connections

All three IO-Link devices use a **standard M12 4-pin** connector wired as per the
IO-Link topology (IEC 61131-9):

| M12 Pin | Signal | L6360 Header |
|---|---|---|
| 1 | L+ (24 V)     | STEVAL-IOM001V1 J3 pin 1 |
| 2 | (unused / DO) | — |
| 3 | GND           | STEVAL-IOM001V1 J3 pin 3 |
| 4 | C/Q           | STEVAL-IOM001V1 J3 pin 4 |

For the MAXREFDES177, connect the field-side screw terminal (AIO/GND) to your
test voltage/current source, or leave open for idle measurements.

For the Zimmer LWR50L-02, ensure 24 V DC is available on the L+ line (check
the gripper's current draw specification against the L6360 shield's output).

## 4. Firmware Architecture

```
master/
├── Core/Src/main.c                  ← Init sequence + main loop
├── BSP/Src/bsp_iolink_master.c      ← USART1, SPI2, TIM3, GPIOs
├── BSP/Src/bsp_spi_slave.c          ← SPI3 slave: PA15(NSS), PC10(SCK), PC11(MISO), PC12(MOSI)
├── IOLink/Src/iolink_master_api.c   ← IO-Link protocol (wake-up, identity, M-sequence)
├── App/Src/app_master.c             ← State machine + device auto-detect
└── App/Src/host_comm.c              ← SPI3 request-response handler (replaces RS485)
```

### 4.1 Device Auto-Detection

After reading the IO-Link identity registers (VendorID / DeviceID), `app_master.c`
maps the identity to a `IOLink_DeviceType_t` and sets the correct PD_in / PD_out
sizes automatically:

| Device | VendorID | DeviceID | PD_in | PD_out |
|---|---|---|---|---|
| P-NUCLEO-IOD02A1 | 0x0001 | — | 19 B (`IOLink_SensorPayload_t`) | 0 B |
| MAXREFDES177      | 0x014F (TMG TE GmbH) | 0x040900 | 4 B (`IOLink_MAXREFDES177_PDIn_t`) | 4 B |
| Zimmer LWR50F     | 0x0344 (Zimmer GmbH) | 0xFFFFFF ¹ | 6 B (`IOLink_ZimmerGripper_PDIn_t`) | 16 B (`IOLink_ZimmerGripper_PDOut_t`) |

> ¹ LWR50F DeviceID is unknown — available IODDs are for the LWR50L series.
> The VendorID (0x0344) is authoritative. DeviceID will be read from the
> connected device at runtime and can be logged for future reference.

### 4.2 Master State Machine

```
WAKEUP → READ_IDENTITY → auto-detect device → PREOPERATE → OPERATE → CYCLIC
                                    ↑ push status to RPi via SPI3
                                                                  ↕ HostComm_Poll()
                                                            (SPI3 req-resp handler,
                                                             1 ms non-blocking)
```

The 2 ms TIM3 IO-Link cycle and the SPI3 host-comm poll are both serviced in
the `CYCLIC` state. `HostComm_Poll` uses a 1 ms SPI receive timeout; if the Pi
does not initiate a transfer within that window, it returns immediately without
blocking the IO-Link cycle.

---

## 5. SPI Host Communication Protocol

### 5.1 Frame Format

Both request and response frames share the same layout:

```
Byte 0 : SOF  — 0xAA = request (RPi→STM32), 0xBB = response (STM32→RPi)
Byte 1 : CMD  — command code (see table below)
Byte 2 : LEN  — number of payload bytes that follow (not including CRC)
Byte 3..N : PAYLOAD
Byte N+1  : CRC8 — computed over [CMD, LEN, PAYLOAD...]
```

Frames are **zero-padded** to `HOST_SPI_FRAME_SIZE = 70 bytes` for fixed-length SPI
transactions. The Pi always transfers exactly 70 bytes per transaction.

**CRC8 polynomial:** Dallas/Maxim 0x31 (x⁸ + x⁵ + x⁴ + 1), initial value 0x00.

### 5.2 Two-Transaction Exchange

```
Time ═════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════Ｔ
 Pi      CS ─┬────────────────────────┬──────────────────────────────┬────────────────────────┬─
 Pi  MOSI ─┤ [SOF][CMD][LEN][PL...][CRC] ├────────────── gap ──────────────┤ [0x00 × 70]─────────────┬─
 Pi  MISO ─┤ [0x00 × 70 -- ignored ---]├─────────────────────────────┤ [SOF][CMD][LEN][ST][PL...][CRC]┙
                   Transaction 1 (request)       ≥ 2 ms  Transaction 2 (response)
```

### 5.3 Commands

| CMD  | Name          | Request payload | Response payload |
|------|---------------|-----------------|------------------|
| 0x01 | GET_STATUS    | none | `[port_status, vid_h, vid_l, did_h, did_m, did_l, rev]` |
| 0x02 | GET_PD        | none | `[len_in, data_in[0..len_in-1]]` |
| 0x03 | SET_PD_OUT    | `[len_out, data_out[0..len_out-1]]` | none |
| 0x04 | READ_PARAM    | `[addr]` | `[value]` |
| 0x05 | WRITE_PARAM   | `[addr, value]` | none |
| 0x06 | DEVICE_RESET  | none | none |
| 0xFF | PING          | none | `[0xAC]` |

All responses begin with a STATUS byte prepended by the STM32 (see `host_comm.h`).

### 5.4 Unsolicited Pushes

The STM32 loads an unsolicited GET_STATUS response into its SPI TX buffer whenever:
- A new IO-Link device is detected and enters PREOPERATE
- The port transitions to OPERATE

The *next* Pi transaction will receive this push as the SPI response payload.

---

## 6. Raspberry Pi Software

### 6.1 raspberry_pi/iolink_spi.py — SPI Transport Layer

Provides the `IOLinkSPI` class which wraps `spidev` into the host-comm protocol:
- Builds request frames with CRC8
- Executes the two-transaction SPI exchange with configurable processing delay
- Parses and validates response frames

### 6.2 raspberry_pi/sorting_demo.py — Smart Sorting Application

The main Command & Control application implementing the closed-loop sorting workflow:

```bash
# Install dependency
pip install spidev

# Enable SPI on Raspberry Pi
# In /boot/config.txt (or via raspi-config): dtparam=spi=on

# Run the demo with default parameters (threshold=50 µT, force=50 N)
python3 raspberry_pi/sorting_demo.py

# Custom parameters
python3 raspberry_pi/sorting_demo.py --threshold 40 --force 60 --cycles 50 --verbose
```

### 6.3 scripts/poc_monitor.py — Diagnostic Utility

Low-level CLI tool for manual inspection and debugging (uses SPI transport):

```bash
# Verify STM32 master is alive
python3 scripts/poc_monitor.py --spi-bus 0 --spi-dev 0 ping

# Read port status + device identity
python3 scripts/poc_monitor.py --spi-bus 0 --spi-dev 0 status

# Continuous process-data monitor
python3 scripts/poc_monitor.py --spi-bus 0 --spi-dev 0 monitor --interval 0.2

# Grip at 60 N (Zimmer)
python3 scripts/poc_monitor.py --spi-bus 0 --spi-dev 0 grip --force 60

# Read parameter register 0x07 (VendorID MSB)
python3 scripts/poc_monitor.py --spi-bus 0 --spi-dev 0 param-read 0x07
```

---

## 7. Process Data Structures

### 7.1 P-NUCLEO-IOD02A1 (PD_in = 19 bytes)

```c
typedef struct __attribute__((packed)) {
    int16_t  accel_x, accel_y, accel_z;   // mg
    int16_t  gyro_x,  gyro_y,  gyro_z;    // mdps
    int16_t  mag_x,   mag_y,   mag_z;     // mGauss
    uint8_t  status;
} IOLink_SensorPayload_t;
```

### 7.2 MAXREFDES177 (PD_in = 4 B, PD_out = 4 B)

```c
// PD_in
typedef struct __attribute__((packed)) {
    int16_t  adc_value;    // ±32000 ↔ ±10 V or ±20 mA
    uint8_t  in_channel;   // 0=V-in, 1=I-in, 2=AI5/RTD, 3=AI6/RTD
    uint8_t  status;       // bit0=valid, bit1=overflow, bit2=error
} IOLink_MAXREFDES177_PDIn_t;

// PD_out
typedef struct __attribute__((packed)) {
    int16_t  dac_value;    // ±32000 ↔ ±10 V or ±20 mA
    uint8_t  out_channel;  // 0=V-out, 1=I-out
    uint8_t  cmd_flags;    // bit0=output_enable, bit1=update
} IOLink_MAXREFDES177_PDOut_t;
```

> ⚠️ Verify exact byte layout and full-scale mapping from the official IODD file
> (`maxrefdes177-iodd.zip` on the Analog Devices product page).

### 7.3 Zimmer LWR50F (PD_in = 4 B, PD_out = 2 B)

```c
// PD_out
typedef struct __attribute__((packed)) {
    uint8_t  command;       // bit0=GRIP, bit1=RELEASE, bit2=STOP, bit3=HOME
    uint8_t  target_force;  // 0–100 % of rated force
} IOLink_ZimmerGripper_PDOut_t;

// PD_in
typedef struct __attribute__((packed)) {
    uint8_t  status;        // bit0=POS_REACHED, bit1=GRIPPED, bit2=NO_PART, bit3=ERR
    uint8_t  jaw_position;  // 0=open … 100=closed (%)
    uint8_t  error_code;
    uint8_t  diag;
} IOLink_ZimmerGripper_PDIn_t;
```

> ⚠️ IODD files in `common/IODDs/` are for the **LWR50L** series.
> The Zimmer VendorID is **0x0344** (Zimmer GmbH, verified from IODD XML).
> The LWR50F DeviceID is not yet confirmed — read it from the live device
> with `python3 scripts/poc_monitor.py --spi-bus 0 --spi-dev 0 status`.
> PD layouts (6 B in / 16 B out) are verified from the LWR50L IODD and are
> expected to be identical for LWR50F (same gripper family).
> Controlword bit definitions require the Zimmer LWR50 IO-Link programming manual.

---

## 8. Step-by-Step PoC Verification Checklist

### Phase 1 — P-NUCLEO-IOD02A1 (sensor node, safest to test first)

- [ ] Flash master firmware to NUCLEO-F446RE (`make -C master flash`)
- [ ] Flash device firmware to NUCLEO-L452RE  (`make -C device flash`)
- [ ] Connect M12 cable between master shield and IOD02A1
- [ ] Power both boards via USB
- [ ] Open minicom/PuTTY on debug UART (USART2, PA2/PA3, 115200)
- [ ] Observe: `[APP] IO-Link OPERATE — starting cyclic PD`
- [ ] Observe sensor data lines on debug UART every 200 ms
- [ ] On RPi: `python3 scripts/poc_monitor.py --spi-bus 0 --spi-dev 0 ping`
- [ ] Confirm VendorID, accel/gyro/mag readings displayed on RPi

### Phase 2 — MAXREFDES177 (analog I/O)

- [ ] Disconnect IOD02A1 cable; connect MAXREFDES177 M12 cable
- [ ] Confirm L+ 24 V is available on the shield (check jumper JP3 on STEVAL-IOM001V1)
- [ ] Observe debug UART: `Device: MAXREFDES177`
- [ ] On RPi: `python3 scripts/poc_monitor.py --spi-bus 0 --spi-dev 0 status`
  - VendorID should show **0x014F** (TMG TE GmbH — IO-Link stack on MAXREFDES177)
- [ ] Connect 0–10 V signal to MAXREFDES177 AIO terminal
- [ ] On RPi: `python3 scripts/poc_monitor.py --spi-bus 0 --spi-dev 0 monitor --interval 0.2`
  - Verify voltage reading tracks applied signal

### Phase 3 — Zimmer LWR50F-00-04-A (gripper)

> ⚠️ Confirm PD byte layout from IODD **before** sending grip commands.
> Incorrect PD_out may cause unexpected gripper movement.

- [ ] Ensure adequate 24 V supply current for gripper (check LWR50F datasheet)
- [ ] Disconnect MAXREFDES177; connect Zimmer M12 cable
- [ ] Observe debug UART: `Device: Zimmer LWR50F gripper`
- [ ] On RPi: `python3 scripts/poc_monitor.py --spi-bus 0 --spi-dev 0 status`
  - VendorID should show **0x0344** (Zimmer GmbH); note the actual LWR50L DeviceID for records
- [ ] On RPi: `python3 scripts/poc_monitor.py --spi-bus 0 --spi-dev 0 monitor` — observe jaw position
- [ ] On RPi: `python3 scripts/poc_monitor.py --spi-bus 0 --spi-dev 0 grip --force 50`
  - Watch jaw close and `gripped` flag set in monitor
- [ ] On RPi: `python3 scripts/poc_monitor.py --spi-bus 0 --spi-dev 0 release`
  - Watch jaw open

---

## 9. Files Added / Modified

| File | Change |
|---|---|
| `common/iolink_types.h` | Added `IOLink_MAXREFDES177_PDIn/Out_t`, `IOLink_ZimmerGripper_PDIn/Out_t`, `IOLink_DeviceType_t`, vendor/device ID constants |
| `master/BSP/Inc/bsp_spi_slave.h` | **New** — SPI3 slave BSP header (PA15/PC10/PC11/PC12, AF6) |
| `master/BSP/Src/bsp_spi_slave.c` | **New** — SPI3 slave BSP implementation |
| `master/App/Inc/host_comm.h` | **New** — SPI3 host-comm protocol header |
| `master/App/Src/host_comm.c` | **New** — SPI3 request-response handler |
| `master/App/Src/app_master.c` | Updated: device auto-detect, per-device PD sizing, host_comm integration |
| `master/Makefile` | Added `bsp_spi_slave.c` and `host_comm.c` to C_SOURCES |
| `raspberry_pi/iolink_spi.py` | **New** — SPI transport + host-comm protocol (`IOLinkSPI` class) |
| `raspberry_pi/sorting_demo.py` | **New** — Smart sorting main application (`SmartSortingDemo` class) |
| `raspberry_pi/requirements.txt` | **New** — `spidev>=3.5` dependency |
| `scripts/poc_monitor.py` | **New** — Raspberry Pi diagnostic CLI (SPI transport) |
| `docs/poc_system_design.md` | **New** — this document |

---

## 10. Known Limitations & Next Steps

| Item | Detail |
|---|---|
| Single IO-Link port | Cable swap required between devices; no simultaneous multi-device |
| MAXREFDES177 IODD | PD byte layout approximated — download and verify against official IODD |
| Zimmer IODD | PD byte layout approximated — download from ioddfinder.io and verify |
| SPI clock speed | Start at 1 MHz; check signal integrity before increasing |
| Gripper 24 V | L6360 shield L+ may not supply enough peak current; use external 24 V PSU |
| Multi-port expansion | Replace L6360 shield with MAX14819 (dual IO-Link) to reach two devices simultaneously |

---
---
The "Smart Sorting and Assembly Demo" is an integrated industrial automation test case designed to showcase a **closed-loop system** that encompasses perception, data collection, logic decision-making, and mechanical execution. By incorporating a **Raspberry Pi**, the system transitions from a PC-based setup to a compact, standalone **embedded control architecture**.

### **Integrated System Architecture**
The system is organized into a three-layer hierarchy connected via **IO-Link** and **SPI protocols**:

*   **Command & Control Layer (The Brain):** The **Raspberry Pi** serves as the host controller, running application logic and managing communications.
*   **Interface Layer (The Bridge):** The **P-NUCLEO-IOM01M1** acts as the IO-Link Master, receiving SPI commands from the Pi and translating them into IO-Link protocols for the connected devices.
*   **Peripheral Layer (The Execution):**
    *   **P-NUCLEO-IOD02A1:** A sensor node utilizing a magnetometer to identify workpieces.
    *   **MAXREFDES177:** A universal analog I/O node acting as a data logger to monitor motor voltage and temperature.
    *   **Zimmer LWR50L-02:** A servo-electric parallel gripper that performs physical movements and provides real-time status feedback.

### **Hardware and Software Integration**
The Raspberry Pi integrates with the Master board through a high-speed **SPI interface** using the **ST morpho connectors (CN7 and CN10)**. The specific pin mapping is as follows:
*   **SPI Clock (SCLK):** Pi GPIO 11 to CN7 Pin 26.
*   **MOSI:** Pi GPIO 10 to CN7 Pin 28.
*   **MISO:** Pi GPIO 9 to CN7 Pin 30.
*   **Chip Select (CS):** Pi GPIO 8 to CN7 Pin 24.
*   **Ground (GND):** Pi GPIO 14 to CN10 Pin 20.

To control this hardware, the system utilizes the **UM2421 "Low-Level IO-Link Master Access"** framework. This includes a library of **Low-Level APIs** that allow the Pi to send protocol frames over SPI, and a high-level **Python** program to execute the specific sorting logic.

### **Operational Workflow**
The demo simulates an automated sorting line with the following steps:
1.  **Identification:** Workpieces pass the **IOD02A1** sensor. Metal parts ("Type A") are detected by a change in magnetic field strength, which is reported to the Pi in real-time.
2.  **Decision:** If the magnetic field strength exceeds a threshold (e.g., 50 µT), the Pi identifies it as "Type A"; otherwise, it is "Type B" (plastic).
3.  **Execution:** For metal parts, the Pi commands the **Zimmer gripper** to move, grab with a set force (e.g., 50N), and relocate the part. Plastic parts are ignored and pass through the line.
4.  **Monitoring:** During execution, the **MAXREFDES177** logs the gripper's motor current and temperature, while the gripper itself provides feedback on its position and actual force to ensure stability.

### **System Significance**
This framework demonstrates **Multi-Vendor Interoperability**, allowing components from **ST, ADI, and Zimmer** to work together seamlessly under the IO-Link standard. It features **Software-Defined Hardware**, enabling users to remotely adjust parameters like gripping force or sensor modes without physical wiring changes. Ultimately, it provides the computational power for **real-time data analysis** and **precision control** in a compact format.
