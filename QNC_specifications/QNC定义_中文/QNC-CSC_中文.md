# QNC 规范语义核心 (v1)

**文档编号:** QNC-CSC-001
**版本:** 1.0
**状态:** 规范性文档 —— 跨接口语义的唯一权威来源
**分类:** 内部 / 项目受控

---

## 1. 目的

本文档定义 QNC v1 的**规范语义模型**。所有外部契约**应**由此核心派生:

- REST/OpenAPI(机器人侧与北向 API 系列)
- WebSocket 事件与流式模式
- 结构化日志与审计事件
- **QNC DDS 扩展包** 的 IDL 类型与主题载荷(当安装了 DDS 扩展包时)
- 设备配置文件归一化目标(南向映射**至**本语义)

**规则:** 其他规范**不得**重新定义相互竞争的命令、响应、遥测、故障或生命周期语义。它们**只能**绑定传输方式、编码、分层和治理。

---

## 2. 语义基元

### 2.1 标识符与关联

| 字段 | 含义 |
| --- | --- |
| `schema_version` | 消息模式绑定的版本(API 或 DDS 扩展包) |
| `message_id` | 本消息实例的唯一 ID |
| `correlation_id` | 将响应、异步完成状态及相关遥测与某条命令关联起来 |
| `timestamp_utc` | ISO 8601 UTC 时间戳 |
| `qnc_instance_id` | 已部署 QNC 实例的标识符 |

### 2.2 命令(规范)

**意图:** 在当前生效的配置文件、模式与策略范围内,请求执行一项受支持的操作。

| 字段 | 是否必需 | 说明 |
| --- | --- | --- |
| `header` | 是 | 通用头部(§2.1) |
| `command_category` | 是 | `profile`(配置文件类)\| `transport`(传输类)\| `management`(管理类) |
| `command_id` | 是 | 归一化命令标识符 |
| `target` | 是 | 目标设备/槽位/端口绑定引用 |
| `parameters` | 是 | 按命令模式限定边界的键值对 |
| `timeout_ms` | 否 | 在配置文件/策略限制内的上限值 |
| `caller_context` | 否 | 来自集成方的不透明、边界受限元数据 |

**前置条件:** 由运行时评估;违反前置条件将产生 `rejected`(拒绝)结果,且不产生任何副作用。

### 2.3 响应(规范)

**意图:** 针对某条命令给出确定性结果(同步或异步完成)。

| 字段 | 是否必需 | 说明 |
| --- | --- | --- |
| `header` | 是 | 通用头部 |
| `request_message_id` | 是 | 原始命令的 `message_id` |
| `completion` | 是 | 见下方枚举 |
| `result_code` | 是 | 稳定的机器可读代码 |
| `details` | 否 | 结构化载荷 |

**完成状态枚举:**
`accepted`(已接受) \| `rejected`(已拒绝) \| `completed`(已完成) \| `completed_with_warning`(带告警完成) \| `failed`(失败) \| `timed_out`(超时) \| `canceled`(已取消) \| `inhibited_by_mode`(因模式受限)

### 2.4 遥测(规范)

**意图:** 时间序列与事件采样的运行数据。

| 字段 | 是否必需 | 说明 |
| --- | --- | --- |
| `header` | 是 | 通用头部 |
| `lifecycle_state` | 是 | 规范生命周期状态(§2.6) |
| `connection_state` | 是 | 南向链路状态 |
| `protocol_state` | 是 | 该协议实例的会话/发现/事务状态 |
| `profile_state` | 是 | 配置文件加载/有效性状态 |
| `device_class_state` | 否 | 设备类别范围内的归一化状态 |
| `vendor_state` | 否 | 原始厂商字段;需明确标注 |
| `health` | 是 | 汇总对象(例如 `ok`(正常)、`degraded`(降级)、`faulted`(故障)) |
| `telemetry_quality` | 否 | `valid`(有效) \| `stale`(过期) \| `estimated`(估计) \| `unsupported`(不支持) \| `faulted`(故障) |

**说明:** DDS 扩展包可将部分字段镜像为 DDS 样本;字段**应**始终与本结构保持语义一致。

### 2.5 故障(规范)

**意图:** 面向集成与运维的结构化故障报告。

| 字段 | 是否必需 | 说明 |
| --- | --- | --- |
| `header` | 是 | 通用头部 |
| `fault_code` | 是 | 稳定的故障代码 |
| `severity` | 是 | `info`(信息) \| `warning`(告警) \| `error`(错误) \| `critical`(严重) |
| `fault_domain` | 是 | `configuration`(配置) \| `profile`(配置文件) \| `southbound_connection`(南向连接) \| `southbound_protocol`(南向协议) \| `southbound_device`(南向设备) \| `command`(命令) \| `service`(服务) \| `security`(安全) \| `northbound`(北向) \| `robot_api`(机器人 API) \| `dds_pack`(DDS 扩展包) |
| `latched` | 是 | 是否需要显式恢复操作 |
| `summary` | 是 | 人类可读的摘要 |
| `context` | 否 | 结构化诊断上下文 |
| `recommended_action` | 否 | 操作员处置建议 |

**隔离规则:** `dds_pack` 域内的故障**不得**隐含南向 `southbound_*` 系列的故障语义,反之亦然,除非 PSM 中已明确规定了某个单一的共享资源故障情形。

### 2.6 生命周期(规范)

**意图:** QNC 运行时(而非机器人应用层)的单一生命周期状态机。

**状态:**
`BOOTING`(启动中) → `INITIALIZING`(初始化中) → `READY`(就绪) → `ACTIVE`(运行中) → (`DEGRADED`(降级) ↔ `ACTIVE`) → `SAFE_MODE`(安全模式) | `SERVICE_MODE`(维护模式) → (来自严重锁存条件的 `FAULT_LOCKED`(故障锁定))

| 状态 | 含义 |
| --- | --- |
| `BOOTING` | 上电 / 进程启动 |
| `INITIALIZING` | 配置文件校验、传输链路建立 |
| `READY` | 配置有效;尚未按策略启用执行机构 |
| `ACTIVE` | 正常运行 |
| `DEGRADED` | 部分受限;存在非严重故障 |
| `SAFE_MODE` | 操作性抑制:**不允许新的执行动作**;按策略保留诊断/遥测功能 |
| `SERVICE_MODE` | 维护状态;默认抑制执行动作 |
| `FAULT_LOCKED` | 严重锁存状态;需要显式恢复 |

**安全模式**并**非**功能安全模式(参见 PRS 安全定位说明)。

---

## 3. API 绑定命名空间(版本管理)

| API 系列 | 用途 | 版本管理方式 |
| --- | --- | --- |
| **机器人侧 API** | 来自机器人控制器/单元软件的命令与高频集成 | 独立 MAJOR.MINOR 版本;兼容性策略见 PRS |
| **北向 API** | 面向工厂/云端/IT 消费者的可观测性接口 | 独立 MAJOR.MINOR 版本;对外部消费者采取更严格的弃用策略 |

相同的规范语义;不同的路由前缀、鉴权策略和生命周期节奏。

---

## 4. 编码策略

- **权威的配置与配置文件工件:** 静态存储及 CI 中均采用 YAML。
- **线上格式:** REST/WebSocket/日志按 ICD/OpenAPI 规定采用 JSON。
- **内部运行时:** 实现**可以**将 YAML 编译为优化的内部表示(例如 JSON 二进制、编译后的结构体)。
- **DDS 扩展包:** IDL 类型**应**由上述规范字段生成/映射,而非采用发散的并行模型。

---

## 5. 可追溯性

| 配套文档 | 如何使用本核心 |
| --- | --- |
| PRS | 产品范围、非功能需求、打包、容量 |
| ICD | REST/WS/日志字段与本核心的绑定 |
| DPS | 配置文件将设备原生数据映射到命令/遥测/故障子集 |
| PSM | 模块职责、故障域、协议矩阵 |
| QNC-IDL | DDS 扩展包 IDL 分层与 IDL 规范中的 L2/L3 对齐 |
| ORG | 运维手册引用生命周期与故障域 |

---

## 文档控制

| 版本 | 日期 | 摘要 |
| --- | --- | --- |
| 1.0 | 2026-05-12 | QNC v1 规范语义核心的首个整合版本 |


---
---
---
# QNC\-CSC

# QNC Canonical Semantic Core (v1)

**Document Number:** QNC-CSC-001    
**Version:** 1.0    
**Status:** Normative — single source of truth for cross-interface semantics    
**Classification:** Internal / Project Controlled  

---

## 1. Purpose

This document defines the **canonical semantic model** for QNC v1. All external contracts SHALL derive from this core:

- REST/OpenAPI (robot-facing and northbound API families)
- WebSocket event and stream schemas
- Structured logging and audit events
- **QNC DDS Pack** IDL types and topic payloads (when the DDS Pack is installed)
- Device profile normalization targets (southbound mapping **into** these semantics)

**Rule:** Other specifications SHALL NOT redefine competing command, response, telemetry, fault, or lifecycle semantics. They SHALL only bind transport, encoding, layering, and governance.

---

## 2. Semantic Primitives

### 2.1 Identifiers and Correlation

| Field | Meaning |
| --- | --- |
| `schema_version` | Version of the message schema binding (API or DDS Pack) |
| `message_id` | Unique id for this message instance |
| `correlation_id` | Links response, async completion, and related telemetry to a command |
| `timestamp_utc` | ISO 8601 UTC |
| `qnc_instance_id` | Deployed QNC instance identifier |

### 2.2 Command (canonical)

**Intent:** Request a supported operation within active profile, mode, and policy.

| Field | Required | Description |  |  |
| --- | --- | --- | --- | --- |
| `header` | Yes | Common header (§2.1) |  |  |
| `command_category` | Yes | `profile` \\ | `transport` \\ | `management` |
| `command_id` | Yes | Normalized command identifier |  |  |
| `target` | Yes | Target device/slot/port binding reference |  |  |
| `parameters` | Yes | Bounded key-value per command schema |  |  |
| `timeout_ms` | No | Upper bound within profile/policy limits |  |  |
| `caller_context` | No | Opaque, bounded metadata from integrator |  |  |

**Preconditions:** Evaluated by runtime; violations yield `rejected` without side effects.

### 2.3 Response (canonical)

**Intent:** Deterministic outcome for a command (sync or async completion).

| Field | Required | Description |
| --- | --- | --- |
| `header` | Yes | Common header |
| `request_message_id` | Yes | Original command `message_id` |
| `completion` | Yes | Enumeration below |
| `result_code` | Yes | Stable machine-readable code |
| `details` | No | Structured payload |

**Completion enumeration:**    
`accepted` \\| `rejected` \\| `completed` \\| `completed_with_warning` \\| `failed` \\| `timed_out` \\| `canceled` \\| `inhibited_by_mode`

### 2.4 Telemetry (canonical)

**Intent:** Time-series and event-sampled operational data.

| Field | Required | Description |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- |
| `header` | Yes | Common header |  |  |  |  |
| `lifecycle_state` | Yes | Canonical lifecycle (§2.6) |  |  |  |  |
| `connection_state` | Yes | Southbound link state |  |  |  |  |
| `protocol_state` | Yes | Session/discovery/transaction state **for that protocol instance** |  |  |  |  |
| `profile_state` | Yes | Profile load/validity |  |  |  |  |
| `device_class_state` | No | Class-scoped normalized state |  |  |  |  |
| `vendor_state` | No | Raw vendor fields; clearly labeled |  |  |  |  |
| `health` | Yes | Summary object (e.g., `ok`, `degraded`, `faulted`) |  |  |  |  |
| `telemetry_quality` | No | `valid` \\ | `stale` \\ | `estimated` \\ | `unsupported` \\ | `faulted` |

**Note:** DDS Pack may mirror a subset as DDS samples; fields SHALL remain semantically aligned with this structure.

### 2.5 Fault (canonical)

**Intent:** Structured fault reporting for integration and operations.

| Field | Required | Description |  |  |  |  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `header` | Yes | Common header |  |  |  |  |  |  |  |  |  |  |
| `fault_code` | Yes | Stable fault code |  |  |  |  |  |  |  |  |  |  |
| `severity` | Yes | `info` \\ | `warning` \\ | `error` \\ | `critical` |  |  |  |  |  |  |  |
| `fault_domain` | Yes | `configuration` \\ | `profile` \\ | `southbound_connection` \\ | `southbound_protocol` \\ | `southbound_device` \\ | `command` \\ | `service` \\ | `security` \\ | `northbound` \\ | `robot_api` \\ | `dds_pack` |
| `latched` | Yes | Whether explicit recovery required |  |  |  |  |  |  |  |  |  |  |
| `summary` | Yes | Human-readable |  |  |  |  |  |  |  |  |  |  |
| `context` | No | Structured diagnostic context |  |  |  |  |  |  |  |  |  |  |
| `recommended_action` | No | Operator guidance |  |  |  |  |  |  |  |  |  |  |

**Isolation rule:** Faults in `dds_pack` SHALL NOT imply southbound `southbound_*` fault semantics and vice versa, except where a single documented shared resource failure is explicitly specified in the PSM.

### 2.6 Lifecycle (canonical)

**Intent:** Single lifecycle state machine for QNC runtime (not robot application lifecycle).

**States:**    
`BOOTING` → `INITIALIZING` → `READY` → `ACTIVE` → (`DEGRADED` ↔ `ACTIVE`) → `SAFE_MODE` | `SERVICE_MODE` → (`FAULT_LOCKED` from critical latched conditions)

| State | Meaning |
| --- | --- |
| `BOOTING` | Power-on / process start |
| `INITIALIZING` | Artifact validation, transport bring-up |
| `READY` | Valid config; actuation not yet enabled per policy |
| `ACTIVE` | Normal operation |
| `DEGRADED` | Partial restrictions; non-critical faults |
| `SAFE_MODE` | Operational inhibit: **no new actuation**; diagnostics/telemetry per policy |
| `SERVICE_MODE` | Maintenance; actuation inhibited by default |
| `FAULT_LOCKED` | Critical latched; explicit recovery |

**Safe Mode** is **not** a functional safety mode (see PRS safety position).

---

## 3. API Binding Namespaces (versioning)

| API family | Purpose | Version surface |
| --- | --- | --- |
| **Robot-facing API** | Commands, high-rate integration from robot controller / cell software | Independent MAJOR.MINOR; compatibility policy: PRS |
| **Northbound API** | Plant/cloud/IT consumers, observability | Independent MAJOR.MINOR; stricter deprecation for external consumers |

Same canonical semantics; different route prefixes, auth policies, and lifecycle cadence.

---

## 4. Encoding Policy

- **Authoritative configuration and profile artifacts:** YAML at rest and in CI.  
- **Wire formats:** JSON for REST/WebSocket/logs as specified in ICD/OpenAPI.  
- **Internal runtime:** Implementations MAY compile YAML to optimized internal representation (e.g., JSON binary, compiled structs).  
- **DDS Pack:** IDL types SHALL be generated/mapped from the canonical fields above, not divergent parallel models.

---

## 5. Traceability

| Companion | How it uses this core |
| --- | --- |
| PRS | Product scope, NFRs, packaging, capacity |
| ICD | REST/WS/log field binding to this core |
| DPS | Profile maps device native → Command/Telemetry/Fault subsets |
| PSM | Module responsibilities, fault domains, protocol matrices |
| QNC-IDL | DDS Pack IDL layers align to L2/L3 in IDL spec |
| ORG | Runbooks reference lifecycle and fault domains |

---

## Document Control

| Version | Date | Summary |
| --- | --- | --- |
| 1.0 | 2026-05-12 | Initial consolidated canonical semantic core for QNC v1 |
