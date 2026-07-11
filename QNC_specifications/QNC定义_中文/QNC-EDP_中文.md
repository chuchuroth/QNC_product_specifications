# QNC-EDP

# QNC v1 —— 工程交付物包

**文档编号:** QNC-EDP-001
**版本:** 1.0
**日期:** 2026-05-12
**分类:** 内部 / 项目受控

本文档满足与六份精炼规范一并要求的**跨文档交付物**。**具有权威性的规范文本**存在于 §0 所列的各版本化文档中。

---

## 0. 精炼规范集合(文件索引)

| # | 交付物 | 文件 |
| --- | --- | --- |
| 1 | 产品需求与系统定义 | `PRS_v3.0.md` |
| 2 | 协议与服务模块规范 | `QNC-PSM_v1.0.md` |
| 3 | 接口控制文档 | `ICD_v3.0.md` |
| 4 | 设备配置文件规范 | `DPS_v2.0.md` |
| 5 | DDS 扩展包 —— IDL 注册表与治理 | `QNC-IDL_v2.0.md` |
| 6 | 运维与恢复指南 | `QNC-ORG_v1.1.md` |
| — | **规范语义模型** | `QNC-SEMANTIC-CORE-v1.md` |

**在 v1 规划中已被取代:** `PRS_v2.3.md`、`ICD_v2.3.md`、`DPS_v1.1.md`、`QNC-PSM.md`(草稿 0.1 版)、`QNC-IDL.md`(v1.1 版)、`QNC-ORG-001.md`(草稿 1.0 版) —— 保留以备历史查阅;**未经**差距评审,不得将其用作 v1 基线。

---

## 1. 综合变更日志(按文档)

| 文档 | 精炼内容摘要 |
| --- | --- |
| **PRS v3.0** | 三层级产品架构;DDS 扩展包为可选项;YAML 具权威性;产品组合对齐;v1 范围削减;打包、容量、SEC-OPS、兼容性矩阵治理 |
| **ICD v3.0** | 双 API 接口面;绑定规范语义核心;DDS 扩展包为可选项;发现机制 → 协议矩阵;故障域(`fault_domain`)隔离 |
| **DPS v2.0** | 仅 YAML 具规范性;新增 `semantic_mapping`;配置文件中剔除 DDS;故障 `source_layer` 剔除 DDS |
| **PSM v1.0** | 分层模块;各协议专属矩阵;修订的服务 API;DDS 扩展包隔离;取消通用 discover() |
| **QNC-IDL v2.0** | DDS 扩展包框架;v1 仅支持 SIMPLE 发现方式;语义对齐;厂商扩展降级 |
| **ORG v1.1** | 分层级运维手册;SEC-OPS;DDS 扩展包为可选路径;v1 不含发现服务器(Discovery Server) |
| **语义核心** | **新增** —— 单一的命令/响应/遥测/故障/生命周期定义 |

---

## 2. 跨文档一致性矩阵

| 主题 | PRS v3.0 | ICD v3.0 | DPS v2.0 | PSM v1.0 | QNC-IDL v2.0 | ORG v1.1 | 语义核心 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **命令语义** | 需与 REQ 对齐 | REST/WS 绑定 | `canonical_command_id` | 代理(Broker) | DDS 映射可选 | — | **归其所有** |
| **遥测语义** | 非功能需求目标 | REST/WS 绑定 | `canonical_telemetry_key` | 映射器 | DDS L2 层 | 监控 | **归其所有** |
| **故障语义** | 安全模式策略 | 发布 | 南向 `source_layer` | 故障管理器 | `dds_pack` 域 | 恢复阶梯 | **归其所有**(各故障域) |
| **生命周期** | 安全模式 | ICD §15 | 仅限序列定义 | 生命周期管理器 | 安全 DDS | 安全运维 | **归其所有**(各状态) |
| **YAML 权威性** | §12 治理 | 配置应用 | 配置文件 | 系统 YAML | DDS 包单独管理 | 试运行 | — |
| **DDS 可选** | DDS 扩展包层级 | 仅作扩展 | 配置文件中**禁止** | 扩展包模块 | **归其所有**(DDS) | 可选步骤 | 与 DDS 无关 |
| **发现机制** | 非通用 | §12 矩阵 | `binding_hints` | §5 矩阵 | 不适用(DDS SIMPLE) | 按协议区分 | 不适用 |
| **API 版本管理** | 双重策略 | 两份 OpenAPI 文档 | 兼容性字段 | — | DDS 语义化版本 | — | 命名空间 |
| **非 v1 路线图** | §5.3 | §5.4 | §6 | §7 附录 | 附录 A | §2 DDS 未来规划 | — |

**图例说明:** “**归其所有**”表示规范性语义来源;其他文档仅为引用。

---

## 3. 精炼后的产品架构定义

```text
┌─────────────────────────────────────────────────────────────────────────┐
│                       MAiRA / LARA / MAV 生态系统                        │
│              (运动控制、安全、ROS2 模式、NeuraPy 工具链)                │
└───────────────┬───────────────────────────────┬─────────────────────────┘
                │ 机器人侧 API                   │ 北向 API
                │ (版本 A)                        │ (版本 B)
                ▼                               ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                            QNC 核心                                     │
│  北向边缘层:REST / WebSocket / 结构化日志                                │
│  集成与控制层:命令代理、生命周期、故障(规范语义核心)                     │
│  配置文件层:YAML 校验 → 语义归一化                                       │
│  南向层:IO-Link、Modbus RTU、EtherNet/IP 适配器、离散 IO                  │
└───────┬─────────────────────────────────────────────────────┬──────────┘
        │ 可选软件包                                            │
        ▼                                                       ▼
┌───────────────────┐                                 ┌───────────────────┐
│ 工业扩展包         │                                 │ QNC DDS 扩展包     │
│ Modbus TCP,       │                                 │ FastDDS、IDL、QoS  │
│ CANopen, OPC UA,  │                                 │ 故障隔离           │
│ MQTT, …           │                                 │ v1 仅支持 SIMPLE   │
└───────────────────┘                                 └───────────────────┘
```

**数据平面:** 南向帧 → 配置文件映射 → **规范遥测/故障** → API 分发(+ 可选 DDS 写入器)。
**控制平面:** YAML 系统配置 + 配置文件包 → 原子化生成 → 回滚。
**故障平面:** `fault_domain` 区分机制,防止将 DDS 问题误报为夹爪故障。

---

## 4. 已移除或重新界定范围的功能(v1 规范性内容)

| 原有 / 草案概念 | v1 处置方式 |
| --- | --- |
| 通用设备发现(FR-001 风格) | **已移除**;改为按协议区分的矩阵 |
| 将 FastDDS 作为战略基础 / 差异化优势 | **重新界定**为可选的 **DDS 扩展包** |
| 发现服务器模式(基础版/高级版) | 仅作为**路线图**规划 |
| 将 DDS 到 OPC UA / MQTT 的桥接默认视为基线功能 | 改为**扩展功能**,每个桥接均需明确发布 |
| 集群聚合与集群遥测 | **路线图**规划 |
| SCADA / MES / ERP 集成 | **不属于** v1 产品范围 |
| EtherCAT / PROFINET | **路线图**规划 |
| 通用厂商扩展 DDS 主题 | **降级**处理 / 仅限项目受控 |
| JSON 作为与 YAML 同等地位的配置文件编写格式 | **降级处理**;YAML 具权威性 |
| 单一、未区分的北向"API" | **拆分**为机器人侧与北向两类 |
| 混合的配置文件/DDS 治理机制 | **已分离** |

---

## 5. 兼容性与治理策略(综合)

### 5.1 语义与 ICD 绑定

- **语义核心主版本(MAJOR)**升级需同步进行 ICD 绑定版本升级、映射器更新以及配置文件 `schema_version` 策略调整。
- **ICD 机器人侧与北向**版本相互**独立**;变更日志**应**分别说明对各类消费者的影响。

### 5.2 设备配置文件(YAML)

- 语义**主版本(MAJOR)**变更:命令/遥测含义发生破坏性变化 → 需要新的配置文件主版本及迁移指南。
- **次版本(MINOR)**:新增字段;消费者需容许字段缺失情况。
- **补丁版本(PATCH)**:文档说明/澄清,不涉及行为变化。

### 5.3 DDS 扩展包(IDL / QoS)

- 遵循 **QNC-IDL\_v2.0** 演进规则;假定中间件**不**具备自动的安全演进能力。
- QoS 协商失败 → 触发 `dds_pack` 故障;**不得**静默降级至不兼容的 QoS 等级。

### 5.4 配置包

- **生成 ID(generation id)**单调递增;激活操作**原子化**;**强制**支持回滚至上一代(N-1)。

### 5.5 弃用生命周期

1. 在发布说明中公告弃用计划
2. 针对次版本(MINOR)弃用,**至少保留一个发布周期**的重叠期
3. 仅在遥测数据显示无关键消费者**或**已达合同约定日期后方可正式退役

---

## 6. 遗留风险与未决决策

| ID | 风险/决策 | 建议路径 |
| --- | --- | --- |
| R-01 | 客户现场的**组播策略**可能阻断 DDS SIMPLE 发现机制 | 架构决策:采用 UDP 单播对等列表,或推迟 DDS 扩展包的销售 |
| R-02 | **双 OpenAPI** 维护成本 | 从单一类 IDL 模式生成代码,而非手动同步 |
| R-03 | **容量**默认值与嵌入式 SKU 实际情况的差距 | 制定按 SKU 验证的矩阵表 |
| R-04 | **第三方夹爪**配置文件迭代速度与治理关卡之间的矛盾 | 分级验证类别(开发版 vs 生产签名版) |
| R-05 | **MAiRA/LARA/MAV** 版本错位 | 明确机器人兼容性矩阵的归属(机器人团队 vs QNC 团队) |
| R-06 | **DDS 扩展包**安全性(DDS-Security) | 限期开展评估;若默认禁用,则不作为 v1 阻塞项 |
| R-07 | **OPC UA / MQTT** 在工业扩展包中的定位与云端营销宣传的矛盾 | 将其排除在 v1 **规范性** PRD 之外,以防范范围蔓延 |

---

## 7. 最终可实施的 v1 产品定义

**名称:** QNC Core v1
**一句话概括:** 一个**以软件为核心的工业边缘网关**,面向 NEURA 单元,通过**已发布的南向协议**集成末端工具(EOAT)及外围设备,由**已签名的 YAML 配置文件**治理,通过**版本化的机器人侧与北向 REST/WebSocket/日志 API** 暴露**经语义归一化**的数据,并具备**安全模式**、**故障域隔离**及**原子化配置回滚**能力。

**规范性范围之内:**

- 协议:IO-Link、Modbus RTU、EtherNet/IP 适配器、离散数字 IO
- API:REST + WebSocket + 结构化日志(**两条**版本线)
- 配置文件:YAML,具备语义核心可追溯性
- 运维:SEC-OPS 检查清单、容量目标、回滚机制

**可选打包:**

- **工业扩展包**(例如 Modbus TCP、CANopen、OPC UA、MQTT)
- **QNC DDS 扩展包**(FastDDS、IDL 注册表、SIMPLE 发现、有边界的主题)

**明确排除在 v1 规范范围之外:**

- 机器人控制器替代方案、功能安全、通用中间件、工厂级/MES/SCADA、集群平台、EtherCAT/PROFINET、发现服务器、通用发现机制

**主要产品组合对齐对象:** MAiRA、LARA、MAV —— 复用 NeuraPy 及现有集成模式。

---

## 8. 可追溯性说明

**PRS v3.0** 中的需求映射至(独立文档)V&V(验证与确认)计划中的验证用例;本 EDP 文档不替代正式的**需求 ↔ 测试** ID 对应关系。

---

## 文档控制

| 版本 | 日期 | 编制说明 |
| --- | --- | --- |
| 1.0 | 2026-05-12 | 综合工程精炼版本 |


---
---
---
# QNC\-EDP

# QNC v1 — Engineering Deliverables Package

**Document Number:** QNC-EDP-001    
**Version:** 1.0    
**Date:** 2026-05-12    
**Classification:** Internal / Project Controlled  

This package satisfies the **cross-cutting deliverables** requested alongside the six refined specifications. **Authoritative specification text** lives in the versioned documents listed in §0.

---

## 0. Refined Specification Set (File Index)

| # | Deliverable | File |
| --- | --- | --- |
| 1 | Product Requirements & System Definition | `PRS_v3.0.md` |
| 2 | Protocol & Service Module Specification | `QNC-PSM_v1.0.md` |
| 3 | Interface Control Document | `ICD_v3.0.md` |
| 4 | Device Profile Specification | `DPS_v2.0.md` |
| 5 | DDS Pack — IDL Registry & Governance | `QNC-IDL_v2.0.md` |
| 6 | Operations & Recovery Guide | `QNC-ORG_v1.1.md` |
| — | **Canonical semantic model** | `QNC-SEMANTIC-CORE-v1.md` |

**Superseded for v1 planning:** `PRS_v2.3.md`, `ICD_v2.3.md`, `DPS_v1.1.md`, `QNC-PSM.md` (draft 0.1), `QNC-IDL.md` (v1.1), `QNC-ORG-001.md` (draft 1.0) — retain for history; **do not** use for v1 baseline without gap review.

---

## 1. Consolidated Change Log (By Document)

| Document | Summary of refinement |
| --- | --- |
| **PRS v3.0** | Three-tier product; DDS Pack optional; YAML authority; portfolio alignment; v1 scope cuts; packaging, capacity, SEC-OPS, compatibility matrix governance |
| **ICD v3.0** | Dual API surfaces; semantic core binding; DDS Pack optional; discovery → protocol matrices; fault\_domain isolation |
| **DPS v2.0** | YAML-only normative; `semantic_mapping`; DDS expunged from profiles; fault source\_layer DDS removal |
| **PSM v1.0** | Tiered modules; per-protocol matrices; revised service API; DDS Pack isolation; no universal discover() |
| **QNC-IDL v2.0** | DDS Pack framing; SIMPLE-only v1; semantic alignment; vendor extension demotion |
| **ORG v1.1** | Tier-aware runbooks; SEC-OPS; DDS Pack optional path; no Discovery Server v1 |
| **Semantic Core** | **New** — single Command/Response/Telemetry/Fault/Lifecycle definition |

---

## 2. Cross-Document Consistency Matrix

| Topic | PRS v3.0 | ICD v3.0 | DPS v2.0 | PSM v1.0 | QNC-IDL v2.0 | ORG v1.1 | Semantic Core |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **Command semantics** | Requires REQ alignment | REST/WS bind | `canonical_command_id` | Broker | DDS mapping optional | — | **Owns** |
| **Telemetry semantics** | NFR targets | REST/WS bind | `canonical_telemetry_key` | Mapper | DDS L2 | Monitoring | **Owns** |
| **Fault semantics** | Safe Mode policy | Publication | Southbound `source_layer` | Fault mgr | `dds_pack` domain | Recovery ladder | **Owns** domains |
| **Lifecycle** | Safe Mode | ICD §15 | Sequences only | Lifecycle mgr | Safe DDS | Safe ops | **Owns** states |
| **YAML authority** | §12 governance | Config apply | Profiles | System YAML | DDS bundle separate | Commissioning | — |
| **DDS optional** | DDS Pack tier | Extension only | **Forbidden** in profile | Pack module | **Owns** DDS | Optional steps | DDS-agnostic |
| **Discovery** | Not universal | §12 matrices | `binding_hints` | §5 matrices | N/A (DDS SIMPLE) | Per-protocol | N/A |
| **API versioning** | Dual policy | Two OpenAPIs | compat fields | — | DDS semver | — | Namespaces |
| **Non-v1 roadmap** | §5.3 | §5.4 | §6 | §7 appendix | Appendix A | §2 DDS future | — |

**Legend:** “**Owns**” = normative semantic source; others reference.

---

## 3. Refined Product Architecture Definition

```text
┌─────────────────────────────────────────────────────────────────────────┐
│                         MAiRA / LARA / MAV ecosystem                     │
│              (motion, safety, ROS2 patterns, NeuraPy tooling)           │
└───────────────┬───────────────────────────────┬─────────────────────────┘
                │ Robot-facing API              │ Northbound API
                │ (version A)                   │ (version B)
                ▼                               ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                            QNC Core                                      │
│  Northbound Edge: REST / WebSocket / structured logs                     │
│  Integration & Control: Command broker, lifecycle, fault (semantic core) │
│  Profile layer: YAML validate → semantic normalization                   │
│  Southbound: IO-Link, Modbus RTU, EtherNet/IP adapter, DIO               │
└───────┬─────────────────────────────────────────────────────┬──────────┘
        │ optional packages                                     │
        ▼                                                       ▼
┌───────────────────┐                                 ┌───────────────────┐
│ Industrial Ext.   │                                 │ QNC DDS Pack       │
│ Modbus TCP,       │                                 │ FastDDS, IDL, QoS  │
│ CANopen, OPC UA,  │                                 │ isolated faults    │
│ MQTT, …           │                                 │ SIMPLE v1          │
└───────────────────┘                                 └───────────────────┘
```

**Data plane:** Southbound frames → profile mappings → **canonical telemetry/fault** → API fan-out (+ optional DDS writers).    
**Control plane:** YAML system config + profile bundle → atomic generation → rollback.    
**Fault plane:** `fault_domain` discrimination prevents DDS issues from being mis-reported as gripper faults.

---

## 4. Removed or Re-Scoped Features (v1 Normative)

| Former / draft concept | v1 disposition |
| --- | --- |
| Universal device discovery (FR-001 style) | **Removed**; per-protocol matrix |
| FastDDS as strategic baseline / differentiator | **Re-scoped** to optional **DDS Pack** |
| Discovery Server mode (baseline/advanced) | **Roadmap** only |
| DDS-to-OPC UA / MQTT bridging as implied baseline | **Extension** + explicit release per bridge |
| Fleet aggregation & cluster telemetry | **Roadmap** |
| SCADA / MES / ERP integration | **Out of v1** product scope |
| EtherCAT / PROFINET | **Roadmap** |
| Generic vendor extension DDS topics | **Demoted** / project-controlled only |
| JSON as co-equal profile authoring | **Demoted**; YAML authoritative |
| Single undifferentiated northbound “API” | **Split** robot vs northbound |
| Mixed profile/DDS governance | **Separated** |

---

## 5. Compatibility & Governance Policies (Consolidated)

### 5.1 Semantic & ICD Binding

- **Semantic core MAJOR** bump requires coordinated ICD binding version + mapper updates + profile `schema_version` policy.  
- **ICD robot vs northbound** versions **independent**; changelog **SHALL** state consumer impact separately.

### 5.2 Device Profiles (YAML)

- Semantic **MAJOR**: breaking command/telemetry meaning → new profile major; migration guide.  
- **MINOR**: additive fields; consumers tolerate absence.  
- **PATCH**: documentation/clarifications without behavior change.

### 5.3 DDS Pack (IDL / QoS)

- Follow **QNC-IDL\_v2.0** evolution rules; assume **no** middleware automatic safe evolution.  
- QoS negotiation failure → `dds_pack` fault; **no** silent downgrade to incompatible QoS.

### 5.4 Configuration Bundles

- **Generation id** monotonic; activation **atomic**; rollback to **N-1** generation mandatory.

### 5.5 Deprecation Lifecycle

1. Announce deprecation in release notes  
2. Maintain **≥ 1 release** overlap for MINOR deprecations  
3. Retire only after telemetry shows zero critical consumers **or** contractual date  

---

## 6. Outstanding Risks & Unresolved Decisions

| ID | Risk / decision | Suggested path |
| --- | --- | --- |
| R-01 | **Multicast policy** in customer sites blocks DDS SIMPLE | Architecture decision: UDP unicast peer list vs delay DDS Pack sale |
| R-02 | **Dual OpenAPI** maintenance cost | Codegen from single IDL-like schema vs manual sync |
| R-03 | **Capacity** defaults vs embedded SKU reality | Per-SKU verified matrix row |
| R-04 | **Third-party gripper** profile velocity vs governance gates | Tiered validation classes (dev vs production signed) |
| R-05 | **MAiRA/LARA/MAV** version skew | Robot compatibility matrix ownership (robot team vs QNC team) |
| R-06 | **DDS Pack** security (DDS-Security) | Time-boxed assessment; not v1 blocker if disabled by default |
| R-07 | **OPC UA / MQTT** in Industrial Extensions vs cloud marketing | Keep out of v1 **normative** PRD to prevent scope creep |

---

## 7. Final Implementation-Ready v1 Product Definition

**Name:** QNC Core v1    
**One-line:** A **software-first industrial edge gateway** for NEURA cells integrating EOAT and peripherals over **released southbound protocols**, governed by **signed YAML profiles**, exposing **semantically normalized** data via **versioned robot-facing and northbound REST/WebSocket/logging APIs**, with **Safe Mode**, **fault-domain isolation**, and **atomic configuration rollback**.

**In scope (normative):**

- Protocols: IO-Link, Modbus RTU, EtherNet/IP adapter, discrete DIO  
- APIs: REST + WebSocket + structured logging (**two** version lines)  
- Profiles: YAML, semantic core traceability  
- Ops: SEC-OPS checklist, capacity targets, rollback  

**Packaged optionally:**

- **Industrial Extensions** (e.g., Modbus TCP, CANopen, OPC UA, MQTT)  
- **QNC DDS Pack** (FastDDS, IDL registry, SIMPLE discovery, bounded topics)

**Explicitly out of v1 normative scope:**

- Robot controller replacement, functional safety, universal middleware, plant/MES/SCADA, fleet platforms, EtherCAT/PROFINET, Discovery Server, universal discovery

**Primary portfolio alignment:** MAiRA, LARA, MAV — reuse NeuraPy and existing integration patterns.

---

## 8. Traceability Note

Requirements in **PRS v3.0** map to verification cases in the (separate) V&V plan; this EDP does not replace formal **req ↔ test** IDs.

---

## Document Control

| Version | Date | Authoring |
| --- | --- | --- |
| 1.0 | 2026-05-12 | Consolidated engineering refinement |
