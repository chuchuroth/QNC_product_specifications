# IO-Link Debug Findings — Zimmer LWR50L-02 via STEVAL-IOM001V1

## Hardware Setup
- **MCU**: NUCLEO-F446RE, HSI 16 MHz
- **Transceiver board**: STEVAL-IOM001V1 (L6360 QFN-26L + IPS161H), stacked via morpho
- **Gripper**: Zimmer LWR50L-02 (IO-Link v1.1, COM3 230400 baud)
- **Serial debug**: /dev/ttyACM1 @ 115200

## Build / Flash Commands
```bash
cd /home/chuchu.xu/Downloads/P-NUCLEO-IOM01M1_dev/l6360_uart_transparent
make -j$(nproc)
export PATH="/opt/st/stm32cubeide_2.1.0_2/plugins/com.st.stm32cube.ide.mcu.externaltools.cubeprogrammer.linux64_2.2.400.202601091506/tools/bin:$PATH"
STM32_Programmer_CLI -c port=SWD -w build/l6360_transparent.elf -v -rst
```

## Pin Mapping (F446RE ↔ STEVAL-IOM001V1)
| Signal   | Pin | Notes |
|----------|-----|-------|
| ENCQ     | PC0 | CQ driver enable (ENCQ=1: TX mode, ENCQ=0: RX mode) |
| INCQ(TX) | PA9 | UART1_TX / GPIO bit-bang. **Inverted**: PA9 HIGH → CQ LOW |
| OUTCQ(RX)| PA10| UART1_RX / GPIO read. **Inverted**: PA10=0 → CQ HIGH |
| RST      | PB5 | L6360 reset (active LOW) |
| ENL+     | PA6 | L6360 internal L+ switch enable |
| L+_ON    | PB4 | IPS161H high-side switch → CN3 pin 1 |
| IRQ      | PA4 | L6360 fault interrupt |
| I2C_SCL  | PB8 | I2C1, 100 kHz |
| I2C_SDA  | PB9 | I2C1 |

## STEVAL-IOM001V1 Switch Settings (all verified)
| Switch | Setting | Function |
|--------|---------|----------|
| SW3    | 2-3     | VDD = 3.3V |
| SW4    | 1-2     | ENCQ via GPIO (PC0) |
| SW5    | 1-2     | ENL+ via GPIO (PA6) |
| SW6    | 2-3     | VH = VCC (24V) — **SW6 was bad, bridged CN1 pin2-pin3 manually** |
| SW7    | 1-2     | Type A (DI/DQ) |

## L6360 I2C Protocol (DS8900 Rev 9)

### Three I2C Access Modes
1. **Current Write** (our L6360_Reg_Write): `[data_byte, parity+addr_byte]`
   - Frame 3 = [P2,P1,P0,0,RA3,RA2,RA1,RA0]
   - P0 = XOR of all 8 data bits, P1 = XOR of odd bits, P2 = XOR of even bits
2. **Address Read** (our L6360_Reg_Read): `Transmit(reg) + STOP + Receive(data)`
   - Reads correctly but **DOES NOT clear latched fault bits**
3. **Current Read** (our L6360_CurrentRead): bare `Receive(data)` only
   - No preceding address transmit. Reads last-addressed register.
   - **REQUIRED to clear latched faults** (CQOL, PE, REGLN, etc.)

### Register Configuration (all verified working)
| Register | Addr | Value | Meaning |
|----------|------|-------|---------|
| STATUS   | 0x00 | 0x00  | Clean (POR=0x80 with PO flag) |
| CFG      | 0x01 | 0x60  | Push-pull mode (POR=0x80 tri-state) |
| CTRL1    | 0x02 | 0x60  | ICOQ=580mA, tdbq=0µs (POR=0x00) |
| CTRL2    | 0x03 | 0x21  | Default |

### STATUS Register Bits
| Bit | Name  | Description |
|-----|-------|-------------|
| 0   | PE    | Parity Error (**NOT PO** as in some docs) |
| 1   | REGLN | Linear regulator undervoltage |
| 3   | LOL   | L+ overload |
| 4   | CQOL  | CQ overload → driver cut-off |
| 5   | OVT   | Over-temperature |
| 7   | PO    | Power-on (L+ under-voltage) |

### CTRL1 Register Bits (verified from ST driver)
| Bits | Field  | 0x60 value |
|------|--------|------------|
| 7    | ENCGQ  | 0 = guard timer OFF |
| 6:5  | ICOQ   | 11 = 580mA (max) |
| 4:3  | TDCOQ  | 00 = 100µs blanking |
| 2    | TRCOQ  | 0 = 255×TDCOQ restart |
| 1:0  | TDBQ   | 00 = 0µs debounce (critical for COM3!) |

### CFG Register Mode Transitions — FORBIDDEN JUMPS
- **Cannot** go directly between push-pull (011) and HS_ON (110) / LS_ON (101)
- **Must** go through OFF (000) or TRI-STATE (100) between modes
- Forbidden writes: silently rejected, PE flag set, blocks subsequent writes
- Verified sequence: PP → OFF → HS_ON works; PP → HS_ON does NOT

## Bugs Found and Fixed (chronological)

| # | Bug | Fix | Session |
|---|-----|-----|---------|
| 1 | L6360 I2C write used standard `[addr, data]` | Rewrote as 3-frame Current Write with parity | 1 |
| 2 | CQI debounce tdbq=5µs > 1 COM3 bit (4.34µs) | CTRL1=0x60 (tdbq=0µs) | 1 |
| 3 | ICOQ=220mA too low for gripper inrush | CTRL1=0x60 (ICOQ=580mA) | 1 |
| 4 | TYPE_0 frame sent 3 bytes (TYPE_2 format) | Fixed to 2 bytes [MC, CKT] | 2 |
| 5 | MC byte CC=00 → wrong channel | Changed to CC=01 (Page channel), base 0xA0 | 2 |
| 6 | ENCQ gap between WURQ and TX | Kept ENCQ=1 continuously from WURQ through TX | 2 |
| 7 | Slow ENCQ→RX switch after TX | Moved into `__disable_irq()` section | 2 |
| 8 | `delay_us()` had static linkage issues | Replaced with inline DWT spins | 2 |
| 9 | SW6 bad contact (VH not reaching L6360) | User bridged CN1 pin2-pin3 physically | 3 |
| 10 | CFG forbidden transitions (PP→HS/LS) | Go through OFF first; clear PE between steps | 3 |
| 11 | **CQOL latched, never cleared** | Added `L6360_CurrentRead()` — bare I2C receive clears faults | 3 |

## CQ Driver — Fully Verified Working
- **HS_ON mode**: CFG=0xC0, PA10=0 (CQ=24V), STATUS=0x00 for 10+ seconds
- **LS_ON mode**: CFG=0xA0, PA10=1 (CQ=0V), STATUS=0x00
- **Push-pull**: CQ swings 0–24V (measured with multimeter on CN3 pin 4)
- **No CQOL**: STATUS stays 0x00 after L6360_CurrentRead fault clearing
- **L+ power cycle**: L+_ON toggle + ENL+ cycle clears PO flag (STATUS 0x80→0x00)

## Current State: Gripper Not Responding

### What works
- CQ driver: 24V HIGH, 0V LOW (multimeter confirmed)
- All L6360 registers correct, no faults
- CRC-6 correct (MC=0xA2→CKT=0x25, MC=0xA4→CKT=0x2F, MC=0xA7→CKT=0x2A)
- TX polarity correct (PA9 inverted through L6360)
- L+ power cycle works (IPS161H + ENL+)
- WURQ pulse timing correct (80µs/640µs/5120µs via DWT)

### What fails
- **WURQ-only test**: 0/100 PA10 zeros at all 3 COM speeds — device produces zero CQ activity
- **Bit-bang probe**: 45 WURQ+Type0 attempts (3 speeds × 3 MC × 5 retries) — all timeout
- **UART probe**: 3 speeds × 2-byte Type0 — all timeout
- **Waveform capture**: PA10 flat (all 1s = CQ LOW) at 10µs intervals after every attempt

### Most Likely Root Cause
The gripper is **not receiving the CQ signal** or **not powered properly**:
1. M12 cable continuity issue (CN3 pin 4 ↔ gripper pin 4 C/Q)
2. Connector gender mismatch or loose seating
3. Gripper variant without IO-Link
4. Gripper needs specific power-up sequence or is defective

### Next Steps — Physical Verification Required
1. Check M12 cable continuity: CN3 pin 4 ↔ gripper pin 4 (C/Q data line)
2. Measure voltage at gripper end of cable (pin 1-3 should be ~24V, pin 4-3 should be ~24V idle)
3. Verify M12 connector gender (CN3 = male → cable needs female on that end)
4. Confirm gripper model supports IO-Link (check label for IO-Link marking)
5. Try a different M12 cable if available

## Key Files
- `Core/Src/main.c`: Command dispatch, `cmd_bitbang_probe()` with CQ driver tests + IO-Link probe
- `Core/Src/l6360.c`: L6360 driver — I2C (Reg_Read, CurrentRead, Reg_Write), GPIO, bit-bang API
- `Core/Inc/l6360.h`: Register defs, pin mappings, API declarations
- Interactive commands: h(elp), b(itbang probe), n(diagnostics), w(ake), l(oopback), y(L+ cycle), etc.

## TEConcept IO-Link Stack
- Pre-loaded only on P-NUCLEO-IOM01M1 bundle boards (not our standalone F446RE)
- 10,000-minute time-limited license, accessed via SPI
- STSW-IOM001 is just an L6360 register demo, NOT a protocol stack

---
---

---

### IO-Link Python Automated GRIP/RELEASE Test (2026-03-15)

**Test script:** `gripper_minimal.py` (automated mode)  
**Setup:** RPi5 → TIOL221EVM (J10 UART) → Zimmer LWR50L-02  
**Commands:**  
1. Send GRIP (0xA2, 0x11), print status  
2. Wait 1s  
3. Send RELEASE (0xA2, 0x12), print status

**Observed Results:**
- Script initializes UART, sends wake-up, enters OPERATE mode, and issues GRIP/RELEASE commands.
- For both GRIP and RELEASE, device response is always:  
  - RX: 3 bytes (echo only, no device tail)
  - Statusword: 0x00 (ready=False, work_pos=False, base_pos=False, grip_error=False, error=False)
  - "No device response" printed after each command.
- No physical gripper motion observed.

**Interpretation:**
- The script is communicating at the protocol level, but the device is not responding to GRIP/RELEASE commands.
- Possible causes:
  - Device not fully in OPERATE mode or not accepting commands.
  - Power supply or wiring issue.
  - Timing or sequence issue after initialization.

**Next Steps:**
- Double-check 24V supply and IO-Link wiring.
- Confirm device state with manual/known-good tool.
- Add diagnostic RX output and/or retry logic if needed.
# Debug Results — Industrial Automation Closed-Loop Demo

**System:** Raspberry Pi 5 → NUCLEO-F746ZG → TIOL221EVM → Zimmer LWR50L-02 gripper  
**Date:** 2026-03-13  
**Repo:** `git@github.com:chuchuroth/IO-Link.git`

---

## System Architecture

```
RPi5 (Python controller)
  │  SPI0.0 @ 1 MHz, Mode 0, full-duplex
  ▼
NUCLEO-F746ZG (STM32F746ZG, SPI1 slave)
  │  UART4 @ 38400 baud (IO-Link COM2), half-duplex C/Q line
  ▼
TIOL221EVM (IO-Link transceiver)
  │  24 V IO-Link C/Q line
  ▼
Zimmer LWR50L-02 (Class B IO-Link gripper)
```

**SPI Protocol (1-byte frames):**

| Byte (RPi5 → Nucleo) | Meaning |
|---|---|
| `0x01` CMD_GRIP | Close gripper |
| `0x02` CMD_RELEASE | Open gripper |
| `0x03` CMD_STATUS | Read device status |

| Byte (Nucleo → RPi5) | Meaning |
|---|---|
| `0x00` RSP_IDLE | Idle / waiting |
| `0x01` RSP_GRIPPING | Grip command accepted / confirmed |
| `0x02` RSP_RELEASING | Release command accepted / confirmed |
| `0xFF` RSP_ERROR | IO-Link timeout or CRC failure |

---

## Bugs Found and Fixed

### Bug 1 — Wrong source file edited (all early fixes went to the wrong path)

**Symptom:** Every firmware fix had no effect after reflash.

**Root cause:** All edits were going to `nucleo/Core/Src/iolink.c` and `nucleo/Core/Src/main.c`, but the CubeIDE project compiles from `nucleo/industrial_automation_nucleo/Core/Src/`. The two directory trees are separate — git was tracking the wrong one.

**Fix:** Identified the correct path via `find`. All subsequent edits target `nucleo/industrial_automation_nucleo/Core/Src/`.

---

### Bug 2 — TIOL221EVM UART echo consumed device response

**Symptom:** `iolink_cycle()` always returned `IOLINK_ERR_TIMEOUT` or garbled data. Gripper never moved.

**Root cause:** While `EN=HIGH` (transmitting), the TIOL221EVM echoes all 3 TX bytes back on the DOUT/RX line. `HAL_UART_Receive` consumed these echo bytes instead of the real 2-byte device D-sequence response.

**Fix:** Added echo flush in `iolink_cycle()` after `HAL_UART_Transmit`, before reading the device response:

```c
// After HAL_UART_Transmit (EN set LOW):
uint8_t echo[3];
HAL_UART_Receive(&huart4, echo, 3, 3);          // drain echo bytes
huart4.Instance->ICR = USART_ICR_ORECF | USART_ICR_FECF | USART_ICR_NCF;
huart4.RxState = HAL_UART_STATE_READY;
// Now receive real response:
HAL_UART_Receive(&huart4, rx, 2, UART_RX_TIMEOUT_MS);
```

**Note on macro name:** STM32F7 uses `USART_ICR_NCF` (Noise Clear Flag), not `USART_ICR_NECF` — the latter does not exist on this family.

**Commit:** `6bcc2ec`

---

### Bug 3 — CubeMX regenerated `main.h`, wiping CMD/RSP constants

**Symptom:** CubeIDE build error: `'RSP_IDLE' undeclared here (not in a function)` for all CMD/RSP constants.

**Root cause:** CubeMX regenerates `main.h` on every "Generate Code" run, overwriting custom `#define` constants.

**Fix:** Moved all CMD/RSP constants from `main.h` to `iolink.h` (never touched by CubeMX). Added `void Error_Handler(void);` declaration to `main.h` and `Error_Handler()` definition to `main.c` (required by CubeMX-generated `stm32f7xx_hal_msp.c`).

**Commits:** `ee81cf0`, `4951f34`

---

### Bug 4 — `spi_arm_next_transfer()` called in ISR before `spi_tx_byte` was updated

**Symptom:** After the echo flush fix, SPI responses were still always `RSP_IDLE` regardless of command.

**Root cause:** `HAL_SPI_TxRxCpltCallback` called `spi_arm_next_transfer()` immediately when the SPI transfer completed. This loaded the **current** `spi_tx_byte` (still `RSP_IDLE`) into the SPI TX hardware register. The main loop then ran `iolink_cycle()` and updated `spi_tx_byte = RSP_GRIPPING`, but it was too late — the TX register was already loaded for the next transfer.

**Fix:** Removed `spi_arm_next_transfer()` from the ISR. The ISR now only sets `new_cmd_flag = 1`. The main loop calls `spi_arm_next_transfer()` **after** updating `spi_tx_byte`:

```c
// ISR — only flag:
void HAL_SPI_TxRxCpltCallback(SPI_HandleTypeDef *hspi) {
    if (hspi->Instance != SPI1) return;
    new_cmd_flag = 1;   // NO spi_arm_next_transfer() here
}

// Main loop — update response THEN arm:
case CMD_GRIP:
    rc = iolink_cycle(IOLINK_PD_GRIP, &pd_in);
    spi_tx_byte = (rc == IOLINK_OK) ? RSP_GRIPPING : RSP_ERROR;
    break;
// ...
spi_arm_next_transfer();   // called AFTER switch block
```

**Commit:** `a86af3d`

---

### Bug 5 — `main.c` init stubs had no bodies (root cause of 0x00 SPI response)

**Symptom:** Diagnostic PING test (`0xAA → expect 0x55`) returned `0x00`. SPI peripheral was not responding at all.

**Root cause:** Our repo's `main.c` contained **stub** implementations of `SystemClock_Config()`, `MX_GPIO_Init()`, `MX_SPI1_Init()`, `MX_USART4_UART_Init()` with no code bodies. When pulled from git to the CubeIDE machine, these stubs silently replaced the CubeMX-generated functions that contain the real peripheral init code. The MCU booted but SPI was never initialized.

**Fix:** Removed `main.c` from git tracking entirely (`.gitignore`). Moved all application logic into `app.c` / `app.h` which are CubeMX-safe (never touched by code regeneration).

Integration into the CubeMX-owned `main.c` is reduced to 3 lines in USER CODE sections — see [INTEGRATION.md](nucleo/industrial_automation_nucleo/INTEGRATION.md).

**Commit:** `007d643`

---

## Current Status

| Layer | Status |
|---|---|
| RPi5 Python controller (`main.py`) | ✅ Running |
| SPI communication (RPi5 ↔ Nucleo) | ⏳ Pending re-test after Bug 5 fix |
| IO-Link UART driver (echo-flush) | ✅ Fixed |
| Gripper physical motion | ⏳ Pending reflash with new app.c structure |

---

## Next Steps

1. In CubeIDE: pull latest → add `app.c` to build → add 3 USER CODE lines to `main.c` → Clean → Build → Flash
2. Run PING test to confirm SPI layer:
   ```bash
   sudo python3 - <<'EOF'
   import spidev, time
   spi = spidev.SpiDev(); spi.open(0,0); spi.max_speed_hz=1_000_000; spi.mode=0
   def xfr(c): time.sleep(0.00002); return spi.xfer2([c])[0]
   xfr(0x01); time.sleep(0.3); r = xfr(0x01)
   print("GRIP:", {0x01:"GRIPPING",0x00:"IDLE",0xFF:"ERROR"}.get(r, hex(r)))
   spi.close()
   EOF
   ```
3. Once `GRIPPING` / `RELEASING` confirmed, run full closed-loop demo: `sudo python3 main.py`

---

## File Map (tracked in git)

```
nucleo/industrial_automation_nucleo/
├── Core/
│   ├── Inc/
│   │   ├── app.h          ← application API + CubeMX integration instructions
│   │   ├── iolink.h       ← IO-Link driver API + CMD/RSP SPI constants
│   │   └── main.h         ← Error_Handler declaration only
│   └── Src/
│       ├── app.c          ← SPI dispatch loop + HAL_SPI callback (CubeMX-safe)
│       └── iolink.c       ← IO-Link master driver (echo-flush fix included)
└── INTEGRATION.md         ← 3-line recipe for CubeMX main.c
```

`main.c` is **not tracked** (gitignored) — owned entirely by CubeMX.
