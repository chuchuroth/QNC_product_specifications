# QNC-PRD

# QNC 产品需求与系统定义规范

**文档编号:** QNC-PRD-001
**版本:** 3.0(v1 产品基线)
**状态:** 可实施基线版本 —— 在 v1 范围内取代 PRS v2.3
**分类:** 内部 / 项目受控

---

## 文档变更日志(v2.3 → v3.0)

| 领域 | 变更内容 |
| --- | --- |
| 产品定义 | 重新定位为面向 NEURA 单元的**以软件为核心的工业边缘连接网关**;强化明确的**非目标**声明 |
| 架构 | 严格的**三层**产品模型:**核心**、**工业扩展包**、**DDS 扩展包** |
| DDS | FastDDS 仅作为**可选的 QNC DDS 扩展包**;从不作为强制性平台基础设施 |
| 配置文件/配置 | **YAML** 在编写/治理方面具权威性;仅在运行时确有需要处才使用 JSON |
| 语义 | **单一规范模型** —— 外置为 **QNC-SEMANTIC-CORE-v1** |
| 发现机制 | 移除通用发现要求;改为**按协议区分的能力矩阵** |
| v1 范围 | **规范性移除**发现服务器、集群聚合、SCADA/MES/ERP、EtherCAT/PROFINET 的 v1 声明、广泛的 DDS 拓扑结构、通用厂商插件系统 |
| 产品组合 | 主要对齐对象:**MAiRA、LARA、MAV**;MiPA / 4NE1 为**次要/未来**规划 |
| 新增 | 打包、容量限制、安全运维、**双 API 版本管理**、平台兼容性矩阵、兼容性/弃用策略摘要 |

---

## 1. 目的

本规范定义 **QNC(Quick Network Connector,快速网络连接器)** 的**产品目的**、**v1 范围**、**架构**、**功能与非功能需求**、**治理机制**、**打包方案**、**容量**、**安全运维**、**兼容性策略**以及**验证预期**。

配套规范性文档如下:

| 文档 | 角色 |
| --- | --- |
| **QNC-SEMANTIC-CORE-v1** | 规范的命令 / 响应 / 遥测 / 故障 / 生命周期语义 |
| **ICD** | 接口契约、OpenAPI/WS 绑定 |
| **DPS** | YAML 设备配置文件模式与治理 |
| **PSM** | 协议模块、DDS 扩展包隔离、能力矩阵 |
| **QNC-IDL** | DDS 扩展包 IDL 注册表与治理(仅限扩展) |
| **ORG** | 运维、恢复、试运行 |

---

## 2. 产品定义(v1)

### 2.1 正式定义

**QNC 是一个模块化、以软件为核心的工业边缘连接网关**,面向 **NEURA** 机器人单元及相邻自动化环境。它提供**南向**工业/设备集成、**YAML 治理的设备配置文件**、向规范核心的**语义归一化**、**机器人侧**与**北向** API(REST、WebSocket、结构化日志)、具备**故障隔离**的**安全模式**,以及**可原子化激活并回滚的配置管理**。

### 2.2 明确的非目标

QNC **不是**、v1 版本**必须不**被定位为:

- 机器人运动**控制器**,
- **ROS/ROS2 替代方案**或通用机器人**中间件**,
- 通用**工厂/云端/MES/SCADA** 平台,
- **通用协议翻译**层,
- **功能安全**控制器或安全等级设备。

### 2.3 产品组合对齐

**主要集成对象(v1):** **MAiRA**、**LARA**、**MAV** —— 复用现场总线/工具集成模式、ROS2 相关集成、Python(NeuraPy)工作流以及外围设备型号相关模式。

**次要/未来规划:** **MiPA**、**4NE1** —— 仅在接口成熟并具备验证工件后方作为集成目标。

**v1 标准化重点:** 末端工具(EOAT)与工业外围设备;**移动/ROS2 集成路径**作为**文档化模式**呈现,而非无边界的中间件承诺。

### 2.4 复用原则

优先复用 **NeuraPy**、现有的 **YAML** 配置方法、试运行用户体验、工具/夹爪配置文件模型以及已建立的 ROS2 集成路径。**不得**引入并行的 SDK、重复的配置文件生态,或与现有运维工具相竞争的方案。

---

## 3. 三层产品模型(规范性)

### 3.1 QNC 核心(v1 强制性产品)

**仅**包含:

| 能力 | 说明 |
| --- | --- |
| 南向集成 | v1 基线已发布的工业/设备协议(见 §5) |
| 配置文件治理 | 经验证、签名、生命周期管理的 YAML 配置文件 |
| 语义归一化 | 将南向数据映射**至** QNC-SEMANTIC-CORE-v1 |
| 机器人侧 API | REST/WebSocket(独立版本管理) |
| 北向 API | 用于可观测性的 REST/WebSocket/日志(独立版本管理) |
| 安全模式与故障隔离 | 操作性抑制;南向/北向/服务域相互区分 |
| 配置管理 | **原子化**激活、**回滚**、审计 |

**核心不得依赖**DDS、OPC UA 或 MQTT 的运行时存在。

### 3.2 QNC 工业扩展包(可选软件包)

可选的、独立发布的模块,尽可能与核心的编译/运行时**隔离**:

| 扩展包(示意) | 说明 |
| --- | --- |
| Modbus TCP | 扩展软件包 |
| CANopen | 扩展软件包 |
| OPC UA 服务器 | 扩展 —— **非** v1 核心规范性要求 |
| MQTT 客户端 | 扩展 —— **非** v1 核心规范性要求 |

即便存在 OPC UA/MQTT 扩展,工厂/MES/ERP/SCADA 集成**仍不属于** v1 规范范围(不规定企业级语义)。

### 3.3 QNC DDS 扩展包(可选软件包)

包含以下内容的**可选**软件包:

- FastDDS 参与(DomainParticipant),
- 已治理的 IDL 与 QoS 工件(与设备配置文件**分开**治理),
- 若明确发布,可提供经验证的、**范围狭窄**的 DDS 桥接,
- **隔离的 DDS 故障域**(按语义核心规定为 `fault_domain = dds_pack`)。

DDS 扩展包**必须**与核心南向运行保持负载故障隔离及升级隔离。

---

## 4. 问题陈述(简述)

工业末端工具(EOAT)及外围设备呈现出异构的协议与语义。机器人应用团队需要**快速、经验证的集成**,而无需自行维护每一个底层协议栈。QNC 提供**治理边界**以及与 NEURA 生态工具**对齐的归一化语义**。

---

## 5. v1 基线范围

### 5.1 v1 南向(核心基线)

规范性的 **v1 基线**南向协议:

- **IO-Link**(v1.1 主站),
- **Modbus RTU**(串口),
- **EtherNet/IP**(适配器,按发布情况支持 Class 1 / Class 3),
- **离散数字 IO**。

### 5.2 v1 北向(核心基线)

- **REST**(HTTPS,JSON),
- **WebSocket**(JSON),
- **结构化日志**(例如 JSON 行格式;可选部署为 syslog 映射)。

### 5.3 v1 明确非规范内容(仅路线图规划)

以下内容**不得**作为 v1 强制性要求出现:

| 项目 | 分类 |
| --- | --- |
| DDS 发现服务器 | 路线图 / 未来架构说明 |
| 集群级聚合 | 路线图 |
| SCADA / MES / ERP 集成 | 路线图 / v1 产品范围之外 |
| EtherCAT、PROFINET | 路线图;依赖硬件 |
| 广泛的多跳 DDS 桥接拓扑 | 路线图 |
| 通用厂商扩展/插件运行时 | 路线图 |
| "通用"工厂/云端连接假设 | 非 v1 内容 |

它们**可以**出现在配套文档的**附录 —— 未来架构**中;**不得**作为 v1 基线验收标准。

### 5.4 按协议区分的发现/识别

**不存在通用的"发现"要求。**

每个南向协议**应**在 PSM 中记录**能力矩阵**:

| 能力 | IO-Link | Modbus RTU | EtherNet/IP | DIO |
| --- | --- | --- | --- | --- |
| 自动发现/枚举 | 按 PSM 规定 | 不适用 —— 需要寻址 | 按 PSM 规定 | 不适用 |
| 识别/绑定 | 是 | 单元 ID + 可选 ID 寄存器 | 按 CIP identity | 通道映射 |
| 热插拔容忍度 | 按 PSM 规定 | 按 PSM 规定 | 按 PSM 规定 | 按 PSM 规定 |

*(示意性内容 —— 权威细节以 PSM 为准。)*

---

## 6. 功能需求(v1 核心亮点)

需求 ID 使用 **REQ-CORE-xxx** 表示核心需求,**REQ-EXT-xxx** 表示扩展需求(除非安装相应软件包,否则不属于 v1 基线)。

| ID | 说明 | 优先级 |
| --- | --- | --- |
| REQ-CORE-010 | 加载并验证 **YAML** 设备配置文件;拒绝无效文件;支持回滚 | 高 |
| REQ-CORE-011 | 将南向设备映射至 **QNC-SEMANTIC-CORE-v1** 的命令/遥测/故障 | 高 |
| REQ-CORE-020 | 按 ICD 暴露**机器人侧** REST/WS;**独立**版本管理 | 高 |
| REQ-CORE-021 | 按 ICD 暴露**北向** REST/WS/日志;**独立**版本管理 | 高 |
| REQ-CORE-030 | **原子化**配置激活;失败激活**不得**部分应用 | 高 |
| REQ-CORE-031 | 持久化已验证的配置;重启后在**≤ 30 秒**冷启动时限内恢复(见 §8) | 高 |
| REQ-CORE-040 | 在规定的严重故障时进入**安全模式**;**抑制执行动作**;发布故障 | 高 |
| REQ-CORE-041 | 南向故障**不得**与 DDS 扩展包故障混淆 | 高 |

**DDS 扩展包**(如已安装):按 PSM 与 QNC-IDL 定义的可选 REQ-DDS-\* 需求集 —— **不属于**核心符合性要求。

---

## 7. 非功能需求与容量(v1)

### 7.1 v1 默认容量目标(按 SKU 可调)

| 参数 | v1 默认目标值 | 备注 |
| --- | --- | --- |
| 最大南向设备端点数 | **8** 个活动端点 | 跨所有协议实例统计 |
| 最大并发 REST 会话数 | **16** 个 | 机器人侧与北向的拆分策略见部署指南 |
| 最大 WebSocket 客户端连接数 | **8** 个 | 可配置为更低值 |
| DDS 扩展包已订阅主题数 | **≤ 20** 个 | 仅当安装 DDS 扩展包时适用 |
| 已存储设备配置文件(YAML)数量 | **≥ 50** 个 | 磁盘上的治理存储 |
| 故障历史保留量 | **≥ 1000** 条事件或 **7 天**,以先到者为准 | 可配置 |
| 日志轮转 | 默认 **100 MB** 文件集 | 典型保留期**≥ 7 天** |

### 7.2 性能(设计目标 —— 证据以 V&V 为准)

| ID | 目标 |
| --- | --- |
| NFR-001 | REST 命令 → 设备:p95 **≤ 50 毫秒**(经验证场景) |
| NFR-002 | 遥测频率**≥ 10 Hz**(每设备默认类别) |
| NFR-003 | 状态查询 p95 **≤ 10 毫秒**(本地网络) |
| NFR-004 | 严重故障后**≤ 1 秒**进入安全模式 |

### 7.3 降级处理

过载情况下,QNC **宜**:降低低优先级北向流量、保留故障/安全模式通道、保留执行动作抑制语义。具体行为**应**按模块在 PSM 中详细记录。

---

## 8. 打包定义(v1)

QNC v1 **应**支持以下至少一种部署类别(产品打包可组合使用):

| 类别 | 说明 |
| --- | --- |
| **A —— 纯软件** | 在已批准的工业级 Linux 上以容器或裸机方式打包 |
| **B —— 一体化镜像** | 经验证的操作系统 + QNC + 扩展包,作为**参考栈**交付 |
| **C —— 嵌入式运行时** | 面向已批准嵌入式目标平台的 BSP 集成二进制文件 |

**认证参考栈:** 附带 SBOM、锁定版本及发布说明的 **B** 类镜像,**可**作为默认的企业级交付物。

硬件 SKU(导轨式设备)**可**基于 **B** 或 **C** 类进行封装;机械/电气部分**应**在硬件 ICD 补充文档中说明。

---

## 9. 安全运维(v1 要求)

在接口安全(TLS 1.2+、令牌、RBAC)基础上,扩展以下**运维**要求:

| ID | 要求 |
| --- | --- |
| SEC-OPS-001 | 记录**证书配发**模式(客户 PKI 与出厂预置方式) |
| SEC-OPS-002 | **令牌生命周期**:签发、刷新、吊销;最长有效期策略 |
| SEC-OPS-003 | API 密钥及 TLS 私钥的**密钥轮换**流程 |
| SEC-OPS-004 | 每次发布的**SBOM**(SPDX 或 CycloneDX 格式) |
| SEC-OPS-005 | **CVE 响应**:分级处置时限(SLA)、补丁节奏、客户通知路径 |
| SEC-OPS-006 | **第三方依赖治理**:注册表白名单、许可证扫描关卡 |

---

## 10. 兼容性与弃用策略(摘要)

**完整矩阵:** 见 **QNC-ENGINEERING-DELIVERABLES-v1** §策略部分。

| 接口面 | 策略 |
| --- | --- |
| 机器人侧 REST/WS | 主版本(MAJOR)= 破坏性变更;次版本(MINOR)= 向后兼容的新增内容 |
| 北向 REST/WS | 采取更严格的外部通知机制;每次发布至少记录 **N-1** 客户端支持情况 |
| YAML 设备配置文件 | 语义化版本管理;主版本变更配套迁移工具 |
| DDS 扩展包 IDL | 按 QNC-IDL 规定;**不**依赖中间件自身的演进安全性 |
| QoS 目录 | 次版本追加式变更;不兼容的协商**将被阻断**并触发故障 |

---

## 11. 平台兼容性矩阵(治理)

作为**受控工件**维护(电子表格或 YAML 注册表):

| 维度 | 行内容 |
| --- | --- |
| 机器人系列 | MAiRA、LARA、MAV(及已测试固件版本) |
| QNC 核心版本 | x.y.z |
| 已安装扩展包 | 工业扩展包/DDS 扩展包模块及版本 |
| 配置文件包 ID | 已验证的组合 |
| 网络/操作系统 | 已批准的操作系统内核、容器运行时 |

**规则:** 未配备矩阵行及测试证据引用,不得对外宣称兼容性。

---

## 12. 配置治理域(相互分离)

| 域 | 工件类型 | 不得 |
| --- | --- | --- |
| 设备配置文件 | 按 DPS 规定的 YAML | 嵌入 DDS IDL 或 QoS |
| 系统配置 | YAML 系统配置模式 | 编码设备语义映射 |
| DDS 扩展包 IDL | IDL + 注册表清单 | 编码 Modbus 寄存器映射 |
| DDS QoS 目录 | 按批准的 YAML/JSON | 混入设备配置文件验证规则 |
| 桥接映射 | 独立治理的软件包 | 暗示在无桥接软件包情况下核心具备该能力 |
| OpenAPI 契约 | 机器人侧与北向拆分 | 单一未区分的"API" |

---

## 13. 安全定位

保持不变:**QNC 不是安全等级设备。**安全模式是**操作性抑制**,而非安全功能。

---

## 14. 验证(v1)

核心验证**应**包括:配置文件 YAML 验证;原子化配置回滚测试;按协议进行的南向符合性测试;故障/安全模式测试;**双 API** 契约测试;按 §7 规定的容量浸泡测试。

DDS 扩展包验证**仅**在该软件包纳入范围时进行。

---

## 15. 风险与未决决策

参见 **QNC-ENGINEERING-DELIVERABLES-v1** §风险部分的综合登记表。

---

## 16. 治理

对 **v1 基线范围**、**三层模型**、**语义核心关联**、**安全定位**或**兼容性策略**的变更,均需按项目规则经过正式变更控制委员会(CCB)审批。

---

## 17. 术语表

| 术语 | 定义 |
| --- | --- |
| **QNC 核心** | 不含可选软件包的强制性网关能力 |
| **工业扩展包** | 可选的南向/北向协议软件包 |
| **DDS 扩展包** | 可选的 FastDDS 扩展软件包 |
| **NEURA 单元** | 已部署的机器人 + 外围设备 + QNC 集成边界 |

---

## 18. v1 执行摘要

QNC v1 是一款**聚焦的工业边缘网关**:经验证的 **YAML** 配置文件、**规范语义**、**REST/WebSocket/日志**、**安全模式**、**原子化配置**,并与 **MAiRA/LARA/MAV** 保持一致。**DDS 扩展包**、**OPC UA**、**MQTT** 及其他现场总线均为**可选**、**隔离**且**非核心基础**的组成部分。

---

## 相关文档

| ID | 标题 |
| --- | --- |
| QNC-SEMANTIC-CORE-v1 | 规范语义模型 |
| ICD\_v3.0 | 接口控制文档 |
| DPS\_v2.0 | 设备配置文件规范 |
| QNC-PSM\_v1.0 | 协议与服务模块规范 |
| QNC-IDL\_v2.0 | DDS 扩展包 IDL 注册表与治理 |
| QNC-ORG\_v1.1 | 运维与恢复指南 |
| QNC-ENGINEERING-DELIVERABLES-v1 | 变更日志、矩阵、策略、风险 |



---
---
---
# QNC\-PRD

# QNC Product Requirements & System Definition Specification

**Document Number:** QNC-PRD-001    
**Version:** 3.0 (v1 product baseline)    
**Status:** Implementation-ready baseline — supersedes PRS v2.3 for v1 scope    
**Classification:** Internal / Project Controlled  

---

## Document Change Log (v2.3 → v3.0)

| Area | Change |
| --- | --- |
| Product definition | Reframed as **software-first industrial edge connectivity gateway** for NEURA cells; explicit **non-goals** reinforced |
| Architecture | Strict **three-tier** product model: **Core**, **Industrial Extensions**, **DDS Pack** |
| DDS | FastDDS is **optional QNC DDS Pack** only; never mandatory platform infrastructure |
| Profiles / config | **YAML** authoritative for authoring/governance; JSON only where runtime requires |
| Semantics | **Single canonical model** — externalized to **QNC-SEMANTIC-CORE-v1** |
| Discovery | Removed universal discovery requirement; **protocol-specific capability matrices** |
| v1 scope | **Normative removal** of Discovery Server, fleet aggregation, SCADA/MES/ERP, EtherCAT/PROFINET v1 claims, broad DDS topologies, generic vendor plugin systems |
| Portfolio | Primary alignment: **MAiRA, LARA, MAV**; MiPA / 4NE1 **secondary / future** |
| New | Packaging, capacity limits, security operations, **dual API versioning**, platform compatibility matrix, compatibility/deprecation policy summaries |

---

## 1. Purpose

This specification defines **product purpose**, **v1 scope**, **architecture**, **functional and non-functional requirements**, **governance**, **packaging**, **capacity**, **security operations**, **compatibility policy**, and **validation expectations** for **QNC (Quick Network Connector)**.

Companion normative artifacts:

| Document | Role |
| --- | --- |
| **QNC-SEMANTIC-CORE-v1** | Canonical Command / Response / Telemetry / Fault / Lifecycle semantics |
| **ICD** | Interface contracts, OpenAPI/WS bindings |
| **DPS** | YAML device profile schema and governance |
| **PSM** | Protocol modules, DDS Pack isolation, capability matrices |
| **QNC-IDL** | DDS Pack IDL registry and governance (extension only) |
| **ORG** | Operations, recovery, commissioning |

---

## 2. Product Definition (v1)

### 2.1 Formal Definition

**QNC is a modular, software-first industrial edge connectivity gateway** for **NEURA** robot cells and adjacent automation environments. It provides **southbound** industrial/device integration, **YAML-governed device profiles**, **semantic normalization** to the canonical core, **robot-facing** and **northbound** APIs (REST, WebSocket, structured logging), **Safe Mode** with **fault isolation**, and **atomic configuration activation with rollback**.

### 2.2 Explicit Non-Goals

QNC **is not** and v1 **must not** be positioned as:

- a robot motion **controller**,
- a **ROS/ROS2 replacement** or universal robotics **middleware**,
- a generic **plant/cloud/MES/SCADA** platform,
- a **universal protocol translation** layer,
- a **functional safety** controller or safety-rated device.

### 2.3 Portfolio Alignment

**Primary integration references (v1):** **MAiRA**, **LARA**, **MAV** — reuse patterns for fieldbus/tooling, ROS2-related integration, Python (NeuraPy) workflows, and peripheral models.

**Secondary / future:** **MiPA**, **4NE1** — integration targets only after interface maturity and validation artifacts exist.

**v1 standardization focus:** EOAT and industrial peripherals; **mobile / ROS2 integration pathways** as **documented patterns**, not an unbounded middleware commitment.

### 2.4 Reuse Principles

Prioritize **NeuraPy**, existing **YAML** configuration approaches, commissioning UX, tool/gripper profile models, and established ROS2 integration pathways. **Do not** introduce parallel SDKs, duplicate profile ecosystems, or competing operational tooling.

---

## 3. Three-Tier Product Model (Normative)

### 3.1 QNC Core (Mandatory v1 Product)

Contains **only**:

| Capability | Description |
| --- | --- |
| Southbound integration | Released industrial/device protocols for v1 baseline (see §5) |
| Profile governance | YAML profiles validated, signed, lifecycle-managed |
| Semantic normalization | Maps southbound data **to** QNC-SEMANTIC-CORE-v1 |
| Robot-facing API | REST/WebSocket (versioned independently) |
| Northbound API | REST/WebSocket/logging for observability (versioned independently) |
| Safe Mode & fault isolation | Operational inhibit; southbound vs. northbound vs. service domains |
| Configuration management | **Atomic** activate, **rollback**, audit |

**Core MUST NOT depend on** DDS, OPC UA, or MQTT runtime presence.

### 3.2 QNC Industrial Extensions (Optional Packages)

Optional, separately released modules, **isolated** from Core compile/runtime where feasible:

| Extension (illustrative) | Note |
| --- | --- |
| Modbus TCP | Extension package |
| CANopen | Extension package |
| OPC UA Server | Extension — **not** v1 Core normative requirement |
| MQTT Client | Extension — **not** v1 Core normative requirement |

Plant/MES/ERP/SCADA integration **is out of v1 normative scope** even when OPC UA/MQTT exist as extensions (no mandated enterprise semantics).

### 3.3 QNC DDS Pack (Optional Package)

**Optional** package containing:

- FastDDS participation (DomainParticipant),
- Governed IDL and QoS artifacts (**separate** governance from device profiles),
- Validated, **narrow** DDS bridges if explicitly released,
- **Isolated DDS fault domain** (`fault_domain = dds_pack` per semantic core).

DDS Pack **must** be load-fault- and upgrade-isolated from Core southbound operation.

---

## 4. Problem Statement (Abbreviated)

Industrial EOAT and peripherals expose heterogeneous protocols and semantics. Robot application teams need **fast, validated integration** without owning every low-level stack. QNC provides a **governed boundary** and **normalized semantics** aligned with NEURA ecosystem tooling.

---

## 5. v1 Baseline Scope

### 5.1 v1 Southbound (Core Baseline)

Normative **v1 baseline** southbound protocols:

- **IO-Link** (v1.1 master),
- **Modbus RTU** (serial),
- **EtherNet/IP** (adapter, Class 1 / Class 3 as released),
- **Discrete digital I/O**.

### 5.2 v1 Northbound (Core Baseline)

- **REST** (HTTPS, JSON),
- **WebSocket** (JSON),
- **Structured logging** (e.g., JSON lines; syslog mapping as deployment option).

### 5.3 Explicitly Non-Normative for v1 (Roadmap Only)

The following **MUST NOT** appear as v1 mandatory requirements:

| Item | Classification |
| --- | --- |
| DDS Discovery Server | Roadmap / future architecture note |
| Fleet-level aggregation | Roadmap |
| SCADA / MES / ERP integration | Roadmap / out of product scope for v1 |
| EtherCAT, PROFINET | Roadmap; hardware-dependent |
| Broad multi-hop DDS bridge topologies | Roadmap |
| Universal vendor extension/plugin runtime | Roadmap |
| “Universal” plant/cloud connectivity assumptions | Non-v1 |

They **MAY** appear in **Appendix — Future architecture** in companion docs; they **MUST NOT** be baseline acceptance criteria for v1.

### 5.4 Protocol-Specific Discovery / Identification

**There is no universal “discovery” requirement.**

Each southbound protocol **SHALL** document a **capability matrix** in the PSM:

| Capability | IO-Link | Modbus RTU | EtherNet/IP | DIO |
| --- | --- | --- | --- | --- |
| Auto-discovery / enumeration | Per PSM | N/A — addressing required | Per PSM | N/A |
| Identification / binding | Yes | Unit ID + optional ID registers | Per CIP identity | Channel map |
| Hot-plug tolerance | Per PSM | Per PSM | Per PSM | Per PSM |

*(Illustrative — authoritative detail in PSM.)*

---

## 6. Functional Requirements (v1 Core Highlights)

Requirements use **REQ-CORE-xxx** for Core, **REQ-EXT-xxx** for extensions (non-v1 baseline unless package installed).

| ID | Statement | Priority |
| --- | --- | --- |
| REQ-CORE-010 | Load and validate **YAML** device profiles; reject invalid; support rollback | High |
| REQ-CORE-011 | Map southbound devices to **QNC-SEMANTIC-CORE-v1** command/telemetry/fault | High |
| REQ-CORE-020 | Expose **robot-facing** REST/WS per ICD; **versioned** independently | High |
| REQ-CORE-021 | Expose **northbound** REST/WS/logs per ICD; **versioned** independently | High |
| REQ-CORE-030 | **Atomic** configuration activation; failed activation **MUST NOT** partial apply | High |
| REQ-CORE-031 | Persist validated config; restore after restart within **≤ 30 s** cold-start budget (see §8) | High |
| REQ-CORE-040 | Enter **Safe Mode** on defined critical faults; **inhibit actuation**; publish faults | High |
| REQ-CORE-041 | Southbound faults **SHALL NOT** be conflated with DDS Pack faults | High |

**DDS Pack** (when installed): optional REQ-DDS-\* set as defined in PSM and QNC-IDL — **not** part of Core conformance.

---

## 7. Non-Functional Requirements & Capacity (v1)

### 7.1 Default v1 Capacity Targets (Tunable per SKU)

| Parameter | v1 default target | Notes |
| --- | --- | --- |
| Max southbound device endpoints | **8** active | Across all protocol instances |
| Max concurrent REST sessions | **16** total | Split policy robot vs northbound in deployment guide |
| Max WebSocket client connections | **8** | Configurable lower |
| DDS Pack subscribed topics | **≤ 20** | Only when DDS Pack installed |
| Stored device profiles (YAML) | **≥ 50** | On-disk governance store |
| Fault history retained | **≥ 1000** events or **7 days** whichever first | Configurable |
| Log rotation | **100 MB** default file set | **≥ 7 days** retention typical |

### 7.2 Performance (Design Targets — Evidence per V&V)

| ID | Target |
| --- | --- |
| NFR-001 | REST command → device: **≤ 50 ms** p95 (profiled scenarios) |
| NFR-002 | Telemetry **≥ 10 Hz** per device default class |
| NFR-003 | Status query p95 **≤ 10 ms** (local network) |
| NFR-004 | Safe Mode entry **≤ 1 s** from critical fault |

### 7.3 Degradation

Under overload, QNC **SHOULD**: shed lower-priority northbound traffic, preserve fault/Safe Mode paths, preserve actuation inhibit semantics. Exact behavior **SHALL** be documented in PSM per module.

---

## 8. Packaging Definition (v1)

QNC v1 **SHALL** support at least one of the following deployment classes (product packaging may combine):

| Class | Description |
| --- | --- |
| **A — Software-only** | Container or bare-metal package on approved industrial Linux |
| **B — Appliance image** | Verified OS + QNC + extensions as a **reference stack** |
| **C — Embedded runtime** | BSP-integrated binary for approved embedded target |

**Certified reference stack:** A **B** image with SBOM, pinned versions, and release notes **MAY** be positioned as the default enterprise deliverable.

Hardware SKU (DIN rail device) **MAY** wrap **B** or **C**; mechanical/power **SHALL** be specified in hardware ICD supplement.

---

## 9. Security Operations (v1 Requirements)

Extends interface security (TLS 1.2+, tokens, RBAC) with **operational** requirements:

| ID | Requirement |
| --- | --- |
| SEC-OPS-001 | **Certificate provisioning** model documented (customer PKI vs factory) |
| SEC-OPS-002 | **Token lifecycle**: issuance, refresh, revocation; max lifetime policy |
| SEC-OPS-003 | **Secrets rotation** procedure for API keys and TLS private keys |
| SEC-OPS-004 | **SBOM** (SPDX or CycloneDX) per release artifact |
| SEC-OPS-005 | **CVE response**: triage SLA, patching cadence, customer notification path |
| SEC-OPS-006 | **Third-party dependency governance**: allowlist registries, license scan gate |

---

## 10. Compatibility & Deprecation Policy (Summary)

**Full matrix:** **QNC-ENGINEERING-DELIVERABLES-v1** §Policies.

| Surface | Policy |
| --- | --- |
| Robot-facing REST/WS | MAJOR = breaking; MINOR = backward compatible additions |
| Northbound REST/WS | Stricter external notice; minimum **N-1** client support documented per release |
| YAML device profiles | Semantic versioning; migration tooling for **major** |
| DDS Pack IDL | Per QNC-IDL; **no** reliance on middleware evolution safety |
| QoS catalogs | Additive MINOR; incompatible negotiation **blocked** with fault |

---

## 11. Platform Compatibility Matrix (Governance)

Maintained as **controlled artifact** (spreadsheet or YAML registry):

| Dimension | Rows |
| --- | --- |
| Robot family | MAiRA, LARA, MAV (+ tested firmware) |
| QNC Core version | x.y.z |
| Installed extension packages | Industrial / DDS Pack modules + versions |
| Profile bundle id | Validated combination |
| Network / OS | Approved OS kernel, container runtime |

**Rule:** No marketing claim of compatibility without a matrix row + test evidence reference.

---

## 12. Configuration Governance Domains (Separated)

| Domain | Artifact type | MUST NOT |
| --- | --- | --- |
| Device profiles | YAML per DPS | Embed DDS IDL or QoS |
| System configuration | YAML system config schema | Encode device semantic mappings |
| DDS Pack IDL | IDL + registry manifest | Encode Modbus register maps |
| DDS QoS catalog | YAML/JSON as approved | Mix device profile validation rules |
| Bridge mappings | Separate governed bundle | Imply Core without bridge package |
| OpenAPI contracts | Robot vs Northbound split | Single undifferentiated “API” |

---

## 13. Safety Position

Unchanged: **QNC is not safety-rated.** Safe Mode is **operational inhibit**, not a safety function.

---

## 14. Validation (v1)

Core validation **SHALL** include: profile YAML validation; atomic config rollback tests; southbound conformance per protocol; fault/Safe Mode; **dual API** contract tests; capacity soak **per §7**.

DDS Pack validation **only** when package is in scope.

---

## 15. Risks & Open Decisions

See **QNC-ENGINEERING-DELIVERABLES-v1** §Risks for the consolidated register.

---

## 16. Governance

Changes to **v1 baseline scope**, **three-tier model**, **semantic core linkage**, **safety position**, or **compatibility policy** require formal CCB per project rules.

---

## 17. Glossary

| Term | Definition |
| --- | --- |
| **QNC Core** | Mandatory gateway capabilities without optional packages |
| **Industrial Extensions** | Optional southbound/northbound protocol packages |
| **DDS Pack** | Optional FastDDS extension package |
| **NEURA cell** | Deployed robot + peripherals + QNC integration boundary |

---

## 18. v1 Executive Summary

QNC v1 is a **focused industrial edge gateway**: validated **YAML** profiles, **canonical semantics**, **REST/WebSocket/logging**, **Safe Mode**, **atomic config**, aligned with **MAiRA/LARA/MAV**. **DDS Pack**, **OPC UA**, **MQTT**, and additional fieldbuses are **optional**, **isolated**, and **non-foundational** to Core.

---

## Related Documents

| ID | Title |
| --- | --- |
| QNC-SEMANTIC-CORE-v1 | Canonical semantic model |
| ICD\_v3.0 | Interface Control Document |
| DPS\_v2.0 | Device Profile Specification |
| QNC-PSM\_v1.0 | Protocol & Service Module Specification |
| QNC-IDL\_v2.0 | DDS Pack IDL registry & governance |
| QNC-ORG\_v1.1 | Operations & Recovery Guide |
| QNC-ENGINEERING-DELIVERABLES-v1 | Changelogs, matrix, policies, risks |
