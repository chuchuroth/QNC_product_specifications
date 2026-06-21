# QNC 工业边缘网关 PCB / 原理图设计资料（中文翻译版）

**说明**：本文档由 `PCB_design.md`、`Schaltplan-Hierarchie.md`、`Sheet-to-Sheet-Verbindungsmatrix.md` 全文翻译整合而成。技术名词在首次出现时酌情附德文或英文原文（括号）。文末附译者注（可选优化说明）。

---

# 第一部分：PCB 设计（原文档：PCB_design.md）

在审阅四份 QNC 文档后，核心结论是：规范非常清晰地定义了**系统功能**、**协议边界**、**基线（Baseline）与扩展（Extension）范围**、**安全模式（Safe Mode）/故障隔离**、**配置治理**以及**环境要求**，但**并未**给出最终 PCB 所需的细节数据，例如确切连接器、引脚分配、供电电压、各变体的端口数量、电流预算、绝缘耐压、外壳/IP 防护以及机械边界条件。因此，以下方案是基于上述文档的**接近量产的参考设计计划**——在资料未明确之处，以**显式假设**标出。[QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb) [PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik) [DPS](https://www.genspark.ai/api/files/s/MSkOUBCE)

---

## 1）系统解读与设计假设

将 QNC 解读为**软件优先（software-first）的工业边缘网关**：基线（Baseline）支持 **IO-Link、Modbus RTU、EtherNet/IP 适配器及离散数字 I/O**；向北（northbound）提供 **REST、WebSocket 与结构化日志**；并应具备 **FastDDS、Modbus TCP、CANopen、OPC UA、MQTT** 等扩展能力。系统**非安全认证级（nicht safety-rated）**，但必须支持故障隔离、安全模式、配置签名、回滚、诊断接入以及稳健的工业现场安装。对硬件而言，即：支持 Linux 的计算平台、分离的现场接口、清晰的故障包容（Fault Containment）分区、便于维护的诊断路径，以及足以覆盖基线与可选扩展的 I/O/网络余量。[QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb) [PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

### 设计冻结 A（Design Freeze A）的推荐硬件规格

| 假设项 | 建议 |
|---|---|
| 输入供电 | 24 VDC 标称，允许 18…36 VDC |
| 计算架构 | Linux 计算模块 + 独立的实时/监控 MCU（Supervisory MCU） |
| 以太网 | 2× 千兆以太网，经磁性元件（Magnetics）与线缆侧电气隔离 |
| Modbus RTU | 2× 隔离 RS-485 |
| IO-Link | 4× 主站端口（Master-Port） |
| 离散 I/O | 8× DI + 8× DO，可按组（bankweise）配置 |
| 扩展预留 | 1× CAN/CANopen，布局预留，可选贴片 |
| 维护 | USB-C 维护/控制台、隔离调试接头、启动恢复（Boot-Recovery） |
| 安装 | DIN 导轨或面板式，工作温度 0…50 °C |
| 目标产品形态 | 工业计算模块的载板（Carrier-/Baseboard） |

上述假设与文档记载的范围、环境 0…50 °C / 污染等级（Pollution Degree）2、至少多台设备/客户端并发、基线协议及扩展路径一致，但**不能替代**仍待敲定的硬件细节。[PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

### 架构决策

针对首次可量产迭代，建议**不要**采用裸 MPU + DDR 的单板一体化方案，而采用**工业级 SoM（System on Module）+ 基板**。这可降低 DDR/PMIC/BGA 风险、缩短布局与信号完整性（SI）周期；在底层硬件数据尚缺的情况下，这是通往 EVT/DVT/PVT 最稳健的路径。亦与文档性质相符：QNC 资料主要规定系统与接口行为，而非固定 CPU 平台。[PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

---

## 2）原理图分区与层次

### 推荐的原理图页（Sheet）层次

1. **Sheet 01 – 电源入口与保护（Power Entry & Protection）**  
   24 V 输入、反接保护、热插拔/浪涌电流（Inrush）、TVS、EMI 滤波、熔断器、供电监测。

2. **Sheet 02 – 计算核心（Compute Core）**  
   工业 SoM、启动配置（Boot-Straps）、复位、TPM、RTC、看门狗接口、状态 LED。

3. **Sheet 03 – 监控/实时 MCU（Supervisory / RT MCU）**  
   状态监测、电源轨监测、板卡 ID、恢复、欠压（Brownout）处理、硬件看门狗。

4. **Sheet 04 – 北向以太网（Northbound Ethernet）**  
   2× 以太网端口、磁性元件、ESD、屏蔽/外壳（Chassis）连接、可选链路/活动 LED。

5. **Sheet 05 – 南向 RS-485 / Modbus RTU**  
   2× 隔离 RS-485 通道、端接、偏置、保护电路、诊断。

6. **Sheet 06 – IO-Link 主站**  
   4× 主站端口、端口供电 L+、C/Q 驱动、短路/热保护、端口 ID。

7. **Sheet 07 – 离散数字 I/O**  
   8× DI、8× DO，可选按组源输出/灌电流（sourcing/sinking）、隔离/保护层、状态诊断。

8. **Sheet 08 – CAN / CANopen 扩展**  
   可选贴片的 CAN 接口，含总线保护与端接。

9. **Sheet 09 – 维护、调试、恢复（Service, Debug, Recovery）**  
   USB-C 设备/控制台、UART、SWD/JTAG、恢复接头、出厂配置 DIP。

10. **Sheet 10 – 安全与非易失状态（Security & Nonvolatile State）**  
    TPM、安全身份/板卡序列号存储、可选安全元件（Secure Element）、事件计数器。

11. **Sheet 11 – 测试与量产支持（Test & Production Support）**  
    边界测试点、针床（Bed-of-Nails）接入、电流检测分流电阻、回环（Loopback）选项。

该分区映射了文档要求的物理/协议适配、归一化、控制、诊断与维护之间的分离——在硬件层清晰落实。[QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

### 板级功能域

- **计算域**：Linux/QNC 运行时、REST/WebSocket、日志、DDS 可选。
- **监控域**：对板卡/电源/故障的确定性监视。
- **现场总线域**：Modbus RTU、CANopen 可选、IO-Link、DIO。
- **维护域**：更新、诊断、恢复。
- **电源域**：逻辑、接口、隔离现场侧的独立电源轨。

由此可在物理层面支持故障包容：RS-485 或 IO-Link 故障不得干扰计算域或其他端口——这与 QNC 故障模型一致。[QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

---

## 3）详细 BOM 建议

重要：此为落实文档功能的**推荐参考 BOM**，**并非** QNC 文档规定。连接器、部分功率级与最终 DIO 拓扑在接口发布前仍待定。

### 关键有源器件

| 功能块 | 推荐器件 | 功能 | 关键参数 | 采购提示 |
|---|---|---|---|---|
| 计算 SoM | **Toradex Verdin iMX8M Plus Quad 8GB WB IT** | Linux/边缘/DDS/REST/WebSocket | 工业 SoM，降低 DDR 风险 | 核对厂商/分销商，优先长期供货 [Toradex](https://www.toradex.com/computer-on-modules/verdin-arm-family/nxp-imx-8m-plus?srsltid=AfmBOoqKgcokkZBOHE2eLQYVikifUyZdY12uMnYiCEbgnh8hNAinrX0U) |
| 备选 SoM | **UCM-iMX8M-Plus** | 备选计算平台 | i.MX 8M Plus SoM | 仅作备用货源认证 [CompuLab](https://www.compulab.com/products/computer-on-modules/ucm-imx8m-plus-nxp-i-mx-8m-plus-som-system-on-module-computer/) |
| 监控 MCU | **STM32G474RET6** | 电源/复位/看门狗/健康监控 | 工业级，足够 ADC/定时器/I/O | 难以直接第二货源，需核对生命周期 |
| TPM | **OPTIGA-TPM-SLB-9672-FW15** | 安全启动/密钥/证明（Attestation） | SPI TPM 2.0 | 安全 BOM 单独版本管理 [Infineon](https://www.infineon.com/part/OPTIGA-TPM-SLB-9672-FW15) |
| IO-Link 主站 | **MAX14819ATM+** ×2 | 4 个主站端口（2 片×2 端口） | 集成成帧器/L+ 控制 | 关键长周期器件，尽早备货 [ADI](https://www.analog.com/en/products/max14819.html) |
| RS-485 | **ISO1410DWR** ×2 | 2 个隔离 Modbus RTU 端口 | 5 kVrms 隔离，EMC 稳健 | 工业前端优选 [TI](https://www.ti.com/product/ISO1410) |
| CAN（扩展） | **TCAN1042HVDRQ1** | CAN/CANopen 就绪 | 故障保护型 CAN 收发器 | 仅扩展获批后贴片 [TI](https://www.ti.com/product/TCAN1042HV-Q1/part-details/TCAN1042HVDRQ1) |
| 数字输入 | **ISO1212DBQR** ×4 | 8 路隔离 24 V DI | 2 通道/IC，工业输入接收器 | 接近 IEC 61131 的输入拓扑 [TI](https://www.ti.com/product/ISO1212) |
| 数字输出（源输出） | **TPS272C45CRHFR** ×4 | 8 路高边 DO | 36 V，3 A，智能高边 | 仅 sourcing 最终获批后采用 [TI](https://www.ti.com/product/TPS272C45) |
| DIO 备选 | **MAX14900EAGM+CKT** ×2 | 灵活 DIO 拓扑 | 每通道可配置，工业 DO/DIO | 核对逻辑模型是否与 ICD 一致 [ADI](https://www.analog.com/en/products/max14900e.html) |
| 热插拔/浪涌电流 | **LM5069MM-2/NOPB** | 24 V 输入保护 | 热插拔/浪涌/故障切断 | 稳健工业电源入口必备 [TI DS](https://www.ti.com/lit/ds/symlink/lm5069.pdf) |
| 主 Buck | **LM76005RNPR** | 24 V → 5 V 主供电 | 60 V 输入耐压，5 A | 24 V 母线侧稳健性好 [TI](https://www.ti.com/product/LM76005/part-details/LM76005RNPR) |
| 隔离 DC/DC | **NXJ2S0505MC-R13** ×2…4 | 现场侧隔离辅助电源 | 5 V / 2 W 隔离 | 数量按通道分组 [Murata](https://pim.murata.com/en-us/pim/details/?partNum=NXJ2S0505MC) |
| RTC | **RV-3028-C7** | 时间戳/审计/恢复 | 超低功耗 RTC | 可选但推荐 |
| ESD 阵列 | **TPD4E05U06QDRQ1** | 维护/数据线 ESD 保护 | 低电容 ESD | 以太网需单独保护策略 |
| 24 V 输入 TVS | **SMBJ33A** | 浪涌/瞬态钳位 | 600 W 等级 | 按浪涌等级选厂牌 |
| 共模扼流圈 | Würth / TDK 工业系列 | 传导发射 | 各接口差异化 | 预合规（Pre-Compliance）后最终调参 |
| 以太网连接器 | **待外壳确定（TBD）** | 屏蔽 RJ45 或 M12 X-coded | 机械未定 | 无机械发布则无法定稿 |
| 现场连接器 | **待机械确定（TBD）** | IO-Link / RS-485 / DIO / CAN | 螺钉端子或 M8/M12 | 取决于机械与 IP |

### 无源 BOM 规则与容差

| 无源类别 | 建议 |
|---|---|
| 24 V 路径 | 耐压 50 V 或 100 V；MLCC 用 X7R；大容量处聚合物/电解 |
| 5 V/3V3 去耦 | 0.1 µF + 1 µF + 10 µF 分级，X7R/X5R，10% 通常足够 |
| 反馈/检测网络 | 标准 1%；电压/电流测量用 0.1%/25 ppm |
| 总线端接 | RS-485/CAN 120 Ω，1%，功率≥0.25 W；以太网按 PHY/磁性元件应用笔记 |
| 安全相关诊断 RC 滤波 | 若触发阈值紧，用 1%/C0G 或稳定 X7R |
| Boot/Reset 上拉下拉 | 按绑带要求选 1% 或 5% |
| 电流检测分流 | 1%，1 W 至 3 W 视端口电流，建议 50 ppm |

### 采购策略

- **A 类物料**：SoM、IO-Link、隔离 RS-485、TPM、电源前端尽早按 LTB/LTS（最后购买/长期供货）锁定。
- **B 类物料**：保护器件、隔离模块、DIO 驱动双货源。
- **C 类物料**：无源统一 0402/0603/0805，允许厂商池。
- **禁止**：无生命周期批准的 NCNR（不可取消不可退货）物料；安全器件非授权 broker。

---

## 4）PCB 叠层（Stack-up）

计算侧采用 SoM 时，载板采用**6 层受控阻抗叠层**即可，在成本、可制造性与 SI/EMI 安全性之间折中最优。

### 推荐叠层

| 层 | 功能 | 铜厚 | 说明 |
|---|---|---:|---|
| L1 | 器件 + 关键信号 | 35 µm | 短的服务/管理/差分对 |
| L2 | 完整 GND 平面 | 35 µm | L1/L3 参考层 |
| L3 | 电源 + 低速信号 | 18 µm | 24 V / 5 V / 3V3 分区 |
| L4 | 受控信号/差分对 | 18 µm | 以太网/USB/MCU 互连 |
| L5 | 完整 GND/外壳耦合 | 35 µm | 屏蔽与回流 |
| L6 | 器件 + 现场信号 | 35 µm | 收发器、连接器、保护 |

### 材料与制造目标

- **FR-4，Tg ≥ 170 °C**
- 总厚度 **1.6 mm**
- 外层 **1 oz**，内层 **0.5 oz**
- 受控阻抗测试条（Coupon）：
  - 50 Ω 单端
  - 若使用 USB 2.0：90 Ω 差分
  - 以太网/高速差分：100 Ω 差分
  - CAN/RS-485 线路模型在板上不必 120 Ω 标称，但保持受控对称有益

适用于工业网关、高速占比有限、需清晰回流路径与 EMC 余量的场景。[PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

---

## 5）高速与敏感布线指南

### 通用布线规则

- 高速信号下不得分割 GND 回流路径。
- 屏蔽/外壳连接应在连接器处直连，**勿**深入板内再接地。
- 现场接口靠板边，计算/TPM/维护靠中部。
- 隔离屏障以 Keep-out + 爬电（Creepage）沟槽几何上显式呈现。
- 每条外线先经 **ESD/浪涌/滤波**，再进入 IC。

### 以太网

- 若 SoM 集成 PHY：MDI 对至磁性元件模块短、对称，无短桩线（Stub）。
- 100 Ω 差分，对内等长差小，优先同层。
- 避免尖锐 90°；用 45°/圆弧。
- Bob-Smith/屏蔽网络严格按磁性元件/连接器应用笔记。
- 外壳屏蔽在端口附近按 360° 与外壳概念配合。

### RS-485 / CAN

- 靠近端口端接；偏置每段总线仅一处。
- TVS + 可选共模扼流圈靠近端口。
- 隔离器/隔离收发器置于逻辑侧与现场侧之间。
- 总线勿与 24 V Buck 的强开关节点长距离平行。

### IO-Link

- 各端口结构一致、布局对称。
- L+ 电流路径宽、散热充分、单独保护。
- C/Q 线短、低干扰、回流路径明确。
- 避免多端口回流共用瓶颈。

### 敏感网络

- TPM-SPI、复位、Boot 绑带、监控信号远离现场线缆。
- ADC/检测线采用开尔文（Kelvin）走线。
- 晶振/RTC 靠近 IC，极敏感网络可加保护环（Guarding）。

上述规则源自文档对故障隔离、可诊断性、更新安全与稳健现场接口的要求。[QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

---

## 6）PDN（配电网络）设计

### 推荐电源树

| 电源轨 | 来源 | 典型负载 | 备注 |
|---|---|---:|---|
| 24V_IN | 外部工业输入 | 全系统 | 受保护、受监测 |
| 24V_FIELD | 自 24V_IN 分路保护 | IO-Link/DIO/现场 | 每端口/每组单独保护 |
| 5V_MAIN | 自 24V_IN 经 LM76005 | SoM + 逻辑 | 中央逻辑轨 |
| 5V_ISO_A | 隔离 DC/DC | RS-485/CAN 组 A | 现场隔离 |
| 5V_ISO_B | 隔离 DC/DC | RS-485/CAN 组 B | 现场隔离 |
| 3V3_LOGIC | 自 5V_MAIN | MCU/TPM/胶合逻辑 | 本地滤波 |
| 1V8_AUX | 若需要 | 电平转换/辅助 | 仅外设需要时 |

### PDN 规则

- 24 V 入口：热插拔、反接保护、TVS、π型（Pi）滤波。
- SoM 置于独立洁净 5 V 岛，模块近旁大容量缓冲。
- 每现场端口本地限流或熔断。
- 在 24V_IN、5V_MAIN 及至少一组现场母线上做电流测量。
- 监控 MCU 监测欠压/过压、浪涌故障、热故障与端口过流。
- 电源从电源中心星形分布，勿在整板 Daisy Chain。

### 尺寸建议

- 24V_IN 预算：持续 **2 A** 作为起点
- 5V_MAIN：**4–5 A**
- 24V_FIELD：视端口配置每组 **0.5–1 A**
- 3V3_LOGIC：**1–2 A**
- 余量：**电气 30%**，**热 20%**

支撑文档中的恢复、故障处理、诊断与多设备运行目标。[PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb)

---

## 7）接地与屏蔽策略

建议三套参考系统：

1. **DGND / 逻辑地**  
   SoM、MCU、TPM、内部逻辑。

2. **FGND / 现场隔离地**  
   隔离 RS-485/CAN/部分 DIO 区域。

3. **CHASSIS / PE**  
   屏蔽、连接器外壳、浪涌泄放路径。

### 规则

- 电缆屏蔽**始终**先接 **CHASSIS**，勿直接接 DGND。
- CHASSIS 与 DGND 经定义的 RC/HF 耦合，勿大面积硬短接。
- 隔离分界清晰，爬电/电气间隙（Creepage/Clearance）充分。
- TVS 回流引至合适参考电位；勿无控跳至 DGND。
- LED/调试/维护地勿穿过现场电流路径。

从而在传导干扰下仍支持文档要求的故障包容。[ICD](https://www.genspark.ai/api/files/s/Fjro3Pik) [QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb)

---

## 8）热分析与散热

### 冻结 A 粗略功耗估算

| 功能块 | 粗略损耗 |
|---|---:|
| SoM | 6…10 W |
| 电源前端 + Buck | 1.5…3 W |
| IO-Link（4 端口，无负载） | 1…2 W |
| RS-485 / CAN / DIO 逻辑 | 1…2 W |
| 负载相关 DO 损耗 | 强可变，0…6 W |

### 热概念

- SoM 经散热片（Heatspreader）耦合至外壳或金属支架。
- 电源 IC、IO-Link 驱动、高边开关下方密集热过孔（Thermal Vias）。
- L1/L6 大面积铜皮 + 内层导热。
- 现场连接器勿紧贴 SoM 热源。
- DIN 导轨封闭外壳：尽量支持垂直对流。

### 热验证

- 布局冻结前做轻量 CFD/仿真估算。
- 24 V、满载、环境温度 50 °C 下实测。
- 热点：无源磁性/ESD 簇 < 105 °C；有源功率器件结温 < 125 °C。
- 最坏情况：全部 DIO 高边导通 + DDS/以太网流量 + IO-Link 遥测。

QNC 资料给出 0…50 °C；板级设计建议至少 **10 °C** 裕量。[PRS](https://www.genspark.ai/api/files/s/wjz4yo8M)

---

## 9）EMI/EMC 合规

文档未列具体标准。对工业网关建议以下**目标标准**：

- **EN 55032 / CISPR 32 Class A**
- **EN 61000-6-2**（工业抗扰度）
- **IEC 61000-4-2** ESD
- **IEC 61000-4-4** EFT（快速瞬变脉冲群）
- **IEC 61000-4-5** Surge（浪涌）
- **IEC 61000-4-6** 传导射频抗扰度

### 设计措施

- 所有外线加 TVS。
- 预合规合理处加共模扼流圈。
- 开关稳压器缩小热回路（Hot Loop）、屏蔽地回流、预留缓冲吸收（Snubber）焊盘。
- 现场/逻辑分区分离。
- 外壳/屏蔽概念尽早与机械协同。
- 时钟源短且参考干净。
- 域交界可选铁氧体磁珠，**不能**替代良好 PDN 拓扑。

### EMC 测试计划

- 先对裸板与目标外壳做 **Pre-Compliance**。
- 再针对 CMC、RC Snubber、屏蔽接地做调参。
- 重点：以太网发射、24 V 输入传导、IO-Link/DIO 突发抗扰度。

QNC 要求安全通信、诊断与可用性，EMC 稳健前端是核心而非可选项。[PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

---

## 10）DFM/DFA 与制造提示

### DFM（可制造性设计）

- 目标：量产 **IPC Class 2**；仅客户/行业要求时 Class 3。
- 起始设计规则：**6/6 mil**；仅 SoM 载板扇出需要时用激光盲孔。
- 除 SoM 厂商要求外，避免盘中孔（Via-in-pad）。
- 1.6 mm 板厚、定义铜平衡、控制翘曲。
- 全局 + 局部基准点（Fiducials）。
- 阻抗测试条。
- 细间距 QFN 优化阻焊开窗。

### DFA（可装配性设计）

- 连接器仅布于外缘。
- 维护/恢复接口无需拆卸整系统即可触及。
- LED 正面可读。
- DIN 导轨与面板安装方向一并考虑。
- 螺钉端子或 M12 预留足够工具操作空间。

### 工艺说明

- 三防漆（Conformal Coating）可选；污染等级 2 通常不强制，视客户而定。
- 无真实需求勿用底部填充（Underfill）。
- 非机械强制避免选择性焊接。
- EMC 高风险处为保护器件预留替代封装焊盘。

文档未规定制造标准；以上为务实工业 DFM 基线。[PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

---

## 11）测试点、调试与验证

### 必备测试点

- 24V_IN、24V_FIELD、5V_MAIN、3V3_LOGIC、各隔离 5 V 轨
- 复位、Boot Mode、看门狗踢狗（Kick）、故障汇总
- RS-485 A/B 每通道
- CANH/CANL
- IO-Link 每端口 C/Q + L+
- DIO 组输入/输出
- UART 控制台、SWD/JTAG、TPM SPI 接头可选不贴
- 屏蔽至外壳（Chassis）测量点

### 调试策略

- 监控 MCU 作为**板卡健康监视器**，独立故障日志。
- Linux 控制台与应用网络分离。
- 恢复启动无需专用夹具（Jig）。
- 软件逐端口使能/禁用 + 硬件静默路径。
- 每组电流测量便于快速定位故障。

### 验证计划

**EVT**  
- 上电、欠压、浪涌电流、基础 ESD  
- Linux 启动、SoM 健康、与监控握手  
- 端口枚举与基本通信  

**DVT**  
- IO-Link 主站测试  
- Modbus RTU 长期测试  
- EtherNet/IP 吞吐量/链路恢复  
- 安全模式响应链  
- 回滚/更新恢复  
- 安全：签名/TPM/启动路径  

**PVT**  
- 生产测试  
- 温度循环  
- 72 h 老化  
- 预合规 + 回归  

### QNC 专项验收准则

- 严重故障 → **安全模式 ≤ 1 s**
- 重启 → **≤ 30 s** 恢复运行
- 目标路径 REST 至设备延迟 **≤ 50 ms**
- DDS/扩展故障不影响基线南向（Southbound）

直接源自文档 NFR/故障/生命周期要求。[PRS](https://www.genspark.ai/api/files/s/wjz4yo8M) [QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb) [ICD](https://www.genspark.ai/api/files/s/Fjro3Pik)

---

## 12）风险、权衡与仍缺信息

### 最大项目风险

| 风险 | 影响 | 建议 |
|---|---|---|
| 连接器/引脚未发布 | PCB 无法定稿 | 机械 + ICD 硬件附录补齐 |
| 输入电压/电流预算不清 | 电源可能超配或欠配 | 24 V 系统预算书面批准 |
| DIO sourcing/sinking 未精确定义 | 输出拓扑错误 | 立即决定按组或按通道配置 |
| 绝缘要求不清 | 安全/EMC/成本风险 | 至少规定工作电压与耐压（HiPot）目标 |
| 端口数量不清 | 面积与热风险 | 定义产品变体矩阵 |
| 连接器标准不清（RJ45 vs M12，端子 vs 圆形） | 机械/EMV 返工 | 尽早固定 |
| FastDDS 端口共享未澄清 | 网络拓扑不明 | 专用口 vs VLAN vs 共享网卡 |

### 重要权衡

- **SoM vs 分立 MPU**：SoM 快、风险低、单价高；分立量产单价低但 NRE 与 SI 风险高。  
- **全隔离 vs 部分隔离**：隔离越多越稳健/易包容故障，但占面积、BOM、发热。  
- **2 以太网 vs 3**：2 口通常满足基线；3 口利于 DDS/维护/现场解耦，但成本与外壳复杂度上升。  
- **4 vs 8 IO-Link**：4 口对冻结 A 现实；8 口显著推高电流、热与 PCB 面积。  
- **RJ45 vs M12**：RJ45 成本低、实验室友好；M12 工业更稳，但机械与成本更高。

### 定稿前仍缺信息

1. 精确**供电规格**：标称/最小/最大、欠压、瞬态  
2. 最终**连接器族**与**引脚定义**  
3. 各产品变体**端口数量**目标  
4. **绝缘强度**与爬电/电气间隙目标  
5. 机械：板尺寸、固定、外壳、IP 目标  
6. 法规目标：CE/FCC/UL/UKCA、工业 EMV 标准  
7. DIO 电气：电平、负载、sourcing/sinking 概念、防护等级  
8. 以太网拓扑：北向/南向/DDS 是否专用端口  
9. 维护概念：本地 USB、仅以太网或二者兼有  
10. 制造目标：IPC 等级、涂覆、可追溯性、批量  

这些缺口并非次要；它们恰在 QNC 文档有意排除的硬件细节范围之外。[ICD](https://www.genspark.ai/api/files/s/Fjro3Pik) [DPS](https://www.genspark.ai/api/files/s/MSkOUBCE) [QNC-PSM](https://www.genspark.ai/api/files/s/C2VslvCb)

---

## 13）明确建议

若需将本项目推进至量产，建议下一步：

- **固定架构**：确认 SoM 载板方案  
- **关闭开放项**：供电、连接器、绝缘目标、DIO 拓扑  
- **建立设计冻结 A**：6 层载板、2× GbE、2× RS-485、4× IO-Link、8×DI/8×DO、可选 CAN  
- **EVT 板**：预留 EMC/热裕量  
- 随后 **DVT**：最终外壳 + 预合规  

---

# 第二部分：原理图层次结构（原文档：Schaltplan-Hierarchie.md）

## QNC 设计的完整原理图层次结构（中文翻译版）

**适用于 CAD 的完整原理图层次结构**，包含**顶层块结构**、**建议的页边界**、**层次化网络名**、**各页之间的接口**以及**每页的信号方向**。在计划有意留空之处，将逻辑接口冻结，仅将连接器/机械细节作为后续冻结点。与 PCB 计划假设一致：24 V 工业供电、SoM + 监控 MCU、2× GbE、2× 隔离 RS-485、4× IO-Link 主站、8× DI + 8× DO、可选 CAN、维护/恢复、TPM/RTC，以及清晰分离的故障包容域。

---

## 1. 层次化总体结构

```text
00_TOP_QNC_GATEWAY
├── 01_PWR_ENTRY_PROTECTION
├── 02_COMPUTE_CORE
├── 03_SUPERVISOR_MCU
├── 04_ETHERNET_PORTS
├── 05_RS485_MODBUS_RTU
├── 06_IOLINK_MASTER_4PORT
├── 07_DISCRETE_IO_8DI_8DO
├── 08_CAN_CANOPEN_EXT
├── 09_SERVICE_DEBUG_RECOVERY
├── 10_SECURITY_NVM_RTC
└── 11_TEST_PRODUCTION_SUPPORT
```

### CAD 中推荐的层次化 Sheet 名称

| 页 | Sheet 名称 | 功能 |
|---|---|---|
| 00 | `TOP_QNC_GATEWAY` | 根页，含全部层次化端口（Hierarchical Ports） |
| 01 | `PWR_ENTRY_PROTECTION` | 24 V 输入、保护、时序、主变换器 |
| 02 | `COMPUTE_CORE` | SoM、启动、状态、主机 I/O |
| 03 | `SUPERVISOR_MCU` | 电源监测、复位、看门狗、安全模式 |
| 04 | `ETHERNET_PORTS` | 2× 北向以太网 |
| 05 | `RS485_MODBUS_RTU` | 2× 隔离 RS-485 |
| 06 | `IOLINK_MASTER_4PORT` | 4× IO-Link 主站 |
| 07 | `DISCRETE_IO_8DI_8DO` | 8 入 8 出 |
| 08 | `CAN_CANOPEN_EXT` | 可选 CAN/CANopen |
| 09 | `SERVICE_DEBUG_RECOVERY` | USB-C、UART、SWD/JTAG、恢复 |
| 10 | `SECURITY_NVM_RTC` | TPM、EEPROM、RTC、板卡身份 |
| 11 | `TEST_PRODUCTION_SUPPORT` | 测试点、回环、量产辅助 |

该分页直接遵循 PCB 计划中电源、计算、监控、现场总线、维护与安全域的功能划分。

---

## 2. 顶层框图

以下为原文档 ASCII 框图；框内德文/英文标签已译为中文，**网络名**仍为 EDA 兼容的英文标识。

```text
                 +----------------------------------+
24V_IN / PE ---> | 01 电源入口与保护                |
                 | TVS / 反接保护 / 热插拔 / EMI / Buck |
                 +--+-----------+---------+---------+
                    |           |         |
                    |           |         +--> 24V_FIELD_A/B/C
                    |           +------------> 5V_ISO_A/B
                    +------------------------> 5V_MAIN / 3V3_LOGIC
                                  |
                                  v
                    +------------------------------+
                    | 03 监控 MCU                    |
                    | 电源轨监测 / 复位 / 看门狗 WDOG |
                    +----+-------------+-----------+
                         |             |
                         |             +--> 各使能、安全模式、复位树
                         |
                         v
              +------------------------------+
              | 02 计算核心（SoM）           |
              | Linux 运行时 / 北向接口       |
              +--+---------+--------+--------+
                 |         |        | \
                 |         |        |  \
                 v         v        v   v
        +-----------+ +---------+ +---------+ +------------------+
        |04 以太网×2| |05 RS485 | |06 IO-Link| |07 DIO 8DI / 8DO  |
        +-----------+ +---------+ +---------+ +------------------+
                 \         |            |              /
                  \        |            |             /
                   \       v            v            /
                    \   +---------------------------+
                     -->| 08 CAN 扩展（可选）       |
                        +---------------------------+

              +------------------+    +------------------+
              |09 维护/调试/恢复 |    |10 安全/NVM/RTC   |
              +------------------+    +------------------+

                        +---------------------------+
                        |11 测试 / 量产支持          |
                        +---------------------------+
```

---

## 3. 全局网络命名约定

为使层次化原理图保持清晰，建议按下述方式规范化网络命名（**符号名保持英文**，便于工具与 BOM 对照）。

### 电源/地（Power / Ground）
- `24V_IN_RAW`
- `24V_IN_PROT`
- `24V_FIELD_A`、`24V_FIELD_B`、`24V_FIELD_C`
- `5V_MAIN`
- `5V_ISO_A`、`5V_ISO_B`
- `3V3_LOGIC`
- `1V8_AUX`
- `VBAT_RTC`
- `DGND`
- `FGND_A`、`FGND_B`
- `CHASSIS`

### 复位/监控/安全（Reset / Supervision / Safety）
- `PWR_GOOD`
- `SYS_RESET_N`
- `SOM_RESET_N`
- `MCU_RESET_N`
- `SAFE_MODE_REQ`
- `SAFE_MODE_ACK`
- `WDOG_KICK`
- `WDOG_FAIL_N`
- `BOOT_RECOVERY_N`
- `FORCE_RECOVERY_N`
- `24V_UV_WARN`
- `24V_OV_WARN`
- `PWR_FAULT_N`
- `THERM_WARN_N`

### 管理/维护（Management / Service）
- `I2C_SEC_SCL`、`I2C_SEC_SDA`
- `I2C_MGMT_SCL`、`I2C_MGMT_SDA`
- `SPI_TPM_SCLK`、`SPI_TPM_MOSI`、`SPI_TPM_MISO`、`SPI_TPM_CS_N`
- `UART_DBG_TX`、`UART_DBG_RX`
- `USB0_D_P`、`USB0_D_N`
- `USB0_CC1`、`USB0_CC2`
- `SWDIO`、`SWCLK`、`NRST_MCU`

### 以太网（Ethernet）
- `ETH1_MDI0_P/N` … `ETH1_MDI3_P/N`
- `ETH2_MDI0_P/N` … `ETH2_MDI3_P/N`
- `ETH1_LED_LINK_N`、`ETH1_LED_ACT_N`
- `ETH2_LED_LINK_N`、`ETH2_LED_ACT_N`
- `ETH1_SHIELD`、`ETH2_SHIELD`

### RS-485
- `RS485A_TXD`、`RS485A_RXD`、`RS485A_DE`、`RS485A_RE_N`、`RS485A_TERM_EN`
- `RS485B_TXD`、`RS485B_RXD`、`RS485B_DE`、`RS485B_RE_N`、`RS485B_TERM_EN`
- `RS485A_A`、`RS485A_B`
- `RS485B_A`、`RS485B_B`

### IO-Link
- `IOL_SPI_SCLK`、`IOL_SPI_MOSI`、`IOL_SPI_MISO`
- `IOL_CS0_N`、`IOL_CS1_N`
- `IOL_INT0_N`、`IOL_INT1_N`
- `IOL_RESET_N`
- `IOL0_CQ` … `IOL3_CQ`
- `IOL0_LPLUS` … `IOL3_LPLUS`
- `IOL0_OC_N` … `IOL3_OC_N`
- `IOL0_TSD_N` … `IOL3_TSD_N`

### DIO（离散 I/O）
- `DI_SENSE[7:0]`
- `DO_CMD[7:0]`
- `DO_FLT_N[7:0]`
- `DO_EN[7:0]`
- `DIO_BANK0_MODE`、`DIO_BANK1_MODE`
- 对外逻辑：`DI0_IN` … `DI7_IN`，`DO0_OUT` … `DO7_OUT`

### CAN
- `CAN1_TXD`、`CAN1_RXD`、`CAN1_STB_N`、`CAN1_TERM_EN`
- `CANH`、`CANL`
- `CAN1_FAULT_N`

### 测试/制造（Test / Manufacturing）
- `TP_24V_IN_RAW`、`TP_5V_MAIN`、`TP_3V3_LOGIC`、`TP_SYS_RESET_N`
- `TP_RS485A_A`、`TP_RS485A_B`、`TP_CANH`、`TP_CANL`
- `TP_ETH1_MDI0_P/N` 等
- `LB_RS485A_EN`、`LB_CAN1_EN`、`LB_IOL_SIM_EN`

---

## 4. 方向定义

表格中统一约定：

- `IN`：进入**本页**
- `OUT`：离开**本页**
- `BIDI`：双向
- `MON`：测量/状态，非驱动型功能信号
- `PWR`：供电能量
- `REF`：地/屏蔽/参考

---

## 5. 各页详细定义（01–11）

以下各节均包含：**框图**（逻辑流程）、**主要功能块**、**页间接口表**（接口/网络、方向、对页/目标、功能说明）、**注释**。

---

## 01 – [PWR_ENTRY_PROTECTION]（电源入口与保护）

### 框图

```text
J1_24V_IN / PE
   -> 熔断器 / 反接保护 / TVS
   -> 热插拔 / 浪涌电流 / EMI π 型滤波
   -> 24V_IN_PROT
      -> Buck 5V_MAIN
      -> Buck/LDO 3V3_LOGIC
      -> 隔离 DC/DC 5V_ISO_A
      -> 隔离 DC/DC 5V_ISO_B
      -> eFuse 24V_FIELD_A
      -> eFuse 24V_FIELD_B
      -> eFuse 24V_FIELD_C
   -> 电流/电压检测 -> Supervisor（监控）
```

### 主要功能块
- J1：24 VDC 输入 + PE（保护地）
- F1：输入熔断器
- D1：反接保护 / 理想二极管
- TVS1：浪涌钳位
- U1：热插拔 / 浪涌电流控制器
- EMI 滤波：共模扼流圈 + π 型网络
- U2：主 Buck，`24V_IN_PROT` → `5V_MAIN`
- U3：次级稳压器，`5V_MAIN` → `3V3_LOGIC`
- U4/U5：隔离式变换器 → `5V_ISO_A/B`
- U6/U7/U8：eFuse/限流 → `24V_FIELD_A/B/C`

### 页间接口

| 接口/网络 | 方向 | 对页/目标 | 功能 |
|---|---:|---|---|
| `24V_IN_RAW` | IN | J1 | 未经处理的 24 V 输入 |
| `PE_CHASSIS` | IN | J1 / 外壳 | 保护导体 / 外壳 |
| `EN_5V_MAIN` | IN | 03 | 监控侧释放 5 V 主轨 |
| `EN_3V3_LOGIC` | IN | 03 | 可选的独立 3V3 使能 |
| `EN_ISO_A`、`EN_ISO_B` | IN | 03 | 隔离总线电源释放 |
| `EN_FIELD_A/B/C` | IN | 03 | 现场电压支路释放 |
| `24V_IN_PROT` | OUT | 03/06/07 | 受保护的 24 V 母线 |
| `5V_MAIN` | OUT | 02/03/09/10/11 | 中央 5 V 逻辑供电 |
| `3V3_LOGIC` | OUT | 02/03/09/10/11 | 3.3 V 逻辑 |
| `5V_ISO_A`、`5V_ISO_B` | OUT | 05/08 | 隔离总线供电 |
| `24V_FIELD_A/B/C` | OUT | 06/07 | 受保护的现场供电 |
| `PWR_GOOD` | OUT | 02/03 | 系统电源良好（Power Good） |
| `24V_UV_WARN` | OUT | 03 | 欠压检测 |
| `24V_OV_WARN` | OUT | 03 | 过压检测 |
| `PWR_FAULT_N` | OUT | 03 | 锁存的电源故障 |
| `I_MON_24V` | OUT | 03 | 总电流模拟/ADC 量测 |
| `TEMP_PWR_WARN` | OUT | 03 | 电源树温度警告 |
| `DGND` | REF | 所有逻辑页 | 逻辑地 |
| `CHASSIS` | REF | 04/05/08/09 | 屏蔽/外壳参考 |

**注释**：页 01 是初级电流路径的唯一来源；所有现场与隔离域由此**星形**供电，符合计划中的故障包容与 PDN 策略。

---

## 02 – [COMPUTE_CORE]（计算核心）

### 框图

```text
5V_MAIN / 3V3_LOGIC
   -> 工业 SoM
   -> 启动绑带 / 复位 / 状态 LED
   -> 主机接口
      -> ETH x2
      -> RS485 x2
      -> IO-Link SPI
      -> DIO 控制/状态
      -> CAN
      -> USB/UART 维护
      -> TPM / EEPROM / RTC
```

### 主要功能块
- X1：工业 SoM
- Boot 绑带网络
- 复位逻辑
- 主机状态 LED
- 必要时电平转换
- 本地大容量/高频去耦岛

### 页间接口

| 接口/网络 | 方向 | 对页/目标 | 功能 |
|---|---:|---|---|
| `5V_MAIN` | IN | 01 | SoM 主供电 |
| `3V3_LOGIC` | IN | 01 | I/O 与辅助电压 |
| `PWR_GOOD` | IN | 01 | 启动时序许可 |
| `SYS_RESET_N` | IN | 03 | 监控发出的全局复位 |
| `SOM_RESET_N` | IN | 03 | SoM 独立复位 |
| `SAFE_MODE_REQ` | IN | 03 | 启动/运行进入安全模式 |
| `FORCE_RECOVERY_N` | IN | 09/03 | 强制恢复启动（见第三部分矩阵：建议经 03 仲裁） |
| `WDOG_KICK` | OUT | 03 | 看门狗喂狗/存活信号 |
| `SAFE_MODE_ACK` | OUT | 03 | SoM 已接受安全模式 |
| `SOM_HEARTBEAT` | OUT | 03 | 周期性存活脉冲 |
| `ETH1_MDI0..3_P/N` | BIDI | 04 | 以太网端口 1 数据对 |
| `ETH2_MDI0..3_P/N` | BIDI | 04 | 以太网端口 2 数据对 |
| `ETH1_LED_LINK_N`、`ETH1_LED_ACT_N` | OUT | 04 | 端口 1 LED |
| `ETH2_LED_LINK_N`、`ETH2_LED_ACT_N` | OUT | 04 | 端口 2 LED |
| `RS485A_TXD` | OUT | 05 | 通道 A UART TX |
| `RS485A_RXD` | IN | 05 | 通道 A UART RX |
| `RS485A_DE`、`RS485A_RE_N` | OUT | 05 | 通道 A 方向控制 |
| `RS485A_TERM_EN` | OUT | 05 | 可切换端接 |
| `RS485B_TXD` | OUT | 05 | 通道 B UART TX |
| `RS485B_RXD` | IN | 05 | 通道 B UART RX |
| `RS485B_DE`、`RS485B_RE_N` | OUT | 05 | 通道 B 方向控制 |
| `RS485B_TERM_EN` | OUT | 05 | 可切换端接 |
| `IOL_SPI_SCLK`、`IOL_SPI_MOSI` | OUT | 06 | IO-Link 主机 SPI |
| `IOL_SPI_MISO` | IN | 06 | IO-Link 主机 SPI 回读 |
| `IOL_CS0_N`、`IOL_CS1_N` | OUT | 06 | IO-Link 主站 IC 片选 |
| `IOL_INT0_N`、`IOL_INT1_N` | IN | 06 | IO-Link 中断 |
| `IOL_RESET_N` | OUT | 06 | IO-Link 控制器复位 |
| `DI_SENSE[7:0]` | IN | 07 | 至主机的数字输入 |
| `DO_CMD[7:0]` | OUT | 07 | 输出开关命令 |
| `DO_EN[7:0]` | OUT | 07 | 独立通道使能 |
| `DO_FLT_N[7:0]` | IN | 07 | 故障/短路状态 |
| `CAN1_TXD` | OUT | 08 | CAN TX |
| `CAN1_RXD` | IN | 08 | CAN RX |
| `CAN1_STB_N` | OUT | 08 | CAN 待机 |
| `CAN1_TERM_EN` | OUT | 08 | 可切换 CAN 端接 |
| `USB0_D_P/N` | BIDI | 09 | USB-C 设备/控制台 |
| `UART_DBG_TX` | OUT | 09 | 串口调试发送 |
| `UART_DBG_RX` | IN | 09 | 串口调试接收 |
| `BOOT_MODE[2:0]` | IN | 09 | DIP/跳线 Boot 绑带 |
| `SPI_TPM_*` | BIDI | 10 | TPM 2.0 |
| `I2C_SEC_SCL/SDA` | BIDI | 10 | EEPROM / RTC / 板卡 ID |
| `RTC_INT_N` | IN | 10 | 报警/时间基准 |
| `BOARD_ID[3:0]` | IN | 10 | 固定变体识别 |

**注释**：页 02 刻意集中于主机/数据路径；硬保护、时序与恢复留在监控与电源分支，符合 SoM + 监控架构。

---

## 03 – [SUPERVISOR_MCU]（监控 MCU）

### 框图

```text
电源轨检测 + 故障标志 + 电流检测
      -> MCU ADC / GPIO / 定时器看门狗
      -> 复位树 / 各使能 / 安全模式
      -> 状态 LED / 事件记录 / 维护挂钩
```

### 主要功能块
- U30：监控 MCU
- ADC 测量网络（电源轨与温度）
- 复位/电源时序
- 硬件看门狗窗口
- 事件/故障锁存
- 板卡状态 LED

### 页间接口

| 接口/网络 | 方向 | 对页/目标 | 功能 |
|---|---:|---|---|
| `5V_MAIN` | IN | 01 | MCU 供电 |
| `3V3_LOGIC` | IN | 01 | MCU I/O |
| `24V_UV_WARN`、`24V_OV_WARN` | IN | 01 | 电压警告 |
| `PWR_FAULT_N` | IN | 01 | 电源故障 |
| `I_MON_24V` | IN | 01 | 总电流监测 |
| `TEMP_PWR_WARN` | IN | 01 | 电源温度警告 |
| `SOM_HEARTBEAT` | IN | 02 | SoM 活动 |
| `WDOG_KICK` | IN | 02 | 看门狗服务 |
| `SAFE_MODE_ACK` | IN | 02 | 主机处于安全模式 |
| `DO_FLT_N[7:0]` | IN | 07 | DO 故障状态 |
| `IOL0_OC_N`…`IOL3_OC_N` | IN | 06 | IO-Link 过流 |
| `IOL0_TSD_N`…`IOL3_TSD_N` | IN | 06 | IO-Link 热关断 |
| `CAN1_FAULT_N` | IN | 08 | CAN 故障 |
| `RS485_FAULT_SUM_N` | IN | 05 | RS-485 汇总故障 |
| `EN_5V_MAIN` | OUT | 01 | 释放主供电 |
| `EN_3V3_LOGIC` | OUT | 01 | 释放 3.3 V |
| `EN_ISO_A`、`EN_ISO_B` | OUT | 01 | 释放隔离变换器 |
| `EN_FIELD_A/B/C` | OUT | 01 | 释放现场支路 |
| `SYS_RESET_N` | OUT | 02/09/11 | 全局复位 |
| `SOM_RESET_N` | OUT | 02 | 定向 SoM 复位 |
| `SAFE_MODE_REQ` | OUT | 02 | 请求安全模式 |
| `FORCE_RECOVERY_N` | OUT | 02/09 | 触发恢复启动 |
| `STATUS_LED_G`、`STATUS_LED_Y`、`STATUS_LED_R` | OUT | 09/前面板 | 系统状态 |
| `SWDIO`、`SWCLK`、`NRST_MCU` | BIDI | 09 | MCU 调试 |
| `EVENT_LOG_INT` | OUT | 10 | 可选安全事件计数 |

**注释**：页 03 是“系统仍存活”与“主机软件挂死”之间的分界，落实 PCB 计划中的故障隔离、欠压处理、恢复与安全模式。

---

## 04 – [ETHERNET_PORTS]（以太网端口）

### 框图

```text
SoM 以太网差分对
   -> ESD / 共模扼流圈 / 可选 PHY 侧调理
   -> 磁性元件模块
   -> RJ45 或 M12 X-coded
   -> 屏蔽接 CHASSIS
```

### 冻结假设
在**端口侧以太网数据路径**冻结本页接口。若所选 SoM 输出 MAC/RGMII 而非 MDI 对，则外部端口定义不变；在页 04 增加 PHY 子模块。

### 页间接口

| 接口/网络 | 方向 | 对页/目标 | 功能 |
|---|---:|---|---|
| `ETH1_MDI0..3_P/N` | BIDI | 02 | 端口 1 数据对 |
| `ETH2_MDI0..3_P/N` | BIDI | 02 | 端口 2 数据对 |
| `ETH1_LED_LINK_N`、`ETH1_LED_ACT_N` | IN | 02 | LED 驱动 |
| `ETH2_LED_LINK_N`、`ETH2_LED_ACT_N` | IN | 02 | LED 驱动 |
| `3V3_LOGIC` | IN | 01 | LED/端口辅助电 |
| `DGND` | REF | 01 | 逻辑参考 |
| `CHASSIS` | REF | 01 | 屏蔽/外壳连接 |
| `ETH1_SHIELD` | OUT | J2 | 端口 1 屏蔽 |
| `ETH2_SHIELD` | OUT | J3 | 端口 2 屏蔽 |
| `ETH1_TXRX_PORT` | BIDI | J2 | 外部以太网端口 1 |
| `ETH2_TXRX_PORT` | BIDI | J3 | 外部以太网端口 2 |

### 外部连接器
- `J2_ETH1`
- `J3_ETH2`

**注释**：实现经磁性元件的电气隔离、ESD 与规范屏蔽至外壳；对应北向以太网域。

---

## 05 – [RS485_MODBUS_RTU]

### 框图

```text
主机 UART / DE-RE
   -> 隔离 RS-485 收发器 A
   -> TVS / 偏置 / 端接 / 连接器
   -> MBUS_A

主机 UART / DE-RE
   -> 隔离 RS-485 收发器 B
   -> TVS / 偏置 / 端接 / 连接器
   -> MBUS_B
```

### 主要功能块
- 通道 A 全隔离
- 通道 B 全隔离
- 可选切换端接
- 偏置网络
- 浪涌/ESD 保护
- 诊断/故障输出

### 页间接口

| 接口/网络 | 方向 | 对页/目标 | 功能 |
|---|---:|---|---|
| `3V3_LOGIC` | IN | 01 | RS-485 逻辑侧 |
| `5V_ISO_A` | IN | 01 | 通道 A 隔离现场供电 |
| `5V_ISO_B` | IN | 01 | 通道 B 隔离现场供电 |
| `DGND` | REF | 01 | 主机参考 |
| `FGND_A`、`FGND_B` | REF | 本地/现场 | 现场参考 |
| `RS485A_TXD` | IN | 02 | 主机 TX A |
| `RS485A_RXD` | OUT | 02 | 主机 RX A |
| `RS485A_DE`、`RS485A_RE_N` | IN | 02 | 方向 A |
| `RS485A_TERM_EN` | IN | 02 | 端接 A |
| `RS485B_TXD` | IN | 02 | 主机 TX B |
| `RS485B_RXD` | OUT | 02 | 主机 RX B |
| `RS485B_DE`、`RS485B_RE_N` | IN | 02 | 方向 B |
| `RS485B_TERM_EN` | IN | 02 | 端接 B |
| `RS485_FAULT_SUM_N` | OUT | 03 | 双通道汇总故障 |
| `RS485A_A`、`RS485A_B` | BIDI | J4 | 外部总线 A |
| `RS485B_A`、`RS485B_B` | BIDI | J5 | 外部总线 B |
| `CHASSIS` | REF | 01 | 屏蔽/外壳参考 |

### 外部连接器
- `J4_MBUS_A`
- `J5_MBUS_B`

**注释**：实现隔离、可诊断的 Modbus RTU 域；每通道电气独立，现场故障不回耦至计算域。

---

## 06 – [IOLINK_MASTER_4PORT]

### 框图

```text
主机 SPI
   -> IO-Link 主站 IC #0 -> 端口 0 / 端口 1
   -> IO-Link 主站 IC #1 -> 端口 2 / 端口 3

24V_FIELD_A/B
   -> 每端口受保护 L+
   -> C/Q 驱动 / 限流 / 热反馈
   -> M12 端口连接器
```

### 内部分区
- `PORT0_BLOCK` … `PORT3_BLOCK`：四端口块须**拓扑完全一致**。

### 页间接口

| 接口/网络 | 方向 | 对页/目标 | 功能 |
|---|---:|---|---|
| `24V_FIELD_A` | IN | 01 | 端口组 0/1 现场供电 |
| `24V_FIELD_B` | IN | 01 | 端口组 2/3 现场供电 |
| `5V_MAIN` | IN | 01 | 辅助供电 |
| `3V3_LOGIC` | IN | 01 | 逻辑供电 |
| `DGND` | REF | 01 | 逻辑参考 |
| `IOL_SPI_SCLK`、`IOL_SPI_MOSI` | IN | 02 | 主机 SPI |
| `IOL_SPI_MISO` | OUT | 02 | SPI 回读 |
| `IOL_CS0_N`、`IOL_CS1_N` | IN | 02 | 片选 |
| `IOL_INT0_N`、`IOL_INT1_N` | OUT | 02 | 中断 |
| `IOL_RESET_N` | IN | 02 | 主站 IC 复位 |
| `IOL0_OC_N`…`IOL3_OC_N` | OUT | 03 | 各端口过流 |
| `IOL0_TSD_N`…`IOL3_TSD_N` | OUT | 03 | 各端口热故障 |
| `IOL0_CQ` | BIDI | J6 | IO-Link 端口 0 C/Q |
| `IOL0_LPLUS` | OUT | J6 | 端口 0 L+ |
| `IOL0_LMINUS` | REF | J6 | 端口 0 L− |
| `IOL1_CQ` | BIDI | J7 | IO-Link 端口 1 C/Q |
| `IOL1_LPLUS` | OUT | J7 | 端口 1 L+ |
| `IOL1_LMINUS` | REF | J7 | 端口 1 L− |
| `IOL2_CQ` | BIDI | J8 | IO-Link 端口 2 C/Q |
| `IOL2_LPLUS` | OUT | J8 | 端口 2 L+ |
| `IOL2_LMINUS` | REF | J8 | 端口 2 L− |
| `IOL3_CQ` | BIDI | J9 | IO-Link 端口 3 C/Q |
| `IOL3_LPLUS` | OUT | J9 | 端口 3 L+ |
| `IOL3_LMINUS` | REF | J9 | 端口 3 L− |
| `CHASSIS` / `FE_IOL` | REF | J6…J9 | 功能地/屏蔽（若使用） |

### 外部连接器
- `J6_IOL0` … `J9_IOL3`

**注释**：4 端口主站域，独立 L+、短路/热保护与对称通道。

---

## 07 – [DISCRETE_IO_8DI_8DO]

### 框图

```text
24V_FIELD_C
   -> DI 前端 x8 -> DI_SENSE[7:0] -> 主机
   -> DO 高边/可配置组 x8 <- DO_CMD[7:0]（来自主机）
                         -> DO_FLT_N[7:0] -> 主机/监控
```

### 结构
- `DI_BANK0` = DI0…DI3；`DI_BANK1` = DI4…DI7
- `DO_BANK0` = DO0…DO3；`DO_BANK1` = DO4…DO7  
若量产改为灵活 DIO IC，**本页对外接口保持不变**，仅通道实现改变。

### 页间接口

| 接口/网络 | 方向 | 对页/目标 | 功能 |
|---|---:|---|---|
| `24V_FIELD_C` | IN | 01 | DIO 现场供电 |
| `3V3_LOGIC` | IN | 01 | 逻辑供电 |
| `DGND` | REF | 01 | 逻辑参考 |
| `DI_SENSE[7:0]` | OUT | 02 | 至主机的数字输入 |
| `DO_CMD[7:0]` | IN | 02 | 开关命令 |
| `DO_EN[7:0]` | IN | 02 | 独立使能 |
| `DO_FLT_N[7:0]` | OUT | 02/03 | 短路/过流/过温等 |
| `DIO_BANK0_MODE` | IN | 03/09 | 组 0 模式 |
| `DIO_BANK1_MODE` | IN | 03/09 | 组 1 模式 |
| `DI0_IN`…`DI7_IN` | IN | J10 | 外部 24 V 输入 |
| `DO0_OUT`…`DO7_OUT` | OUT | J11 | 外部 24 V 输出 |
| `DI_COM_A/B` | REF | J10 | 输入公共端 |
| `DO_COM_A/B` | REF | J11 | 输出回线/公共端 |
| `FE_DIO` | REF | J10/J11 | 可选功能地/屏蔽 |

### 外部连接器
- `J10_DI_BANK`
- `J11_DO_BANK`

**注释**：覆盖 8DI/8DO；sourcing/sinking/灵活 DIO 的硬件决策在架构上仍可演进。

---

## 08 – [CAN_CANOPEN_EXT]

### 框图

```text
主机 CAN TX/RX
   -> CAN 收发器
   -> 保护 / 可选隔离 / 端接
   -> CAN 连接器
```

### 页间接口

| 接口/网络 | 方向 | 对页/目标 | 功能 |
|---|---:|---|---|
| `3V3_LOGIC` | IN | 01 | 逻辑供电 |
| `5V_ISO_B` | IN | 01 | 可选隔离现场侧 |
| `DGND` | REF | 01 | 主机参考 |
| `FGND_B` | REF | 现场侧 | 总线参考 |
| `CAN1_TXD` | IN | 02 | 主机 TX |
| `CAN1_RXD` | OUT | 02 | 主机 RX |
| `CAN1_STB_N` | IN | 02 | 待机释放 |
| `CAN1_TERM_EN` | IN | 02 | 120 Ω 端接 |
| `CAN1_FAULT_N` | OUT | 03 | 故障状态 |
| `CANH`、`CANL` | BIDI | J12 | CAN 差分对 |
| `CHASSIS` | REF | J12 / 01 | 屏蔽/外壳 |

### 外部连接器
- `J12_CAN_EXT`

**注释**：按 PCB 计划可选贴片；与主机接口仍完整定义以保持可扩展布局。

---

## 09 – [SERVICE_DEBUG_RECOVERY]

### 框图

```text
USB-C 维护 + ESD + CC
UART 调试接头
SWD/JTAG 接头
恢复按键 + Boot DIP
状态 LED
```

### 页间接口

| 接口/网络 | 方向 | 对页/目标 | 功能 |
|---|---:|---|---|
| `5V_MAIN` | IN | 01 | 维护供电 |
| `3V3_LOGIC` | IN | 01 | 调试逻辑 |
| `USB0_D_P/N` | BIDI | 02 / J13 | USB-C 数据 |
| `USB0_CC1`、`USB0_CC2` | BIDI | J13 | USB-C CC |
| `UART_DBG_TX` | IN | 02 | SoM 至控制台的 TX |
| `UART_DBG_RX` | OUT | 02 | 至 SoM 的 RX |
| `SWDIO`、`SWCLK`、`NRST_MCU` | BIDI | 03 / J15 | MCU 调试 |
| `BOOT_MODE[2:0]` | OUT | 02 | DIP/跳线 Boot 绑带 |
| `FORCE_RECOVERY_N` | OUT | 02/03 | 恢复按键/跳线（矩阵建议采用 `FORCE_RECOVERY_REQ_N` 入 03） |
| `STATUS_LED_G/Y/R` | IN | 03 | 前面板状态 |
| `DGND` | REF | 01 | 逻辑参考 |
| `CHASSIS` | REF | 01 / J13 | USB 屏蔽至外壳 |

### 外部连接器/控件
- `J13_USB_C_SERVICE`
- `J14_UART_DEBUG`
- `J15_SWD_HEADER`
- `S1_RECOVERY`
- `SW1_BOOT_MODE_DIP`

**注释**：对应更新、诊断与 Boot 恢复路径。

---

## 10 – [SECURITY_NVM_RTC]

### 框图

```text
SoM SPI -> TPM 2.0
SoM I2C -> EEPROM / 板卡 ID / RTC
可选防拆 / 安全事件计数器
```

### 主要功能块
- TPM 2.0
- 板卡 EEPROM / 序列号 / 变体数据
- 带备份的 RTC
- 可选安全元件或单调计数器

### 页间接口

| 接口/网络 | 方向 | 对页/目标 | 功能 |
|---|---:|---|---|
| `3V3_LOGIC` | IN | 01 | 逻辑供电 |
| `VBAT_RTC` | IN | 本地备份源 | RTC 备份 |
| `DGND` | REF | 01 | 参考地 |
| `SPI_TPM_SCLK`、`SPI_TPM_MOSI` | IN | 02 | TPM SPI |
| `SPI_TPM_MISO` | OUT | 02 | TPM SPI 回读 |
| `SPI_TPM_CS_N` | IN | 02 | TPM 片选 |
| `TPM_IRQ_N` | OUT | 02 | 可选 TPM 中断 |
| `I2C_SEC_SCL/SDA` | BIDI | 02 | EEPROM / RTC / ID |
| `RTC_INT_N` | OUT | 02 | 报警/唤醒 |
| `BOARD_ID[3:0]` | OUT | 02 | 变体 ID |
| `EVENT_LOG_INT` | IN | 03 | 监控事件计数 |
| `TAMPER_IN` | IN | 可选外部/开关 | 防拆检测 |

**注释**：支撑安全身份、签名、回滚/恢复与持久板卡标识。

---

## 11 – [TEST_PRODUCTION_SUPPORT]

### 框图

```text
全局电源轨测试点
总线回环
电流分流/测量焊盘
针床（Bed-of-Nails）接入
出厂绑带
```

### 页间接口

| 接口/网络 | 方向 | 对页/目标 | 功能 |
|---|---:|---|---|
| `TP_24V_IN_RAW` | MON | 01 | 输入电压测试点 |
| `TP_24V_IN_PROT` | MON | 01 | 受保护 24 V 测试点 |
| `TP_5V_MAIN` | MON | 01 | 5V_MAIN 测试点 |
| `TP_3V3_LOGIC` | MON | 01 | 3V3 测试点 |
| `TP_PW_GOOD` | MON | 01/03 | 时序检查 |
| `TP_SYS_RESET_N` | MON | 03 | 复位行为 |
| `TP_WDOG_KICK` | MON | 02/03 | 看门狗 |
| `TP_RS485A_A/B`、`TP_RS485B_A/B` | MON | 05 | 总线检查 |
| `TP_CANH`、`TP_CANL` | MON | 08 | CAN 检查 |
| `TP_IOL0_CQ`…`TP_IOL3_CQ` | MON | 06 | IO-Link 针床 |
| `TP_DI[7:0]`、`TP_DO[7:0]` | MON | 07 | DIO 验证 |
| `LB_RS485A_EN`、`LB_RS485B_EN` | IN | 跳线/绑带 | RS-485 回环测试 |
| `LB_CAN1_EN` | IN | 跳线/绑带 | CAN 测试路径 |
| `LB_IOL_SIM_EN` | IN | 跳线/绑带 | IO-Link 仿真 |
| `FACTORY_ID_PROG` | BIDI | 10 | 序列号/EEPROM 编程 |

**注释**：正式承载全部测试点、回环、序列编程与弹簧针盘接口；与 EVT/DVT/PVT 的 DFT 策略一致。

---

## 6. 页间骨干：顶层页中的层次化端口建议

### 电源骨干
`24V_IN_PROT`、`24V_FIELD_A/B/C`、`5V_MAIN`、`5V_ISO_A/B`、`3V3_LOGIC`、`DGND`、`CHASSIS`

### 安全/监控骨干
`PWR_GOOD`、`SYS_RESET_N`、`SOM_RESET_N`、`SAFE_MODE_REQ/ACK`、`WDOG_KICK`、`PWR_FAULT_N`、`24V_UV_WARN`、`24V_OV_WARN`

### 主机外设骨干
以太网端口对、`RS485A/B_*`、`IOL_*`、`DI_SENSE[7:0]`、`DO_CMD/EN/FLT_N`、`CAN1_*`、`USB0_*`、`UART_DBG_*`、`SPI_TPM_*`、`I2C_SEC_*`

---

## 7. 产品级外部接口定义

- **J1_POWER_IN**：24V_IN+、0V_IN、`PE_CHASSIS`
- **J2_ETH1**、**J3_ETH2**
- **J4_MBUS_A**、**J5_MBUS_B**、**J12_CAN_EXT**（A/B、CANH/L，可选 FGND、屏蔽）
- **J6…J9**：L+、L−、C/Q、可选 P2、可选 FE/屏蔽
- **J10_DI_BANK**、**J11_DO_BANK**
- **J13_USB_C_SERVICE**、**J14_UART_DEBUG**、**J15_SWD_HEADER**

---

## 8. 明确列为“仍开放但已逻辑冻结”的项

1. 以太网连接器族：RJ45 vs M12 X-coded  
2. 现场连接器：螺钉端子 vs M8/M12  
3. DIO 末端拓扑：固定 sourcing、按组源/灌、或灵活 DIO IC  
4. CAN 贴片：可选 DNI 或量产功能  
5. 隔离边界细节：最终试验电压、爬电/间隙  
6. 各现场组精确电流预算，尤其 IO-Link L+ 与 DO 负载角  
7. PE/外壳耦合网络：RC/HF 与外壳概念  
8. SoM 以太网边界：直连 MDI vs 载板 MAC/RGMII + PHY  

（与 PCB 计划所列或隐含开放点一致。）

---

# 第三部分：页间连接矩阵（原文档：Sheet-to-Sheet-Verbindungsmatrix.md）

已将原理图层次转化为**规范化页间连接矩阵**。保留前述 01–11 各页，并将两条在 CAD 中可能多驱动的控制路径规范为**单源信号**：

- `FORCE_RECOVERY_REQ_N` 由 **Sheet 09** 至 **Sheet 03** 的监控；由 **Sheet 03** 产生实际 `FORCE_RECOVERY_N` 至 **Sheet 02** 计算核心。  
- 组配置逻辑源自维护/配置路径，但经 **Sheet 03** 输出 `DIO_BANK0_MODE`、`DIO_BANK1_MODE` 至 **Sheet 07**，避免原理图中的多驱动网络。

## A. 供电、参考与监控骨干

全局供电、参考地及自电源页至监控页的监测信号。

| 编号 | 网络名 | 源（页.块） | 目标（页.块） | 方向 | 信号类型 | 用途 |
|:---:|---|---|---|:---:|---|---|
| A001 | `24V_IN_PROT` | 01.PowerTree | 03.Supervisor_ADC | 01 → 03 | PWR/MON | 受保护 24 V 母线，供监控侧监测 |
| A002 | `24V_FIELD_A` | 01.FieldPower_A | 06.IOLink_PortGroup_0_1 | 01 → 06 | PWR | IO-Link 端口 0/1 的现场供电 |
| A003 | `24V_FIELD_B` | 01.FieldPower_B | 06.IOLink_PortGroup_2_3 | 01 → 06 | PWR | IO-Link 端口 2/3 的现场供电 |
| A004 | `24V_FIELD_C` | 01.FieldPower_C | 07.DIO_FieldPower | 01 → 07 | PWR | 数字量输入/输出的现场供电 |
| A005 | `5V_MAIN` | 01.MainBuck | 02.SoM_Power | 01 → 02 | PWR | 计算核心主供电 |
| A006 | `5V_MAIN` | 01.MainBuck | 03.Supervisor_Power | 01 → 03 | PWR | 监控 MCU 主供电 |
| A007 | `5V_MAIN` | 01.MainBuck | 06.IOLink_LogicAux | 01 → 06 | PWR | IO-Link 辅助供电 |
| A008 | `5V_MAIN` | 01.MainBuck | 09.ServicePower | 01 → 09 | PWR | 维护/调试供电 |
| A009 | `3V3_LOGIC` | 01.LogicReg | 02.SoM_IO | 01 → 02 | PWR | 计算侧逻辑供电 |
| A010 | `3V3_LOGIC` | 01.LogicReg | 03.Supervisor_IO | 01 → 03 | PWR | 监控侧逻辑供电 |
| A011 | `3V3_LOGIC` | 01.LogicReg | 04.Eth_LED_ESD | 01 → 04 | PWR | LED/端口辅助电压 |
| A012 | `3V3_LOGIC` | 01.LogicReg | 05.RS485_LogicSide | 01 → 05 | PWR | RS-485 逻辑侧供电 |
| A013 | `3V3_LOGIC` | 01.LogicReg | 06.IOLink_Logic | 01 → 06 | PWR | IO-Link 逻辑供电 |
| A014 | `3V3_LOGIC` | 01.LogicReg | 07.DIO_Logic | 01 → 07 | PWR | DIO 逻辑供电 |
| A015 | `3V3_LOGIC` | 01.LogicReg | 08.CAN_Logic | 01 → 08 | PWR | CAN 逻辑供电 |
| A016 | `3V3_LOGIC` | 01.LogicReg | 09.DebugLogic | 01 → 09 | PWR | 调试/Boot DIP 供电 |
| A017 | `3V3_LOGIC` | 01.LogicReg | 10.SecurityCore | 01 → 10 | PWR | TPM/EEPROM/RTC 供电 |
| A018 | `5V_ISO_A` | 01.IsoDCDC_A | 05.RS485_FieldSide_A | 01 → 05 | PWR | 通道 A 隔离供电 |
| A019 | `5V_ISO_B` | 01.IsoDCDC_B | 05.RS485_FieldSide_B | 01 → 05 | PWR | 通道 B 隔离供电 |
| A020 | `5V_ISO_B` | 01.IsoDCDC_B | 08.CAN_FieldSide | 01 → 08 | PWR | CAN 可选隔离供电 |
| A021 | `PWR_GOOD` | 01.PowerSeq | 02.SoM_ResetBoot | 01 → 02 | MON | 允许计算侧干净启动 |
| A022 | `PWR_GOOD` | 01.PowerSeq | 03.Supervisor_GPIO | 01 → 03 | MON | 监控获知电源状态 |
| A023 | `24V_UV_WARN` | 01.PowerMonitor | 03.Supervisor_ADC | 01 → 03 | MON | 欠压警告 |
| A024 | `24V_OV_WARN` | 01.PowerMonitor | 03.Supervisor_ADC | 01 → 03 | MON | 过压警告 |
| A025 | `PWR_FAULT_N` | 01.PowerMonitor | 03.Supervisor_GPIO | 01 → 03 | MON | 锁存电源故障 |
| A026 | `I_MON_24V` | 01.CurrentSense | 03.Supervisor_ADC | 01 → 03 | MON | 总电流测量 |
| A027 | `TEMP_PWR_WARN` | 01.ThermalSense | 03.Supervisor_GPIO | 01 → 03 | MON | 电源树温度警告 |
| A028 | `DGND` | 01.LogicGround | 02.LogicGround | REF | REF | 逻辑参考地 |
| A029 | `DGND` | 01.LogicGround | 03.LogicGround | REF | REF | 逻辑参考地 |
| A030 | `DGND` | 01.LogicGround | 04.LogicGround | REF | REF | 逻辑参考地 |
| A031 | `DGND` | 01.LogicGround | 05.LogicGround | REF | REF | 逻辑参考地 |
| A032 | `DGND` | 01.LogicGround | 06.LogicGround | REF | REF | 逻辑参考地 |
| A033 | `DGND` | 01.LogicGround | 07.LogicGround | REF | REF | 逻辑参考地 |
| A034 | `DGND` | 01.LogicGround | 08.LogicGround | REF | REF | 逻辑参考地 |
| A035 | `DGND` | 01.LogicGround | 09.LogicGround | REF | REF | 逻辑参考地 |
| A036 | `DGND` | 01.LogicGround | 10.LogicGround | REF | REF | 逻辑参考地 |
| A037 | `CHASSIS` | 01.ChassisBond | 04.PortShieldBond | REF | REF | 以太网屏蔽/外壳 |
| A038 | `CHASSIS` | 01.ChassisBond | 05.PortShieldBond | REF | REF | RS-485 屏蔽/外壳 |
| A039 | `CHASSIS` | 01.ChassisBond | 08.PortShieldBond | REF | REF | CAN 屏蔽/外壳 |
| A040 | `CHASSIS` | 01.ChassisBond | 09.USBShieldBond | REF | REF | USB/维护口屏蔽 |

**说明**：该矩阵汇总全局供电与参考网络及自电源页至监控页的全部监测信号；对应星形 PDN 与逻辑/现场/外壳域分离。

## B. 各页之间的控制、数据与管理连接

各功能页之间的控制、数据与管理连接。

| 编号 | 网络名 | 源（页.块） | 目标（页.块） | 方向 | 信号类型 | 用途 |
|:---:|---|---|---|:---:|---|---|
| B001 | `EN_5V_MAIN` | 03.PowerSequencer | 01.MainBuck_Enable | 03 → 01 | CTRL | 主供电使能 |
| B002 | `EN_3V3_LOGIC` | 03.PowerSequencer | 01.LogicReg_Enable | 03 → 01 | CTRL | 3.3 V 使能 |
| B003 | `EN_ISO_A` | 03.PowerSequencer | 01.IsoDCDC_A_Enable | 03 → 01 | CTRL | 隔离供电 A 使能 |
| B004 | `EN_ISO_B` | 03.PowerSequencer | 01.IsoDCDC_B_Enable | 03 → 01 | CTRL | 隔离供电 B 使能 |
| B005 | `EN_FIELD_A` | 03.PowerSequencer | 01.FieldPower_A_Enable | 03 → 01 | CTRL | IO-Link 组 0/1 使能 |
| B006 | `EN_FIELD_B` | 03.PowerSequencer | 01.FieldPower_B_Enable | 03 → 01 | CTRL | IO-Link 组 2/3 使能 |
| B007 | `EN_FIELD_C` | 03.PowerSequencer | 01.FieldPower_C_Enable | 03 → 01 | CTRL | DIO 现场供电使能 |
| B008 | `SYS_RESET_N` | 03.ResetTree | 02.SoM_ResetBoot | 03 → 02 | CTRL | 全局系统复位 |
| B009 | `SOM_RESET_N` | 03.ResetTree | 02.SoM_ResetBoot | 03 → 02 | CTRL | 定向计算核心复位 |
| B010 | `SAFE_MODE_REQ` | 03.SafeModeCtrl | 02.SafeModeInput | 03 → 02 | CTRL | 请求安全模式 |
| B011 | `FORCE_RECOVERY_N` | 03.RecoveryCtrl | 02.RecoveryBoot | 03 → 02 | CTRL | 强制恢复启动 |
| B012 | `SOM_HEARTBEAT` | 02.RuntimeMonitor | 03.WatchdogInput | 02 → 03 | MON | SoM 存活/心跳 |
| B013 | `WDOG_KICK` | 02.RuntimeMonitor | 03.WatchdogInput | 02 → 03 | MON | 看门狗喂狗服务 |
| B014 | `SAFE_MODE_ACK` | 02.SafeModeOutput | 03.SafeModeCtrl | 02 → 03 | MON | 安全模式已确认 |
| B015 | `STATUS_LED_G` | 03.FrontPanelCtrl | 09.StatusLEDs | 03 → 09 | CTRL | 绿色状态 LED |
| B016 | `STATUS_LED_Y` | 03.FrontPanelCtrl | 09.StatusLEDs | 03 → 09 | CTRL | 黄色状态 LED |
| B017 | `STATUS_LED_R` | 03.FrontPanelCtrl | 09.StatusLEDs | 03 → 09 | CTRL | 红色状态 LED |
| B018 | `SWDIO` | 09.DebugHeader | 03.MCU_Debug | BIDI | DBG | SWD 数据 |
| B019 | `SWCLK` | 09.DebugHeader | 03.MCU_Debug | 09 → 03 | DBG | SWD 时钟 |
| B020 | `NRST_MCU` | 09.DebugHeader | 03.MCU_Debug | BIDI | DBG | MCU 调试复位 |
| B021 | `FORCE_RECOVERY_REQ_N` | 09.RecoveryButton | 03.RecoveryArbiter | 09 → 03 | CTRL | 由维护页发出的恢复请求 |
| B022 | `EVENT_LOG_INT` | 03.EventCtrl | 10.SecureEventCounter | 03 → 10 | CTRL | 监控事件至安全/NVM |
| B023 | `DIO_BANK0_MODE` | 03.DIO_ModeCtrl | 07.DO_Bank0 | 03 → 07 | CTRL | DIO/DO 组模式 0 |
| B024 | `DIO_BANK1_MODE` | 03.DIO_ModeCtrl | 07.DO_Bank1 | 03 → 07 | CTRL | DIO/DO 组模式 1 |
| B025 | `ETH1_MDI0_P` | 02.SoM_ETH1 | 04.ETH1_PortPath | BIDI | HS_DIFF | 以太网端口 1 线对 0+ |
| B026 | `ETH1_MDI0_N` | 02.SoM_ETH1 | 04.ETH1_PortPath | BIDI | HS_DIFF | 以太网端口 1 线对 0− |
| B027 | `ETH1_MDI1_P` | 02.SoM_ETH1 | 04.ETH1_PortPath | BIDI | HS_DIFF | 以太网端口 1 线对 1+ |
| B028 | `ETH1_MDI1_N` | 02.SoM_ETH1 | 04.ETH1_PortPath | BIDI | HS_DIFF | 以太网端口 1 线对 1− |
| B029 | `ETH1_MDI2_P` | 02.SoM_ETH1 | 04.ETH1_PortPath | BIDI | HS_DIFF | 以太网端口 1 线对 2+ |
| B030 | `ETH1_MDI2_N` | 02.SoM_ETH1 | 04.ETH1_PortPath | BIDI | HS_DIFF | 以太网端口 1 线对 2− |
| B031 | `ETH1_MDI3_P` | 02.SoM_ETH1 | 04.ETH1_PortPath | BIDI | HS_DIFF | 以太网端口 1 线对 3+ |
| B032 | `ETH1_MDI3_N` | 02.SoM_ETH1 | 04.ETH1_PortPath | BIDI | HS_DIFF | 以太网端口 1 线对 3− |
| B033 | `ETH2_MDI0_P` | 02.SoM_ETH2 | 04.ETH2_PortPath | BIDI | HS_DIFF | 以太网端口 2 线对 0+ |
| B034 | `ETH2_MDI0_N` | 02.SoM_ETH2 | 04.ETH2_PortPath | BIDI | HS_DIFF | 以太网端口 2 线对 0− |
| B035 | `ETH2_MDI1_P` | 02.SoM_ETH2 | 04.ETH2_PortPath | BIDI | HS_DIFF | 以太网端口 2 线对 1+ |
| B036 | `ETH2_MDI1_N` | 02.SoM_ETH2 | 04.ETH2_PortPath | BIDI | HS_DIFF | 以太网端口 2 线对 1− |
| B037 | `ETH2_MDI2_P` | 02.SoM_ETH2 | 04.ETH2_PortPath | BIDI | HS_DIFF | 以太网端口 2 线对 2+ |
| B038 | `ETH2_MDI2_N` | 02.SoM_ETH2 | 04.ETH2_PortPath | BIDI | HS_DIFF | 以太网端口 2 线对 2− |
| B039 | `ETH2_MDI3_P` | 02.SoM_ETH2 | 04.ETH2_PortPath | BIDI | HS_DIFF | 以太网端口 2 线对 3+ |
| B040 | `ETH2_MDI3_N` | 02.SoM_ETH2 | 04.ETH2_PortPath | BIDI | HS_DIFF | 以太网端口 2 线对 3− |
| B041 | `ETH1_LED_LINK_N` | 02.SoM_ETH1 | 04.ETH1_LED | 02 → 04 | CTRL | 端口 1 链路 LED |
| B042 | `ETH1_LED_ACT_N` | 02.SoM_ETH1 | 04.ETH1_LED | 02 → 04 | CTRL | 端口 1 活动 LED |
| B043 | `ETH2_LED_LINK_N` | 02.SoM_ETH2 | 04.ETH2_LED | 02 → 04 | CTRL | 端口 2 链路 LED |
| B044 | `ETH2_LED_ACT_N` | 02.SoM_ETH2 | 04.ETH2_LED | 02 → 04 | CTRL | 端口 2 活动 LED |
| B045 | `RS485A_TXD` | 02.UART_A | 05.RS485A_LogicIF | 02 → 05 | DATA | Modbus RTU 通道 A 发送 |
| B046 | `RS485A_RXD` | 05.RS485A_LogicIF | 02.UART_A | 05 → 02 | DATA | Modbus RTU 通道 A 接收 |
| B047 | `RS485A_DE` | 02.UART_A | 05.RS485A_LogicIF | 02 → 05 | CTRL | 驱动使能 A |
| B048 | `RS485A_RE_N` | 02.UART_A | 05.RS485A_LogicIF | 02 → 05 | CTRL | 接收使能 A |
| B049 | `RS485A_TERM_EN` | 02.UART_A | 05.RS485A_Termination | 02 → 05 | CTRL | 可切换端接 A |
| B050 | `RS485B_TXD` | 02.UART_B | 05.RS485B_LogicIF | 02 → 05 | DATA | Modbus RTU 通道 B 发送 |
| B051 | `RS485B_RXD` | 05.RS485B_LogicIF | 02.UART_B | 05 → 02 | DATA | Modbus RTU 通道 B 接收 |
| B052 | `RS485B_DE` | 02.UART_B | 05.RS485B_LogicIF | 02 → 05 | CTRL | 驱动使能 B |
| B053 | `RS485B_RE_N` | 02.UART_B | 05.RS485B_LogicIF | 02 → 05 | CTRL | 接收使能 B |
| B054 | `RS485B_TERM_EN` | 02.UART_B | 05.RS485B_Termination | 02 → 05 | CTRL | 可切换端接 B |
| B055 | `RS485_FAULT_SUM_N` | 05.FaultAggregator | 03.BusFaultInput | 05 → 03 | MON | RS-485 汇总故障 |
| B056 | `IOL_SPI_SCLK` | 02.SPI_IOL | 06.IOLink_SPI | 02 → 06 | DATA | IO-Link SPI 时钟 |
| B057 | `IOL_SPI_MOSI` | 02.SPI_IOL | 06.IOLink_SPI | 02 → 06 | DATA | IO-Link SPI MOSI |
| B058 | `IOL_SPI_MISO` | 06.IOLink_SPI | 02.SPI_IOL | 06 → 02 | DATA | IO-Link SPI MISO |
| B059 | `IOL_CS0_N` | 02.SPI_IOL | 06.IOLink_Master0 | 02 → 06 | CTRL | IO-Link IC0 片选 |
| B060 | `IOL_CS1_N` | 02.SPI_IOL | 06.IOLink_Master1 | 02 → 06 | CTRL | IO-Link IC1 片选 |
| B061 | `IOL_INT0_N` | 06.IOLink_Master0 | 02.GPIO_IOL | 06 → 02 | MON | IO-Link IC0 中断 |
| B062 | `IOL_INT1_N` | 06.IOLink_Master1 | 02.GPIO_IOL | 06 → 02 | MON | IO-Link IC1 中断 |
| B063 | `IOL_RESET_N` | 02.GPIO_IOL | 06.IOLink_Reset | 02 → 06 | CTRL | IO-Link 主站复位 |
| B064 | `IOL0_OC_N` | 06.Port0_Fault | 03.PortFaults | 06 → 03 | MON | 端口 0 过流 |
| B065 | `IOL1_OC_N` | 06.Port1_Fault | 03.PortFaults | 06 → 03 | MON | 端口 1 过流 |
| B066 | `IOL2_OC_N` | 06.Port2_Fault | 03.PortFaults | 06 → 03 | MON | 端口 2 过流 |
| B067 | `IOL3_OC_N` | 06.Port3_Fault | 03.PortFaults | 06 → 03 | MON | 端口 3 过流 |
| B068 | `IOL0_TSD_N` | 06.Port0_Fault | 03.PortFaults | 06 → 03 | MON | 端口 0 热故障 |
| B069 | `IOL1_TSD_N` | 06.Port1_Fault | 03.PortFaults | 06 → 03 | MON | 端口 1 热故障 |
| B070 | `IOL2_TSD_N` | 06.Port2_Fault | 03.PortFaults | 06 → 03 | MON | 端口 2 热故障 |
| B071 | `IOL3_TSD_N` | 06.Port3_Fault | 03.PortFaults | 06 → 03 | MON | 端口 3 热故障 |
| B072 | `DI_SENSE0` | 07.DI_Channel0 | 02.GPIO_DI | 07 → 02 | DATA | 数字输入 0 |
| B073 | `DI_SENSE1` | 07.DI_Channel1 | 02.GPIO_DI | 07 → 02 | DATA | 数字输入 1 |
| B074 | `DI_SENSE2` | 07.DI_Channel2 | 02.GPIO_DI | 07 → 02 | DATA | 数字输入 2 |
| B075 | `DI_SENSE3` | 07.DI_Channel3 | 02.GPIO_DI | 07 → 02 | DATA | 数字输入 3 |
| B076 | `DI_SENSE4` | 07.DI_Channel4 | 02.GPIO_DI | 07 → 02 | DATA | 数字输入 4 |
| B077 | `DI_SENSE5` | 07.DI_Channel5 | 02.GPIO_DI | 07 → 02 | DATA | 数字输入 5 |
| B078 | `DI_SENSE6` | 07.DI_Channel6 | 02.GPIO_DI | 07 → 02 | DATA | 数字输入 6 |
| B079 | `DI_SENSE7` | 07.DI_Channel7 | 02.GPIO_DI | 07 → 02 | DATA | 数字输入 7 |
| B080 | `DO_CMD0` | 02.GPIO_DO | 07.DO_Channel0 | 02 → 07 | CTRL | 输出 0 开关命令 |
| B081 | `DO_CMD1` | 02.GPIO_DO | 07.DO_Channel1 | 02 → 07 | CTRL | 输出 1 开关命令 |
| B082 | `DO_CMD2` | 02.GPIO_DO | 07.DO_Channel2 | 02 → 07 | CTRL | 输出 2 开关命令 |
| B083 | `DO_CMD3` | 02.GPIO_DO | 07.DO_Channel3 | 02 → 07 | CTRL | 输出 3 开关命令 |
| B084 | `DO_CMD4` | 02.GPIO_DO | 07.DO_Channel4 | 02 → 07 | CTRL | 输出 4 开关命令 |
| B085 | `DO_CMD5` | 02.GPIO_DO | 07.DO_Channel5 | 02 → 07 | CTRL | 输出 5 开关命令 |
| B086 | `DO_CMD6` | 02.GPIO_DO | 07.DO_Channel6 | 02 → 07 | CTRL | 输出 6 开关命令 |
| B087 | `DO_CMD7` | 02.GPIO_DO | 07.DO_Channel7 | 02 → 07 | CTRL | 输出 7 开关命令 |
| B088 | `DO_EN0` | 02.GPIO_DO | 07.DO_Channel0 | 02 → 07 | CTRL | 输出通道 0 使能 |
| B089 | `DO_EN1` | 02.GPIO_DO | 07.DO_Channel1 | 02 → 07 | CTRL | 输出通道 1 使能 |
| B090 | `DO_EN2` | 02.GPIO_DO | 07.DO_Channel2 | 02 → 07 | CTRL | 输出通道 2 使能 |
| B091 | `DO_EN3` | 02.GPIO_DO | 07.DO_Channel3 | 02 → 07 | CTRL | 输出通道 3 使能 |
| B092 | `DO_EN4` | 02.GPIO_DO | 07.DO_Channel4 | 02 → 07 | CTRL | 输出通道 4 使能 |
| B093 | `DO_EN5` | 02.GPIO_DO | 07.DO_Channel5 | 02 → 07 | CTRL | 输出通道 5 使能 |
| B094 | `DO_EN6` | 02.GPIO_DO | 07.DO_Channel6 | 02 → 07 | CTRL | 输出通道 6 使能 |
| B095 | `DO_EN7` | 02.GPIO_DO | 07.DO_Channel7 | 02 → 07 | CTRL | 输出通道 7 使能 |
| B096 | `DO_FLT_N0` | 07.DO_Channel0 | 02.GPIO_DO_Fault | 07 → 02 | MON | 输出 0 故障至主机 |
| B097 | `DO_FLT_N1` | 07.DO_Channel1 | 02.GPIO_DO_Fault | 07 → 02 | MON | 输出 1 故障至主机 |
| B098 | `DO_FLT_N2` | 07.DO_Channel2 | 02.GPIO_DO_Fault | 07 → 02 | MON | 输出 2 故障至主机 |
| B099 | `DO_FLT_N3` | 07.DO_Channel3 | 02.GPIO_DO_Fault | 07 → 02 | MON | 输出 3 故障至主机 |
| B100 | `DO_FLT_N4` | 07.DO_Channel4 | 02.GPIO_DO_Fault | 07 → 02 | MON | 输出 4 故障至主机 |
| B101 | `DO_FLT_N5` | 07.DO_Channel5 | 02.GPIO_DO_Fault | 07 → 02 | MON | 输出 5 故障至主机 |
| B102 | `DO_FLT_N6` | 07.DO_Channel6 | 02.GPIO_DO_Fault | 07 → 02 | MON | 输出 6 故障至主机 |
| B103 | `DO_FLT_N7` | 07.DO_Channel7 | 02.GPIO_DO_Fault | 07 → 02 | MON | 输出 7 故障至主机 |
| B104 | `DO_FLT_N0` | 07.DO_Channel0 | 03.PortFaults | 07 → 03 | MON | 输出 0 故障至监控 |
| B105 | `DO_FLT_N1` | 07.DO_Channel1 | 03.PortFaults | 07 → 03 | MON | 输出 1 故障至监控 |
| B106 | `DO_FLT_N2` | 07.DO_Channel2 | 03.PortFaults | 07 → 03 | MON | 输出 2 故障至监控 |
| B107 | `DO_FLT_N3` | 07.DO_Channel3 | 03.PortFaults | 07 → 03 | MON | 输出 3 故障至监控 |
| B108 | `DO_FLT_N4` | 07.DO_Channel4 | 03.PortFaults | 07 → 03 | MON | 输出 4 故障至监控 |
| B109 | `DO_FLT_N5` | 07.DO_Channel5 | 03.PortFaults | 07 → 03 | MON | 输出 5 故障至监控 |
| B110 | `DO_FLT_N6` | 07.DO_Channel6 | 03.PortFaults | 07 → 03 | MON | 输出 6 故障至监控 |
| B111 | `DO_FLT_N7` | 07.DO_Channel7 | 03.PortFaults | 07 → 03 | MON | 输出 7 故障至监控 |
| B112 | `CAN1_TXD` | 02.CAN_IF | 08.CAN_Transceiver | 02 → 08 | DATA | CAN 发送 |
| B113 | `CAN1_RXD` | 08.CAN_Transceiver | 02.CAN_IF | 08 → 02 | DATA | CAN 接收 |
| B114 | `CAN1_STB_N` | 02.CAN_IF | 08.CAN_Transceiver | 02 → 08 | CTRL | CAN 待机释放 |
| B115 | `CAN1_TERM_EN` | 02.CAN_IF | 08.CAN_Termination | 02 → 08 | CTRL | CAN 端接投入 |
| B116 | `CAN1_FAULT_N` | 08.CAN_Fault | 03.BusFaultInput | 08 → 03 | MON | CAN 故障状态 |
| B117 | `USB0_D_P` | 02.USB_Device | 09.USB_C_Service | BIDI | HS_DIFF | USB 2.0 D+ |
| B118 | `USB0_D_N` | 02.USB_Device | 09.USB_C_Service | BIDI | HS_DIFF | USB 2.0 D− |
| B119 | `UART_DBG_TX` | 02.UART_Debug | 09.UART_Header | 02 → 09 | DBG | 调试控制台 TX |
| B120 | `UART_DBG_RX` | 09.UART_Header | 02.UART_Debug | 09 → 02 | DBG | 调试控制台 RX |
| B121 | `BOOT_MODE0` | 09.BootDIP | 02.BootStraps | 09 → 02 | CFG | 启动模式位 0 |
| B122 | `BOOT_MODE1` | 09.BootDIP | 02.BootStraps | 09 → 02 | CFG | 启动模式位 1 |
| B123 | `BOOT_MODE2` | 09.BootDIP | 02.BootStraps | 09 → 02 | CFG | 启动模式位 2 |
| B124 | `SPI_TPM_SCLK` | 02.SecuritySPI | 10.TPM_SPI | 02 → 10 | DATA | TPM SPI 时钟 |
| B125 | `SPI_TPM_MOSI` | 02.SecuritySPI | 10.TPM_SPI | 02 → 10 | DATA | TPM SPI MOSI |
| B126 | `SPI_TPM_MISO` | 10.TPM_SPI | 02.SecuritySPI | 10 → 02 | DATA | TPM SPI MISO |
| B127 | `SPI_TPM_CS_N` | 02.SecuritySPI | 10.TPM_SPI | 02 → 10 | CTRL | TPM 片选 |
| B128 | `TPM_IRQ_N` | 10.TPM_IRQ | 02.SecurityIRQ | 10 → 02 | MON | TPM 中断 |
| B129 | `I2C_SEC_SCL` | 02.SecurityI2C | 10.EEPROM_RTC_I2C | BIDI | DATA | 安全/NVM I²C 时钟 |
| B130 | `I2C_SEC_SDA` | 02.SecurityI2C | 10.EEPROM_RTC_I2C | BIDI | DATA | 安全/NVM I²C 数据 |
| B131 | `RTC_INT_N` | 10.RTC | 02.RTC_Wakeup | 10 → 02 | MON | RTC 报警/唤醒 |
| B132 | `BOARD_ID0` | 10.BoardID | 02.BoardVariantInput | 10 → 02 | CFG | 变体 ID 位 0 |
| B133 | `BOARD_ID1` | 10.BoardID | 02.BoardVariantInput | 10 → 02 | CFG | 变体 ID 位 1 |
| B134 | `BOARD_ID2` | 10.BoardID | 02.BoardVariantInput | 10 → 02 | CFG | 变体 ID 位 2 |
| B135 | `BOARD_ID3` | 10.BoardID | 02.BoardVariantInput | 10 → 02 | CFG | 变体 ID 位 3 |

**说明**：涵盖运行与管理路径：时序、复位、安全模式、以太网、Modbus RTU、IO-Link、DIO、CAN、维护/调试、安全/NVM；与计算/监控/现场总线/维护/安全域划分一致。

## C. 测试、量产与验证矩阵

测试、量产与验证用连接。

| 编号 | 网络名 | 源（页.块） | 目标（页.块） | 方向 | 信号类型 | 用途 |
|:---:|---|---|---|:---:|---|---|
| C001 | `TP_24V_IN_RAW` | 01.InputEntry | 11.TestPads_Power | 01 → 11 | MON | 原始输入测试点 |
| C002 | `TP_24V_IN_PROT` | 01.PowerTree | 11.TestPads_Power | 01 → 11 | MON | 受保护 24 V 测试点 |
| C003 | `TP_5V_MAIN` | 01.MainBuck | 11.TestPads_Power | 01 → 11 | MON | 5V_MAIN 测试点 |
| C004 | `TP_3V3_LOGIC` | 01.LogicReg | 11.TestPads_Power | 01 → 11 | MON | 3V3_LOGIC 测试点 |
| C005 | `TP_PWR_GOOD` | 01.PowerSeq | 11.TestPads_Power | 01 → 11 | MON | PWR_GOOD 测试点 |
| C006 | `TP_SYS_RESET_N` | 03.ResetTree | 11.TestPads_Reset | 03 → 11 | MON | 系统复位测试点 |
| C007 | `TP_WDOG_KICK` | 02.RuntimeMonitor | 11.TestPads_Reset | 02 → 11 | MON | 看门狗喂狗测试点 |
| C008 | `TP_RS485A_A` | 05.PortA_Field | 11.TestPads_Bus | 05 → 11 | MON | RS-485 A 线 A 测试点 |
| C009 | `TP_RS485A_B` | 05.PortA_Field | 11.TestPads_Bus | 05 → 11 | MON | RS-485 A 线 B 测试点 |
| C010 | `TP_RS485B_A` | 05.PortB_Field | 11.TestPads_Bus | 05 → 11 | MON | RS-485 B 线 A 测试点 |
| C011 | `TP_RS485B_B` | 05.PortB_Field | 11.TestPads_Bus | 05 → 11 | MON | RS-485 B 线 B 测试点 |
| C012 | `TP_CANH` | 08.CAN_Field | 11.TestPads_Bus | 08 → 11 | MON | CANH 测试点 |
| C013 | `TP_CANL` | 08.CAN_Field | 11.TestPads_Bus | 08 → 11 | MON | CANL 测试点 |
| C014 | `TP_IOL0_CQ` | 06.Port0_Field | 11.TestPads_IOL | 06 → 11 | MON | IO-Link 端口 0 测试点 |
| C015 | `TP_IOL1_CQ` | 06.Port1_Field | 11.TestPads_IOL | 06 → 11 | MON | IO-Link 端口 1 测试点 |
| C016 | `TP_IOL2_CQ` | 06.Port2_Field | 11.TestPads_IOL | 06 → 11 | MON | IO-Link 端口 2 测试点 |
| C017 | `TP_IOL3_CQ` | 06.Port3_Field | 11.TestPads_IOL | 06 → 11 | MON | IO-Link 端口 3 测试点 |
| C018 | `TP_DI0` | 07.DI_Channel0 | 11.TestPads_DIO | 07 → 11 | MON | DI0 测试点 |
| C019 | `TP_DI1` | 07.DI_Channel1 | 11.TestPads_DIO | 07 → 11 | MON | DI1 测试点 |
| C020 | `TP_DI2` | 07.DI_Channel2 | 11.TestPads_DIO | 07 → 11 | MON | DI2 测试点 |
| C021 | `TP_DI3` | 07.DI_Channel3 | 11.TestPads_DIO | 07 → 11 | MON | DI3 测试点 |
| C022 | `TP_DI4` | 07.DI_Channel4 | 11.TestPads_DIO | 07 → 11 | MON | DI4 测试点 |
| C023 | `TP_DI5` | 07.DI_Channel5 | 11.TestPads_DIO | 07 → 11 | MON | DI5 测试点 |
| C024 | `TP_DI6` | 07.DI_Channel6 | 11.TestPads_DIO | 07 → 11 | MON | DI6 测试点 |
| C025 | `TP_DI7` | 07.DI_Channel7 | 11.TestPads_DIO | 07 → 11 | MON | DI7 测试点 |
| C026 | `TP_DO0` | 07.DO_Channel0 | 11.TestPads_DIO | 07 → 11 | MON | DO0 测试点 |
| C027 | `TP_DO1` | 07.DO_Channel1 | 11.TestPads_DIO | 07 → 11 | MON | DO1 测试点 |
| C028 | `TP_DO2` | 07.DO_Channel2 | 11.TestPads_DIO | 07 → 11 | MON | DO2 测试点 |
| C029 | `TP_DO3` | 07.DO_Channel3 | 11.TestPads_DIO | 07 → 11 | MON | DO3 测试点 |
| C030 | `TP_DO4` | 07.DO_Channel4 | 11.TestPads_DIO | 07 → 11 | MON | DO4 测试点 |
| C031 | `TP_DO5` | 07.DO_Channel5 | 11.TestPads_DIO | 07 → 11 | MON | DO5 测试点 |
| C032 | `TP_DO6` | 07.DO_Channel6 | 11.TestPads_DIO | 07 → 11 | MON | DO6 测试点 |
| C033 | `TP_DO7` | 07.DO_Channel7 | 11.TestPads_DIO | 07 → 11 | MON | DO7 测试点 |
| C034 | `LB_RS485A_EN` | 11.LoopbackCtrl | 05.RS485A_TestMux | 11 → 05 | TEST | RS-485 A 内部回环 |
| C035 | `LB_RS485B_EN` | 11.LoopbackCtrl | 05.RS485B_TestMux | 11 → 05 | TEST | RS-485 B 内部回环 |
| C036 | `LB_CAN1_EN` | 11.LoopbackCtrl | 08.CAN_TestMux | 11 → 08 | TEST | CAN 内部回环 |
| C037 | `LB_IOL_SIM_EN` | 11.LoopbackCtrl | 06.IOLink_TestMux | 11 → 06 | TEST | IO-Link 仿真路径 |
| C038 | `FACTORY_ID_PROG` | 11.FactoryProg | 10.EEPROM_ID_Prog | BIDI | TEST/CFG | 序列号/板卡 ID 编程 |

**说明**：与 PCB 计划中 DFT/Bring-up 思路一致：电源轨、复位/看门狗、现场总线、IO-Link、DIO 及序列号编程经独立测试/工厂页引出。

## D. 紧凑页对页总览

| 源 | 目标 | 网络组摘要 |
|---|---|---|
| 01 | 02 | `5V_MAIN`, `3V3_LOGIC`, `PWR_GOOD`, `DGND` |
| 01 | 03 | `24V_IN_PROT`, `5V_MAIN`, `3V3_LOGIC`, `PWR_GOOD`, 欠压/过压/故障/电流/温度警告, `DGND` |
| 01 | 04 | `3V3_LOGIC`, `DGND`, `CHASSIS` |
| 01 | 05 | `3V3_LOGIC`, `5V_ISO_A/B`, `DGND`, `CHASSIS` |
| 01 | 06 | `24V_FIELD_A/B`, `5V_MAIN`, `3V3_LOGIC`, `DGND` |
| 01 | 07 | `24V_FIELD_C`, `3V3_LOGIC`, `DGND` |
| 01 | 08 | `3V3_LOGIC`, `5V_ISO_B`, `DGND`, `CHASSIS` |
| 01 | 09 | `5V_MAIN`, `3V3_LOGIC`, `DGND`, `CHASSIS` |
| 01 | 10 | `3V3_LOGIC`, `DGND` |
| 03 | 01 | `EN_5V_MAIN`, `EN_3V3_LOGIC`, `EN_ISO_A/B`, `EN_FIELD_A/B/C` |
| 03 | 02 | `SYS_RESET_N`, `SOM_RESET_N`, `SAFE_MODE_REQ`, `FORCE_RECOVERY_N` |
| 02 | 03 | `SOM_HEARTBEAT`, `WDOG_KICK`, `SAFE_MODE_ACK` |
| 03 | 07 | `DIO_BANK0_MODE`, `DIO_BANK1_MODE` |
| 03 | 09 | `STATUS_LED_G/Y/R` |
| 03 | 10 | `EVENT_LOG_INT` |
| 09 | 03 | `FORCE_RECOVERY_REQ_N`, `SWDIO`, `SWCLK`, `NRST_MCU` |
| 02 | 04 | `ETH1_*`, `ETH2_*`, `ETHx_LED_*` |
| 02 | 05 | `RS485A_*`, `RS485B_*` |
| 05 | 03 | `RS485_FAULT_SUM_N` |
| 02 | 06 | `IOL_SPI_*`, `IOL_CS*`, `IOL_RESET_N` |
| 06 | 02 | `IOL_SPI_MISO`, `IOL_INT0_N`, `IOL_INT1_N` |
| 06 | 03 | `IOLx_OC_N`, `IOLx_TSD_N` |
| 07 | 02 | `DI_SENSE0..7`, `DO_FLT_N0..7` |
| 02 | 07 | `DO_CMD0..7`, `DO_EN0..7` |
| 07 | 03 | `DO_FLT_N0..7` |
| 02 | 08 | `CAN1_TXD`, `CAN1_STB_N`, `CAN1_TERM_EN` |
| 08 | 02 | `CAN1_RXD` |
| 08 | 03 | `CAN1_FAULT_N` |
| 02 | 09 | `USB0_D_P/N`, `UART_DBG_TX` |
| 09 | 02 | `UART_DBG_RX`, `BOOT_MODE0..2` |
| 02 | 10 | `SPI_TPM_SCLK/MOSI/CS_N`, `I2C_SEC_SCL/SDA` |
| 10 | 02 | `SPI_TPM_MISO`, `TPM_IRQ_N`, `RTC_INT_N`, `BOARD_ID0..3` |
| 01/02/03/05/06/07/08 | 11 | `TP_*` 监测网络 |
| 11 | 05/06/08/10 | `LB_*`, `FACTORY_ID_PROG` |

---

# 附录：译者注（可选优化说明）

1. **“extensionsfähig”**：译为“具备扩展能力/可扩展”，与 Extension 范围对应。  
2. **“bankweise”**：译为“按组（Bank）”，指若干通道共享同一配置或电源分组。  
3. **“DDS/Ethernet Traffic”**（原文热测试最坏情况）：DDS 通常经以太网承载；译文保留 DDS 与以太网并列以与原文一致。  
4. 文档中 **Sheet 10** 在 PCB_design 列表为 “Security & Nonvolatile State”，在层次文件中为 `SECURITY_NVM_RTC`；**Sheet 11** 测试页命名一致，仅顶层树中 11 为 `TEST_PRODUCTION_SUPPORT`——翻译未改编号，建议 CAD 与计划文档统一别名。  
5. 矩阵中 **`FORCE_RECOVERY_REQ_N`** 为规范化新增网络名，用于单源恢复请求；与层次文件第二节表中 `FORCE_RECOVERY_N` 同时出现在 02/09/03 的情形由矩阵澄清，实施时应以矩阵为准避免多驱动。

---

**文件路径**：`/home/chuchu.xu/Downloads/QNC_local_files/qnc_work_package/product_guidelines/PCB_design_SAMPLE_de/QNC_PCB与原理图设计文档_中文翻译版.md`

**原文档**：`PCB_design.md`、`Schaltplan-Hierarchie.md`、`Sheet-to-Sheet-Verbindungsmatrix.md`
