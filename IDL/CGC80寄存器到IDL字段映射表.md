基于你提供的 `DH-Robotics_CGC-80_RTU.json`，我直接给你**精确的寄存器到 IDL 字段映射表**，以及适配层伪代码。
## CGC-80 关键寄存器映射总结

| IDL 字段 | Modbus 寄存器 | 读/写 | 原始范围 | 归一化公式 (0.0~1.0) | 备注 |
|----------|---------------|-------|----------|---------------------|------|
| **position** | 514 (实际位置) / 259 (Echo) | READ_INPUT / READ_HOLDING | 0~1000 | `raw / 1000.0` | 位置百分比 ×10 |
| **force** | 257 | READ_HOLDING / WRITE_HOLDING | 20~100% | `(raw - 20) / 80` | 闭合力百分比 |
| **speed** | 260 | WRITE_HOLDING | 1~100% | `raw / 100.0` | 速度百分比 |
| **status** | 513 | READ_INPUT | 0~3 | 见状态映射表 | 夹爪状态 |
| **error** | QNC Response | - | - | EC_* 常量 | 通过桥反馈 |

## GripperState 状态映射表

CGC-80 的 `getGripperState` (寄存器 513) 有 4 个状态，直接映射：

| 寄存器值 | 含义 | → GripperStatus | → GraspResult |
|----------|------|----------------|---------------|
| 0 | In motion | `STATUS_MOVING` | `GRASP_IN_PROGRESS` |
| 1 | Reached position | `STATUS_IDLE` | `GRASP_UNKNOWN` |
| 2 | Object caught | `STATUS_HOLDING` | `GRASP_SUCCESS` |
| 3 | Object dropped | `STATUS_IDLE` | `GRASP_SLIPPED` |

## 适配层实现伪代码

### 1. GripperCapabilities（静态，启动时发布一次）

```cpp
// 从 JSON 解析后，填充 Capabilities
GripperCapabilities caps;
caps.device_id = "DH-Robotics_CGC-80";
caps.manufacturer = "DH-Robotics";
caps.model = "CGC-80";
caps.has_position_control = true;
caps.has_force_control = true;      // 支持闭合力控制
caps.has_speed_control = true;      // 支持速度控制
caps.has_multi_joint = false;       // 单逻辑关节
caps.joint_count = 1;
caps.joints[0].min_position = 0.0;
caps.joints[0].max_position = 1.0;
caps.joints[0].max_speed = 1.0;
caps.joints[0].max_force = 1.0;     // (100-20)/80 = 1.0
caps.has_grasp_detection = true;    // 有 Object caught / dropped
publish_caps(caps);
```

### 2. GripperCommand → Modbus WriteMultipleCommand

收到 `GripperCommand` 后：

```cpp
void on_gripper_command(GripperCommand cmd) {
    // 映射到寄存器值
    uint16_t force_raw = 20 + cmd.max_force * 80;           // 20~100
    uint16_t speed_raw = cmd.max_speed * 100;               // 1~100
    uint16_t position_raw = cmd.position * 1000;             // 0~1000
    
    // 打包成 WriteMultipleCommand (寄存器连续写，效率更高)
    WriteMultipleCommand multi_cmd;
    multi_cmd.slave_id = 1;  // 从 JSON
    multi_cmd.start_register = 257;  // force(257), speed(260), pos(259) 不连续，手动分开写
    
    // 先写 force + speed（连续）
    multi_cmd.values = {force_raw, 0, 0, speed_raw};  // 257,258(padding),259(pos稍后),260
    multi_cmd.tag = cmd.tag;
    publish_write_multi(multi_cmd);
    
    // 再写 position (259)
    WriteCommand pos_cmd;
    pos_cmd.slave_id = 1;
    pos_cmd.register_address = 259;
    pos_cmd.value = position_raw;
    pos_cmd.tag = cmd.tag + "_pos";
    publish_write_single(pos_cmd);
}
```

### 3. Modbus Response → GripperState（周期性读取）

QNC 定期读取状态寄存器后：

```cpp
void on_modbus_response(Response resp) {
    if (resp.slave_id != 1 || resp.error_code != EC_SUCCESS) {
        // 错误处理
        GripperState state;
        state.device_id = "DH-Robotics_CGC-80";
        state.error_code = resp.error_code;
        state.message = resp.error_message;
        state.status = STATUS_ERROR;
        publish_state(state);
        return;
    }
    
    // 从定期读取的状态寄存器组装 GripperState
    GripperState state;
    state.device_id = "DH-Robotics_CGC-80";
    state.last_command_tag = get_last_correlated_tag(resp.tag);
    state.last_command_latency_us = resp.latency_us;
    
    // position: 寄存器 514 (实际位置)
    if (has_register(resp, 514)) {
        uint16_t pos_raw = resp.data[0];
        state.position = pos_raw / 1000.0f;  // 0~1000 → 0.0~1.0
    } else {
        state.position = get_last_commanded_position();  // 回退到命令值
    }
    
    // force: 寄存器 257 echo
    if (has_register(resp, 257)) {
        uint16_t force_raw = resp.data[0];
        state.force = (force_raw - 20.0f) / 80.0f;  // 20~100 → 0.0~1.0
    } else {
        state.force = 0.0f;
    }
    
    // speed: 当前无反馈，设为0或估算
    state.speed = 0.0f;
    
    // status: 寄存器 513
    if (has_register(resp, 513)) {
        uint16_t status_raw = resp.data[0];
        switch(status_raw) {
            case 0: state.status = STATUS_MOVING; state.grasp_result = GRASP_IN_PROGRESS; break;
            case 1: state.status = STATUS_IDLE; state.grasp_result = GRASP_UNKNOWN; break;
            case 2: state.status = STATUS_HOLDING; state.grasp_result = GRASP_SUCCESS; break;
            case 3: state.status = STATUS_IDLE; state.grasp_result = GRASP_SLIPPED; break;
            default: state.status = STATUS_UNKNOWN; break;
        }
    }
    
    state.error_code = 0;  // 成功
    state.timestamp = get_iso_timestamp();
    publish_state(state);
}
```

### 4. 周期性监控配置

在 QNC 的循环读取中，确保包含这些寄存器：
```json
// 在你的监控配置里加这些
"monitored_registers": [257, 259, 513, 514]  // force echo, pos echo, status, actual pos
```

## 测试验证步骤

1. **启动后检查 Capabilities**：确认 `has_force_control=true`, `max_force=1.0`
2. **发命令测试**：`position=1.0, max_force=0.5, max_speed=0.8`
   - 期望 Modbus：257=60(%), 260=80(%), 259=1000(1/10%)
3. **观察 State**：position 应该渐变到 1.0，status 变成 `HOLDING` 或 `MOVING`
4. **错误处理**：断开夹爪，观察 `error_code` 和 QNC 的 `BridgeStats`

这样，你的通用 Gripper IDL 就能**零改动**无缝适配 CGC-80，其他 DH 夹爪只要按相同逻辑写 JSON 描述符，就能复用同一套 DDS 话题和上层逻辑！ 

---
---
