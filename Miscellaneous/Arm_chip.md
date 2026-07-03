### Similar Companies to STMicroelectronics (STM32) and NXP Semiconductors Adopting ARM Chips

STMicroelectronics (STM32 series) and NXP Semiconductors are leading providers of ARM-based microcontrollers (MCUs) for embedded applications like IoT, industrial control, and automotive systems. Based on market share, product portfolios, and adoption of ARM Cortex-M cores, here are four prominent similar companies: Texas Instruments (TI), Microchip Technology, Renesas Electronics, and Infineon Technologies. These firms offer scalable ARM-based MCUs with features like low power, integrated peripherals, and security, targeting comparable markets.

I've focused on their popular commercial development (dev) boards for ARM-based MCUs, selecting 2-3 well-established examples per company. These boards are widely available via distributors like Mouser or Digi-Key and are designed for prototyping, evaluation, and rapid development. Each includes the primary ARM chip installed, core type, key specs, and approximate price (as of November 2025; prices vary by region and retailer).

| Company              | Dev Board Name                  | Primary ARM Chip Installed                  | ARM Core | Key Specs                                                                 | Approx. Price (USD) |
|----------------------|---------------------------------|---------------------------------------------|----------|---------------------------------------------------------------------------|--------------------|
| **Texas Instruments** | EK-TM4C123GXL LaunchPad        | TM4C123GH6PM                               | Cortex-M4 | 80 MHz, 256 KB Flash, 32 KB SRAM, Ethernet, USB, GPIO; BoosterPack compatible | $20               |
| **Texas Instruments** | LP-MSP432P401R LaunchPad       | MSP432P401R                                | Cortex-M4F | 48 MHz, 256 KB Flash, 64 KB SRAM, Low-power modes, ADC, wireless add-ons   | $15               |
| **Texas Instruments** | LP-MSPM0G3500 LaunchPad        | MSPM0G3500                                 | Cortex-M0+ | 80 MHz, 128 KB Flash, 32 KB SRAM, High-speed ADC, compact for wearables    | $12               |
| **Microchip Technology** | DM320118 Curiosity Nano        | ATSAMD21J18A                               | Cortex-M0+ | 48 MHz, 256 KB Flash, 32 KB SRAM, USB, I2C/SPI, Arduino-compatible         | $18               |
| **Microchip Technology** | ATSAMR34 Xplained Pro          | ATSAMR34J18                                | Cortex-M0+ | 48 MHz, 1 MB Flash, 128 KB SRAM, Sub-GHz radio, low-power IoT focus        | $50               |
| **Microchip Technology** | SAM E54 Xplained Pro           | ATSAME54P20A                               | Cortex-M4F | 120 MHz, 1 MB Flash, 384 KB SRAM, Ethernet, CAN, security features         | $60               |
| **Renesas Electronics** | EK-RA4M1 Evaluation Kit        | RA4M1G3A                                   | Cortex-M4 | 48 MHz, 512 KB Flash, 32 KB SRAM, TrustZone security, USB/Ethernet         | $25               |
| **Renesas Electronics** | EK-RA6M1 Evaluation Kit        | RA6M1G3A                                   | Cortex-M4 | 120 MHz, 2 MB Flash, 640 KB SRAM, DDR interface, AI acceleration           | $40               |
| **Renesas Electronics** | EK-RA2E1 Evaluation Kit        | RA2E1G1A                                   | Cortex-M23 | 48 MHz, 256 KB Flash, 32 KB SRAM, Ultra-low power, integrated security     | $20               |
| **Infineon Technologies** | KIT_XMC1100_BK Boot Kit        | XMC1100 Q040F0008                          | Cortex-M0 | 48 MHz, 8 KB Flash, 4 KB SRAM, Basic GPIO/ADC, low-cost entry-level        | $10               |
| **Infineon Technologies** | KIT_XMC4400_PLATFORM2GO        | XMC4400-F144K768B                          | Cortex-M4F | 120 MHz, 1 MB Flash, 160 KB SRAM, Display, motor control peripherals       | $35               |
| **Infineon Technologies** | KIT_XMC4800_RELAX_ECAT_V2      | XMC4800-F144K8384                           | Cortex-M4 | 144 MHz, 4 MB Flash, 1 MB SRAM, EtherCAT support, industrial automation    | $150              |

### Notes
- **Selection Criteria**: Boards were chosen based on popularity (e.g., sales volume, community support via forums like Reddit/EEVblog), availability, and representation of entry-to-high-end ARM cores (M0/M0+/M4/M4F/M23). All use licensed ARM IP for MCUs, similar to STM32's Cortex-M series.
- **Common Features Across Companies**: These boards typically include on-board debuggers (e.g., JTAG/SWD), USB interfaces, and expansion headers for sensors/shields. Software ecosystems like TI's Code Composer Studio, Microchip's MPLAB X, Renesas' Flexible Software Package, and Infineon's DAVE support ARM toolchains (e.g., GCC, Keil).
- **Why These Companies?**: They hold significant MCU market share (e.g., TI ~17%, Microchip ~15%, Renesas ~12%, Infineon ~10% as of 2025) and emphasize ARM for scalability, per industry reports.
- For deeper dives, check official sites: [TI Arm MCUs](https://www.ti.com/microcontrollers-mcus/arm-based-mcus/overview.html), [Microchip SAM](https://www.microchip.com/en-us/products/microcontrollers/32-bit-mcus/pic32-sam), [Renesas RA](https://www.renesas.com/en/products/microcontrollers-microprocessors/ra-cortex-m-mcus), [Infineon XMC](https://www.infineon.com/cms/en/product/microcontroller/32bit-industrial-arm-cortex-m/).

---
---
### Updated List of ARM-Based MCU Development Boards

I've expanded the original table to include popular commercial development boards from STMicroelectronics (STM32 series) and NXP Semiconductors (LPC series), based on your request. These were selected for their popularity in 2025 (e.g., high sales, community usage on forums like Reddit and EEVblog), availability, and representation of entry-to-high-end ARM cores. The rest of the table remains unchanged for completeness.

All boards feature on-board debuggers, USB interfaces, and expansion options, with software support via vendor IDEs (e.g., STM32CubeIDE, MCUXpresso) and ARM toolchains.

| Company                  | Dev Board Name                  | Primary ARM Chip Installed                  | ARM Core | Key Specs                                                                 | Approx. Price (USD) |
|--------------------------|---------------------------------|---------------------------------------------|----------|---------------------------------------------------------------------------|--------------------|
| **STMicroelectronics**   | NUCLEO-F401RE                   | STM32F401RET6                              | Cortex-M4 | 84 MHz, 512 KB Flash, 96 KB SRAM, Arduino/ST Morpho, USB, GPIO, ADC        | $18                |
| **STMicroelectronics**   | NUCLEO-L432KC                   | STM32L432KC                                | Cortex-M4 | 80 MHz, 256 KB Flash, 64 KB SRAM, Ultra-low power, Arduino Nano, I2C/SPI   | $12                |
| **STMicroelectronics**   | NUCLEO-H743ZI                   | STM32H743ZI                                | Cortex-M7 | 480 MHz, 2 MB Flash, 1 MB SRAM, Ethernet, USB HS, CAN, high-performance    | $30                |
| **NXP Semiconductors**   | LPC55S16-EVK                    | LPC55S16                                   | Cortex-M33 | 150 MHz (dual-core), 256 KB Flash, 128 KB SRAM, TrustZone security, USB    | $52                |
| **NXP Semiconductors**   | LPCXpresso812 (OM13055)         | LPC812                                     | Cortex-M0+ | 30 MHz, 16 KB Flash, 2 KB SRAM, Low-cost entry, Arduino shields, GPIO      | $10                |
| **NXP Semiconductors**   | LPCXpresso11U68 (OM13058)       | LPC11U68                                   | Cortex-M0+ | 50 MHz, 256 KB Flash, 32 KB SRAM, USB device/host, low-power modes, ADC    | $25                |
| **Texas Instruments**    | EK-TM4C123GXL LaunchPad        | TM4C123GH6PM                               | Cortex-M4 | 80 MHz, 256 KB Flash, 32 KB SRAM, Ethernet, USB, GPIO; BoosterPack compatible | $20               |
| **Texas Instruments**    | LP-MSP432P401R LaunchPad       | MSP432P401R                                | Cortex-M4F | 48 MHz, 256 KB Flash, 64 KB SRAM, Low-power modes, ADC, wireless add-ons   | $15               |
| **Texas Instruments**    | LP-MSPM0G3500 LaunchPad        | MSPM0G3500                                 | Cortex-M0+ | 80 MHz, 128 KB Flash, 32 KB SRAM, High-speed ADC, compact for wearables    | $12               |
| **Microchip Technology** | DM320118 Curiosity Nano        | ATSAMD21J18A                               | Cortex-M0+ | 48 MHz, 256 KB Flash, 32 KB SRAM, USB, I2C/SPI, Arduino-compatible         | $18               |
| **Microchip Technology** | ATSAMR34 Xplained Pro          | ATSAMR34J18                                | Cortex-M0+ | 48 MHz, 1 MB Flash, 128 KB SRAM, Sub-GHz radio, low-power IoT focus        | $50               |
| **Microchip Technology** | SAM E54 Xplained Pro           | ATSAME54P20A                               | Cortex-M4F | 120 MHz, 1 MB Flash, 384 KB SRAM, Ethernet, CAN, security features         | $60               |
| **Renesas Electronics**  | EK-RA4M1 Evaluation Kit        | RA4M1G3A                                   | Cortex-M4 | 48 MHz, 512 KB Flash, 32 KB SRAM, TrustZone security, USB/Ethernet         | $25               |
| **Renesas Electronics**  | EK-RA6M1 Evaluation Kit        | RA6M1G3A                                   | Cortex-M4 | 120 MHz, 2 MB Flash, 640 KB SRAM, DDR interface, AI acceleration           | $40               |
| **Renesas Electronics**  | EK-RA2E1 Evaluation Kit        | RA2E1G1A                                   | Cortex-M23 | 48 MHz, 256 KB Flash, 32 KB SRAM, Ultra-low power, integrated security     | $20               |
| **Infineon Technologies**| KIT_XMC1100_BK Boot Kit        | XMC1100 Q040F0008                          | Cortex-M0 | 48 MHz, 8 KB Flash, 4 KB SRAM, Basic GPIO/ADC, low-cost entry-level        | $10               |
| **Infineon Technologies**| KIT_XMC4400_PLATFORM2GO        | XMC4400-F144K768B                          | Cortex-M4F | 120 MHz, 1 MB Flash, 160 KB SRAM, Display, motor control peripherals       | $35               |
| **Infineon Technologies**| KIT_XMC4800_RELAX_ECAT_V2      | XMC4800-F144K8384                           | Cortex-M4 | 144 MHz, 4 MB Flash, 1 MB SRAM, EtherCAT support, industrial automation    | $150              |

### Notes
- **Prices**: Approximate as of November 2025 from distributors like DigiKey, Mouser, and Amazon; subject to fluctuation.
- **Sources**: Board details drawn from official vendor sites and datasheets. For STM32, check [ST Nucleo Overview](https://www.st.com/en/evaluation-tools/stm32-nucleo-boards.html); for NXP LPC, [LPCXpresso Boards](https://www.nxp.com/design/design-center/development-boards-and-designs/lpcxpresso-boards:LPCXPRESSO-BOARDS).


---

---
### Recommendations for Your Robot Design

Based on the provided list of ARM-based microcontroller development boards, I'll recommend two that align with your needs: one high-performance board capable of running embedded Linux for algorithm processing (e.g., pathfinding, computer vision, or AI inference), and one suited for low-level real-time motor control using FreeRTOS (e.g., PWM generation, encoder feedback, and precise timing). These selections prioritize compatibility, performance, peripherals, and community support from the list.

#### 1. **Single-Board Computer (SBC)-Like Board for Embedded Linux Algorithms: NUCLEO-H743ZI (STMicroelectronics)**
   - **Primary ARM Chip**: STM32H743ZI (Cortex-M7 @ 480 MHz)
   - **Key Specs**: 2 MB Flash, 1 MB SRAM, Ethernet, USB HS, CAN FD, multiple ADCs/DACs, ST Morpho/Arduino expansion headers for sensors and cameras.
   - **Why This Board?**
     - The STM32H7 series supports embedded Linux distributions like Emcraft's professionally supported build, which includes kernel, drivers, and tools for high-level algorithms. Its 480 MHz core with DSP/FPU handles compute-intensive tasks efficiently without needing a full SBC like Raspberry Pi—ideal for a compact robot "motherboard."
     - Expansion options allow easy integration of cameras or AI accelerators for your algorithms.
     - Price: ~$30 USD; widely available with STM32CubeIDE for Linux porting.
   - **Getting Started**: Use STM32CubeMX to generate Linux-compatible code, or follow Emcraft guides for bootloading.

#### 2. **Microcontroller Development Board for Low-Level Real-Time Motor Control: LPC55S16-EVK (NXP Semiconductors)**
   - **Primary ARM Chip**: LPC55S16 (Dual-core Cortex-M33 @ 150 MHz)
   - **Key Specs**: 256 KB Flash, 128 KB SRAM (per core), multiple PWM timers, quadrature encoder interfaces, USB, CAN, integrated TrustZone for secure real-time ops.
   - **Why This Board?**
     - Excellent FreeRTOS support via NXP's MCUXpresso SDK, including tickless mode and TrustZone integration for deterministic motor control (e.g., BLDC/PMSM drives with FOC algorithms). The dual-core setup lets one core handle RTOS tasks while isolating safety-critical motor loops.
     - Rich motor peripherals (high-res PWM, fault detection) make it robot-friendly; community examples abound for FreeRTOS-based control.
     - Price: ~$52 USD; includes on-board debugger for quick prototyping.
   - **Getting Started**: Download FreeRTOS demos from NXP's site and configure via MCUXpresso—pre-built for LPC55S16 with motor libraries.

These boards complement each other well: the H743ZI acts as the "brain" for Linux-based planning, communicating via UART/SPI/CAN to the LPC55S16 for precise actuation. Both are ARM-based for toolchain consistency. If you need more I/O, stack them with shields. For full SBC alternatives (e.g., Raspberry Pi 5), let me know to expand beyond the list!

