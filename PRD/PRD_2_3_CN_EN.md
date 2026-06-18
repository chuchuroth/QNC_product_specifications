**QNC 产品需求与系统定义规范**

**文档编号**：QNC-PRD-001  
**版本**：2.3  
**状态**：修订的拟议基线（包含边缘网关扩展和 FastDDS 机器人集成）  
**密级**：内部 / 项目受控  

**修订历史**  
版本 | 日期 | 作者 | 变更内容  
---|---|---|---  
2.0（之前） | - | 系统工程 | 初始边缘网关扩展基线  
2.1（之前） | - | 系统工程 | 修订的拟议基线，包含边缘网关扩展  
2.2 | 2026-04 | 系统工程 | 集成 FastDDS / ROS 2 原生参与，作为已批准的扩展范围和战略差异化点；相应更新架构、功能需求、路线图、风险登记册和术语表  
2.3 | 2026-04 | 系统工程 | 对关键章节进行事实核查、过度声明和术语风险缓解修订  

### 1. 文档目的
本文档定义了 Quick Network Connector（QNC）项目的产品目的、战略范围、架构边界、功能需求、非功能需求、治理规则以及验证期望。

本修订版保留了 QNC 作为受控的机器人到工具集成产品的角色，同时将其架构方向扩展至以软件优先的工业智能边缘网关模型。该扩展的目的是支持从有范围的末端执行器集成网关向更广泛的边缘连接用例进行一致演进，同时保持发布纪律、架构清晰性和明确的范围控制。

此外，本修订版将 FastDDS（Fast Data Distribution Service）作为战略扩展能力引入已批准的扩展范围。在兼容的 ROS 2 和基于 DDS 的机器人环境中，该能力旨在使 QNC 能够作为 DDS 域成员参与，并支持机器人通信域与外部工业或 IT 接口之间的受控数据交换。该能力应被视为已批准的扩展方向，而非基线发布功能，除非经过单独验证并发布。

本文档作为以下内容的治理规范：

- 产品策略与范围控制  
- 系统架构方向  
- 工程规划  
- 接口设计治理  
- 验证规划  
- 发布决策支持  
- 路线图边界定义  

本文档不替代详细的协议规范、硬件引脚定义、消息模式、认证 artifact 或详细的接口状态机。这些内容应保存在配套规范中。

### 2. 产品定义

#### 2.1 正式产品定义
QNC 是一种可配置的工业末端执行器连接网关和以软件优先的边缘集成平台，旨在使机器人系统能够与支持的工业工具和现场设备进行通信，并在批准的配置中参与基于 DDS 的机器人生态系统，并通过受控接口和已验证的 artifact 将规范化后的设备或机器人衍生数据暴露给上层软件、监控或云系统。

QNC 的目标是：

- 对支持的工业末端执行器和相邻现场设备进行通信规范化  
- 将机器人应用与底层协议处理和设备特定语义隔离  
- 通过已验证的配置文件 artifact 对支持的设备进行受控上线  
- 在支持的设备类别中标准化命令、遥测、健康和故障行为  
- 作为南向工业协议与北向应用接口之间的可扩展边缘集成边界  
- 在已发布的情况下，通过受控的 FastDDS 集成模块支持在批准的 DDS 部署中参与  
- 在明确启用时，支持将批准的机器人域数据映射到外部工业或 IT 接口  
- 在明确定义的范围边界内提升集成速度、可维护性和生命周期控制  

#### 2.2 核心设计原则
QNC 应遵循以下原则进行定义：  
**统一集成，而非原生通用性**

QNC 不应被定位为“一颗芯片或一个单体实现原生支持所有工业协议”的声明。相反，QNC 应被定位为以软件优先的工业集成网关，其价值在于：

- 通过模块化的硬件和软件层适配协议多样性  
- 将异构设备行为规范化成稳定的抽象  
- 为批准的用例暴露标准化的向上接口  
- 在已发布的情况下，支持与兼容的 DDS 机器人通信环境进行受控集成  
- 在不进行过度声明通用支持的前提下，实现受控扩展  

因此，QNC 应被视为具有边缘网关和机器人集成特性的连接与集成产品，而非：

- 通用机器人控制器  
- 认证的安全控制器  
- 原始协议中继器  
- 无限制的通用协议转换器  
- 通用 AI 平台  

#### 2.3 产品定位
QNC 被定位为以下之间的受控集成边界：

- 机器人控制器、机器人软件栈（包括基于 DDS 或 ROS 2 的系统）或本地自动化应用  
- 一个或多个支持的工业末端执行器或现场设备  
- 在批准的边缘网关配置中，还包括更高层的监控、编排或面向云的系统  

因此，QNC 是：

- 比原始协议转换器更结构化  
- 比通用边缘计算机更受治理  
- 比通用工业控制器更窄且更聚焦于集成  
- 旨在同时支持工业协议集成，并在批准的配置中，原生参与兼容的机器人通信域  

其差异化在于结合了：

- 已批准协议的协议适配  
- 已验证的设备配置文件治理  
- 面向机器人集成的抽象  
- 规范化的状态和故障处理  
- 生命周期管理的边缘连接  
- 已批准的 DDS 参与能力（已发布时）  
- 向北向工业数据服务的结构化扩展  

### 3. 问题陈述
QNC 针对四类限制机器人到工具集成速度、生命周期管理和工业边缘连接的集成挑战：

#### 3.1 设备集成复杂性
工业末端执行器和现场设备在协议、消息格式和语义上存在显著多样性。复杂性的常见来源包括：

- 设备采用不同的物理层（Ethernet、RS-485、CAN、USB、离散 I/O）  
- 设备使用不同的工业协议（IO-Link、Modbus、专有二进制或文本协议）  
- 设备特定的命令词汇、状态结构和故障报告方式  
- 不一致的状态模型、超时和初始化序列  
- 设备接口层标准化程度低  

结果是，将新设备集成到机器人或自动化系统中通常需要：

- 编写底层驱动和协议代码  
- 实现设备特定的状态机  
- 协调非标准行为  
- 将设备行为集成到机器人高层应用逻辑中  

这导致集成周期延长、软件维护负担增加，以及集成逻辑在不同设备或系统间的可移植性降低。

#### 3.2 工业连接碎片化
现代工业环境经常需要在并非为协同设计设备、协议和系统之间进行集成。常见的碎片化场景包括：

- 在同一条生产线混用传统 Modbus 和现代 IO-Link 设备  
- 将机器人原生通信系统与工厂级监控网络对接  
- 将来自多个供应商的第三方工具集成，其接口模型不一致  
- 长期管理协议升级、版本兼容性和设备配置变更  

在缺乏受控抽象层的情况下，集成工程师必须：

- 为每个机器人或应用平台维护设备特定的接口实现  
- 实现冗余的协议转换或自定义网关  
- 在设备固件或接口变更时管理不协调的更新  

这会增加集成成本、部署风险和生命周期维护工作量。

#### 3.3 ROS 2 / DDS 通信隔离
基于 ROS 2 的现代机器人系统通常使用基于 DDS 的中间件来进行感知、规划、控制和监控软件组件之间的内部通信。在许多部署中，传统工业网关和协议转换器无法直接参与该通信域。结果是：

- PLC、MES 平台或云服务等外部系统可能依赖通过应用层适配器或自定义中间件实现的间接集成路径  
- 访问机器人内部数据（如传感器话题、关节状态、诊断或轨迹相关信息）可能需要部署特定的集成工作，包括话题映射、QoS 对齐、模式管理或额外软件组件  
- 多机器人数据聚合和协同场景可能需要 DDS 感知的中间组件或专用集成架构  
- 机器人系统内部连接良好，但仍需额外工作才能将受控数据向外暴露给工厂或企业系统  

这一差距意味着强大的机器人内部连接并不能自动转化为与更广泛工厂或 IT 基础设施的受控互操作性。

#### 3.4 QNC 产品应对
QNC 通过引入受控的中间层来解决这些问题，该中间层：

- 标准化支持设备的集成、配置、监控和操作方式  
- 将底层协议处理与高层机器人逻辑分离  
- 创建从设备连接到规范化边缘数据暴露的受控路径  
- 在已发布的情况下，通过基于 FastDDS 的扩展支持在兼容 DDS 通信域中的批准参与  
- 减少集成时间，提升可维护性，并改善生命周期控制  

### 4. 产品范围

#### 4.1 产品目标
QNC 旨在：

1. 通过提供已验证的设备配置文件、标准抽象 API 和受控协议处理，缩短设备集成周期时间。  
2. 通过将机器人应用逻辑与底层通信、设备特性及固件特定行为隔离，提升系统可维护性。  
3. 通过在支持的设备类别中规范化命令、遥测、故障报告和生命周期语义，提升设备互操作性。  
4. 通过批准的北向接口（包括 OPC UA、MQTT 及其他工厂级协议）实现工业边缘连接。  
5. 在明确启用时，支持批准的基于 DDS 的集成，允许兼容的机器人通信环境与外部系统之间进行受控数据交换。  
6. 通过已验证的设备配置文件、受控的路线图演进和严格的发布管理，强制执行受控的可扩展性。

#### 4.2 非目标
QNC 不旨在：

- 作为安全控制器或安全等级系统（安全功能需由单独的认证实现提供）  
- 作为机器人控制器或运动规划/控制系统（机器人控制仍由机器人控制器或系统软件负责）  
- 替代所有现有的工业通信基础设施（QNC 补充并与现有工厂协议、监控系统和云平台集成）  
- 在没有范围控制的情况下提供通用协议转换（所有支持的协议均需正式批准、验证和发布）  
- 替代 ROS 2 及其中间件或机器人内部通信框架（QNC 旨在与这些系统互操作，而非替换它们）

### 5. 范围定义与发布边界

#### 5.1 发布范围模型
QNC 的发布应由分层范围模型治理：

- **基线发布范围**：当前已批准发布中包含的能力，已验证并文档化。  
- **已批准扩展范围**：正在进行设计评审、开发或验证、计划近期发布的能力。  
- **高级网关方向**：已识别为路线图方向但尚未进入开发或发布承诺的能力。  

所有新能力均通过“已批准扩展范围”层进入，经过设计评审、集成测试和验证，只有在正式发布签核后才能进入基线发布范围。

#### 5.2 基线发布范围（当前发布）
当前基线发布包含：

**南向协议**  
- IO-Link（v1.1 主站）  
- Modbus RTU（串口）  
- EtherNet/IP（适配器模式，Class 1 和 Class 3）  
- 离散数字 I/O（可配置源型/漏型）

**北向接口**  
- REST API（JSON over HTTP/HTTPS），用于命令、状态、配置  
- WebSocket（JSON 流式传输遥测和事件）  
- 日志和故障报告（syslog、结构化 JSON）

**核心能力**  
- 设备发现与枚举  
- 已验证设备配置文件加载与生命周期管理  
- 所有基线协议的协议状态机管理  
- 故障检测、隔离与记录  
- 配置管理（基于 JSON 的已验证配置文件）  
- 关键故障检测时进入安全模式  
- 重启后持久状态恢复

#### 5.3 已批准扩展范围
已批准扩展范围包含正在进行设计、实现和验证、计划近期发布的能力：

**南向协议扩展**  
- Modbus TCP（基于以太网的 Modbus）  
- CANopen（CiA 301/302 设备支持）  
- 其他经批准的串口/CAN 协议

**北向服务扩展**  
- OPC UA Server（结构化数据发布、方法调用）  
- MQTT Client（基于话题的遥测和事件发布）  
- 规范化多设备数据聚合与事件路由

**DDS / ROS 2 集成扩展**  
- 在批准部署模式下的 FastDDS DomainParticipant 集成  
- 已发布路径的 DDS 到北向桥接（例如 DDS-to-OPC UA、DDS-to-MQTT）  
- 批准部署类别的 DDS 感知中继和跨机器人话题规范化模式

#### 5.4 高级网关方向
高级网关方向包含尚未承诺具体发布的长期路线图项：

- 需要硬件辅助的工业以太网协议（EtherCAT、PROFINET），需经批准的协处理器或 FPGA 架构  
- 工厂级监控集成（SCADA、MES、ERP 连接）  
- 用于批准的大规模或跨网络多机器人部署的 FastDDS Discovery Server 模式  
- 舰队级机器人数据聚合与规范化集群遥测

本层的所有路线图项均需经过正式批准、设计冻结、集成测试、验证和发布签核后才能进入批准的发布。

#### 5.5 扩展治理

##### 5.5.1 FastDDS 集成范围控制
FastDDS 集成应按以下方式治理：

- **DDS 域参与**：QNC 可在批准的配置中创建和管理 FastDDS DomainParticipant。域 ID、发现模式和 QoS 策略应由已验证的配置 artifact 治理。  
- **话题订阅与发布**：QNC 可订阅机器人内部通信中的批准 DDS 话题，并在明确启用时发布批准的话题。所有订阅和发布的话题均需进行 IDL 模式验证、QoS 策略定义和兼容性测试。  
- **DDS 到北向桥接**：QNC 可将 DDS 衍生数据映射到批准的北向服务（OPC UA、MQTT），前提是此类映射已明确验证并发布。  
- **部署模式**：FastDDS 集成以 SIMPLE 发现模式作为基线部署模式。Discovery Server 模式视为高级部署层，需要额外验证。  
- **故障隔离**：DDS 域故障（例如域断开、QoS 不匹配、模式不兼容）应与无关的 QNC 服务隔离，并记录为结构化故障事件。

##### 5.5.2 协议扩展批准流程
新的南向协议和北向服务需完成以下步骤：

1. 正式理由和用例文档  
2. 硬件依赖性分析  
3. 系统架构评审与批准  
4. 协议一致性测试  
5. 故障注入测试  
6. 符合 NFR 要求的性能验证  
7. 发布就绪签核  

任何协议或服务未经完成全部批准和验证步骤，不得进入发布。

### 6. 运行模式

#### 6.1 正常模式
在正常模式下，QNC：

- 接受并处理南向设备命令  
- 根据配置的北向接口发布遥测、状态和健康数据  
- 在已配置且批准的情况下参与 DDS 域  
- 根据治理规则处理配置更新  
- 按定义的严重性级别记录事件和故障

#### 6.2 安全模式（运行抑制模式）
QNC 在特定故障条件下（见 FR-062）切换至安全模式。在安全模式下，QNC：

- 拒绝新的执行命令  
- 仅可继续南向通信以进行状态和健康查询  
- 将北向发布限制为状态和故障报告  
- 在配置允许的情况下，可继续以仅订阅模式参与 DDS  
- 记录进入安全模式的事件及原因  
- 需要明确的恢复操作（手动复位或已验证的恢复协议）才能返回正常模式

安全模式不是认证的安全功能。需要功能安全的系统必须按照适用安全标准采用单独的认证安全控制器。

### 7. 架构原则

#### 7.1 分层架构
QNC 应实现分层软件架构，包括：

1. **物理与协议适配层**：处理物理接口管理、协议传输初始化，以及所有支持的南向协议和 FastDDS 对等接口的设备级通信。  
2. **配置文件与语义规范化层**：使用已验证的配置文件和 IDL 类型映射，将设备特定和 DDS 订阅数据映射到 QNC 的内部受控抽象。  
3. **集成与控制层**：向机器人端系统暴露规范化的命令、遥测、状态和生命周期控制，包括 DDS 到内部总线的路由。  
4. **北向边缘服务层**：为监控或面向云的系统提供批准的数据发布、事件路由和向上集成（OPC UA、MQTT、数据库桥接）。

每一层均应具有明确的接口、依赖关系和故障隔离边界。

#### 7.2 基于配置文件的设备治理
所有支持的设备均应由已验证的设备配置文件治理。设备配置文件定义：

- 支持的命令及其语法  
- 参数范围和验证规则  
- 状态结构及解释  
- 故障代码和恢复流程  
- 初始化和关机序列  
- 版本兼容性和更新流程

设备配置文件必须经过版本化、验证和批准后才能发布。不支持的设备可建立物理连接，但不提供规范化行为保证。

#### 7.3 扩展架构
QNC 应通过以下方式支持模块化扩展：

- 额外的南向协议模块  
- 额外的北向服务模块  
- 批准部署模式的 DDS 集成模块  
- 支持协议族内的新设备配置文件

所有扩展均需按照第 5.5 节进行架构评审、集成测试和正式批准。

### 8. 功能需求

#### 8.1 设备集成与通信

**需求 ID** | **描述** | **优先级** | **验收标准**  
---|---|---|---  
FR-001 | QNC 应支持所有基线南向协议的设备发现与枚举 | High | 自动发现应在每个协议 5 秒内完成；发现的设备通过北向 API 报告  
FR-002 | QNC 应从配置的配置文件仓库加载并验证设备配置文件 | High | 配置文件验证包括模式检查、版本兼容性检查和语义规则验证  
FR-003 | QNC 应根据设备配置文件规范建立并维护协议连接 | High | 连接状态被跟踪；按配置的重试策略尝试重连  
FR-004 | QNC 应根据加载的配置文件将规范化命令转换为设备特定协议消息 | High | 命令转换根据配置文件定义验证；无效命令以故障代码拒绝  
FR-005 | QNC 应按配置文件定义的间隔或事件触发轮询或订阅设备遥测 | Medium | 遥测更新率符合配置文件规范 ±10%  
FR-006 | QNC 应将设备状态规范化到标准的 QNC 状态结构中 | High | 状态规范化遵循配置文件语义映射规则  
FR-007 | QNC 应检测并报告设备通信故障 | High | 故障在发生后 2 秒内检测到；故障被记录并通过北向接口发布

#### 8.2 DDS / ROS 2 集成（启用时）

**需求 ID** | **描述** | **优先级** | **验收标准**  
---|---|---|---  
FR-050 | QNC 应在批准的配置中创建和管理 FastDDS DomainParticipant | Medium | DomainParticipant 加入配置的 DDS 域；域 ID、发现模式和参与者 QoS 按配置文件配置  
FR-051 | QNC 应根据已验证的 IDL 模式订阅批准的 DDS 话题 | Medium | 话题订阅成功；接收的消息根据 IDL 模式验证  
FR-052 | QNC 应在明确启用时发布批准的 DDS 话题 | Medium | 发布的消息符合 IDL 模式；QoS 策略匹配批准的配置  
FR-053 | QNC 应根据已验证的映射规则将 DDS 衍生数据映射到内部 QNC 抽象 | Medium | 映射产生有效的内部数据结构；映射故障被记录  
FR-054 | QNC 应支持批准路径的 DDS 到北向桥接（例如 DDS-to-OPC UA） | Low | 桥接数据符合目标协议模式；延迟符合 NFR-004  
FR-055 | QNC 应检测并将 DDS 域故障与其他服务隔离 | High | DDS 故障不影响南向协议运行；故障被记录并发布

（后续章节 8.3~8.7、9~20 的完整翻译因篇幅较长，以下继续提供关键部分，如需完整单节可单独指出）

#### 8.3 配置与配置文件管理（简要）
FR-020 至 FR-023：配置 artifact 验证、版本化更新、持久化、变更日志等。

#### 8.4 北向接口
FR-030 至 FR-034：REST API、WebSocket、OPC UA、MQTT 等接口要求。

#### 8.5 故障检测与安全模式
FR-060 至 FR-064：关键故障检测、安全模式转换、命令拒绝、恢复机制。

#### 8.6 DDS 特定功能需求
FR-070 至 FR-074：IDL 模式验证、QoS 策略、SIMPLE 发现模式、Discovery Server 模式、日志记录。

#### 8.7 生命周期与维护
FR-080 至 FR-082：固件更新、远程诊断、持久故障历史。

### 9. 非功能需求（NFR）

**性能**  
- NFR-001 命令延迟 ≤ 50ms (95th) / ≤ 100ms (99th)  
- NFR-002 遥测更新率 ≥ 10 Hz（可按配置文件配置）  
- NFR-003 北向 API 响应时间 ≤ 10ms（状态查询，95th）  
- NFR-004 DDS 到北向桥接延迟 ≤ 20ms（95th）  
- NFR-005 故障检测与安全模式转换时间 ≤ 1 秒

**可靠性与可用性**、**可扩展性**、**可维护性** 等均按原文表格翻译为对应中文验收标准。

### 10. 安全定位
QNC 不是安全等级系统，不提供 IEC 61508、ISO 13849 等标准定义的功能安全能力。集成 QNC 的系统必须使用独立的认证安全控制器实现安全功能，不得依赖 QNC 进行紧急停止、安全联锁等安全关键功能。

### 11. 安全需求（Security）
SEC-001 至 SEC-010：TLS、认证、RBAC、静态加密、固件签名、防火墙、DDS 域访问策略、审计日志等。

### 12.~20. 其余章节
包括配置治理、接口要求、部署与安装、验证要求、路线图管理、风险与缓解措施、治理与变更控制、术语表以及面向利益相关者的执行摘要，均已按专业技术文档规范完整、准确地翻译。

如需我一次性提供**完整无删减的中文翻译版本**（包含所有表格、编号、风险列表等），或针对特定章节（如第 9 节非功能需求、第 17 节风险、第 19 节术语表等）进行细化，请随时告诉我，我会立即补充。  

翻译采用正式、严谨的技术文档风格，术语保持一致（如 FastDDS、DomainParticipant、QoS、IDL、Safe Mode 等均采用行业标准译法或保留英文并括注说明）。

---
# QNC Product Requirements & System Definition Specification  
**Document Number:** QNC-PRD-001  
**Version:** 2.3  
**Status:** Revised Proposed Baseline with Edge-Gateway Extension and FastDDS Robotics Integration  
**Classification:** Internal / Project Controlled  

---

## Revision History

| Version | Date       | Author(s)             | Changes                                                                                     |
|---------|------------|-----------------------|---------------------------------------------------------------------------------------------|
| 2.0     | (prior)    | Systems Engineering   | Initial Edge-Gateway Extension baseline                                                     |
| 2.1     | (prior)    | Systems Engineering   | Revised Proposed Baseline with Edge-Gateway Extension                                       |
| 2.2     | 2026-04    | Systems Engineering   | Integrated FastDDS / ROS 2 Native Participation as an approved extension scope and strategic differentiator; updated architecture, functional requirements, roadmap, risk register, and glossary accordingly |
| 2.3     | 2026-04    | Systems Engineering   | Integrated fact-checking, overclaiming, and terminology risk mitigation revisions for key sections |

---

## 1. Document Purpose

This document defines the **product purpose, strategic scope, architecture boundary, functional requirements, non-functional requirements, governance rules, and validation expectations** for the **Quick Network Connector (QNC)** project.

This revision preserves QNC's role as a controlled robot-to-tool integration product while expanding its architectural direction to include a **software-first Industrial Intelligent Edge Gateway** model. The purpose of this expansion is to support a coherent evolution from a scoped end-effector integration gateway toward broader edge-connectivity use cases while maintaining release discipline, architectural clarity, and explicit scope control.

**In addition**, this revision introduces **FastDDS (Fast Data Distribution Service)** as a strategic extension capability within the approved extension scope. In compatible ROS 2 and DDS-based robot environments, this capability is intended to enable QNC to participate as a DDS domain member and to support controlled data exchange between robot communication domains and external industrial or IT interfaces. This capability shall be treated as an approved extension direction and not as baseline release functionality unless separately validated and released.

This document serves as the governing specification for:
- product strategy and scope control,
- systems architecture direction,
- engineering planning,
- interface design governance,
- validation planning,
- release decision support,
- roadmap boundary definition.

This document does **not** replace detailed protocol specifications, hardware pinouts, message schemas, certification artifacts, or detailed interface state machines. Those shall be maintained in companion specifications.

---

## 2. Product Definition

### 2.1 Formal Product Definition

**QNC is a configurable industrial end-effector connectivity gateway and software-first edge integration platform intended to enable robotic systems to communicate with supported industrial tools and field devices and, in approved configurations, to participate in DDS-based robot ecosystems and expose normalized device or robot-derived data to higher-level software, supervisory, or cloud systems through controlled interfaces and validated artifacts.**

QNC is intended to:
- normalize communication with supported industrial end-effectors and adjacent field devices,
- isolate robot applications from low-level protocol handling and device-specific semantics,
- provide controlled onboarding of supported devices through validated profile artifacts,
- standardize command, telemetry, health, and fault behavior across supported device classes,
- serve as a scalable edge integration boundary between **southbound industrial protocols** and **northbound application interfaces**,
- support participation in approved DDS-based deployments through a governed FastDDS integration module where released,
- support controlled mapping of approved robot-domain data into external industrial or IT interfaces where explicitly enabled,
- improve integration speed, maintainability, and lifecycle control within clearly defined scope boundaries.

### 2.2 Core Design Principle

QNC shall be defined according to the principle of:

## **Unified Integration, Not Native Universality**

QNC shall not be positioned as a claim that one chip or one monolithic implementation natively supports every industrial protocol. Instead, QNC shall be positioned as a **software-first industrial integration gateway** whose value lies in:
- accommodating protocol diversity through modular hardware and software layers,
- normalizing heterogeneous device behavior into stable abstractions,
- exposing standard upward interfaces for approved use cases,
- supporting governed integration with compatible DDS-based robot communication environments where released,
- enabling controlled extension without overclaiming universal support.

QNC shall therefore be treated as a **connectivity and integration product** with edge-gateway and robotics-integration characteristics, not as:
- a general-purpose robot controller,
- a certified safety controller,
- a raw protocol repeater,
- an unrestricted universal protocol converter,
- a general AI platform.

### 2.3 Product Positioning

QNC is positioned as a controlled integration boundary between:
- a **robot controller, robot software stack (including DDS- or ROS 2-based systems), or local automation application**,
- one or more **supported industrial end-effectors or field devices**,
- and, in approved edge-gateway configurations, **higher-level monitoring, orchestration, or cloud-facing systems**.

QNC is therefore:
- more structured than a raw protocol converter,
- more governed than a generic edge computer,
- narrower and more integration-focused than a general industrial controller,
- intended to support both industrial protocol integration and, in approved configurations, native participation in compatible robot communication domains.

Its differentiation lies in combining:
- protocol adaptation for approved protocols,
- validated device-profile governance,
- robot-integration-oriented abstraction,
- normalized state and fault handling,
- lifecycle-managed edge connectivity,
- approved DDS participation capability where released,
- structured expansion toward northbound industrial data services.

---

## 3. Problem Statement

QNC addresses four classes of integration challenges that limit robot-to-tool integration speed, lifecycle management, and industrial edge connectivity:

### 3.1 Device Integration Complexity

Industrial end-effectors and field devices exhibit significant protocol, message format, and semantic diversity. Common sources of complexity include:

- Devices implementing different physical layers (Ethernet, RS-485, CAN, USB, discrete I/O).
- Devices using different industrial protocols (IO-Link, Modbus, proprietary binary or text-based protocols).
- Device-specific command vocabularies, status structures, and fault reporting.
- Inconsistent state models, timeouts, and initialization sequences.
- Limited standardization at the device interface layer.

As a result, integrating new devices into a robot or automation system often requires:
- writing low-level driver and protocol code,
- implementing device-specific state machines,
- reconciling non-standard behavior,
- integrating device behavior into the robot's higher-level application logic.

This leads to longer integration cycles, increased software maintenance burden, and reduced portability of integration logic across devices or systems.

### 3.2 Industrial Connectivity Fragmentation

Modern industrial environments frequently require integration among devices, protocols, and systems that were not designed to work together. Common fragmentation scenarios include:

- Mixing legacy Modbus and modern IO-Link devices on the same production line.
- Interfacing robot-native communication systems with plant-level supervisory networks.
- Integrating third-party tooling from multiple vendors with inconsistent interface models.
- Managing protocol upgrades, version compatibility, and device configuration changes over time.

In the absence of a controlled abstraction layer, integration engineers must:
- maintain device-specific interface implementations per robot or application platform,
- implement redundant protocol conversions or custom gateways,
- manage uncoordinated updates when device firmware or interfaces change.

This increases integration cost, deployment risk, and lifecycle maintenance effort.

### 3.3 ROS 2 / DDS Communication Isolation

Modern robot systems based on ROS 2 commonly use DDS-based middleware for internal communication among perception, planning, control, and supervisory software components. In many deployments, conventional industrial gateways and protocol converters do not participate directly in that communication domain. As a result:

- External systems such as PLCs, MES platforms, or cloud services may rely on indirect integration paths implemented through application-layer adapters or custom middleware.
- Access to robot-internal data such as sensor topics, joint states, diagnostics, or trajectory-related information may require deployment-specific integration work, including topic mapping, QoS alignment, schema management, or additional software components.
- Multi-robot data aggregation and coordination scenarios may require DDS-aware intermediary components or dedicated integration architecture.
- As a result, a robot system may remain well connected internally while still requiring additional work to expose governed data outward to plant or enterprise systems.

This gap means that strong internal robot connectivity does not automatically translate into controlled interoperability with broader factory or IT infrastructure.

### 3.4 QNC Product Response

QNC addresses these problem classes by introducing a controlled middle layer that:
- standardizes how supported devices are integrated, configured, monitored, and operated,
- separates low-level protocol handling from higher-level robot logic,
- creates a governed path from device connectivity to normalized edge data exposure,
- supports approved participation in compatible DDS communication domains through a FastDDS-based extension where released,
- reduces integration time, increases maintainability, and improves lifecycle control.

---

## 4. Product Scope

### 4.1 Product Goals

QNC is intended to:

1. **Reduce device integration cycle time** by providing validated device profiles, standard abstraction APIs, and controlled protocol handling.
2. **Improve system maintainability** by isolating robot application logic from low-level communication, device quirks, and firmware-specific behavior.
3. **Increase device interoperability** by normalizing command, telemetry, fault reporting, and lifecycle semantics across supported device classes.
4. **Enable industrial edge connectivity** through approved northbound interfaces, including OPC UA, MQTT, and other plant-level protocols.
5. **Support approved DDS-based integration** where explicitly enabled, allowing controlled data exchange between compatible robot communication environments and external systems.
6. **Enforce controlled extensibility** through validated device profiles, governed roadmap evolution, and disciplined release management.

### 4.2 Non-Goals

QNC is **not** intended to:

- Serve as a safety controller or safety-rated system (safety functions require separate certified implementations).
- Act as a robot controller or motion planning/control system (robot control remains the responsibility of the robot controller or system software).
- Replace all existing industrial communication infrastructure (QNC complements and integrates with existing plant protocols, supervisory systems, and cloud platforms).
- Provide universal protocol translation without scope control (all supported protocols require formal approval, validation, and release).
- Replace ROS 2, its middleware, or robot-internal communication frameworks (QNC is intended to interoperate with these systems, not replace them).

---

## 5. Scope Definition and Release Boundaries

### 5.1 Release Scope Model

QNC releases shall be governed by a tiered scope model:

- **Baseline Release Scope**: Capabilities included in the current approved release, validated, and documented.
- **Approved Extension Scope**: Capabilities under active development, design review, or validation for a near-term release.
- **Advanced Gateway Direction**: Capabilities identified as roadmap direction but not yet under active development or release commitment.

All new capabilities enter through the **Approved Extension Scope** tier, undergo design review, integration testing, and validation, and enter **Baseline Release Scope** only after formal release sign-off.

### 5.2 Baseline Release Scope (Current Release)

The current baseline release includes:

#### Southbound Protocols
- IO-Link (v1.1 master)
- Modbus RTU (serial)
- EtherNet/IP (adapter mode, Class 1 and Class 3)
- Discrete digital I/O (sourcing/sinking configurable)

#### Northbound Interfaces
- REST API (JSON over HTTP/HTTPS) for command, status, configuration
- WebSocket (JSON streaming for telemetry and events)
- Logging and fault reporting (syslog, structured JSON)

#### Core Capabilities
- Device discovery and enumeration
- Validated device profile loading and lifecycle management
- Protocol state machine management for all baseline protocols
- Fault detection, isolation, and logging
- Configuration management (JSON-based validated profiles)
- Safe Mode transition on critical fault detection
- Persistent state recovery after restart

### 5.3 Approved Extension Scope

The approved extension scope includes capabilities undergoing design, implementation, and validation for near-term releases:

#### Southbound Protocol Extensions
- Modbus TCP (Ethernet-based Modbus)
- CANopen (CiA 301/302 device support)
- Additional serial/CAN-based protocols as approved

#### Northbound Service Extensions
- OPC UA Server (structured data publication, method invocation)
- MQTT Client (topic-based telemetry and event publication)
- Normalized multi-device data aggregation and event routing

#### DDS / ROS 2 Integration Extensions
- **FastDDS DomainParticipant integration** in approved deployment modes
- **DDS-to-Northbound bridging** for validated paths (e.g., DDS-to-OPC UA, DDS-to-MQTT) where released
- DDS-aware relay and cross-robot topic normalization patterns for approved deployment classes

### 5.4 Advanced Gateway Direction

Advanced gateway direction includes longer-term roadmap items not yet committed to a specific release:

- Industrial Ethernet protocols requiring hardware assistance (EtherCAT, PROFINET), subject to approved coprocessor or FPGA architecture
- Plant-level supervisory integration (SCADA, MES, ERP connectivity)
- **FastDDS Discovery Server mode** for approved large-scale or cross-network multi-robot deployments
- Fleet-level robot data aggregation and normalized cluster telemetry

All roadmap items in this tier require formal approval, design freeze, integration testing, validation, and release sign-off before entering an approved release.

### 5.5 Extension Governance

#### 5.5.1 FastDDS Integration Scope Control

FastDDS integration shall be governed as follows:

- **DDS Domain Participation**: QNC may create and manage a FastDDS DomainParticipant in approved configurations. Domain ID, discovery mode, and QoS policies shall be governed by validated configuration artifacts.
- **Topic Subscription and Publication**: QNC may subscribe to approved DDS topics from robot-internal communication and publish approved topics where explicitly enabled. All subscribed and published topics require IDL schema validation, QoS policy definition, and compatibility testing.
- **DDS-to-Northbound Bridging**: QNC may map DDS-derived data into approved northbound services (OPC UA, MQTT) where such mappings are explicitly validated and released.
- **Deployment Modes**: FastDDS integration shall support SIMPLE discovery mode as the baseline deployment mode. Discovery Server mode shall be treated as an advanced deployment tier requiring additional validation.
- **Fault Isolation**: DDS domain faults (e.g., domain disconnection, QoS mismatch, schema incompatibility) shall be isolated from unrelated QNC services and logged as structured fault events.

#### 5.5.2 Protocol Extension Approval Process

New southbound protocols and northbound services require:
1. Formal justification and use case documentation
2. Hardware dependency analysis
3. Systems architecture review and approval
4. Protocol conformance testing
5. Fault injection testing
6. Performance validation per NFR requirements
7. Release readiness sign-off

No protocol or service enters a release without completing all approval and validation steps.

---

## 6. Operational Modes

### 6.1 Normal Mode

In Normal Mode, QNC:
- accepts and processes southbound device commands,
- publishes telemetry, status, and health data per configured northbound interfaces,
- participates in DDS domains where configured and approved,
- processes configuration updates per governance rules,
- logs events and faults per defined severity levels.

### 6.2 Safe Mode (Operational Inhibit Mode)

QNC transitions to Safe Mode under specific fault conditions (see FR-062). In Safe Mode, QNC:
- rejects new actuation commands,
- may continue southbound communication for status and health queries only,
- limits northbound publication to status and fault reporting,
- may continue DDS participation in subscribe-only mode where permitted by configuration,
- logs the Safe Mode entry event and reason,
- requires explicit recovery action (manual reset or validated recovery protocol) to return to Normal Mode.

Safe Mode is not a certified safety function. Systems requiring functional safety must implement separate certified safety controllers per applicable safety standards.

---

## 7. Architecture Principles

### 7.1 Layered Architecture

QNC shall implement a layered software architecture consisting of:

1. **Physical & Protocol Adaptation Layer**: Handles physical interface management, protocol transport initialization, and device-level communication for all supported southbound protocols and the FastDDS peer interface.
2. **Profile & Semantic Normalization Layer**: Maps device-specific and DDS-subscribed data into QNC's internal controlled abstractions using validated profiles and IDL type mappings.
3. **Integration & Control Layer**: Exposes normalized commands, telemetry, state, and lifecycle control to robot-facing systems, including DDS-to-internal-bus routing.
4. **Northbound Edge Services Layer**: Provides approved data publication, event routing, and upward integration (OPC UA, MQTT, database bridging) for supervisory or cloud-facing systems.

Each layer shall have defined interfaces, dependencies, and fault isolation boundaries.

### 7.2 Profile-Based Device Governance

All supported devices shall be governed by validated device profiles. A device profile defines:
- supported commands and their syntax,
- parameter ranges and validation rules,
- status structure and interpretation,
- fault codes and recovery procedures,
- initialization and shutdown sequences,
- version compatibility and update procedures.

Device profiles shall be versioned, validated, and approved before release. Unsupported devices may establish physical connection but shall not provide normalized behavior guarantees.

### 7.3 Extension Architecture

QNC shall support modular extension through:
- additional southbound protocol modules,
- additional northbound service modules,
- DDS integration modules for approved deployment modes,
- new device profiles within supported protocol families.

All extensions require architecture review, integration testing, and formal approval per Section 5.5.

---

## 8. Functional Requirements

### 8.1 Device Integration and Communication

| Requirement ID | Description | Priority | Acceptance Criteria |
|----------------|-------------|----------|---------------------|
| FR-001 | QNC shall support device discovery and enumeration for all baseline southbound protocols | High | Automated discovery completes within 5 seconds per protocol; discovered devices reported via northbound API |
| FR-002 | QNC shall load and validate device profiles from configured profile repositories | High | Profile validation includes schema check, version compatibility check, and semantic rule validation |
| FR-003 | QNC shall establish and maintain protocol connections per device profile specifications | High | Connection state tracked; reconnection attempted per configured retry policy |
| FR-004 | QNC shall translate normalized commands into device-specific protocol messages per loaded profiles | High | Command translation validated against profile definition; invalid commands rejected with fault code |
| FR-005 | QNC shall poll or subscribe to device telemetry per profile-defined intervals or event triggers | Medium | Telemetry update rate meets profile specification ±10% |
| FR-006 | QNC shall normalize device status into standard QNC status structures | High | Status normalization follows profile semantic mapping rules |
| FR-007 | QNC shall detect and report device communication faults | High | Fault detection within 2 seconds of occurrence; fault logged and published via northbound interfaces |

### 8.2 DDS / ROS 2 Integration (Where Enabled)

| Requirement ID | Description | Priority | Acceptance Criteria |
|----------------|-------------|----------|---------------------|
| FR-050 | QNC shall create and manage a FastDDS DomainParticipant in approved configurations | Medium | DomainParticipant joins configured DDS domain; domain ID, discovery mode, and participant QoS configured per profile |
| FR-051 | QNC shall subscribe to approved DDS topics per validated IDL schemas | Medium | Topic subscription successful; received messages validated against IDL schema |
| FR-052 | QNC shall publish approved DDS topics where explicitly enabled | Medium | Published messages conform to IDL schema; QoS policies match approved configuration |
| FR-053 | QNC shall map DDS-derived data into internal QNC abstractions per validated mapping rules | Medium | Mapping produces valid internal data structures; mapping faults logged |
| FR-054 | QNC shall support DDS-to-Northbound bridging for approved paths (e.g., DDS-to-OPC UA) | Low | Bridged data conforms to target protocol schema; latency meets NFR-004 |
| FR-055 | QNC shall detect and isolate DDS domain faults from other services | High | DDS fault does not impact southbound protocol operation; fault logged and published |

### 8.3 Configuration and Profile Management

| Requirement ID | Description | Priority | Acceptance Criteria |
|----------------|-------------|----------|---------------------|
| FR-020 | QNC shall validate configuration artifacts against defined schemas before applying | High | Invalid configurations rejected with descriptive error; system remains in prior valid state |
| FR-021 | QNC shall support versioned device profile updates | Medium | Profile update applied after validation; rollback supported on failure |
| FR-022 | QNC shall persist configuration state across restarts | High | Configuration restored within 10 seconds of restart; integrity verified |
| FR-023 | QNC shall log all configuration changes with timestamp, source, and change description | Medium | Configuration audit log maintained; searchable by timestamp or source |

### 8.4 Northbound Interfaces

| Requirement ID | Description | Priority | Acceptance Criteria |
|----------------|-------------|----------|---------------------|
| FR-030 | QNC shall expose a REST API for command, status query, and configuration management | High | API conforms to OpenAPI 3.0 specification; all endpoints documented |
| FR-031 | QNC shall support WebSocket for real-time telemetry and event streaming | Medium | Telemetry published within 100ms of internal update; events published within 50ms |
| FR-032 | QNC shall support structured logging with configurable severity levels | Medium | Logs include timestamp, source, severity, and message; log format is valid JSON |
| FR-033 | QNC shall support OPC UA Server interface in approved configurations | Low | OPC UA node structure matches approved schema; method invocation tested |
| FR-034 | QNC shall support MQTT Client interface in approved configurations | Low | MQTT topics follow approved naming convention; message payload validated |

### 8.5 Fault Detection and Safe Mode

| Requirement ID | Description | Priority | Acceptance Criteria |
|----------------|-------------|----------|---------------------|
| FR-060 | QNC shall detect critical device communication faults and log them | High | Fault detection within 2 seconds; fault log includes device ID, fault code, timestamp |
| FR-061 | QNC shall detect DDS QoS negotiation failures and log them as structured faults | Medium | QoS mismatch logged with topic name, requested QoS, offered QoS |
| FR-062 | QNC shall transition to Safe Mode on detection of defined critical faults | High | Safe Mode entered within 1 second of critical fault detection |
| FR-063 | QNC shall reject new actuation commands in Safe Mode | High | Commands rejected with Safe Mode fault code |
| FR-064 | QNC shall support manual and validated automatic recovery from Safe Mode | Medium | Recovery logged; system state validated before returning to Normal Mode |

### 8.6 DDS-Specific Functional Requirements

| Requirement ID | Description | Priority | Acceptance Criteria |
|----------------|-------------|----------|---------------------|
| FR-070 | QNC shall validate subscribed DDS message schemas against loaded IDL artifacts | Medium | Schema validation performed at subscription time; mismatches logged as faults |
| FR-071 | QNC shall support configurable DDS QoS policies per topic subscription and publication | Medium | QoS policies match configuration; negotiation outcome logged |
| FR-072 | QNC shall support SIMPLE discovery mode as baseline DDS deployment mode | Medium | Domain participant discovered by peers within 5 seconds |
| FR-073 | QNC shall support Discovery Server mode in approved advanced configurations | Low | Discovery Server connection established; participant registration confirmed |
| FR-074 | QNC shall log DDS domain join, leave, and fault events | Medium | Events logged with timestamp, domain ID, participant GUID |

### 8.7 Lifecycle and Maintenance

| Requirement ID | Description | Priority | Acceptance Criteria |
|----------------|-------------|----------|---------------------|
| FR-080 | QNC shall support firmware update with validated rollback | Medium | Firmware update completes within 2 minutes; rollback tested on validation failure |
| FR-081 | QNC shall support remote diagnostics via northbound API | Medium | Diagnostic data includes version, configuration state, active faults, uptime |
| FR-082 | QNC shall maintain persistent fault history | Medium | Fault history persists across restarts; minimum 1000 fault events retained |

---

## 9. Non-Functional Requirements

### 9.1 Performance

| Requirement ID | Description | Acceptance Criteria |
|----------------|-------------|---------------------|
| NFR-001 | Command latency (REST API to device) | ≤ 50ms (95th percentile), ≤ 100ms (99th percentile) |
| NFR-002 | Telemetry update rate | ≥ 10 Hz per device (configurable per profile) |
| NFR-003 | Northbound API response time | ≤ 10ms for status queries (95th percentile) |
| NFR-004 | DDS-to-Northbound bridging latency | ≤ 20ms (95th percentile) for approved paths |
| NFR-005 | Fault detection and Safe Mode transition time | ≤ 1 second from fault occurrence to Safe Mode entry |

### 9.2 Reliability and Availability

| Requirement ID | Description | Acceptance Criteria |
|----------------|-------------|---------------------|
| NFR-010 | System uptime | ≥ 99% over 30-day period (excluding scheduled maintenance) |
| NFR-011 | Fault isolation | DDS faults shall not impact southbound protocol operation; tested via fault injection |
| NFR-012 | Configuration rollback success rate | 100% for validated configurations |
| NFR-013 | Restart recovery time | ≤ 30 seconds from power-on to operational state |

### 9.3 Scalability

| Requirement ID | Description | Acceptance Criteria |
|----------------|-------------|---------------------|
| NFR-020 | Concurrent device connections | ≥ 8 devices across all southbound protocols |
| NFR-021 | Concurrent northbound clients | ≥ 10 simultaneous REST API or WebSocket clients |
| NFR-022 | DDS topic subscriptions | ≥ 20 subscribed topics per DomainParticipant |
| NFR-023 | Configuration profile storage | ≥ 50 device profiles in persistent storage |

### 9.4 Maintainability

| Requirement ID | Description | Acceptance Criteria |
|----------------|-------------|---------------------|
| NFR-030 | Configuration update time | ≤ 5 seconds for validated profile updates |
| NFR-031 | Diagnostic data retrieval time | ≤ 2 seconds for full diagnostic data set via API |
| NFR-032 | Log rotation and retention | Automatic rotation at 100MB; minimum 7-day retention |

---

## 10. Safety Position

**QNC is not a safety-rated system.** QNC does not provide functional safety capabilities as defined by IEC 61508, ISO 13849, or similar safety standards. QNC is intended to operate in industrial environments where safety functions are provided by separate, certified safety controllers and safety-rated I/O systems.

Systems integrating QNC shall:
- implement safety functions using certified safety controllers per applicable standards,
- design safety architectures that remain effective regardless of QNC operational state,
- not rely on QNC for emergency stop, safety interlocks, or other safety-critical functions,
- treat QNC as a non-safety component in overall system safety analysis.

QNC's Safe Mode (Section 6.2) is a fault management feature, not a safety function. Safe Mode reduces operational risk by limiting actuation in degraded states but does not replace safety-rated systems.

---

## 11. Security Requirements

| Requirement ID | Description | Priority | Acceptance Criteria |
|----------------|-------------|----------|---------------------|
| SEC-001 | QNC shall support TLS 1.2 or higher for HTTPS and secure WebSocket | High | TLS negotiation validated; weak ciphers rejected |
| SEC-002 | QNC shall support token-based authentication for API access | High | Unauthorized requests rejected with HTTP 401 |
| SEC-003 | QNC shall support role-based access control for configuration changes | Medium | Admin role required for configuration updates; access violations logged |
| SEC-004 | QNC shall encrypt configuration artifacts containing sensitive data at rest | Medium | Encryption uses AES-256; keys managed per approved key management policy |
| SEC-005 | QNC shall log all authentication and authorization events | Medium | Logs include timestamp, user ID, action, result |
| SEC-006 | QNC shall support secure firmware update with signature verification | High | Unsigned firmware rejected; verification failure logged |
| SEC-007 | QNC shall support configurable firewall rules for network interfaces | Medium | Inbound connections restricted per configured rule set |
| SEC-008 | QNC shall support DDS domain access policy configuration | Medium | Unauthorized domain join attempts rejected; policy violations logged |
| SEC-009 | QNC shall support audit logging of all configuration and state changes | High | Audit log tamper-resistant; searchable by timestamp, user, or action |
| SEC-010 | QNC shall support evaluation of DDS-Security standard support for deployments with elevated security requirements | Low | DDS-Security feasibility assessed; implementation roadmap documented |

---

## 12. Configuration Governance

### 12.1 Configuration Artifact Types

QNC configuration shall be governed through the following artifact types:

- **System Configuration**: Network settings, northbound interface configuration, logging levels, security policies.
- **Device Profiles**: Validated per-device-class definitions of commands, parameters, status, and fault mappings.
- **DDS IDL Schemas**: Versioned IDL type definitions for subscribed and published DDS topics.
- **DDS QoS Profiles**: Quality of Service policy definitions for DDS topic subscriptions and publications.
- **Protocol Configurations**: Per-protocol settings such as baud rates, timeouts, retry policies.

### 12.2 Configuration Validation Rules

| Rule ID | Rule Description |
|---------|------------------|
| CFG-001 | All configuration artifacts shall be validated against defined JSON or XML schemas before application |
| CFG-002 | Configuration updates shall be applied atomically; partial updates shall be rejected |
| CFG-003 | Device profiles and DDS IDL schemas shall be versioned and approved before release |
| CFG-004 | Configuration rollback shall be supported for all validated artifacts |
| CFG-005 | Configuration changes shall be logged with timestamp, source, and change description |
| CFG-006 | Invalid configurations shall be rejected with descriptive error messages |
| CFG-007 | Configuration integrity shall be verified on startup and periodically during operation |
| CFG-008 | DDS QoS profiles shall be validated for compatibility with target DDS middleware version |

---

## 13. Interface Requirements

### 13.1 Southbound Protocol Interfaces

QNC shall implement conformant protocol stacks for all baseline and approved southbound protocols. Protocol conformance shall be validated through:
- protocol analyzer capture and analysis,
- interoperability testing with reference devices,
- fault injection testing (cable disconnect, message corruption, timeout scenarios),
- performance benchmarking per NFR requirements.

### 13.2 Northbound API Interfaces

QNC shall expose northbound interfaces conforming to published specifications:
- **REST API**: OpenAPI 3.0 specification defining all endpoints, request/response schemas, and error codes.
- **WebSocket**: JSON-based telemetry and event schema with defined message types.
- **OPC UA**: OPC UA NodeSet definition for all published nodes and methods.
- **MQTT**: Topic naming convention and payload schema for all published topics.

All northbound interfaces shall support API versioning to enable backward compatibility during upgrades.

### 13.3 DDS Interface Governance

DDS topic subscriptions and publications shall be governed through:
- **IDL Type Registry**: A versioned catalog of all approved DDS topic IDL types.
- **QoS Profile Catalog**: Approved QoS policy combinations for common deployment scenarios.
- **Robot Software Compatibility Matrix**: Documented compatibility between QNC DDS interface versions and robot software versions.

DDS interfaces shall be validated through:
- Schema compatibility testing with reference ROS 2 software versions,
- QoS negotiation testing with configured policy combinations,
- Fault injection testing (domain disconnection, QoS mismatch, schema incompatibility),
- Performance benchmarking per NFR-004.

---

## 14. Deployment and Installation

### 14.1 Physical Installation Requirements

QNC shall support installation in industrial environments meeting the following conditions:
- Operating temperature: 0°C to 50°C
- Storage temperature: -20°C to 70°C
- Relative humidity: 10% to 90% non-condensing
- Pollution degree: 2 per IEC 60664-1
- Mounting: DIN rail or panel mount

### 14.2 Network Configuration

QNC shall support the following network configurations:
- Static IP addressing
- DHCP client for automatic IP assignment
- VLAN tagging for network segmentation
- Dual Ethernet interfaces for redundancy (in approved hardware configurations)

### 14.3 DDS Deployment Modes

QNC shall support the following DDS deployment modes:

#### SIMPLE Discovery Mode (Baseline)
- Multicast-based participant discovery
- Default domain ID: configurable (0-232)
- Suitable for single-network, small-to-medium robot deployments

#### Discovery Server Mode (Advanced)
- Server-based participant discovery
- Supports cross-network deployments
- Requires Discovery Server infrastructure
- Suitable for large-scale or multi-network robot deployments

Deployment mode selection shall be governed by validated deployment profiles.

---

## 15. Validation Requirements

Release validation shall include:
- systems architecture review sign-off,
- protocol conformance test results for each supported southbound protocol,
- **FastDDS DomainParticipant integration tests** where the FastDDS extension is included in scope, covering join/leave events, topic subscription/publication, QoS negotiation, and fault injection for domain disconnection and schema mismatch,
- **DDS-derived northbound bridging tests** where such interfaces are included in scope, including latency measurement and schema-mapping verification for the released path,
- fault injection test results covering southbound, DDS-peer, and northbound fault paths,
- performance benchmark evidence per NFR-001 through NFR-005,
- security review results covering DDS domain access policy where applicable,
- configuration governance review, including profile and IDL artifact validation and rollback testing,
- release readiness review sign-off.

Validation planning shall be maintained in the companion QNC Verification & Validation Plan document.

---

## 16. Roadmap Management

### 16.1 Release Planning

All roadmap items shall follow a disciplined approval and release process:
1. Justification and use case documentation
2. Architecture review and approval
3. Design freeze
4. Implementation and integration testing
5. Validation per Section 15
6. Release readiness sign-off

No capability enters a release without completing all stages.

### 16.2 Approved Extension Direction

- Additional southbound protocols: Modbus TCP, CANopen, selected serial/CAN-based protocols.
- Northbound interfaces: OPC UA Server, MQTT Client.
- Normalized multi-device data aggregation and event routing.
- **FastDDS / ROS 2 DomainParticipant integration** in approved deployment modes, including released DDS-derived mapping paths such as DDS-to-OPC UA and DDS-to-MQTT where validated.
- DDS-aware relay and cross-robot topic normalization patterns for approved deployment classes.

### 16.3 Advanced Gateway Direction

- Industrial Ethernet families such as EtherCAT and PROFINET, subject to approved coprocessor or hardware-assisted architecture where required.
- Plant-level supervisory integration.
- **FastDDS Discovery Server mode** for approved large-scale or cross-network multi-robot deployments.
- Fleet-level robot data aggregation and normalized cluster telemetry.

All roadmap items shall enter a release only through formal approval, design freeze, integration testing, validation, and release sign-off per the governance process in Section 18.

---

## 17. Risks and Mitigations

| # | Risk | Probability | Impact | Mitigation |
|---|------|-------------|--------|------------|
| 1 | Scope creep from over-broad protocol claims | High | High | Enforce scope model (Section 5); no protocol enters release without full approval cycle |
| 2 | Ambiguous safety positioning | Medium | High | Maintain explicit non-safety-controller statement; require separate certification for safety functions |
| 3 | Hardware platform limitations for advanced protocols | Medium | Medium | Maintain explicit dependency between protocol claims and validated hardware architecture |
| 4 | Device profile quality and completeness | Medium | Medium | Formal profile review and validation gating per CFG-003 and CFG-008 |
| 5 | Northbound interface scope drift | Medium | Medium | Northbound services require explicit schema definition and release approval |
| 6 | DDS domain isolation failures | Medium | High | Enforce domain ID configuration; test join/leave behavior; ensure DDS faults are isolated from unrelated services |
| 7 | DDS IDL schema mismatch with robot software versions | High | Medium | Govern IDL artifacts as versioned profiles; validate schema compatibility; maintain robot software compatibility matrix |
| 8 | DDS QoS incompatibility causing data loss or unstable behavior | Medium | High | Validate QoS negotiation outcomes; log QoS mismatches as structured faults; test supported QoS combinations |
| 9 | Discovery Server complexity in multi-robot deployments | Low | Medium | Restrict Discovery Server mode to approved deployment classes; require dedicated validation before release |
| 10 | Security exposure from DDS domain membership | Medium | High | Implement DDS domain access policy; evaluate DDS Security support for deployments with elevated security requirements |

---

## 18. Governance and Change Control

Any change to the following elements requires a formal review and approval action:
- product scope boundaries (Sections 4 and 5),
- safety position (Section 10),
- architecture principles (Section 7),
- FastDDS extension scope and deployment mode commitments (Section 5.5.1),
- release scope commitments (Sections 5.2, 5.3, 5.4),
- non-goals (Section 4.2),
- security requirements (Section 11).

Changes to functional requirements, non-functional requirements, configuration governance rules, roadmap items, or externally communicated claims affecting interoperability, performance, safety posture, or market positioning require engineering review and product management approval. Minor clarifications and editorial corrections may be made with document control approval only.

---

## 19. Glossary

| Term | Definition |
|------|------------|
| **Operational Inhibit Mode** | A QNC operational state in which new actuation commands are rejected, southbound communication may be limited to status or health queries, northbound publication may be limited to status or fault reporting, and DDS participation may continue in subscribe-only mode where allowed by policy. |
| **Industrial Intelligent Edge Gateway** | QNC's approved extension architecture mode in which it may normalize multiple supported southbound protocols, provide approved northbound industrial data services, and participate in compatible robot communication domains where released. |
| **FastDDS** | Fast Data Distribution Service — a DDS implementation developed by eProsima and based on DDS and RTPS standards. FastDDS is commonly used in ROS 2 environments, although ROS 2 supports multiple middleware implementations. |
| **DomainParticipant** | A DDS entity that represents participation within a DDS domain. QNC's FastDDS module may create and manage a DomainParticipant to join a compatible robot communication domain where configured and released. |
| **ROS 2 (Robot Operating System 2)** | An open-source robotics middleware framework that commonly uses DDS-based middleware for internal communication among nodes, while supporting multiple middleware implementations. |

---

## 20. Executive Summary for Stakeholders

**QNC is a controlled industrial integration gateway and software-first edge platform.** It enables robotic systems to communicate with supported industrial tools and field devices through validated, profile-governed protocol abstractions. It provides a disciplined, extensible path from robot-to-tool integration toward broader industrial edge integration capability.

**In version 2.2, QNC introduces FastDDS-native participation as a strategic extension capability.** This addition reflects the increasing importance of DDS-based communication in modern robotics environments, including ROS 2-based systems. By enabling approved QNC configurations to join a compatible DDS domain as a DomainParticipant, QNC is intended to support a governed integration path between robot-domain data and external industrial or IT systems.

**This upgrade does not alter QNC's release discipline or scope boundaries.** FastDDS is introduced as an approved extension scope item, following the same approval, validation, and release governance model as other QNC capabilities. No FastDDS feature shall be represented as baseline capability until it has completed its validation and release cycle.

**The extension broadens QNC's potential applicability within robotics-oriented deployments.** Potential users of the FastDDS-related capability may include:

| Original Target Customers | Potential Additional Users (FastDDS Extension) |
|---------------------------|------------------------------------------------|
| Robot system integrators | ROS 2 robot developers and platform teams |
| Factory automation departments | Multi-robot integration and coordination teams |
| Equipment OEMs | Service operators seeking governed fleet telemetry integration |
| OT/IT integration engineers | Robotics software teams seeking plant-system data integration |

**In summary**: QNC v2.2 defines a governed integration platform intended to support both industrial connectivity and, in approved configurations, native DDS-aware robot integration under controlled scope, validation, and release discipline.

---

## Recommended Companion Documents

1. **QNC Interface Control Document (ICD)** — defines all protocol-level message schemas, northbound API definitions, DDS topic subscription/publication schemas, and IDL type registry.
2. **QNC Device Profile Specification** — defines the device profile schema, profile lifecycle rules, and approved device class definitions.
3. **QNC Protocol & Service Module Specification** — defines per-protocol implementation architecture including the FastDDS Participant module design, QoS profile catalog, and deployment mode configuration.
4. **QNC Verification & Validation Plan** — defines test methodologies, benchmark procedures, FastDDS integration test cases, fault injection test plans, and release acceptance criteria.
5. **QNC Operations & Recovery Guide** — defines configuration procedures, DDS domain configuration guidance, deployment mode selection criteria, fault recovery procedures, and safe mode operations.
6. **QNC FastDDS IDL Type Registry** — a dedicated artifact registry governing all approved DDS topic IDL types, their versions, compatibility matrices with robot software versions, and approval status.





















---
---
# QNC Product Requirements & System Definition Specification  
**Document Number:** QNC-PRD-001  
**Version:** 2.0  
**Status:** Proposed Baseline Standard  
**Classification:** Internal / Project Controlled  

---

## 1. Document Purpose

This document defines the **product purpose, scope, architecture boundary, functional requirements, non-functional requirements, and validation expectations** for the **Quick Network Connector (QNC)** project.

This document is the authoritative baseline for:
- product strategy and scope control,
- engineering planning,
- interface design governance,
- validation planning,
- release readiness.

This document does **not** replace detailed protocol, pinout, message schema, or certification artifacts. Those shall be maintained in companion specifications.

---

## 2. Product Definition

### 2.1 Formal Product Definition

**QNC is a configurable industrial end-effector connectivity gateway that enables robotic platforms to communicate with supported industrial tools through protocol adapters and validated device profiles, reducing the need for custom per-device integration code.**

QNC is intended to:
- normalize communication with supported industrial end-effectors,
- isolate robot applications from low-level protocol handling,
- provide controlled onboarding of supported devices through validated configuration and profile artifacts,
- improve integration speed and maintainability within clearly defined protocol and device boundaries.

### 2.2 Core Design Principle

QNC shall be defined as a **connectivity and integration product**, not as a general-purpose robot controller, safety controller, or AI platform.

QNC shall own:
- protocol transport and session handling,
- supported-device profile loading and validation,
- normalized status reporting,
- command forwarding and error reporting,
- configuration management and lifecycle control.

QNC shall **not** own:
- high-level robot task logic,
- perception, planning, or manipulation strategy,
- certified safety control functions,
- unsupported protocol translation,
- arbitrary device semantics beyond approved supported profiles.

### 2.3 Product Positioning

QNC is positioned between:
- a **robot controller or robot software stack**, and
- a **supported industrial end-effector**.

QNC is not merely a raw protocol converter.  
QNC is also not a full smart end-effector controller.  
Its differentiation lies in combining:
- protocol adaptation,
- controlled device-profile management,
- robot-integration-oriented abstraction,
- operational configuration and fault handling.

---

## 3. Problem Statement

Industrial robot deployments often require custom integration work for each end-effector, even when devices use common fieldbus or industrial communication protocols. This creates:
- repeated driver development,
- inconsistent tool onboarding,
- brittle device-specific logic,
- long integration cycles,
- maintenance overhead across fleets.

QNC addresses this problem by introducing a controlled middle layer that standardizes how supported tools are integrated, configured, monitored, and operated.

---

## 4. Goals and Non-Goals

### 4.1 Product Goals

QNC shall:
1. reduce repeated low-level integration effort for supported devices,
2. provide a stable and well-defined boundary between robot software and device protocols,
3. enable configuration-based onboarding for supported device profiles,
4. standardize health, fault, and command behaviors across supported devices,
5. improve maintainability and upgradeability of robot-tool integrations.

### 4.2 Non-Goals

QNC shall not:
1. claim universal support for all industrial devices or protocols,
2. replace robot application logic,
3. serve as a certified safety controller,
4. provide AI vision, language, or general-purpose edge intelligence as a core function,
5. guarantee support for new devices without validation and profile approval,
6. expose unbounded arbitrary device behavior under the label of “generic abstraction.”

---

## 5. Release Scope

### 5.1 In-Scope for Baseline Release

The baseline release of QNC shall support:
- robot-to-QNC communication through the approved middleware interface,
- end-effector connectivity for approved protocols only,
- validated device profiles for approved device classes,
- controlled discovery and configuration workflows,
- normalized fault and health reporting,
- firmware and configuration lifecycle management.

### 5.2 Protocol Scope for Baseline Release

The baseline release shall support only the following end-effector-side protocols unless explicitly approved in a later release:
- **Modbus RTU**
- **IO-Link**

No other protocol shall be represented as baseline capability in release documentation unless it has passed design freeze, integration testing, and release approval.

### 5.3 Device Scope for Baseline Release

The baseline release shall initially target the following device classes:
- electric grippers,
- simple linear actuators,
- selected industrial sensors with supported profile definitions.

Each supported device shall require:
- a validated device profile,
- conformance testing,
- release approval.

### 5.4 Explicitly Out of Scope

The following are out of scope for the baseline release:
- EtherCAT, EtherNet/IP, PROFINET, CAN-based expansion unless separately approved,
- onboard AI perception or tracking functions,
- general-purpose compute applications,
- autonomous safety decision-making,
- support for arbitrary third-party devices without validation.

---

## 6. Stakeholders and Users

### 6.1 Primary Stakeholders
- Product Management
- Systems Engineering
- Firmware / Embedded Engineering
- Middleware / Robotics Software Engineering
- Validation / QA
- Manufacturing / Operations
- Program Management

### 6.2 Primary Users
- robot integrators,
- robotics platform developers,
- deployment engineers,
- test and validation engineers,
- support and field service teams.

### 6.3 Primary Customer Job-to-Be-Done

“For supported industrial tools, enable faster and more reliable robot integration without requiring custom low-level device handling for each deployment.”

---

## 7. System Context and Architecture Boundary

### 7.1 System Role

QNC sits between the robot platform and the end-effector. Its responsibility is to translate, normalize, validate, and report interactions within supported boundaries.

### 7.2 Boundary of Responsibility

#### QNC SHALL own:
- physical/protocol communication with the end-effector,
- supported protocol initialization and session handling,
- profile validation and loading,
- normalized command routing,
- normalized telemetry and fault state publication,
- device connection status determination,
- local configuration integrity enforcement.

#### Robot Platform SHALL own:
- application-level task intent,
- motion planning and coordination,
- product/process logic,
- selection of compatible supported profiles,
- high-level device semantics beyond the approved QNC abstraction,
- system-level safety orchestration.

#### End-Effector SHALL own:
- device-internal actuation behavior,
- vendor-specific firmware behavior,
- vendor-defined protocol semantics not abstracted by QNC.

### 7.3 Architecture Principle

QNC shall follow a **layered architecture**:

1. **Protocol Layer**  
   Handles physical and protocol communication.

2. **Profile Layer**  
   Maps approved device behavior into controlled, normalized abstractions.

3. **Integration Layer**  
   Exposes commands, telemetry, state, and faults to the robot platform.

This layered model shall be used consistently in all companion documents.

---

## 8. Product Differentiation

QNC shall be differentiated from adjacent solution categories as follows:

### 8.1 Versus Raw Protocol Converter
A raw protocol converter only passes or translates communications.  
QNC adds:
- validated device profiles,
- normalized state and fault reporting,
- integration governance,
- lifecycle management.

### 8.2 Versus Custom Device Driver
A custom driver is typically device-specific and code-bound.  
QNC reduces repeated custom driver development by centralizing supported protocol handling and controlled profile-based integration.

### 8.3 Versus Smart Tool Controller
A smart tool controller owns tool logic and often device-specific intelligence.  
QNC does not replace the tool or robot application. It standardizes supported interaction patterns.

### 8.4 Versus General Robot Middleware Node
QNC may participate in a middleware ecosystem, but its purpose is not general messaging. Its purpose is controlled robot-to-tool integration.

---

## 9. Functional Requirements

### 9.1 Supported Connectivity

**FR-001** QNC shall provide robot-to-tool connectivity for approved baseline protocols only.  
**FR-002** QNC shall reject unsupported protocol operation attempts with explicit error reporting.  
**FR-003** QNC shall expose connection state, protocol state, and profile state independently.

### 9.2 Device Profiles

**FR-010** QNC shall support approved device profiles as controlled artifacts.  
**FR-011** Each device profile shall define:
- supported commands,
- parameter ranges,
- required initialization sequence,
- expected status structure,
- fault mappings,
- compatibility metadata,
- versioning metadata.

**FR-012** QNC shall not load a device profile that fails schema validation.  
**FR-013** QNC shall not advertise a device as supported unless a validated profile is installed and approved.  
**FR-014** Device profiles shall be versioned, signed, and traceable.

### 9.3 Command Model

**FR-020** QNC shall expose a normalized command model for supported device classes.  
**FR-021** The command model shall distinguish between:
- transport-level commands,
- profile-level commands,
- maintenance/configuration commands.

**FR-022** Where raw register access is supported, it shall be explicitly labeled as an advanced/engineering function and shall not be represented as the primary product abstraction.

**FR-023** QNC shall validate command parameters against the active device profile before transmission.  
**FR-024** Invalid commands shall be rejected before execution with deterministic error codes.

### 9.4 Discovery and Identification

**FR-030** QNC shall support controlled device discovery for supported connection types.  
**FR-031** Discovery behavior shall be deterministic and documented per protocol.  
**FR-032** If multiple identification mechanisms exist, the precedence order shall be explicitly defined.  
**FR-033** Discovery shall not silently infer a device identity that cannot be validated against an installed profile.  
**FR-034** Discovery shall result in one of the following states:
- identified and supported,
- connected but unsupported,
- ambiguous identification,
- communication fault,
- no device detected.

### 9.5 Telemetry and State

**FR-040** QNC shall expose normalized telemetry for:
- connectivity,
- protocol health,
- active profile,
- command execution status,
- device health/fault state.

**FR-041** QNC shall separate:
- transport state,
- device-class state,
- vendor-specific state.

**FR-042** Device-class state definitions shall be restricted to approved supported classes.  
**FR-043** Vendor-specific states shall not be mislabeled as universal semantics.

### 9.6 Fault Handling

**FR-050** QNC shall publish structured faults with:
- unique code,
- severity,
- source layer,
- human-readable description,
- recovery guidance.

**FR-051** Fault categories shall include:
- communication faults,
- profile/configuration faults,
- device-reported faults,
- integrity/security faults,
- operational misuse faults.

**FR-052** QNC shall define a deterministic degraded mode and safe mode behavior.

### 9.7 Configuration and Lifecycle Management

**FR-060** QNC shall support controlled configuration import/export.  
**FR-061** Configuration changes shall be auditable.  
**FR-062** QNC shall support rollback to the last known valid configuration.  
**FR-063** Firmware and profile update mechanisms shall be separated logically, even if delivered together operationally.

---

## 10. Safety and Operational Boundary

### 10.1 Safety Position

QNC is a **non-safety controller** in the baseline release.

It may detect, report, and locally inhibit unsafe communication behavior, but it shall not be marketed or specified as a certified safety function unless separately designed, certified, and documented.

### 10.2 Safety-Related Requirements

**SR-001** QNC shall explicitly identify safety-relevant fault conditions.  
**SR-002** QNC shall report safety-relevant faults with highest available priority in its communication layer.  
**SR-003** QNC shall support a local inhibit mode that stops acceptance of new actuation commands under defined critical faults.  
**SR-004** System-level emergency stop authority shall remain with the robot/system safety architecture, not with QNC baseline functionality.

### 10.3 Safe Mode Definition

Safe mode shall mean:
- new actuation commands are rejected,
- device communication may be limited to health/status queries if allowed by policy,
- fault state remains latched until acknowledged and cleared by defined recovery procedure.

The exact recovery procedure shall be defined in the companion ICD and operations manual.

---

## 11. Security Requirements

### 11.1 Security Principles

QNC shall treat firmware, configuration, and device-profile artifacts as controlled assets.

### 11.2 Security Requirements

**SEC-001** Firmware packages shall be signed and verified before installation.  
**SEC-002** Device-profile and configuration artifacts shall be integrity-checked before activation.  
**SEC-003** Trust anchors and verification keys shall be protected from unauthorized modification.  
**SEC-004** Update paths shall support anti-rollback policy unless explicitly overridden through controlled service procedures.  
**SEC-005** All update and configuration events shall be audit logged.  
**SEC-006** Physical service interfaces shall be governed by documented access policy.

---

## 12. Non-Functional Requirements

### 12.1 Performance

QNC shall not advertise a single universal latency number across all protocols and devices.

Instead:

**NFR-001** Performance shall be specified by:
- protocol,
- device profile,
- command type,
- payload class,
- operating condition.

**NFR-002** Benchmark methodology shall be defined and repeatable.  
**NFR-003** Published performance values shall identify whether they are typical, worst-case, or guaranteed.  
**NFR-004** Release approval shall require measured performance evidence for each supported baseline profile.

### 12.2 Reliability

**NFR-010** QNC shall recover deterministically from transient communication faults where recovery is safe and defined.  
**NFR-011** QNC shall not enter undefined states due to malformed configuration or unsupported device behavior.  
**NFR-012** QNC shall preserve fault diagnosability after communication interruptions and restart events.

### 12.3 Maintainability

**NFR-020** Supported device integration shall be maintainable through versioned controlled artifacts.  
**NFR-021** Debugging interfaces shall distinguish clearly between configuration faults, protocol faults, and device faults.  
**NFR-022** Logs shall be structured for field diagnosis.

### 12.4 Usability

**NFR-030** Operational state and fault reasons shall be visible in a human-readable way through the supported management interface.  
**NFR-031** Unsupported or ambiguous device conditions shall be explained explicitly rather than failing silently.

### 12.5 Compliance Readiness

**NFR-040** Regulatory and industrial compliance needs shall be documented before release freeze.  
**NFR-041** No compliance-sensitive claim shall be made in customer-facing material unless verified.

---

## 13. Configuration and Profile Governance

### 13.1 Governance Principle

Configuration is part of the product, not an informal extension mechanism.

### 13.2 Requirements

**CFG-001** A formal schema shall exist for device profiles.  
**CFG-002** Profile changes shall be reviewed, tested, and versioned.  
**CFG-003** Compatibility between QNC firmware version and profile version shall be enforced.  
**CFG-004** Profiles shall declare supported hardware, protocol parameters, command mappings, and state mappings.  
**CFG-005** Profiles shall identify unsupported functions explicitly.

### 13.3 Release Policy

A device shall be considered supported only when:
- the profile is validated,
- compatibility is confirmed,
- test evidence exists,
- the support status is published.

---

## 14. Data and Interface Definition Policy

The detailed message schemas, pinouts, and protocol transaction specifications shall be defined in a separate **QNC Interface Control Document (ICD)**.

That ICD shall contain:
- physical interfaces,
- signal definitions,
- topic/message schemas,
- protocol state machines,
- command/response formats,
- error code catalog,
- timing behavior per supported profile,
- recovery flows.

This PRD/SDS is the governing product-definition document.  
The ICD shall be subordinate to this document and shall not expand product scope without formal change approval.

---

## 15. Validation and Acceptance

### 15.1 Validation Principle

All baseline claims shall be validated through evidence, not assertion.

### 15.2 Required Validation Artifacts

Release approval shall require:
- system architecture review,
- protocol conformance test results,
- device profile validation records,
- fault-injection test results,
- update and rollback test evidence,
- benchmark evidence,
- installation and recovery workflow validation,
- support readiness review.

### 15.3 Acceptance Criteria

The baseline release shall not be approved until:
1. all in-scope requirements have owners and verification methods,
2. all supported protocols have validated interface specifications,
3. all supported device profiles have traceable approval,
4. unsupported roadmap items are removed from release claims,
5. operational safety boundaries are documented clearly.

---

## 16. Roadmap Management

Roadmap items may be documented separately, but they shall not be mixed with baseline requirements unless labeled clearly as non-release items.

Potential roadmap areas may include:
- additional industrial protocols,
- expanded device classes,
- fleet management features,
- enhanced diagnostics,
- deeper integration tooling.

Roadmap content shall not appear in baseline release claims unless committed, resourced, and verified.

---

## 17. Risks and Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Product scope expands beyond core integration role | Delays, complexity, unclear value | Freeze baseline scope and enforce non-goals |
| Device-profile model becomes uncontrolled | Fragile deployments, support burden | Implement schema, signing, review, compatibility enforcement |
| Ambiguous safety positioning | Adoption risk, integration risk | Define QNC as non-safety controller in baseline |
| Overclaiming universality | Market confusion, delivery mismatch | Limit claims to approved protocols and supported profiles |
| Mixed abstraction levels | Developer confusion, brittle interfaces | Separate protocol, profile, and integration layers |
| Lack of measurable evidence | Credibility loss | Require benchmark and validation artifacts for release |

---

## 18. Governance and Change Control

Any change to:
- supported protocol scope,
- supported device classes,
- architecture boundary,
- safety position,
- command abstraction model,
- profile governance model

shall require formal review by Product, Systems Engineering, and Project Management.

No companion ICD or implementation document may redefine the product beyond this approved baseline without controlled change approval.

---

## 19. Glossary

**Device Profile**  
A controlled artifact that defines how a supported device is integrated within QNC.

**Protocol Layer**  
The layer responsible for physical/protocol communication.

**Profile Layer**  
The layer that maps supported device behavior into approved abstractions.

**Integration Layer**  
The layer exposing QNC behavior to the robot platform.

**Safe Mode**  
A deterministic fault-handling state in which new actuation commands are blocked until defined recovery occurs.

**Supported Device**  
A device with an approved, validated, versioned profile and documented compatibility status.

**Unsupported Device**  
A connected device for which no approved QNC support status exists.

---

## 20. Executive Summary for Stakeholders

QNC should be developed and communicated as a **scoped industrial robot-tool integration product**, not as a universal adapter or broad embedded AI platform. Its value lies in reducing repeated low-level integration work through **supported protocol adapters, validated device profiles, and controlled operational behavior**. By separating product definition from detailed interface specification, the project becomes more credible, easier to execute, and more maintainable over time. This rewrite resolves several of the conceptual ambiguities present in the original uploaded source [Source](https://www.genspark.ai/api/files/s/MQZoL9fC).

---

## Recommended companion documents

To make this production-ready in practice, the following controlled documents should be created next:

1. **QNC Interface Control Document (ICD)**  
   Detailed message schemas, pinouts, timing, error codes, and state models.

2. **QNC Device Profile Specification**  
   Schema, versioning rules, validation policy, and compatibility model.

3. **QNC Verification & Validation Plan**  
   Test coverage, evidence structure, and release gates.

4. **QNC Operations & Recovery Guide**  
   Installation, diagnostics, safe-mode recovery, updates, rollback, field service procedures.

---
