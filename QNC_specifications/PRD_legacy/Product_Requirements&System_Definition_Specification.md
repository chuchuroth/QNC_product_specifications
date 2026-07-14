这份文档是关于 QNC（快速网络连接器） 的产品需求与系统定义规范（PRD/SDS），版本为 2.0。其核心目标是为工业机器人集成提供一个标准化的、可控的连接方案。

以下是该文档的要点总结：

1. 产品定义与核心定位

正式定义：QNC 是一款可配置的工业末端执行器连接网关。它通过协议适配器和经过验证的“设备描述文件”（Device Profiles），使机器人平台能够与各种工业工具进行通信，从而大幅减少为每个设备编写自定义集成代码的需求。
核心原则：QNC 被明确定义为连接与集成产品，而非通用机器人控制器、安全控制器或 AI 平台。
职责边界：QNC 负责底层的协议传输、描述文件加载与验证、标准化的状态报告和命令转发。它不负责高层任务逻辑、感知规划、经过认证的安全控制以及未经批准的协议转换。

2. 解决的问题与产品目标

痛点：解决工业部署中常见的驱动开发重复、工具上架不一致、集成周期长以及维护成本高等问题。
目标：在机器人软件与设备协议之间建立稳定且定义明确的边界，实现基于配置的设备上架，并标准化不同设备之间的健康、故障和指令行为。

3. 发布范围与技术规格

通讯协议：基准版本仅支持 Modbus RTU 和 IO-Link 协议。
目标设备：初期主要针对电力夹爪、简单线性执行器及选定的工业传感器。
明确排除项：EtherCAT、板载 AI 感知或追踪功能、自主安全决策等均不在基准版本范围内。
分层架构：系统分为三层：协议层（处理物理通讯）、描述文件层（映射设备行为）和集成层（向机器人平台暴露标准接口）。

4. 功能与非功能需求

描述文件管理：所有设备描述文件必须是版本化、经过数字签名且可追溯的，且必须通过架构校验才能加载。
指令与遥测：QNC 必须提供标准化的指令模型，并在传输前验证指令参数。同时，它将连接状态、协议健康度、活动描述文件等遥测数据进行隔离和规范化发布。
故障处理：QNC 会发布结构化的故障信息（包括代码、严重程度、来源及恢复建议），并定义了确定的降级模式和安全模式。
安全性 (Security)：固件包必须经过签名验证，配置文件在激活前需进行完整性检查。

5. 安全定位与验证

安全等级 (Safety)：QNC 在基准版本中属于非安全级控制器。系统级的急停（E-stop）权限仍保留在机器人系统的安全架构中。
安全模式 (Safe Mode)：在此模式下，QNC 将拒绝新的执行指令，且故障状态将被锁定，直到通过定义的程序恢复。
验证原则：所有产品声明（如性能指标）必须通过证据（如测试结果、基准数据）而非断言来验证。

6. 治理与路线图

配置治理：配置和描述文件被视为产品的一部分，必须遵循严格的审核、测试和兼容性强制执行流程。
路线图预测：未来可能扩展至更多工业协议、扩展设备类别及机队管理功能，但这些均不属于目前的基准发布范畴。

---

---
QNC (Quick Network Connector) is a configurable industrial end-effector connectivity gateway and software-first edge integration platform. It follows the principle of unified integration—not native universality: value comes from modular protocol layers, validated device profiles, and governed northbound interfaces, not from claiming one implementation supports every industrial protocol without release discipline.

Release scope is tiered. The current baseline southbound stack includes IO-Link (v1.1 master), Modbus RTU, EtherNet/IP (adapter), and configurable discrete digital I/O; baseline northbound interfaces are REST, WebSocket, and structured logging. Approved extension scope adds southbound protocols such as Modbus TCP and CANopen, northbound OPC UA Server and MQTT Client, multi-device aggregation, and FastDDS DomainParticipant integration in compatible ROS 2 / DDS deployments—enabling controlled subscribe/publish and, where validated and released, DDS-to-northbound bridging (e.g., to OPC UA or MQTT). Advanced gateway direction covers hardware-assisted industrial Ethernet such as EtherCAT and PROFINET, plant-level supervisory integration, FastDDS Discovery Server mode for larger or cross-network deployments, and fleet-style telemetry—each subject to formal approval, validation, and release sign-off.

QNC is not a safety-rated controller; Safe Mode is operational fault management. FastDDS participation is an approved extension direction until separately validated and released, not baseline functionality by default.
