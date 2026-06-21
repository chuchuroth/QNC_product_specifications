这是目前对IO-Link讨论的总结 ：



---



## 1️⃣ 系统架构说明



* **新的 MCU 作为 IO-Link 主站**



  * 充当本地 Motion Master 与 QNC 其他部分之间的接口。
  * 功能类似于 Flange Interface Board。
  * **MotionMaster 和 EtherCAT 不再使用。**



* **数据流**



  * QNC 接收 **NeuraSync 数据包**，解析数据后使用 **IO-Link 主站栈**（例如 RT-Labs）生成 IO-Link 数据包。
  * 将 IO-Link 数据发送到机械手。



* **架构关键认知**



  * Raspberry Pi **不应**作为 IO-Link “核心”使用，仅作为 **命令发送端**。
  * IO-Link 协议转换由 **专用 IO-Link 主站硬件模块**处理。
  * 这一认知修正避免了 Pi 在工业环境中的误用。



---



## 2️⃣ 通信考量



* IO-Link 主站 MCU 与 QNC 其余系统的通信方式仍在澄清中。
* 可能无法直接使用 DDS 通信。



---



## 3️⃣ IO-Link 技术架构



* **主站要求**



  * 需要专用 MCU。
  * 无法直接处理来自 Raspberry Pi 的串行信号。
  * 启动时需加载 IO-Link 协议栈。



* **架构示意**



  CM5 (串行)
  
      ↓
      
  MCU (运行 IO-Link 协议栈)
  
      ↓
      
  IO-Link 芯片 (例如 MAX)



---



## 4️⃣ IO-Link 协议栈开发状态



* 新 TCP 的 IO-Link 协议栈开发仍处于 **初期阶段**：



  * 学习 IO-Link 基础知识。
  * 理解不同机械手的需求。
  * 对比现有协议栈。
  * 规划协议结构。



---



## 5️⃣ 协议栈选项与关注点



* **首选方案：** RT-Labs 协议栈



  * 第三方工业级方案。
  * RT-Labs 不提供硬件。



* **备选方案：**



  * 自行开发 → **不可行**。
  * GitHub 上的开源最小协议栈 → 功能是否足够尚不确定。



* **关注点：**



  * 最小协议栈不支持 **SMI（标准化主站接口）**。
  * 不确定其是否满足演示需求。



---



## 6️⃣ 硬件选型与考虑



* **Hilscher netX90**



  * 不支持在线切换。
  * 需多颗芯片 → 成本较高。



* **NXP 半导体**



  * 与 Hilscher 存在同样限制。



* **基于 MCU 的 CoM（例如 emtrion STM32 模块）**



  * 仍需实现协议栈。



* **DigiKey 板卡疑问**



  * 原以为可用作 IO-Link 主站。
  * 实际可能仅为 **设备端**，需确认。



* **结论：**



  * 可能需要 **专用 MCU**、首选第三方协议栈，**不支持协议热切换**。



---



## 7️⃣ 核心主题与关键发现



* 架构一致性和工业级系统设计的重要性。
* 主站与设备端的混淆（硬件必须与角色匹配）。
* 协议栈成熟度与 SMI 支持至关重要。
* 避免将 Raspberry Pi 用作 IO-Link 核心。
* 早期明确硬件、协议栈和通信路径非常必要。



---

Here are the **main points** from the conversation:

---

### 1️⃣ System Architecture Clarification

* The new MCU will act as the **IO-Link Master**.
* It serves as the interface between:

  * The localized Motion Master
  * The rest of QNC (similar role as the Flange Interface Board)
* QNC will:

  * Receive **NeuraSync packages**
  * Parse the data
  * Use an **IO-Link Master stack (e.g., RT-Labs)** to generate IO-Link packages
  * Send IO-Link data to the gripper
* **MotionMaster and EtherCAT are NOT being used.**

---

### 2️⃣ Communication Question

* How will the IO-Link Master MCU communicate with the rest of QNC?
* It likely cannot use DDS directly.
* This architectural detail is still under clarification.

---

### 3️⃣ IO-Link Stack Development Status

* Development of the IO-Link stack for the new TCP is still in early stages.
* Currently:

  * Learning IO-Link fundamentals
  * Understanding requirements for different grippers
  * Comparing available stacks
  * Planning protocol structure

---

### 4️⃣ Stack Options & Concerns

* A free minimal IO-Link Master stack exists (GitHub project).
* Concern: It does **not implement SMI (Standardized Master Interface)**.
* Uncertainty whether minimal functionality is sufficient for the demo.

---

### 5️⃣ Hardware Confusion

* A board ordered from DigiKey was assumed to be usable as an IO-Link Master.
* It appears to be **IO-Link Device only**, raising doubts about suitability.
* Need clarification whether it can actually function as a Master.

---

### 6️⃣ Key Architectural Realization

* Raspberry Pi should **NOT** be treated as the IO-Link “core.”
* It should only act as a **command sender**.
* IO-Link protocol conversion should be handled by a **dedicated IO-Link Master hardware module**.
* This was a newly realized architectural correction.

---

### 🔎 Core Themes

* Architecture alignment
* Master vs Device confusion
* Stack maturity and SMI support
* Proper industrial-grade system design
* Avoiding misuse of Raspberry Pi in IO-Link systems

---
Here are the **main points and decisions** from the conversation, structured by topic:

---

## 1️⃣ Overall Scope & Timeline

* Only build a **PoC (Proof of Concept)** for now.
* Do **not aim for full feature release** before end of March.
* AI Kit Gen 1 PM will scale it into a real product after PoCs.
* Only **4 weeks left before Easter** → strict prioritization required.
* Block unrelated topics and reallocate key resources if needed.
* Stop parallel efforts that distract (e.g., possibly pause Sony collaboration).

---

## 2️⃣ QNC Board – Priority & Role

* QNC is becoming a **major, complex project (comparable to a robotic hand)**.
* It requires a **dedicated team focus**.
* QNC board defines the release of **Maira**.
* It may:

  * Run a **localized motion master**
  * Potentially run an **EtherCAT master**
  * Stream data via **NEURASync over TSN**
  * Be robot-agnostic (usable for both Lara and Maira)
* It will **not communicate with MotionMaster** in the original way.
* The QNC is positioned as a **universal interface/protocol converter**.

⚠️ Concern:

* QNC planning was repeatedly disrupted by other priorities.
* It may compete with:

  * Sony LiDAR work
  * Anand’s bionic hands
  * TCP board development

---

## 3️⃣ Communication Focus for PoC

Focus narrowed to:

* **Modbus RTU**
* **IO-Link**
* Via abstracted **NEURASync IDLs**
* Feature parity with new TCP board.

Avoid:

* On-the-fly protocol switching (too complex and expensive).

Preferred approach:

* Each protocol gets:

  * Dedicated port OR
  * Separated via MUX
* Avoid sideloading stacks dynamically.

---

## 4️⃣ IO-Link Technical Architecture

### Key Findings

* IO-Link Master requires:

  * Dedicated MCU
  * Cannot directly process serial signal from Pi
  * Needs protocol stack loaded at startup
* Architecture idea:

  ```
  CM5 (serial)
      ↓
  MCU (runs IO-Link stack)
      ↓
  IO-Link chip (e.g., MAX)
  ```

### Stack Options

* **RT-Labs stack** → preferred

  * But they don’t provide hardware
* Alternative:

  * Write stack ourselves → considered NOT feasible
* Open-source minimal stack found on GitHub

  * Unclear if sufficient

### Hardware Options Discussed

* **Hilscher netX90**

  * Does not support on-the-fly switching
  * Would require multiple chips → high cost
* **NXP Semiconductors**

  * Same limitation as Hilscher
* MCU-based CoM (e.g., emtrion STM32 module)

  * Would still require stack implementation

Conclusion:

* Likely need:

  * Dedicated MCU for IO-Link Master
  * Third-party stack (preferred)
  * No protocol hot-switching

---

## 5️⃣ Mesco Engineering Question

* **MESCO Engineering** also proposed similar solution.
* Their solution includes:

  * Multiple MCUs + netX90.
* Question raised:

  * Have they already solved the blocker?
  * Should we ask them?

Unclear if they provide a QNC-equivalent solution.

---

## 6️⃣ Resource & Organizational Concerns

Key resource conflicts:

* Mahmoud:

  * Planned 2–3 weeks in March for Sony LiDAR
  * Also needed for engineering work
* Xiyao:

  * Might need to be pulled back
* Shine:

  * Screen
* Localization:

  * Mahmoud

Questions raised:

* Do we prioritize:

  * QNC over Sony?
  * Comm board redesign over bionic hands?
* Who maintains the new software stack after hackathon?
* Will SW department take ownership?

Decision:

* The new stack must be maintained internally.

---

## 7️⃣ Sony Collaboration

* Sony working fast on LiDAR.
* Sensor arrives March 6 for testing.
* Another French company involved for full sensor solution.
* Sony + QNC board integration doesn’t make sense.
* Potential resource conflict (Mahmoud).

---

## 8️⃣ TCP Board & Neura Flange Interface

* New TCP board (by Konstantin) is intended to replace aging TCP board.
* Neura Flange Interface:

  * Converts EtherCAT master ↔ gripper
  * Already supports Modbus RS485
  * IO-Link support now being added.
* Potential overlap with QNC regarding IO-Link.

---

## 9️⃣ Software Stack & Hackathon

* Stop all stack development for hackathon.
* After hackathon → rebrand software.
* No longer run USB cables through robots.
* AI department was main USB user.
* Open question:

  * Who maintains the stack after hackathon?

---

# 🔎 Core Strategic Tensions

1. **QNC vs Sony LiDAR**
2. **QNC vs Comm Board Redesign**
3. **QNC vs Bionic Hands**
4. Buy IO-Link stack vs build internally
5. Prototype fast vs over-engineer architecture

---

# 🎯 Likely Immediate Action Points

* Lock scope to:

  * Modbus RTU + IO-Link only
* Decide:

  * Buy IO-Link Master stack?
  * Which MCU?
* Clarify:

  * QNC priority vs Sony
  * Who owns stack long-term
* Evaluate:

  * Whether Mesco already solved this problem
* Freeze non-critical parallel efforts

---

If helpful, I can also convert this into:

* A decision matrix
* A resource conflict map
* A simplified technical architecture diagram
* Or a prioritized execution roadmap for the next 4 weeks

