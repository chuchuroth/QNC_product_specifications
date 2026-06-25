先给你一版“能直接改着用”的极简版本，风格尽量贴近 DDS/FastDDS/ROS 2 的习惯，用归一化开度和力、简单枚举，方便后面扩展。 [control.ros](https://control.ros.org/humble/doc/ros2_controllers/gripper_controllers/doc/userdoc.html)

下面假设三个文件名分别是：

- `GripperCommand.idl`
- `GripperState.idl`
- `GripperCapabilities.idl`

***

## GripperCommand.idl（控制命令）

```idl
module universal_gripper
{

// 夹爪模式（可按项目需要精简/扩展）
enum GripperMode
{
    MODE_POSITION = 0,   // 位置/开度控制
    MODE_FORCE    = 1,   // 力控制为主（位置次要）
    MODE_BASIC    = 2    // 简单开/关型夹爪
};

// 通用命令：尽量抽象，少字段
struct GripperCommand
{
    // 目标开度，0.0 = 完全张开，1.0 = 完全闭合
    float position;

    // 目标最大力（归一化 0.0~1.0，是否支持由 Capabilities 决定）
    float max_force;

    // 目标最大速度（归一化 0.0~1.0，可选，0 或负值表示使用默认）
    float max_speed;

    // 期望工作模式（位置优先、力优先等）
    GripperMode mode;

    // 是否为“阻塞式”命令：true 表示直到动作完成/失败才认为执行结束
    boolean blocking;

    // 超时时间（秒），blocking=true 时有效，<=0 表示使用设备默认
    float timeout;
};

};
```

设计要点（方便你后面迭代）：

- position / max_force / max_speed 都做 0~1 归一化，上层逻辑不用关心具体行程和牛顿值，适配层做换算。 [develop.realman-robotics](https://develop.realman-robotics.com/en/robot/ros2/rosInterfaces/)
- mode 预留了“力控制”等模式，你可以根据设备能力选择用不用。  
- blocking + timeout 一般在工业夹爪接口里都挺常见，方便上位机做简单逻辑。 [develop.realman-robotics](https://develop.realman-robotics.com/en/robot/ros2/rosInterfaces/)

***

## GripperState.idl（状态反馈）

```idl
module universal_gripper
{

// 夹爪整体状态
enum GripperStatus
{
    STATUS_UNKNOWN = 0,
    STATUS_IDLE    = 1,  // 空闲
    STATUS_MOVING  = 2,  // 正在运动
    STATUS_HOLDING = 3,  // 已抓稳（力达到/位置到位）
    STATUS_ERROR   = 4   // 错误状态
};

// 抓取结果/接触状态（可根据项目剪裁）
enum GraspResult
{
    GRASP_UNKNOWN      = 0,
    GRASP_IN_PROGRESS  = 1,
    GRASP_SUCCESS      = 2,
    GRASP_SLIPPED      = 3,
    GRASP_NO_CONTACT   = 4,
    GRASP_ABORTED      = 5,
    GRASP_FAILED_ERROR = 6
};

// 通用状态反馈
struct GripperState
{
    // 当前开度（0.0~1.0）
    float position;

    // 当前估计力（0.0~1.0，如果设备支持；不支持时可固定为 0 或 NAN）
    float force;

    // 当前估计速度（0.0~1.0，可选）
    float speed;

    // 当前高层状态（空闲/运动/持物/错误）
    GripperStatus status;

    // 当前最近一次抓取结果（如果有概念“当前抓取”）
    GraspResult grasp_result;

    // 设备自身错误码（0 表示无错误，其它值含义由设备自己定义）
    long error_code;

    // 设备返回的额外状态信息（预留，可空串）
    string<128> message;
};

};
```

设计要点：

- GripperStatus 和 GraspResult 拆开，可以支持“正在抓的这一次结果”和“当前整体运行状态”同时表达。  
- error_code + message 给厂商/具体设备扩展错误信息留了口子，不会把通用枚举搞得太复杂。

***

## GripperCapabilities.idl（能力描述）

```idl
module universal_gripper
{

// 单个关节/指的能力（如需要可以以后扩展）
struct JointCapability
{
    // 逻辑名字，比如 "finger_left"、"finger_right"，也可以用空串
    string<32> name;

    // 归一化位移范围（一般是 0.0~1.0，但保留可扩展空间）
    float min_position;
    float max_position;

    // 最大速度（归一化）
    float max_speed;

    // 最大力（归一化）
    float max_force;
};

// 夹爪整体能力
struct GripperCapabilities
{
    // 是否支持位置控制
    boolean has_position_control;

    // 是否支持力控制（max_force 字段是否有效）
    boolean has_force_control;

    // 是否支持速度设定（max_speed 字段是否有效）
    boolean has_speed_control;

    // 是否是多指/多关节（true 则 joints 数组可能有多个元素）
    boolean has_multi_joint;

    // 该夹爪总的关节/指的数量（与 joints 长度对应）
    long joint_count;

    // 每个关节/指的范围与能力
    sequence<JointCapability, 8> joints;

    // 是否具备抓取状态判断（滑落检测等）
    boolean has_grasp_detection;

    // 是否具备触觉/力矩传感器（高阶特性，可在后续扩展更多细节）
    boolean has_tactile_sensing;

    // 设备/厂商标识（比如 "Robotiq_2F85"）
    string<64> model_name;

    // 固件/软件版本信息（可选）
    string<32> firmware_version;
};

};
```

设计要点：

- has_xxx_control 这几个 boolean 决定上层要不要用 `GripperCommand` 里的某些字段。  
- joints 用一个小的上限（这里例子给了 8），够大部分平行夹爪、多指手；你后面可以改成更大或者无限长的 sequence。  
- model_name / firmware_version 对调试和日志很有用，强烈建议保留。

***

## 怎么用这三类 IDL 组合你的系统

典型流程可以是：

1. 机器人启动时，每个夹爪驱动节点发布一条（或定期发布）`GripperCapabilities`，上层根据能力决定 UI 和策略。  
2. 运动规划/任务层发布 `GripperCommand` 到统一 Topic（或 action/service），不同夹爪由自己的适配层翻译成底层协议。  
3. 适配层周期性发布 `GripperState`，上层根据 position/force/status/grasp_result 来决定是否继续任务、重试等。 [control.ros](https://control.ros.org/rolling/doc/ros2_control/hardware_interface/doc/hardware_interface_types_userdoc.html)

你可以先用这套“极简版”在一两种夹爪上跑通，然后再根据真实项目需求往里加字段或拆分更细的枚举。如果你愿意，把你现在手里一两种具体夹爪的能力列一下，我可以帮你一起对照这三个 IDL 看看哪里需要微调。
