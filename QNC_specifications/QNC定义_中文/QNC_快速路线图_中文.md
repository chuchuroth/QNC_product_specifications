# QNC 工业协议网关 —— 快速通道精简路线图

## 1. 关键洞察摘要

以下洞察来自对商业价值与架构分析的转译,是 QNC 快速通道战略中价值最高的部分。每一条洞察要么消除了重大的路线图风险,要么增加了此前缺失的商业维度,要么提供了可立即执行的决策依据。

**商业洞察**

- **协议碎片化是客户真实、持续存在的痛点。** 多协议混合环境 —— 机器人本体采用 EtherCAT、PLC 采用 PROFINET、视觉系统采用 GigE Vision、传感器采用 IO-Link、监控系统要求 OPC UA —— 是当前机器人系统集成商面临的主要集成难题。成本的关键不在硬件本身,而在于集成所耗费的时间。一款能够省去 3-4 个专用转换器、并将试运行时间从两周压缩到三天的网关产品,足以支撑每台 1,500-2,500 美元的定价。
- **最具价值的客户是机器人系统集成商,而非终端工厂。** 系统集成商每个项目可节省数天的调试时间。他们的支付意愿较高(每台 500-2,000 美元),且采购由项目驱动,意味着销售管道可预测性较强。这应作为试点阶段的目标客户群体。
- **OEM 嵌入式客户是走量路径。** 大批量情况下单价降至 50-200 美元,但该细分市场带来规模效应(每年 1,000 台以上)。OEM 合作应从第 13 个月起再行推进,而非在产品成熟之前。
- **该财务模型具有很高的毛利率(>80%),但前期研发投入(NRE)较为集中。** 标准配置(netX 90 + 主 CPU)的硬件物料成本(BOM)为 55-75 美元。以 399 美元的平均售价(中性情景)计算,毛利率超过 85%。中性情景下的盈亏平衡点出现在第 18-20 个月。最大的风险是销量不足,而非开发成本。
- **仅在锁定 2-3 个客户承诺后才启动开发。** 分析建议,在投入全面开发资源之前,先从试点集成商处获得购买意向。这样可以降低最大的单一财务风险敞口:即产品开发完成后销量却低于预期。

**架构洞察**

- **数据总线(发布-订阅式过程映像)是 QNC 正确的内部架构。** 分析比较了三种内部模式:协议转换器(点对点规则)、数据总线(带发布-订阅机制的共享过程映像)以及混合模式。数据总线被推荐作为起始架构,原因在于:新增协议只需编写一个适配器(无需规则爆炸式增长)、该架构原生支持多个以不同更新速率读取的消费者、且该架构与 QNC 的语义归一化模型天然契合。混合模式可以在此基础上逐步叠加。
- **混合模式的三通道路由模式与 QNC 的数据分类直接对应。** 通道 1(实时总线,500 微秒周期):关节位置、扭矩、离散 IO 状态。通道 2(零拷贝高速流):QNC 最小可行产品(MVP)阶段不需要,但适用于未来的视觉/点云集成。通道 3(转换器模式,5 毫秒以上):配置、诊断、北向遥测。这与 QNC 现有的协议/北向拆分方式相匹配。
- **转换层的死区滤波机制可防止不必要的 API 扇出。** 过程映像中的每个数据点应携带可配置的死区阈值;落在该阈值范围内的更新在向北发布前会被抑制。这可以在满足 NFR-002(遥测频率 ≥10 Hz)要求的同时,减少静态或近静态设备状态下的 REST/WebSocket 流量 —— 与此直接相关。
- **必须为关键信号设立异常旁路路径。** 当主实时总线路径失效或数据过期时,关键控制信号(如急停、目标位置)必须回退至直接的转换器模式路径。用 QNC 的术语表述,这对应安全模式降级路径:故障管理器检测到过程映像过期 → 触发回退 → 发布 DEGRADED(降级)状态 → 通过直接适配器调用继续南向操作。

**市场推广洞察**

- **异构机器人生产线集成是商业价值最高的场景。** 用单一 QNC 网关连接 KUKA(EtherCAT)、ABB(PROFINET)、FANUC(CC-Link IE)以及 UR 协作机器人(Modbus TCP/EtherCAT)—— 并向 MES 提供统一的 OPC UA 输出 —— 是核心应用场景。它可以用一台售价 1,500-2,500 美元、试运行仅需三天(而非两周)的设备,取代 3-4 台各售 500 美元的专用网关。
- **IO-Link 智能工具集成是机器人原生的最佳切入点。** 通过 IO-Link 通信的智能夹爪、力矩传感器和摄像头无法直接连接机器人控制器。QNC 作为 IO-Link 主站桥接至 EtherCAT 或 PROFINET,是一个真正新颖的集成点,使 QNC 成为末端工具(EOAT)生态系统中的结构性节点。
- **协议广度、设备兼容深度以及易用性工具是三大持久的竞争优势** —— 而非专利。建议申请实用新型专利(1-2 项),用于市场宣传及获取政府补助资格,但真正的护城河在于积累的设备兼容性、协议认证徽章,以及竞争对手难以复制的面向用户的配置工具。
- **Node-RED 节点库是一项低成本、高可见度的社区投入。** 为 QNC 各协议适配器及 REST/WebSocket API 构建并开源 Node-RED 节点库,能以最小成本建立社区存在感、开发者上手渠道及集成粘性。这应安排在第三阶段,与云端管理平台并行推进。
- **分级产品系列(基础版/标准版/全功能版)在财务上已具备可行性。** BOM 成本分析支持三种 SKU:基础版(约 30 美元 BOM,仅 Modbus + OPC UA)、标准版(约 65 美元 BOM,含 netX 90)、全功能版(约 85 美元 BOM,新增 IO-Link + CAN)。各自面向不同的买家与销量层级。

---

## 2. 执行摘要

### 核心洞察

竞品研究与商业分析共同证实了一个决定性发现:**市场的实时协议问题早已被 Hilscher netX 90 芯片方案解决,市场真正的商业问题在于其之上缺少一个机器人原生的语义与集成层。** HMS Anybus X-Gateway、Hilscher netTAP 以及 Moxa MGate 均采用同一种经过认证的实时协处理器架构。它们都没有提供机器人原生的 API、设备配置文件治理机制,或 NEURA 生态集成能力。

QNC 的捷径在于**构建语义与集成层** —— 这是目前没有任何竞争对手具备的部分 —— 同时**将经认证的实时协议层委托给** netX 90 处理。其结果是:在机器人客户真正关心的维度上,该产品交付的价值超过市场领导者,同时相较全定制实现方案,上市速度更快、开发成本更低。

### 捷径战略一段话总结

从第 1 天起就**采用 Hilscher netX 90** 作为完整的实时协议引擎,配备经认证的 EtherCAT、PROFINET、EtherNet/IP 和 Modbus 固件栈。搭配运行 Linux 的 **TI AM62x** 应用处理器。通过**成熟、开源、经生产验证的 Hilscher cifX 驱动 API** 连接两者。将内部数据建模为**带发布-订阅适配器的共享过程映像**(数据总线架构),并在北向发布前扩展一个死区滤波层。加入 QNC 的差异化层:带语义归一化的 YAML 设备配置文件、双 REST/WebSocket API、原子化配置回滚以及安全模式。在启动开发之前,**先从 2-3 个机器人系统集成商处获取购买意向**,以验证核心应用场景。目标平均售价定在 399-499 美元区间,以实现 85% 以上的毛利率,并在第 18-20 个月实现盈亏平衡。

### 量化影响

| 维度 | 完整路线图 v3.0 | 快速通道 v2.0 | 节省 |
| --- | --- | --- | --- |
| 达成可运行演示的时间 | 约第 5-6 个月 | **约第 2-3 个月** | 约 3 个月 |
| 达成 GA(正式发布)的时间 | 约第 11 个月 | **约第 12 个月** | 相当 |
| PROFINET + EtherCAT 可用时间 | 第 8 个月以后 | **第 6-7 个月** | 约 2 个月 |
| R5F 固件开发 | 11 周以上 | **已消除** | 整条工作线 |
| PRU-ICSS EtherNet/IP 联调 | 8-10 周 | **已消除** | 整条工作线 |
| 协议认证成本 | 15 万-45 万欧元 | **0 欧元** | 完全规避 |
| 399 美元售价下的毛利率 | —— | **约 85%** | 商业清晰度 |
| 盈亏平衡点(中性情景) | 未建模 | **第 18-20 个月** | 明确目标 |

---

## 3. 商业背景与市场机会

### 3.1 核心客户痛点:协议碎片化

工业自动化环境 —— 尤其是机器人生产线 —— 普遍存在严重的多协议碎片化问题。没有任何一种总线一统天下,集成商必须在每个项目中桥接互不兼容的世界。

| 痛点 | 具体影响 |
| --- | --- |
| **协议孤岛** | 机器人本体运行 EtherCAT;PLC 运行 PROFINET;视觉系统运行 GigE Vision;传感器运行 IO-Link;监控系统要求 OPC UA。每种组合都需要单独的转换器或定制代码。 |
| **集成成本高昂** | 系统集成商需购买多台专用网关、编写协议专属映射代码、调试跨厂商兼容性问题 —— 每个项目耗费数天到数周不等。 |
| **数据被锁在总线内** | 设备数据无法统一采集至 MES/ERP 或云端进行分析。每个协议孤岛都需要自己的数据提取工具链。 |

### 3.2 目标客户细分与定价

| 客户类型 | 典型使用场景 | 支付意愿 | 可接受单价 |
| --- | --- | --- | --- |
| **机器人系统集成商** | 将不同品牌机器人连接至生产线 | 高 —— 显著节省试运行时间 | 500-2,000 美元 |
| **设备制造商** | 为自有设备增加多协议通信能力 | 中高 —— 作为可选附加模块 | 100-500 美元 |
| **工厂自动化部门** | 存量产线改造、云端数据集成 | 中等 | 300-1,000 美元 |
| **OEM 嵌入式客户** | 大批量嵌入自有产品 | 高 —— 优先考虑长期单位成本 | 大批量下 50-200 美元 |

**MVP 试点优先级:** 机器人系统集成商。他们代表最高的支付意愿、最快的采购决策(项目驱动型),以及在协议兼容性与配置工具方面最优质的反馈质量。

### 3.3 商业价值最高的场景

**场景一 —— 异构机器人生产线集成(价值最高)**

传统方式下,一条同时包含 KUKA(EtherCAT)、ABB(PROFINET)、FANUC(CC-Link IE)以及 UR 协作机器人(Modbus TCP 或 EtherCAT)的产线,需要 3-4 台不同的专用网关。QNC 用一台设备取代所有网关,向 MES 呈现统一的 OPC UA 数据模型,并从生产控制系统接收统一命令。

商业价值:单台设备定价 1,500-2,500 美元,相比四台专用网关(每台约 500 美元),同时 QNC 将试运行时间从两周压缩到约三天。仅集成人工成本的节省(按典型集成商费率计算)就是主要的价值驱动因素。

**场景二 —— IO-Link 智能工具集成(机器人原生的最佳切入点)**

通过 IO-Link 通信的智能夹爪、力矩传感器与视觉相机无法直接连接机器人控制器,因为后者不支持 IO-Link 主站功能。QNC 可作为最多四个智能工具的 IO-Link 主站,并通过 EtherCAT 或 PROFINET 桥接至机器人控制器。这使 QNC 成为末端工具(EOAT)生态系统中的结构性节点,并为与工具厂商建立自然的合作机会创造条件。

**场景三 —— 机器人云端数据集成(工业物联网增值销售)**

工厂希望从机器人子系统采集振动、温度、电流和扭矩数据用于预测性维护,但机器人控制器往往不暴露这些数据,或对访问加以限制。QNC 通过 OPC UA 或 MQTT 采集这些数据,并转发至 NEURA Cloud 或第三方工业物联网平台。该场景支持订阅收入模式(每设备年度 SaaS 费用),可从第三阶段起激活。

### 3.4 可触达市场

- 全球工业网关市场:2024 年约 27.6 亿美元,预计到 2031 年达 43.7 亿美元(年复合增长率约 15%),其中机器人相关连接约占总量的 20-30% —— 即一个 5.5 亿-8.75 亿美元、且增速与市场持平或更快的可触达细分市场。
- 机器人专属的多协议网关是一个尚未被充分服务、且没有主导性开放标准玩家的细分市场。HMS Anybus X-Gateway 凭借硬件占据主导地位,但缺乏软件/API 层。QNC 面向机器人的原生定位占据了结构性未被挑战的空间。

---

## 4. 研究关键发现

### 4.1 市场结构:两大阵营,一个缺口

工业网关市场清晰地分为两大阵营 —— 而 QNC 的机会正是两者之间的缺口。

| 阵营 | 示例 | 优势 | 缺口 |
| --- | --- | --- | --- |
| **经认证的实时网关** | HMS Anybus、Hilscher netTAP、Moxa MGate | 亚毫秒级确定性现场总线、完整协议一致性 | 无开放 API、无设备配置文件、无机器人语境 |
| **开放式 Linux 工业物联网网关** | Elastel EG324、Advantech ADAM-6717、Red Lion DA30 | 开放操作系统、Docker、MQTT、云集成 | 无实时现场总线、无 EtherCAT/PROFINET、非机器人级别 |

**QNC 的结构性定位:** 唯一将经认证的实时现场总线转换与开放的、机器人原生的语义及 API 层相结合的产品。目前没有任何竞品占据这一位置。

### 4.2 netX 90 是行业的实时核心

Hilscher netX 90 在竞品分析中获得 10/10 的可行性评分 —— 是所有方案中最高的。它已在 NEURA 的评估之中。关键在于:

- 可在专用硬件上同时运行两套经认证的现场总线协议栈(xC0 + xC1)
- 已提供 EtherCAT、PROFINET、EtherNet/IP、CANopen、Modbus RTU、CC-Link IE 的经认证固件
- 提供独立于 Linux 应用负载的亚 0.1 毫秒实时保证
- cifX 驱动 API 是开源的,已在数千个部署中经过生产验证
- 这正是 HMS 用来构建 27.6 亿美元市场地位的同一架构

早期路线图版本中的 R5F 实时固件工作线**完全可以**由 netX 90 授权协议栈替代。

### 4.3 市场数据证实第一阶段协议优先级

HMS Networks 2025 市场份额数据:PROFINET 27%、EtherNet/IP 23%、EtherCAT 17%、Modbus TCP 5%。这三种协议覆盖了目标市场的 67%,且均可通过 netX 90 固件切换支持 —— 彼此之间无需更改硬件。

### 4.4 数据总线架构适合 QNC 的软件层

评估了三种内部软件架构:协议转换器(点对点映射规则)、数据总线(共享过程映像、发布-订阅)以及混合模式。数据总线架构被推荐作为基础架构,原因如下:

- 新增协议只需为共享过程映像编写一个适配器 —— 不会导致规则数量爆炸式增长
- 原生支持多个以不同更新速率并发读取的消费者
- 该架构与 QNC 的语义归一化模型(南向帧 → 过程映像 → 规范遥测)直接对应
- 混合路由层可以在不进行架构重构的情况下逐步添加

**数据总线模式即是 QNC 语义映射器的实现策略。** 共享过程映像是 CSC 遥测结构的内部表示,每个协议适配器充当发布者,北向 REST/WebSocket 服务器充当订阅者。

### 4.5 三种可直接采用的技术模式

**死区滤波**可防止不必要的北向扇出。过程映像中的每个数据点携带一个可配置的变化阈值;只有超过该阈值的变化才会向北转发。这对于在静态设备状态下满足 NFR-002(遥测频率 ≥10 Hz)且不淹没 REST/WebSocket 客户端至关重要。

**关键信号的异常旁路机制**确保:当主过程映像路径过期(在可配置窗口内无更新,例如 10 毫秒)时,关键控制信号将回退至直接的转换器模式路径。用 QNC 的术语表述:故障管理器检测到过程映像过期 → 发布 DEGRADED 生命周期状态 → 为执行关键信号启用直接适配器路径。

**每个过程映像点的质量标记**(BAD(坏) / UNCERTAIN(不确定) / GOOD(好),外加单调递增时间戳)与 CSC 中 `telemetry_quality`(遥测质量)字段的取值(`faulted`(故障) / `stale`(过期) / `valid`(有效))直接对应。在过程映像层实现质量标记,可使 `telemetry_quality` 字段自动具有意义,而无需在语义映射器中增加额外逻辑。

### 4.6 ROS2 集成是真正的差异化优势

目前没有任何网关产品提供原生 ROS2 集成。该集成路径已作为经生产验证的开源方案存在:`ethercat_driver_ros2`(ICube-Robotics)、cifX ROS2 硬件接口(ADLINK/松下案例研究)以及 Intel ECI EtherCAT 转 ROS2 网关。这是一个 1-2 周的集成任务,而非固件开发项目,也是最能凸显 QNC 在 NEURA 机器人生态中差异化优势的单一特性。

### 4.7 协议认证是成本最高的风险 —— netX 90 消除了这一风险

自研协议栈开发及一致性测试的成本为每个协议 5-15 万欧元。以三个目标协议计算,总成本为 15-45 万欧元,且认证周期长达 12-18 个月。netX 90 的授权协议栈已预先通过 PI(PROFINET)、ODVA(EtherNet/IP)和 ETG(EtherCAT)认证。QNC 只需进行系统级测试,无需重新进行协议栈认证。

---

## 5. 捷径战略

### 5.1 四项取消决策

**取消项一 —— 完全放弃 R5F 实时固件工作线。**

原计划为 IO-Link 有限状态机、Modbus RTU 和 CANopen 开发 8-10 周的专用 Cortex-R5F 固件。netX 90 已将这三者作为经认证、经生产验证的协议栈交付。用以下方式替代整个 R5F 工作线:通过 cifX API 从 Linux 端配置 netX 90,并集成 Hilscher SDK。预计节省:8-10 周的专用固件工程工作量。

**取消项二 —— 放弃第一阶段的 PRU-ICSS EtherNet/IP 联调。**

原计划用 6-8 周时间,将工业以太网通过 AM64x PRU-ICSS 作为第一阶段的临时方案实现。取而代之的是,从第 1 天起就搭载 netX 90 —— 它在第一阶段处理 EtherNet/IP,并在第二阶段通过固件切换处理 PROFINET/EtherCAT,且无需任何额外硬件改动。成本:每台 BOM 增加 15-20 美元。节省:6-8 周,架构路径更为简洁统一。

**取消项三 —— 将 CANopen 与 Modbus TCP 扩展推迟至 MVP 后的固件配置阶段。**

两者均由 netX 90 固件支持。它们在 MVP 之后变为一周的固件配置任务,而非 4-6 周的开发工作线。

**取消项四 —— 用 MAX14819 + 主线 Linux 驱动取代自研 IO-Link 主站有限状态机。**

MAX14819 主线 Linux SPI 驱动(Analog Devices,已并入上游内核)省去了单独协议栈许可及自研固件开发的需求。节省:3-4 周。

### 5.2 三项加速决策

**加速项一 —— 采用 cifX 驱动作为协议抽象层。**

Hilscher cifX 驱动(C/C++,GitHub 上开源)与 PSM 的协议服务模块直接对应。它提供了一套简洁的过程数据 API:写入以发送过程映像、读取接收过程映像、处理事件。协议抽象层已经预先构建完成。QNC 的工作在于其上的语义映射。

**加速项二 —— 采用数据总线架构作为内部过程映像。**

将语义映射器实现为发布-订阅式数据总线:每个协议适配器(用于工业以太网的 cifX、用于 IO-Link 的 MAX14819 SPI 驱动、用于离散 IO 的 GPIO 层)向共享过程映像写入数据。北向 REST/WebSocket 服务器以可配置的死区阈值订阅该过程映像。这是多协议网关的正确架构,并可避免随协议增加而出现规则爆炸式增长。

**加速项三 —— 第一阶段采用 TI AM62x。**

AM62x(四核 Cortex-A53,仅支持 Linux)足以运行 QNC 运行时 + REST/WebSocket + cifX 驱动。AM64x 上的 R5F 和 PRU-ICSS 此前仅为自研实时固件所需,而这一需求现已由 netX 90 承担。AM62x 成本更低、PCB 更简单(无需 PRU-ICSS 工业以太网布线)、上电调试更快。若第二阶段某特定第二端口场景确实需要 PRU-ICSS,再升级至 AM64x。

### 5.3 商业前提条件:在全面投入前先锁定客户

商业分析证实,**在全面投入开发资源之前锁定 2-3 个试点客户承诺,是当前可获得的单一最高影响力风险缓释措施。** 盈亏平衡模型对销量高度敏感:保守情景(每年 200 台)三年内几乎勉强打平,而中性情景(每年 500 台)可产生 117 万美元的累计利润。这两种情景之间的差异在于客户管道,而非产品功能。

**行动建议:** 在投入全部硬件与软件开发预算之前,先从两三家代表异构生产线场景的机器人系统集成商处,确认并锁定购买意向(不一定是正式采购订单)。一封意向书或一份付费试点承诺,即足以支撑有信心地向前推进。

### 5.4 自研 vs 采购决策矩阵

| 组件 | 原方案 | 快速通道方案 | 节省的工作量 |
| --- | --- | --- | --- |
| EtherCAT 主站固件 | 基于 R5F 自研(6 周) | **netX 90 经认证协议栈** | 6 周 |
| PROFINET 设备固件 | 基于 R5F 自研(5 周) | **netX 90 经认证协议栈** | 5 周 |
| EtherNet/IP 适配器固件 | 基于 PRU-ICSS 自研(6 周) | **netX 90 经认证协议栈** | 6 周 |
| CANopen 协议栈 | 基于 R5F 自研(3 周) | **netX 90 经认证协议栈** | 3 周 |
| Modbus RTU 协议栈 | 基于 R5F 自研(3 周) | **netX 90 授权协议栈** | 3 周 |
| IO-Link 主站有限状态机 | 基于 R5F 自研(4 周) | **MAX14819 + ADI Linux 驱动** | 3 周 |
| 协议认证 | 自定义(15-45 万欧元) | **包含在 netX 90 授权费中** | 全部成本 |
| 内部数据模型 | 自定义(4 周) | **数据总线模式(成熟方案)** | 2 周 |
| ROS2 硬件接口 | 自研(4 周) | **ethercat\_driver\_ros2 + cifX** | 3 周 |
| 协议抽象 API | 自研(4 周) | **Hilscher cifX SDK** | 3 周 |
| **合计** |  |  | **约 38 个开发者·周** |

### 5.5 QNC 真正要构建的内容(真正的差异化优势)

应用捷径战略后,工程投入将集中于以下没有任何竞品具备的内容:

1. **YAML 设备配置文件治理引擎** —— 经验证、签名、生命周期管理的配置文件,并通过 `semantic_mapping` 关联至规范核心。目前没有竞品具备。
2. **带质量标记与死区滤波的数据总线过程映像** —— 将 cifX 过程数据映射到 CSC 遥测/故障/命令 JSON 的结构化类型数据模型。目前没有竞品具备。
3. **双 REST/WebSocket API** —— 机器人侧(`/robot/v1`)与北向(`/nb/v1`),各自独立版本管理。目前没有竞品具备。
4. **带回滚的原子化配置管理** —— YAML 工件分离、生成 ID、TPM 签名激活。目前没有竞品具备。
5. **带 `fault_domain` 隔离与异常旁路的安全模式** —— DEGRADED(降级)→ 关键信号的直接适配器回退。目前没有竞品具备。
6. **NEURA 生态集成** —— ROS2 `ros2_control` 硬件接口、MAiRA/LARA/MAV 兼容性矩阵、NeuraPy 模式。目前没有竞品具备。

---

## 6. 建议的 MVP 范围

### 6.1 MVP 定义

> **QNC MVP:** 一款运行于 TI AM62x + Hilscher netX 90 之上的导轨式网关设备,暴露 IO-Link(4 端口)、EtherNet/IP 适配器、Modbus RTU(2 端口)以及 8 路数字输入 + 8 路数字输出的南向接口,将设备数据归一化为带质量标记与死区滤波的发布-订阅过程映像,通过已签名的 YAML 配置文件映射至规范语义核心,并通过面向机器人侧的 REST/WebSocket API 发布结构化 JSON 遥测数据与命令 —— 同时具备安全模式、关键信号异常旁路,以及原子化配置回滚能力。

### 6.2 MVP 协议范围

| 协议 | 是否属于 MVP? | 理由 |
| --- | --- | --- |
| IO-Link v1.1 主站(4 端口) | ✅ 是 | MAiRA/LARA/MAV 的主要末端工具接口;IO-Link 智能工具核心场景 |
| EtherNet/IP 适配器(Class 1 + 3) | ✅ 是 | 23% 市场份额;netX 90 第 1 天即支持;PLC 连接 |
| Modbus RTU(2 端口) | ✅ 是 | 存量设备;netX 90 已授权;一周内可启用 |
| 离散数字 IO(8 DI + 8 DO) | ✅ 是 | 任何机器人单元均需要 |
| PROFINET 设备 | ⚡ 第 4 个月 | 27% 市场份额;固件切换;无需硬件改动 |
| EtherCAT 主站 | ⚡ 第 5 个月 | 17% 市场份额;固件切换;支持 ROS2 集成 |
| CANopen | 🔜 第 13 个月 | netX 90 固件;工业扩展包 |
| Modbus TCP | 🔜 第 13 个月 | 软件协议栈;工业扩展包 |
| OPC UA 服务器 | 🔜 第 14 个月 | open62541;北向扩展 |
| MQTT 客户端 | 🔜 第 13 个月 | 北向扩展 |
| CC-Link IE Field | 🔜 第 15 个月 | netX 90 固件;日本/中国市场 |

### 6.3 MVP 软件范围

**需构建(QNC 的差异化层):**

- QNC 运行时:生命周期状态(CSC §2.6)、重启/恢复、安全模式(NFR-004:≤1 秒)
- 数据总线过程映像:带 `(id, name, type, storage, timestamp_ns, quality, protocol_source, subscriber_mask)` 结构的类型化数据点;每个数据点可配置死区
- 异常旁路:健康监控器检测过程映像过期(超过 10 毫秒无更新)→ 激活直接适配器回退 → 发布 DEGRADED 生命周期状态
- 原子化配置与工件管理器:YAML 哈希校验、签名校验、生成 ID、回滚
- 配置文件解析器:YAML 模式验证、`semantic_mapping` 完整性检查、生命周期绑定
- 语义映射器:通过数据总线将 cifX 过程数据 + IO-Link 数据 → CSC 遥测/命令/故障 JSON
- 命令代理:关联、完成状态枚举、安全模式抑制
- 故障管理器:`fault_domain` 路由、锁存、带重启阶梯的恢复机制
- 北向服务器:`qnc-robot-api-v1.yaml` OpenAPI + `/robot/v1` 上的 WebSocket
- TPM 集成:YAML 工件签名、TLS 1.2+

**需集成(而非自研):**

- Hilscher cifX 驱动(netX 90 过程数据交换)
- MAX14819 SPI 驱动(ADI 主线 Linux 内核驱动)
- TI AM62x Yocto BSP(TI 提供;无需定制开发)
- `ethercat_driver_ros2` / cifX ROS2 硬件接口

**推迟至第二/三阶段:**

- 北向 API(`/nb/v1`)拆分 —— 在第 7 个月之前,机器人侧与北向共用同一 API,之后再拆分
- DDS 扩展包 —— MVP 之后的可选软件包
- 工业扩展包(OPC UA、MQTT、CANopen、Modbus TCP) —— MVP 之后
- Node-RED 节点库 —— 第三阶段
- 云端管理平台 —— 第三阶段

### 6.4 MVP 硬件范围

| 组件 | 决策 |
| --- | --- |
| 主控 SoC | TI AM6254(AM62x 四核 A53)—— 仅 Linux;快速通道方案无需 R5F/PRU-ICSS |
| 实时协处理器 | Hilscher netX 90 —— **从第 1 天起即搭载并启用**(而非预留不装) |
| netX 90 主机接口 | SPI(50 MHz;足以支持 8 设备过程映像交换) |
| PCB | 6 层板,IPC Class 2 等级 |
| IO-Link | MAX14819 ×2(4 端口)+ 主线 Linux SPI 驱动 |
| RS-485 | ISO1410 ×2(2 端口,Modbus RTU,5 kVrms 隔离) |
| 离散 IO | ISO1212 ×4(8 路数字输入)+ TPS272C45 ×4(8 路数字输出) |
| CAN | ISO1042 —— 预留封装位置,MVP 阶段不装配;第 5 个月启用 |
| TPM | SLB9672(SPI TPM 2.0) |
| 监控 MCU | STM32G071(电压轨监控、看门狗、复位) |
| 电源 | LM5069 热插拔 + LM76005 24V→5V 降压转换器 |

**相较 v3.0 的关键 MVP 硬件变化:** netX 90 从第 1 天起即搭载并启用。从 PCB 中移除 PRU-ICSS 工业以太网布线,原理图从 12 页减少至 10 页,并使全部第一阶段协议能力在 EVT(工程验证测试)硬件调试当天即可通过固件配置获得。

### 6.5 MVP 硬件 BOM 成本

| 配置 | BOM 成本(美元) | 目标使用场景 |
| --- | --- | --- |
| 基础版(仅 Modbus RTU + REST/WebSocket,不含 netX 90) | 30-45 美元 | 入门级存量产线改造 |
| **标准版(netX 90 + IO-Link + 离散 IO) —— 主打 SKU** | **65-80 美元** | 机器人系统集成商、设备制造商 |
| 全功能版(netX 90 + IO-Link + CAN + 双千兆以太网) | 85-100 美元 | OEM 嵌入式、全功能部署 |

标准配置在 399-499 美元的平均售价下,毛利率约为 85%。这是主打上市 SKU。

### 6.6 MVP 配置文件目录目标

| 阶段 | 配置文件数量 | 设备类别 |
| --- | --- | --- |
| 开发板演示(第 2 个月) | 2 个配置文件(开发签名) | 电动夹爪 · 力矩传感器 |
| EVT 硬件调试(第 4 个月) | 5 个配置文件(生产签名) | + IO-Link 接近传感器 · Modbus RTU 阀岛 · 离散 IO 安全继电器 |
| MVP GA(第 12 个月) | 10 个配置文件 | + 5 种覆盖主要使用场景的 MAiRA/LARA/MAV 末端工具类型 |

---

## 7. 分阶段路线图

### 概览

```
月份:  0    1    2    3    4    5    6    7    8    9   10   11   12   18   24
        ├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤
        │ 前 │◄──────── 第一阶段:速赢阶段(第 1-6 个月) ──────────────────►│
        │期  │                         │◄──── 第二阶段:MVP GA(6-12) ───────►│
        │    │                                               │◄── 第三阶段:规模化(12-24) ──►│
        │ M0 │     M0.5         M1                    M2           M3    M4
```

**里程碑图例:** M0 = 技术预验证 + 客户意向;M0.5 = 开发板演示;M1 = EVT 硬件 + 内部演示;M2 = 试点完成;M3 = GA 上市;M4 = 产品系列与规模化

---

### 第零阶段 —— 开发前验证(第 1-4 周,全面投入之前)

**目标:** 在投入全部开发预算前,验证商业赌注与技术可行性。

| 周次 | 行动 | 交付物 |
| --- | --- | --- |
| 第 1 周 | 采购 TI SK-AM62B 评估板及 Hilscher netX 90 开发套件 | 开发套件到位 |
| 第 1 周 | 启动客户调研:就异构生产线场景联系 5-8 个机器人系统集成商 | 访谈记录 |
| 第 1-2 周 | 与 Hilscher 展开商业合作洽谈:确认 netX 90 的授权条款及目标批量下的单价 | 条款清单/报价 |
| 第 2-3 周 | 在 AM62x 评估板上进行 cifX 驱动集成实验:加载 EtherNet/IP 固件、交换过程数据、测量延迟 | 实验报告:延迟基线对比 NFR-001 |
| 第 2-3 周 | MAX14819 SPI 驱动集成实验:确认主线驱动可在 AM62x BSP 上运行 | 实验报告:IO-Link 链路建立成功 |
| 第 3-4 周 | 锁定 2-3 个试点客户承诺(意向书或付费试点条款) | 已签署的意向文件 |
| 第 4 周 | 架构冻结决策:确认采用 AM62x;netX 90 第 1 天启用;数据总线内部模型;6 层 PCB | 架构决策记录 |

**M0 关卡(第 4 周末):** cifX 实验确认达到延迟目标;IO-Link 实验确认驱动可用;试点客户承诺已锁定;Hilscher 条款可接受;架构已冻结。

*若 cifX 延迟实验或客户承诺目标未能达成,应暂停全面投入并重新评估后再继续推进。*

---

### 第一阶段 —— 速赢阶段(第 1-6 个月)

**目标:** 在 EVT 硬件上运行可运行、可演示的网关,具备 IO-Link + EtherNet/IP + Modbus RTU + 离散 IO 南向接口、QNC 运行时、REST API 及 5 个设备配置文件。内部演示于第 5 个月前就绪。

#### 硬件方向(第 1-4 个月)

| 时段 | 交付物 |
| --- | --- |
| 第 1-2 周 | 架构冻结、连接器系列决策(M12 vs RJ45)、BOM 最终确定 |
| 第 2-4 周 | 提前下达采购订单:AM62x + netX 90 + MAX14819(交货周期 8-12 周) |
| 第 4-8 周 | 原理图设计:10 页(AM62x SoC、监控 MCU、netX 90 启用、北向千兆以太网、RS-485、IO-Link、离散 IO、安全/非易失存储、电源、测试) |
| 第 7-11 周 | PCB 布局,6 层板 —— 相较 v3.0,因移除 PRU-ICSS 工业以太网布线而简化(约节省 1.5 周) |
| 第 11-14 周 | 电路板生产及 EMS 组装(3-4 周生产周期) |
| 第 14-19 周 | EVT 调试:上电 → STM32G071 → AM62x Linux → netX 90 枚举 → IO-Link 端口 → 离散 IO |

#### 软件方向(第 1-5 个月,与 AM62x 评估板并行开展)

| 月份 | 交付物 |
| --- | --- |
| 第 1 个月 | 确认 TI AM62x Yocto BSP;cifX 驱动加载 EtherNet/IP 固件;netX 90 过程数据交换功能正常;MAX14819 IO-Link 链路在全部 4 个端口建立成功 |
| 第 2 个月 | QNC 运行时:生命周期状态、配置/回滚、故障管理器、安全模式 ≤1 秒;实现数据总线过程映像结构;完成前 2 个开发签名设备配置文件 |
| 第 3 个月 | 数据总线适配器:cifX → 过程映像(EtherNet/IP + Modbus RTU);MAX14819 → 过程映像(IO-Link);GPIO → 过程映像(离散 IO);死区滤波层;质量标记 |
| 第 4 个月 | 异常旁路路径:健康监控器(过期检测 >10 毫秒)→ 直接适配器回退 → 发布 DEGRADED 状态;命令代理;机器人侧 REST API(`qnc-robot-api-v1.yaml`);WebSocket 遥测 |
| 第 5 个月 | TPM 集成;YAML 签名校验;SEC-OPS-001–004;5 个生产签名配置文件;确认原子化回滚功能 |

#### 第一阶段关卡(第 5 个月末):内部演示

**成功标准:**

- 与参考 PLC 建立 EtherNet/IP 适配器会话
- 发现 IO-Link 设备,并通过 REST/WebSocket 流式传输遥测数据
- 确认离散 IO 组切换正常
- 轮询 Modbus RTU 设备,数据归一化为 CSC 遥测数据
- 安全模式进入时间 ≤ 1 秒(故障注入测试)
- 异常旁路:过程映像过期 → 在 <10 毫秒内完成直接适配器回退
- 原子化回滚:YAML 激活失败 → 恢复至上一代配置
- 验证通过 5 个生产签名配置文件

---

### 第二阶段 —— MVP 实施(第 6-12 个月)

**目标:** 通过 netX 90 固件切换实现 PROFINET + EtherCAT 的 GA 就绪产品、北向 API 拆分、10 个以上配置文件、DVT(设计验证测试)证据,以及与已承诺的集成商开展试点部署。

#### 第 6-7 个月:通过 netX 90 固件切换实现协议扩展(无需硬件变更)

| 任务 | 工作量 | 方法 |
| --- | --- | --- |
| PROFINET 设备模式 | 1-2 周 | netX 90 固件配置;cifX API 保持不变;与西门子/安川 PLC 联调测试 |
| EtherCAT 主站模式 | 1-2 周 | netX 90 固件;集成 `ethercat_driver_ros2`;与伺服驱动器联调测试 |

新增两个经一致性认证的主要工业以太网协议,总耗时 2-4 周。若从零开始自研,仅这两个协议的认证工作就要花费 6-12 个月。

#### 第 7 个月:北向 API 拆分

按 ICD §5.2 将 `qnc-northbound-api-v1.yaml` 与机器人侧 API 分离。独立版本管理、更严格的弃用通知、不同的鉴权策略。预计工作量:2 周。

#### 第 7-9 个月:DVT 与验证

聚焦于核心符合性检查清单(PRD §14)。相较 v3.0 更为精简,因为实时协议正确性已委托给 netX 90 的经认证协议栈 —— QNC 的 DVT 验证的是集成与语义层,而非协议栈本身。

| 测试项 | 目标 |
| --- | --- |
| 配置文件 YAML 验证(全部 10 个配置文件) | 100% 通过模式与签名校验 |
| 原子化配置回滚 | 每次故障路径注入后均恢复至上一代配置 |
| 南向协议矩阵:IO-Link、EtherNet/IP、PROFINET、Modbus RTU | 按 PSM §5.1 提供分协议证据 |
| `fault_domain` 隔离 | DDS 扩展包故障不影响南向状态(REQ-CORE-041) |
| 安全模式时序 | ≤ 1 秒(NFR-004) |
| 异常旁路路径 | 过期检测 → 回退时间 ≤ 10 毫秒 |
| 重启时序 | 冷启动 ≤ 30 秒(REQ-CORE-031) |
| REST 延迟 | EtherNet/IP 路径上 p95 ≤ 50 毫秒(NFR-001) |
| 容量浸泡测试 | 8 台设备、16 个 REST 会话、8 个 WS 连接,持续 72 小时 |
| 热测试 | 环境温度 50°C,最坏负载情况;满足所有结温限制 |
| EMC 预合规测试 | EN 55032 Class A;EN 61000-6-2 |

#### 第 9-11 个月:与已承诺集成商的测试版试点

- 向第零阶段已锁定意向的 2-3 个集成商部署 3-5 台设备
- 试点重点:异构生产线场景(商业价值最高的场景)
- 反馈收集:配置文件编写耗时、试运行时长、REST/WebSocket 集成难易程度、真实 PLC 流量下的协议稳定性
- 填充平台兼容性矩阵(PRD §11):来自试点的 MAiRA/LARA/MAV 数据行
- 生成 SBOM(SEC-OPS-004);启动 CE/FCC 申报

#### 第 11-12 个月:发布候选版与 GA

- 发布两份 OpenAPI 规范(`qnc-robot-api-v1.yaml`、`qnc-northbound-api-v1.yaml`)
- 完成 ORG v1.1 试运行运行手册,并与试点集成商共同评审
- 移交持续维护工程团队
- 下达首批量产订单(至少 100 台);确认 AM62x、MAX14819、netX 90、TPM 的末次采购(LTB)安排

**M3 关卡(第 12 个月):GA 上市**

成功标准:通过 netX 90 全部确认 PROFINET + EtherCAT + EtherNet/IP + Modbus RTU;两份 OpenAPI 规范均已发布;目录中有 10 个以上配置文件;DVT 证据齐全;通过 EMC 预合规测试;已纳入试点反馈;已发布 SBOM。

---

### 第三阶段 —— 规模化扩张(第 12-24 个月)

**目标:** 扩大协议覆盖范围、新增工业扩展包、发布 ROS2 驱动、构建软件生态、推进认证工作、拓展 OEM 及更广阔市场。

#### 第 12-15 个月:工业扩展包(每冲刺一项模式)

每个扩展包都是一个小型、独立、可单独发布的模块。由于 netX 90 负责实时部分、cifX 负责协议抽象,新增扩展包主要是软件配置与适配任务。

| 扩展包 | 工作量 | 方法 |
| --- | --- | --- |
| CANopen 主站 | 2-3 周 | netX 90 固件 + CiA402 配置文件,用于夹爪/驱动器集成 |
| Modbus TCP 桥接 | 2 周 | Linux 软件协议栈;复用 Modbus RTU 语义层 |
| OPC UA 服务器 | 3-4 周 | 在 AM62x 上运行 open62541;向 SCADA/MES 暴露机器人状态模型 |
| MQTT 客户端 | 2 周 | Eclipse Mosquitto;NEURA Cloud 遥测集成;支持 SaaS 模式 |
| CC-Link IE Field | 3-4 周 | netX 90 固件;对日本/中国机器人单元市场至关重要 |

#### 第 14-18 个月:ROS2 原生集成发布(生态投资回报最高)

ROS2 集成是目前没有任何竞争对手提供的独特差异化优势。将其作为一等公民、独立版本管理的交付物打包发布:

- `qnc_ros2_driver` 软件包:`ros2_control` 硬件接口 → cifX → netX 90
- 发布至 ROS2 Humble 生态系统及 ROS-Industrial Consortium
- 在 MAiRA 关节伺服(EtherCAT)及 LARA 外围设备(IO-Link)上完成测试验证
- 文档:用 15 分钟的 QNC 配置流程取代典型的自研 EtherCAT 调试过程
- 提交 ROSCon 演讲与演示

这将使 QNC 在全球 ROS2 社区中成为"如何将我的 NEURA 机器人连接到[任意工业协议]"这一问题的标准答案。

#### 第 15-16 个月:Node-RED 节点库(社区护城河)

为每个 QNC 协议适配器及 REST/WebSocket API 开发并开源 Node-RED 节点。以最小成本为社区存在感、集成粘性做出贡献。目标锁定从 OT 转向 IT 网关的、遵循 Elastel 模式的开发者社区。

#### 第 16-18 个月:云端管理平台 V0.5

- 设备注册、固件更新管理、遥测可视化
- 支持每设备年度订阅收入(SaaS 模式,场景三)
- 迈向机器人健康管理商业化服务的第一步

#### 第 15-20 个月:认证计划

| 认证 | 目标市场 | 备注 |
| --- | --- | --- |
| CE + FCC 正式认证 | 欧盟/美国市场 | 第二阶段完成预合规测试 → 正式测试 |
| PROFINET 一致性认证(PI) | 欧盟市场 | netX 90 协议栈已预先认证;仅需系统级测试 |
| EtherNet/IP 一致性认证(ODVA) | 北美市场 | 采取同样方式 |
| EtherCAT 主站认证(ETG) | 全球市场 | netX 90 已获 ETG 认证;仅需系统级测试 |
| UL 508A | 北美市场 | 系统集成商所需 |
| CC-Link IE 一致性认证(CLPA) | 日本、中国市场 | netX 90 已获授权;与三菱联合测试 |

#### 第 18-20 个月:实用新型专利申请(1-2 项)

针对硬件架构("集成多协议协处理器的工业网关")以及可能涉及 YAML 配置文件治理机制新颖之处,申请 1-2 项实用新型专利。主要目的:营销资质、政府补助资格,以及威慑小规模仿冒厂商。并非主要的护城河战略。

#### 第 18-24 个月:OEM 与产品线扩展

- 与机器人本体制造商(FANUC、KUKA、UR 生态系统)启动 OEM 嵌入式定价洽谈
- 面向价格敏感型存量改造市场推出基础版 SKU(30-45 美元 BOM,仅 Modbus + REST)
- 目标第二年年销量:1,500 台(中性情景);在年销量超过 500 台的所有情况下,单位经济效益均保持稳健

**M4 关卡(第 24 个月):规模化产品系列**

成功标准:已发布 5 个以上工业扩展包;`qnc_ros2_driver` 已发布并被采用;Node-RED 库已上线;至少获得一项正式协议认证;云端平台 V0.5 已投入运行;年运行速率 ≥1,000 台;OEM 洽谈活跃进行中。

---

### 里程碑摘要

| ID | 月份 | 里程碑 | 成功标准 |
| --- | --- | --- | --- |
| M0 | 第 4 周 | 预验证关卡 | cifX 延迟实验通过;IO-Link 实验通过;锁定 2-3 个试点承诺;Hilscher 条款可接受 |
| M0.5 | 第 2-3 个月 | 开发板演示 | EtherNet/IP + IO-Link + 离散 IO + QNC 运行时在评估板上运行 |
| M1 | 第 5 个月 | EVT 硬件内部演示 | 满足第一阶段全部关卡标准 |
| M2 | 第 9 个月 | DVT 完成 | 满足 PRD §14 全部符合性证据要求;通过 EMC 预合规测试 |
| M3 | 第 12 个月 | **GA 上市** | 已发布两份 OpenAPI 规范;10 个以上配置文件;已纳入试点反馈;已下达量产订单 |
| M4 | 第 24 个月 | 规模化产品系列 | 5 个以上扩展包;已发布 ROS2 驱动;至少 1 项正式认证;年运行速率 ≥1,000 台 |

---

## 8. 财务模型与单位经济效益

### 8.1 硬件 BOM 汇总

| SKU | BOM 成本(美元) | 目标售价(美元) | 毛利率 |
| --- | --- | --- | --- |
| 基础版(Modbus RTU + REST,不含 netX 90) | 30-45 美元 | 149-199 美元 | 约 78% |
| **标准版(netX 90 + IO-Link + 离散 IO)—— 主打款** | **65-80 美元** | **399-499 美元** | **约 85%** |
| 全功能版(netX 90 + IO-Link + CAN + 双千兆以太网) | 85-100 美元 | 599-799 美元 | 约 87% |

### 8.2 非经常性工程成本(MVP 阶段)

| 项目 | 预估成本(美元) |
| --- | --- |
| 硬件开发(原理图 + PCB + 3 次原型迭代) | 10,000-18,000 美元 |
| 软件开发(Linux BSP + cifX 集成 + QNC 运行时 + REST/WS API) | 35,000-65,000 美元 |
| 外壳(试点阶段采用现成或 3D 打印导轨式外壳) | 0-5,000 美元 |
| CE + FCC 预合规及正式测试 | 10,000-20,000 美元 |
| 协议一致性测试(1 个协议 —— 建议首选 PROFINET) | 5,000-15,000 美元 |
| **MVP 非经常性工程总成本** | **60,000-123,000 美元** |

相较于全定制实现方案,快速通道方案通过省去 R5F 固件开发及协议栈认证,将非经常性工程成本降低约 4-8 万美元。

### 8.3 三年期财务预测

#### 假设条件

| 参数 | 保守情景 | 中性情景 | 乐观情景 |
| --- | --- | --- | --- |
| 第一年销量 | 200 台 | 500 台 | 1,000 台 |
| 第二年销量 | 500 台 | 1,500 台 | 3,000 台 |
| 第三年销量 | 1,000 台 | 3,000 台 | 6,000 台 |
| 平均售价(ASP) | 299 美元 | 399 美元 | 499 美元 |
| 标准版 SKU BOM 成本 | 70 美元 | 70 美元 | 70 美元 |
| 单台毛利 | 229 美元 | 329 美元 | 429 美元 |

#### 三年累计预测

| 情景 | 三年销量 | 三年营收 | 三年毛利 | 非经常性工程成本 + 三年运营成本* | **净利润** |
| --- | --- | --- | --- | --- | --- |
| 保守 | 1,700 台 | 50.8 万美元 | 38.9 万美元 | 约 41.9 万美元 | **约 -3 万美元**(接近盈亏平衡) |
| **中性** | **5,000 台** | **200 万美元** | **165 万美元** | **约 55 万美元** | **约 +110 万美元** |
| 乐观 | 10,000 台 | 499 万美元 | 429 万美元 | 约 72 万美元 | **约 +357 万美元** |

*年度运营成本估算:12-20 万美元(工程维护、技术支持、云端运维、销售与市场推广)。

#### 关键财务结论

- **盈亏平衡点(中性情景):** 约第 18-20 个月
- **毛利率是结构性优势所在:** 所有 SKU 均超过 80%
- **销量是主要杠杆,而非毛利率:** 保守情景勉强打平;中性情景则相当具有吸引力
- **风险缓释:** 在全面投入非经常性工程资源前锁定 2-3 个试点承诺,可直接降低落入保守情景的概率

### 8.4 融资需求

| 阶段 | 资金需求(美元) | 用途 |
| --- | --- | --- |
| 种子轮(第 0-6 个月) | 6-10 万美元 | MVP 开发:硬件原型制作 + 软件核心 |
| Pre-A 轮(第 6-12 个月) | 15-25 万美元 | 小批量生产(100-500 台)+ 法规认证 + 试点支持 |
| A 轮(第 12-24 个月) | 50-100 万美元 | 规模化生产 + 销售团队 + 产品线扩展 + 云端平台 |

---

## 9. 竞争护城河战略

### 9.1 为何专利并非主要护城河

多协议工业网关功能属于既有现有技术。定制化的协议检测或转换算法或许具备可专利性,但竞争对手可以通过替代性的硬件或软件架构实现功能等效的行为。工业通信领域的专利诉讼成本高昂且耗时 —— 对早期阶段的产品而言并不现实。

**建议:** 就硬件架构以及可能涉及的 YAML 配置文件治理机制,申请 1-2 项实用新型专利。将其用于市场推广、政府补助资格获取,以及威慑直接仿冒者。不要在专利战略上投入大量资源,也不应将专利视为主要的竞争壁垒。

### 9.2 四项持久竞争优势

| 护城河 | 实现方式 | 有效原因 |
| --- | --- | --- |
| **协议广度 × 设备兼容深度** | 通过项目经验积累经认证的协议支持及真实设备兼容性。针对已发布规范未覆盖的非标准设备特殊情况进行优化。 | 竞争对手无法复制已积累的调试经验。每设备的兼容性知识会随时间不断积累叠加。 |
| **官方协议认证** | 争取 PROFINET(PI)、EtherNet/IP(ODVA)以及 EtherCAT(ETG)认证。每项认证都携带客户在询价中要求的可识别徽章。 | 认证成本与周期为新进入者设立了壁垒。netX 90 大幅降低了 QNC 自身的认证成本。 |
| **软件生态与工具链** | 配置工具(网页界面)、Node-RED 节点库、云端管理平台。一旦集成商采用 QNC 的工具工作流,转换成本就会很高。 | 工具采用会形成硬件规格无法带来的行为锁定效应。Elastel 的市场成功在很大程度上正是工具驱动的。 |
| **机器人原生集成(ROS2 驱动)** | `qnc_ros2_driver` 已发布至 ROS2 生态系统。是唯一具备原生 `ros2_control` → 现场总线集成能力的产品。 | 在 NEURA 及更广泛的人形/协作机器人市场中具有结构性排他优势。社区采用会不断叠加。 |

### 9.3 防御性知识产权:技术披露

对于不值得申请专利、但也不应被他人抢先申请专利的技术方案,应发布技术白皮书或将实现方案开源。这将形成现有技术证据,阻止第三方就这些方案提出专利主张,从而保护 QNC 的自由实施权。

---

## 10. 风险与缓释措施

### 10.1 快速通道特有风险

| ID | 风险 | 概率 | 影响 | 缓释措施 |
| --- | --- | --- | --- | --- |
| FT-R01 | Hilscher netX 90 单一来源依赖 | 中 | 高 | 将 ASIX AX58101 认证为仅支持 EtherCAT 的备用 SKU;保留 AM64x PRU-ICSS 作为 EtherNet/IP 的应急备选方案;通过 cifX 抽象层实现架构可移植性 |
| FT-R02 | Hilscher 授权成本高于预算 | 低 | 中 | 在第零阶段第 1-2 周确认商业条款;典型批量定价为每台 5-15 欧元 —— 相较 15-45 万欧元的认证替代方案而言可接受 |
| FT-R03 | 高设备数量或高周期速率下 cifX SPI 带宽不足 | 低 | 中 | 50 MHz 下的 SPI 支持约 800 字节/毫秒 —— 足以支撑 8 设备网关;AM64x 上提供无需 PCB 变更的 PCIe 升级路径 |
| FT-R04 | AM62x 在并发北向 + 协议负载下性能不足 | 低 | 中 | 依据 TI 参考配置,AM62x 四核 A53 可从容处理网关工作负载;若 8 设备下余量 <20%,升级至 AM64x 属于原理图兼容的替换 |
| FT-R05 | `ethercat_driver_ros2` 集成难度超出预期 | 中 | 中 | ADLINK/松下案例研究已经过生产验证;在第零阶段安排 2 周集成实验;cifX API 是有据可查的备用方案 |
| FT-R06 | 高数据点数量或高更新速率下数据总线过程映像性能问题 | 中 | 中 | 在第一阶段第 3 个月进行 1,000 点 × 1 kHz 的压力测试;若锁竞争成为问题,采用无锁环形缓冲区模式作为备选方案 |
| FT-R07 | 配置文件编写瓶颈:MAiRA/LARA 末端工具配置文件在 MVP 试点前未就绪 | 中 | 中 | 第 2 个月预先完成 2 个配置文件(夹爪 + 力矩传感器);从设备集成团队指派一名配置文件编写负责人;DPS v2.0 YAML 骨架模板加速编写工作 |
| FT-R08 | 销量未达到中性情景(第一年 500 台) | 中 | 高 | 在全面投入前锁定 2-3 个试点承诺;保守情景仍接近盈亏平衡;试点客户反馈可直接改善产品与市场的契合度 |
| FT-R09 | netX 90 在 PROFINET 与 EtherCAT 间进行固件切换需要重新提交 CE/FCC 认证 | 低 | 高 | 固件切换不改变硬件射频特性;CE/FCC 覆盖的是设备本身;Hilscher 预认证协议栈自带各自的协议一致性证据;在第一阶段与测试实验室确认相关解读 |

### 10.2 沿用风险(源自 v3.0,仍然适用)

| ID | 风险 | 缓释措施 |
| --- | --- | --- |
| R02 | 连接器/外壳方案未定 → PCB 被阻塞 | 在第 3 周锁定 M12 与 RJ45 的选型决策 |
| R09 | 规范命令命名空间注册表未预先建立 → 配置文件编写受阻 | 命名空间注册表为第零/第一阶段交付物;第 1 周指派软件架构负责人 |
| R12 | EMC 预合规测试未通过 | 第 6 个月前预订实验室;预留 2 周调优窗口期 |
| R16 | 客户材料中出现 SCADA/MES 范围蔓延 | 以 PRD §2.2 非目标声明作为关卡;任何对外发布前均需评审 |
| R17 | AM62x + MAX14819 + netX 90 交货周期较长 | 第 4 周提前下单(交货周期 8-12 周) |

### 10.3 快速通道战略已消除的风险

| 已消除风险 | 消除原因 |
| --- | --- |
| R5F 实时固件复杂度及进程间通信延迟 | 已移除 R5F 工作线;实时功能由 netX 90 承担 |
| PRU-ICSS EtherNet/IP 联调问题 | 已消除 PRU-ICSS EtherNet/IP 路径;第 1 天起即采用 netX 90 |
| ADI 方面 IO-Link 协议栈授权延迟 | MAX14819 使用主线 Linux 内核驱动;无需单独授权 |
| 协议认证成本(15-45 万欧元) | netX 90 协议栈自带 Hilscher 的一致性证据 |
| PRU-ICSS 引脚复用冲突 | 快速通道方案中工业以太网未使用 PRU-ICSS |
| netX 90 第二阶段 SKU 未确认 | netX 90 在第一阶段即已启用;SKU 从一开始就已确认 |
| 随协议增加而出现的协议转换规则爆炸式增长 | 数据总线架构避免了这一问题;每个新协议只需一个适配器 |

---

## 11. 预期收益

### 11.1 时间节省

| 指标 | v3.0 路线图 | 快速通道 v2.0 | 节省 |
| --- | --- | --- | --- |
| 首个可信软件演示 | 第 5-6 个月 | **第 2-3 个月** | 约 3 个月 |
| PROFINET + EtherCAT 可用 | 第 8 个月以后 | **第 6-7 个月**(固件切换) | 约 2 个月 |
| GA 上市 | 第 11 个月 | **第 12 个月** | 相当 |
| 首个 ROS2 驱动发布 | 未纳入计划 | **第 16 个月** | 新增能力 |
| 云端平台/SaaS 模式 | 不在范围内 | **第 16-18 个月** | 新增收入来源 |

硬件 GA 时间基本相当,因为 PCB 生产周期(约 3-4 周)是不可压缩的约束条件。快速通道的收益体现在软件信心、更早的协议广度,以及大幅降低的工程投入上。

### 11.2 成本降低

| 成本类别 | 原方案 | 快速通道 v2.0 | 节省 |
| --- | --- | --- | --- |
| 协议一致性认证 | 15-45 万欧元(自研协议栈) | 0 欧元(netX 90 预认证) | **15-45 万欧元** |
| R5F 固件工程 | 约 3-4 个专职工程师·月 | 已消除 | **约 6-12 万欧元** |
| PRU-ICSS EtherNet/IP 工程 | 约 2 个专职工程师·月 | 已消除 | **约 4-8 万欧元** |
| IO-Link 协议栈授权 | 存在不确定性及延迟 | ADI 主线驱动,零成本 | **1-3 万欧元** |
| **预计节省总额** |  |  | **26-68 万欧元** |

netX 90 按台授权费用(批量情况下约每台 5-15 欧元)是已知且可控的成本。相较自研认证方案,节省金额是授权成本的 20-50 倍。

### 11.3 复杂度与风险降低

| 复杂度指标 | v3.0 | 快速通道 v2.0 | 降低幅度 |
| --- | --- | --- | --- |
| 实时固件开发 | 自研(R5F + PRU-ICSS) | 无(netX 90) | 完全消除 |
| 内部数据模型 | 自研 | 数据总线(成熟模式) | 风险降低 |
| 原理图页数 | 12 页 | **10 页** | 减少 2 页 |
| 硬件调试路径 | A53 + R5F + PRU-ICSS + netX 90 | A53 + netX 90 | 减少 2 条路径 |
| 待验证的 DVT 协议栈数量 | 5 个自研协议栈 | 0 个自研(netX 90 已测试) | 减少 5 个 |
| 第一/二阶段外部认证阻塞项 | 3 项(PI、ODVA、ETG) | 推迟至第三阶段 | 减少 3 个关键路径阻塞项 |
| GA 阶段进度信心 | 预期 60% | **预期 75%** | 提升 15% 信心度 |

---

## 12. 假设、依赖与权衡

### 12.1 关键假设

| # | 假设 | 若不成立的影响 |
| --- | --- | --- |
| A1 | Hilscher netX 90 授权条款可接受(批量下约每台 5-15 欧元) | 若不可接受:改回 PRU-ICSS + R5F 方案实现 EtherNet/IP;进度延后 8 周 |
| A2 | cifX SPI 在 50 MHz 下为 8 设备网关过程映像提供足够带宽 | 若带宽不足:升级至 AM64x 上的 PCIe;需变更 PCB;影响约 4 周 |
| A3 | TI AM62x 四核 A53 有足够余量支撑 QNC 运行时 + 北向 + cifX | 若余量不足:更换为 AM64x;原理图兼容;约需 2 周返工 |
| A4 | `ethercat_driver_ros2` 可在 ≤2 周内适配至 QNC 语义层 | 若难度更高:进行 4 周专项攻关;不影响核心网关 MVP |
| A5 | MAX14819 主线 Linux SPI 驱动无需修改即可在 AM62x BSP 上运行 | 可能性很高;第零阶段安排 1 周实验以确认 |
| A6 | netX 90 在协议间的固件切换不需要重新提交 CE/FCC 认证 | 在第一阶段与测试实验室确认;根据 Hilscher 文档,风险较低 |
| A7 | 可在第零阶段锁定 2-3 个试点集成商承诺 | 关键商业假设;若未达成,应在全面投入非经常性工程资源前重新评估投资规模 |
| A8 | 1,000 点 × 1 kHz 的数据总线过程映像在 AM62x 上具备可接受的锁竞争水平 | 第一阶段第 3 个月进行压力测试;无锁环形缓冲区模式可作为备选方案 |

### 12.2 硬性依赖项

| 依赖项 | 所需时间点 | 交货周期 |
| --- | --- | --- |
| Hilscher netX 90 商业协议 | 第 2 周 | 2-4 周确认;第 1 周启动 |
| TI AM62x 评估板(SK-AM62B) | 第 1 周 | TI 可立即提供 |
| Hilscher cifX 驱动源码 + SDK | 第 2 周(商业协议达成之后) | Hilscher GitHub 免费提供 |
| MAX14819 生产样品 | 第 3 个月(EVT BOM) | 8-12 周 —— 第 4 周下单 |
| netX 90 生产样品 | 第 3 个月(EVT BOM) | 8-12 周 —— 第 4 周下单 |
| AM62x 生产样品 | 第 3 个月(EVT BOM) | 8-12 周 —— 第 4 周下单 |
| 锁定 2-3 个试点集成商承诺 | 第零阶段结束前 | 第 1 周起主动开展外联工作 |
| 用于配置文件编写的 MAiRA/LARA 末端工具设备访问权限 | 第 2 个月 | NEURA 内部资源;项目经理于第 1 个月确认 |
| 确认 EMS 合作伙伴并签署保密协议 | 第 3 个月(PCB 生产下单前) | 4 周上手周期 |

### 12.3 已接受的权衡

| 权衡 | 获得 | 牺牲 | 结论 |
| --- | --- | --- | --- |
| 第 1 天起启用 netX 90(而非预留不装) | 第一阶段协议广度;单一简洁的调试路径 | 每台 BOM 增加 15-20 欧元 | ✅ 已接受 —— 消除 PRU-ICSS 工作线 |
| 采用 AM62x 而非 AM64x | 更低的 BOM 成本;更简单的 PCB;更快的原理图设计 | 无 PRU-ICSS 备选路径 | ✅ 已接受 —— 如有需要可提供升级路径 |
| 数据总线优于协议转换器 | 未来新增协议只需适配器,而非规则表 | RAM 占用略高(1-4 MB 对比 256-512 KB) | ✅ 已接受 —— 可扩展性证明该开销合理 |
| 将 OPC UA / MQTT 推迟至第三阶段 | 更快交付 MVP | 北向扩展在 GA 阶段尚不可用 | ✅ 已接受 —— PRD §3.2 将其归类为扩展功能 |
| 将 DDS 扩展包推迟至第三阶段 | 更快交付 MVP | DDS 扩展包在 GA 阶段尚不可用 | ✅ 已接受 —— PRD §3.3 明确将其列为可选 |
| MVP 阶段采用单一 API(第 7 个月拆分) | 更快交付 Alpha 版本 | 暂时违反双 API 要求 | ⚠️ 仅可接受用于内部演示;试点前必须拆分 |
| 实用新型专利优于深度专利战略 | 营销资质;补助资格;仿冒威慑 | 并非主要知识产权护城河 | ✅ 已接受 —— 主要护城河在于生态,而非专利 |

### 12.4 不得推迟的事项

无论进度压力如何,以下事项在 MVP GA 阶段均不可协商:

- ✅ 每个设备配置文件中均包含 `semantic_mapping`(DPS v2.0 §7.1)
- ✅ 原子化配置回滚(PRD REQ-CORE-030)
- ✅ 每个故障事件均包含 `fault_domain` 区分标识(CSC §2.5)
- ✅ 过程映像过期情况下,关键信号的异常旁路路径
- ✅ 安全模式进入时间 ≤ 1 秒(PRD NFR-004)
- ✅ YAML 作为权威工件格式(PRD §12);JSON 仅作为运行时导出格式
- ✅ 生产配置文件采用 TPM 签名激活(DPS §15)
- ✅ 按试点阶段(而非仅按 GA 阶段)拆分两份 OpenAPI 文档 —— 机器人侧 + 北向(ICD §5.2)
- ✅ QNC **不是**安全等级设备 —— 在所有产品材料中均需明确声明(PRD §13)
- ✅ 在全面投入非经常性工程资源前,已锁定试点客户承诺(商业前提条件)

---

## 立即行动清单(前 30 天)

| 天数 | 行动 | 责任人 |
| --- | --- | --- |
| **第 1-3 天** | 采购 TI SK-AM62B 评估板及 Hilscher netX 90 开发套件 | 硬件负责人 |
| **第 1-3 天** | 就异构生产线场景,启动对 5-8 个机器人系统集成商的外联工作 | 项目经理 |
| **第 1-5 天** | 联系 Hilscher,启动商业授权洽谈 | 项目经理 + 采购 |
| **第 3-7 天** | 在 AM62x 评估板上启动 cifX 驱动集成实验 | 软件负责人 |
| **第 3-7 天** | 在 AM62x BSP 上启动 MAX14819 SPI 驱动实验 | 软件负责人 |
| **第 7-14 天** | 建立规范命令命名空间注册表雏形(夹爪、力矩传感器、接近传感器、阀门、离散 IO、传输类、管理类) | 软件架构师 |
| **第 7-14 天** | 锁定连接器系列决策(M12 vs RJ45) | 机械 + 项目经理 |
| **第 14-21 天** | 起草 2-3 份试点客户意向书;跟进初期外联工作 | 项目经理 |
| **第 21-28 天** | 架构冻结决策:确认 AM62x;netX 90 第 1 天启用;数据总线模型;6 层 PCB | 项目经理 + 硬件负责人 + 软件负责人 |
| **第 28 天** | M0 关卡评审:全部前提条件已满足 → 授权全面开发投入 | 全体相关方 |

---

---
---
# QNC Industrial Protocol Gateway — Fast\-Track Lean Roadmap

## 1. Key Insights Summary

The following insights from the translated business value and architecture analysis provide the highest value for the QNC fast-track strategy. Each insight either removes a significant roadmap risk, adds a commercial dimension previously absent, or provides an actionable decision that can be made immediately.

**Commercial Insights**

- **Protocol fragmentation is the real, persistent customer pain.** Multi-protocol mixed environments — robot bodies on EtherCAT, PLCs on PROFINET, vision systems on GigE Vision, sensors on IO-Link, supervisory systems expecting OPC UA — are the dominant integration challenge for robot system integrators today. The cost is not the hardware; it is the integration time. A gateway that eliminates 3–4 specialist converters and compresses commissioning from two weeks to three days justifies pricing at $1,500–$2,500 per unit.
- **The highest-value customer is the robot system integrator, not the end factory.** System integrators save days of debugging time per project. Their willingness to pay is high ($500–$2,000/unit) and their purchasing is project-driven, meaning pipeline builds predictably. This should be the target for the pilot.
- **OEM embedded customers are the volume path.** At large volume, unit pricing drops to $50–$200 but this segment drives scale (1,000+ units/year). OEM deals should be pursued from Month 13 onwards, not before the product is mature.
- **The financial model is attractive with high gross margins (\>80%) but front-loaded NRE.** Hardware BOM cost for the standard configuration (netX 90 + main CPU) is $55–75. At an ASP of $399 (mid-case), gross margin exceeds 85%. The break-even point in the neutral scenario occurs at Month 18–20. The biggest risk is insufficient sales volume, not development cost.
- **Start development only after locking 2–3 customer commitments.** The analysis recommends securing intent-to-purchase from pilot integrators before full development investment. This de-risks the single largest financial exposure: building a product that sells fewer units than projected.

**Architecture Insights**

- **The Data Bus (publish-subscribe process image) is the correct internal architecture for QNC.** Three internal patterns were compared: Protocol Converter (point-to-point rules), Data Bus (shared process image with publish-subscribe), and Hybrid. The Data Bus was recommended as the starting architecture because: adding a new protocol requires only writing an adapter (no rule explosion), the architecture supports multiple simultaneous readers at different update rates, and it naturally maps to the QNC semantic normalisation model. The Hybrid mode can be layered on top incrementally.
- **The Hybrid mode's three-channel routing pattern directly maps to QNC's data classification.** Channel 1 (real-time bus, 500 µs cycle): joint positions, torques, DIO states. Channel 2 (zero-copy high-speed stream): not needed for QNC MVP but applicable for future vision/point-cloud integration. Channel 3 (converter mode, 5 ms+): configuration, diagnostics, northbound telemetry. This matches QNC's existing protocol/northbound split.
- **Deadband filtering on the conversion layer prevents unnecessary API fan-out.** Each data point in the process image should carry a configurable deadband threshold; updates that fall within the band are suppressed before being published northbound. This reduces REST/WebSocket traffic under static or near-static device conditions — directly relevant to meeting NFR-002 (telemetry ≥10 Hz) without flooding northbound consumers.
- **An exception bypass path must exist for critical signals.** When the primary real-time bus path fails or goes stale, critical control signals (e.g., emergency-stop, target position) must fall back to a direct converter-mode path. In QNC terms this maps to the Safe Mode degradation path: fault manager detects stale process image → triggers fallback → publishes DEGRADED state → continues southbound operation via direct adapter calls.

**Go-to-Market Insights**

- **Heterogeneous robot production line integration is the highest commercial value scenario.** A single QNC gateway connecting KUKA (EtherCAT), ABB (PROFINET), FANUC (CC-Link IE), and UR cobots (Modbus TCP/EtherCAT) — with a unified OPC UA output for MES — is the anchor use case. It replaces 3–4 specialist gateways, each costing $500, with a single $1,500–$2,500 device that requires three days instead of two weeks to commission.
- **IO-Link smart tool integration is the robotics-native sweet spot.** Smart grippers, force-torque sensors, and cameras that communicate via IO-Link cannot connect to robot controllers directly. QNC as an IO-Link master that bridges to EtherCAT or PROFINET is a genuinely novel integration point that makes QNC a structural node in the EOAT ecosystem.
- **Protocol breadth, device compatibility depth, and ease-of-use tooling are the three durable competitive advantages** — not patents. Utility model patents are recommended (1–2 applications) for marketing and government grant eligibility, but the real moat is accumulated device compatibility, protocol certification badges, and a user-facing configuration tool that competitors cannot easily replicate.
- **Node-RED node library is a low-cost, high-visibility community investment.** Building and open-sourcing a Node-RED node library for QNC protocol adapters creates community presence, developer onboarding, and integration stickiness at minimal cost. This should be scheduled for Phase 3 alongside the cloud management platform.
- **A tiered product family (Basic / Standard / Full) is already financially justified.** The BOM cost analysis supports three SKUs: Basic (\~$30 BOM, Modbus + OPC UA only), Standard (\~$65 BOM, includes netX 90), Full (\~$85 BOM, adds IO-Link + CAN). Each targets a different buyer and volume tier.

---

## 2. Executive Summary

### The Core Insight

The competitive research and business analysis together confirm a single, decisive finding: **the market's real-time protocol problem is already solved in silicon by the Hilscher netX 90, and the market's commercial problem is the absence of a robotics-native semantic and integration layer on top of it.** HMS Anybus X-Gateway, Hilscher netTAP, and Moxa MGate all use the same certified real-time co-processor architecture. None of them expose robotics-native APIs, device profile governance, or NEURA ecosystem integration.

QNC's shortcut is to **build the semantic and integration layer** — the part no competitor has — while **delegating the certified real-time protocol layer** to the netX 90. The result is a product that delivers more value than the market leader in the dimensions that matter to robotics customers, while arriving faster and at lower development cost than a full custom implementation.

### The Shortcut in One Paragraph

Populate the **Hilscher netX 90 from Day 1** as the complete real-time protocol engine with certified EtherCAT, PROFINET, EtherNet/IP, and Modbus firmware stacks. Pair it with a **TI AM62x** application processor running Linux. Connect them via the **Hilscher cifX driver API** (mature, open-source, production-tested). Model the internal data as a **shared process image with publish-subscribe adapters** (Data Bus architecture), extended with a deadband filter layer before northbound publishing. Add QNC's differentiating layer: YAML device profiles with semantic normalisation, dual REST/WebSocket APIs, atomic configuration rollback, and Safe Mode. Before launching development, **secure intent-to-purchase from 2–3 robot system integrators** to validate the anchor use case. Target an ASP of $399–499 for a 85%+ gross margin and break-even at Month 18–20.

### Quantified Impact

| Dimension | Full Roadmap v3.0 | Fast-Track v2.0 | Saving |
| --- | --- | --- | --- |
| Time to working demo | \~Month 5–6 | **\~Month 2–3** | \~3 months |
| Time to GA | \~Month 11 | **\~Month 12** | Comparable |
| PROFINET + EtherCAT available | Month 8+ | **Month 6–7** | \~2 months |
| R5F firmware development | 11+ weeks | **Eliminated** | Full workstream |
| PRU-ICSS EtherNet/IP bringup | 8–10 weeks | **Eliminated** | Full workstream |
| Protocol certification cost | €150–450K | **€0** | Full cost avoided |
| Gross margin at $399 ASP | — | **\~85%** | Commercial clarity |
| Break-even (neutral scenario) | Not modelled | **Month 18–20** | Defined target |

---

## 3. Commercial Context and Market Opportunity

### 3.1 The Core Customer Pain: Protocol Fragmentation

Industrial automation environments — and robot production lines in particular — suffer from severe multi-protocol fragmentation. No single bus has won, and integrators must bridge incompatible worlds on every project.

| Pain Point | Concrete Impact |
| --- | --- |
| **Protocol islands** | Robot bodies run EtherCAT; PLCs run PROFINET; vision systems run GigE Vision; sensors run IO-Link; supervisory systems expect OPC UA. Each combination requires a separate converter or custom code. |
| **High integration cost** | System integrators buy multiple specialist gateways, write protocol-specific mapping code, and debug cross-vendor compatibility — consuming days to weeks per project. |
| **Data locked in the bus** | Device data cannot be uniformly collected to MES/ERP or cloud for analytics. Each protocol silo requires its own extraction toolchain. |

### 3.2 Target Customer Segments and Pricing

| Customer Type | Typical Use Case | Willingness to Pay | Acceptable Unit Price |
| --- | --- | --- | --- |
| **Robot system integrator** | Connecting different robot brands to a production line | High — saves significant commissioning time | $500–$2,000 |
| **Equipment manufacturer** | Adding multi-protocol communication to their own devices | Medium-high — as an optional add-on module | $100–$500 |
| **Factory automation department** | Brownfield line retrofit, cloud data integration | Medium | $300–$1,000 |
| **OEM embedded customer** | Volume embedding in their own product | High — prioritises long-term unit cost | $50–$200 at volume |

**Priority for MVP pilot:** Robot system integrators. They represent the highest willingness to pay, fastest purchase decision (project-driven), and best feedback quality for protocol compatibility and configuration tooling.

### 3.3 Highest-Value Commercial Scenarios

**Scenario 1 — Heterogeneous Robot Production Line Integration (Highest Value)**

A single line with KUKA (EtherCAT), ABB (PROFINET), FANUC (CC-Link IE), and UR cobots (Modbus TCP or EtherCAT) conventionally requires 3–4 different specialist gateways. QNC replaces all of them with one device that presents a unified OPC UA data model to MES and accepts unified commands from the production control system.

Commercial value: Single unit priced at $1,500–$2,500 versus four specialist gateways at \~$500 each, plus QNC compresses commissioning from two weeks to approximately three days. The integration labour saving alone — at typical integrator rates — is the dominant value driver.

**Scenario 2 — IO-Link Smart Tool Integration (Robotics-Native Sweet Spot)**

Smart grippers, force-torque sensors, and vision cameras communicating via IO-Link cannot connect directly to robot controllers, which do not support IO-Link mastering. QNC acts as the IO-Link master for up to four smart tools and bridges to the robot controller via EtherCAT or PROFINET. This positions QNC as a structural node in the EOAT ecosystem and creates a natural partnership opportunity with tool vendors.

**Scenario 3 — Robot Cloud Data Integration (IIoT Upsell)**

Factories want to collect vibration, temperature, current, and torque data from robot subsystems for predictive maintenance, but robot controllers either do not expose this data or limit access. QNC collects this via OPC UA or MQTT and forwards to NEURA Cloud or a third-party IIoT platform. This scenario supports a subscription revenue model (annual SaaS fee per device) that becomes activatable from Phase 3 onward.

### 3.4 Addressable Market

- Global industrial gateway market: \~$2.76B in 2024, projected $4.37B by 2031 (CAGR \~15%), with robotics-related connectivity representing approximately 20–30% of total — a $550M–$875M addressable segment growing at or above the market rate.
- Robotics-specific multi-protocol gateways are an underserved sub-segment with no dominant open-standard player. HMS Anybus X-Gateway dominates through hardware but lacks a software/API layer. QNC's robotics-native positioning occupies structurally unchallenged ground.

---

## 4. Key Findings from Research

### 4.1 Market Structure: Two Architectures, One Gap

The industrial gateway market divides cleanly into two camps — and QNC's opportunity is the gap between them.

| Camp | Examples | Strengths | Gap |
| --- | --- | --- | --- |
| **Real-time certified gateways** | HMS Anybus, Hilscher netTAP, Moxa MGate | Sub-1 ms deterministic fieldbus, full protocol conformance | No open API, no device profiles, no robotics context |
| **Open Linux IIoT gateways** | Elastel EG324, Advantech ADAM-6717, Red Lion DA30 | Open OS, Docker, MQTT, cloud integration | No real-time fieldbus, no EtherCAT/PROFINET, not robotics-grade |

**QNC's structural position:** The only product combining certified real-time fieldbus translation with an open, robotics-native semantic and API layer. No current competitor occupies this position.

### 4.2 The netX 90 Is the Industry's Real-Time Core

The Hilscher netX 90 has a 10/10 feasibility rating — the highest in the competitive analysis. It is already in NEURA evaluation. Crucially:

- Runs two simultaneous certified fieldbus stacks (xC0 + xC1) in dedicated hardware
- Certified firmware available for EtherCAT, PROFINET, EtherNet/IP, CANopen, Modbus RTU, CC-Link IE
- Sub-0.1 ms real-time guarantees independent of Linux application load
- cifX driver API is open-source, production-tested in thousands of deployments
- This is the same architecture HMS used to build a $2.76B market position

The R5F real-time firmware workstream in earlier roadmap versions is **entirely replaceable** by netX 90 licensed stacks.

### 4.3 Phase 1 Protocol Priority Confirmed by Market Data

HMS Networks 2025 market share data: PROFINET 27%, EtherNet/IP 23%, EtherCAT 17%, Modbus TCP 5%. These three protocols cover 67% of the target market and are all supported by netX 90 firmware swap — no hardware changes required between them.

### 4.4 The Data Bus Architecture Is Correct for QNC's Software Layer

Three internal software architectures were evaluated: Protocol Converter (point-to-point mapping rules), Data Bus (shared process image, publish-subscribe), and Hybrid. The Data Bus architecture is recommended as the foundation because:

- Adding a new protocol requires only writing one adapter to the shared process image — no rule explosion
- Multiple simultaneous consumers at different update rates are natively supported
- The architecture maps directly to QNC's semantic normalisation model (southbound frames → process image → canonical telemetry)
- The Hybrid routing layer can be added incrementally without architectural rework

**The Data Bus pattern is QNC's Semantic Mapper implementation strategy.** The shared process image is the internal representation of the CSC Telemetry structure, with each protocol adapter acting as a publisher and the northbound REST/WebSocket server acting as a subscriber.

### 4.5 Three Technical Patterns Directly Adoptable

**Deadband filtering** prevents unnecessary northbound fan-out. Each process image data point carries a configurable change threshold; only changes exceeding the threshold are forwarded northbound. This is critical for meeting NFR-002 (telemetry ≥10 Hz) without flooding REST/WebSocket clients under static device conditions.

**Exception bypass for critical signals** ensures that when the primary process image path becomes stale (no update within a configurable window, e.g., 10 ms), critical control signals fall back to a direct converter-mode path. In QNC terms: Fault Manager detects stale process image → publishes DEGRADED lifecycle state → activates direct adapter path for actuation-critical signals.

**Quality stamping on every process image point** (BAD / UNCERTAIN / GOOD, plus monotonic timestamp) maps directly to CSC `telemetry_quality` field values (`faulted` / `stale` / `valid`). Implementing quality stamps in the process image layer makes the `telemetry_quality` field automatically meaningful without additional logic in the Semantic Mapper.

### 4.6 ROS2 Integration Is the Genuine Differentiator

No current gateway product offers native ROS2 integration. The integration path exists as production-validated open-source: `ethercat_driver_ros2` (ICube-Robotics), cifX ROS2 hardware interface (ADLINK/Panasonic case study), and Intel ECI EtherCAT-to-ROS2 Gateway. This is a 1–2 week integration task, not a firmware development project, and it is the single feature that most strongly differentiates QNC for NEURA's robot ecosystem.

### 4.7 Protocol Certification Is the Highest-Cost Risk — netX 90 Eliminates It

Custom protocol stack development and conformance testing costs €50–150K per protocol. With three target protocols that is €150–450K and 12–18 months of certification timelines. The netX 90's licensed stacks arrive pre-certified by PI (PROFINET), ODVA (EtherNet/IP), and ETG (EtherCAT). QNC requires only system-level testing, not stack re-certification.

---

## 5. Shortcut Strategy

### 5.1 The Four Elimination Decisions

**Elimination 1 — Drop the R5F real-time firmware workstream entirely.**

Eight to ten weeks of specialised Cortex-R5F firmware development for IO-Link FSM, Modbus RTU, and CANopen. The netX 90 ships all three as certified, production-hardened stacks. Replace the entire R5F workstream with: configure the netX 90 via cifX API from Linux, integrate the Hilscher SDK. Estimated saving: 8–10 weeks of specialised firmware engineering.

**Elimination 2 — Drop PRU-ICSS EtherNet/IP Phase 1 bringup.**

Six to eight weeks routing industrial Ethernet through the AM64x PRU-ICSS as a Phase 1 stopgap. Instead, populate the netX 90 from Day 1 — it handles EtherNet/IP in Phase 1 *and* PROFINET/EtherCAT in Phase 2 via firmware swap, with zero additional hardware changes. Cost: +$15–20/unit BOM. Saving: 6–8 weeks, cleaner single-path architecture.

**Elimination 3 — Defer CANopen and Modbus TCP extensions to post-MVP firmware configuration.**

Both are supported by netX 90 firmware. They become one-week firmware configuration tasks post-MVP, not 4–6 week development workstreams.

**Elimination 4 — Replace custom IO-Link master FSM with MAX14819 + mainlined Linux driver.**

The MAX14819 mainlined Linux SPI driver (Analog Devices, upstream kernel) eliminates the need for a separate protocol stack license and custom firmware development. Saving: 3–4 weeks.

### 5.2 The Three Acceleration Decisions

**Acceleration 1 — cifX driver as the protocol abstraction layer.**

The Hilscher cifX driver (C/C++, open-source on GitHub) maps directly to PSM's Protocol Service module. It exposes a clean process-data API: write to transmit process image, read from receive process image, handle events. The protocol abstraction layer is pre-built. QNC's work is the semantic mapping above it.

**Acceleration 2 — Data Bus architecture as the internal process image.**

Implement the Semantic Mapper as a publish-subscribe Data Bus: each protocol adapter (cifX for industrial Ethernet, MAX14819 SPI driver for IO-Link, GPIO layer for DIO) writes to a shared process image. The northbound REST/WebSocket server subscribes to the process image with configurable deadband thresholds. This is the correct architecture for multi-protocol gateways and avoids rule explosion as protocols are added.

**Acceleration 3 — Use TI AM62x for Phase 1.**

The AM62x (quad Cortex-A53, Linux-only) is sufficient for running QNC runtime + REST/WebSocket + cifX driver. The R5F and PRU-ICSS on AM64x were needed only for custom real-time firmware, which is now handled by netX 90. AM62x is lower cost, simpler PCB (no PRU-ICSS industrial Ethernet routing), and faster to bring up. Upgrade to AM64x in Phase 2 if PRU-ICSS is needed for a specific second-port scenario.

### 5.3 Commercial Precondition: Lock Customers Before Full Investment

The business analysis confirms that **securing 2–3 pilot customer commitments before full development investment is the single highest-impact risk mitigation available.** The break-even model is sensitive to volume: the conservative scenario (200 units/year) barely breaks even over three years, while the neutral scenario (500 units/year) yields $1.17M cumulative profit. The difference between these scenarios is customer pipeline, not product features.

**Action:** Before committing the full hardware and software development budget, identify and secure intent-to-purchase (not necessarily purchase orders) from two or three robot system integrators who represent the heterogeneous production line scenario. A letter of intent or a paid pilot commitment is sufficient to proceed with confidence.

### 5.4 The Build-vs-Buy Decision Matrix

| Component | Previous Approach | Fast-Track Approach | Effort Saved |
| --- | --- | --- | --- |
| EtherCAT master firmware | Build on R5F (6 wks) | **netX 90 certified stack** | 6 wks |
| PROFINET device firmware | Build on R5F (5 wks) | **netX 90 certified stack** | 5 wks |
| EtherNet/IP adapter firmware | Build on PRU-ICSS (6 wks) | **netX 90 certified stack** | 6 wks |
| CANopen stack | Build on R5F (3 wks) | **netX 90 certified stack** | 3 wks |
| Modbus RTU stack | Build on R5F (3 wks) | **netX 90 licensed stack** | 3 wks |
| IO-Link master FSM | Build on R5F (4 wks) | **MAX14819 + ADI Linux driver** | 3 wks |
| Protocol certification | Custom (€150–450K) | **Included in netX 90 license** | Full cost |
| Internal data model | Custom (4 wks) | **Data Bus pattern (well-known)** | 2 wks |
| ROS2 hardware interface | Build (4 wks) | **ethercat\_driver\_ros2 + cifX** | 3 wks |
| Protocol abstraction API | Build (4 wks) | **Hilscher cifX SDK** | 3 wks |
| **Total** |  |  | **\~38 developer-weeks** |

### 5.5 What QNC Actually Builds (The Genuine Differentiator)

After applying the shortcut strategy, engineering effort concentrates on the things no competitor offers:

1. **YAML device profile governance engine** — validated, signed, lifecycle-managed profiles with `semantic_mapping` to the canonical core. No competitor has this.
2. **Data Bus process image with quality stamps and deadband filtering** — structured typed data model mapping cifX process data to CSC Telemetry/Fault/Command JSON. No competitor has this.
3. **Dual REST/WebSocket API** — robot-facing (`/robot/v1`) and northbound (`/nb/v1`) with independent versioning. No competitor has this.
4. **Atomic configuration management with rollback** — YAML artifact separation, generation IDs, TPM-signed activation. No competitor has this.
5. **Safe Mode with **`fault_domain`** isolation and exception bypass** — DEGRADED → direct adapter fallback for critical signals. No competitor has this.
6. **NEURA ecosystem integration** — ROS2 `ros2_control` hardware interface, MAiRA/LARA/MAV compatibility matrix, NeuraPy patterns. No competitor has this.

---

## 6. Recommended MVP Scope

### 6.1 MVP Definition

> **QNC MVP:** A DIN-rail gateway device running on TI AM62x + Hilscher netX 90 that exposes IO-Link (4 ports), EtherNet/IP adapter, Modbus RTU (2 ports), and 8 DI + 8 DO southbound, normalises device data into a publish-subscribe process image with quality stamps and deadband filtering, maps to the canonical semantic core via signed YAML profiles, and publishes structured JSON telemetry and commands via a robot-facing REST/WebSocket API — with Safe Mode, exception bypass for critical signals, and atomic configuration rollback.

### 6.2 MVP Protocol Scope

| Protocol | MVP? | Rationale |
| --- | --- | --- |
| IO-Link v1.1 master (4 ports) | ✅ YES | Primary EOAT interface for MAiRA/LARA/MAV; IO-Link smart tool anchor scenario |
| EtherNet/IP adapter (Class 1 + 3) | ✅ YES | 23% market share; netX 90 Day 1; PLC connectivity |
| Modbus RTU (2 ports) | ✅ YES | Brownfield devices; netX 90 licensed; 1 week to enable |
| Discrete DIO (8 DI + 8 DO) | ✅ YES | Required for any robot cell |
| PROFINET device | ⚡ Month 4 | 27% market; firmware swap; no hardware change |
| EtherCAT master | ⚡ Month 5 | 17% market; firmware swap; enables ROS2 integration |
| CANopen | 🔜 Month 13 | netX 90 firmware; Industrial Extension |
| Modbus TCP | 🔜 Month 13 | Soft stack; Industrial Extension |
| OPC UA Server | 🔜 Month 14 | open62541; northbound extension |
| MQTT Client | 🔜 Month 13 | Northbound extension |
| CC-Link IE Field | 🔜 Month 15 | netX 90 firmware; Japan/China market |

### 6.3 MVP Software Scope

**Build (QNC's differentiating layer):**

- QNC runtime: lifecycle states (CSC §2.6), restart/recovery, Safe Mode (NFR-004: ≤1 s)
- Data Bus process image: typed data points with `(id, name, type, storage, timestamp_ns, quality, protocol_source, subscriber_mask)` structure; configurable deadband per data point
- Exception bypass: health monitor detects stale process image (\>10 ms no update) → activates direct adapter fallback → publishes DEGRADED lifecycle state
- Atomic Config & Artifact Manager: YAML hash verify, signature verify, generation IDs, rollback
- Profile Resolver: YAML schema validation, `semantic_mapping` completeness, lifecycle binding
- Semantic Mapper: cifX process data + IO-Link data → CSC Telemetry/Command/Fault JSON via Data Bus
- Command Broker: correlation, completion enumeration, Safe Mode inhibit
- Fault Manager: `fault_domain` routing, latching, recovery with restart ladder
- Northbound Server: `qnc-robot-api-v1.yaml` OpenAPI + WebSocket on `/robot/v1`
- TPM integration: YAML artifact signing, TLS 1.2+

**Integrate (not build):**

- Hilscher cifX driver (netX 90 process data exchange)
- MAX14819 SPI driver (ADI mainlined Linux kernel driver)
- TI AM62x Yocto BSP (TI provides; no custom work)
- `ethercat_driver_ros2` / cifX ROS2 hardware interface

**Defer to Phase 2/3:**

- Northbound API (`/nb/v1`) split — use robot-facing API for both until Month 7, then split
- DDS Pack — post-MVP optional package
- Industrial Extensions (OPC UA, MQTT, CANopen, Modbus TCP) — post-MVP
- Node-RED node library — Phase 3
- Cloud management platform — Phase 3

### 6.4 MVP Hardware Scope

| Component | Decision |
| --- | --- |
| Main SoC | TI AM6254 (AM62x quad A53) — Linux-only; no R5F/PRU-ICSS needed for fast-track |
| Real-time co-processor | Hilscher netX 90 — **populated and active from Day 1** (not DNP) |
| netX 90 host interface | SPI (50 MHz; sufficient for 8-device process image exchange) |
| PCB | 6-layer, IPC Class 2 |
| IO-Link | MAX14819 ×2 (4 ports) + mainlined Linux SPI driver |
| RS-485 | ISO1410 ×2 (2 ports, Modbus RTU, 5 kVrms isolated) |
| DIO | ISO1212 ×4 (8 DI) + TPS272C45 ×4 (8 DO) |
| CAN | ISO1042 — footprint present, DNP in MVP; enable in Month 5 |
| TPM | SLB9672 (SPI TPM 2.0) |
| Supervisor MCU | STM32G071 (rail monitoring, watchdog, reset) |
| Power | LM5069 hot-swap + LM76005 24V→5V buck |

**Key MVP hardware change vs v3.0:** netX 90 populated and active from Day 1. Removes PRU-ICSS industrial Ethernet routing from PCB, reduces schematic from 12 to 10 sheets, and makes full Phase 1 protocol capability available via firmware configuration on day of EVT bring-up.

### 6.5 MVP Hardware BOM Cost

| Configuration | BOM Cost (USD) | Target Use Case |
| --- | --- | --- |
| Basic (Modbus RTU + REST/WebSocket only, no netX 90) | $30–45 | Entry-level brownfield retrofit |
| **Standard (netX 90 + IO-Link + DIO) — Primary SKU** | **$65–80** | Robot system integrator, equipment manufacturer |
| Full (netX 90 + IO-Link + CAN + dual GbE) | $85–100 | OEM embedded, full-featured deployment |

Standard configuration carries \~85% gross margin at $399–499 ASP. This is the launch SKU.

### 6.6 MVP Profile Catalog Target

| Phase | Profile Count | Device Classes |
| --- | --- | --- |
| Dev board demo (Month 2) | 2 profiles (dev-signed) | Electric gripper · Force-torque sensor |
| EVT hardware bring-up (Month 4) | 5 profiles (production-signed) | + IO-Link proximity · Modbus RTU valve island · DIO safety relay |
| MVP GA (Month 12) | 10 profiles | + 5 MAiRA/LARA/MAV EOAT types covering primary use cases |

---

## 7. Phased Roadmap

### Overview

```
Month:  0    1    2    3    4    5    6    7    8    9   10   11   12   18   24
        ├────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┼────┤
        │ Pre│◄──────── PHASE 1: QUICK WINS (Months 1–6) ──────────────────►│
        │    │                         │◄──── PHASE 2: MVP GA (6–12) ───────►│
        │    │                                               │◄── PHASE 3: SCALE (12–24) ──►│
        │ M0 │     M0.5         M1                    M2           M3    M4
```

**Milestone legend:** M0 = tech pre-validation + customer intent; M0.5 = dev board demo; M1 = EVT hardware + internal demo; M2 = pilot complete; M3 = GA launch; M4 = product family and scale

---

### Phase 0 — Pre-Development Validation (Weeks 1–4, before full investment)

**Goal:** Validate the commercial bet and technical feasibility before committing the full development budget.

| Week | Action | Deliverable |
| --- | --- | --- |
| Week 1 | Procure TI SK-AM62B evaluation board and Hilscher netX 90 development kit | Dev kits on desk |
| Week 1 | Begin customer discovery: contact 5–8 robot system integrators on the heterogeneous production line scenario | Call notes |
| Week 1–2 | Commercial agreement outreach with Hilscher: confirm netX 90 licensing terms and per-unit pricing at target volumes | Term sheet / quote |
| Week 2–3 | Run cifX driver integration spike on AM62x eval board: load EtherNet/IP firmware, exchange process data, measure latency | Spike report: latency baseline vs NFR-001 |
| Week 2–3 | MAX14819 SPI driver integration spike: confirm mainlined driver works on AM62x BSP | Spike report: IO-Link link established |
| Week 3–4 | Lock 2–3 pilot customer commitments (letter of intent or paid pilot terms) | Signed intent documents |
| Week 4 | Architecture freeze decision: AM62x confirmed; netX 90 active Day 1; Data Bus internal model; 6-layer PCB | Architecture Decision Record |

**Gate M0 (End of Week 4):** cifX spike confirms latency target; IO-Link spike confirms driver; pilot commitments secured; Hilscher terms acceptable; architecture frozen.

*If either the cifX latency spike or the customer commitment target fails, pause full investment and reassess before proceeding.*

---

### Phase 1 — Quick Wins (Months 1–6)

**Goal:** Working, demonstrable gateway with IO-Link + EtherNet/IP + Modbus RTU + DIO southbound, QNC runtime, REST API, 5 device profiles running on EVT hardware. Internal demo ready by Month 5.

#### Hardware Track (Months 1–4)

| Period | Deliverable |
| --- | --- |
| Wk 1–2 | Architecture freeze, connector family decision (M12 vs RJ45), BOM finalised |
| Wk 2–4 | Advance procurement order: AM62x + netX 90 + MAX14819 (8–12 week lead times) |
| Wk 4–8 | Schematic design: 10 sheets (AM62x SoC, Supervisor MCU, netX 90 active, northbound GbE, RS-485, IO-Link, DIO, Security/NVM, Power, Test) |
| Wk 7–11 | PCB layout, 6-layer — simplified vs v3.0 by removal of PRU-ICSS industrial Ethernet routing (\~1.5 weeks saved) |
| Wk 11–14 | Board fabrication and EMS assembly (3–4 week fab lead time) |
| Wk 14–19 | EVT bring-up: Power → STM32G071 → AM62x Linux → netX 90 enumerate → IO-Link port → DIO |

#### Software Track (Months 1–5, parallel on AM62x eval board)

| Month | Deliverable |
| --- | --- |
| Month 1 | TI AM62x Yocto BSP confirmed; cifX driver loading EtherNet/IP firmware; netX 90 process data exchange functional; MAX14819 IO-Link link established on all 4 ports |
| Month 2 | QNC runtime: lifecycle states, config/rollback, fault manager, Safe Mode ≤1 s; Data Bus process image structure implemented; first 2 dev-signed device profiles |
| Month 3 | Data Bus adapters: cifX → process image (EtherNet/IP + Modbus RTU); MAX14819 → process image (IO-Link); GPIO → process image (DIO); deadband filter layer; quality stamps |
| Month 4 | Exception bypass path: health monitor (stale detection \>10 ms) → direct adapter fallback → DEGRADED state publication; Command Broker; Robot-facing REST API (`qnc-robot-api-v1.yaml`); WebSocket telemetry |
| Month 5 | TPM integration; YAML signature verification; SEC-OPS-001–004; 5 production-signed profiles; atomic rollback confirmed |

#### Phase 1 Gate (End of Month 5): Internal Demo

**Success criteria:**

- EtherNet/IP adapter session established with reference PLC
- IO-Link device discovered and telemetry streaming via REST/WebSocket
- DIO bank toggle confirmed
- Modbus RTU device polled and data normalised to CSC telemetry
- Safe Mode entry ≤ 1 s (fault injection)
- Exception bypass: stale process image → direct adapter fallback in \<10 ms
- Atomic rollback: failed YAML activation → prior generation restored
- 5 production-signed profiles validated

---

### Phase 2 — MVP Implementation (Months 6–12)

**Goal:** GA-ready product with PROFINET + EtherCAT via netX 90 firmware swap, northbound API split, 10+ profiles, DVT evidence, and pilot deployment with committed integrators.

#### Month 6–7: Protocol Expansion via netX 90 Firmware Swap (No Hardware Change)

| Task | Effort | Method |
| --- | --- | --- |
| PROFINET device mode | 1–2 weeks | netX 90 firmware configuration; cifX API unchanged; test with Siemens/YASKAWA PLC |
| EtherCAT master mode | 1–2 weeks | netX 90 firmware; `ethercat_driver_ros2` integration; test with servo drive |

Adding two major industrial Ethernet protocols with conformance-tested stacks takes 2–4 weeks total. A team building from scratch would spend 6–12 months on protocol certification alone for the same two protocols.

#### Month 7: Northbound API Split

Separate `qnc-northbound-api-v1.yaml` from robot-facing API per ICD §5.2. Independent versioning, stricter deprecation notice, different authentication policies. Estimated effort: 2 weeks.

#### Month 7–9: DVT and Validation

Focused on the Core conformance checklist (PRD §14). Leaner than v3.0 because the real-time protocol correctness is delegated to netX 90's certified stacks — QNC's DVT proves the integration and semantic layer, not the protocol stacks themselves.

| Test | Target |
| --- | --- |
| Profile YAML validation (all 10 profiles) | 100% schema + signature pass |
| Atomic config rollback | Prior generation restored on every failure-path injection |
| Southbound protocol matrix: IO-Link, EtherNet/IP, PROFINET, Modbus RTU | Per-protocol PSM §5.1 evidence |
| `fault_domain` isolation | DDS Pack fault does not affect southbound states (REQ-CORE-041) |
| Safe Mode timing | ≤ 1 s (NFR-004) |
| Exception bypass path | Stale detection → fallback in ≤ 10 ms |
| Restart timing | ≤ 30 s cold-start (REQ-CORE-031) |
| REST latency | ≤ 50 ms p95 on EtherNet/IP path (NFR-001) |
| Capacity soak | 8 devices, 16 REST sessions, 8 WS connections, 72 hours |
| Thermal | 50 °C ambient, worst-case load; all junction limits met |
| EMC pre-compliance | EN 55032 Class A; EN 61000-6-2 |

#### Month 9–11: Beta Pilot with Committed Integrators

- Deploy 3–5 units to the 2–3 integrators whose intent was secured in Phase 0
- Pilot focus: heterogeneous production line scenario (highest commercial value scenario)
- Feedback collection: profile authoring time, commissioning duration, REST/WebSocket integration ease, protocol stability under real PLC traffic
- Platform compatibility matrix (PRD §11) populated: MAiRA/LARA/MAV rows from pilot
- SBOM generated (SEC-OPS-004); CE/FCC filing initiated

#### Month 11–12: Release Candidate and GA

- Both OpenAPI specs published (`qnc-robot-api-v1.yaml`, `qnc-northbound-api-v1.yaml`)
- ORG v1.1 commissioning runbook finalised and reviewed with pilot integrators
- Sustaining engineering handoff
- First series production order placed (minimum 100 units); LTB confirmed for AM62x, MAX14819, netX 90, TPM

**Gate M3 (Month 12): GA Launch**

Success criteria: PROFINET + EtherCAT + EtherNet/IP + Modbus RTU all confirmed via netX 90; both OpenAPI specs published; 10+ profiles in catalog; DVT evidence complete; EMC pre-compliance pass; pilot feedback incorporated; SBOM published.

---

### Phase 3 — Scale-Up (Months 12–24)

**Goal:** Expand protocol coverage, add Industrial Extensions, publish ROS2 driver, build software ecosystem, pursue certification, target OEM and broader markets.

#### Months 12–15: Industrial Extensions (One-Per-Sprint Pattern)

Each extension is a small, isolated, releasable module. Because netX 90 handles the real-time side and cifX abstracts the protocol, adding an extension is primarily a software configuration and adapter task.

| Extension | Effort | Method |
| --- | --- | --- |
| CANopen master | 2–3 weeks | netX 90 firmware + CiA402 profile for gripper/drive integration |
| Modbus TCP bridge | 2 weeks | Linux soft stack; Modbus RTU semantic layer reused |
| OPC UA Server | 3–4 weeks | open62541 on AM62x; exposes robot state model to SCADA/MES |
| MQTT Client | 2 weeks | Eclipse Mosquitto; NEURA Cloud telemetry integration; enables SaaS model |
| CC-Link IE Field | 3–4 weeks | netX 90 firmware; critical for Japan/China robot cell market |

#### Months 14–18: ROS2 Native Integration Release (Highest Ecosystem ROI)

ROS2 integration is the unique differentiator that no competitor offers. Package and publish as a first-class, separately versioned deliverable:

- `qnc_ros2_driver` package: `ros2_control` hardware interface → cifX → netX 90
- Published to ROS2 Humble ecosystem and ROS-Industrial Consortium
- Tested and validated with MAiRA joint servos (EtherCAT) and LARA peripherals (IO-Link)
- Documentation: replacing typical DIY EtherCAT bringup with a 15-minute QNC setup
- ROSCon submission and demonstration

This positions QNC as the standard answer to "how do I connect my NEURA robot to \[any industrial protocol\]" in the global ROS2 community.

#### Months 15–16: Node-RED Node Library (Community Moat)

Open-source Node-RED nodes for each QNC protocol adapter and the REST/WebSocket APIs. Contributes to community presence and creates integration stickiness at minimal cost. Targets the Elastel-pattern developer community migrating from OT-to-IT gateways.

#### Months 16–18: Cloud Management Platform V0.5

- Device registration, firmware update management, telemetry visualisation
- Enables annual subscription revenue per device (SaaS model, Scenario 3)
- First step toward robot health management commercial offering

#### Months 15–20: Certification Program

| Certification | Target | Notes |
| --- | --- | --- |
| CE + FCC formal | EU/US market | Pre-compliance pass in Phase 2 → formal testing |
| PROFINET conformance (PI) | EU market | netX 90 stacks pre-certified; system-level testing only |
| EtherNet/IP conformance (ODVA) | North America | Same approach |
| EtherCAT master certification (ETG) | Global | netX 90 ETG-certified; system-level testing |
| UL 508A | North America | Required for system integrators |
| CC-Link IE conformance (CLPA) | Japan, China | netX 90 licensed; testing with Mitsubishi |

#### Months 18–20: Utility Model Patent Filing (1–2 Applications)

File 1–2 utility model patents targeting the hardware architecture ("industrial gateway integrating multi-protocol co-processor") and potentially a novel aspect of the YAML profile governance mechanism. Primary purpose: marketing credential, government grant eligibility, and deterrence of small-scale copycat manufacturers. Not a primary moat strategy.

#### Months 18–24: OEM and Product Line Expansion

- Initiate OEM embedded pricing conversations with robot body manufacturers (FANUC, KUKA, UR ecosystem)
- Launch Basic SKU ($30–45 BOM, Modbus + REST only) for price-sensitive brownfield retrofit market
- Target year-2 annual volume: 1,500 units (neutral scenario); unit economics remain strong at all volumes above 500 units/year

**Gate M4 (Month 24): Scaled Product Family**

Success criteria: 5+ Industrial Extensions released; `qnc_ros2_driver` published and adopted; Node-RED library live; at least one formal protocol certification; cloud platform V0.5 operating; annual run-rate ≥1,000 units; OEM conversations active.

---

### Milestone Summary

| ID | Month | Milestone | Success Criteria |
| --- | --- | --- | --- |
| M0 | Wk 4 | Pre-validation gate | cifX latency spike passes; IO-Link spike passes; 2–3 pilot commitments; Hilscher terms acceptable |
| M0.5 | Month 2–3 | Dev board demo | EtherNet/IP + IO-Link + DIO + QNC runtime running on eval board |
| M1 | Month 5 | Internal demo on EVT hardware | All Phase 1 gate criteria met |
| M2 | Month 9 | DVT complete | Full PRD §14 conformance evidence; EMC pre-compliance pass |
| M3 | Month 12 | **GA launch** | Both OpenAPI specs published; 10+ profiles; pilot feedback incorporated; production order placed |
| M4 | Month 24 | Scaled product family | 5+ extensions; ROS2 driver published; ≥1 formal certification; annual run-rate ≥1,000 units |

---

## 8. Financial Model and Unit Economics

### 8.1 Hardware BOM Summary

| SKU | BOM Cost (USD) | Target ASP (USD) | Gross Margin |
| --- | --- | --- | --- |
| Basic (Modbus RTU + REST, no netX 90) | $30–45 | $149–199 | \~78% |
| **Standard (netX 90 + IO-Link + DIO) — Primary** | **$65–80** | **$399–499** | **\~85%** |
| Full (netX 90 + IO-Link + CAN + dual GbE) | $85–100 | $599–799 | \~87% |

### 8.2 Non-Recurring Engineering Cost (MVP Phase)

| Item | Estimated Cost (USD) |
| --- | --- |
| Hardware development (schematic + PCB + 3 prototype iterations) | $10,000–18,000 |
| Software development (Linux BSP + cifX integration + QNC runtime + REST/WS APIs) | $35,000–65,000 |
| Enclosure (off-the-shelf or 3D-printed DIN rail housing for pilot phase) | $0–5,000 |
| CE + FCC pre-compliance and formal testing | $10,000–20,000 |
| Protocol conformance testing (1 protocol — PROFINET recommended first) | $5,000–15,000 |
| **Total MVP NRE** | **$60,000–123,000** |

The fast-track approach reduces NRE versus a full custom implementation by approximately $40,000–80,000 through elimination of R5F firmware development and protocol stack certification.

### 8.3 Three-Year Financial Projection

#### Assumptions

| Parameter | Conservative | Neutral | Optimistic |
| --- | --- | --- | --- |
| Year 1 unit sales | 200 | 500 | 1,000 |
| Year 2 unit sales | 500 | 1,500 | 3,000 |
| Year 3 unit sales | 1,000 | 3,000 | 6,000 |
| Average selling price (ASP) | $299 | $399 | $499 |
| Standard SKU BOM cost | $70 | $70 | $70 |
| Gross margin per unit | $229 | $329 | $429 |

#### Three-Year Cumulative Projection

| Scenario | 3-Yr Units | 3-Yr Revenue | 3-Yr Gross Profit | NRE + 3-Yr Ops\* | **Net Profit** |
| --- | --- | --- | --- | --- | --- |
| Conservative | 1,700 | $508K | $389K | \~$419K | **\~−$30K** (near break-even) |
| **Neutral** | **5,000** | **$2.0M** | **$1.65M** | **\~$550K** | **\~+$1.1M** |
| Optimistic | 10,000 | $4.99M | $4.29M | \~$720K | **\~+$3.57M** |

\*Annual operating cost estimate: $120–200K (engineering maintenance, technical support, cloud ops, sales and marketing).

#### Key Financial Conclusions

- **Break-even (neutral scenario):** Approximately Month 18–20
- **Gross margin is the structural strength:** \>80% across all SKUs
- **Volume is the primary lever, not margin:** Conservative scenario barely breaks even; neutral scenario is highly attractive
- **Risk mitigation:** Securing 2–3 pilot commitments before full NRE investment directly reduces the probability of the conservative scenario

### 8.4 Funding Requirement

| Stage | Capital Need (USD) | Purpose |
| --- | --- | --- |
| Seed (Months 0–6) | $60K–100K | MVP development: hardware prototyping + software core |
| Pre-A (Months 6–12) | $150K–250K | Small batch production (100–500 units) + regulatory certification + pilot support |
| Series A (Months 12–24) | $500K–1M | Scale production + sales team + product line expansion + cloud platform |

---

## 9. Competitive Moat Strategy

### 9.1 Why Patents Are Not the Primary Moat

Multi-protocol industrial gateway functionality is established prior art. Custom protocol detection or conversion algorithms may be patentable, but competitors can implement functionally equivalent behaviour through alternative hardware or software architectures. Patent litigation in industrial communications is expensive and time-consuming — not practical for an early-stage product.

**Recommendation:** File 1–2 utility model patents for hardware architecture and potentially the YAML profile governance mechanism. Use them for marketing purposes, government grant eligibility, and deterrence of direct copycats. Do not invest significant resources in patent strategy or treat patents as a primary competitive barrier.

### 9.2 The Four Durable Competitive Advantages

| Moat | Implementation | Why It Works |
| --- | --- | --- |
| **Protocol breadth × device compatibility depth** | Accumulate certified protocol support and real-world device compatibility through project experience. Optimise for non-standard device quirks that published specs don't cover. | Competitors cannot replicate accumulated debugging experience. Per-device compatibility knowledge compounds over time. |
| **Official protocol certifications** | Pursue PROFINET (PI), EtherNet/IP (ODVA), and EtherCAT (ETG) certifications. Each certification carries a recognisable badge that customers require in RFQs. | Certification costs and timelines create a barrier for new entrants. netX 90 dramatically reduces QNC's own certification cost. |
| **Software ecosystem and tooling** | Configuration tool (web UI), Node-RED node library, cloud management platform. Once integrators adopt QNC's tooling workflow, switching cost is high. | Tool adoption creates behavioural lock-in that hardware specs cannot. Elastel's market success is largely tool-driven. |
| **Robotics-native integration (ROS2 driver)** | `qnc_ros2_driver` published in ROS2 ecosystem. Only product with native `ros2_control` → fieldbus integration. | Structural exclusivity in the NEURA and broader humanoid/collaborative robot market. Community adoption compounds. |

### 9.3 Defensive Intellectual Property: Technical Disclosure

For any technical approach that is not worth patenting but should not be patented by others, publish a technical white paper or open-source the implementation. This creates prior art that blocks third-party patent claims on those approaches, protecting QNC's freedom to operate.

---

## 10. Risks and Mitigations

### 10.1 Fast-Track Specific Risks

| ID | Risk | Prob | Impact | Mitigation |
| --- | --- | --- | --- | --- |
| FT-R01 | Hilscher netX 90 single-source dependency | Med | High | Qualify ASIX AX58101 as EtherCAT-only fallback SKU; keep AM64x PRU-ICSS as emergency EtherNet/IP fallback; architecture portability via cifX abstraction layer |
| FT-R02 | Hilscher licensing cost higher than budgeted | Low | Med | Confirm commercial terms in Week 1–2 of Phase 0; typical volume pricing €5–15/unit — acceptable vs €150–450K certification alternative |
| FT-R03 | cifX SPI bandwidth insufficient at high device count or high cycle rate | Low | Med | SPI at 50 MHz supports \~800 bytes/ms — sufficient for 8-device gateway; PCIe upgrade path available on AM64x without PCB change |
| FT-R04 | AM62x insufficient for concurrent northbound + protocol load | Low | Med | AM62x quad A53 handles gateway workload comfortably per TI reference profiles; upgrade to AM64x is a schematic-compatible swap if headroom \<20% at 8 devices |
| FT-R05 | `ethercat_driver_ros2` integration harder than expected | Med | Med | ADLINK/Panasonic case study is production-validated; allocate 2-week integration spike in Phase 0; cifX API is documented fallback |
| FT-R06 | Data Bus process image performance under high point count or update rate | Med | Med | Stress test at 1,000 points × 1 kHz during Phase 1 Month 3; if lock contention is problematic, adopt lock-free ring buffer pattern for high-frequency data points |
| FT-R07 | Profile authoring bottleneck: MAiRA/LARA EOAT profiles not ready for MVPpilot | Med | Med | Seed 2 profiles in Month 2 (gripper + force-torque); assign one profile author from device integration team; DPS v2.0 YAML skeleton template accelerates authoring |
| FT-R08 | Sales volume does not reach neutral scenario (500 units/Year 1) | Med | High | Lock 2–3 pilot commitments in Phase 0 before full investment; conservative scenario still reaches near-break-even; pilot customer feedback directly improves product-market fit |
| FT-R09 | netX 90 firmware swap between PROFINET and EtherCAT requires re-submission for CE/FCC | Low | High | Firmware swap does not change hardware RF characteristics; CE/FCC covers the device; Hilscher pre-certified stacks carry their own protocol conformance evidence; confirm interpretation with test lab in Phase 1 |

### 10.2 Retained Risks (From v3.0, Still Applicable)

| ID | Risk | Mitigation |
| --- | --- | --- |
| R02 | Connector/enclosure undecided → PCB blocked | Lock M12 vs RJ45 decision in Week 3 |
| R09 | Canonical command namespace registry not seeded → profile authoring blocked | Namespace registry is Phase 0/1 deliverable; assign SW Arch owner in Week 1 |
| R12 | EMC pre-compliance failure | Book lab by Month 6; 2-week tuning window reserved |
| R16 | SCADA/MES scope creep in customer materials | PRD §2.2 non-goals gate; review before any external release |
| R17 | AM62x + MAX14819 + netX 90 long lead times | Advance order in Week 4 (8–12 week lead times) |

### 10.3 Risks Eliminated by Fast-Track Strategy

| Eliminated Risk | Why Eliminated |
| --- | --- |
| R5F real-time firmware complexity and IPC latency | R5F workstream removed; netX 90 handles real-time |
| PRU-ICSS EtherNet/IP bringup issues | PRU-ICSS EtherNet/IP path eliminated; netX 90 from Day 1 |
| IO-Link stack license delay from ADI | MAX14819 uses mainlined Linux kernel driver; no separate license |
| Protocol certification cost (€150–450K) | netX 90 stacks carry Hilscher's conformance evidence |
| PRU-ICSS pin-mux conflicts | PRU-ICSS not used for industrial Ethernet in fast-track |
| netX 90 Phase 2 SKU not confirmed | netX 90 is active in Phase 1; SKU confirmed at outset |
| Rule explosion in protocol conversion as protocols are added | Data Bus architecture avoids this; each new protocol is one adapter |

---

## 11. Expected Benefits

### 11.1 Time Savings

| Metric | v3.0 Roadmap | Fast-Track v2.0 | Saving |
| --- | --- | --- | --- |
| First credible software demo | Month 5–6 | **Month 2–3** | \~3 months |
| PROFINET + EtherCAT available | Month 8+ | **Month 6–7** (firmware swap) | \~2 months |
| GA launch | Month 11 | **Month 12** | Comparable |
| First ROS2 driver published | Not planned | **Month 16** | New capability |
| Cloud platform / SaaS model | Not in scope | **Month 16–18** | New revenue stream |

Hardware GA timing is comparable because PCB fabrication lead time (\~3–4 weeks) is the irreducible constraint. The fast-track gains are in software confidence, earlier protocol breadth, and substantially lower engineering effort.

### 11.2 Cost Reduction

| Cost Category | Previous Approach | Fast-Track v2.0 | Saving |
| --- | --- | --- | --- |
| Protocol conformance certification | €150–450K (custom stacks) | €0 (netX 90 pre-certified) | **€150–450K** |
| R5F firmware engineering | \~3–4 specialist FTE-months | Eliminated | **\~€60–120K** |
| PRU-ICSS EtherNet/IP engineering | \~2 FTE-months | Eliminated | **\~€40–80K** |
| IO-Link stack licensing | Uncertain + delay | ADI mainlined driver, zero cost | **€10–30K** |
| **Total estimated saving** |  |  | **€260–680K** |

netX 90 per-unit licensing (\~€5–15/unit at volume) is a known, manageable cost. The saving against custom certification is 20–50× the licensing cost.

### 11.3 Complexity and Risk Reduction

| Complexity Metric | v3.0 | Fast-Track v2.0 | Reduction |
| --- | --- | --- | --- |
| Real-time firmware development | Custom (R5F + PRU-ICSS) | None (netX 90) | Full elimination |
| Internal data model | Custom | Data Bus (well-known pattern) | Lower risk |
| Schematic sheets | 12 | **10** | −2 sheets |
| Hardware bring-up paths | A53 + R5F + PRU-ICSS + netX 90 | A53 + netX 90 | −2 paths |
| DVT protocol stacks to validate | 5 custom stacks | 0 custom (netX 90 tested) | −5 |
| External certification blockers (Phase 1–2) | 3 (PI, ODVA, ETG) | Deferred to Phase 3 | −3 critical-path blockers |
| Schedule confidence at GA | 60% expected | **75% expected** | +15% confidence |

---

## 12. Assumptions, Dependencies, and Trade-offs

### 12.1 Key Assumptions

| # | Assumption | If Wrong, Impact |
| --- | --- | --- |
| A1 | Hilscher netX 90 licensing terms acceptable (\~€5–15/unit volume) | If unacceptable: revert to PRU-ICSS + R5F for EtherNet/IP; +8 weeks schedule |
| A2 | cifX SPI at 50 MHz provides sufficient bandwidth for 8-device gateway process image | If insufficient: upgrade to PCIe on AM64x; PCB change required; \~4-week impact |
| A3 | TI AM62x quad A53 has sufficient headroom for QNC runtime + northbound + cifX | If insufficient: swap to AM64x; schematic-compatible; \~2-week rework |
| A4 | `ethercat_driver_ros2` can be adapted to QNC's semantic layer in ≤2 weeks | If harder: 4-week spike; does not affect core gateway MVP |
| A5 | MAX14819 mainlined Linux SPI driver works on AM62x BSP without modification | Highly likely; 1-week spike to confirm in Phase 0 |
| A6 | netX 90 firmware swap between protocols does not require CE/FCC re-submission | Confirm with test lab in Phase 1; low risk per Hilscher documentation |
| A7 | 2–3 pilot integrator commitments can be secured in Phase 0 | Critical commercial assumption; if not met, reassess investment scale before full NRE |
| A8 | Data Bus process image at 1,000 points × 1 kHz is feasible on AM62x with acceptable lock contention | Stress test in Phase 1 Month 3; lock-free ring buffer pattern available as fallback |

### 12.2 Hard Dependencies

| Dependency | Required By | Lead Time |
| --- | --- | --- |
| Hilscher netX 90 commercial agreement | Week 2 | 2–4 weeks to confirm; start Week 1 |
| TI AM62x evaluation board (SK-AM62B) | Week 1 | Available immediately from TI |
| Hilscher cifX driver source + SDK | Week 2 (after commercial agreement) | Free from Hilscher GitHub |
| MAX14819 production samples | Month 3 (EVT BOM) | 8–12 weeks — order at Week 4 |
| netX 90 production samples | Month 3 (EVT BOM) | 8–12 weeks — order at Week 4 |
| AM62x production samples | Month 3 (EVT BOM) | 8–12 weeks — order at Week 4 |
| 2–3 pilot integrator commitments | End of Phase 0 | Active outreach from Week 1 |
| MAiRA/LARA EOAT device access for profile authoring | Month 2 | Internal NEURA resource; PM confirms in Month 1 |
| EMS partner confirmed and NDA signed | Month 3 (before PCB fab order) | 4-week onboarding |

### 12.3 Trade-offs Accepted

| Trade-off | Gained | Sacrificed | Verdict |
| --- | --- | --- | --- |
| netX 90 active from Day 1 (not DNP) | Phase 1 protocol breadth; single clean bringup path | +€15–20/unit BOM | ✅ Accepted — eliminates PRU-ICSS workstream |
| AM62x instead of AM64x | Lower BOM cost; simpler PCB; faster schematic | No PRU-ICSS fallback path | ✅ Accepted — upgrade path available if needed |
| Data Bus over Protocol Converter | Future protocol additions are adapters not rule tables | Slightly higher RAM (1–4 MB vs 256–512 KB) | ✅ Accepted — scalability justifies the overhead |
| Defer OPC UA / MQTT to Phase 3 | Faster MVP delivery | Northbound extensions not at GA | ✅ Accepted — PRD §3.2 classifies these as extensions |
| Defer DDS Pack to Phase 3 | Faster MVP delivery | DDS Pack not at GA | ✅ Accepted — PRD §3.3 explicitly optional |
| Single API at MVP (split in Month 7) | Faster alpha delivery | Temporary violation of dual-API requirement | ⚠️ Acceptable for internal demo only; split before pilot |
| Utility model patents over deep patent strategy | Marketing credential; grant eligibility; copycat deterrence | Not a primary IP moat | ✅ Accepted — primary moat is ecosystem, not patents |

### 12.4 What Must NOT Be Deferred

These items are non-negotiable at MVP GA regardless of schedule pressure:

- ✅ `semantic_mapping` in every device profile (DPS v2.0 §7.1)
- ✅ Atomic config rollback (PRD REQ-CORE-030)
- ✅ `fault_domain` discrimination in every fault event (CSC §2.5)
- ✅ Exception bypass path for critical signals under stale process image
- ✅ Safe Mode entry ≤ 1 s (PRD NFR-004)
- ✅ YAML as authoritative artifact format (PRD §12); JSON is runtime export only
- ✅ TPM-signed profile activation for production profiles (DPS §15)
- ✅ Both OpenAPI documents split by pilot phase (not just by GA) — robot-facing + northbound (ICD §5.2)
- ✅ QNC is NOT a safety-rated device — stated explicitly in all product materials (PRD §13)
- ✅ Pilot customer commitments secured before full NRE investment (commercial prerequisite)

---

## Immediate Action List (First 30 Days)

| Day | Action | Owner |
| --- | --- | --- |
| **Day 1–3** | Procure TI SK-AM62B eval board and Hilscher netX 90 devkit | HW Lead |
| **Day 1–3** | Begin outreach to 5–8 robot system integrators on heterogeneous production line scenario | PM |
| **Day 1–5** | Contact Hilscher to initiate commercial licensing discussion | PM + Procurement |
| **Day 3–7** | Start cifX driver integration spike on AM62x eval board | SW Lead |
| **Day 3–7** | Start MAX14819 SPI driver spike on AM62x BSP | SW Lead |
| **Day 7–14** | Seed canonical command namespace registry (gripper, force-torque, proximity, valve, DIO, transport, management) | SW Arch |
| **Day 7–14** | Lock connector family decision (M12 vs RJ45) | Mech + PM |
| **Day 14–21** | Draft 2–3 pilot customer intent letters; follow up from initial outreach | PM |
| **Day 21–28** | Architecture freeze decision: AM62x confirmed; netX 90 active Day 1; Data Bus model; 6-layer PCB | PM + HW Lead + SW Lead |
| **Day 28** | Gate M0 review: all preconditions met → full development investment authorised | All stakeholders |

---
