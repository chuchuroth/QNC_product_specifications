下面给你一份**两个产品线共用的PCB方案**。核心思路是：**一块主板，两种装配配置**。

---

## 一、核心理念：一块PCB，两个SKU

```
                    ┌─────────────────────────────────────┐
                    │         QNC SmartWrist Main PCB     │
                    │              (4-layer FR4)          │
                    │             100mm × 80mm            │
                    └─────────────────────────────────────┘
                                      │
              ┌───────────────────────┼───────────────────────┐
              │                       │                       │
              ▼                       ▼                       ▼
    ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
    │  Base SKU       │     │  Pro SKU        │     │  Flex SKU       │
    │  (SmartWrist    │     │  (SmartWrist    │     │  (FlexGrip One) │
    │   Lite/China)   │     │   Premium)      │     │                 │
    ├─────────────────┤     ├─────────────────┤     ├─────────────────┤
    │ • 所有基础器件  │     │ • 所有基础器件  │     │ • 精简版基础    │
    │ • 不贴EtherCAT  │     │ • 贴EtherCAT    │     │ • 不贴EtherCAT  │
    │ • 不贴IO-Link   │     │ • 贴IO-Link     │     │ • 不贴IO-Link   │
    │ • 不贴电池充电  │     │ • 贴电池充电    │     │ • 不贴高级DI/O  │
    │ • 基础DI/O      │     │ • 完整DI/O      │     │ • 精简DI/O      │
    │ • 电容保持供电  │     │ • 电容+电池     │     │ • 仅电容保持    │
    └─────────────────┘     └─────────────────┘     └─────────────────┘
```

---

## 二、PCB分区设计

### 2.1 物理分区（功能岛）

将PCB划分为6个独立的功能区域，每个区域可以独立选择是否贴装：

```
┌─────────────────────────────────────────────────────────────────┐
│  PCB 100mm × 80mm                                                │
│  ┌──────────────┐  ┌──────────────────────────────────────────┐ │
│  │  区域1       │  │  区域2                                    │ │
│  │  核心MCU     │  │  电源管理                                 │ │
│  │  (必须)      │  │  (必须)                                   │ │
│  └──────────────┘  └──────────────────────────────────────────┘ │
│                                                                   │
│  ┌──────────────┐  ┌──────────────────────────────────────────┐ │
│  │  区域3       │  │  区域4                                    │ │
│  │  电机驱动    │  │  通信接口                                 │ │
│  │  (必须)      │  │  (RS485/CAN基础必须)                      │ │
│  └──────────────┘  └──────────────────────────────────────────┘ │
│                                                                   │
│  ┌──────────────────────────┐  ┌──────────────────────────────┐ │
│  │  区域5                   │  │  区域6                        │ │
│  │  高级通信扩展区          │  │  可选功能扩展区               │ │
│  │  (EtherCAT/IO-Link)      │  │  (电池/BLE升级/IMU)           │ │
│  │  可选贴装                │  │  可选贴装                     │ │
│  └──────────────────────────┘  └──────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 各区域详细设计

#### 区域1：核心MCU（必须，所有SKU相同）

| 元件 | 型号 | 说明 |
| :--- | :--- | :--- |
| MCU | GD32H7xx 或 STM32H7 | LQFP144，550MHz |
| Flash | W25Q128 (16MB) | QSPI，固件+配置存储 |
| RAM | 内置 | MCU内置1MB+ |

**设计要点**：
- MCU的引脚需要**同时引出**到RS485收发器、CAN收发器、EtherCAT PHY、IO-Link接口
- 通过固件检测硬件是否存在来决定启用哪些协议栈
- **所有SKU使用完全相同的MCU型号**，不降级

#### 区域2：电源管理（必须，所有SKU相同）

| 元件 | 型号 | 说明 |
| :--- | :--- | :--- |
| 输入保护 | TVS + 反接保护 + eFuse | 24V输入 |
| 24V→5V | Mornsun K7805-500R3 | 5V/1A，主逻辑供电 |
| 24V→12V | TPS54331 | 12V/1A，编码器/通信供电 |
| 5V→3.3V | LDO | 3.3V/500mA，MCU内核 |
| 超级电容 | 10F/5.4V | 保持供电，**所有SKU都有** |

**设计要点**：
- 电池充电电路（区域6的一部分）**不放在主电源区**，避免Base SKU产生不必要的BOM成本
- 超级电容是所有SKU的标配，保证500ms保持时间

#### 区域3：电机驱动（必须，所有SKU相同）

| 元件 | 型号 | 说明 |
| :--- | :--- | :--- |
| 栅极驱动器 | 例如 DRV8323 或 国产替代 | 三相或单相BLDC驱动 |
| H桥MOSFET | 4× N沟道 | 5A持续/10A峰值 |
| 电流采样 | INA240 × 2 | 两相电流，用于力估计 |

**设计要点**：
- **所有SKU使用完全相同的电机驱动级**
- 不因为FlexGrip One价格低而降低电机驱动性能——这是核心体验

#### 区域4：基础通信（必须，所有SKU相同）

| 元件 | 型号 | 数量 | 说明 |
| :--- | :--- | :--- | :--- |
| RS485收发器 | MAX3485 或 国产隔离 | 2 | 一路用于机器人通信，一路用于工具通信 |
| CAN收发器 | TJA1050 或 国产 | 1 | 可选，Base SKU可NC |
| DI输入 | 光耦 + 分压 | 2 | 24V输入 |
| DO输出 | 光耦 + MOSFET | 2 | 24V/500mA输出 |

**设计要点**：
- CAN收发器在Base SKU和FlexGrip One上**不贴装**（NC），Pro SKU贴装
- PCB上保留焊盘，通过装配BOM控制

#### 区域5：高级通信扩展区（仅Pro SKU贴装）

| 元件 | 型号 | 说明 |
| :--- | :--- | :--- |
| EtherCAT PHY | LAN9252 或 LAN8720 | SPI转EtherCAT |
| RJ45/Ethernet 磁隔离 | HX1188 | 工业级以太网变压器 |
| IO-Link 主站 | 例如 SFM10R01 | 1端口IO-Link主站 |
| IO-Link PHY | MAX14821 或 国产 | 物理层收发器 |

**设计要点**：
- 这个区域在Base SKU和FlexGrip One上**完全不贴装**
- PCB上保留完整的焊盘和走线，但NC处理
- 未来如果要升级，客户无法现场加装（需要回流焊）——这是**故意的**，为了区分SKU

#### 区域6：可选功能扩展区（按需贴装）

| 功能 | 元件 | Base | Pro | Flex |
| :--- | :--- | :--- | :--- | :--- |
| 电池充电 | TP4056 + 锂电池保护 | 不贴 | 贴 | 不贴 |
| 高精度IMU | ICM-42688 | 不贴 | 贴 | 不贴 |
| BLE模块 | ESP32-C3 | 贴 | 贴 | 贴（相同） |
| 温度传感器 | DS18B20 ×2 | 贴 | 贴 | 贴（相同） |
| 工具ID EEPROM | DS2431 | 贴 | 贴 | 贴（相同） |

**设计要点**：
- BLE、温度传感器、工具ID是所有SKU的标配——这些是平台生态的基础
- 电池和IMU是Pro SKU独有

---

## 三、三种SKU的具体装配矩阵

### 3.1 贴装/不贴装对照表

| 组件/功能 | Base SKU<br>(SmartWrist Lite) | Pro SKU<br>(SmartWrist Premium) | Flex SKU<br>(FlexGrip One) |
| :--- | :---: | :---: | :---: |
| **区域1：核心MCU** | | | |
| MCU (GD32H7) | ✔️ | ✔️ | ✔️ |
| QSPI Flash | ✔️ | ✔️ | ✔️ |
| **区域2：电源** | | | |
| 24V保护+eFuse | ✔️ | ✔️ | ✔️ |
| 24V→5V | ✔️ | ✔️ | ✔️ |
| 24V→12V | ✔️ | ✔️ | ✔️ |
| 超级电容 | ✔️ | ✔️ | ✔️ |
| 电池充电电路 | ❌ | ✔️ | ❌ |
| **区域3：电机驱动** | | | |
| 栅极驱动器 | ✔️ | ✔️ | ✔️ |
| H桥MOSFET | ✔️ | ✔️ | ✔️ |
| 电流采样 | ✔️ | ✔️ | ✔️ |
| **区域4：基础通信** | | | |
| RS485 (机器人侧) | ✔️ | ✔️ | ✔️ |
| RS485 (工具侧) | ✔️ | ✔️ | ✔️ |
| CAN收发器 | ❌ | ✔️ | ❌ |
| DI输入 (2路) | ✔️ | ✔️ | ✔️ |
| DO输出 (2路) | ✔️ | ✔️ | 1路(精简) |
| **区域5：高级通信** | | | |
| EtherCAT PHY | ❌ | ✔️ | ❌ |
| 以太网磁隔离 | ❌ | ✔️ | ❌ |
| IO-Link主站 | ❌ | ✔️ | ❌ |
| **区域6：可选功能** | | | |
| 高精度IMU | ❌ | ✔️ | ❌ |
| BLE模块 | ✔️ | ✔️ | ✔️ |
| 温度传感器(2路) | ✔️ | ✔️ | ✔️ |
| 工具ID EEPROM | ✔️ | ✔️ | ✔️ |
| **连接器** | | | |
| M12 (主接口) | ✔️ | ✔️ | ✔️ |
| M8 (服务口) | ✔️ | ✔️ | ✔️ |
| M12 D-coded (EtherCAT) | ❌ | ✔️ | ❌ |
| M8 (IO-Link) | ❌ | ✔️ | ❌ |

### 3.2 不同SKU的BOM差异

| SKU | 贴装元件数 | 相对Pro的BOM成本 | 说明 |
| :--- | :---: | :---: | :--- |
| Pro SKU | ~120个 | 100% | 完整功能 |
| Base SKU | ~85个 | ~70% | 去掉EtherCAT/IO-Link/CAN/电池/IMU |
| Flex SKU | ~75个 | ~60% | 比Base进一步精简DI/DO，更低成本电机驱动可选 |

---

## 四、固件统一方案

### 4.1 运行时硬件检测

固件启动时，通过以下方式自动检测硬件配置：

```c
// 伪代码示例
typedef enum {
    SKU_PRO,      // 完整功能
    SKU_BASE,     // 中配
    SKU_FLEX,     // 经济型
} sku_type_t;

sku_type_t detect_hardware_config(void) {
    // 1. 检查EtherCAT PHY是否存在
    if (ethercat_phy_probe() == FOUND) {
        return SKU_PRO;
    }
    
    // 2. 检查电池充电IC是否存在
    if (battery_charger_probe() == FOUND) {
        return SKU_PRO;  // 有电池充电说明是Pro
    }
    
    // 3. 检查IMU是否存在
    if (imu_probe() == FOUND) {
        return SKU_PRO;
    }
    
    // 4. 区分Base和Flex：检查DI/DO数量或特定GPIO上拉
    if (gpio_read(SKU_STRAP_PIN) == HIGH) {
        return SKU_BASE;
    } else {
        return SKU_FLEX;
    }
}
```

**PCB上增加一个SKU识别电阻**：
- 在区域6预留一个10k上拉电阻的焊盘
- Base SKU：贴装0Ω电阻 → GPIO读到HIGH
- Flex SKU：不贴 → GPIO读到LOW（内部下拉）
- Pro SKU：不需要这个识别（因为EtherCAT PHY已经能识别）

### 4.2 统一的固件镜像

- **所有SKU使用完全相同的固件二进制文件**
- 启动时根据检测到的硬件配置，启用/禁用相应的功能模块
- 功能模块通过函数指针表实现，未检测到的硬件对应的API返回`NOT_SUPPORTED`

```c
// 协议栈初始化
void comm_init(void) {
    // RS485永远可用
    modbus_init();
    
    // 可选协议栈
    if (hardware_detected(HW_ETHERCAT)) {
        ethercat_init();
    }
    
    if (hardware_detected(HW_IOLINK)) {
        iolink_init();
    }
    
    if (hardware_detected(HW_CAN)) {
        can_init();
    }
}
```

---

## 五、PCB布局建议

### 5.1 连接器布局

```
┌─────────────────────────────────────────────────────────────────┐
│                           PCB 顶部视图                          │
│  ┌─────────┐                                    ┌─────────┐     │
│  │  J1     │                                    │  J3     │     │
│  │  M12    │            MCU                     │  M12    │     │
│  │  8-pin  │           区域1                    │  D-code │     │
│  │  (主口) │                                    │(EtherCAT)│     │
│  └─────────┘                                    └─────────┘     │
│                                                                   │
│  ┌─────────┐                                    ┌─────────┐     │
│  │  J2     │           电源                      │  J4     │     │
│  │  M8     │           区域2                    │  M8     │     │
│  │(服务口) │                                    │(IO-Link)│     │
│  └─────────┘                                    └─────────┘     │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                      电机驱动 区域3                       │    │
│  │  栅极驱动 | H桥 | 电流采样 | 电机连接器                    │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                   │
│  ┌─────────────────────┐  ┌─────────────────────────────────┐   │
│  │  通信 区域4          │  │  扩展 区域5+6                   │   │
│  │  RS485×2 | DI/O     │  │  EtherCAT | IO-Link | 电池 | IMU│   │
│  └─────────────────────┘  └─────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 5.2 关键设计规则

| 规则 | 要求 |
| :--- | :--- |
| 层叠结构 | 4层：信号-GND-PWR-信号 |
| 电机驱动隔离 | 与逻辑区域保持>2mm间距，或开槽 |
| 通信隔离 | RS485/CAN/EtherCAT建议隔离（使用隔离收发器） |
| 散热 | MCU和电机驱动下方增加过孔到GND平面 |
| 测试点 | 为每个电源轨、关键信号预留测试点 |

---

## 六、生产与供应链建议

### 6.1 PCB制造

| 项目 | 要求 |
| :--- | :--- |
| PCB供应商 | JLCPCB 或 PCBWay |
| 板材 | FR4，TG≥150°C |
| 层数 | 4层 |
| 铜厚 | 1oz（电源走线可2oz） |
| 表面处理 | ENIG（沉金）或HASL-LF |
| 阻抗控制 | 100Ω±10%（EtherCAT差分对） |

### 6.2 贴装策略

| SKU | 贴装文件 | 说明 |
| :--- | :--- | :--- |
| Pro | `BOM_PRO.csv` + `CPL_PRO.csv` | 完整贴装 |
| Base | `BOM_BASE.csv` + `CPL_BASE.csv` | 跳过区域5、部分区域6 |
| Flex | `BOM_FLEX.csv` + `CPL_FLEX.csv` | 最精简 |

**一次贴装，多个SKU**：
- 先贴装所有SKU共用的元件（区域1-4的大部分）
- 然后根据SKU类型，选择性贴装区域5和区域6
- 或者：分别贴装，但共用同一个PCB裸板

### 6.3 库存管理

| 物料类型 | 策略 |
| :--- | :--- |
| 共用物料 | 集中采购，所有SKU共享库存 |
| SKU独有物料 | 按预测采购，Pro SKU的物料可做最小起订量 |
| 长交期物料 | MCU、电机驱动IC、连接器提前6个月锁定 |

---

## 七、成本对比

### 7.1 PCB+贴装成本估算（批量500pcs）

| SKU | PCB裸板 | 贴装费 | 元件成本 | 合计 |
| :--- | :---: | :---: | :---: | :---: |
| Pro | $3 | $8 | ~$85 | ~$96 |
| Base | $3 | $6 | ~$55 | ~$64 |
| Flex | $3 | $5 | ~$45 | ~$53 |

### 7.2 整机COGS对比

| SKU | PCB总成 | 机械件 | 电机+齿轮箱 | 线束+其他 | 组装 | 合计COGS |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| Pro | $96 | $85 | $39 | $15 | $25 | **$260** |
| Base | $64 | $75 | $39 | $12 | $20 | **$210** |
| Flex | $53 | $55 | $28* | $10 | $18 | **$164** |

*Flex使用更低成本的电机（$18 vs $28）

### 7.3 建议MSRP

| SKU | COGS | 建议MSRP | 毛利率 |
| :--- | :---: | :---: | :---: |
| Pro | $260 | $1,399–$1,699 | ~82–85% |
| Base | $210 | $899–$1,099 | ~77–81% |
| Flex | $164 | $599–$799 | ~72–79% |

---

## 八、总结：这个方案的核心优势

| 优势 | 说明 |
| :--- | :--- |
| **单一PCB设计** | 一次布局布线，三种产品，大幅降低工程投入 |
| **单一固件** | 所有SKU共用同一份固件，运行时自动适配 |
| **灵活的装配BOM** | 通过贴装/不贴装区分SKU，不需要改PCB |
| **清晰的升级路径** | 客户可以从Flex升级到Base再到Pro，但需要买新硬件（保护高端SKU） |
| **共用供应链** | 长交期物料（MCU、连接器）统一采购，降低风险 |
| **成本可控** | Flex SKU的COGS可以压到$164以下，MSRP $599仍有70%+毛利 |

**一句话总结**：

> 用一块PCB、一套固件、三份装配BOM，覆盖从$599到$1,699的价格带，同时保持Pro SKU的高端功能独占性——这正是苹果iPhone的产品线策略（标准版/Pro/Pro Max），你把它应用到了机器人EOAT层。

需要我进一步细化某个区域的原理图，或者准备一份给PCB工厂的Layout Guideline吗？

---
---
下面是一份**可直接交付给PCB工厂和Layout工程师**的Layout Guideline文档。

这份文档按照专业PCB设计规范编写，包含机械尺寸、层叠结构、走线规则、阻抗控制、EMC设计要求等。你可以直接作为设计输入文件发给PCB设计团队或工厂。

---

# QNC SmartWrist Platform - PCB Layout Guideline

## Document Control

| Item | Value |
| :--- | :--- |
| Document Title | PCB Layout Guideline for QNC SmartWrist Platform |
| Version | v1.0 |
| Status | Release for Layout |
| Target PCB | 4-layer, FR4, 100mm × 80mm |
| Applicable SKUs | Pro / Base / Flex (unified PCB) |
| Intended Use | Design input for PCB layout engineer and PCB fabrication |

---

## 1. General Requirements

### 1.1 PCB Basic Specifications

| Parameter | Requirement | Notes |
| :--- | :--- | :--- |
| PCB Dimensions | 100mm × 80mm (±0.2mm) | Rectangular |
| PCB Thickness | 1.6mm (±10%) | Standard |
| Layer Count | 4 layers | Stackup defined in Section 2 |
| Material | FR4, TG ≥ 150°C | High Tg for industrial temp range |
| Copper Weight | 1 oz (all layers) | Power traces may use 2 oz |
| Surface Finish | ENIG (Immersion Gold) | Recommended for fine-pitch and reliability |
| Solder Mask Color | Black (standard) | Matte finish preferred |
| Silkscreen Color | White | Legible, minimum 0.2mm line width |
| Board Outline Tolerance | ±0.1mm | |
| Hole Diameter Tolerance | ±0.05mm (PTH), ±0.03mm (NPTH) | |
| Warpage | ≤ 0.75% | For SMT assembly |

### 1.2 Operating Environment

| Parameter | Requirement |
| :--- | :--- |
| Temperature Range | -40°C to +85°C (industrial) |
| Humidity | 5% to 95% non-condensing |
| Vibration | IEC 60068-2-6 compliant |

---

## 2. Layer Stackup

### 2.1 Recommended Stackup

```
┌─────────────────────────────────────────────────────────────────┐
│ Layer 1 (Top)      : Signal + Components + Ground pour          │
│ ===================== 0.2mm prepreg ============================ │
│ Layer 2 (Inner 1)  : GND (solid plane)                          │
│ ===================== 1.0mm core =============================== │
│ Layer 3 (Inner 2)  : Power (split planes: 24V, 12V, 5V, 3.3V)   │
│ ===================== 0.2mm prepreg ============================ │
│ Layer 4 (Bottom)   : Signal + Components + Ground pour          │
└─────────────────────────────────────────────────────────────────┘

Total thickness: 1.6mm (±10%)
```

### 2.2 Layer Assignment

| Layer | Type | Primary Use | Fill/Pour |
| :--- | :--- | :--- | :--- |
| Layer 1 (Top) | Signal | MCU, connectors, high-speed signals | GND pour |
| Layer 2 (Inner 1) | GND Plane | Solid ground reference | 100% GND (no splits) |
| Layer 3 (Inner 2) | Power Plane | Split power rails | 24V / 12V / 5V / 3.3V planes |
| Layer 4 (Bottom) | Signal | Motor drive, low-speed signals, passives | GND pour |

### 2.3 Impedance Control Requirements

| Signal Type | Impedance | Tolerance | Layer Pair |
| :--- | :--- | :--- | :--- |
| Ethernet (EtherCAT) diff pair | 100Ω | ±10% | Top - L2 |
| USB (D+/D-) diff pair | 90Ω | ±10% | Top - L2 |
| CAN bus diff pair | 120Ω | ±10% | Top - L2 |
| RS485 diff pair | 120Ω | ±10% | Top - L2 |
| Single-ended 50Ω signals | 50Ω | ±15% | Top - L2 |

---

## 3. Mechanical & Mounting

### 3.1 Board Outline

```
                    Y
                    ↑
                    │
      ┌─────────────┼─────────────────────────────┐
      │             │                             │
      │   100mm     │                             │
      │  ┌──────┐   │                             │
      │  │      │   │                             │
      │  │ MCU  │   │                             │
      │  │ area │   │                             │
      │  └──────┘   │                             │
      │             │                             │
      │             │         80mm                │
      │             │                             │
      │             │                             │
      │             │                             │
      │             │                             │
      │             │                             │
      └─────────────┴─────────────────────────────┘
                    │
                    └─────────────────────────────→ X
```

### 3.2 Mounting Holes

| Hole ID | X (mm) | Y (mm) | Diameter | Type | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- |
| MH1 | 5 | 5 | 3.2mm | NPTH | Mounting to housing |
| MH2 | 95 | 5 | 3.2mm | NPTH | Mounting to housing |
| MH3 | 5 | 75 | 3.2mm | NPTH | Mounting to housing |
| MH4 | 95 | 75 | 3.2mm | NPTH | Mounting to housing |

**Mounting hole requirements:**
- Keep 3mm clearance around each mounting hole (no traces, no components)
- Connect to chassis ground via 1nF/2kV capacitor (parallel with 1MΩ resistor)
- No solder mask on mounting hole pads

### 3.3 Keep-out Areas

| Area | X Range (mm) | Y Range (mm) | Restriction |
| :--- | :--- | :--- | :--- |
| Connector keep-out (J1) | 0–15 | 30–50 | No components within 3mm of connector |
| Connector keep-out (J3) | 85–100 | 30–50 | No components within 3mm of connector |
| Motor connector area | 40–60 | 70–80 | High-current, keep logic away |
| Antenna keep-out (BLE) | 70–80 | 5–15 | No GND pour under antenna |

---

## 4. Component Placement

### 4.1 Placement Hierarchy (Priority Order)

| Priority | Component Type | Placement Rule |
| :--- | :--- | :--- |
| 1 | Connectors (J1-J4) | Fixed positions, along board edges |
| 2 | MCU (U1) | Center of board, accessible for routing |
| 3 | Motor drive (Q1-Q4, U2) | Near motor connector, with heat dissipation |
| 4 | Power supply (U3-U6) | Near input connector (J1) |
| 5 | Communication ICs | Near respective connectors |
| 6 | Decoupling capacitors | Within 2mm of IC power pins |
| 7 | Passive components | After all active components |

### 4.2 Key Component Positions

| Component | Reference | X (mm) | Y (mm) | Notes |
| :--- | :--- | :--- | :--- | :--- |
| Main MCU | U1 | 50 | 40 | Center, LQFP144 |
| M12 main connector | J1 | 0 | 40 | Left edge, centered |
| M8 service port | J2 | 0 | 65 | Left edge, lower |
| M12 EtherCAT (Pro) | J3 | 100 | 40 | Right edge (optional) |
| M8 IO-Link (Pro) | J4 | 100 | 65 | Right edge (optional) |
| Motor connector | J5 | 50 | 80 | Bottom edge |
| BLDC gate driver | U2 | 50 | 65 | Near motor connector |
| H-bridge MOSFETs | Q1-Q4 | 45–55 | 70–75 | Grouped, with heatsink vias |
| RS485 transceivers | U7, U8 | 20 | 20 | Near J1 |
| EtherCAT PHY (Pro) | U9 | 80 | 35 | Near J3 |
| BLE module | U10 | 75 | 10 | Corner, antenna facing out |
| Power management | U3-U6 | 15 | 50–70 | Near J1 |

### 4.3 Placement Rules

| Rule | Requirement |
| :--- | :--- |
| Component orientation | All ICs with pin 1 in same direction (left or up) |
| Decoupling caps | One 0.1µF per power pin, within 2mm |
| Bulk caps | One 10µF–47µF per power rail, near regulator output |
| Crystal | Within 10mm of MCU, with ground guard ring |
| High-current path | Short, wide traces, no sharp corners |
| Thermal relief | Use spokes on power planes for PTH components |

---

## 5. Routing Rules

### 5.1 General Routing Rules

| Parameter | Requirement |
| :--- | :--- |
| Minimum trace width (signal) | 0.15mm (6 mil) |
| Minimum trace width (power) | 0.5mm (20 mil) for 1A, 1.0mm (40 mil) for 3A |
| Minimum spacing (signal to signal) | 0.15mm (6 mil) |
| Minimum spacing (signal to GND) | 0.2mm (8 mil) |
| Minimum via diameter | 0.3mm hole, 0.6mm pad |
| Minimum via-to-via spacing | 0.5mm |
| Maximum trace length mismatch (diff pair) | 0.5mm |
| 45° corners | Use 45° or arc, no 90° corners |

### 5.2 High-Speed Signal Routing

| Signal Type | Length Matching | Differential Spacing | Reference Plane |
| :--- | :--- | :--- | :--- |
| Ethernet (EtherCAT) | ±0.5mm | 0.2mm (edge-to-edge) | L2 (GND) |
| USB D+/D- | ±0.5mm | 0.2mm | L2 (GND) |
| CAN H/L | ±1mm | 0.2mm | L2 (GND) |
| RS485 A/B | ±1mm | 0.2mm | L2 (GND) |
| SPI (QSPI flash) | < 10mm total | N/A | L2 (GND) |

### 5.3 Power Routing

| Power Rail | Min Width | Via Count | Notes |
| :--- | :--- | :--- | :--- |
| 24V input | 1.5mm (60 mil) | 4+ vias at connector | Use polygon pour |
| 24V to motor | 2.0mm (80 mil) | 6+ vias | Star connection |
| 12V rail | 0.8mm (32 mil) | 2+ vias | |
| 5V rail | 0.5mm (20 mil) | 2+ vias | |
| 3.3V rail | 0.3mm (12 mil) | 1+ via per IC | |

### 5.4 Current Carrying Capacity

| Current | Trace Width (1 oz, 10°C rise) | Via Count (0.3mm hole) |
| :--- | :--- | :--- |
| 1A | 0.5mm | 1 via |
| 2A | 1.0mm | 2 vias |
| 3A | 1.5mm | 3 vias |
| 4A (peak) | 2.0mm + polygon | 4 vias |

### 5.5 Return Path Requirements

| Rule | Requirement |
| :--- | :--- |
| Solid GND plane | Layer 2 must be continuous (no splits under signals) |
| Signal return | Every high-speed signal must have GND return within 0.5mm |
| Via stitching | GND vias every 5mm along board edges |
| Split plane crossing | Never route signals across split planes |

---

## 6. Power Delivery Network (PDN)

### 6.1 Power Plane Splitting (Layer 3)

```
┌─────────────────────────────────────────────────────────────────┐
│ Layer 3 - Power Plane Split                                     │
│                                                                  │
│  ┌────────────┐  ┌────────────────────────────────────────────┐ │
│  │  24V Plane │  │               5V Plane                     │ │
│  │            │  │                                            │ │
│  │  (Input)   │  │  (Logic, MCU, comms)                       │ │
│  └────────────┘  └────────────────────────────────────────────┘ │
│                                                                  │
│  ┌────────────────────┐  ┌────────────────────────────────────┐ │
│  │     12V Plane      │  │          3.3V Plane                │ │
│  │                    │  │                                    │ │
│  │  (Encoder, RS485)  │  │  (MCU core, sensors)               │ │
│  └────────────────────┘  └────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 Decoupling Capacitor Requirements

| IC | Power Pin | Capacitor Value | Quantity | Placement |
| :--- | :--- | :--- | :--- | :--- |
| MCU | VDD (3.3V) | 0.1µF + 4.7µF | 1+1 per 2 pins | Within 2mm |
| MCU | VDDA (analog) | 0.1µF + 10µF | 1+1 | Within 5mm |
| Motor driver | VM (24V) | 10µF + 100µF | 1+1 | Within 10mm |
| RS485 | VCC (5V) | 0.1µF | 1 per IC | Within 2mm |
| EtherCAT PHY | VDD (3.3V) | 0.1µF + 10µF | 1+1 | Within 5mm |
| BLE module | VDD (3.3V) | 0.1µF + 4.7µF | 1+1 | Within 5mm |

### 6.3 Bulk Capacitance

| Power Rail | Capacitance | Voltage Rating | Placement |
| :--- | :--- | :--- | :--- |
| 24V input | 100µF + 10µF | 50V | Near J1 |
| 5V rail | 47µF | 10V | Near DC/DC output |
| 12V rail | 22µF | 25V | Near DC/DC output |
| 3.3V rail | 22µF | 6.3V | Near LDO output |

---

## 7. Signal Integrity

### 7.1 Clock Routing

| Clock | Frequency | Routing Rule |
| :--- | :--- | :--- |
| MCU main oscillator (HSE) | 25MHz | Guard ring with GND vias, keep length < 15mm |
| RTC oscillator (LSE) | 32.768kHz | Guard ring, minimize parasitic capacitance |
| Ethernet PHY clock | 25MHz | Match length to PHY, series termination at source |

**Crystal layout rules:**
- Place crystal within 10mm of MCU OSC pins
- Ground guard ring around crystal (via fence)
- No other traces under crystal
- Load capacitors (18pF–22pF) placed within 2mm of crystal

### 7.2 High-Speed Interface Routing

| Interface | Max Length | Reference | Termination |
| :--- | :--- | :--- | :--- |
| Ethernet (RMII) | < 50mm | L2 | 33Ω series (optional) |
| QSPI (Flash) | < 40mm | L2 | 22Ω series (clock) |
| USB (D+/D-) | < 100mm | L2 | 27Ω series, 15kΩ pull-down |

### 7.3 Signal Integrity Rules

| Rule | Requirement |
| :--- | :--- |
| Stub length | Maximum 3mm for any signal |
| Via count per signal | Maximum 2 vias for high-speed signals |
| Series termination | Place within 5mm of driver pin |
| Parallel termination | Place at receiver end (if used) |

---

## 8. EMC & EMI Design

### 8.1 Grounding Strategy

| Rule | Requirement |
| :--- | :--- |
| Chassis ground (PE) | Separate from signal GND, connect via 1nF/2kV cap |
| Signal GND | Solid plane on Layer 2 |
| GND pour on top/bottom | Stitch to Layer 2 every 5mm with vias |
| Mounting holes | Connect to chassis ground via 1nF/2kV + 1MΩ |

### 8.2 Shielding and Guarding

| Area | Shielding Method |
| :--- | :--- |
| MCU | GND pour around perimeter, via fence |
| Crystal | GND guard ring, via fence |
| Motor drive | Separate GND island, connect at single point |
| BLE antenna | No GND under antenna, keep-out area |

### 8.3 Filtering

| Signal/Line | Filter Type | Components |
| :--- | :--- | :--- |
| 24V input | Common mode choke | L1 (optional) |
| 24V input | TVS + capacitor | D1 (TVS), C1 (0.1µF) |
| Ethernet | Common mode choke | Built into RJ45 magjack |
| USB | Common mode choke | L2 (optional) |
| DI inputs | RC filter | 1kΩ + 1nF |

### 8.4 Ferrite Beads

| Rail | Ferrite Bead | Rating |
| :--- | :--- | :--- |
| 5V to analog section | BLM21PG221SN1 | 220Ω @ 100MHz |
| 3.3V to MCU PLL | BLM15HD102SN1 | 1000Ω @ 100MHz |
| USB VBUS | BLM21PG221SN1 | 220Ω @ 100MHz |

---

## 9. Thermal Management

### 9.1 Heat Dissipation Areas

| Component | Power Dissipation | Cooling Method |
| :--- | :--- | :--- |
| MCU (GD32H7) | ~0.5W | GND pour, thermal vias |
| Motor driver (gate driver + MOSFETs) | ~2W (peak) | Thermal vias to bottom, optional heatsink pad |
| 24V→5V DC/DC | ~0.3W | GND pour |
| 24V→12V DC/DC | ~0.2W | GND pour |

### 9.2 Thermal Via Pattern

| Component | Via Pattern | Via Size |
| :--- | :--- | :--- |
| MCU exposed pad | 4×4 array | 0.3mm hole, 0.6mm pad |
| MOSFETs (each) | 3×3 array | 0.3mm hole, 0.6mm pad |
| Motor driver IC | 2×2 array | 0.3mm hole, 0.6mm pad |

### 9.3 Heatsink Pad (Bottom side)

For Pro SKU, add a heatsink pad area under motor drive:

| Parameter | Requirement |
| :--- | :--- |
| Location | Bottom side, under Q1-Q4 |
| Size | 15mm × 15mm |
| Finish | ENIG, no solder mask |
| Thermal vias | 0.3mm hole, 0.6mm pad, 0.8mm pitch |

---

## 10. Manufacturing & Assembly

### 10.1 SMT Requirements

| Parameter | Requirement |
| :--- | :--- |
| Fiducial marks | 3 marks (corners), 1mm diameter, no solder mask |
| Panelization | 2×2 or 2×3 array, V-score or tab routing |
| Tooling holes | 4 holes, 3mm diameter, corners |
| Solder paste stencil | 0.12mm thickness, laser-cut |
| Component spacing | Minimum 0.5mm between components |

### 10.2 Test Points

| Test Point | Signal | Location |
| :--- | :--- | :--- |
| TP1 | 24V_IN | Near J1 |
| TP2 | 5V | Near U3 output |
| TP3 | 3.3V | Near MCU |
| TP4 | GND | Multiple locations |
| TP5 | SWD_CLK | MCU programming |
| TP6 | SWD_DIO | MCU programming |
| TP7 | BOOT0 | MCU boot select |
| TP8 | RESET | MCU reset |

**Test point requirements:**
- 1.0mm diameter round pad
- No solder mask on test points
- Accessible from top side
- Label with silkscreen

### 10.3 Programming & Debug Interface

| Signal | Pin | Description |
| :--- | :--- | :--- |
| SWD_CLK | TP5 | 10k pull-down |
| SWD_DIO | TP6 | 10k pull-up |
| NRST | TP8 | 10k pull-up, 0.1µF to GND |
| BOOT0 | TP7 | 10k pull-down |
| VDD (3.3V) | - | For programmer power |
| GND | TP4 | |

### 10.4 Assembly Notes

| Note | Requirement |
| :--- | :--- |
| Moisture sensitivity | MSL3 handling required for certain ICs |
| ESD protection | All handling in ESD-safe area |
| Conformal coating | Optional for Pro SKU (IP65) |
| Bottom side components | Avoid under motor drive area if heatsink used |

---

## 11. DFM (Design for Manufacturing) Checklist

| Item | Requirement | Status |
| :--- | :--- | :--- |
| Minimum trace width | ≥ 0.15mm | ☐ |
| Minimum spacing | ≥ 0.15mm | ☐ |
| Minimum via hole | ≥ 0.3mm | ☐ |
| Minimum annular ring | ≥ 0.15mm | ☐ |
| SMD pad to pad spacing | ≥ 0.3mm | ☐ |
| Component to board edge | ≥ 1.5mm | ☐ |
| Fiducial marks present | 3 marks | ☐ |
| Tooling holes present | 4 holes | ☐ |
| Silkscreen on solder mask | Yes (not on pads) | ☐ |
| Solder mask sliver | ≥ 0.1mm | ☐ |
| Copper to board edge | ≥ 0.5mm | ☐ |

---

## 12. Revision History

| Version | Date | Author | Changes |
| :--- | :--- | :--- | :--- |
| v1.0 | 2026-04-08 | QNC Engineering | Initial release for layout |

---

## 13. Appendix: Reference Documents

| Document | Source | Purpose |
| :--- | :--- | :--- |
| SmartWrist Interface Spec v2.0 | QNC | Electrical interface definition |
| QNC BOM v1.0 | QNC | Component list and specifications |
| GD32H7xx Datasheet | GigaDevice | MCU pinout and electrical specs |
| IPC-2221 | IPC | PCB design standards |
| IPC-7351 | IPC | SMD land pattern standards |

---

**End of Document**

---

## 快速参考卡片（供Layout工程师打印）

```
┌─────────────────────────────────────────────────────────────────┐
│                    QNC SmartWrist PCB Layout                    │
│                      Quick Reference Card                       │
├─────────────────────────────────────────────────────────────────┤
│ 尺寸: 100×80mm | 层数: 4 | 厚度: 1.6mm | TG≥150°C | ENIG        │
├─────────────────────────────────────────────────────────────────┤
│ 层叠:   L1: Signal+GND  |  L2: GND Plane  |  L3: Power Split   │
│        L4: Signal+GND   |  阻抗: 100Ω diff / 90Ω USB / 50Ω SE   │
├─────────────────────────────────────────────────────────────────┤
│ 关键走线:                                                       │
│   • 以太网差分: 100Ω, ±0.5mm, 0.2mm间距                        │
│   • USB差分: 90Ω, ±0.5mm, 0.2mm间距                            │
│   • 电源: 24V: 1.5mm, 5V: 0.5mm, 3.3V: 0.3mm                   │
├─────────────────────────────────────────────────────────────────┤
│ 关键位置:                                                       │
│   J1(M12主口): (0,40)  |  J3(EtherCAT): (100,40)               │
│   MCU: (50,40)         |  电机驱动: (45-55, 70-75)              │
├─────────────────────────────────────────────────────────────────┤
│ EMC要点:                                                        │
│   • L2为完整GND平面，不得分割                                   │
│   • 高速信号禁止跨越分割平面                                    │
│   • GND过孔每5mm一个                                           │
│   • 晶体加GND保护环                                            │
├─────────────────────────────────────────────────────────────────┤
│ 检查清单: □ 阻抗控制 □ 长度匹配 □ 去耦电容 □ 测试点 □ 安装孔   │
└─────────────────────────────────────────────────────────────────┘
```

这份文档你可以直接发给PCB layout工程师或PCB工厂作为设计输入。需要我补充某个特定部分（比如具体的差分对长度计算、或者某个区域的详细布局）吗？
