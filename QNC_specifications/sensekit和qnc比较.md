# 项目概览

**核心目标与背景**
该文档本质上是一份针对内部项目 **SenseKit CM5 / Gen 1** 的**技术搭建、部署与运维文档**。重点在于如何可重复地为 SenseKit 使用场景配置 Compute Module 5（CM5），包括操作系统刷写、网络配置、音频/语音处理器配置、Docker 部署、时间同步以及服务管理等内容。
从文档内容可以看出，该项目旨在实现一个稳定运行的 SenseKit 系统，涉及 **麦克风/XVF3800、背光（Backlight）、RealSense、容器化部署以及设备间同步** 等功能。

---

# 项目参与者、角色与协作

文档中明确列出了相关负责人：

* **Paul Kocher**：项目管理
* **Yitao Ding**、**Mikhail Belikin**：CM5 IO Board（硬件接口板）
* **Moritz Mähr**：CM5 软件

由此可以看出，项目采用了明确的分工结构，涵盖：

* 项目管理
* 硬件/IO 板开发
* CM5 软件与部署

协作方式主要围绕以下技术接口展开：

* 硬件板卡
* XVF3800 固件
* Host/OS
* 部署工具链
* 运行时服务

---

# 代码与交付物结构

根据文档，所有相关仓库均位于 GitLab 的 **Adv_Dev/SenseKit** 下；Docker 镜像与软件包位于 **Software_Images/Adv-Dev**。

主要组件包括：

* 麦克风：`xmos-xvf-3800`
* 背光：`button-monitor`
* RealSense：`ros-realsense`
* Container Registry
* Package Registry
* `sensekit-remote-installer`（自动化安装工具）
* `neura_sensekit_ros1_docker`（基于 Docker 的部署方案）

这表明项目已经建立起一条较完整的技术交付链，从固件、主机配置一直延伸到容器部署。

---

# 关键进展

## 1）技术目标状态已被明确

一个重要进展是，多个核心技术标准已经确定。
尤其是 **XVF3800 固件配置** 已非常具体：

* UA 模式（通过 USB 提供 Host Audio）
* 16 kHz 采样率
* 16 Bit
* 矩形麦克风阵列
* ASR 位于右声道
* DAC 输出兼容 IO 板 DAC

这些规范非常关键，因为它们统一了硬件与音频接口标准，从而减少集成错误。

---

## 2）Gen-1 基础环境已固定

针对 **SenseKit Gen 1**，目标运行环境已经明确：

* **Raspberry Pi OS（64-bit）Desktop**
* 发布版本：**2025-05-13**
* Host 依赖：

  * `pulseaudio 16.1.0`
  * `chrony`
  * `supervisor`
  * `docker`

文档中还引用了对应 Debian 软件包。

这是项目成熟度的重要体现，因为现在已经存在一个**可复现的标准参考环境**。
其中 OS 发布日期也是文档中最明确的时间锚点。

---

## 3）Setup 与 Deployment 流程已结构化

文档描述了新 CM5 的完整部署流程：

1. 使用 `usbboot` 和 Raspberry Pi Imager 刷写 OS
2. 配置默认账号与 SSH
3. 为 `eth0` 设置静态 IP（`192.168.2.61`）
4. 启用 GPIO14 的 PWM Overlay
5. 配置 Docker、Chrony、Supervisor 与 PulseAudio
6. 从 GitLab 拉取 Docker 镜像
7. 创建基于 Supervisor 的 systemd 服务
8. 确保整个系统的时间同步

这不仅仅是一个状态说明，更说明团队已经建立起一个**可实际投入使用的设备初始化流程**。

---

## 4）已引入自动化，但尚未完全自动化

文档中特别强调了 **`sensekit-remote-installer`** 的引入。
系统建议在刷写 OS 后优先使用该工具，而手动文档仅作为参考。

这意味着项目正在从：

> “完全手工配置”

逐步过渡到：

> “半自动化部署”

但 OS 刷写阶段显然仍未完全自动化。

---

## 5）关键技术决策及原因

文档已经明确了多项关键技术选择：

### 时间同步

使用 **Chrony** 进行时间同步，并禁用：

* `systemd-timesyncd`
* `ntp`

原因是避免多个时间服务之间发生冲突。

---

### 音频系统

优先使用 **原生 PulseAudio**，并关闭 PipeWire 兼容层。

原因：

* 更可控
* 音频后端行为更稳定可预测

---

### 服务管理

使用：

* `Supervisor`
* `systemd`

管理运行服务。

原因：

* 自动启动稳定
* 不同功能模块之间隔离更清晰

---

### 部署方式

使用 GitLab Registry 的 **Docker 化部署**。

原因：

* 保证运行环境一致性

---

### 网络

采用 **静态 IP**。

原因：

* 保证目标网络中的通信稳定性

---

整体来看，这些决策都明显围绕以下目标展开：

* 稳定性
* 可复现性
* 集中化维护

---

## 6）文档中的时间/阶段信息

文档仅包含有限的传统项目时间规划信息。

明确可见的包括：

* Gen-1 参考环境基于 **2025-05-13** 的 OS Release
* 技术流程顺序：

  * Flashing
  * 基础配置
  * 安装依赖
  * Deployment
  * 服务初始化
  * 时间同步

但文档中缺少：

* 完整里程碑计划
* 工作量估算
* 正式 Roadmap

---

# 问题 / 风险

## 1）安全风险

最明显的问题是：

文档中直接写入了：

* Hostname
* Username
* Password

虽然对内部部署来说方便，但从安全与合规角度存在明显风险。

另外，PulseAudio 配置中的：

`auth-anonymous=1`

也意味着网络内允许匿名访问，这同样存在安全隐患。

---

## 2）对内部基础设施依赖较高

部署流程严重依赖内部资源：

* 内部 GitLab
* Container Registry
* Access Token
* 内部网络 / Neuragroup 网络
* 指定时间服务器（`192.168.2.14`）

任何一个组件不可用，都可能阻塞部署流程。

这对：

* 新人 onboarding
* 大规模部署
* 故障排查

都是明显风险。

---

## 3）仍存在容易出错的手工步骤

尽管已有 `sensekit-remote-installer`，但仍有多个步骤需要人工操作，例如：

* OS 刷写
* 文件与服务配置
* SSH 会话期间切换网络
* 音频后端切换
* systemd/Supervisor 初始化

这些都是典型的人为错误来源，特别是在批量部署设备时。

---

## 4）缺乏管理层面的项目状态信息

从“项目分析”角度来看：

该文档更偏向：

> 技术运维文档

而不是：

> 项目管理文档

缺少：

* 带负责人（Owner）的任务列表
* 正式里程碑
* 各工作包状态
* 验收标准
* 变更记录 / 决策日志

因此不利于：

* 管理层汇报
* 优先级管理

---

## 5）集成与维护风险

技术层面还存在额外复杂性：

* XVF3800 固件特殊要求
* 音频栈定制
* Docker + Supervisor + systemd
* 多设备网络与时间同步

涉及层级越多，集成复杂度越高，因此：

* 回归测试
* 标准镜像

会变得非常重要。

---

# 下一步计划

## 根据文档推导出的建议行动

### 1）进一步标准化 Setup

将 `sensekit-remote-installer` 作为标准安装路径，手动文档仅保留为备用/参考。

---

### 2）优先解决安全问题

重点检查：

* 默认账号密码
* 开放式网络认证
* Token 管理

建议：

* 轮换凭据
* 引入更安全的 Provisioning 机制

---

### 3）建立 Golden Image / 标准参考镜像

由于 Gen 1 已明确绑定：

* 固定 OS 版本
* 固定软件包版本

因此应进一步形成：

* 可版本化 Reference Image
  或
* 可重复生成的 Installer Release

---

### 4）将项目状态与技术文档分离

建议额外维护独立项目文档，包括：

* 里程碑
* 未解决问题
* Owner
* 风险
* 决策记录
* 依赖关系

---

### 5）补充集成验收清单

新设备应有统一验收流程，例如：

* Firmware OK
* Audio OK
* Time Sync OK
* Docker OK
* Services OK
* Registry Access OK
* End-to-End Test OK

---

### 6）进一步明确时间与网络架构

应明确说明：

* `192.168.2.14`
* CM5 静态 IP

为何是架构级决策，并补充：

* 故障场景
* 替代方案

---

# 总结 / 建议

## 简要结论

该文档表明：

SenseKit-CM5 项目在技术结构上已经较为成熟：

* 职责清晰
* 平台明确
* 固件与音频规范清楚
* 已具备手动与半自动化部署能力

其最大价值在于：

> 技术环境的可复现性

---

## 管理视角

但作为“项目文档”来看，它仍不完整：

缺少：

* 里程碑
* 实际进度
* 未决事项
* 优先级信息

因此更适合被理解为：

> 技术/运维文档

而不是完整的项目状态报告。

---

# 我的三点建议

### 短期

优先解决：

* 凭据安全
* 网络认证
* Token 管理流程

---

### 中期

进一步标准化：

* Installer
* Reference Image

以简化：

* Rollout
* 运维支持

---

### 组织层面

建立单独、易读的项目跟踪文档，避免：

* 风险
* 进度
* 决策

仅隐含在技术文档中。






---
---

# 执行摘要（Executive Summary）

这两份文档描述的**并不是完全相同的产品**，但它们彼此非常接近，并且在战略层面上**具有很强的整合潜力**。

**SenseKit** 更像是一个已经具体化的 **基于 CM5 的嵌入式 / 边缘设备平台（Embedded / Edge Platform）**，重点关注：

* 硬件搭建
* 操作系统
* 部署流程
* 音频 / 摄像头外设
* Docker 运行环境
* 时间同步
* 设备管理

而 **QNC** 则更偏向一个**工业级 Connectivity Gateway 产品方案**，目标是将：

* 机器人系统
* End Effectors（末端执行器）
* 工业现场设备

以及扩展方向中的：

* FastDDS
* ROS 2

以受控方式接入工业侧与 IT 侧接口。

---

最大的价值并不在于：

> “把两个系统直接 1:1 合并”

而在于：

> “分层式整合（layered consolidation）”

其中：

* **SenseKit** 提供：

  * 边缘运行平台
  * Runtime
  * 部署与运维基础设施

* **QNC** 提供：

  * 产品架构
  * 治理模型（Governance）
  * 协议抽象
  * 工业化集成逻辑

这样，两个相邻项目就能整合成：

> 一个基于验证过的嵌入式平台的工业级 Robotics Edge Gateway Stack

---

## 核心结论

从专业和技术角度来看：

> 两者的整合是合理且可行的。

但这种整合更适合定义为：

> “平台 + 产品”的集成

而不是：

> “完全融合成一个单一产品”

因此，我建议推进整合，但要明确区分：

1. Runtime / Hardware Platform（运行时与硬件平台）
2. QNC Core Gateway Services（QNC 核心网关服务）
3. 可选的 Robotics / Perception / Audio 模块

---

# 关键重叠点（Key Overlaps）

## 1）共同目标

两个项目本质上都在解决：

> “Robotics-at-the-Edge（边缘机器人）”

以及：

> “如何把真实设备纳入一个可控、可维护、可扩展的系统”

区别在于：

### SenseKit

侧重：

* CM5 平台落地
* 部署流程
* 实际运行环境

### QNC

侧重：

* 工业化集成产品
* 协议标准化
* Device Profiles
* Fault Handling
* Northbound 接口

---

### 共同问题域

两者都涉及：

* 异构设备 / 外设集成
* 可控的 Edge Runtime
* 低层设备与高层逻辑解耦
* 生命周期 / Deployment 管理
* 机器人网络环境集成
* 可复现、可验证的部署环境

---

## 2）技术、工作流与基础设施重叠

在基础设施与运行逻辑方面，两者高度重叠：

* Docker 部署
* GitLab / Container Registry / Package Registry
* Supervisor / systemd 服务管理
* chrony 时间同步
* Embedded Linux / Debian Runtime
* 可重复 Provisioning
* Installer 自动化

---

### Robotics / Edge Connectivity 重叠

#### SenseKit

包含：

* RealSense
* XVF3800 麦克风
* ROS 相关 Docker Deployment
* 多设备同步

#### QNC

明确提出：

* FastDDS / ROS2 Native Participation
* REST
* WebSocket
* OPC UA
* MQTT

作为 Robot Domain 与 IT/工业接口之间的桥梁。

---

## 3）产品与战略方向相似

两者都在向：

> “模块化、可控的 Edge Integration”

发展，而不是：

> “大而全的单体系统”

共同特点：

* 强调可扩展性
* 强调边界控制
* 避免 “everything connects to everything”
* 将硬件复杂性与业务逻辑解耦

---

### 差异

#### QNC

更偏向：

* 产品定义
* Release Governance
* Scope 管理

#### SenseKit

更偏向：

* 技术实现
* 部署
* 运维平台

而这恰恰构成了两者之间最大的协同点。

---

# 集成机会（Integration Opportunities）

## 1）将 SenseKit 作为 QNC 的参考平台

最自然的整合方式是：

> 将 SenseKit CM5 作为 QNC 的 Reference Hardware + Runtime Platform

因为 SenseKit 已经实现了：

* OS Baseline
* 网络配置
* Docker Runtime
* Supervisor/systemd
* 时间同步
* GitLab Rollout

而这些正是 QNC 作为 Edge Gateway 所需要的基础能力。

---

## 2）将 QNC Core 部署在 SenseKit 上

QNC 的分层架构与 SenseKit 非常匹配：

### Physical & Protocol Adaptation Layer

运行在：

* SenseKit 硬件
* Driver
* Interface Layer

---

### Profile & Semantic Normalization Layer

负责将当前偏项目化的设备集成：

转换成：

* 标准化 Device Profiles
* Semantic Models

---

### Integration & Control Layer

负责：

* ROS2 / FastDDS
* 本地控制逻辑
* Device Access

---

### Northbound Edge Services Layer

提供：

* REST
* WebSocket
* 后续 OPC UA / MQTT

---

## 3）FastDDS / ROS2 作为统一集成锚点

QNC 已明确将：

> FastDDS Domain Participation

作为战略扩展方向。

而 SenseKit 已拥有：

* ROS Runtime
* Robotics Docker Deployment

因此非常适合形成：

> “SenseKit Runtime + QNC DDS/ROS2 + Industrial Northbound Services”

的统一架构。

---

## 4）将 SenseKit 外设纳入 QNC Device Classes

SenseKit 中的：

* XVF3800
* 麦克风
* Backlight
* RealSense

不应只是“特殊硬件”，而应成为：

> QNC 管理的标准 Device Classes

但这需要：

* Device Profiles
* Capability Descriptors
* Health / Telemetry 模型
* Adapter Services

进行语义标准化。

---

## 5）团队与工具链整合

整合后还会产生组织协同：

### 来自 QNC

* Systems Engineering
* Architecture Governance

### 来自 SenseKit

* Embedded / IO Board / CM5 能力

### 共用

* GitLab
* Container Registry
* Release Management
* Device Onboarding Test
* Fault Handling

---

# 推荐合并的组件（Recommended Components to Merge）

# 可直接复用

## 1）SenseKit Runtime 与 Deployment 基础

建议直接复用：

* CM5 Reference Platform
* OS Baseline
* reproducible image
* `sensekit-remote-installer`
* Docker Deployment
* Supervisor/systemd
* chrony
* GitLab + Registry

---

## 2）QNC Governance 与产品逻辑

建议作为统一目标架构：

* Layered Architecture
* Device Profile Governance
* Scope Model
* Functional / Non-functional Requirements
* Fault Handling
* Safe Mode
* Northbound Interfaces
* Release Governance

---

# 需要适配后复用

## 3）SenseKit 外设集成

以下组件不应直接硬编码进入 QNC：

* RealSense
* XVF3800
* Backlight

而应通过：

* Device Profiles
* Capability Models
* Adapter Services

接入 QNC 抽象层。

---

## 4）ROS / DDS 模块

ROS / DDS 非常适合集成，但必须明确分层：

* Robot Internal Communication
* QNC Internal Normalization
* Northbound Mapping

否则 QNC 会失控地演变成：

> “万能 Middleware Bridge”

---

## 5）日志、诊断与远程管理

SenseKit 提供运行基础，QNC 提供 Fault/API 模型。

建议统一为：

* Structured JSON Logs
* API Diagnostics
* Fault History
* Config Audits
* Service Health Monitoring

---

# 合并前必须重构的部分

## 6）安全体系（Security）

必须优先修复：

* 默认账号密码
* SSH Password Login
* `auth-anonymous=1`
* 缺乏 Security Governance
* API Access Control 不完善

---

## 7）工业协议栈

QNC 的核心价值在于：

* IO-Link
* Modbus
* EtherNet/IP
* OPC UA
* MQTT
* CANopen
* FastDDS

而这些在 SenseKit 中尚未形成完整能力。

因此必须：

> 新开发或大幅扩展相关模块

---

# 风险与缺口（Risks & Gaps）

## 1）“产品”和“平台”概念混淆

最大的管理风险是：

> 过早将两者视为“同一个产品”

实际上：

### SenseKit

是：

* 平台
* Setup
* Runtime

### QNC

是：

* Gateway Product
* 产品规范

错误融合会导致：

* QNC 失去边界
* SenseKit 变得过重

---

## 2）不是所有 SenseKit 功能都适合进入 QNC

QNC 明确指出：

它不是：

* General AI Platform
* Robot Controller

因此：

* Vision Inference
* Audio Inference
* UX Logic

等功能不适合放入 QNC Core。

更适合作为：

> Optional Edge Applications

运行在 QNC Core 之上。

---

## 3）工业级质量要求差距

QNC 已定义：

* Latency
* Fault Isolation
* Safe Mode
* Availability
* Controlled Releases

而 SenseKit 目前更偏：

* 运维机制

尚未达到：

> 工业产品级验证治理（Product-grade Governance）

---

## 4）硬件与 I/O 缺口

QNC 已面向：

* IO-Link
* Modbus
* EtherNet/IP
* EtherCAT
* PROFINET

而 SenseKit 当前主要覆盖：

* CM5
* Audio
* PWM
* Camera
* Docker

因此：

* Industrial Transceiver
* Interface Boards
* Co-processors

可能仍需补充。

---

## 5）安全债务（Security Debt）

SenseKit 中的一些内部开发习惯：

* Default Credentials
* Open Network Assumptions
* Weak Authentication

对于工业产品来说风险较高。

整合前必须治理。

---

# 建议的统一项目结构（Suggested Unified Project Structure）

# 1）目标架构

建议采用三层架构：

---

## A. Platform Layer

负责：

* CM5 / Hardware
* OS Image
* Provisioning
* Docker Runtime
* systemd / supervisor
* 网络 / 时间同步
* CI/CD
* Driver 与基础外设

---

## B. QNC Core Gateway Layer

负责：

* Protocol Adapters
* Device Profiles
* Semantic Normalization
* Fault Handling
* Config Governance
* REST / WebSocket
* OPC UA / MQTT
* DDS/FastDDS

---

## C. Edge Applications / Domain Modules

负责：

* Vision / RealSense
* Audio / XVF3800
* Robotics-specific Logic
* Analytics
* Cloud Use Cases
* Customer-specific Extensions

---

# 2）组织结构建议

建议划分四条工作流：

## Platform Engineering

负责：

* CM5
* OS
* Installer
* Security Hardening
* Base Services

---

## Gateway Product Engineering

负责：

* QNC Core
* Industrial Protocols
* Device Profiles
* Northbound Services

---

## Robotics & DDS Integration

负责：

* ROS2 / FastDDS
* Topic Governance
* Schema Validation
* Controlled Bridging

---

## Application Modules / Solution Engineering

负责：

* RealSense
* 麦克风
* 客户定制设备
* 行业解决方案

---

# 3）推荐合并顺序

## 第一阶段：统一平台

先稳定并强化 SenseKit Runtime 与 Deployment。

---

## 第二阶段：迁移 QNC Core

引入：

* REST/WebSocket
* Logging
* Config Governance
* Fault Model
* Device Profiles

---

## 第三阶段：集成 DDS/ROS2

接入：

* FastDDS / ROS2
* 第一批设备 Profile

---

## 第四阶段：工业扩展

扩展：

* OPC UA
* MQTT
* 工业协议
* 工业 I/O
* Validation / Release 流程

---

# 最终建议（Final Recommendation）

## 建议：整合，但采用清晰分层的受控整合模式

不建议：

> “把 SenseKit 与 QNC 直接混成一个模糊产品”

而应该定义为：

> SenseKit = Embedded Edge Platform
> QNC = 基于该平台的标准化 Connectivity & Gateway Product

---

## 可以直接整合的部分

* CM5 平台
* Provisioning
* Docker
* Supervisor/systemd
* chrony
* GitLab / Registry
* ROS / DDS Runtime
* Logging 与 Deployment 机制

---

## 应作为统一产品核心的部分

* QNC Layered Architecture
* Device Profile Governance
* Controlled Northbound Services
* FastDDS / ROS2
* Fault / Safe Mode
* Release Governance

---

## 合并前必须重构的部分

* Security Baseline
* Audio / Vision 的产品定位
* QNC Core 与 Optional Apps 的边界
* 工业协议支持
* Product-grade Testing & Validation

---

# 总体判断

这两个项目：

> 高度协同，但并不完全重合。

最佳方案是：

> “共享平台骨干（Platform Backbone）+ 以 QNC 为核心的产品层”

这样既能减少：

* Runtime
* Deployment
* Edge Operation
* Robotics Integration

方面的重复投入，

又不会削弱：

> QNC 清晰的产品逻辑与工业化定位。

---
---
# Executive Document: SenseKit and QNC Integration Strategy

## Executive Summary

This document evaluates the strategic integration of SenseKit and QNC, two distinct yet closely related platforms. While not identical products, SenseKit, an embedded/edge device platform based on CM5, and QNC, an industrial-grade Connectivity Gateway solution, possess significant integration potential. The core recommendation is a **layered consolidation** rather than a direct merger, positioning SenseKit as the foundational platform and QNC as the standardized connectivity and gateway product built upon it. This approach leverages their respective strengths to create a robust Industrial Robotics Edge Gateway Stack, optimizing resource allocation and maintaining clear product boundaries.

## Key Overlaps and Synergies

Both SenseKit and QNC address the critical domain of "Robotics-at-the-Edge" and the challenge of integrating real-world devices into controllable, maintainable, and scalable systems. 

**SenseKit** focuses on hardware implementation, OS, deployment, and runtime infrastructure on the CM5 platform, including peripherals like audio/camera and Docker environments. 

**QNC** emphasizes industrial integration, protocol standardization, device profiles, and fault handling, acting as a bridge between robot domains and IT/industrial interfaces, supporting technologies like FastDDS and ROS 2.

Significant technical and workflow overlaps exist in areas such as Docker deployment, GitLab-based CI/CD, supervisor/systemd services, chrony time synchronization, and embedded Linux runtimes. Strategically, both projects are evolving towards modular, controlled edge integration, prioritizing scalability and boundary control over monolithic systems.

## Integration Opportunities

1.  **SenseKit as QNC's Reference Platform**: SenseKit's established OS baseline, network configuration, Docker runtime, and deployment infrastructure make it an ideal reference hardware and runtime platform for QNC.
2.  **Deployment of QNC Core on SenseKit**: QNC's layered architecture (Physical & Protocol Adaptation, Profile & Semantic Normalization, Integration & Control, Northbound Edge Services) aligns well with SenseKit's capabilities, allowing QNC core services to run efficiently on the SenseKit platform.
3.  **FastDDS / ROS2 as Unified Integration Anchor**: SenseKit's existing ROS runtime and Docker deployments, combined with QNC's strategic focus on FastDDS/ROS2, create a strong foundation for a unified architecture.
4.  **Integrating SenseKit Peripherals into QNC Device Classes**: SenseKit's specific peripherals (e.g., XVF3800, RealSense) should be standardized into QNC's managed Device Classes through profiles, capability descriptors, and adapter services.
5.  **Team and Toolchain Consolidation**: Integration fosters organizational synergy, combining QNC's systems engineering and architecture governance with SenseKit's embedded/CM5 expertise, utilizing shared tools like GitLab and unified release management.

## Risks and Gaps

1.  **Conceptual Confusion**: A primary risk is prematurely treating SenseKit and QNC as a single product. SenseKit is a platform/runtime, while QNC is a gateway product. Blurring these lines could lead to QNC losing its defined scope and SenseKit becoming overly complex.
2.  **Unsuitable SenseKit Features for QNC**: Features like general AI platforms, robot controllers, audio inference, or UX logic from SenseKit are not suitable for the QNC Core and should be managed as optional edge applications.
3.  **Industrial Quality Requirements Gap**: SenseKit's current operational mechanisms do not yet meet QNC's industrial product-grade governance for latency, fault isolation, safe mode, availability, and controlled releases.
4.  **Hardware and I/O Gaps**: QNC's focus on industrial protocols (IO-Link, Modbus, EtherNet/IP) highlights a gap with SenseKit's current CM5, audio, PWM, and camera coverage, necessitating additional industrial transceivers and interface boards.
5.  **Security Debt**: SenseKit's existing security practices (default credentials, open network assumptions, weak authentication) pose high risks for an industrial product and require significant remediation before integration.

## Recommended Unified Project Structure

### Target Architecture (Three-Layered)

*   **Platform Layer**: Handles CM5/hardware, OS, provisioning, Docker runtime, systemd/supervisor, networking, CI/CD, and basic drivers.
*   **QNC Core Gateway Layer**: Manages protocol adapters, device profiles, semantic normalization, fault handling, config governance, and northbound services (REST, WebSocket, OPC UA, MQTT, DDS/FastDDS).
*   **Edge Applications / Domain Modules**: Encompasses vision/RealSense, audio/XVF3800, robotics-specific logic, analytics, and customer-specific extensions, running atop the QNC Core.

### Organizational Structure (Four Workflows)

*   **Platform Engineering**: Focuses on CM5, OS, installer, security hardening, and base services.
*   **Gateway Product Engineering**: Manages QNC Core, industrial protocols, device profiles, and northbound services.
*   **Robotics & DDS Integration**: Oversees ROS2/FastDDS, topic governance, schema validation, and controlled bridging.
*   **Application Modules / Solution Engineering**: Handles RealSense, microphones, custom devices, and industry solutions.

## Phased Integration Approach

1.  **Phase One: Unified Platform**: Stabilize and strengthen SenseKit's runtime and deployment.
2.  **Phase Two: Migrate QNC Core**: Introduce QNC's REST/WebSocket, logging, config governance, fault model, and device profiles.
3.  **Phase Three: Integrate DDS/ROS2**: Incorporate FastDDS/ROS2 and initial device profiles.
4.  **Phase Four: Industrial Expansion**: Extend support for OPC UA, MQTT, industrial protocols, and I/O, alongside product validation and release processes.

## Final Recommendation

The optimal strategy is a **controlled, layered integration** where SenseKit serves as the Embedded Edge Platform, and QNC is the standardized Connectivity & Gateway Product built upon it. This approach avoids redundant efforts in runtime, deployment, edge operations, and robotics integration, while preserving QNC's clear product logic and industrial positioning. Key areas for direct integration include the CM5 platform, provisioning, Docker, Supervisor/systemd, chrony, GitLab/Registry, ROS/DDS runtime, and logging/deployment mechanisms. The QNC layered architecture, device profile governance, controlled northbound services, FastDDS/ROS2, fault/safe mode, and release governance should form the core of the unified product. Critical pre-merger refactoring is required for security baselines, product positioning of audio/vision, defining boundaries between QNC Core and optional apps, industrial protocol support, and product-grade testing/validation.
