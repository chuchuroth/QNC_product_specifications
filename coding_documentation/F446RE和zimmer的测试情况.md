# IO-Link Master — Zimmer LWR50L Gripper Control

Bare-metal STM32F446RE firmware that drives a **Zimmer LWR50L-02** parallel
gripper over IO-Link v1.1 through the **L6360** master transceiver.

## Hardware

| Board / Module         | Role                                |
|------------------------|-------------------------------------|
| NUCLEO-F446RE          | MCU (STM32F446RE, Cortex-M4, 16 MHz HSI) |
| STEVAL-IOM001V1        | L6360 IO-Link master PHY (on Arduino headers) |
| IPS161H (on-board)     | 24 V L+ high-side switch            |
| Zimmer LWR50L-02       | IO-Link gripper (COM3, 230.4 kbaud) |

### Pin Map

| Signal   | Pin  | AF  | Notes                        |
|----------|------|-----|------------------------------|
| UART1 TX | PA9  | AF7 | C/Q to L6360 (9-bit, 8E1)   |
| UART1 RX | PA10 | AF7 | C/Q from L6360               |
| UART2 TX | PA2  | AF7 | Debug VCP 115200 8N1         |
| UART2 RX | PA3  | AF7 | Debug VCP                    |
| I2C1 SCL | PB8  | AF4 | L6360 registers (addr 0x60)  |
| I2C1 SDA | PB9  | AF4 |                              |
| RST      | PB5  | —   | L6360 reset (active low)     |
| ENCQ     | PC0  | —   | L6360 CQ driver enable       |
| ENL+     | PA6  | —   | L6360 L+ enable              |
| L+_ON    | PB4  | —   | IPS161H enable (24 V supply) |
| IRQ      | PA4  | —   | L6360 interrupt (pull-up)    |

## Architecture

```
┌────────────────────┐
│      main.c        │  Interactive command dispatcher (~20 debug commands)
│  (cmd_* handlers)  │
└────────┬───────────┘
         │ calls
┌────────▼───────────┐
│  zimmer_gripper.c  │  Gripper application layer (PDO/PDI packing, startup
│  zimmer_gripper.h  │  handshake, open/close/moveTo/poll)
└────────┬───────────┘
         │ calls
┌────────▼───────────┐
│  iolink_master.c   │  IO-Link v1.1 master protocol (CRC-6, TYPE_0/TYPE_2_V
│  iolink_master.h   │  M-sequences, COM auto-detect, OPERATE transition)
└────────┬───────────┘
         │ calls
┌────────▼───────────┐
│     l6360.c        │  L6360 PHY driver (I2C registers, UART transparent mode,
│     l6360.h        │  GPIO, wake-up pulse, CRC-8, bit-bang)
└────────┴───────────┘
         │ uses
    STM32 HAL (UART, I2C, GPIO, DWT)
```

### Layer Responsibilities

- **l6360.c/h** — Hardware abstraction for the L6360 transceiver: GPIO init,
  UART1 (9-bit even parity) init, I2C1 init, register read/write with parity,
  wake-up pulse generation (DWT cycle counter), TX/RX direction switching,
  CRC-8 checksum, and raw register-level operations.

- **iolink_master.c/h** — IO-Link v1.1 master protocol: CRC-6 per the spec
  (polynomial x⁶+x+1), TYPE_0 M-sequence (3-out/2-in for page parameter
  access), TYPE_2_V M-sequence (variable-length process data), COM speed
  auto-detection (COM3→COM2→COM1 fallback), Direct Parameter Page read,
  and OPERATE mode transition.

- **zimmer_gripper.c/h** — Zimmer LWR50L application: PDO struct (16 bytes:
  ControlWord, GripMode, positions, force, velocity) and PDI struct (6 bytes:
  StatusWord, Diagnose, ActualPosition), big-endian wire packing/unpacking,
  full startup handshake (PLCActive poll → DataTransfer → MotorON →
  PositionStd), open/close/moveTo motions with timeout, status polling.

- **main.c** — Interactive serial console (UART2 @ 115200) with ~20 single-key
  commands for diagnostics, gripper control, and protocol debugging.

## About X-CUBE-IOD02

The `x-cube-iod02/` package (STM32CubeExpansion_IOD02_V3.0.0) is an IO-Link
**device** stack for L6362A/L6364 transceivers. It implements the *device* side
of the IO-Link protocol and **cannot** be used as a master stack. This project
implements the master side from scratch using the L6360 in UART transparent
mode, which is the correct approach for the STEVAL-IOM001V1 board.

The `stsw-iom001/` package provides a BSP wrapper for the L6360 (`BSP_IOLinkMaster_*`
API) but contains no IO-Link protocol stack — only register-level PHY access.
The custom `l6360.c` driver in this project provides equivalent and more robust
functionality (I2C parity handling, fault clearing via Current Read, etc.).

## Build

Requires `arm-none-eabi-gcc` (tested with GCC 14.3 from STM32CubeIDE 2.1.0).

```bash
cd l6360_uart_transparent
make clean && make
```

The build expects HAL drivers at `../../Smart_Sorting_and_Assembly_Demo/Drivers/`.
Edit `DRIVERS_DIR` in the Makefile if your HAL drivers are elsewhere.

Output: `build/l6360_transparent.elf` and `build/l6360_transparent.bin`.

## Flash

```bash
# Via ST-Link (OpenOCD)
openocd -f interface/stlink.cfg -f target/stm32f4x.cfg \
  -c "program build/l6360_transparent.elf verify reset exit"

# Or drag-and-drop .bin to the NUCLEO mass-storage drive
cp build/l6360_transparent.bin /media/$USER/NOD_F446RE/
```

## Usage

Connect to the Nucleo's VCP at **115200 8N1**. Press `?` for the command menu:

| Key | Command           | Description                              |
|-----|-------------------|------------------------------------------|
| `g` | Gripper startup   | Full IO-Link + gripper bring-up          |
| `o` | Gripper open      | Move to BasePosition                     |
| `c` | Gripper close     | Move to WorkPosition                     |
| `p` | Gripper poll      | Read StatusWord, Diagnose, ActualPosition|
| `w` | Wake-up           | Send WURQ pulse on C/Q                   |
| `s` | Send frame        | Manual TYPE_0 exchange                   |
| `r` | Reset             | Hard-reset L6360                         |
| `d` | Diagnostics       | L6360 register + GPIO dump               |
| `b` | Bitbang probe     | Bit-bang TX/RX on C/Q line               |
| `t` | Deep test         | Comprehensive L6360 + comm test          |
| `?` | Help              | Print command menu                       |

### Typical Session

```
> g          ← full startup (WURQ → COM detect → OPERATE → MotorON)
> o          ← open gripper (move to base position)
> c          ← close gripper (move to work position)
> p          ← poll current status
```

## Gripper Parameters (Defaults)

| Parameter          | Value   | Units |
|--------------------|---------|-------|
| BasePosition       | 1.00    | mm    |
| WorkPosition       | 40.00   | mm    |
| ShiftPosition      | 20.00   | mm    |
| TeachPosition      | 38.00   | mm    |
| GripForce          | 50      | %     |
| DriveVelocity      | 50      | %     |
| PositionTolerance  | 0.50    | mm    |

Parameters can be customized via `Gripper_SetConfig()` before calling
`Gripper_Startup()` — see the commented-out example in `cmd_gripper_startup()`.

## Known Issues

See [DEBUG_FINDINGS.md](DEBUG_FINDINGS.md) for a detailed record of 11 bugs
found and fixed during bring-up. The current suspected failure point is the
physical connection (M12 cable, connector, or gripper hardware).

## IO-Link Device Identity (from IODD)

| Field       | Value                |
|-------------|----------------------|
| Vendor      | Zimmer Group (836)   |
| Device ID   | 0x020106 (131334)    |
| IO-Link     | v1.1, COM3           |
| PDO (out)   | 16 bytes (128 bits)  |
| PDI (in)    | 6 bytes (48 bits)    |
| MinCycleTime| 10 ms                |
