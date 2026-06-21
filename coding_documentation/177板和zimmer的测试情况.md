# IO-Link Device Reference — MAXREFDES177 & Zimmer LWR50F-00-04-A

This document summarises the role, specifications, and integration notes for two
additional IO-Link devices that can be used alongside the P-NUCLEO-IOM01M1 master
evaluated in this project.

---

## 1. MAXREFDES177 — IO-Link Universal Analog I/O Module

**Manufacturer:** Analog Devices (formerly Maxim Integrated)  
**Product page:** <https://www.analog.com/en/resources/reference-designs/maxrefdes177.html>  
**IO-Link role:** **Device**  
**IO-Link version:** v1.1 (IEC 61131-9 compliant)  
**Form factor:** Industrial PCB module, 61 mm × 25 mm, M12 4-pin connector  

### 1.1 Key ICs

| IC | Function |
|---|---|
| **MAX22515** | IO-Link v1.1 device transceiver — integrated surge & reverse-polarity protection, dual LDO (3.3 V / 5 V), low R_on C/Q driver |
| **MAX22000** | Industrial configurable analog I/O — voltage/current input + output, PGA for RTD inputs |
| **MAX14483** | Digital isolator — galvanic isolation between IO-Link side and analog field side |
| **MAX13256 / MAXM17552** | Isolated DC-DC converter — field-side power derived from L+ (24 V) |
| **Atmel ATSAM** | On-board MCU — pre-flashed with TMG TE IO-Link device stack; **no external host MCU required** |

### 1.2 Analog I/O Capabilities

| Mode | Range | Accuracy |
|---|---|---|
| Voltage input | ±10 V | ≤ 0.1 % over ±50 °C |
| Current input | ±20 mA | ≤ 0.1 % over ±50 °C |
| Voltage output | ±10 V | ≤ 0.1 % over ±50 °C |
| Current output | ±20 mA | ≤ 0.1 % over ±50 °C |
| Temperature (RTD) | PT100 / PT1000 | via MAX22000 PGA on AI5/AI6 |

Linear range is set at **105 %** of nominal; full-scale range at **125 %**.

### 1.3 Connectivity

- **IO-Link side:** male M12 4-pin connector → standard M12 IO-Link cable to master
- **Field side:** 4-way PCB screw terminal (AIO, GND, AI5/RTD, AI6/RTD)
- **Power:** drawn from L+ (24 V); no separate field-side supply needed

### 1.4 Integration Notes

- Connects directly to **any IO-Link v1.1-compliant master** — including the P-NUCLEO-IOM01M1
- IODD file available on the [Analog Devices product page](https://www.analog.com/media/en/reference-design-documentation/design-integration-files/maxrefdes177-iodd.zip)
- Can be evaluated standalone via a **MAXREFDES165#** USB IO-Link master + PC software

---

## 2. Zimmer LWR50F-00-04-A — IO-Link Compact Parallel Gripper

**Manufacturer:** Zimmer Group  
**Part number:** LWR50F-00-04-A (`-04-A` suffix = IO-Link interface)  
**IO-Link role:** **Device** (actuator)  
**IO-Link version:** v1.1  
**IODD:** available via [ioddfinder.io](https://ioddfinder.io) — search `LWR50F`

### 2.1 Key Characteristics

| Parameter | Value |
|---|---|
| Type | Compact parallel gripper |
| Interface | IO-Link v1.1 via M12 4-pin |
| Gripping force | Adjustable via IO-Link parameter |
| Feedback | Jaw position + status via process data |
| Supply voltage | 24 V DC (from IO-Link L+) |

### 2.2 Process Data (approximate — confirm from IODD)

| Direction | Content |
|---|---|
| **PD_out** (master → gripper) | Grip/release command byte, target force/position setpoint |
| **PD_in** (gripper → master) | Current jaw position, grip status flags (gripped / no-part / error) |

> ⚠️ Always import the official IODD from ioddfinder.io to get the exact byte layout,
> VendorID, and DeviceID before coding the master side. Zimmer VendorID is typically
> **0x01F4**; confirm DeviceID for LWR50F in the IODD.

---

## 3. System-Level Role Summary

| Device | IO-Link Role | MCU / Stack |
|---|---|---|
| P-NUCLEO-IOM01M1 (STM32F446RE + L6360) | **Master** | Custom firmware (this repo) |
| P-NUCLEO-IOD02A1 (STM32L452RE + L6364Q + IKS02A1) | **Device** — sensor node | Custom firmware (this repo) |
| MAXREFDES177 (ATSAM + MAX22515) | **Device** — analog I/O | On-board ATSAM + TMG TE stack |
| Zimmer LWR50F-00-04-A | **Device** — gripper actuator | Built-in controller |

---

## 4. Topology Constraints — IO-Link Is Point-to-Point

> IO-Link is **not a fieldbus**. Each master port drives exactly **one** device.
> The P-NUCLEO-IOM01M1 has a single IO-Link port.

### Option A — One device at a time (simplest)

```
P-NUCLEO-IOM01M1 ──C/Q──→  IOD02A1          (swap cable to switch device)
                       or→  MAXREFDES177
                       or→  Zimmer LWR50F
```

Requires only a cable swap; firmware PD format must match the connected device's IODD.

### Option B — Multi-device with MAXREFDES165# (4-port USB master)

```
MAXREFDES165# (USB, 4 ports)
  port 1 ──C/Q──→ P-NUCLEO-IOD02A1     (9-axis sensor node)
  port 2 ──C/Q──→ MAXREFDES177         (analog I/O)
  port 3 ──C/Q──→ Zimmer LWR50F        (gripper)
```

The MAXREFDES165# connects to a PC via USB; control software provided by ADI.
The STM32 master is not involved in this topology.

### Option C — STM32 master + multi-port IO-Link hub

Replace the single-port L6360 shield with a multi-port IO-Link master IC
(e.g. MAX14819 dual-channel) or add a commercial IO-Link hub between the STM32
master and the devices.

---

## 5. Firmware Adaptation Required (Option A)

When connecting the STM32 master to **MAXREFDES177** or **Zimmer LWR50F** instead of
the IOD02A1 sensor node, the following must change in the master firmware:

| File | Change |
|---|---|
| `common/iolink_types.h` | Replace `IOLink_SensorPayload_t` with the new device's PD struct |
| `master/IOLink/Src/iolink_master_api.c` | Update expected VendorID / DeviceID in `IOLink_Master_ReadIdentity()` |
| `master/App/Src/app_master.c` | Update cyclic PD_out content and PD_in parsing to match device IODD |

The wake-up sequence, parameter handshake, and M-sequence framing are identical
for all IO-Link v1.1 devices and do **not** need to change.

---

## 6. References

| Resource | Link |
|---|---|
| MAXREFDES177 product page | <https://www.analog.com/en/resources/reference-designs/maxrefdes177.html> |
| MAXREFDES177 IODD download | <https://www.analog.com/media/en/reference-design-documentation/design-integration-files/maxrefdes177-iodd.zip> |
| MAX22515 datasheet | <https://www.analog.com/en/products/max22515.html> |
| MAX22000 datasheet | <https://www.analog.com/en/products/max22000.html> |
| MAXREFDES165# (4-port USB master) | <https://www.analog.com/en/resources/reference-designs/maxrefdes165.html> |
| Zimmer Group LWR50F product page | <https://www.zimmer-group.com/en/products/gripping-technology/parallel-grippers/lwr50f> |
| IODD Finder | <https://ioddfinder.io> |
| IO-Link Specification v1.1 | IEC 61131-9 |

---
---
# IO-Link Protocol Library

Drivers and utilities for IO-Link communication via MAX22515 transceiver.

## Contents

### drivers/
Python drivers for IO-Link gripper control:
- `zimmer_gripper_gpiod.py` - PIN mode driver using gpiod (recommended)
- `zimmer_gripper_pin_mode.py` - PIN mode driver using RPi.GPIO (backup)

### scripts/
Scanner utilities:
- `scan_iolink.py` - UART-based IO-Link scanner
- `scan_iolink_spi.py` - SPI-based IO-Link scanner
- `scan_zimmer_iolink.py` - I2C + UART scanner for MAX22515

### docs/
- `MAX22515.pdf` - MAX22515 IO-Link transceiver datasheet
- `MAX22515.txt` - Extracted text from datasheet

### iodd/
IO Device Description files:
- `Zimmer-LWR50L-02-00002-A-20240219-IODD1.1.xml` - Zimmer LWR50L gripper IODD

## Hardware Configuration

### MAX22515 IO-Link Master

**GPIO Connections:**
- GPIO 2 (SDA) → MAX22515 SDA (I2C mode)
- GPIO 3 (SCL) → MAX22515 SCL (I2C mode)
- GPIO 8 (TX) → MAX22515 TX (UART)
- GPIO 9 (RX) → MAX22515 RX (UART)
- GPIO 17 → MAX22515 TXEN (transmit enable)
- GPIO 27 → MAX22515 WU/IRQ (wake-up/interrupt)

**Power Supply:**
- V24: 24V supply input
- LIN: 8-36V input for regulator
- V5: 5V output (internal regulator)
- GND: Ground

### Zimmer LWR50L Gripper

**Process Data:**
- Input (6 bytes): Statusword, Diagnose, Position
- Output (16 bytes): Control, Parameters, Positions

**IO-Link Parameters:**
- Vendor ID: 836 (0x0344)
- Device ID: 133126 (0x02080006)
- COM modes: COM1 (4.8k), COM2 (38.4k), COM3 (230.4k)

## Usage

### Prerequisites
```bash
# Install dependencies
sudo apt-get install python3-serial python3-libgpiod
```

### Scan for Gripper (PIN Mode - Recommended)
```bash
cd iolink/drivers
sudo python3 zimmer_gripper_gpiod.py
```

### Scan for Gripper (I2C Mode)
```bash
cd iolink/scripts
python3 scan_zimmer_iolink.py
```

### Scan with SPI
```bash
cd iolink/scripts
python3 scan_iolink_spi.py
```

## MAX22515 Operating Modes

### PIN Mode (I2C/PIN = LOW)
- Controlled via GPIO pins directly
- No I2C configuration needed
- GPIO 17 (TXEN) enables C/Q driver
- Simpler setup

### I2C Mode (I2C/PIN = HIGH)
- Full configuration via I2C interface
- I2C address: 0x34 (A0=0) or 0x35 (A0=1)
- Advanced control and monitoring
- Requires I2C initialization

## Troubleshooting

**No data received:**
1. Verify MAX22515 power supply (V24, V5)
2. Check EN/POK pin is HIGH
3. Verify I2C/PIN pin setting matches driver mode
4. Check Zimmer gripper power and connection
5. Verify GPIO 17 (TXEN) is HIGH for transmission
6. Check UART connections (GPIO 8/9)

**GPIO errors:**
- Run with sudo: `sudo python3 <script>`
- Check GPIO permissions
- Verify gpiod installation

**I2C not detected:**
- Verify I2C/PIN pin is HIGH for I2C mode
- Check I2C connections (GPIO 2/3)
- Run: `i2cdetect -y 1`
- Check MAX22515 power

## Protocol Details

IO-Link communication characteristics:
- **Physical Layer**: 24V industrial line (C/Q)
- **UART Parity**: Even
- **Data bits**: 8
- **Stop bits**: 1
- **Baudrates**: 4800 (COM1), 38400 (COM2), 230400 (COM3)

## References

- MAX22515 Datasheet: `docs/MAX22515.pdf`
- Zimmer IODD: `iodd/Zimmer-LWR50L-02-00002-A-20240219-IODD1.1.xml`
- IO-Link Specification: www.io-link.com

---
---
# IO-Link Device Reference — MAXREFDES177 & Zimmer LWR50F-00-04-A

This document summarises the role, specifications, and integration notes for two
additional IO-Link devices that can be used alongside the P-NUCLEO-IOM01M1 master
evaluated in this project.

---

## 1. MAXREFDES177 — IO-Link Universal Analog I/O Module

**Manufacturer:** Analog Devices (formerly Maxim Integrated)  
**Product page:** <https://www.analog.com/en/resources/reference-designs/maxrefdes177.html>  
**IO-Link role:** **Device**  
**IO-Link version:** v1.1 (IEC 61131-9 compliant)  
**Form factor:** Industrial PCB module, 61 mm × 25 mm, M12 4-pin connector  

### 1.1 Key ICs

| IC | Function |
|---|---|
| **MAX22515** | IO-Link v1.1 device transceiver — integrated surge & reverse-polarity protection, dual LDO (3.3 V / 5 V), low R_on C/Q driver |
| **MAX22000** | Industrial configurable analog I/O — voltage/current input + output, PGA for RTD inputs |
| **MAX14483** | Digital isolator — galvanic isolation between IO-Link side and analog field side |
| **MAX13256 / MAXM17552** | Isolated DC-DC converter — field-side power derived from L+ (24 V) |
| **Atmel ATSAM** | On-board MCU — pre-flashed with TMG TE IO-Link device stack; **no external host MCU required** |

### 1.2 Analog I/O Capabilities

| Mode | Range | Accuracy |
|---|---|---|
| Voltage input | ±10 V | ≤ 0.1 % over ±50 °C |
| Current input | ±20 mA | ≤ 0.1 % over ±50 °C |
| Voltage output | ±10 V | ≤ 0.1 % over ±50 °C |
| Current output | ±20 mA | ≤ 0.1 % over ±50 °C |
| Temperature (RTD) | PT100 / PT1000 | via MAX22000 PGA on AI5/AI6 |

Linear range is set at **105 %** of nominal; full-scale range at **125 %**.

### 1.3 Connectivity

- **IO-Link side:** male M12 4-pin connector → standard M12 IO-Link cable to master
- **Field side:** 4-way PCB screw terminal (AIO, GND, AI5/RTD, AI6/RTD)
- **Power:** drawn from L+ (24 V); no separate field-side supply needed

### 1.4 Integration Notes

- Connects directly to **any IO-Link v1.1-compliant master** — including the P-NUCLEO-IOM01M1
- IODD file available on the [Analog Devices product page](https://www.analog.com/media/en/reference-design-documentation/design-integration-files/maxrefdes177-iodd.zip)
- Can be evaluated standalone via a **MAXREFDES165#** USB IO-Link master + PC software

---

## 2. Zimmer LWR50F-00-04-A — IO-Link Compact Parallel Gripper

**Manufacturer:** Zimmer Group  
**Part number:** LWR50F-00-04-A (`-04-A` suffix = IO-Link interface)  
**IO-Link role:** **Device** (actuator)  
**IO-Link version:** v1.1  
**IODD:** available via [ioddfinder.io](https://ioddfinder.io) — search `LWR50F`

### 2.1 Key Characteristics

| Parameter | Value |
|---|---|
| Type | Compact parallel gripper |
| Interface | IO-Link v1.1 via M12 4-pin |
| Gripping force | Adjustable via IO-Link parameter |
| Feedback | Jaw position + status via process data |
| Supply voltage | 24 V DC (from IO-Link L+) |

### 2.2 Process Data (approximate — confirm from IODD)

| Direction | Content |
|---|---|
| **PD_out** (master → gripper) | Grip/release command byte, target force/position setpoint |
| **PD_in** (gripper → master) | Current jaw position, grip status flags (gripped / no-part / error) |

> ⚠️ Always import the official IODD from ioddfinder.io to get the exact byte layout,
> VendorID, and DeviceID before coding the master side. Zimmer VendorID is typically
> **0x01F4**; confirm DeviceID for LWR50F in the IODD.

---

## 3. System-Level Role Summary

| Device | IO-Link Role | MCU / Stack |
|---|---|---|
| P-NUCLEO-IOM01M1 (STM32F446RE + L6360) | **Master** | Custom firmware (this repo) |
| P-NUCLEO-IOD02A1 (STM32L452RE + L6364Q + IKS02A1) | **Device** — sensor node | Custom firmware (this repo) |
| MAXREFDES177 (ATSAM + MAX22515) | **Device** — analog I/O | On-board ATSAM + TMG TE stack |
| Zimmer LWR50F-00-04-A | **Device** — gripper actuator | Built-in controller |

---

## 4. Topology Constraints — IO-Link Is Point-to-Point

> IO-Link is **not a fieldbus**. Each master port drives exactly **one** device.
> The P-NUCLEO-IOM01M1 has a single IO-Link port.

### Option A — One device at a time (simplest)

```
P-NUCLEO-IOM01M1 ──C/Q──→  IOD02A1          (swap cable to switch device)
                       or→  MAXREFDES177
                       or→  Zimmer LWR50F
```

Requires only a cable swap; firmware PD format must match the connected device's IODD.

### Option B — Multi-device with MAXREFDES165# (4-port USB master)

```
MAXREFDES165# (USB, 4 ports)
  port 1 ──C/Q──→ P-NUCLEO-IOD02A1     (9-axis sensor node)
  port 2 ──C/Q──→ MAXREFDES177         (analog I/O)
  port 3 ──C/Q──→ Zimmer LWR50F        (gripper)
```

The MAXREFDES165# connects to a PC via USB; control software provided by ADI.
The STM32 master is not involved in this topology.

### Option C — STM32 master + multi-port IO-Link hub

Replace the single-port L6360 shield with a multi-port IO-Link master IC
(e.g. MAX14819 dual-channel) or add a commercial IO-Link hub between the STM32
master and the devices.

---

## 5. Firmware Adaptation Required (Option A)

When connecting the STM32 master to **MAXREFDES177** or **Zimmer LWR50F** instead of
the IOD02A1 sensor node, the following must change in the master firmware:

| File | Change |
|---|---|
| `common/iolink_types.h` | Replace `IOLink_SensorPayload_t` with the new device's PD struct |
| `master/IOLink/Src/iolink_master_api.c` | Update expected VendorID / DeviceID in `IOLink_Master_ReadIdentity()` |
| `master/App/Src/app_master.c` | Update cyclic PD_out content and PD_in parsing to match device IODD |

The wake-up sequence, parameter handshake, and M-sequence framing are identical
for all IO-Link v1.1 devices and do **not** need to change.

---

## 6. References

| Resource | Link |
|---|---|
| MAXREFDES177 product page | <https://www.analog.com/en/resources/reference-designs/maxrefdes177.html> |
| MAXREFDES177 IODD download | <https://www.analog.com/media/en/reference-design-documentation/design-integration-files/maxrefdes177-iodd.zip> |
| MAX22515 datasheet | <https://www.analog.com/en/products/max22515.html> |
| MAX22000 datasheet | <https://www.analog.com/en/products/max22000.html> |
| MAXREFDES165# (4-port USB master) | <https://www.analog.com/en/resources/reference-designs/maxrefdes165.html> |
| Zimmer Group LWR50F product page | <https://www.zimmer-group.com/en/products/gripping-technology/parallel-grippers/lwr50f> |
| IODD Finder | <https://ioddfinder.io> |
| IO-Link Specification v1.1 | IEC 61131-9 |
