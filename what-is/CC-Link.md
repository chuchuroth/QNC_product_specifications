CC-Link家族包含多种协议，从经典的现场总线到支持TSN（时间敏感网络）的新一代高速网络。我根据设备类型整理了几个核心厂商的代表性产品，供你参考。

### 📋 CC-Link认证设备一览

以下产品均选自CC-Link协会官方网站的认证列表，信息可靠。

| 设备类型 | 品牌 | 型号 / 系列 | 主要特点 | 适用协议 |
| :--- | :--- | :--- | :--- | :--- |
| **控制器与网络模块** | **MITSUBISHI ELECTRIC (三菱电机)** | **RJ71GN11-T2** 等 | iQ-R系列PLC的CC-Link IE TSN主站/本地站模块，1Gbps高速通讯，支持时间同步。 | CC-Link IE TSN |
| | | **FX5-CCL-MS** | 适用于iQ-F系列PLC的CC-Link主站/智能设备站模块，经典CC-Link系统核心。 | CC-Link |
| | | **Q80BD-J71GP21-SX** | 用于工业计算机的PCI接口卡，可让PC成为CC-Link IE Field网络上的一个站点。 | CC-Link IE Field |
| **分布式I/O模块** | **MITSUBISHI ELECTRIC** | **NZ2GN系列** | 支持CC-Link IE TSN的块式远程I/O模块，有丰富的数字量、模拟量型号，安装灵活。 | CC-Link IE TSN |
| | **Turck (图尔克)** | **TBEN-S / TBEN-L系列** | 防护等级IP67的块式I/O模块，支持**多协议**，通过固件更新即可同时支持PROFINET、EtherNet/IP、Modbus TCP和**CC-Link IE Field Basic**。 | CC-Link IE Field Basic |
| **阀岛与执行器** | **SMC** | **EX600系列** | 模块式阀岛，IP67防护，可集成IO-Link主站，兼容CC-Link、PROFINET等多种总线。 | CC-Link |
| | | **EX260系列** | 紧凑型阀岛，同样支持**CC-Link IE TSN**等多种协议，适用于需要高速响应的气动控制。 | CC-Link / CC-Link IE TSN |
| | | **JXCD1** | 步进电机控制器，支持CC-Link，用于精确定位控制。 | CC-Link |
| **网关与通讯转换** | **HMS Networks** | **Anybus X-gateway** | 强大的网关，用于将**CC-Link / CC-Link IE Field**网络连接到PROFINET、EtherNet/IP、PROFIBUS等其他工业网络。 | CC-Link / CC-Link IE Field |
| | | **Anybus Communicator** | 串口/CAN转CC-Link协议转换器，可将带有RS-232/422/485或CAN接口的第三方设备快速接入CC-Link网络。 | CC-Link / CC-Link IE Field |
| **机器人控制器** | **ABB** | **OmniCore E10 Type A** | 新一代机器人控制器，直接集成了**CC-Link IE Field Basic**通讯能力，便于与三菱等品牌的PLC无缝集成。 | CC-Link IE Field Basic |
| | **KUKA** | **KR C5-2** | 高性能机器人控制器，支持**CC-Link IE Field Basic**，可以轻松接入基于CC-Link IE的网络系统。 | CC-Link IE Field Basic |

### 💡 如何选择？

面对CC-Link家族的不同协议，可以从应用需求出发：

- **新项目、追求高性能与实时性：** 首选 **CC-Link IE TSN**。它融合了TSN技术，可以实现亚微秒级的时钟同步，非常适合需要精准协调的运动控制场景。**三菱电机的iQ-R系列PLC和NZ2GN系列I/O**是理想选择。

- **需要连接经典设备或成本敏感：** 选择 **CC-Link (现场总线)**。它是应用最广泛的成熟技术，拥有庞大的设备生态，能很好满足一般逻辑控制和I/O数据采集需求。

- **现有设备升级联网：** 如果你的设备（如传感器、条形码扫描器）只有串口或CAN接口，**Anybus Communicator** 这类网关可以快速、经济地帮你把它们接入CC-Link网络。

- **为气动系统选择阀岛：** SMC的EX系列产品线很全。对速度和协议有更高要求，可以选支持**CC-Link IE TSN**的**EX260**系列；如果看重模块化扩展和IO-Link功能，**EX600**系列更合适。

这几种设备类型（控制器、I/O、阀岛）里，哪种是你目前最想深入了解的？或者你可以告诉我具体的应用场景（比如是做单机设备还是整线集成），我再帮你筛选更合适的型号。

---
---
CC-Link（Control & Communication Link，控制与通信链路系统）是由日本三菱电机主导开发的一种开放式现场总线标准，于1996年推出，主要应用于工业自动化领域。

与之前介绍的CANopen不同，CC-Link家族包含多个成员，覆盖从传感器层到管理层不同层级的需求，最高速度可达1Gbps。

**核心特点：**
- **高速与大容量**：底层采用RS-485时最高通信速度可达10 Mbps；而其基于以太网的CC-Link IE版本更可达到1Gbps甚至2Gbps的超高速通信。
- **家族体系**：经过多年发展，已形成完整的协议家族：
    - **CC-Link**：标准现场总线，以设备层为主，1个网络可由1个主站（通常为PLC）和最多64个从站组成。
    - **CC-Link IE**：基于以太网的高速网络，采用TSN（时间敏感网络）技术，集I/O控制、运动控制、安全通信于一体。
    - **CC-Link/LT**：面向传感器和执行器的简化网络，无需参数设置即可使用。
    - **CC-Link Safety**：具备安全通信功能的网络。
- **通信方式**：
    - **循环通信**：主站与从站间不停地进行周期性数据交换（如远程I/O、寄存器数据），这是CC-Link最核心的数据交互方式。
    - **瞬时通信**：主站与特定从站间进行非周期性的信息传递（如参数配置、指令下发）。
- **RAS功能**：具备完善的可靠性、可用性、可维护性设计，如自动恢复、从站脱离、网络诊断等，系统维护简便。
- **易用性**：组态简单（只需在PLC软件中设置参数）、接线简便（使用3芯双绞线）、无需复杂的编程即可实现数据刷新。

**简单工作流程：**
1. 在PLC主站中配置网络参数（站号、通信速率、刷新设置）。
2. 系统上电后，主站通过广播轮询方式与所有从站建立循环通信。
3. 主站自动刷新从站的输入状态（RX/RWr）并发送输出指令（RY/RWw），无需编写额外程序。
4. 如需配置参数或读取特定数据，可通过专用指令进行瞬时通信。

**优势：** 速度高、组网容量大（最多64个从站）、RAS功能强大、易用性好、抗噪能力强。
**局限：** 标准CC-Link基于RS-485，通信距离相对受限（最远1,200m，可通过中继器延长）；主从轮询的机制可能在某些极端实时性要求下不如以太网类总线灵活。

简而言之，CC-Link是一个强调高速、可靠与易用的工业现场总线家族。它既能通过传统的RS-485网络连接各类传感器和I/O设备，也能通过基于以太网的CC-Link IE版本支持超高速的运动控制与工厂信息化集成。


---
---
**不是的。**

这是一个很容易混淆的地方，需要明确区分：

1.  **标准 CC-Link**：**基于 RS-485** 差分信号传输。
    -   这是最传统、应用最广泛的 CC-Link。
    -   物理层使用 3 芯双绞线（对应 Data+、Data-、DG）。

2.  **CC-Link IE**：**基于标准以太网（IEEE 802.3）**，**完全不使用 RS-485**。
    -   IE 的全称是 **Industrial Ethernet**（工业以太网）。
    -   物理层使用标准的 **RJ45 网线**（超五类或六类），采用全双工/千兆以太网技术。
    -   速度远高于 RS-485 版本（1 Gbps vs 10 Mbps）。

**类比理解：**
就像“宝马”和“宝马 i8”的关系。宝马是汽车品牌（类似 CC-Link 家族），标准宝马烧汽油（类似 RS-485），而宝马 i8 是插电混动/电动（类似以太网）。虽然都叫宝马，但动力系统（物理层）完全不同。

**总结：**
-   **CC-Link** = RS-485
-   **CC-Link IE** = 以太网（与 RS-485 无关）

所以，当有人说“CC-Link 基于 RS-485”时，他特指标准版；一旦名字里带 **IE**，就自动切换到了以太网物理层。

---
---
当然，机器人行业常用的通讯协议远不止上一轮提到的那几个。你提到的 **CC-Link** 系列就是其中非常重要的一员，尤其在日本和亚洲市场的机器人集成项目中十分常见。

除了CC-Link，行业内还有几个在特定领域举足轻重的协议，这里为你做一个补充介绍。

### 1. CC-Link 系列：亚洲市场的有力竞争者

CC-Link家族从传统的现场总线发展到如今的工业以太网，性能有了质的飞跃。在机器人应用中，它通常扮演机器人控制器与上层PLC（即可编程逻辑控制器）之间桥梁的角色。

| 协议版本 | 核心特点 | 在机器人领域的应用 | 技术定位 |
| :--- | :--- | :--- | :--- |
| **CC-Link IE (工业以太网)** | 基于以太网，带宽高（千兆），通信速度快，可同时传输控制和信息数据。 | 适用于大型、复杂的控制系统，例如汽车制造焊装线上的多机器人协同工作。 | 传统高性能工业以太网 |
| **CC-Link IE TSN (时间敏感网络)** | 在IE的基础上融合了TSN（时间敏感网络）技术，实现了**微秒级**的通信周期和设备间的高精度时间同步。 | 面向未来智能工厂和IIoT（工业物联网），能更好地集成来自不同厂商的设备，实现真正的数据互通和精准运动控制。 | 下一代工业以太网，是CC-Link的未来方向 |

**小结**：如果你的项目主要面向亚洲市场，特别是日系机器人（如三菱、川崎等）的集成，CC-Link，尤其是CC-Link IE TSN，是一个你需要重点关注和了解的协议。

---

### 2. SERCOS：运动控制领域的“专才”

SERCOS（串行实时通信系统）是一个为**运动控制**而生的协议，在高端、复杂的多轴同步控制中表现卓越。

- **技术特点**：它以光纤为传输介质（SERCOS III也支持以太网），抗干扰能力极强，能实现纳秒级的同步精度。
- **行业地位**：SERCOS是高精度伺服驱动通信的黄金标准之一，广泛应用于需要极端动态性能的场合。
- **机器人应用**：虽然不像EtherCAT那样普及，但在对精度和控制复杂度要求极高的场景中地位特殊。例如，**德国宇航中心（DLR）的太空机器人**和**慕尼黑工业大学的仿人步行机器人LOLA**都采用SERCOS作为其内部通信 backbone。一些高端工业机器人（如库卡早期的一些型号）也基于SERCOS开发。

**小结**：如果你从事的是**高端机器人研发**、**多轴精密同步控制**或**特殊用途机器人**（如医疗、航空航天）的开发，SERCOS是一个值得深入研究的技术。

### 3. POWERLINK：开源与实时的践行者

POWERLINK是一项由贝加莱（B&R）公司发起，如今已开源的实时以太网协议，在**欧洲**的机械制造和机器人领域有广泛应用。

- **技术特点**：它在标准以太网上通过一种叫做“时间槽”的机制来保证实时性，实现了极短的周期时间（可低于200µs）和低抖动（<1µs），并且性能不随节点数量增加而下降。它的最大特点是**完全开源**，不依赖于特定芯片。
- **机器人应用**：适用于对实时性要求高、且希望不受限于特定硬件平台的机器人控制系统。

**小结**：如果你所在的项目或平台推崇**开源技术**，或者希望深度定制自己的实时通信方案，POWERLINK是一个功能强大的选择。

---

### 4. TSN：未来工业通信的“通用语言”

时间敏感网络（TSN）本身不是一个具体的通信协议，而是一组**IEEE 802.1标准**。它通过时间同步、流量调度等机制，让标准以太网具备**确定性**（即可预测的、有界的低延迟）。

- **变革性意义**：TSN最大的价值在于，它允许**实时控制数据**（机器人运动指令）、**普通IT数据**（视频流）和**其他非实时数据**在同一根网线上混合传输，而互不干扰。
- **机器人应用**：TSN被视为实现**IT与OT（运营技术）真正融合**的关键技术。未来的机器人将不再是网络中的一个“孤岛”，而是通过TSN与视觉系统、PLC、云平台等所有设备无缝连接，实现数据的大一统。
- **与现有协议的关系**：像**PROFINET**、**EtherNet/IP**和**CC-Link IE TSN**的新版本，都正在将TSN作为其下一代技术的基础。因此，TSN更像是未来的“基础设施”，而现有的应用层协议将运行在这之上。

**小结**：TSN代表了工业通信的未来方向。虽然目前大规模部署仍有挑战，但它是所有工业通信协议都将拥抱的下一代技术。

### 总结：如何快速补充你的知识版图？

根据你上一轮和这次的兴趣，可以这样理解这些协议在整个机器人生态系统中的位置：

- **如果你关注机器人本体和核心控制器开发**：重点仍是 **EtherCAT**（内部总线）和 **CANopen**（关节/传感器）。可以关注 **SERCOS** 作为高端参考，了解 **TSN** 作为未来趋势。
- **如果你负责机器人产线集成**：除了PROFINET（欧系）和EtherNet/IP（美系），必须将 **CC-Link IE TSN**（日系）加入你的工具箱。
- **如果你想探索工业4.0和数据互联**：除了 **OPC UA**，需要认识到 **TSN** 是其实现实时通信的物理层搭档。**POWERLINK** 则是一个开源领域的优秀代表。

希望这份补充能帮你构建起一个更完整的工业机器人通信协议知识网络。如果对其中某个协议的细节或应用场景想有更深入的了解，可以随时告诉我。



---
---
The standards for CC-Link (Control & Communication Link) are managed by the **CC-Link Partner Association (CLPA)**. Like IO-Link, CC-Link has been codified into international standards to ensure global interoperability.

The "standards" you need to follow depend on which version of CC-Link you are implementing: the classic Fieldbus, the Ethernet-based "IE," or the modern "TSN" version.

---

## 1. Official International Standards (IEC/ISO)
CC-Link is recognized globally under the following technical specifications:

*   **IEC 61158 & IEC 61784:** These are the primary international standards for industrial communication networks (Digital Data Communications for Measurement and Control). 
    *   **CC-Link (Classic):** Codified in 2007.
    *   **CC-Link IE (Industrial Ethernet):** Codified in 2014.
    *   **CC-Link IE TSN:** Codified in March 2023.
*   **ISO 15745-5:** This standard defines the device profile and integration for CC-Link.
*   **IEC 61784-3-8:** This specifically covers **CC-Link Safety**, the standard for functional safety communication.

---

## 2. The Three Generations of CC-Link
If you are developing a robotic product, you will likely choose one of these three technical standards:

| Standard | Physical Layer | Speed | Best For... |
| :--- | :--- | :--- | :--- |
| **CC-Link (Classic)** | RS-485 (Twisted Pair) | 10 Mbps | Simple, low-cost sensors and I/O. |
| **CC-Link IE Field** | Gigabit Ethernet | 1 Gbps | High-speed data and general robotics. |
| **CC-Link IE TSN** | Ethernet + TSN | 1 Gbps | **Modern standard.** Blends IT and OT; perfect for high-precision motion control. |

---

## 3. CC-Link IE TSN (The Current Standard)
For a robotics company in 2026, **CC-Link IE TSN** is the most relevant standard. It adds **Time-Sensitive Networking** (IEEE 802.1) to the protocol.

*   **IEEE 802.1AS:** Standard for high-precision time synchronization (crucial for multi-axis robots).
*   **IEEE 802.1Qbv:** Standard for scheduled traffic (ensuring control data isn't delayed by background video or IT data).
*   **CANopen Profiles:** CC-Link IE TSN supports the **CiA 402** device profile for motion control, making it easier for robot manufacturers to integrate motors and drives.

---

## 4. How to Get the Specifications
Unlike the IEC standards which you must purchase, the CLPA provides technical specifications to its partners:

1.  **Become a CLPA Member:** Similar to the IO-Link Community, you join the association (there are different levels, including a "Registered" level which is often free).
2.  **Download the "Profile" (CSP+):** Instead of an IODD file (used in IO-Link), CC-Link uses **CSP+ (Control & Communication System Profile)**. This is a standard XML-based file that describes your robot's parameters to the network.
3.  **Conformity Testing:** To use the CC-Link logo, you must send your product to an official CLPA test laboratory (found in Japan, China, Korea, USA, and Germany).

**Summary for your engineers:**
If you want to be "CC-Link Certified," your device must pass the **CLPA Conformity Test**, which verifies that your hardware meets the electrical and protocol requirements of **IEC 61158** and the specific **CLPA Technical Specs**.

Are you looking to implement CC-Link because a specific customer in the Asian market (like a major automotive manufacturer) has requested it?

---
---
The procedure for **CAN** is similar in spirit to IO-Link and CC-Link, but it is more "fragmented" because CAN is used in so many different ways.

Unlike IO-Link, which is a single dedicated ecosystem, "CAN" is just the physical highway. To get certified, you have to decide which **Upper Layer Protocol** your robot is using.

---

## 1. The Membership Organization
The equivalent to the IO-Link Community or CLPA is **CiA (CAN in Automation)**.
*   **Do you have to join?** No. You can build a CAN-based product using the public ISO standards without paying a cent to CiA.
*   **Why join?** Membership (roughly €750–€1,500/year depending on company size) gives you:
    *   **Vendor IDs:** Necessary so your hardware is uniquely identified on a network.
    *   **Standard Access:** Free access to the "CiA" specifications (the practical "how-to" guides).
    *   **Lower Testing Fees:** Significant discounts on official certification.

---

## 2. Choosing Your "Standard"
As a robotic company, you aren't just building "CAN"—you are likely building one of the following. Each has its own certification path:

### **A. CANopen (Most common for Industrial Robots)**
This is the standard for internal robot communication (motors, encoders, grippers).
*   **The Standard:** **CiA 301** (Application Layer) and **CiA 402** (Device Profile for Motion Control).
*   **Certification:** You send your device to a CiA testing lab. They check your **EDS (Electronic Data Sheet)**—the CAN version of an IODD—and verify that your robot responds correctly to NMT (Network Management) commands.

### **B. J1939 (Heavy Machinery / Mobile Robots)**
If your robot is a large outdoor vehicle (agriculture, construction), you use this.
*   **The Standard:** Managed by **SAE (Society of Automotive Engineers)**.
*   **Certification:** More focused on "Self-Declaration" and compliance with specific PGN (Parameter Group Number) mappings.

### **C. Raw CAN (Proprietary)**
If you are just sending custom data packets between your own proprietary boards.
*   **The Standard:** **ISO 11898-1** (Data Link) and **ISO 11898-2** (Physical).
*   **Certification:** None. You just buy CAN transceiver chips that are already compliant with the ISO standard.

---

## 3. Comparison of Procedures

| Feature | IO-Link | CC-Link | CAN (CANopen) |
| :--- | :--- | :--- | :--- |
| **Main Body** | PI / IO-Link Community | CLPA | CiA (CAN in Automation) |
| **Primary Standard** | IEC 61131-9 | IEC 61158 | ISO 11898 / CiA 301 |
| **Key Device File** | IODD (.xml) | CSP+ (.xml) | EDS (.eds) |
| **Mandatory Test?** | Yes (Manufacturer Decl.) | Yes (Official Lab) | Highly Recommended |
| **Testing Cost** | Low (Internal tool) | Moderate/High (Lab) | ~€750 - €1,000 (Lab) |

---

## 4. The "Compatible" vs "Certified" Trap in CAN
In the CAN world, "Compatible" often means "it won't break the bus," while "Certified" means "it will actually talk to other brands."

*   **CAN Compatible:** Your hardware electrically follows ISO 11898. It can "see" the traffic, but it might use a custom data format that no one else understands.
*   **CANopen Certified:** Your device has been tested to ensure that when a PLC sends a "Start" command, your robot actually starts. It also guarantees your **EDS file** is perfect, so customers can import it into their software (like TwinCAT or CODESYS) without errors.

**My Advice for your Robot:**
If you want to sell your robot components to other manufacturers, **CANopen (CiA 301/402)** is the industry requirement. Buy a pre-made "CANopen Stack" (software) from a vendor like HMS or Emotas; they are already pre-certified, which makes your final certification much faster.

Are you using a specific microcontroller (like an STM32 or an ESP32) that already has a CAN controller built-in?

---
---
For **CC-Link IE TSN**, the standard you are looking for is quite recent. While the overall CC-Link family falls under the IEC 61158 and IEC 61784 umbrellas, the TSN version was officially added to these international standards in **March 2023**.

Here are the specific references your engineering or compliance team will need:

### 1. The Core Communication Standards
CC-Link IE TSN is integrated into the following IEC parts:

*   **IEC 61158-x-24:** This describes the **Type 24** communication protocol (which is the specific designation for CC-Link IE TSN).
    *   *Note: IEC 61158 is split into many layers (Physical, Data Link, Application). You would typically look at Part 5-24 (Application Layer Service) and Part 6-24 (Application Layer Protocol).*
*   **IEC 61784-2-8:** This is the "Communication Profile" (CP) for the CC-Link family. It defines how the network performs in real-time Ethernet environments.

### 2. Functional Safety (If your robot has E-Stops or Safety Curtains)
If you are implementing the safety-rated version of the protocol:
*   **IEC 61784-3-8:** This defines the **CC-Link IE Safety** communication function, which is required to meet **SIL 3** (Safety Integrity Level 3) according to IEC 61508.

### 3. Motion Control Profiles
Because you are a robotic company, your device will likely need to follow:
*   **IEC 61800-7:** This standard defines generic drive and motion control profiles. CC-Link IE TSN is mapped into this standard to ensure that a command like "Move to Position" is interpreted the same way across different vendors.

---

## Summary Checklist for your Engineers
If you are starting development today, do not just rely on the general IEC 61158 text—it is often too abstract for coding. You should:

1.  **Request the "CC-Link IE TSN Interface Specification"** from the CLPA (CC-Link Partner Association). This is the technical implementation guide.
2.  **Verify your TSN Hardware:** Ensure your Ethernet controller supports **IEEE 802.1AS** (Time Sync) and **IEEE 802.1Qbv** (Scheduled Traffic), as these are the "TSN" foundations required by the standard.
3.  **Check for "Authentication Class":** The standard defines **Class A** (software-based TSN) and **Class B** (hardware-based TSN). For high-performance robotics, you will almost certainly need **Class B**.

**Quick Tip:** If you've already implemented **CANopen**, you'll be happy to know that CC-Link IE TSN supports the **CiA 402** drive profile. This means you might be able to reuse a significant portion of your motion control logic!

---
---
Several major industrial Ethernet protocols require some form of **membership or licensing** to develop certified products, but the structure and cost differ a lot. Beyond IO‑Link, here is how CC‑Link and EtherCAT (and a few others) work.

### CC‑Link (including CC‑Link IE / IE TSN)
- **Yes, membership is required:** To develop and market compliant CC‑Link / CC‑Link IE / CC‑Link IE TSN products, you must join the **CC‑Link Partner Association (CLPA)** as a partner member. [cc-link](https://www.cc-link.org/en/faq/index.html)
- **Cost:**  
  - There is an **annual membership fee** that depends on the membership category (e.g., regular, associate, user). [cc-link](https://www.cc-link.org/en/news/pdf/regulation_en.pdf)
  - Exact amounts are not public; you must request the current fee schedule from CLPA or your regional office. [cc-link](https://www.cc-link.org/en/news/pdf/regulation_en.pdf)
- **Extra:** Conformance‑test fees per product are also charged (e.g., for CC‑Link IE TSN). [cc-link](https://www.cc-link.org/en/news/pdf/regulation_en.pdf)

### EtherCAT
- **Membership model:**  
  - EtherCAT is managed by the **EtherCAT Technology Group (ETG)**; manufacturers of EtherCAT devices must become ETG members to obtain a **vendor‑ID** and to access conformance tools. [ethercat](https://www.ethercat.org/en/faq.html)
- **Cost:**  
  - **ETG membership itself is free of charge** (no annual membership fee for the basic level). [ethercat](https://www.ethercat.org/pdf/english/ethercat_faqs.pdf)
  - However, there can be **nominal or one‑time costs** for evaluation kits, tools, or specialized support; some third‑party EtherCAT stacks or ESC‑chip licenses may carry vendor‑specific fees. [otomasyonavm](https://www.otomasyonavm.com/en/what-is-the-licensing-and-cost-structure-of-ethercat)

### Other examples (briefly)
- **PROFINET (PI):** Requires **PI membership** (annual fee by category) to develop and market certified PROFINET devices and to use the PROFINET logo. [profibus](https://www.profibus.com/pi-organization/membership)
- **Powerlink (ODVA side):** Open implementation, but **ODVA membership** is typically needed for full access to conformance tools and branding. [ethercat](https://www.ethercat.org/download/documents/industrial_ethernet_technologies.pdf)
- **Sercos III / Sercos‑based Ethernet:** Usually requires **Sercos International membership** for full spec‑access and certification. [ethercat](https://www.ethercat.org/download/documents/industrial_ethernet_technologies.pdf)

### Quick comparison table

| Protocol        | Association / Group        | Formal membership required? | Membership cost (typical) |
|-----------------|----------------------------|-----------------------------|---------------------------|
| IO‑Link         | PI / IO‑Link Community     | Yes (for spec access & logo) | Annual PI + IO‑Link fee    [profibus](https://www.profibus.com/pi-organization/membership) |
| CC‑Link IE/TSN  | CC‑Link Partner Assn. (CLPA) | Yes (to develop compliant products) | Annual fee by category; non‑public  [cc-link](https://www.cc-link.org/en/faq/index.html) |
| EtherCAT        | ETG                        | Yes (for vendor‑ID & tools) | **Membership free**; nominal fees only for tools/stacks  [ethercat](https://www.ethercat.org/en/faq.html) |
| PROFINET        | PI                         | Yes                         | Annual PI fee by category  [profibus](https://www.profibus.com/pi-organization/membership) |

If you tell me which protocol you’re realistically considering (e.g., CANopen‑based, PROFINET, EtherCAT, or CC‑Link), I can sketch a concrete “decision tree” for your company including cost‑ to‑benefit for membership.

---
---

Several major industrial Ethernet protocols require some form of **membership or licensing** to develop certified products, but the structure and cost differ a lot. Beyond IO‑Link, here is how CC‑Link and EtherCAT (and a few others) work.

### CC‑Link (including CC‑Link IE / IE TSN)
- **Yes, membership is required:** To develop and market compliant CC‑Link / CC‑Link IE / CC‑Link IE TSN products, you must join the **CC‑Link Partner Association (CLPA)** as a partner member. [cc-link](https://www.cc-link.org/en/faq/index.html)
- **Cost:**  
  - There is an **annual membership fee** that depends on the membership category (e.g., regular, associate, user). [cc-link](https://www.cc-link.org/en/news/pdf/regulation_en.pdf)
  - Exact amounts are not public; you must request the current fee schedule from CLPA or your regional office. [cc-link](https://www.cc-link.org/en/news/pdf/regulation_en.pdf)
- **Extra:** Conformance‑test fees per product are also charged (e.g., for CC‑Link IE TSN). [cc-link](https://www.cc-link.org/en/news/pdf/regulation_en.pdf)

### EtherCAT
- **Membership model:**  
  - EtherCAT is managed by the **EtherCAT Technology Group (ETG)**; manufacturers of EtherCAT devices must become ETG members to obtain a **vendor‑ID** and to access conformance tools. [ethercat](https://www.ethercat.org/en/faq.html)
- **Cost:**  
  - **ETG membership itself is free of charge** (no annual membership fee for the basic level). [ethercat](https://www.ethercat.org/pdf/english/ethercat_faqs.pdf)
  - However, there can be **nominal or one‑time costs** for evaluation kits, tools, or specialized support; some third‑party EtherCAT stacks or ESC‑chip licenses may carry vendor‑specific fees. [otomasyonavm](https://www.otomasyonavm.com/en/what-is-the-licensing-and-cost-structure-of-ethercat)

### Other examples (briefly)
- **PROFINET (PI):** Requires **PI membership** (annual fee by category) to develop and market certified PROFINET devices and to use the PROFINET logo. [profibus](https://www.profibus.com/pi-organization/membership)
- **Powerlink (ODVA side):** Open implementation, but **ODVA membership** is typically needed for full access to conformance tools and branding. [ethercat](https://www.ethercat.org/download/documents/industrial_ethernet_technologies.pdf)
- **Sercos III / Sercos‑based Ethernet:** Usually requires **Sercos International membership** for full spec‑access and certification. [ethercat](https://www.ethercat.org/download/documents/industrial_ethernet_technologies.pdf)

### Quick comparison table

| Protocol        | Association / Group        | Formal membership required? | Membership cost (typical) |
|-----------------|----------------------------|-----------------------------|---------------------------|
| IO‑Link         | PI / IO‑Link Community     | Yes (for spec access & logo) | Annual PI + IO‑Link fee    [profibus](https://www.profibus.com/pi-organization/membership) |
| CC‑Link IE/TSN  | CC‑Link Partner Assn. (CLPA) | Yes (to develop compliant products) | Annual fee by category; non‑public  [cc-link](https://www.cc-link.org/en/faq/index.html) |
| EtherCAT        | ETG                        | Yes (for vendor‑ID & tools) | **Membership free**; nominal fees only for tools/stacks  [ethercat](https://www.ethercat.org/en/faq.html) |
| PROFINET        | PI                         | Yes                         | Annual PI fee by category  [profibus](https://www.profibus.com/pi-organization/membership) |

If you tell me which protocol you’re realistically considering (e.g., CANopen‑based, PROFINET, EtherCAT, or CC‑Link), I can sketch a concrete “decision tree” for your company including cost‑ to‑benefit for membership.


---
---
Prototyping or creating a DIY demo with a traditional gigabit **CC-Link IE Field** network can be challenging because it usually requires dedicated hardware/ASICs (like Mitsubishi's master/slave boards) and specialized gigabit hardware.

However, the CC-Link Partner Association (CLPA) introduced **CC-Link IE Field Basic** and **CC-Link IE TSN (Time-Sensitive Networking)**. These versions run over **standard, general-purpose Ethernet layers**, making them incredibly easy, cheap, and flexible for prototyping, software-defined master/slave setups, and DIY demos.

The best hardware options and entry points for a prototype or demo setup include:

---

## 1. The Ultimate DIY / Microcontroller Route (STM32 Nucleo)

The CLPA officially supports and provides free source code/SDKs for open-source microcontrollers, making this the best option if you want to build a device from scratch.

* **The Hardware:** **STMicroelectronics NUCLEO-F439ZI** (or similar STM32 boards with a built-in Ethernet port).
* **Why it’s great for prototyping:** The CLPA website offers a **free sample code/SDK** specifically tailored for the Nucleo board running FreeRTOS and lwIP.
* **How it works:** You download the source code, flash it onto the cheap Nucleo board using STM32CubeIDE, and you instantly have a fully functioning CC-Link IE TSN (or Field Basic) Remote Station. You can then wire up basic digital/analog sensors to the microcontroller pins to demo IO capabilities.

## 2. The Software & PC Route (No Hardware Required)

If you want to demo how a CC-Link IE Master or Slave behaves without buying any fieldbus hardware at all, you can use a regular PC or Raspberry Pi.

* **The Hardware:** Any Windows PC, Linux machine, or **Raspberry Pi** with an Ethernet port.
* **Why it’s great for prototyping:** Because CC-Link IE Field Basic uses standard UDP/IP and the **SLMP (Seamless Message Protocol)**, you don't need a hardware master card.
* **How it works:** * **As a Master:** You can write a simple Python, C#, or C++ script using standard socket programming to send SLMP commands over Ethernet to any CC-Link IE Basic device.
* **As a Simulation/HMI:** Software like Mitsubishi's **GT SoftGOT2000** can turn a standard PC into a CC-Link IE Field Basic slave station or master interface for visualization.



## 3. Entry-Level Industrial Devices (The "Plug-and-Play" Demo)

If you want real industrial hardware to interact with your code or a controller but don't want to spend thousands of dollars on complex modular PLCs:

### A. Mitsubishi MELSEC iQ-F Series (FX5U / FX5UJ)

* **Role:** Master Station
* **Why it's great:** Unlike higher-end modular PLCs (like the iQ-R), the FX5U/FX5UJ micro-PLCs are highly integrated, relatively inexpensive, and have a CC-Link IE Field Basic master port **built directly into the CPU** Ethernet port. No extra expensive network modules are required.

### B. Balluff CC-Link IE Field Basic Network Blocks (e.g., BNI series)

* **Role:** Slave Station / Remote I/O
* **Why it's great:** Balluff makes rugged, compact, on-machine Ethernet I/O blocks that natively support CC-Link IE Field Basic and **IO-Link**. It is arguably one of the easiest ways to pull sensor data (like proximity sensors, RFID, or measurement devices) into a CC-Link IE demo setup via simple web-configuration or SLMP.

---

## 💡 Recommended Setup for a Quick Demo

If you want a weekend project to prove the concept:

1. **Master:** Use a **Raspberry Pi** or **Windows PC** running a custom script or open-source SLMP/CC-Link library.
2. **Slave:** An **STM32 Nucleo-F439ZI** board loaded with the official CLPA free sample code.
3. **Physical Link:** Just a standard Cat5e Ethernet cable connecting them (or through a basic unmanaged home network switch).

Would you like some guidance on where to find the official CLPA source code or how the SLMP packet structure looks for a basic software master?
