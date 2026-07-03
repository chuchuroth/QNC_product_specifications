https://neurarobotics.atlassian.net/wiki/spaces/N/pages/2417721431/WPS

---



**以下是《work_package.txt》文档的完整中文翻译**（翻译力求准确、专业、流畅，保留所有技术术语的规范性）：

---

这份文档概述了 **2026年第二季度** 的**综合技术路线图**，重点是将 **IQ10平台计划** 迁移到 **QCS 6490处理器**。主要目标是将现有的机器人软件（包括 **Motion Master HAL** 和 **Robot Model v4**）从 ARM CM4 和 x86 架构迁移到基于 **Xenomai 3.1** 的**无头、实时 Linux 环境**。该策略优先开发**工业协作者机器人和认知协作者机器人**，同时通过**模块化、容器化架构**为未来的**人形机器人和四足机器人**项目奠定基础。关键基础设施升级包括集成 **FastDDS** 以实现零拷贝通信，并验证 **TSN** 用于高速网络。最终，该工作包作为一项**关键工程蓝图**，旨在在后续季度扩展生态系统之前，锁定硬件规格和软件基线。

---

**“work_package.txt”** 来源文件概述了 **2026年第二季度 IQ10平台计划**，这是一个为期90天的执行计划（4月8日–7月7日），重点是将机器人计算迁移到 **QCS 6490芯片**。

文档主要内容如下：

### **1. 范围与主要目标**
- **目标平台**：重点发布基于 QCS 6490芯片的 **LARA（工业协作者机器人）** 和 **MAiRA（认知协作者机器人）**。
- **Smart Limb SOM概念**：路线图建立了一个统一的硬件基础，即所有平台（包括人形机器人和四足机器人）均使用相同的 **QCS 6490硬件、载板和板级支持包（BSP）**。
- **无头架构**：所有计算单元（CU）均为**无头**设计，不具备本地图形界面；外部客户端（笔记本电脑/平板电脑）通过基于 DDS 的 API 与机器人进行交互。

### **2. 关键路径：Xenomai 实时内核**
- **Xenomai 3.1 Cobalt**：移植该实时协内核是**绝对关键路径**（WP-01）。
- **性能目标**：必须实现最坏情况下的 RT 线程延迟 **< 10 μs**，以解锁本季度几乎所有其他工作包。

### **3. 软件与中间件架构**
- **3-Docker模型**：软件被划分为三个容器：**硬件接口**（驱动/HAL）、**伺服逻辑**（动力学/控制）和**应用**（规划/用户任务）。
- **NeuraSync & FastDDS**：通信依赖 **FastDDS 3.4.1**，使用共享内存（SHM）零拷贝实现计算单元内部高性能，并通过 **TSN（时间敏感网络）** 实现多计算单元同步。
- **统一 IDL 契约**：一套标准化的 DDS 消息定义，允许软件从 7 自由度协作者机器人扩展到 30+ 自由度人形机器人，使用**固定大小数组**以避免动态内存分配。

### **4. 硬件抽象（Motion Master）**
- **多总线支持**：Motion Master HAL 正在被移植到 QCS 6490，以提供**总线无关的接口**。
- **协议后端**：虽然 EtherCAT 是协作者机器人的主要协议，但路线图包括为四足机器人和人形机器人添加 **CANFD 和 CANopen** 后端。

### **5. 控制与动力学（Robot Model v4）**
- **ARM 移植**：将基于 **Pinocchio** 的动力学引擎移植到 ARM 架构，对于在目标芯片上运行逆运动学（IK）、逆动力学（ID）和全身控制（WBC）至关重要。
- **控制链**：路线图遵循从基础关节级控制（摩擦补偿）到高级全身动力学和轨迹规划的顺序“关键链”。

### **6. 战略决策关口**
90天窗口内包含三个关键的“转向关口”，用于决定未来的技术方向：
- **控制模式验证（M10）**：基于位置、力矩和阻抗模式性能，对协作者机器人发布进行 Go/No-Go 决策。
- **NPU 抖动评估（M5）**：判断 QCS 6490 NPU 是否能以 **< 5% 抖动** 运行强化学习（RL）策略；否则需转向 CPU 或外部加速器。
- **TSN Go/No-Go（M9）**：评估 TSN 是否对多计算单元实时通信严格必要。

### **7. 开发者工具与基础设施**
- **统一构建系统**：迁移到基于 **Docker 的交叉编译工具链**，应用使用 CMake，操作系统镜像使用 Yocto。
- **HIL 测试**：建立**硬件在环自动回归测试套件**，延迟回归超过 5% 将自动阻止代码合并。
- **机器人无关 API**：重构 C++ API，为机器人舰队中的任意机器人类型提供通用命令集。

---

### **QCS 6490芯片的统一作用**

QCS 6490芯片通过作为 **Smart Limb SOM（系统级模块）概念** 的标准化计算核心，统一了不同机器人平台。该概念利用相同硬件、载板和板级支持包（BSP）覆盖所有平台。

统一的具体层面包括：

#### **1. 硬件与操作系统标准化**
- **Smart Limb SOM**：所有平台——从工业协作者机器人（LARA）到人形机器人（4NE1）——均使用完全相同的 QCS 6490硬件。这使得在一个机器人上验证的控制策略可以立即部署到其他机器人上。
- **统一 BSP**：针对 QCS 6490 开发单一的基于 Yocto 的板级支持包（`meta-neura-robotics`），使用设备树叠加层（device tree overlays）处理产品特定的差异（如不同的传感器接口或通信网卡）。
- **实时协内核**：每个平台均在 QCS 6490 上运行 **Xenomai 3.1 Cobalt**，确保一致的硬实时性能，最坏情况延迟低于 10 μs。

#### **2. 总线无关的硬件抽象层（HAL）**
**Motion Master** HAL 被移植到 QCS 6490，以提供统一的接口，抽象掉物理通信总线。
- **多总线支持**：支持 **EtherCAT**（用于 LARA 和 MAiRA）、**CANFD**（用于四足机器人）和 **CANopen**（用于 4NE1 Mini）。
- **一致性 API**：高层代码通过单一 API 交互，而 Motion Master 处理具体的总线后端、错误模型（例如将 CAN bus-off 映射为 EtherCAT 故障）以及传感器读取。

#### **3. 统一中间件与通信**
- **NeuraSync 和 DDS**：所有组件间通信均使用 NeuraSync（基于 FastDDS 3.4.1 的抽象层）。它支持计算单元内部的**共享内存零拷贝**，以及针对需要多个 QCS 6490 单元的机器人（如拥有八个 SoC 的人形机器人）的 **TSN/UDP** 通信。
- **可扩展的 IDL 契约**：统一的接口定义语言（IDL）契约定义了 DDS 主题，可从 7-DOF 协作者机器人扩展到 30+ DOF 人形机器人，使用固定大小数组避免动态分配。

#### **4. 通用控制框架**
- **Robot Model v4**：该后端（基于 Pinocchio）被移植到 QCS 6490 的 ARM 架构。它提供一套标准化的求解器（FK、IK、QP、TSID），能够加载任意机器人的 URDF 文件。
- **3-Docker 模型**：无论机器人类型如何，软件架构均划分为三个标准容器——**硬件接口、伺服逻辑和应用**。对于人形机器人等复杂平台，伺服逻辑容器可替换为 **RL Policy 容器**，同时保持与 HAL 的相同接口。

#### **5. 机器人无关 API**
一个通用的 C++ API 为外部客户端（笔记本电脑或平板电脑）提供一致的方式与任何无头 QCS 6490 计算单元进行交互。它包含**机器人无关的基础功能**（例如 `getRobotState()`），并针对特定产品线提供专门扩展（如人形机器人的步态阶段、协作者机器人的关节空间控制）。

---

### **2026年 IQ10 平台路线图结构**

路线图围绕一个90天执行窗口（2026年4月8日至7月7日）展开，包含十二个关键里程碑（**M1–M12**）和若干“关口”决策，用于决定全年剩余时间的技术方向。

#### **关键路径与关键链**
软件路线图由两个主要依赖驱动：
- **关键路径（平台）**：将 **Xenomai 3.1 Cobalt 协内核** 移植到 QCS 6490（**M2**）。这几乎解锁了所有其他工作包。
- **关键链（控制）**：从 **Motion Master 移植**（**M4**）开始，依次经过**关节级控制逻辑**（**M8**）、**全身控制**（**M11**）直至**规划集成**（**M12**）。

#### **分阶段里程碑分解**

**阶段 1：基础建设（4月）**
- **M1：硬件与技术栈冻结**（4月18日）：锁定 QCS 6490 计算单元的 BOM 和引脚定义。
- **M2：内核与工具链**（4月25日）：在 QCS 6490 上启动 Xenomai，最坏情况延迟 **< 10 μs**，并验证交叉编译工具链。
- **M3：统一软件架构**（5月2日）：最终确定 **3-Docker 模型**（硬件接口、伺服逻辑、应用）以及库的划分。

**阶段 2：抽象与集成（5月）**
- **M4：HAL 与 rollout 规划**（5月9日）：将 EtherCAT 主站移植到新芯片，并定义不同产品线的计算单元部署顺序。
- **M5：评估与移植**（5月16日）：将 **Robot Model v4**（Pinocchio/Eigen）移植到 ARM 架构，并验证 **Orocos RT 框架**。
- **M6：基线冻结与测试**（5月23日）：冻结 **NEURON OS 基线**（Ubuntu 18.04 + Xenomai + Docker），并建立硬件在环（HIL）自动回归测试套件。
- **M7：多计算单元与 API 扩展**（5月30日）：扩展 **NeuraSync** 以支持多计算单元通信，并将 C++ API 重构为机器人无关。

**阶段 3：控制与验证（6月–7月）**
- **M8：关节级控制**（6月6日）：在实时循环中验证摩擦补偿和执行器阻抗控制。
- **M9：规格制定与 TSN 决策**（6月13日）：完成 **DDS/TSN 基准测试套件**，并起草 v1 规格文档。
- **M10：控制模式验证**（6月20日）：在协作者机器人硬件上验证**位置、力矩和阻抗**控制模式的重要关口。
- **M11：动力学与上游贡献**（6月27日）：在目标芯片上运行完整逆动力学（RNEA）和全身控制（WBC）。
- **M12：规划集成**（7月4日）：集成轨迹展开和轨迹中段融合功能。

#### **关键决策关口（转向点）**
1. **NPU 与 RL 抖动评估（M5）**：判断 QCS 6490 NPU 是否能以 **< 5% 抖动** 运行人形机器人行走策略。若失败，则转向 CPU-only 或外部加速器。
2. **TSN Go/No-Go（M9）**：决定 **时间敏感网络（TSN）** 是否对多计算单元实时通信（尤其是高自由度人形机器人）是强制要求的。
3. **控制模式验证（M10）**：针对 LARA/MAiRA 协作者机器人在 QCS 6490 平台上发布的最终“Go/No-Go”决策。

#### **2026年下半年及以后**
- **2026年第三季度**：扩展至 **MiPA（移动操作臂）**、**MAV（移动车辆）** 和**四足机器人**平台。
- **2026年第四季度至2027年第一季度**：完成 **4NE1 Mini** 和 **4NE1 Gen 3.5** 人形机器人舰队的集成。

---

**其他重要模块翻译摘要**（保持简洁准确）：

### **Motion Master 多总线支持**
Motion Master 作为**硬件抽象层（HAL）**，采用多总线抽象设计，使上层软件完全独立于具体总线，通过统一的 API 进行交互。它支持 EtherCAT（协作者机器人主用）、CANFD（四足机器人）和 CANopen（4NE1 Mini 人形机器人），并通过配置文件实现运行时总线选择，同时提供一致的错误建模和传感器抽象。

### **机器人无关 C++ API**
该 API 是外部客户端与无头 QCS 6490 计算单元交互的主要接口。它采用模块化架构，包含适用于所有平台的**机器人无关基础**（如 `getRobotState()`、`sendCommand()`），并针对不同机器人类型提供专门扩展（协作者：关节/笛卡尔/力控；人形：重心分配、步态阶段等）。通过 NeuraSync（DDS）通信，支持编译期分发，无运行时开销。

### **统一 IDL 契约（WP-15）**
使用 OMG IDL 4.2 作为单一事实来源，定义所有 DDS 主题。采用固定大小的 `std::array` 以支持从 7-DOF 到 30+ DOF 的可扩展性，实现共享内存零拷贝，适用于实时控制路径。主题命名遵循 `/{产品}/{域}/{主题}` 规范，并与 NeuraSync 的 QoS 配置紧密集成。

### **Xenomai 3.1 Cobalt（WP-01）**
作为整个路线图的**绝对关键路径**，其移植工作解锁了几乎所有其他工作包。它负责提供硬实时能力，目标最坏情况延迟 **< 10 μs**，是稳定执行器控制和整个软件栈的基础。

### **Sensor Abstraction Layer（传感器抽象层，WP-13）**
提供统一的传感器发现、访问和读取 API，支持不同总线（EtherCAT、CANFD 等）和不同平台（F/T 传感器、IMU、足压传感器等），所有数据通过 FastDDS 以固定频率发布（F/T 1kHz、IMU 500Hz 等），并保证实时安全读取和优雅降级。

### **Smart Limb SOM 概念**
以 QCS 6490 为核心的标准化计算模块，所有平台共用相同硬件、载板和 NEURON OS（Ubuntu 18.04 + Xenomai 3.1）。通过设备树叠加层处理产品差异，实现“垂直策略包”即插即用，支持从传统控制到强化学习策略的切换，并通过 TSN 以太网实现多 SOM 网络化。

### **3-Docker 软件模型（WP-14）**
将软件划分为三个标准容器：
1. **HW Interface**：包含 Motion Master、总线驱动和传感器驱动；
2. **Servo Logic**：运行 Robot Model v4 动力学引擎，进行控制循环计算；
3. **Application**：负责轨迹规划、遥操作和用户应用。

容器间通过 NeuraSync DDS 通信，支持共享内存零拷贝。不同机器人可灵活调整容器分布（如人形机器人可将 Servo Logic 替换为 RL Policy 容器）。所有容器均为无头设计，外部通过 DDS 网络访问。

---

翻译完成。如需对特定章节进行更详细的逐句润色、制作中英对照版，或提取特定部分单独使用，请随时告诉我！

---
---
---
This document outlines a **comprehensive technical roadmap** for the second quarter of 2026, focusing on migrating the **IQ10 platform initiative** to the **QCS 6490 processor**. The primary objective is to transition existing robotic software—including the **Motion Master HAL** and **Robot Model v4**—from ARM CM4 and x86 architectures to a **headless, real-time Linux environment** powered by **Xenomai 3.1**. The strategy prioritizes the development of **industrial and cognitive cobots** while establishing the groundwork for future **humanoid and quadruped** projects through a modular, **containerized architecture**. Key infrastructure upgrades include the integration of **FastDDS for zero-copy communication** and the validation of **TSN for high-speed networking**. Ultimately, this work package serves as a **critical engineering blueprint** to lock in hardware specifications and software baselines before expanding the ecosystem in the following quarters.

---
The "work_package.txt" source outlines the **IQ10 Platform Initiative for Q2 2026**, a 90-day execution plan (April 8 – July 7) focused on transitioning robot compute to the **QCS 6490 chip**. 

The main points of the document are as follows:

### **1. Scope and Primary Objectives**
*   **Target Platforms:** The primary focus is the release of **LARA (Industrial Cobot)** and **MAiRA (Cognitive Cobot)** on the QCS 6490 chip.
*   **Smart Limb SOM Concept:** The roadmap establishes a unified hardware foundation where the **same QCS 6490 hardware, carrier board, and Board Support Package (BSP)** are used across all platforms, including humanoids and quadrupeds.
*   **Headless Architecture:** All Computing Units (CUs) are **headless**, meaning they have no local GUI; external clients (laptops/tablets) interact with the robots via a DDS-based API.

### **2. The Critical Path: Xenomai RT Kernel**
*   **Xenomai 3.1 Cobalt:** Porting this real-time co-kernel is the **absolute critical path** (WP-01). 
*   **Performance Target:** It must achieve a worst-case RT thread latency of **< 10 us** to unlock nearly every other work package in the quarter.

### **3. Software and Middleware Architecture**
*   **3-Docker Model:** The software is partitioned into three containers: **Hardware Interface** (drivers/HAL), **Servo Logic** (dynamics/control), and **Application** (planning/user tasks).
*   **NeuraSync & FastDDS:** Communication relies on **FastDDS 3.4.1** with shared-memory (SHM) zero-copy for intra-CU performance and **TSN (Time-Sensitive Networking)** for multi-CU synchronization.
*   **Unified IDL Contract:** A standardized set of DDS message definitions allows the software to scale from 7-DOF cobots to 30+ DOF humanoids using **fixed-size arrays** to avoid dynamic memory allocation.

### **4. Hardware Abstraction (Motion Master)**
*   **Multi-Bus Support:** The Motion Master HAL is being ported to QCS 6490 to provide a **bus-agnostic interface**.
*   **Protocol Backends:** While EtherCAT is the primary focus for cobots, the roadmap includes adding **CANFD and CANopen** backends for quadrupeds and humanoids.

### **5. Controls and Dynamics (Robot Model v4)**
*   **ARM Porting:** Porting the **Pinocchio-based dynamics engine** to ARM architecture is essential for running Inverse Kinematics (IK), Inverse Dynamics (ID), and Whole-Body Control (WBC) on the target chip.
*   **Control Chain:** The roadmap follows a sequential "Critical Chain" from basic joint-level control (friction compensation) to advanced whole-body dynamics and trajectory planning.

### **6. Strategic Decision Gates**
The 90-day window includes three critical "Pivot Gates" that determine future technical direction:
*   **Control Mode Validation (M10):** A Go/No-Go decision for the cobot release based on position, torque, and impedance mode performance.
*   **NPU Jitter Evaluation (M5):** Determines if the QCS 6490 NPU can run Reinforcement Learning (RL) policies with **< 5% jitter**; otherwise, a pivot to CPU or external accelerators is required.
*   **TSN Go/No-Go (M9):** Evaluates if TSN is strictly necessary for multi-CU real-time communication.

### **7. Developer Tools and Infrastructure**
*   **Unified Build System:** Migration to a **Docker-based cross-compile toolchain** using CMake for apps and Yocto for OS images.
*   **HIL Testing:** Establishing a **Hardware-in-the-Loop automated regression suite** where latency regressions > 5% automatically block code merges.
*   **Robot-Agnostic API:** Refactoring the C++ API to provide a universal set of commands for any robot type in the fleet.

---
---

---
The QCS 6490 chip unifies different robot platforms by serving as the standardized compute core for the **Smart Limb SOM (System on Module) concept**, which utilizes the same hardware, carrier board, and Board Support Package (BSP) across every platform. 

The sources detail several layers of this unification:

### 1. Hardware and OS Standardisation
*   **Smart Limb SOM:** All platforms—from industrial cobots (LARA) to humanoids (4NE1)—share the identical QCS 6490 hardware. This allows a control policy refined on one robot to be immediately deployable to others.
*   **Unified BSP:** A single Yocto-based Board Support Package (`meta-neura-robotics`) is adapted for the QCS 6490, using device tree overlays to handle product-specific variations like different sensor interfaces or communication NICs.
*   **RT Co-Kernel:** Every platform runs **Xenomai 3.1 Cobalt** on the QCS 6490 to ensure consistent hard real-time performance with worst-case latency under 10 us.

### 2. Bus-Agnostic Hardware Abstraction (HAL)
The **Motion Master** HAL is ported to the QCS 6490 to provide a unified interface that abstracts away physical communication buses.
*   **Multi-Bus Support:** It supports **EtherCAT** (for LARA and MAiRA), **CANFD** (for quadrupeds), and **CANopen** (for 4NE1 Mini).
*   **Consistent API:** High-level code interacts with a single API, while Motion Master handles the specific bus backend, error models (e.g., mapping CAN bus-off to EtherCAT faults), and sensor reads.

### 3. Unified Middleware and Communication
*   **NeuraSync and DDS:** All inter-component communication uses NeuraSync (an abstraction over FastDDS 3.4.1). It facilitates **SHM zero-copy** for internal communication and **TSN/UDP** for robots requiring multiple QCS 6490 units, such as humanoids with eight SoCs.
*   **Scalable IDL Contract:** A unified IDL (Interface Definition Language) contract defines DDS topics that scale from a 7-DOF cobot to a 30+ DOF humanoid using fixed-size arrays to avoid dynamic allocation.

### 4. Common Control Framework
*   **Robot Model v4:** This backend (utilizing Pinocchio) is ported to the QCS 6490's ARM architecture. It provides a standardized suite of solvers (FK, IK, QP, TSID) that can load an arbitrary URDF for any robot type.
*   **3-Docker Model:** The software architecture is partitioned into three standard containers—**HW Interface, Servo Logic, and Application**—regardless of the robot type. For more complex robots like humanoids, the Servo Logic container can be replaced by an **RL Policy container** while maintaining the same interface to the HAL.

### 5. Robot-Agnostic API
A generic C++ API provides a consistent way for external clients (like tablets or laptops) to interact with any headless QCS 6490 unit. It features a **robot-agnostic base** (e.g., `getRobotState()`) with specialized extensions for specific product lines, such as gait phase for humanoids or joint-space control for cobots.


---
The 2026 roadmap for the IQ10 Platform is structured around a 90-day execution window (April 8 to July 7, 2026) defined by twelve key milestones (**M1–M12**) and several "Gate" decisions that determine the technical direction for the remainder of the year,.

### **The Critical Path and Chain**
The software roadmap is driven by two primary dependencies:
*   **Critical Path (Platform):** Porting the **Xenomai 3.1 Cobalt co-kernel** to the QCS 6490 (**M2**). This unlocks nearly all other work packages, including the OS baseline and hardware abstraction,.
*   **Critical Chain (Control):** A sequential progression from the **Motion Master port** (**M4**) through **Joint-Level Control Logic** (**M8**) to **Whole-Body Control** (**M11**) and **Planning Integration** (**M12**).

---

### **Chronological Milestone Breakdown**

#### **Phase 1: Foundation (April)**
*   **M1: Hardware & Tech Stack Freeze (Apr 18):** Locking the BOM and pinouts for the QCS 6490 CU.
*   **M2: Kernel & Toolchain (Apr 25):** Booting Xenomai on the QCS 6490 with worst-case latency **< 10 us** and validating the cross-compile toolchain,.
*   **M3: Unified SW Architecture (May 2):** Finalizing the **3-Docker model** (HW Interface, Servo Logic, and Application) and library partitioning.

#### **Phase 2: Abstraction & Integration (May)**
*   **M4: HAL & Rollout Planning (May 9):** Porting the EtherCAT master to the new chip and defining the CU rollout sequence for different product lines,.
*   **M5: Evaluation & Porting (May 16):** Porting **Robot Model v4** (Pinocchio/Eigen) to ARM and validating the **Orocos RT framework**,.
*   **M6: Baseline Freeze & Testing (May 23):** Freezing the **NEURON OS baseline** (Ubuntu 18.04 + Xenomai + Docker) and establishing the HIL automated regression suite,.
*   **M7: Multi-CU & API Expansion (May 30):** Extending **NeuraSync** for multi-CU communication and refactoring the C++ API to be robot-agnostic,.

#### **Phase 3: Control & Validation (June – July)**
*   **M8: Joint-Level Control (Jun 6):** Validating friction compensation and actuator impedance in the RT loop,.
*   **M9: Specification & TSN Decision (Jun 13):** Completing the **DDS/TSN benchmark suite** and drafting the v1 Spec Document,.
*   **M10: Control Mode Validation (Jun 20):** A major gate validating **Position, Torque, and Impedance** control modes on cobot hardware.
*   **M11: Dynamics & Upstreaming (Jun 27):** Running full inverse dynamics (RNEA) and WBC on the target chip.
*   **M12: Planning Integration (Jul 4):** Integrating trajectory unrolling and mid-trajectory blending.

---

### **Key Decision Gates (Pivot Points)**
The roadmap includes three critical gates that determine the Q3/Q4 strategy:
1.  **NPU & RL Jitter Evaluation (M5):** Determines if the QCS 6490 NPU can run humanoid walking policies with **< 5% jitter**. Failure results in a pivot to CPU-only or external accelerators,.
2.  **TSN Go/No-Go (M9):** Decides if **Time-Sensitive Networking (TSN)** is mandatory for multi-CU real-time communication, particularly for high-DOF robots like the humanoid,.
3.  **Control Mode Validation (M10):** A final "Go/No-Go" for the release of cobots (LARA/MAiRA) on the QCS 6490 platform,.

### **Beyond Q2 2026**
The roadmap extends into the latter half of the year with the following rollout targets:
*   **Q3 2026:** Expansion to the **MiPA** (Mobile Manipulator), **MAV** (Mobile Vehicle), and **Quadruped** platforms.
*   **Q4 2026 – Q1 2027:** Integration for the **4NE1 Mini** and **4NE1 Gen 3.5** humanoid fleets.


---
Motion Master supports different bus types by acting as a **Hardware Abstraction Layer (HAL)** that utilizes a multi-bus abstraction design. This architecture allows the upper software layers to remain completely bus-agnostic, interacting with a single, consistent API regardless of the underlying physical communication protocol.

The sources detail several mechanisms through which this support is implemented:

*   **Unified Bus-Agnostic Interface:** Motion Master provides a "bus-agnostic interface" where the high-level control code (such as the Servo Logic) sees the same API whether the robot uses EtherCAT, CANFD, or CANopen.
*   **Plug-in Backends:** It uses specific backends to interface with different hardware protocols. While **EtherCAT** is the primary HAL for cobots like LARA and MAiRA, **CANFD** and **CANopen** backends are added to support other platforms like quadrupeds and the 4NE1 Mini humanoid.
*   **Runtime Selection:** Bus types are selected at runtime via configuration files rather than being fixed at compile-time. This allows the same binary to work across different robot platforms simply by swapping the bus configuration.
*   **Consistent Error Modeling:** Motion Master maps different bus-specific errors to a unified internal model. For instance, it maps an EtherCAT fault and a CAN bus-off to the same internal error types so the robot's state machine can respond consistently.
*   **Sensor Abstraction:** The abstraction extends to sensors such as IMUs, encoders, and Force/Torque (F/T) sensors. Motion Master provides timestamped, typed sensor data through a uniform read API regardless of the physical interface used to collect that data.
*   **Swappable Internal Port Layer:** The internal port layer is designed to be swappable. This is critical for future roadmaps where the EtherCAT bus may move from the QCS 6490 to a dedicated TI AM2434 safety master, at which point Motion Master will transition to serving as a setpoint generator over PCIe or SPI.


---
The **robot-agnostic C++ API** is the primary interface used by external clients (such as laptops or tablets) to interact with headless QCS 6490 Computing Units (CUs). It is designed to provide a unified way to monitor, program, and diagnose any robot in the fleet—ranging from 6-DOF cobots to 30+ DOF humanoids—over a DDS network.

The API is being refactored in Q2 2026 under **Work Package 23 (WP-23)** to move beyond its previous cobot-only focus.

### **Core Structure and Capabilities**
The API follows a modular architecture that combines a universal base with specialized extensions:

*   **Robot-Agnostic Base:** Provides fundamental commands applicable to any platform, such as `getRobotState()`, `sendCommand()`, and `getCapabilities()`.
*   **Platform-Specific Extensions:** Specialized functions are available depending on the robot type:
    *   **Cobots (LARA/MAiRA):** Joint-space, Cartesian, and force control.
    *   **Humanoids (4NE1):** Center of Mass (CoM), wrench distribution, and gait phase.
    *   **Mobile Bases (MAV):** Velocity, odometry, and path status.
    *   **Quadrupeds:** Foot contact and terrain adaptation.

### **Technical Implementation**
*   **Decoupled Architecture:** The API communicates via **NeuraSync (DDS)** and has no direct dependency on lower-level libraries like Motion Master (HAL) or Robot Model v4 (dynamics).
*   **Zero Runtime Cost:** It utilizes **header-only compile-time dispatch**, ensuring that there is no runtime overhead for robot types that are not being used in a specific build.
*   **Versioning:** It employs semantic versioning with compile-time checks to ensure compatibility across software updates.
*   **Network-First Design:** Because all CUs are headless (no local GUI), the API is the sole gateway for external client access to perform tasks like live RT metric monitoring and teleop command routing.

The development of this API is blocked by the finalization of the **Unified SW Architecture (WP-14)** and the **Unified IDL Contract (WP-15)**, which define the underlying DDS message structures.


---
The **Unified IDL Contract (Work Package 15)** is a set of Interface Definition Language files that establish a standardized "contract" for every DDS topic used in communication between software containers. It is the foundational layer that allows the **robot-agnostic C++ API** to interact with different robot platforms using a consistent message structure.

Key aspects of this contract include:

### **1. Scalable Design**
The contract is designed to be universal, scaling from **7-DOF cobots** (like LARA) to **30+ DOF humanoids** (like 4NE1). It achieves this by using **fixed-size `std::array` types** governed by a "max DOF" compile-time constant, which avoids the need for dynamic memory allocation in the real-time hot path.

### **2. Technical Specifications**
*   **Source of Truth:** It uses **OMG IDL 4.2** as the source of truth, managed through an `interfaces.git` pipeline that automatically generates C++ and Python stubs.
*   **Real-Time Optimization:** To ensure high performance, the IDL types are structured to support **shared-memory (SHM) zero-copy** and avoid CDR serialization overhead during critical operations.
*   **Topic Naming:** All topics follow a standardized convention: `/{product}/{domain}/{topic}`.

### **3. Standardized Topic Schema**
The contract defines specific message types and rates for critical robot data, such as:
*   **`/joint_state`:** JointState[N] at 1 kHz (HAL to Servo).
*   **`/joint_command`:** JointCommand[N] at 1 kHz (Servo to HAL).
*   **`/ft_sensor`:** WrenchStamped at 1 kHz (SenseKit to Servo).
*   **`/cartesian_state`:** CartesianPose at 500 Hz (Servo to Application).
*   **`/planned_trajectory`:** JointTrajectory at 10-50 Hz (Application to Servo).

### **4. Integration with the Platform**
The contract is strictly versioned (semantic versioning) to prevent breaking changes across the fleet. It is integrated with **NeuraSync’s QoS profile system**, ensuring that RT control topics use "Best Effort" and "Volatile" settings while configuration topics use "Transient Local" settings. 

Because the **QCS 6490 Computing Units are headless**, this IDL contract is essential for external clients using the C++ API to reliably monitor, program, and diagnose the robot over the network.

---
The **Xenomai 3.1 Cobalt real-time co-kernel** (Work Package WP-01) is defined as the roadmap's **critical path** because it serves as the foundational enabler for nearly every other technical milestone in the Q2 2026 development cycle.

Its status as the critical path is driven by the following factors:

*   **Universal Dependency:** The successful porting of Xenomai to the QCS 6490 **unlocks 12 other major work packages**, including the OS baseline, hardware abstraction (Motion Master), robot dynamics (Robot Model v4), and multi-CU communication.
*   **Real-Time Requirement:** Xenomai is responsible for achieving the **hard real-time performance** necessary for robot control, specifically targeting a worst-case latency of **< 10 us**. Without this kernel baseline, the platform cannot guarantee the timing required for stable actuator control.
*   **The "Critical Chain":** Xenomai sits at the very start of the "Critical Chain" for cobot end-to-end functionality, which progresses from the kernel to the Motion Master port, through joint-level and whole-body control, and finally to planning integration.
*   **Middleware Integration:** The communication stack, including the **Orocos RT framework** and **NeuraSync (DDS)**, requires the Xenomai RT context to function, particularly for high-speed features like shared-memory zero-copy between real-time threads.
*   **Validation Gateway:** Key testing and validation packages, such as the Hardware-in-the-Loop (HIL) automated regression and the 3 us DDS latency target, are directly blocked until the Xenomai kernel is operational and providing a stable RT timing baseline.

Because **WP-01 is blocked by nothing** but blocks almost the entire software stack, any delay in booting Xenomai on the QCS 6490 would result in a day-for-day slip for the entire IQ10 platform roadmap.

---
The **Sensor Abstraction Layer** is a key extension of the Motion Master hardware abstraction layer, established under **Work Package 13 (WP-13)**. Its primary objective is to provide a unified API for all sensor inputs across different robot product lines, ensuring that high-level software remains independent of the specific physical sensor interfaces.

Key features and components of this layer include:

### **1. Unified Discovery and Access**
*   **SensorRegistry:** This component is responsible for discovering and enumerating all available sensors at startup based on the specific robot's configuration file.
*   **Uniform Read API:** It provides **timestamped, typed sensor data** through a consistent interface, regardless of whether the sensor is connected via EtherCAT, CANFD, or another physical bus.

### **2. Platform-Specific Sensor Sets**
The abstraction manages the varying sensor requirements of different robot types in the fleet:
*   **LARA (Industrial Cobot):** Focuses primarily on **Force/Torque (F/T)** sensors.
*   **MAiRA (Cognitive Cobot):** Supports **F/T sensors and cameras**.
*   **Quadrupeds:** Manages **IMUs (Inertial Measurement Units) and foot pressure sensors**.

### **3. Technical Integration and Performance**
*   **DDS Communication:** All sensor data is published via **FastDDS 3.4.1** using **shared-memory (SHM) zero-copy** with fixed-size IDL types to ensure high performance.
*   **Standardized Rates:** The layer enforces specific publishing frequencies, such as **1 kHz for F/T sensors**, **500 Hz for IMUs**, and **30 Hz for cameras**.
*   **Real-Time Reliability:** The system is designed for **RT-safe reads** with bounded worst-case performance. It also includes **graceful degradation** features, ensuring that missing or disconnected sensors do not cause the entire software stack to crash.

### **4. Bus-Agnostic Validation**
As part of the broader Motion Master validation (**WP-11**), the system ensures that critical components like IMUs, F/T sensors, and encoders can be read in a completely **bus-agnostic** manner, which is essential for maintaining a unified codebase across platforms like the 4NE1 humanoid and the LARA cobot.


---
The **Smart Limb SOM (System on Module) concept** is a standardized hardware and software framework designed to unify disparate robot platforms—from industrial cobots to humanoids—under a single compute architecture. It is centered on the **QCS 6490 chip**, which serves as the universal compute core.

The concept works through the following layers of standardization and modularity:

### **1. Universal Hardware and OS Baseline**
The core of the concept is the use of the **identical QCS 6490 hardware, carrier board, and Board Support Package (BSP)** across all platforms. 
*   **Unified OS:** Every "Smart Limb" runs the same **NEURON OS** stack, consisting of Ubuntu 18.04 and the **Xenomai 3.1 Cobalt** real-time co-kernel.
*   **Identical SOMs:** By using a "Smart Limb SOM reuse matrix," the company ensures that different platforms share the same physical SOM board, though they may utilize different carrier boards depending on the product's specific pinout needs.

### **2. Software Abstraction via Device Trees**
To manage the specific hardware variations of different robots (such as different sensors or communication buses) while using the same base OS, the concept employs a **device tree overlay system**. 
*   **Product-Specific Overlays:** A base device tree for the QCS 6490 is modified at build-time using overlays tailored for specific roles, such as a **cobot coordinator** (EtherCAT), a **SenseKit** (cameras), or a **humanoid/quadruped limb** (CANFD or CANopen).
*   **Build-Time Selection:** Engineers can select the target robot type in the Yocto build system (e.g., `MACHINE=qcs6490-lara-coord`), which automatically applies the correct hardware configuration.

### **3. Vertical Policy Swapping**
The Smart Limb SOM concept allows for high-level software portability by keeping the underlying infrastructure constant while only changing the "vertical policy package".
*   **Immediate Deployment:** A dexterous manipulation policy refined on a **LARA** or **MAiRA** cobot can be immediately deployed to the arms of a **4NE1 humanoid** because they share the same hardware and HAL.
*   **RL Policy Integration:** For complex platforms like humanoids or quadrupeds, the standard Servo Logic container (which uses traditional solvers) can be swapped for an **RL Policy container** that runs reinforcement learning models directly.

### **4. Scalable Multi-CU Architecture**
The concept enables robots to scale in complexity by networking multiple SOM units together using **NeuraSync (DDS)** over **TSN Ethernet**.
*   **Cobots:** Typically use two QCS 6490 units (one Coordinator and one SenseKit).
*   **Humanoids/Quadrupeds:** These robots utilize a multi-CU topology where a single coordinator manages **six individual Smart Limb SOMs**, providing dedicated compute for each limb.


---
The **3-Docker software model** is a containerized architecture designed to standardize the software topology across different robot platforms while maintaining strict real-time performance. Defined under **Work Package 14 (WP-14)**, this model partitions robot functionality into three distinct containers:

### **The Three Standard Containers**
1.  **Docker 1 — HW Interface:** This container houses the hardware abstraction layer (**Motion Master**), physical bus drivers (e.g., EtherCAT, CANFD), and sensor drivers.
2.  **Docker 2 — Servo Logic:** This container manages the core control loops, utilizing the **Robot Model v4** dynamics engine for Quadratic Programming (QP) solvers, Jacobian calculations, and joint-level control.
3.  **Docker 3 — Application:** This container handles high-level tasks such as trajectory planning, teleop command processing, and custom user application code.

### **Platform-Specific Implementations**
The model is flexible, allowing these containers to be distributed differently depending on the robot's hardware requirements:
*   **LARA (Industrial Cobot):** All three containers run on a single **QCS 6490 coordinator unit**.
*   **MAiRA (Cognitive Cobot):** Docker 1 (HW) and Docker 2 (Servo) run on the QCS 6490, while Docker 3 (Application) runs on an existing **Jetson module** connected via Ethernet.
*   **Humanoids and Quadrupeds:** For these platforms, the Servo Logic in Docker 2 can be swapped for an **RL Policy container**, which bypasses the traditional QP-based control path in favor of reinforcement learning policies.

### **Communication and Frameworks**
*   **Inter-Docker Communication:** Containers communicate using **NeuraSync DDS topics**.
*   **Real-Time Performance:** For containers on the same chip (intra-CU), communication uses **FastDDS shared-memory (SHM) zero-copy** between Xenomai threads to eliminate serialization overhead. For cross-CU communication, it utilizes **UDP or TSN**.
*   **Middleware Framework:** The **Orocos RT framework** is used within Docker 1 and Docker 2 for dataflow and task execution.

### **Headless Operation**
A key characteristic of this model is that all containers are **headless**, containing no GUI or display code. External clients, such as laptops or tablets, interact with the robot by connecting to the DDS network to monitor topics, program the robot, or run diagnostics.
