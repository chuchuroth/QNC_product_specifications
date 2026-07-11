# QNC-PSM

# QNC 协议与服务模块规范

**文档编号:** QNC-PSM-001
**版本:** 1.0(v1 基线)
**状态:** 作为 **QNC-PRD-001 v3.0** 的规范性配套文档
**分类:** 内部 / 项目受控

---

## 文档变更日志(草稿 0.1 → v1.0)

| 领域 | 变更内容 |
| --- | --- |
| 产品层级 | 重组为**核心**、**工业扩展包**、**QNC DDS 扩展包**三层,并具备**故障隔离**能力 |
| 语义 | 所有内部总线均与 **QNC-SEMANTIC-CORE-v1** 对齐 |
| 发现机制 | 按协议区分的**能力矩阵**;移除通用 `discover()` 契约 |
| DDS | 参与者模块**仅存在于** DDS 扩展包中;发现服务器为**路线图**规划,非 v1 内容 |
| 部署 | 移除"高级 DDS 多网络"作为规范性 v1 部署模式 |
| 北向 | OPC UA/MQTT **仅作为扩展**;核心版仅验证 REST/WS/日志 |

---

## 1. 目的与范围

定义实现 PRD v3.0 与 ICD v3.0 所需的**模块架构**、**运行时职责**、**协议适配规则**、**故障域**、**配置工件**及**验证钩子**。

**从属于:** PRD v3.0。**语义来源:** QNC-SEMANTIC-CORE-v1。

---

## 2. 三层运行时构成

| 层级 | 模块 |
| --- | --- |
| **QNC 核心** | 南向协议服务(基线列表)、配置文件解析器、语义映射器、命令代理、生命周期管理器、故障管理器、状态聚合器、北向 **REST/WS/日志**服务器、配置与工件管理器 |
| **工业扩展包** | Modbus TCP、CANopen、OPC UA、MQTT 等 —— **独立**可加载软件包 |
| **QNC DDS 扩展包** | FastDDS 参与者服务、IDL 适配器、QoS 解析器、DDS 故障监控器、可选的**已批准**桥接暂存 —— **与核心无强依赖关系** |

---

## 3. 分层架构(核心)

与既往草案保持相同的四层意图,**DDS 扩展包**呈现为**并列**关系,而非位于核心之下:

1. 物理与协议适配层(南向)
2. 配置文件与语义归一化层
3. 集成与控制层
4. 北向边缘服务层(核心版:REST、WebSocket、日志)

**DDS 扩展包**仅通过**隔离的适配器**进程间通信(IPC)/进程边界(为实现故障隔离,**宜**如此)接入第 1–3 层边界。

---

## 4. 模块表(核心简版)

| 模块 | 职责 |
| --- | --- |
| 传输适配器 | 链路建立、重试 |
| 协议服务 | 状态机、事务处理 |
| 配置文件解析器 | YAML 加载、验证、绑定 |
| 语义映射器 | 设备 → 规范遥测/命令/故障 |
| 命令代理 | 关联、完成状态、抑制处理 |
| 故障管理器 | fault\_domain 路由、安全模式 |
| 北向服务器 | 机器人侧与北向路由表 |
| 配置管理器 | 原子化应用、回滚生成 ID |

**DDS 扩展包:** 参与者控制器、发现管理器(**v1 仅支持 SIMPLE**)、类型注册表适配器、QoS 解析器、样本路由器、DDS 故障监控器。

---

## 5. 按协议区分的能力矩阵(规范性模式)

每个协议**应**发布如下内容:

| 列 | 说明 |
| --- | --- |
| `protocol_id` | 标识符 |
| `auto_discovery` | 是 / 否 / 部分支持 |
| `identification` | 无 / 基于寄存器 / IO-Link PD / CIP Identity / … |
| `hot_plug` | 热插拔策略 |
| `max_bind_time_s` | 识别阶段耗时上限目标 |
| `addressing_required` | 是/否 |

### 5.1 v1 示意性矩阵(权威数值以验证结果为准)

| 协议 | auto\_discovery(自动发现) | identification(识别方式) | addressing\_required(是否需要寻址) |
| --- | --- | --- | --- |
| IO\_LINK | 部分支持(取决于主站) | 厂商/设备参数描述 | 端口映射 |
| MODBUS\_RTU | 否 | 可选的 ID 寄存器 | 单元 ID、串口参数 |
| ETHERNET\_IP\_ADAPTER | 部分支持 | CIP Identity 对象 | IP + 连接路径 |
| DISCRETE\_DIGITAL\_IO | 否 | 电气映射表 | 通道映射 |

**核心不得**在所有协议上实现通用的 `discover()`。**可以**按协议实现 `identify_or_bind()`。

---

## 6. 南向服务内部契约(修订版)

以下内容替代通用的 `discover()`:

- `init(adapter_config, protocol_config, profile_binding)`(初始化)
- `identify_and_bind(context)`(识别并绑定)→ 返回结果:`bound`(已绑定) \| `unresolved`(未解析) \| `unsupported`(不支持)
- `connect()`(连接)
- `execute(...)`(执行)
- `read_telemetry(...)`(读取遥测)
- `read_faults(...)`(读取故障)
- `recover(policy)`(恢复,按策略)
- `shutdown()`(关闭)

---

## 7. QNC DDS 扩展包(可选)

### 7.1 v1 规范性 DDS 部署要求

- 已发布的 v1 版 DDS 扩展包**仅**支持 **SIMPLE** 发现方式。
- **发现服务器:** v1**禁止**作为强制功能;仅列入路线图规划。
- **主题预算:** 除非经工程变更批准,否则**不超过** 20 个订阅。
- **桥接:** v1 版不设通用的多跳 DDS 桥接图;仅支持每次发布中**明确列出**的桥接模板。

### 7.2 故障隔离

DDS 故障监控器**应**仅以 `fault_domain = dds_pack` 发出故障。**不得**仅因 DDS 发现丢失就将南向协议实例转为 FAULTED(故障)状态。

### 7.3 安全模式

若策略启用,允许进入"只订阅"模式;写操作(包括桥接输出)一律阻止。

---

## 8. QoS 目录

QoS 配置**仍**保留在 PSM 中,供工业扩展包/DDS 扩展包使用。**仅核心版构建**时**不要求**存在 QoS 目录。

草案 0.1 版中的相关表格**可**作为 **DDS 扩展包附录**保留 —— 副本维护在实现代码仓库中;PSM 仅通过 ID 引用。

---

## 9. 配置工件

| 工件 | 格式 | 归属方 |
| --- | --- | --- |
| 系统配置 | YAML | 平台 |
| 设备配置文件 | YAML | 设备集成 |
| 扩展包清单 | YAML | 产品 |
| DDS 扩展包 | IDL + qos.yaml + bridge.yaml | 机器人平台 |

示例骨架**应**仅在启用 DDS 的部署包中包含 `dds_pack:` 代码块。

---

## 10. 安全与完整性

强制执行签名、TLS、RBAC —— 与 PRD 中的 SEC 及 SEC-OPS 保持一致。

---

## 11. 验证

核心版矩阵:每个南向协议行均针对识别列进行测试。DDS 扩展包:独立测试套件。

---

## 12. 与 DPS 的边界

DPS 拥有 YAML 模式,PSM 拥有运行时解释与协议矩阵。

---

## 来源文档

PRD v3.0、ICD v3.0、DPS v2.0、QNC-SEMANTIC-CORE-v1、QNC-IDL\_v2.0。

---

## 文档定位

PSM v1.0 是模块的**实现架构**,**消除了**通用发现机制及**核心内嵌 DDS**的假设。



---
---
---
# QNC\-PSM

# QNC Protocol & Service Module Specification

**Document Number:** QNC-PSM-001    
**Version:** 1.0 (v1 baseline)    
**Status:** Normative companion to **QNC-PRD-001 v3.0**    
**Classification:** Internal / Project Controlled  

---

## Document Change Log (Draft 0.1 → v1.0)

| Area | Change |
| --- | --- |
| Product tiers | Reorganized into **Core**, **Industrial Extensions**, **QNC DDS Pack** with **fault isolation** |
| Semantics | All internal buses align to **QNC-SEMANTIC-CORE-v1** |
| Discovery | Per-protocol **capability matrices**; removed universal `discover()` contract |
| DDS | Participant module **only inside DDS Pack**; Discovery Server **roadmap**, not v1 |
| Deployment | Removed "Advanced DDS Multi-Network" as normative v1 mode |
| Northbound | OPC UA/MQTT **extension-only**; Core validates REST/WS/log only |

---

## 1. Purpose & Scope

Defines **module architecture**, **runtime responsibilities**, **protocol adaptation rules**, **fault domains**, **configuration artifacts**, and **validation hooks** realizing PRD v3.0 and ICD v3.0.

**Subordinate to:** PRD v3.0. **Semantic source:** QNC-SEMANTIC-CORE-v1.

---

## 2. Three-Tier Runtime Composition

| Tier | Modules |
| --- | --- |
| **QNC Core** | Southbound protocol services (baseline list), Profile Resolver, Semantic Mapper, Command Broker, Lifecycle Manager, Fault Manager, State Aggregators, Northbound **REST/WS/Log** server, Config & Artifact Manager |
| **Industrial Extensions** | Modbus TCP, CANopen, OPC UA, MQTT, etc. — **separate** loadable packages |
| **QNC DDS Pack** | FastDDS Participant Service, IDL adapter, QoS resolver, DDS fault monitor, optional **approved** bridge staging — **zero hard dependency from Core** |

---

## 3. Layered Architecture (Core)

Same four-layer intent as prior draft, with **DDS Pack** shown **adjacent**, not below Core:

1. Physical & Protocol Adaptation (southbound)  
2. Profile & Semantic Normalization  
3. Integration & Control  
4. Northbound Edge Services (Core: REST, WebSocket, logging)

**DDS Pack** attaches at layer 1–3 boundary **only** via **isolated adapter** IPC/process boundary **SHOULD** for fault containment.

---

## 4. Module Table (Core Abbreviated)

| Module | Responsibility |
| --- | --- |
| Transport Adapter | Link bring-up, retries |
| Protocol Service | State machines, transactions |
| Profile Resolver | YAML load, validate, bind |
| Semantic Mapper | Device → canonical telemetry/command/fault |
| Command Broker | Correlation, completion, inhibit |
| Fault Manager | fault\_domain routing, Safe Mode |
| Northbound Server | Robot vs northbound route tables |
| Config Manager | Atomic apply, rollback generation ids |

**DDS Pack:** Participant Controller, Discovery Manager (**SIMPLE only v1**), Type Registry Adapter, QoS Resolver, Sample Router, DDS Fault Monitor.

---

## 5. Protocol-Specific Capability Matrices (Normative Pattern)

Each protocol **SHALL** publish:

| Column | Description |
| --- | --- |
| `protocol_id` | Identifier |
| `auto_discovery` | yes / no / partial |
| `identification` | none / register-based / IO-Link PD / CIP Identity / … |
| `hot_plug` | policy |
| `max_bind_time_s` | upper target for identification phase |
| `addressing_required` | yes/no |

### 5.1 v1 Illustrative Matrix (Authoritative numbers in validation)

| Protocol | auto\_discovery | identification | addressing\_required |
| --- | --- | --- | --- |
| IO\_LINK | partial (master-dependent) | Vendor/device PD | port map |
| MODBUS\_RTU | no | optional ID registers | unit ID, serial params |
| ETHERNET\_IP\_ADAPTER | partial | CIP Identity objects | IP + connection path |
| DISCRETE\_DIGITAL\_IO | no | electrical map | channel map |

**Core SHALL NOT** implement a generic `discover()` on all protocols. **MAY** implement `identify_or_bind()` per protocol.

---

## 6. Southbound Service Internal Contract (Revised)

Replace universal `discover()` with:

- `init(adapter_config, protocol_config, profile_binding)`  
- `identify_and_bind(context)` → result: `bound` \\| `unresolved` \\| `unsupported`  
- `connect()`  
- `execute(...)`  
- `read_telemetry(...)`  
- `read_faults(...)`  
- `recover(policy)`  
- `shutdown()`  

---

## 7. QNC DDS Pack (Optional)

### 7.1 v1 Normative DDS Deployment

- **SIMPLE** discovery only for released v1 DDS Pack.  
- **Discovery Server:** **prohibited** as v1 mandatory feature; roadmap only.  
- **Topic budget:** ≤ 20 subscriptions unless engineering change approved.  
- **Bridge:** No normative multi-hop DDS bridge graph in v1; only **explicitly listed** bridge templates per release.

### 7.2 Fault Isolation

DDS Fault Monitor **SHALL** emit faults only with `fault_domain = dds_pack`. **SHALL NOT** transition southbound protocol instances to FAULTED solely due to DDS discovery loss.

### 7.3 Safe Mode

Subscribe-only allowed if policy enabled; writes blocked including bridge egress.

---

## 8. QoS Catalog

QoS profiles **remain** in PSM for Industrial/DDS extensions. **Core-only build** **SHALL NOT** require QoS catalog presence.

Tables from draft 0.1 **MAY** be retained as **DDS Pack appendix** — copy maintained in implementation repo; PSM references by ID.

---

## 9. Configuration Artifacts

| Artifact | Format | Owner |
| --- | --- | --- |
| System config | YAML | Platform |
| Device profile | YAML | Device integration |
| Extension manifest | YAML | Product |
| DDS Pack bundle | IDL + qos.yaml + bridge.yaml | Robotics platform |

Example skeleton **SHALL** separate `dds_pack:` block **only** in DDS-enabled deployment bundles.

---

## 10. Security & Integrity

Enforces signing, TLS, RBAC — aligns PRD SEC and SEC-OPS.

---

## 11. Validation

Core matrix: each southbound protocol row tested for identification column. DDS Pack: separate test suite.

---

## 12. Boundary with DPS

DPS owns YAML schema; PSM owns runtime interpretation and protocol matrices.

---

## Source Documents

PRD v3.0, ICD v3.0, DPS v2.0, QNC-SEMANTIC-CORE-v1, QNC-IDL\_v2.0.

---

## Document Positioning

PSM v1.0 is the **implementation architecture** for modules and **eliminates universal discovery** and **Core-embedded DDS** assumptions.
