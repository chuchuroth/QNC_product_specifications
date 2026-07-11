# QNC-ORG

# QNC 运维与恢复指南

**文档编号:** QNC-ORG-001
**版本:** 1.1(v1 基线)
**状态:** 作为 **QNC-PRD-001 v3.0** 的运维配套文档
**分类:** 内部 / 项目受控

---

## 文档变更日志(草稿 1.0 → v1.1)

| 领域 | 变更内容 |
| --- | --- |
| 层级 | 运维文本与**核心 / 工业扩展包 / DDS 扩展包**三层结构对齐 |
| 发现机制 | **取消**通用发现试运行步骤;改为引用按协议区分的矩阵 |
| DDS | **发现服务器**指引 → 移至**未来附录**;v1 版仅在使用 DDS 扩展包时支持 SIMPLE 发现方式 |
| API | 拆分**机器人侧**与**北向**试运行检查项 |
| 安全运维 | 新增 **SEC-OPS** 检查清单(证书、令牌、SBOM、CVE) |
| 容量 | 关联至 PRD §7 默认限制 |

---

## 0. 目的与范围

关于**部署**、**YAML 工件处理**、**监控**、**故障/恢复**、**重启**、**安全模式**以及(可选的)**DDS 扩展包**的操作规程。**不**定义 OpenAPI 路径(请使用生成的运行手册)。

---

## 0.1 运维原则

1. **禁止部分配置激活**
2. **YAML** 源工件具有权威性;须校验哈希值
3. 诊断时应遵循**故障域隔离**原则(南向 vs `dds_pack` vs 北向)
4. 关键故障类别的**安全模式响应时间 ≤ 1 秒**(依据 PRD)
5. 激活失败时**自动回滚**

---

## 1. 配置操作规程

### 1.1 工件分离(强制要求)

| 工件 | 格式 | 归属方 | 说明 |
| --- | --- | --- | --- |
| system-config(系统配置) | YAML | 平台 / 运维 | 网络、TLS、已启用软件包、API 层级 |
| device-profile(设备配置文件) | YAML | 设备集成 | 不含 DDS 字段 |
| dds-pack-bundle(DDS 扩展包) | IDL + yaml | 机器人技术 | 若未安装该软件包则**不**加载 |
| audit-policy(审计策略) | YAML | 安全 | 保留期限、签名 |

### 1.2 试运行步骤(核心)

1. **确认 SKU**:核心版还是含扩展包版本。
2. **硬件/网络检查**:供电、以太网/串口、IO-Link 主站端口、离散 IO 映射表。
3. **加载系统 YAML**:原子化应用。
4. **加载设备 YAML 配置文件**:逐设备加载;校验 `semantic_mapping`(语义映射)。
5. **校验**:模式、签名、兼容性(`robot`(机器人)及 `northbound`(北向)API 模式 ID)。
6. **激活**:单一生成 ID;失败时回滚。
7. **生命周期冒烟测试**:验证 BOOTING(启动中)→ ACTIVE(运行中)路径。
8. **功能测试**:命令/响应、遥测、故障注入触发安全模式、恢复流程。

### 1.3 DDS 扩展包附加步骤(可选)

1. **单独**加载 dds-pack-bundle,与配置文件分开加载。
2. 确认 **SIMPLE** 发现方式的可行性(组播策略)。
3. 依据 **QNC-IDL\_v2.0** 注册表版本核对主题/QoS 矩阵。
4. 验证 **dds\_pack** 故障不会影响南向会话(隔离测试)。

**注意:** v1 基线**不**规划发现服务器(Discovery Server)。

---

## 1.4 SEC-OPS 检查清单(现场)

- \[ \] 已安装 TLS 服务器证书链 + **到期日历**
- \[ \] 已记录 API 令牌签发/轮换责任人
- \[ \] 私钥轮换演练**至少每年**一次
- \[ \] 每次发布均已归档 SBOM(软件物料清单)
- \[ \] 已订阅 FastDDS 及基础操作系统的 CVE 监控清单

---

## 2. DDS 扩展包运维(简述)

来自 v1.0 草案的域 ID(Domain ID)策略**可**继续作为**建议**沿用 —— 本文档不再重复,详见平台运维手册。

**v1 版:** 若组播被阻断 → **停止**部署,或走架构例外流程(发现服务器**不属于** v1 默认路径)。

---

## 3. 部署模式(简化版)

| 模式 | 包含内容 |
| --- | --- |
| **核心独立模式** | 仅核心,单一单元 |
| **核心 + 工业扩展包** | 新增 Modbus TCP / CANopen / OPC UA / MQTT(按授权) |
| **核心 + DDS 扩展包** | 新增 DDS 参与者 |
| **全功能模式** | 所有已授权的软件包 |

从 v1 上线检查清单中**移除高级多网络 DDS** 功能。

---

## 4. 故障恢复

### 4.1 故障域

操作员**必须**将故障分类为:`southbound_*`(南向类)、`dds_pack`(DDS 扩展包类)、`configuration`(配置类)、`northbound`(北向类)、`robot_api`(机器人 API 类)、`security`(安全类)。

### 4.2 DDS 专项处理

DDS 发现丢失时:视为 `dds_pack` 故障;**不要**将现场总线硬件断电重启作为首要处置动作。

---

## 4.3 重启阶梯

服务重启 → 适配器重启 → DDS 参与者重启(若已启用)→ 受控系统重启 → 回滚至上一代配置。

---

## 5. 安全模式

行为保持一致:执行机构锁定;诊断功能保持开放;DDS 订阅可选进入"只订阅"模式。

---

## 6. 监控

最低告警项:安全模式、严重故障、回滚事件、签名失败、**dds\_pack** 发现/QoS 故障、API 鉴权失败。

---

## 7. 容量运维

监控设备数量、WebSocket 扇出数、日志磁盘占用、CPU 使用率 —— 与 PRD §7 对比;在触及限值前及时升级处理。

---

## 8. 限制

每次部署**应**生成对应的运行手册,其中包含**实际的 URL**、**命令行(CLI)**及**主题名称**。

---

## 9. 执行摘要

以**严格的工件分离**、**YAML 权威性**、**核心与 DDS 扩展包隔离**、**机器人侧与北向 API 意识**,以及**按协议区分的绑定方式**(而非假定通用发现机制)来运维 QNC v1。



---
---
---
# QNC\-ORG

# QNC Operations & Recovery Guide

**Document Number:** QNC-ORG-001    
**Version:** 1.1 (v1 baseline)    
**Status:** Operational companion to **QNC-PRD-001 v3.0**    
**Classification:** Internal / Project Controlled  

---

## Document Change Log (Draft 1.0 → v1.1)

| Area | Change |
| --- | --- |
| Tiers | Operations text aligned to **Core / Industrial Extensions / DDS Pack** |
| Discovery | **No** universal discovery commissioning steps; per-protocol matrix referenced |
| DDS | **Discovery Server** guidance → **future appendix**; v1 SIMPLE only when DDS Pack used |
| APIs | Split **robot-facing** vs **northbound** commissioning checks |
| Security ops | Added **SEC-OPS** checklist (certs, tokens, SBOM, CVE) |
| Capacity | Linked to PRD §7 default limits |

---

## 0. Purpose & Scope

Procedures for **deployment**, **YAML artifact handling**, **monitoring**, **fault/recovery**, **restart**, **Safe Mode**, and **DDS Pack** (optional). Does **not** define OpenAPI paths (use generated runbooks).

---

## 0.1 Operating Principles

1. **No partial configuration activation**  
2. **YAML** source artifacts are authoritative; verify hashes  
3. **Fault domain isolation** in diagnosis (southbound vs `dds_pack` vs northbound)  
4. **Safe Mode ≤ 1 s** for critical classes (per PRD)  
5. **Automatic rollback** on failed activation  

---

## 1. Configuration Procedures

### 1.1 Artifact Separation (Mandatory)

| Artifact | Format | Owner | Notes |
| --- | --- | --- | --- |
| system-config | YAML | Platform / Ops | Network, TLS, enabled packages, API tiers |
| device-profile | YAML | Device Integration | No DDS fields |
| dds-pack-bundle | IDL + yaml | Robotics | **Not** loaded if package absent |
| audit-policy | YAML | Security | Retention, signing |

### 1.2 Commissioning Steps (Core)

1. **Verify SKU**: Core vs extension packages installed.  
2. **Hardware/network**: Power, Ethernet/serial, IO-Link master ports, DIO maps.  
3. **Load system YAML**: atomic apply.  
4. **Load device YAML profiles**: per device; verify `semantic_mapping`.  
5. **Validate**: schema, signature, compatibility (`robot` + `northbound` API schema ids).  
6. **Activate**: single generation id; rollback on failure.  
7. **Lifecycle smoke**: BOOTING→ACTIVE path.  
8. **Functional test**: command/response, telemetry, fault inject to Safe Mode, recovery.

### 1.3 DDS Pack Add-On Steps (Optional)

1. Load **dds-pack-bundle** **separately** from profiles.  
2. Confirm **SIMPLE** discovery feasibility (multicast policy).  
3. Topic/QoS matrix check vs **QNC-IDL\_v2.0** registry revision.  
4. Verify **dds\_pack** faults do not alter southbound sessions (isolation test).

**Do not** plan Discovery Server for v1 baseline.

---

## 1.4 SEC-OPS Checklist (Field)

- \[ \] TLS server chain installed + **expiry calendar**  
- \[ \] API token issuance/rotation owners documented  
- \[ \] Private key rotation drill **≤ annual** minimum  
- \[ \] SBOM archived per release  
- \[ \] CVE watchlist subscription for FastDDS + base OS  

---

## 2. DDS Pack Operations (Abbreviated)

Domain ID strategy from v1.0 draft **MAY** be reused as **recommendation** — not duplicated here; maintain in platform playbook.

**v1:** If multicast blocked → **STOP** or architecture exception (Discovery Server **not** v1 default path).

---

## 3. Deployment Modes (Simplified)

| Mode | Contents |
| --- | --- |
| **Core Standalone** | Core only, single cell |
| **Core + Industrial** | Adds Modbus TCP / CANopen / OPC UA / MQTT as licensed |
| **Core + DDS Pack** | Adds DDS participant |
| **Full Stack** | All licensed packages |

Remove **Advanced Multi-Network DDS** from v1 go-live checklist.

---

## 4. Fault Recovery

### 4.1 Domains

Operators **MUST** classify: `southbound_*`, `dds_pack`, `configuration`, `northbound`, `robot_api`, `security`.

### 4.2 DDS-Specific

Loss of DDS discovery: treat as `dds_pack` fault; **do not** power-cycle fieldbus hardware as first action.

---

## 4.3 Restart Ladder

Service restart → adapter restart → DDS participant restart (if used) → controlled system restart → rollback generation.

---

## 5. Safe Mode

Same behavior: actuation lock; diagnostics open; DDS subscribe-only optional.

---

## 6. Monitoring

Minimum alerts: Safe Mode, critical faults, rollback events, signature failures, **dds\_pack** discovery/QoS faults, API auth failures.

---

## 7. Capacity Operations

Monitor device count, WS fan-out, log disk, CPU — compare to PRD §7; escalate before limits.

---

## 8. Limits

Runbook SHALL be generated per deployment with **actual URLs**, **CLI**, and **topic names**.

---

## 9. Executive Summary

Operate QNC v1 with **strict artifact separation**, **YAML authority**, **Core vs DDS Pack isolation**, **robot vs northbound API awareness**, and **protocol-specific binding** instead of assumed discovery.
