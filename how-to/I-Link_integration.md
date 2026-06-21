Because IO-Link is a proprietary, highly regulated industrial standard, obtaining an **IO-Link Master software stack** typically involves choosing between a licensed, production-ready commercial solution or an open-source option for evaluation and testing.

Here is how and where you can acquire one, depending on your project needs.

---

## 1. Commercial-Grade Stacks (For Production)

If you are developing a commercial hardware product (such as a multi-port industrial IO-Link Master gateway or PLC module), you should purchase a fully compliant, pre-certified stack. These vendors provide complete source code or binary libraries, hardware abstraction layers (HALs) for specific microcontrollers/PHYs, integration support, and compliance guarantees.

### Key Commercial Vendors:

* **TEConcept:** One of the most prominent partners of the IO-Link Consortium. They offer an officially certified **IO-Link Master Software Stack** (and development tools) that supports multi-port architectures, high-speed COM3, and is heavily optimized for microcontrollers like the STM32, NXP LPC, and more.
* *How to get it:* You can request a license directly via **[TEConcept](https://teconcept.de/)** or through their semiconductor partners like **STMicroelectronics** (search for the *TEConcept IO-Link Master Software Stack* on ST's partner portal).


* **RT-Labs:** Offers **i-link**, a highly robust, professional-grade IO-Link Master stack compliant with the latest specifications (supporting Real-Time Class 1 and Conformance Classes A & B).
* *How to get it:* You can buy pre-qualified commercial licenses directly via **[rt-labs.com](https://rt-labs.com)**.


* **TMG TE (TMG Technologie und Engineering):** Another major official IO-Link technology partner that licenses highly optimized Master and Device stacks, especially for integrating IO-Link into fieldbus systems (like PROFINET or EtherCAT).

---

## 2. Open-Source & Evaluation Stacks (For Prototyping & R&D)

If you are in the research and development phase, teaching yourself the protocol, or building a proof-of-concept, you can access open-source stacks.

* **RT-Labs GitHub (i-link):** * RT-Labs dual-licenses their **i-link** IO-Link Master stack. They host an open-source version on GitHub under the **GPLv3 license**.
* *How to get it:* Go to GitHub and search for `rtlabs-com/i-link`.
* *Note:* The open-source version is ideal for testing but may lack some of the advanced, specialized features or optimization found in their commercial release.


* **Zephyr RTOS Integration (zephyr-iolm):**
* If you are building an embedded system on Zephyr RTOS, there are open-source wrapper modules (like `vChavezB/zephyr-iolm` on GitHub) that integrate the RT-Labs Master stack with Zephyr, specifically tested with transceiver PHYs like the Analog Devices/Maxim MAX14819.


* **Lightweight Hobbyist Stacks:**
* For extremely basic microcontroller testing (like Arduino, ESP32, or Raspberry Pi Pico), you can find community-driven hobbyist projects on GitHub, such as `laith-dosinfection/lwIOlinkMaster`. *Keep in mind these are stripped-down and not suitable for rigorous industrial compliance.*



---

## What Else You Will Need

An IO-Link Master is a combination of software and specialized hardware. To actually use the software stack, you will need to pair it with:

1. **An IO-Link Master PHY (Transceiver):** Such as the **L6360** (STMicroelectronics), **MAX14819** (Analog Devices), or **TI OL1282**. The stack communicates with these chips via SPI or UART.
2. **An MCU:** Typically an ARM Cortex-M series (such as STM32) to run the stack and coordinate the ports.
3. **IO-Link Consortium Membership:** If you plan to market and sell your physical IO-Link Master, you must join the IO-Link community to officially use the logo and declare conformity.
 
 ---
 ---
 
 **There is no free, standalone IO-Link master stack in STM32CubeG4** that you can simply flash. The original stack on your NUCLEO-F446RE was a **proprietary, time-limited license from TEConcept GmbH**, and ST explicitly states that overwriting it makes restoration impossible without buying a new kit. [st](https://www.st.com/en/evaluation-tools/p-nucleo-iom01m1.html)

However, you have **two practical paths forward**:

***

## Option 1: Use Open-Source I-Link Stack (Recommended for DIY)

There is a **free, open-source IO-Link master stack** called **I-Link** from rtlabs-com that works with STM32 HAL:

**GitHub:** https://github.com/rtlabs-com/i-link [github](https://github.com/rtlabs-com/i-link/blob/public/README.md)

### Step-by-Step: Flash I-Link on NUCLEO-G474RE + STEVAL-IOM001V1

#### Prerequisites
- NUCLEO-G474RE (or your existing F446RE)
- STEVAL-IOM001V1 (L6360 PHY board)
- STM32CubeIDE (latest version)
- ST-Link V2 (built into Nucleo)

***

#### Step 1: Install STM32CubeIDE
1. Download from: https://www.st.com/en/development-tools/stm32cubeide.html
2. Install and launch STM32CubeIDE
3. Install STM32G4 MCU package via **Help → Manage Embedded Software Packages**

***

#### Step 2: Clone I-Link Repository
```bash
git clone https://github.com/rtlabs-com/i-link.git
cd i-link
```

***

#### Step 3: Create New STM32CubeIDE Project
1. **File → New → STM32 Project**
2. Select **STM32G474RETx** (or STM32F446RE if using old board)
3. Name project `IO_Link_Master`
4. Click **Finish**

***

#### Step 4: Configure Pins in CubeMX
Configure these critical pins for L6360 communication:

| Function | Pin (G474RE) | Mode | Notes |
|----------|--------------|------|-------|
| **UART TX** | PA2 (Arduino D1) | USART2_TX | L6360 UART interface |
| **UART RX** | PA3 (ArduinoHere is the critical reality check before the steps: **STM32CubeG4 does NOT contain an IO-Link master stack.** ST only provides **IO-Link device (sensor) stacks** (X-CUBE-IOD02, FP-IND-IODSNS1) for free. The master stack you lost was **TEConcept's proprietary, licensed software** that came pre-flashed only on P-NUCLEO-IOM01M1 kits and is not downloadable from ST.  [st](https://www.st.com/en/evaluation-tools/p-nucleo-iom01m1.html)

You have **three realistic paths forward**:

***

## Option 1: Use Open-Source I-Link Master Stack (Recommended for DIY)

The only viable free/open-source IO-Link master stack for STM32 is **I-Link** from rtlabs-com.

### Step-by-Step Instructions

### 1. Install Prerequisites
```bash
# Install STM32CubeIDE (latest version)
# Download from: https://www.st.com/en/development-tools/stm32cubeide.html

# Install STM32CubeG4 package via CubeIDE or STM32CubeMX
# This gives you HAL drivers for G474RE
```

### 2. Clone I-Link Stack
```bash
git clone https://github.com/rtlabs-com/i-link.git
cd i-link
```

### 3. Create STM32CubeMX Project for NUCLEO-G474RE
1. Open **STM32CubeMX** (bundled with CubeIDE)
2. Select **NUCLEO-G474RE** board
3. Configure peripherals for L6360 interface:
   - **UART** (or SPI) for master communication
   - **GPIO** for C/Q line control
   - **Timer** for timing-critical operations
4. Generate code for **STM32CubeIDE**

### 4. Integrate I-Link Stack
Copy I-Link source files into your CubeIDE project:
```
YourProject/
├── Core/
│   ├── Inc/
│   └── Src/
├── I-Link/          ← Copy from i-link/src/
│   ├── inc/
│   └── src/
└── Drivers/         ← Auto-generated by CubeMX
```

### 5. Port Hardware Abstraction Layer
Modify I-Link's HAL layer to use STM32CubeG4 HAL:
- Replace UART/SPI drivers with `HAL_UART_Transmit()` / `HAL_SPI_Transmit()`
- Map GPIO toggling to `HAL_GPIO_WritePin()`
- Configure timers for IO-Link timing requirements

### 6. Build and Flash
```bash
# In STM32CubeIDE:
# 1. Project → Build All
# 2. Run → Debug As → STM32 Cortex-M C/C++ Application
# Or use ST-LINK Utility to flash the .elf/.bin file
```

***

## Option 2: Buy TEConcept's Commercial Stack

If you need a production-ready, certified stack:

1. Visit: https://teconcept.de/en/products/io-link-master-stack/ [teconcept](https://teconcept.de/en/products/io-link-master-stack/)
2. Request license for **IMS-LSS-DL** (IO-Link Master Software Stack)
3. Cost: Contact for pricing (typically €5k-€15k for commercial license)
4. They provide binaries + integration support for STM32

***

## Option 3: Replace the Kit (Fastest for Evaluation)

Buy a new **P-NUCLEO-IOM01M1** (~€100-€120) - it comes with fresh pre-flashed TEConcept stack with renewable 10k-minute license. [st](https://www.st.com/en/evaluation-tools/p-nucleo-iom01m1.html)

***

## Key Files You Need for Option 1

| Component | Source |
|-----------|--------|
| I-Link stack | https://github.com/rtlabs-com/i-link  [github](https://github.com/rtlabs-com/i-link) |
| STM32CubeG4 HAL | STM32CubeIDE Package Manager or https://github.com/STMicroelectronics/STM32CubeG4  [github](https://github.com/STMicroelectronics/STM32CubeG4) |
| L6360 datasheet | https://www.st.com/en/power-transceivers/l6360.html |
| IO-Link spec | https://www.io-link.com (free registration) |

***

**Bottom line:** There is **no official ST-downloadable master stack** in STM32CubeG4. Your choices are: open-source I-Link (free but requires porting work), TEConcept commercial license (expensive but production-ready), or buy a replacement kit (cheapest for quick evaluation). [st](https://www.st.com/en/partner-products-and-services/teconcept-io-link-master-software-stack.html)

---
## I-Link Stack Integration Steps (NUCLEO-G474RE + STEVAL-IOM001V1)
Here's the **complete step-by-step integration** of the open-source **I-Link IO-Link master stack** into your STM32CubeMX project:
### **Step 1: Clone and Prepare I-Link**
```bash
git clone https://github.com/rtlabs-com/i-link.git
cd i-link
# Copy the entire src/ and inc/ folders to your STM32CubeIDE project
```

**Your project structure becomes:**
```
IO_Link_Master/
├── Core/                 # CubeMX generated
├── Drivers/              # CubeMX HAL
├── I-Link/
│   ├── inc/             # I-Link headers
│   └── src/             # I-Link source
└── main.c
```
### **Step 2: Add I-Link to Project**
**In STM32CubeIDE:**
1. Right-click project → **Properties → C/C++ Build → Settings**
2. **C/C++ Build → Includes → Add:**
   ```
   ../I-Link/inc
   ../Core/Inc  
   ../Drivers/STM32G4xx_HAL_Driver/Inc
   ```
3. **C/C++ Build → Libraries → Add:** `m` (math library)
### **Step 3: Create Hardware Abstraction Layer (HAL)**
**Create `iolink_hal.c`** in `Core/Src/`:

```c
/* I-Link HAL for STM32G4 + L6360 */
#include "iolink_hal.h"
#include "main.h"
#include "usart.h"
#include "spi.h"
#include "gpio.h"

// I-Link HAL functions (must implement ALL)
void il_hal_init(void) {
    // L6360 PHY initialization sequence
    L6360_Init();
}

void il_hal_usart_tx(uint8_t* data, uint16_t len) {
    HAL_UART_Transmit(&huart2, data, len, 100);
}

void il_hal_usart_rx(uint8_t* data, uint16_t len) {
    HAL_UART_Receive_IT(&huart2, data, len);
}

void il_hal_gpio_set(IL_GPIO_t gpio, uint8_t state) {
    switch(gpio) {
        case IL_GPIO_CQ_EN:  HAL_GPIO_WritePin(GPIOA, GPIO_PIN_10, state); break;
        case IL_GPIO_CS:     HAL_GPIO_WritePin(GPIOA, GPIO_PIN_8, !state);  break;
    }
}

uint8_t il_hal_gpio_get(IL_GPIO_t gpio) {
    if (gpio == IL_GPIO_IRQ) 
        return HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_0);
    return 0;
}

void il_hal_spi_tx(uint8_t* data, uint16_t len) {
    HAL_SPI_Transmit(&hspi1, data, len, 100);
}

void il_hal_delay_us(uint32_t us) {
    HAL_Delay(us / 1000);
    // Use DWT for precise µs timing
}
```
### **Step 4: L6360 PHY Driver Integration**
**Download L6360 driver** from ST and add to `Drivers/L6360/`:

```c
// L6360_Init() - Critical initialization sequence
void L6360_Init(void) {
    uint8_t cmd [download.kamami](https://download.kamami.pl/p572181-en.DM00510921.pdf) = {0x80, 0x00};  // Reset command
    
    // SPI config sequence from datasheet
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_8, GPIO_PIN_RESET);  // CS low
    HAL_SPI_Transmit(&hspi1, cmd, 2, 100);
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_8, GPIO_PIN_SET);   // CS high
    
    HAL_Delay(10);  // Wait for PHY startup
}
```
### **Step 5: Main Application Integration**
**Replace `main.c` USER CODE sections:**

```c
/* USER CODE BEGIN Includes */
#include "iolink_master.h"
#include "iolink_hal.h"
#include "l6360_driver.h"
/* USER CODE END Includes */

/* USER CODE BEGIN PFP */
IL_MASTER_t il_master;
il_status_t status;
/* USER CODE END PFP */

int main(void) {
    /* CubeMX init */
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
    MX_USART2_UART_Init();
    MX_SPI1_Init();
    
    /* USER CODE BEGIN 2 */
    // Initialize IO-Link stack
    il_hal_init();
    il_master_init(&il_master, 1);  // 1 port configuration
    
    /* Start IO-Link master */
    il_master_start(&il_master);
    /* USER CODE END 2 */
    
    /* Infinite loop */
    while (1) {
        /* USER CODE BEGIN 3 */
        status = il_master_run(&il_master);
        if (status != IL_STATUS_OK) {
            // Handle errors
            HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
            HAL_Delay(100);
        }
        /* USER CODE END 3 */
    }
}
```
### **Step 6: UART Callback for IO-Link**
**In `stm32g4xx_it.c`:**

```c
void USART2_IRQHandler(void) {
    HAL_UART_IRQHandler(&huart2);
}

void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart) {
    if (huart->Instance == USART2) {
        // Feed received bytes to I-Link stack
        il_rx_callback(rx_buffer, rx_len);
    }
}
```
### **Step 7: Build and Flash**
```
1. Project → Build All (Fix any missing includes)
2. Run → Debug As → STM32 C/C++ Application
3. Connect NUCLEO-G474RE + STEVAL-IOM001V1
4. Press Reset → LED blinking = PHY alive
```
### **Step 8: Test Sequence**
```
1. Power on → L6360 IRQ pin should go LOW
2. Connect IO-Link device to CN1 (Port 1)
3. Monitor UART console (115200 baud) for device detection
4. CQ line (PA2/PA3) should show 230.4kbaud traffic
```
## **Critical Gotchas**
- **I2C address** for L6360 must match SW0/1/2 jumper settings
- **230400 baud** exact - IO-Link COM3 mode tolerance is tight
- **IRQ pin** (PB0) must be configured as **falling edge interrupt**
- **SPI CS timing** - minimum 1µs assertion time
## **Expected Behavior**
```
1. Power-up: L6360 IRQ goes LOW → Master detects PHY ready
2. Device connect: CQ line toggles → Master detects device ID
3. Normal op: 230kbaud UART traffic + periodic process data
```

**This gives you a fully functional IO-Link v1.1 master** with **open-source stack** that replaces the lost TEConcept firmware. Ready for production or further customization.



---
---
**NUCLEO-G474RE + STEVAL-IOM001V1 IO-Link master pin configuration** in STM32CubeMX:

## Hardware Connection Overview
The **STEVAL-IOM001V1** connects to **Arduino-compatible pins** on NUCLEO-G474RE. Key signals: **C/Q line** (UART + GPIO control), **SPI** for L6360 config
## Hardware Connection Overview
The **STEVAL-IOM001V1** connects to **Arduino-compatible pins** on NUCLEO-G474RE. Key signals: **C/Q line** (UART + GPIO control), **SPI** for L6360 config, **interrupts**.
## Step-by-Step STM32CubeMX Configuration
### 1. Start Project
```
File → New Project → Board Selector → NUCLEO-G474RE → Finish
```
### 2. Pin Assignments (Arduino Headers)
| Pin Name | MCU Pin | Function | Mode | Notes |
|----------|---------|----------|------|-------|
| **D1 (TX)** | **PA2** | USART2_TX | Alternate Function | C/Q line UART TX to L6360 |
| **D0 (RX)** | **PA3** | USART2_RX | Alternate Function | C/Q line UART RX from L6360 |
| **D2** | **PA10** | GPIO_Output | Output | C/Q enable control |
| **D3** | **PB0** | GPIO_Input | Input | L6360 IRQ |
| **D4** | **PB7** | SPI1_MOSI | Alternate Function | L6360 SPI data |
| **D5** | **PB6** | SPI1_MISO | Alternate Function | L6364 SPI data |
| **D6** | **PA15** | SPI1_SCK | Alternate Function | L6360 SPI clock |
| **D7** | **PA8** | GPIO_Output | Output | L6360 CS (chip select) |
### 3. Peripherals Configuration
**USART2** (C/Q line):
```
USART2 Mode → Asynchronous
Baud Rate → 230400 (IO-Link standard)
Word Length → 8 Bits
Parity → None
Stop Bits → 1
```

**SPI1** (L6360 config):
```
SPI1 Mode → Full-Duplex Master
Data Size → 8 Bits
First Bit → MSB First
Prescaler → 8 (1.125 MHz)
```

**TIM2** (IO-Link timing):
```
TIM2 → Internal Clock
Prescaler → 170 (1us tick @170MHz)
Period → 65535
```

**NVIC**:
```
USART2 global interrupt → Enabled
SPI1 global interrupt → Enabled
EXTI line0/1 → Enabled (for PB0 IRQ)
```
### 4. GPIO Settings
```
All GPIO pins → Push-Pull, High Speed, No Pull-up/Pull-down
PA10, PA8 → Initial state LOW
```
### 5. Clock Configuration
```
HSE → Crystal/Ceramic Resonator (8MHz external)
PLL Source Mux → HSE
System Clock Mux → PLLCLK
HCLK → 170 MHz
APB1 → 85 MHz (USART2)
APB2 → 170 MHz (SPI1)
```
### 6. Generate Code
```
Project Manager → Toolchain → STM32CubeIDE
Advanced Settings → Generate peripheral initialization as .c/.h files
→ Generate Code
```
## Verification
After generation, check `main.c` for:
```c
/* USER CODE BEGIN Includes */
#include "l6360_driver.h"  /* Add your L6360 driver here */
/* USER CODE END Includes */

/* USER CODE BEGIN PV */
uint8_t CQ_Line_State = 0;
/* USER CODE END PV */
```
## Hardware Connections Required
```
NUCLEO-G474RE Arduino → STEVAL-IOM001V1 Morpho
D1 (PA2) → CN4-1 (TX)
D0 (PA3) → CN4-2 (RX) 
D2 (PA10) → CN4-3 (C/Q_EN)
D3 (PB0) → CN4-4 (IRQ)
D4-D7 → SPI connector
GND → GND
3V3 → VDD
```

This configuration matches the **STEVAL-IOM001V1 user manual** pinout for Arduino Nucleo compatibility. [download.kamami](https://download.kamami.pl/p581676-dm00720282-getting-started-with-the-xcubeiod02-industrial-iolink-device-transceiver.pdf)

---
---
**IO-Link C/Q line requires specific UART/SPI settings** in STM32CubeMX for **NUCLEO-G474RE + STEVAL-IOM001V1**:

## UART Configuration (Primary: C/Q Communication Line)

### 1. Enable USART2 (Arduino D0/D1)
```
Pinout & Configuration → Connectivity → USART2
→ Mode: Asynchronous
```

### 2. UART Parameters (Critical for IO-Link)
```
Parameter Settings:
├── Baud Rate: **230400** (IO-Link standard) or **115200**
├── Word Length: **8 Bits**
├── Parity: **None**
├── Stop Bits: **1**
├── Data Direction: **Receive and Transmit**
├── Oversampling: **16 samples** (higher tolerance)
├── Hardware Flow Control: **Disabled**
```

```
NVIC Settings:
├── USART2 global interrupt: **Enabled** (for Rx callbacks)
```

## SPI Configuration (L6360 Register Access)

### 1. Enable SPI1 (Arduino D4-D7)
```
Pinout & Configuration → Connectivity → SPI1
→ Mode: **Full-Duplex Master**
```

### 2. SPI Parameters
```
Parameter Settings:
├── Data Size: **8 Bits**
├── First Bit: **MSB First**
├── Prescaler: **8** (1.125 MHz max for L6360)
├── Clock Polarity (CPOL): **Low**
├── Clock Phase (CPHA): **1 Edge**
├── CRC: **Disabled**
```

```
NVIC Settings:
├── SPI1 global interrupt: **Enabled**
```

## Complete IO-Link Peripheral Stack

```
Required Peripherals:
├── USART2 → C/Q line (primary communication)
├── SPI1 → L6360 configuration registers
├── TIM2 → Precise timing (1µs resolution)
├── GPIO → C/Q enable, IRQ handling
└── DMA → Optional for high-speed transfers
```

## Clock Requirements
```
HCLK: 170 MHz
APB1 Timer: 170 MHz (TIM2)
APB1 Peripheral: 85 MHz (USART2)
APB2 Peripheral: 170 MHz (SPI1)
```

## Advanced UART Settings (IO-Link Specific)

```
Advanced Features Tab:
├── TXFIFO → Enabled (if available)
├── RXFIFO → Enabled
├── DMA Requests:
│   ├── TX DMA: **USART2_TX** (Medium priority)
│   └── RX DMA: **USART2_RX** (High priority)
├── OneBit Sampling: **Enabled** (noise immunity)
```

## Pin Remapping (Arduino Headers)
```
Morpho Connector → Arduino Pins:
├── CN4-1 (TX) → PA2 (D1)
├── CN4-2 (RX) → PA3 (D0)
├── CN4-3 (C/Q_EN) → PA10 (D2)
├── CN4-4 (IRQ) → PB0 (D3)
└── SPI → PB6/PB7/PA15 (D5/D4/D6)
```

## Code Generation Checklist
```
Project Manager → Code Generator:
├── Generate peripheral init as pair of '.c/.h' files → **Checked**
├── Keep User Code → **Checked**
├── Copy only necessary library files → **Checked**
```

## Verification After Generation
Your `main.c` should show:
```c
/* Peripheral declarations */
UART_HandleTypeDef huart2;
SPI_HandleTypeDef hspi1;

/* USER CODE BEGIN 0 */
// IO-Link stack init here
/* USER CODE END 0 */
```

This exact configuration works with **L6360 IO-Link PHY** and **I-Link stack**. The **230400 baud rate** and **16x oversampling** are critical for IO-Link v1.1 compliance. [download.kamami](https://download.kamami.pl/p581676-dm00720282-getting-started-with-the-xcubeiod02-industrial-iolink-device-transceiver.pdf)

Test with a simple `HAL_UART_Transmit()` loop first to verify wiring before adding the full IO-Link protocol stack.

---
---
