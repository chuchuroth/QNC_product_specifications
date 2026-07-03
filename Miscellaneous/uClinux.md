
### Comparison of Linux Distributions for STM32H7

The STM32H7 series (Cortex-M7 MCUs) lacks a full Memory Management Unit (MMU), so standard Linux distributions cannot run natively. Instead, distributions use **uClinux** (a MMU-less variant of Linux) with external SDRAM for viable operation. This setup is suitable for lightweight embedded applications like your robot's algorithm processing but requires careful configuration for bootloaders (e.g., U-Boot), kernels, and root filesystems.

Based on current (November 2025) resources, the primary options are:
- **Emcraft Linux BSP**: A commercial, pre-configured uClinux package optimized for STM32H7 boards.
- **Buildroot**: An open-source build system with community configs for STM32H7, allowing custom uClinux images.

Yocto Project has limited MCU support (better for ST's STM32MP1 MPUs with Cortex-A cores) and is not recommended here due to complexity and incomplete H7 layers. Other ports (e.g., custom kernels) exist but are niche.

| Aspect              | Emcraft Linux BSP                                                                 | Buildroot for STM32H7                                                                 |
|---------------------|-----------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|
| **Type/Base**      | uClinux (MMU-less Linux) with Buildroot-derived rootfs and U-Boot bootloader.    | uClinux via custom defconfigs; generates kernel, rootfs, and bootloader (e.g., U-Boot). |
| **Kernel Version** | 5.15 (stable, with H7-specific patches for peripherals).                          | Up to 6.6+ (2024.02 release; configurable to latest via upstream STM32 support).     |
| **Supported Boards**| STM32H753I-EVAL, H7-EVAL; portable to custom H7 designs with services.            | STM32H743/753 variants (e.g., Nucleo-H743ZI, Discovery boards) via community repos.   |
| **Key Features**   | Ethernet/TFTP boot, UART/QSPI boot, CAN/USB HS/gadget, I2C/GPIO, FPU support, multi-threading, C++/Micropython (upcoming), remote GDB debugging. Pre-built images for quick start. | Core peripherals (Ethernet, USB, CAN, I2C, PWM/ADC for motors), networking, filesystems (initramfs or ext4). Highly customizable packages (e.g., add Qt for GUIs, drivers for sensors). No pre-builts—build from source. |
| **Ease of Setup**  | High: Pre-built binaries, 5-minute boot guide, Eclipse/GDB integration. Build from source optional. | Medium: Requires menuconfig for H7 defconfig, cross-compilation setup. Steeper learning curve but straightforward for experienced users. |
| **Performance/Size**| Optimized for H7 (e.g., 480 MHz core utilization); rootfs ~10-20 MB, boot in seconds. Low overhead for real-time hybrids (e.g., with FreeRTOS on M4 co-core). | Flexible: Minimal images (~5-15 MB) for low-resource; supports optimization flags. Comparable performance but manual tuning needed. |
| **Support/Community**| Commercial: 3 months included (installation/config); ongoing via contracts. Active GitHub for sources. | Open-source: ST community, Buildroot mailing lists, GitHub forks (e.g., for low-power H743). No official ST support for MCU Linux—community-driven. |
| **Cost/Licensing** | ~$500-1,000 (one-time BSP license; no royalties). Open-source components (GPL/LGPL). | Free (GPL). No licensing fees; full source access.                                           |
| **Build Time**     | 30-60 min on modern host (Buildroot-based).                                       | 20-90 min depending on config/packages; faster for minimal builds.                     |
| **Best For**       | Production robots needing quick prototyping, professional support, and board-specific drivers (e.g., NUCLEO-H743ZI integration). | Custom/open-source projects with heavy customization (e.g., adding robot libs like ROS2 mini); cost-sensitive hobbyist/dev work. |

### Recommendations
- **Choose Emcraft** if you're prioritizing time-to-market and support for your robot's Linux side (algorithms via Python/C++ apps). It integrates well with STM32Cube for hybrid M7/M4 setups (Linux on M7, FreeRTOS on M4).
- **Choose Buildroot** for flexibility and zero cost—start with a GitHub repo like [vietnhatthai/buildroot-STM32H7](https://github.com/vietnhatthai/buildroot-2024.02.2) and add packages for your needs (e.g., OpenCV for vision algorithms).
- **Caveats**: Expect ~100-200 MB external SDRAM for viable rootfs. Kernel upstreaming for H7 is progressing (e.g., H757 support in 6.11+), improving stability. For full Linux (with MMU), consider upgrading to STM32MP2 MPU series.

For setup guides: Emcraft [demo](https://www.emcraft.com/stm32h7-evk-board/running-linux-on-stm32h753i-eval-in-5-minutes); Buildroot [docs](https://buildroot.org/downloads/manual/manual.html). If you need ports for specific H7 variants, provide details!


---
