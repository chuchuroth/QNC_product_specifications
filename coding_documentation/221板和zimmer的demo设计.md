
# Direct RPi5 + TIOL221EVM Gripper Setup
## Zimmer LWR50L-02 via IO-Link (Simplest Setup, No Nucleo Required)

---

## Overview

```
Raspberry Pi 5
 ├─ UART0 (GPIO 14/15)  ──►  TIOL221EVM  ──►  IO-Link 24 V C/Q  ──►  LWR50L-02
 ├─ GPIO17 (EN1)        ──►  TIOL221EVM
 └─ GPIO27 (WAKE)       ──►  TIOL221EVM
```

The TIOL221EVM acts as the IO-Link physical-layer transceiver.
The RPi5 implements the IO-Link master protocol in Python.

---

## 1 · Hardware wiring

### 1.1 Power supplies

| Connection | Notes |
|---|---|
| 24 V DC PSU (+) → J1/J2 pin L+_24V | 1 A minimum |
| 24 V DC PSU (−) → J1/J2 pin GND | common ground |
| Gripper M12 cable → J2 (M12 IO-Link connector) | 4-pin Class A |

### 1.2 RPi5 GPIO → TIOL221EVM (LP header J10 / J11)

| RPi5 (BCM) | RPi5 physical pin | TIOL221EVM | Signal |
|---|---|---|---|
| GPIO 14 (UART0 TX) | Pin  8 | J10 LP-pin 4  | CQ_TX_LP |
| GPIO 15 (UART0 RX) | Pin 10 | J10 LP-pin 3  | CQ_RX_LP |
| GPIO 17            | Pin 11 | J10 LP-pin 5  | CQ_EN_LP (EN1) |
| GPIO 27            | Pin 13 | J11 LP-pin 11 | WAKE_LP |
| 3.3 V              | Pin  1 | J10 LP-pin 1  | 3V3_LP |
| GND                | Pin  6 | J10 LP-pin 12 | GND |

> **LP-pin numbering** counts across both J10 and J11 headers:
> J10 carries LP pins 1–20; J11 carries LP pins 21–40.
> LP-pin 11 on J11 = 11 − 20 + 20 = physical pin 11 of J11 header
> (row 6, left column of the 10×2 header).

Using test points instead of LP headers is equally viable:
`TP5 = CQ_TX`, `TP6 = CQ_RX`, `TP13 = WAKE`, GPIO for EN1.

---

## 2 · TIOL221EVM jumper settings

| Header | Position | Effect |
|---|---|---|
| **J9**  | SPI_PIN-GND (pins 1-2) | **PIN mode** (not SPI mode) |
| **J15** | CS_PP-PU (pins 1-2)    | Push-pull CQ driver |
| **J17** | VSEL-PD (pins 1-2)     | VOUT = 3.3 V from L+_24V |
| **J8**  | CQ_ILIM-FIXED (pins 3-4) | ~150 mA CQ current limit |
| **J4**  | DO_ILIM-FIXED (pins 3-4) | ~150 mA DO current limit |
| **J3**  | *(leave open)*         | CQ enable via EN1 (CQ_EN_LP) |
| **J12** | pins 3-4, 5-6 (optional) | LED fault/reset indicators |
| J13, J5 | *(leave open)*         | No external 5 V needed |

---

## 3 · RPi5 OS configuration

### 3.1 Enable UART0 on GPIO14/15

Edit `/boot/firmware/config.txt` — ensure this line is present:
```ini
dtparam=uart0=on
```

On Raspberry Pi 5, `uart0=on` maps GPIO14 (TX) / GPIO15 (RX) to `/dev/ttyAMA10`
(RP1 serial port). The symlink `/dev/serial0` also points to this device.

If you want to verify this works:
```bash
ls -la /dev/serial0   # should symlink to ttyAMA10 or ttyAMA0
```

### 3.2 Disable serial getty

```bash
sudo systemctl disable hciuart
sudo systemctl disable serial-getty@ttyAMA0.service
```

### 3.3 Reboot

```bash
sudo reboot
```

### 3.4 Verify UART

```bash
ls -l /dev/ttyAMA0
```

---

## 4 · Software dependencies

```bash
pip3 install pyserial RPi.GPIO
```

Both are already installed on this system.

---

## 5 · Running the demo

```bash
cd industrial_automation/rpi5
sudo python3 gripper_tiol221_demo.py
```

`sudo` is required for GPIO access on RPi5.

The script will:
1. Send the IO-Link wake-up pulse.
2. Wait for the gripper to report `GripperPLCActive`.
3. Push operating parameters via the Zimmer handshake.
4. Run outside homing (jaws open to mechanical end-stop).
5. Load force-gripping profile (DeviceMode 62).
6. Execute 5 grip → release cycles.

---

## 6 · Files

| File | Purpose |
|---|---|
| `tiol221_iolink.py` | IO-Link master: UART + GPIO, TYPE_2_V M-Sequence |
| `zimmer_lwr50l.py`  | Zimmer LWR50L-02 PDO/PDI codec and constants |
| `gripper_tiol221_demo.py` | Startup + grip/release demo |

---

## 7 · IO-Link frame details

```
COM3 = 230 400 baud, 8E1

Page 0 (PDO bytes 0-7):
  Master TX  →  [0x98][PDO[0..7]][CKT]          10 bytes (EN1=1)
  Device RX  ←  [PDI[0..5]][0x00][0x00][CKT]    9 bytes  (EN1=0)

Page 1 (PDO bytes 8-15):
  Master TX  →  [0x9A][PDO[8..15]][CKT]          10 bytes (EN1=1)
  Device RX  ←  [0x00…][CKT]                     9 bytes  (EN1=0)

CKT = (CRC8₀ₓ₃₇(all_preceding_bytes) & 0xFC) | 0x01
```

---

## 8 · Process data layout

### PDO (16 bytes, master → gripper)

| Bytes | Field | Type | Notes |
|---|---|---|---|
| 0-1  | ControlWord    | UINT16 BE | One bit at a time, or 0x0000 |
| 2    | DeviceMode     | UINT8     | Movement profile (50=pos, 62=force) |
| 3    | WorkpieceNo    | UINT8     | Recipe 0=current, 1-32=stored |
| 4    | Reserve        | UINT8     | Always 0x00 |
| 5    | PosTolerance   | UINT8     | ±0.01 mm per step |
| 6    | GripForce      | UINT8     | 1-100 % |
| 7    | DriveVelocity  | UINT8     | 1-100 % |
| 8-9  | BasePosition   | UINT16 BE | 0.01 mm units |
| 10-11| ShiftPosition  | UINT16 BE | 0.01 mm units |
| 12-13| TeachPosition  | UINT16 BE | 0.01 mm units |
| 14-15| WorkPosition   | UINT16 BE | 0.01 mm units |

### PDI (6 bytes, gripper → master)

| Bytes | Field | Type | Notes |
|---|---|---|---|
| 0-1 | Diagnosis     | UINT16 BE | Error code (0=ok) |
| 2-3 | StatusWord    | UINT16 BE | See §11.4.12 of LWR50L-02 doc |
| 4-5 | ActualPosition| UINT16 BE | 0.01 mm units |

### Key StatusWord bits

| Bit | Mask | Meaning |
|---|---|---|
| 0 | 0x0001 | HomingPositionOK – must be 1 before moves |
| 1 | 0x0002 | MotorON |
| 2 | 0x0004 | InMotion |
| 3 | 0x0008 | MovementComplete |
| 6 | 0x0040 | GripperPLCActive – device booted |
| 8 | 0x0100 | AtBasePosition |
| 10| 0x0400 | AtWorkPosition |
| 12| 0x1000 | DataTransferOK – handshake ack |
| 15| 0x8000 | Error |

---

## 9 · Minimal quick-start code

```python
from tiol221_iolink import TIOL221IOLink
from zimmer_lwr50l import encode_pdo, decode_pdi, CW_MOVE_TO_WORK, DM_FORCE_OUTSIDE
import time

with TIOL221IOLink() as il:
    il.wakeup()
    time.sleep(0.1)  # let device boot
    pdo = encode_pdo(controlword=CW_MOVE_TO_WORK, device_mode=DM_FORCE_OUTSIDE)
    for _ in range(10):
        pdi = decode_pdi(il.exchange(pdo))
        print(pdi.status_str())
        time.sleep(0.01)
```

---

## 10 · Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `No such file /dev/ttyAMA0` | UART not enabled | Check `config.txt`, reboot |
| Page 0 timeout | Wrong baud / wiring | Verify GPIO14→J10-pin4, check J9 jumper |
| CKT mismatch | Byte-swap / echo flush | Check `_flush_echo(10)` count matches TX length |
| `GripperPLCActive` never set | No 24 V supply | Check PSU and J1/J2 connections |
| Error flag in StatusWord | Gripper fault | Read `diagnosis` field, check error table §14 |
| Jaws don't move after motor ON | Homing not done | Always run DeviceMode 10 homing on first boot |
