# QNC-IDL

# QNC DDS 扩展包 —— Fast DDS IDL 类型注册表与治理标准

**文档编号:** QNC-IDL-001
**版本:** 2.0
**状态:** **仅在**安装 QNC DDS 扩展包时具有规范性
**分类:** 内部 / 项目受控

---

## 文档变更日志(v1.1 → v2.0)

| 领域 | 变更内容 |
| --- | --- |
| 产品定位 | 范围重命名为 **QNC DDS 扩展包**;明确**不属于核心(Core)** |
| 语义对齐 | IDL 类型**必须**映射到 **QNC-SEMANTIC-CORE-v1** 字段 |
| 发现机制 | **发现服务器**相关文档移至**附录 —— 未来规划**;v1 版**仅支持 SIMPLE 发现方式** |
| 厂商扩展 | `qnc.vendor.extension.*` 系列主题在 v1 中**属于非规范性**;取消关于通用厂商扩展的表述 |
| 治理 | 加强与**设备配置文件 YAML** 治理机制的分离 |
| 格式 | 移除杂散的"来源"标注内容;合并整理表格 |

---

## 1. 目的

定义 **QNC DDS 扩展包**如何使用 **Fast DDS** / **Fast DDS-Gen** 来规范、版本管理、验证、批准、发布及退役 **DDS** 主题契约。

**仅在启用 DDS 扩展包时适用。****不**治理设备配置文件。

---

## 2. 符合性

符合规范的 DDS 扩展包发布版本**必须**:使用注册表中的类型;通过第 9 节的验证;遵循演进规则;完成批准流程;随附兼容性声明与回滚方案;**不得**声称仅凭 DDS 扩展包即满足核心(Core)合规性。

---

## 3. 范围

涵盖:QNC 拥有的 DDS 主题、IDL 模块、主题↔类型映射、QoS 引用、兼容性决策、生命周期状态。

**不涵盖:** 未经适配器审批而嵌入第三方 IDL;替代 **YAML** 设备配置文件治理机制。

---

## 4. 规范性引用

DDS-XTypes 概念;**Fast DDS-Gen** 子集限制;本项目承认**运行时演进类型兼容性存在局限**——**兼容性由注册表 + CI 强制保证**,而非依赖中间件自身假设。

---

## 5. 注册表权威性

作为以下内容的唯一目录:已批准的类型、规范主题名称、归属方、字段演进、兼容性类别、QoS 配置引用、生命周期状态。

### 5.1 生命周期状态

`Draft`(草稿) → `UnderReview`(审阅中) → `Validated`(已验证) → `Released`(已发布) → `Deprecated`(已弃用) → `Retired`(已退役)

生产环境 DDS 端点中**仅**允许使用 **Released**(已发布)状态的类型。

### 5.2 原则

- DDS 扩展包为**可选**
- IDL/QoS 与设备配置文件**相互分离**
- 安全模式下:**订阅安全**主题为强制要求
- DDS 配置包支持原子化激活与回滚

---

## 6. 分层 IDL 模型

层级 **L0–L4** 沿用 v1.1 版本定义,并明确映射关系:

| 层级 | 对应语义核心 |
| --- | --- |
| L2 域层 | 遥测、故障、生命周期载荷 |
| L3 服务层 | 命令、响应、管理 |

**规则:** 若序列化至相同的语义对象,字段名**应尽量**与 JSON ICD 字段保持一致。

---

## 7. 规范主题(DDS 扩展包)

最低**逻辑**对齐要求(实际名称由项目自行控制,遵循 `qnc.*` 模式):

| 主题逻辑 ID | 用途 |
| --- | --- |
| qnc.command | 命令入口(若使用) |
| qnc.response | 响应出口 |
| qnc.telemetry | 遥测出口 |
| qnc.fault | 故障出口 |
| qnc.lifecycle | 生命周期 |
| qnc.service | 服务 |

**v1 版:** 仅当某桥接功能已针对该 SKU **正式发布**时,桥接主题(`qnc.bridge.*`)**方可**存在。

**已从 v1 规范性内容中移除:** 不受限制的 `qnc.vendor.extension.*` 主题树。

---

## 8. 版本管理与演进

按类型/包采用语义化版本管理。规则:仅追加字段、限定字符串/序列边界、枚举仅可追加且需配套消费者策略、不得静默重新定义语义。

---

## 9. 强制性 CI 控制

IDL 语法检查、Fast DDS-Gen 生成成功、编译通过、往返一致性测试、边界约束检查、主题唯一性、依赖分层、QoS 引用校验、兼容性矩阵差异比对、N/N-1 互操作测试、安全模式下"只订阅"回归测试、原子化激活演练。

---

## 10. 审批角色

作者、领域负责人、IDL 管理员、架构评审人、QA 负责人、发布经理 —— **职责分离**(每次变更最多兼任两个角色)。

---

## 11. 兼容性矩阵

按发布版本提供矩阵:生产者/订阅者版本、主题级兼容性、安全模式下的行为、桥接影响、迁移方案、回滚目标。

---

## 12. 安全模式

针对健康/故障/生命周期主题,订阅操作**必须**保持可用;与执行动作相关的写入器**必须**被阻止。

---

## 13. 变更模板

提案模板、字段变更记录模板、发布说明模板 —— 沿用 v1.1 版本(实现细节存于代码仓库模板中)。

---

## 14. 最终声明

任何 DDS 主题、类型或 QoS 配置**均不得**在未经注册表登记、验证、批准及兼容性分类的情况下,在 DDS 扩展包中发布。

---

## 附录 A —— 未来规划(非 v1 范围)

- DDS 发现服务器拓扑结构
- 广域 DDS 路由
- 厂商扩展市场生态

---

## 文档定位

QNC-IDL v2.0 **仅**治理 DDS 扩展包,与 **QNC-SEMANTIC-CORE-v1** 保持一致,并与 **DPS** 的 YAML 生命周期相互隔离。


---
---
---
# QNC\-IDL

# QNC DDS Pack — Fast DDS IDL Type Registry & Governance Standard

**Document Number:** QNC-IDL-001    
**Version:** 2.0    
**Status:** Normative **only when QNC DDS Pack is installed**    
**Classification:** Internal / Project Controlled  

---

## Document Change Log (v1.1 → v2.0)

| Area | Change |
| --- | --- |
| Product framing | Renamed scope to **QNC DDS Pack**; explicit **not Core** |
| Semantic alignment | IDL types **MUST** map to **QNC-SEMANTIC-CORE-v1** fields |
| Discovery | **Discovery Server** documentation moved to **Appendix — Future**; v1 **SIMPLE only** |
| Vendor extensions | `qnc.vendor.extension.*` topics **non-normative** for v1; remove universal vendor extension claims |
| Governance | Strengthened separation from **device profile YAML** governance |
| Formatting | Removed stray "Source" artifacts; consolidated tables |

---

## 1. Purpose

Defines how **QNC DDS Pack** specifies, versions, validates, approves, releases, and retires **DDS** topic contracts using **Fast DDS** / **Fast DDS-Gen**.

**Applies only when DDS Pack is enabled.** **Does not** govern device profiles.

---

## 2. Conformance

Conformant DDS Pack release **MUST**: use registry types; pass Section 9 validation; follow evolution rules; complete approval workflow; ship compatibility statement + rollback plan; **not** claim Core-only QNC compliance.

---

## 3. Scope

Covers: QNC-owned DDS topics, IDL modules, topic↔type mappings, QoS references, compatibility decisions, lifecycle states.

**Does not:** embed third-party IDL without adapter approval; replace **YAML** device profile governance.

---

## 4. Normative References

DDS-XTypes concepts; **Fast DDS-Gen** subset restrictions; project acknowledges **runtime evolved-type compatibility limitations** — **compatibility is registry + CI enforced**, not assumed from middleware.

---

## 5. Registry Authority

Sole catalog for: approved types, canonical topic names, ownership, field evolution, compatibility class, QoS profile reference, lifecycle state.

### 5.1 Lifecycle States

`Draft` → `UnderReview` → `Validated` → `Released` → `Deprecated` → `Retired`

Only **Released** types in production DDS endpoints.

### 5.2 Principles

- DDS Pack **optional**  
- IDL/QoS **separate** from device profiles  
- Safe Mode: **subscribe-safe** topics mandatory  
- Atomic activation with rollback for DDS configuration bundles  

---

## 6. Layered IDL Model

Layers **L0–L4** as v1.1 with explicit mapping:

| Layer | Maps to semantic core |
| --- | --- |
| L2 Domain | Telemetry, Fault, Lifecycle payloads |
| L3 Service | Command, Response, management |

**Rule:** Field names **SHOULD** match JSON ICD fields where serializing to same semantic objects.

---

## 7. Canonical Topics (DDS Pack)

Minimum **logical** alignment (actual names project-controlled, pattern `qnc.*`):

| Topic logical id | Purpose |
| --- | --- |
| qnc.command | Command ingress (if used) |
| qnc.response | Response egress |
| qnc.telemetry | Telemetry egress |
| qnc.fault | Fault egress |
| qnc.lifecycle | Lifecycle |
| qnc.service | Service |

**v1:** Bridge topics (`qnc.bridge.*`) **SHALL** exist only if that bridge is **released** for the SKU.

**Removed from v1 normative:** unrestricted `qnc.vendor.extension.*` trees.

---

## 8. Versioning & Evolution

Semantic versioning per type/bundle. Rules: append fields, bounded strings/sequences, enum append-only with consumer policy, no silent semantic redefinition.

---

## 9. Mandatory CI Controls

IDL lint, Fast DDS-Gen success, compile, round-trip, boundedness, topic uniqueness, dependency layers, QoS reference validation, compatibility matrix diff, N/N-1 interop, Safe Mode subscribe-only regression, atomic activation drill.

---

## 10. Approval Roles

Author, Domain Owner, IDL Steward, Architecture Reviewer, QA Owner, Release Manager — **separation of duties** (max two hats per change).

---

## 11. Compatibility Matrix

Per-release matrix: producer/subscriber versions, topic-level compatibility, Safe Mode behavior, bridge impact, migration, rollback target.

---

## 12. Safe Mode

Subscribe operations **MUST** remain available for health/fault/lifecycle topics; actuation-related writers **MUST** be blocked.

---

## 13. Change Templates

Proposal, field change record, release note templates — **retained** from v1.1 (implementation detail in repo templates).

---

## 14. Final Statement

No DDS topic, type, or QoS profile **SHALL** ship in DDS Pack without registry registration, validation, approval, and compatibility classification.

---

## Appendix A — Future (Non-v1)

- DDS Discovery Server topologies  
- Wide-area DDS routing  
- Vendor extension marketplace  

---

## Document Positioning

QNC-IDL v2.0 governs **DDS Pack only**, aligned with **QNC-SEMANTIC-CORE-v1** and isolated from **DPS** YAML lifecycle.
