# QNC-DPS

# QNC 设备配置文件规范

**文档编号:** QNC-DPS-001
**版本:** 2.0(v1 基线)
**状态:** 作为 **QNC-PRD-001 v3.0** 的规范性配套文档
**分类:** 内部 / 项目受控

---

## 文档变更日志(v1.1 → v2.0)

| 领域 | 变更内容 |
| --- | --- |
| YAML | **唯一**具规范性的编写格式;JSON 仅作为**带校验和的编译/运行时导出格式**被允许 |
| 语义 | 命令/遥测/故障目标**明确映射至** **QNC-SEMANTIC-CORE-v1** |
| DDS | 移除任何暗示配置文件引用 DDS 类型的表述;**严格划清边界** |
| 故障映射 | `source_layer`(故障来源层)取值与 `fault_domain`(故障域)分类体系对齐;设备配置文件中的故障来源**不得**出现 `dds` —— DDS 相关故障**仅**存在于 DDS 扩展包中 |
| 兼容性 | 分别引用**机器人 API** 与**北向 API** 的模式版本 |
| 符合性 | v1 版配置文件**不得**要求仅存在于路线图中的协议 |

---

## 1. 目的与范围

定义南向设备的 **YAML** 设备配置文件**结构**、**验证**、**治理**、**生命周期**与**符合性**要求。

**范围之外:** DDS IDL、QoS、桥接映射、OpenAPI —— **均为独立工件**。

---

## 2. 规范性用语

SHALL(应)/ SHOULD(宜)/ MAY(可)/ MUST NOT(不得)。

---

## 3. 相关文档

PRD v3.0、ICD v3.0、PSM v1.0、QNC-SEMANTIC-CORE-v1、QNC-IDL\_v2.0(仅作信息性边界参考)。

---

## 4. 配置文件的角色定位

配置文件是将厂商协议语义**声明式桥接**至规范的**命令(Command)**、**遥测(Telemetry)**与**故障(Fault)**语义(语义核心)的载体。**不得**重新定义生命周期或安全模式。

---

## 5. 架构边界

配置文件**不得**包含:可执行脚本、全网发现断言、DDS 定义、北向端点定义。

---

## 6. YAML 作为权威格式

- 编写、评审、CI 与发布环节**应**统一使用 YAML。
- 运行时**可以**编译为内部 JSON 或二进制格式;构建过程**应**记录**规范 YAML** 的 `artifact_hash`(工件哈希值)。
- **不得**在未与 YAML 对账核实的情况下,将手动编辑的 JSON 视为事实来源。

---

## 7. 顶层结构(必填)

```yaml
profile_metadata:        # 配置文件元数据
device_identity:         # 设备身份
protocol_binding:        # 协议绑定
semantic_mapping:        # 新增 v2:显式绑定至规范核心
capabilities:            # 能力
commands:                # 命令
parameters:              # 参数
telemetry:               # 遥测
faults:                  # 故障
lifecycle_sequences:     # 生命周期序列
compatibility:           # 兼容性
validation:              # 验证
security:                # 安全
```

### 7.1 `semantic_mapping`(v2 必填)

**目的:** 声明配置文件字段如何绑定至规范语义。

```yaml
semantic_mapping:
  canonical_command_namespace: "qnc.v1.commands.gripper"      # 规范命令命名空间
  canonical_telemetry_namespace: "qnc.v1.telemetry.gripper"   # 规范遥测命名空间
  canonical_fault_namespace: "qnc.v1.faults.vendor"           # 规范故障命名空间
```

**规则:** 每一条 `commands[]` 条目**应**声明与语义注册表行(项目受控的枚举表)相匹配的 `canonical_command_id`。

---

## 8. 协议绑定

每个配置文件**恰好**绑定**一种**南向 `protocol`(协议);标识符**应**与目标构建版本中**已发布**的核心版或工业扩展包版本相匹配。

**不得**在 v1 配置文件库中绑定 EtherCAT/PROFINET,除非另行发布(v1 预期为不支持)。

### 8.1 `discovery_hints` → `binding_hints`(建议重命名)

可选提示**不得**以暗示保证发现成功的方式命名;新配置文件中建议采用 `binding_hints`(绑定提示)。

---

## 9. 能力与命令模型

命令**应**包含:

- `canonical_command_id`(字符串,须已注册)
- `command_category`:`profile`(配置文件类) \| `transport`(传输类,仅在策略允许时使用)
- 与语义核心 §2.6 对齐的 `allowed_lifecycle_states`(允许的生命周期状态)列表

---

## 10. 遥测

每个遥测字段**宜**声明 `canonical_telemetry_key`,用于向 REST/WS 归一化导出。

---

## 11. 故障映射

每个故障**应**映射至:

- `qnc_fault_code`(故障代码)
- `severity`(严重程度)
- `source_layer`(来源层):仅限 `protocol`(协议) \| `device`(设备) \| `profile`(配置文件)(南向范围)
- `safe_mode_required`(是否需要安全模式)布尔值

**不得**在设备配置文件中使用 `source_layer: dds`。

---

## 12. 生命周期序列

仅限有边界的声明式步骤 —— 规则与 v1.1 一致。

---

## 13. 兼容性

`compatibility`(兼容性)**应**包含:

```yaml
compatibility:
  min_qnc_core_version: "3.0.0"                    # 最低 QNC 核心版本
  max_validated_qnc_core_version: "3.x"            # 已验证的最高 QNC 核心版本
  required_robot_api_schema: "robot.v1"            # 要求的机器人 API 模式版本
  required_northbound_api_schema: "nb.v1"          # 要求的北向 API 模式版本
  protocol_adapter_constraints: []                 # 协议适配器约束
```

---

## 14. 验证与审批

治理意图保持不变:未经 PRD 规定的测试与签名,不得进入 `Released`(已发布)状态。

---

## 15. 安全

生产环境配置文件**必须**强制进行 YAML 签名。

---

## 16. DDS 与北向

**单段规范性说明:** 设备配置文件**仅**适用于南向。与 DDS 扩展包的任何关联均为**间接**关系(归一化数据可能供内部总线使用,再由桥接代码消费),但**不得**在配置文件文件内编写此类关联。

---

## 17. 符合性

配置文件在以下情况下视为符合规范:通过模式校验;`semantic_mapping` 完整;协议已发布;不包含禁止使用的协议;签名有效。

---

## 附录 A —— 规范性 YAML 骨架

```yaml
profile_metadata:
  profile_id: "qnc.gripper.vendor.model"      # 配置文件 ID
  profile_version: "2.0.0"                     # 配置文件版本
  profile_status: "Draft"                      # 配置文件状态(草稿)
  owner_team: "Integration"                    # 责任团队(集成组)
  created_utc: "2026-05-12T00:00:00Z"          # 创建时间
  updated_utc: "2026-05-12T00:00:00Z"          # 更新时间
  change_summary: "v2 semantic mapping"        # 变更摘要(v2 语义映射)

semantic_mapping:
  canonical_command_namespace: "qnc.v1.commands.gripper"
  canonical_telemetry_namespace: "qnc.v1.telemetry.gripper"
  canonical_fault_namespace: "qnc.v1.faults.vendor"

device_identity:
  vendor: "Vendor"                       # 厂商
  model: "Model"                         # 型号
  device_class: "GRIPPER_ELECTRIC"       # 设备类别(电动夹爪)

protocol_binding:
  protocol: "MODBUS_RTU"                 # 协议
  transport_profile: "serial_default"    # 传输配置(默认串口)
  addressing_mode: "unit_id"             # 寻址模式(单元 ID)
  session_requirements: {}               # 会话要求
  communication_defaults: {}             # 通信默认值

commands: []
parameters: []
telemetry: []
faults: []
lifecycle_sequences: {}
compatibility: {}
validation: {}
security:
  signature_required: true               # 是否要求签名
```

---

## 文档定位

DPS v2.0 通过**YAML 权威性**、**语义核心可追溯性**以及**严格的 DDS 隔离**,使设备治理机制保持一致。



---
---
---
# QNC\-DPS

# QNC Device Profile Specification

**Document Number:** QNC-DPS-001    
**Version:** 2.0 (v1 baseline)    
**Status:** Normative companion to **QNC-PRD-001 v3.0**    
**Classification:** Internal / Project Controlled  

---

## Document Change Log (v1.1 → v2.0)

| Area | Change |
| --- | --- |
| YAML | **Only** normative authoring format; JSON allowed **only** as compiled/runtime export with checksum |
| Semantics | Commands/telemetry/fault targets **explicitly map to** **QNC-SEMANTIC-CORE-v1** |
| DDS | Removed any implication that profiles reference DDS types; **strict boundary** |
| Fault mapping | `source_layer` values aligned with `fault_domain` taxonomy; **no **`dds` inside device profile fault source — DDS faults **only** in DDS Pack |
| Compatibility | References **robot API** and **northbound API** schema versions separately |
| Conformance | v1 profiles **MUST NOT** require roadmap-only protocols |

---

## 1. Purpose & Scope

Defines **YAML** device profile **structure**, **validation**, **governance**, **lifecycle**, and **conformance** for southbound devices.

**Out of scope:** DDS IDL, QoS, bridge maps, OpenAPI — **separate artifacts**.

---

## 2. Normative Language

SHALL / SHOULD / MAY / MUST NOT.

---

## 3. Related Documents

PRD v3.0, ICD v3.0, PSM v1.0, QNC-SEMANTIC-CORE-v1, QNC-IDL\_v2.0 (informational boundary only).

---

## 4. Profile Role

Profiles are the **declarative bridge** from vendor protocol semantics **into** canonical **Command**, **Telemetry**, and **Fault** semantics (semantic core). They **MUST NOT** redefine lifecycle or Safe Mode.

---

## 5. Architectural Boundary

Profiles **SHALL NOT** contain: executable scripts, network-wide discovery assertions, DDS definitions, northbound endpoint definitions.

---

## 6. YAML as Authoritative Format

- Authoring, review, CI, and release **SHALL** use YAML.  
- Runtime **MAY** compile to internal JSON or binary; build **SHALL** record `artifact_hash` of **canonical YAML**.  
- **MUST NOT** treat JSON hand-edits as source of truth without YAML reconciliation.

---

## 7. Top-Level Sections (Required)

```yaml
profile_metadata:
device_identity:
protocol_binding:
semantic_mapping:      # NEW v2: explicit binding to canonical core
capabilities:
commands:
parameters:
telemetry:
faults:
lifecycle_sequences:
compatibility:
validation:
security:
```

### 7.1 `semantic_mapping` (v2 Required)

**Purpose:** Declare how profile fields attach to canonical semantics.

```yaml
semantic_mapping:
  canonical_command_namespace: "qnc.v1.commands.gripper"
  canonical_telemetry_namespace: "qnc.v1.telemetry.gripper"
  canonical_fault_namespace: "qnc.v1.faults.vendor"
```

**Rule:** Every `commands[]` entry **SHALL** declare `canonical_command_id` matching semantic registry row (project-controlled enum table).

---

## 8. Protocol Binding

Exactly **one** southbound `protocol` per profile; identifiers **SHALL** match **released** Core or Industrial Extension for target build.

**MUST NOT** bind EtherCAT/PROFINET in v1 profile corpus unless separately released (expected: false for v1).

### 8.1 `discovery_hints` → `binding_hints` (Rename Recommended)

Optional hints **SHALL NOT** be named in a way that implies guaranteed discovery; prefer `binding_hints` in new profiles.

---

## 9. Capability & Command Models

Commands **SHALL** include:

- `canonical_command_id` (string, registered)  
- `command_category`: `profile` | `transport` (transport only if policy allows)  
- `allowed_lifecycle_states` list aligned to semantic core §2.6  

---

## 10. Telemetry

Each telemetry field **SHOULD** declare `canonical_telemetry_key` for normalized export to REST/WS.

---

## 11. Fault Mapping

Each fault **SHALL** map to:

- `qnc_fault_code`  
- `severity`  
- `source_layer`: `protocol` | `device` | `profile` only (southbound scope)  
- `safe_mode_required` boolean  

**MUST NOT** use `source_layer: dds` in device profiles.

---

## 12. Lifecycle Sequences

Bounded declarative steps only — same rules as v1.1.

---

## 13. Compatibility

`compatibility` **SHALL** include:

```yaml
compatibility:
  min_qnc_core_version: "3.0.0"
  max_validated_qnc_core_version: "3.x"
  required_robot_api_schema: "robot.v1"
  required_northbound_api_schema: "nb.v1"
  protocol_adapter_constraints: []
```

---

## 14. Validation & Approval

Unchanged governance intent: no `Released` without tests + signatures per PRD.

---

## 15. Security

YAML signing **SHALL** be mandatory for production profiles.

---

## 16. DDS & Northbound

**Single paragraph normative:** Device profiles apply to **southbound only**. Any relationship to DDS Pack is **indirect** (normalization may feed internal bus consumed by bridge code) but **SHALL NOT** be authored inside the profile file.

---

## 17. Conformance

Profile is conformant if: YAML validates against schema; semantic\_mapping complete; protocol released; no forbidden protocols; signature valid.

---

## Appendix A — Normative YAML Skeleton

```yaml
profile_metadata:
  profile_id: "qnc.gripper.vendor.model"
  profile_version: "2.0.0"
  profile_status: "Draft"
  owner_team: "Integration"
  created_utc: "2026-05-12T00:00:00Z"
  updated_utc: "2026-05-12T00:00:00Z"
  change_summary: "v2 semantic mapping"

semantic_mapping:
  canonical_command_namespace: "qnc.v1.commands.gripper"
  canonical_telemetry_namespace: "qnc.v1.telemetry.gripper"
  canonical_fault_namespace: "qnc.v1.faults.vendor"

device_identity:
  vendor: "Vendor"
  model: "Model"
  device_class: "GRIPPER_ELECTRIC"

protocol_binding:
  protocol: "MODBUS_RTU"
  transport_profile: "serial_default"
  addressing_mode: "unit_id"
  session_requirements: {}
  communication_defaults: {}

commands: []
parameters: []
telemetry: []
faults: []
lifecycle_sequences: {}
compatibility: {}
validation: {}
security:
  signature_required: true
```

---

## Document Positioning

DPS v2.0 aligns device governance with **YAML authority**, **semantic core traceability**, and **strict DDS separation**.
