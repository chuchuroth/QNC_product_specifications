# QNC-ICD

# QNC 接口控制文档

**文档编号:** QNC-ICD-001
**版本:** 3.0(v1 基线)
**状态:** 作为 **QNC-PRD-001 v3.0** 的规范性配套文档
**分类:** 内部 / 项目受控

---

## 文档变更日志(v2.3 → v3.0)

| 领域 | 变更内容 |
| --- | --- |
| 语义 | 所有消息契约**规范性地引用** **QNC-SEMANTIC-CORE-v1** |
| API | **机器人侧** vs **北向** API:**各自独立**的版本管理与 OpenAPI 工件 |
| DDS | 仅限于 **QNC DDS 扩展包**;**从不**作为核心 ICD 的基线承诺 |
| 发现机制 | §12 重写:**按协议区分的矩阵**;移除 PRS 中 FR-001 风格的通用发现机制 |
| 路线图 | ICD 文本中明确将 EtherCAT/PROFINET/发现服务器/集群/SCADA 列为**非 v1** 范围 |
| 附录 | 字段名称与语义核心对齐(`fault_domain`、`qnc_instance_id`) |

---

## 1. 目的与范围

本 ICD 定义 QNC 的**外部接口契约**:消息形态、状态行为、故障发布、配置接口,以及逻辑通道到 REST、WebSocket 与日志的**绑定**关系。

**规范性语义来源:** **QNC-SEMANTIC-CORE-v1** —— 本 ICD 仅定义传输方式、路径、错误代码及**具体实现**。

**从属于:** QNC-PRD-001 v3.0。

---

## 2. 规范性用语

SHALL / SHOULD / MAY / MUST NOT,遵循 PRS 中 RFC 2119 风格的用法。

---

## 3. 相关文档

- QNC-PRD-001 v3.0
- QNC-SEMANTIC-CORE-v1
- DPS\_v2.0
- QNC-PSM\_v1.0
- QNC-IDL\_v2.0(DDS 扩展包)
- QNC-ORG\_v1.1

---

## 4. 系统上下文与归属

### 4.1 上下文

**机器人 / 单元软件** ↔ **机器人侧 API** ↔ **QNC 核心** ↔ **南向设备**
**北向消费者** ↔ **北向 API** ↔ **QNC**(相同语义核心,**不同的** API 接口面与策略)

### 4.2 归属

| 归属方 | 职责 |
| --- | --- |
| QNC | 南向会话、配置文件执行、归一化处理、发布/订阅**已批准的**主题、配置完整性 |
| 机器人控制器 / MAiRA / LARA / MAV 系统 | 运动控制、安全、应用序列编排 |
| 设备厂商 | 设备内部行为 |
| DDS 域(使用 DDS 扩展包时) | 归属于机器人中间件;QNC 作为**访客参与者** |

---

## 5. v1 接口范围

### 5.1 核心南向接口(v1)

IO-Link、Modbus RTU、EtherNet/IP 适配器、离散数字 IO —— **按 PRD §5 执行**。

### 5.2 核心北向接口(v1)

- **机器人侧:** REST + WebSocket(`/robot/v{MAJOR}` 前缀模式 —— 具体细节见 OpenAPI 工件)
- **北向:** REST + WebSocket + 结构化日志(`/nb/v{MAJOR}` 或按主机分离部署)

**应:** 提供各自独立的 OpenAPI 文档:`qnc-robot-api-vMAJOR.yaml`、`qnc-northbound-api-vMAJOR.yaml`。

### 5.3 DDS 扩展包

**仅当**安装并启用 DDS 扩展包时:按 **QNC-IDL\_v2.0** 提供 DDS 对等接口。**不属于**核心(Core)基线符合性要求。

### 5.4 非 v1 ICD 声明

ICD **不得**要求以下内容作为核心符合性条件:OPC UA、MQTT、发现服务器、集群 API、SCADA 适配器、EtherCAT/PROFINET 协议栈。

---

## 6. 逻辑通道 → 规范核心

| 逻辑通道 | 规范绑定 |
| --- | --- |
| `qnc.command` | §2.2 命令 |
| `qnc.response` | §2.3 响应 |
| `qnc.telemetry` | §2.4 遥测 |
| `qnc.fault` | §2.5 故障 |
| `qnc.lifecycle` | §2.6 生命周期 |
| `qnc.service` | 管理/诊断操作(命令类别中 `management` 的子集 + 只读资源) |

---

## 7. 物理与电气

沿用 v2.3 版本的原始意图:硬件补充文档定义连接器/部件细节。

---

## 8. REST 与 WebSocket

### 8.1 JSON 模式规则

请求/响应体**应**为与语义核心字段兼容的 JSON 对象;`schema_version`**应**标识 ICD 绑定版本(区别于 PRD 产品版本)。

### 8.2 错误模型(HTTP)

HTTP 状态码及消息体**应**包含与**附录 B** 目录对齐的机器可读 `result_code`;不得与 `completion`(完成状态)语义相矛盾。

---

## 9. 命令模型

命令类别及完成状态集合与语义核心 §2.2–2.3 **保持一致**。ICD **不得**新增替代性的完成状态枚举。

---

## 10. 遥测与状态

状态分层:生命周期、连接、协议、配置文件、设备类别、厂商专属 —— **与 v2.3 保持一致**,遥测载荷为语义核心 §2.4 的**子集**。

---

## 11. 故障模型

### 11.1 `fault_domain`(规范性)

故障记录**应**使用来自 **QNC-SEMANTIC-CORE-v1** §2.5 的 `fault_domain`。
**DDS 扩展包**的故障**仅**使用 `dds_pack` —— **不得**将 DDS QoS/IDL 故障复用 `southbound_device`(南向设备)相关代码。

### 11.2 严重程度与锁存

原则与 v2.3 保持一致,未作变更。

---

## 12. 发现、识别与绑定

### 12.1 目标

仅当满足以下条件时才建立**受支持的**操作:**绑定**(地址/端口/槽位)、**身份识别**(如协议支持)以及**已验证的配置文件**三者对齐。

### 12.2 协议能力声明

针对每种南向协议,**PSM** 发布如下矩阵:是否支持发现、识别方式、是否支持热插拔、**如适用**的最大枚举耗时。**不支持**发现机制**不应**被视为缺陷。

### 12.3 优先级顺序

1. 操作员配置的绑定关系
2. 协议识别结果(如支持)
3. 硬件提示信息
4. 配置文件兼容性解析

出现歧义时**应**产生 `DEGRADED`(降级)或 `SAFE_MODE`(安全模式)状态,依据策略而定;**不得**静默假定支持。

---

## 13. 配置接口

### 13.1 工件类别(相互隔离)

按 PRD §12 —— 系统 YAML、配置文件 YAML、扩展包 —— **不得**相互嵌套。

### 13.2 原子化应用

配置应用**应**具备事务性;失败时**应**恢复至上一次已提交的生成 ID。

---

## 14. 时序与性能

相关声明**应**引用 PRD §7 测试场景;ICD 不重复列出数值型非功能需求表格。

---

## 15. 安全模式

与语义核心及 PRD 保持一致:抑制执行动作;遥测/故障通道保持畅通;**若**策略启用,DDS 扩展包可进入"只订阅"模式。

---

## 16. 安全接口

TLS 1.2 及以上版本、写入/配置操作需鉴权、基于角色的访问控制(RBAC)—— **详见** OpenAPI 安全模式定义。**SEC-OPS-\*** 相关要求在系统层面依据 PRD 满足。

---

## 17. 诊断与日志

结构化日志**应**按照日志模式工件嵌入故障/遥测子集信息。

---

## 18. 版本管理

- `robot_api_major`(机器人 API 主版本) / `northbound_api_major`(北向 API 主版本)**相互独立**
- 每条载荷中包含 `icd_binding_version`(ICD 绑定版本)
- 配置文件 `schema_version` 依据 DPS 规定

---

## 19. 验证

核心版:除非明确声明使用 DDS 扩展包,否则测试范围**不**包含 DDS。需提供**协议矩阵**中识别/发现列的验证证据。

---

# 附录 A —— 消息头部(示意性 JSON)

与语义核心对齐;最终字段列表以 OpenAPI 为准。

```json
{
  "schema_version": "icd.robot.v1",
  "message_id": "uuid",
  "correlation_id": "uuid",
  "timestamp_utc": "2026-05-12T12:00:00Z",
  "qnc_instance_id": "qnc-001"
}
```

---

# 附录 B —— 错误/结果代码目录(结构)

| 代码 | fault\_domain(故障域) | 严重程度 | 摘要 |
| --- | --- | --- | --- |
| CFG-001 | configuration(配置) | error(错误) | 系统或配置包模式无效 |
| PRF-001 | profile(配置文件) | error(错误) | 配置文件 YAML 无效 |
| LNK-001 | southbound\_connection(南向连接) | warning(告警) | 链路不稳定 |
| PTL-001 | southbound\_protocol(南向协议) | error(错误) | 会话失败 |
| CMD-001 | command(命令) | error(错误) | 参数无效 |
| DDS-001 | dds\_pack(DDS 扩展包) | error(错误) | 参与者/发现故障 |
| DDS-002 | dds\_pack(DDS 扩展包) | warning(告警) | QoS 不匹配 |

*完整权威目录由项目自行控制。*

---

# 附录 C —— 生命周期状态

与 **QNC-SEMANTIC-CORE-v1** §2.6 的枚举完全一致。

---

# 附录 D —— 配置文件工件接口

权威结构定义见:**DPS\_v2.0** YAML 模式。ICD **不得**重复完整的配置文件模式。

---

# 附录 E —— 符合性检查清单(v1 核心版)

- \[ \] 已发布机器人侧与北向 OpenAPI 拆分文档
- \[ \] 载荷可追溯至语义核心
- \[ \] 核心构建版本中不存在 DDS 依赖要求
- \[ \] 已满足 PSM 规定的协议识别矩阵要求
- \[ \] 已验证安全模式下的执行动作抑制功能
- \[ \] 已验证原子化配置回滚功能

---

## 文档定位

ICD v3.0 是 v1 版本的**接口契约**层;语义**集中统一**由 QNC-SEMANTIC-CORE-v1 定义,以消除跨文档的语义漂移问题。


---
---
---
# QNC\-ICD

# QNC Interface Control Document

**Document Number:** QNC-ICD-001    
**Version:** 3.0 (v1 baseline)    
**Status:** Normative companion to **QNC-PRD-001 v3.0**    
**Classification:** Internal / Project Controlled  

---

## Document Change Log (v2.3 → v3.0)

| Area | Change |
| --- | --- |
| Semantics | All message contracts **normatively reference** **QNC-SEMANTIC-CORE-v1** |
| APIs | **Robot-facing** vs **northbound** APIs: **separate** versioning and OpenAPI artifacts |
| DDS | **QNC DDS Pack** only; never baseline Core ICD commitment |
| Discovery | §12 rewritten: **protocol-specific matrices**; removed PRS FR-001 style universal discovery |
| Roadmap | EtherCAT/PROFINET/Discovery Server/fleet/SCADA explicitly **non-v1** in ICD text |
| Appendices | Aligned field names to semantic core (`fault_domain`, `qnc_instance_id`) |

---

## 1. Purpose & Scope

This ICD defines **external interface contracts** for QNC: message shapes, state behaviors, fault publication, configuration interfaces, and **binding** of logical channels to REST, WebSocket, and logging.

**Normative semantic source:** **QNC-SEMANTIC-CORE-v1** — this ICD defines transport, paths, error codes, and **realization** only.

**Subordinate to:** QNC-PRD-001 v3.0.

---

## 2. Normative Language

SHALL / SHOULD / MAY / MUST NOT per RFC 2119 style usage in PRS.

---

## 3. Related Documents

- QNC-PRD-001 v3.0  
- QNC-SEMANTIC-CORE-v1  
- DPS\_v2.0  
- QNC-PSM\_v1.0  
- QNC-IDL\_v2.0 (DDS Pack)  
- QNC-ORG\_v1.1  

---

## 4. System Context & Ownership

### 4.1 Context

**Robot / cell software** ↔ **Robot-facing API** ↔ **QNC Core** ↔ **Southbound devices**    
**Northbound consumers** ↔ **Northbound API** ↔ **QNC** (same semantic core, **different** API surface and policy)

### 4.2 Ownership

| Owner | Responsibility |
| --- | --- |
| QNC | Southbound sessions, profile execution, normalization, publish/subscribe to **approved** topics, config integrity |
| Robot controller / MAiRA / LARA / MAV stack | Motion, safety, application sequencing |
| Device vendor | Device-internal behavior |
| DDS domain (when DDS Pack used) | Robot middleware ownership; QNC is **guest participant** |

---

## 5. v1 Interface Scope

### 5.1 Core Southbound (v1)

IO-Link, Modbus RTU, EtherNet/IP adapter, discrete DIO — **as PRD §5**.

### 5.2 Core Northbound (v1)

- **Robot-facing:** REST + WebSocket (`/robot/v{MAJOR}` prefix pattern — exact OpenAPI artifact)  
- **Northbound:** REST + WebSocket + structured logs (`/nb/v{MAJOR}` or host-separated deployment)

**SHALL:** distinct OpenAPI documents: `qnc-robot-api-vMAJOR.yaml`, `qnc-northbound-api-vMAJOR.yaml`.

### 5.3 DDS Pack

When **and only when** DDS Pack is installed and enabled: DDS peer interface per **QNC-IDL\_v2.0**. **Not** baseline Core conformance.

### 5.4 Non-v1 ICD Claims

ICD **MUST NOT** require: OPC UA, MQTT, Discovery Server, fleet APIs, SCADA adapters, EtherCAT/PROFINET stacks for Core conformance.

---

## 6. Logical Channels → Canonical Core

| Logical channel | Canonical binding |
| --- | --- |
| `qnc.command` | §2.2 Command |
| `qnc.response` | §2.3 Response |
| `qnc.telemetry` | §2.4 Telemetry |
| `qnc.fault` | §2.5 Fault |
| `qnc.lifecycle` | §2.6 Lifecycle |
| `qnc.service` | Management/diagnostics operations (subset of Command category `management` + read-only resources) |

---

## 7. Physical & Electrical

Unchanged intent from v2.3: hardware supplement defines connector/part details.

---

## 8. REST & WebSocket

### 8.1 JSON Schema Rule

Request/response bodies **SHALL** be JSON objects compatible with semantic core fields; `schema_version` **SHALL** identify ICD binding version (distinct from PRD product version).

### 8.2 Error Model (HTTP)

HTTP status + body **SHALL** include machine `result_code` aligned to **Appendix B** catalog; never contradict `completion` semantics.

---

## 9. Command Model

Command categories and completion set **identical** to semantic core §2.2–2.3. ICD **SHALL NOT** add alternate completion enums.

---

## 10. Telemetry & State

State layers: Lifecycle, Connection, Protocol, Profile, Device-class, Vendor — **as v2.3**, with telemetry payload **subset** of semantic core §2.4.

---

## 11. Fault Model

### 11.1 `fault_domain` (Normative)

Fault records **SHALL** use `fault_domain` from **QNC-SEMANTIC-CORE-v1** §2.5.    
**DDS Pack** faults use `dds_pack` only — **MUST NOT** reuse `southbound_device` codes for DDS QoS/IDL failures.

### 11.2 Severity & Latching

Unchanged from v2.3 principles.

---

## 12. Discovery, Identification, Binding

### 12.1 Objective

Establish **supported** operation only when: **binding** (address/port/slot), **identity** (where protocol provides), and **validated profile** align.

### 12.2 Protocol Capability Statement

For each southbound protocol, the **PSM** publishes a matrix: discovery support, identification means, hot-plug, max enumeration time **where applicable**. **Absence** of discovery **SHALL NOT** be treated as defect.

### 12.3 Precedence

1. Operator-configured binding  
2. Protocol identification result (if supported)  
3. Hardware hint  
4. Profile compatibility resolution  

Ambiguity **SHALL** yield `DEGRADED` or `SAFE_MODE` per policy; **MUST NOT** silently assume support.

---

## 13. Configuration Interfaces

### 13.1 Artifact Classes (Isolated)

Per PRD §12 — system YAML, profile YAML, extension bundles — **no cross-embedding**.

### 13.2 Atomic Apply

Configuration apply **SHALL** be transactional; failure **SHALL** restore prior committed generation id.

---

## 14. Timing & Performance

Claims **SHALL** reference PRD §7 test scenarios; ICD does not duplicate numeric NFR tables.

---

## 15. Safe Mode

Matches semantic core + PRD: actuation inhibit; telemetry/fault paths remain; DDS Pack subscribe-only **if** enabled by policy.

---

## 16. Security Interfaces

TLS 1.2+, authentication for write/config, RBAC — **detailed** in OpenAPI security schemes. **SEC-OPS-**\* satisfied at system level per PRD.

---

## 17. Diagnostics & Logging

Structured logs **SHALL** embed fault/telemetry subsets per logging schema artifact.

---

## 18. Versioning

- `robot_api_major` / `northbound_api_major` **independent**  
- `icd_binding_version` in each payload  
- Profile `schema_version` per DPS  

---

## 19. Verification

Core: no DDS in test scope unless DDS Pack declared. **Protocol matrix** evidence for identification/discovery columns.

---

# Appendix A — Message Headers (Illustrative JSON)

Aligned to semantic core; final field list in OpenAPI.

```json
{
  "schema_version": "icd.robot.v1",
  "message_id": "uuid",
  "correlation_id": "uuid",
  "timestamp_utc": "2026-05-12T12:00:00Z",
  "qnc_instance_id": "qnc-001"
}
```

---

# Appendix B — Error / Result Code Catalog (Structure)

| Code | fault\_domain | Severity | Summary |
| --- | --- | --- | --- |
| CFG-001 | configuration | error | Invalid system or bundle schema |
| PRF-001 | profile | error | Profile YAML invalid |
| LNK-001 | southbound\_connection | warning | Link unstable |
| PTL-001 | southbound\_protocol | error | Session failure |
| CMD-001 | command | error | Invalid parameters |
| DDS-001 | dds\_pack | error | Participant / discovery fault |
| DDS-002 | dds\_pack | warning | QoS mismatch |

*Authoritative full catalog project-controlled.*

---

# Appendix C — Lifecycle States

Identical enumeration to **QNC-SEMANTIC-CORE-v1** §2.6.

---

# Appendix D — Profile Artifact Interface

Authoritative structure: **DPS\_v2.0** YAML schema. ICD **MUST NOT** duplicate full profile schema.

---

# Appendix E — Conformance Checklist (v1 Core)

- \[ \] Robot-facing and northbound OpenAPI split published  
- \[ \] Payloads trace to semantic core  
- \[ \] No DDS requirement in Core build  
- \[ \] Protocol identification matrix satisfied per PSM  
- \[ \] Safe Mode actuation inhibit verified  
- \[ \] Atomic config rollback verified  

---

## Document Positioning

ICD v3.0 is the **interface contract** layer for v1; semantics are **centralized** in QNC-SEMANTIC-CORE-v1 to eliminate cross-document drift.
