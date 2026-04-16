# Jaka 节卡

[libjaka-rs](https://github.com/Robot-Exp-Platform/libjaka-rs)

节卡机器人被实例化为 `JakaRobot`，使用时需要根据具体型号创建对应的类型别名实例。所有型号均为 6 自由度。

=== "Rust"
    ```rust
    use libjaka::JakaZu5;

    let mut robot = JakaZu5::new("172.16.0.10");
    ```
    [libjaka-rs](https://crates.io/crates/libjaka-rs)
=== "Python"
    ```python
    from libjaka import JakaMini2

    robot = JakaMini2("172.16.0.10")
    ```
    [libjaka-python](https://pypi.org/project/libjaka/)

## 支持的型号

每个型号是 `JakaRobot<T, 6>` 的类型别名，区别在于 DH 参数和运动学限位。

| 系列 | 型号 | 类型名 |
|------|------|--------|
| Zu 系列 | Zu3/Zu5/Zu7/Zu12/Zu18/Zu20/Zu30 | `JakaZu3` ~ `JakaZu30` |
| A 系列 | A5/A12/A20 | `JakaA5` / `JakaA12` / `JakaA20` |
| A-L 系列 | A12-L | `JakaA12L` |
| Pro 系列 | Pro5/Pro7/Pro12/Pro16/Pro18 | `JakaPro5` ~ `JakaPro18` |
| S 系列 | S5/S7/S12 | `JakaS5` / `JakaS7` / `JakaS12` |
| MiniCobo | Mini2 | `JakaMini2` |

!!! note
    目前 Python 绑定仅导出了 `JakaMini2`，其余型号可通过 Rust 使用。

## 已实现的通用接口

| 特征 | 说明 |
|------|------|
| `Robot` | 基础状态机（init→上电, enable→使能, disable, shutdown→断电, stop, read_state） |
| `Realtime` | 标记为支持实时控制 |
| `Arm<6>` | 机械臂基础接口（state/set_load/set_coord/set_scale/with_*） |
| `ArmParam<6>` | 关节参数常量（各型号不同） |
| `ArmForwardKinematics<6>` | DH 参数与正运动学 |
| `ArmPreplannedMotion<6>` | 预规划运动（move_joint/move_cartesian） |
| `ArmPreplannedPath<6>` | 路径运动（move_traj/move_waypoints） |
| `ArmStreamingMotion<6>` | 流式控制（ServoJ/ServoP 模式） |
| `ArmRealtimeControl<6>` | 闭包实时控制 |
| `ArmRealtimeControlExt<6>` | 闭包控制扩展 |

### 连接说明

`JakaRobot::new(ip)` 会自动：

1. 建立状态监听连接（端口 10000），后台线程持续读取机器人状态
2. 建立命令连接（端口 10001），用于发送控制指令
3. 设置默认速度比例为 5%（`set_scale(0.05)`）

### 单位说明

驱动内部自动进行单位转换：

- **关节角度**：用户侧使用弧度（rad），驱动自动转换为度（deg）与机器人通信
- **笛卡尔位置**：用户侧使用米（m），驱动自动转换为毫米（mm）与机器人通信
- **旋转角度**：用户侧使用弧度（rad），驱动自动转换为度（deg）

### 流式控制说明

节卡的流式控制基于 ServoJ（关节伺服）和 ServoP（笛卡尔伺服）指令实现，控制频率为 125 Hz。

=== "Rust"
    ```rust
    let mut handle = robot.start_streaming()?;

    // 通过句柄下发关节目标
    handle.move_to(MotionType::Joint([0.0; 6]))?;

    robot.end_streaming()?;
    ```

### 路径运动说明

`move_waypoints` 使用 Ruckig 在线轨迹生成器对路点进行时间最优的轨迹规划（S 曲线），生成密集的关节空间轨迹后通过 ServoJ 执行。目前仅支持关节空间路点。

=== "Rust"
    ```rust
    let waypoints = vec![
        MotionType::Joint([0.0, 0.5, 0.0, -1.0, 0.0, 1.5]),
        MotionType::Joint([0.5, 0.5, 0.0, -1.0, 0.0, 1.0]),
        MotionType::Joint([0.0, 0.0, 0.0, 0.0, 0.0, 0.0]),
    ];
    robot.move_waypoints(waypoints)?;
    ```

## 节卡特有接口

以下接口不属于 `robot_behavior` 通用特征，是节卡驱动独有的功能。

### 工具端电压输出

控制工具端的电压输出（支持 12V 和 24V 模式）。

=== "Rust"
    ```rust
    use libjaka::types::{TioVout, TioVoutMode};

    // 开启 24V 输出
    robot.set_tio_vout(TioVout::Enable(TioVoutMode::V24V))?;

    // 查询当前状态
    let vout = robot.get_tio_vout()?;

    // 关闭输出
    robot.set_tio_vout(TioVout::Disable)?;
    ```

### 底层指令访问

通过 `robot_impl` 字段可以直接访问底层 RPC 指令，适用于需要精细控制的场景：

=== "Rust"
    ```rust
    // 点动控制
    robot.robot_impl._jog(JogData::Mode0 { ... })?;

    // 碰撞灵敏度
    robot.robot_impl._set_clsn_sensitivity(data)?;
    let sensitivity = robot.robot_impl._get_clsn_sensitivity()?;

    // 正/逆运动学
    let result = robot.robot_impl._kine_forward(data)?;
    let result = robot.robot_impl._kine_inverse(data)?;

    // IO 操作
    robot.robot_impl._set_digital_output(data)?;
    let din = robot.robot_impl._get_digital_input_status()?;
    robot.robot_impl._set_analog_output(data)?;

    // 工具/用户坐标系
    robot.robot_impl._set_tool_offsets(data)?;
    robot.robot_impl._set_tool_id(data)?;
    robot.robot_impl._set_user_offsets(data)?;
    robot.robot_impl._set_user_id(data)?;

    // 负载
    robot.robot_impl._set_payload(data)?;
    let payload = robot.robot_impl._get_payload()?;

    // 程序控制
    robot.robot_impl._load_program(data)?;
    robot.robot_impl._play_program()?;
    robot.robot_impl._pause_program()?;
    robot.robot_impl._resume_program()?;
    robot.robot_impl._stop_program()?;
    ```

### 原始状态读取

通过 `read_state()` 可获取节卡 `RobotState`，包含丰富的状态信息：

=== "Rust"
    ```rust
    let state = robot.read_state()?;
    ```

`RobotState` 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| `joint_actual_position` | `[f64; 9]` | 各关节实际位置（度） |
| `actual_position` | `[f64; 9]` | 笛卡尔实际位姿（mm + 度） |
| `din` / `dout` | `[bool; 64]` | 数字输入/输出 |
| `ain` / `aout` | `[f64; 2]` | 模拟输入/输出 |
| `tio_din` / `tio_dout` / `tio_ain` | - | 工具 IO |
| `task_state` | `TaskState` | 任务状态（PowerOff/PowerOn/Disable/Enable） |
| `task_mode` | `TaskMode` | 任务模式（Manual/Auto/Guiding） |
| `interp_mode` | `InterpState` | 插补状态（Idle/Loading/Pause/Running） |
| `enabled` / `paused` / `protective_stop` / `emergency_stop` | `bool` | 各状态标志 |
