**Open ModSim** 是一款**免费、开源的 Modbus 从站（Slave/Server）模拟器**，主要用于模拟 Modbus TCP 和 Modbus RTU 设备，方便开发者和工程师在没有真实硬件的情况下测试主站系统。

它的名字和经典的商业软件 **ModSim32** 很像，可以看作是后者的开源替代品。

### 核心功能

Open ModSim 支持模拟 Modbus 协议中的各类数据点，包括线圈（Coils）、离散输入（Discrete Inputs）、保持寄存器（Holding Registers）和输入寄存器（Input Registers）。

具体来说，它支持以下 Modbus 功能码：

| 数据类型 | 支持的功能码 |
| :--- | :--- |
| **线圈/标志位** | 0x01 - 读线圈<br>0x02 - 读离散输入<br>0x05 - 写单线圈<br>0x0F - 写多线圈 |
| **寄存器** | 0x03 - 读保持寄存器<br>0x04 - 读输入寄存器<br>0x06 - 写单寄存器<br>0x10 - 写多寄存器<br>0x16 - 掩码写寄存器 |

### 数据模拟方式

这是 Open ModSim 非常实用的功能。你可以让模拟的数据按照特定规律变化，从而模拟真实的传感器或执行器行为：

**对于线圈/标志位**：
- **Random（随机）**：标志位随机变化
- **Toggle（翻转）**：标志位周期性翻转

**对于寄存器**：
- **Random（随机）**：寄存器值随机变化
- **Increment（递增）**：从低限到高限按步长递增
- **Decrement（递减）**：从高限到低限按步长递减

### 典型使用场景

1.  **PLC 开发调试**：当你在写 PLC 程序（如西门子 S7-1200）需要读取 Modbus 设备数据时，可以先用 Open ModSim 模拟服务器，验证通信配置是否正确。
2.  **SCADA/HMI 测试**：在连接真实传感器之前，用 Open ModSim 模拟大量数据点，测试上位机的数据显示和报警功能。
3.  **协议学习**：初学者可以用它作为从站，配合 ModScan（主站客户端）直观地学习 Modbus 协议的工作原理。

### 获取与运行

- **项目主页**：https://sanny32.github.io/OpenModSim/ 
- **GitHub 源码**：https://github.com/sanny32/OpenModSim 
- **运行方式**：提供了不同版本（如 Qt5、Qt6），可根据操作系统（Windows/Linux）选择。下载后无需安装，直接运行可执行文件即可。

### 与其他工具的关系

- **Open ModSim** = **从站模拟器**（Slave Simulator）。它扮演**被查询的设备**（如传感器），等待主站来读取数据。
- **ModScan / Open ModScan** = **主站客户端**（Master Client）。它扮演**查询设备**（如 PLC、上位机），主动去读取从站的数据。

简单来说，**Open ModSim 是给“主站”开发者用的测试工具**。当你写完一个主站程序（比如用你手上的 Nucleo-F446RE 做一个 Modbus 主站），就可以用 Open ModSim 来验证你的程序能否正确读写数据。
