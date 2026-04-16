# Franka

[franka-rust](https://github.com/Robot-Exp-Platform/franka-rust)

Franka 机器人（7 自由度）被实例化为 `FrankaRobot`。在使用时需要先创建一个 `FrankaRobot` 实例。

=== "Rust"
    ```rust
    use libfranka::FrankaRobot;

    let mut robot = FrankaRobot::new("172.16.0.2");
    ```
    [libfranka-rs](https://crates.io/crates/libfranka)
=== "Python"
    ```python
    from libfranka import FrankaRobot

    robot = FrankaRobot("172.16.0.2")
    ```
    [libfranka-python](https://pypi.org/project/libfranka/)

## 已实现的通用接口

Franka 驱动实现了以下 `robot_behavior` 通用特征：

| 特征 | 说明 |
|------|------|
| `Robot` | 基础状态机（init/shutdown/enable/disable/stop/pause/resume/emergency_stop/read_state） |
| `Realtime` | 标记为支持实时控制 |
| `Arm<7>` | 机械臂基础接口（state/set_load/set_coord/set_scale/with_*） |
| `ArmParam<7>` | 关节参数常量（限位、速度、加速度、力矩上限） |
| `ArmPreplannedMotion<7>` | 预规划运动（move_joint/move_cartesian） |
| `ArmPreplannedPath<7>` | 路径运动（move_traj/move_traj_async） |
| `ArmStreamingMotion<7>` | 流式控制（句柄模式） |
| `ArmRealtimeControl<7>` | 闭包实时控制（move_with_closure/control_with_closure） |
| `ArmRealtimeControlExt<7>` | 闭包控制扩展 |
| `ArmImpedance<7>` | 阻抗控制（关节/笛卡尔） |

## Franka 特有接口

以下是 Franka 机器人独有的接口，不属于 `robot_behavior` 通用特征。

### 碰撞行为设置

设置加速阶段和匀速阶段的力矩/力阈值。力/力矩在下阈值与上阈值之间时显示为接触（contact），超过上阈值时注册为碰撞（collision）并导致机器人停止。

=== "Rust"
    ```rust
    use libfranka::types::SetCollisionBehaviorData;

    // 使用默认值
    robot.set_collision_behavior(SetCollisionBehaviorData::default())?;

    // 使用统一值
    robot.set_collision_behavior(SetCollisionBehaviorData::from(30.0))?;

    // 自定义各项阈值
    robot.set_collision_behavior(SetCollisionBehaviorData {
        lower_torque_thresholds_acceleration: [20.0; 7],
        upper_torque_thresholds_acceleration: [30.0; 7],
        lower_torque_thresholds_nominal: [10.0; 7],
        upper_torque_thresholds_nominal: [20.0; 7],
        lower_force_thresholds_acceleration: [20.0; 6],
        upper_force_thresholds_acceleration: [30.0; 6],
        lower_force_thresholds_nominal: [10.0; 6],
        upper_force_thresholds_nominal: [20.0; 6],
    })?;
    ```

### 关节阻抗设置

设置内部控制器各关节的阻抗参数（刚度）。

=== "Rust"
    ```rust
    use libfranka::types::SetJointImpedanceData;

    // 默认刚度: [3000, 3000, 3000, 2500, 2500, 2000, 2000]
    robot.set_joint_impedance(SetJointImpedanceData::default())?;

    // 自定义
    robot.set_joint_impedance(SetJointImpedanceData::from([3000.0; 7]))?;
    ```

### 笛卡尔阻抗设置

设置内部控制器笛卡尔空间的阻抗参数（前 3 个为平移刚度，后 3 个为旋转刚度）。

=== "Rust"
    ```rust
    use libfranka::types::SetCartesianImpedanceData;

    // 默认刚度: [3000, 3000, 3000, 300, 300, 300]
    robot.set_cartesian_impedance(SetCartesianImpedanceData::default())?;
    ```

### 默认行为

一键设置碰撞行为、关节阻抗和笛卡尔阻抗为默认值。

=== "Rust"
    ```rust
    robot.set_default_behavior()?;
    ```

### 引导模式

锁定或解锁引导模式下 (x, y, z, roll, pitch, yaw) 各方向的运动。

=== "Rust"
    ```rust
    use libfranka::types::SetGuidingModeData;

    robot.set_guiding_mode(SetGuidingModeData {
        guiding_mode: [true, true, true, true, true, true],
        nullspace: false,
    })?;
    ```

### 坐标系变换

设置末端执行器到刚度坐标系（K 帧）以及法兰到末端执行器（NE→EE）的变换矩阵。

=== "Rust"
    ```rust
    // 设置 EE 到 K 帧变换（4×4 列主序齐次矩阵）
    robot.set_ee_to_k(SetEEToKData { ee_to_k: [/*16 个 f64*/] })?;

    // 设置 NE 到 EE 变换
    robot.set_ne_to_ee(SetNEToEEData { ne_to_ee: [/* 16 个 f64 */] })?;
    ```

### 动力学模型

获取 Franka 动力学模型，用于计算质量矩阵、科里奥利力等。首次调用会自动从控制器下载模型库文件。

=== "Rust"
    ```rust
    // 自动下载并加载模型
    let model = robot.model()?;

    // 从指定路径加载模型
    let model = robot.model_from_path(Path::new("path/to/model.so"))?;
    ```

### 读取原始状态

读取 Franka 原始的 `RobotState`，包含比通用 `ArmState` 更丰富的信息。

=== "Rust"
    ```rust
    let state = robot.read_franka_state()?;

    // 访问原始数据
    println!("关节位置: {:?}", state.q);
    println!("关节速度: {:?}", state.dq);
    println!("关节力矩: {:?}", state.tau_j);
    println!("外部力矩估计: {:?}", state.tau_ext_hat_filtered);
    println!("EE 位姿: {:?}", state.pose_o_to_ee);
    println!("控制命令成功率: {}", state.control_command_success_rate);
    println!("机器人模式: {:?}", state.robot_mode);
    ```

`RobotState` 主要字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| `q` / `q_d` | `[f64; 7]` | 关节位置（测量/期望） |
| `dq` / `dq_d` / `ddq_d` | `[f64; 7]` | 关节速度/加速度 |
| `tau_j` / `tau_j_d` / `dtau_j` | `[f64; 7]` | 关节力矩（测量/期望/导数） |
| `tau_ext_hat_filtered` | `[f64; 7]` | 外部力矩估计（经滤波） |
| `pose_o_to_ee` / `pose_o_to_ee_d` | `[f64; 16]` | EE 位姿（基坐标系，4×4 列主序） |
| `force_ext_in_o` / `force_ext_in_k` | `[f64; 6]` | 外力估计（基坐标系/刚度坐标系） |
| `joint_contact` / `joint_collision` | `[f64; 7]` | 关节接触/碰撞等级 |
| `cartesian_contact` / `cartesian_collision` | `[f64; 6]` | 笛卡尔接触/碰撞等级 |
| `theta` / `dtheta` | `[f64; 7]` | 电机侧位置/速度 |
| `m_ee` / `x_ee` / `i_ee` | - | EE 质量/质心/惯量 |
| `m_load` / `x_load` / `i_load` | - | 负载质量/质心/惯量 |
| `elbow` | `[f64; 2]` | 肘部配置 |
| `control_command_success_rate` | `f64` | 控制命令成功率 |
| `robot_mode` | `RobotMode` | 机器人模式 |

`RobotMode` 枚举值：`Idle`、`Move`、`Guiding`、`Reflex`、`UserStopped`、`AutomaticErrorRecovery`。

## 夹爪接口

Franka 配套的 `FrankaGripper` 提供了独立的夹爪控制接口。

=== "Rust"
    ```rust
    use libfranka::FrankaGripper;

    let mut gripper = FrankaGripper::new("172.16.0.2");
    ```

### 夹爪方法

| 方法 | 签名 | 说明 |
|------|------|------|
| `homing` | `() -> RobotResult<bool>` | 归位校准 |
| `grasp` | `(width, speed, force) -> RobotResult<bool>` | 抓取（指定宽度 m、速度 m/s、力 N） |
| `move_gripper` | `(width, speed) -> RobotResult<bool>` | 移动到指定宽度 |
| `stop` | `() -> RobotResult<bool>` | 停止当前动作 |
| `read_state` | `() -> RobotResult<GripperState>` | 读取夹爪状态 |

=== "Rust"
    ```rust
    // 归位
    gripper.homing()?;

    // 移动到指定宽度（0.04m = 4cm，速度 0.1 m/s）
    gripper.move_gripper(0.04, 0.1)?;

    // 抓取（宽度 0.02m，速度 0.1 m/s，力 60N）
    gripper.grasp(0.02, 0.1, 60.0)?;

    // 读取状态
    let state = gripper.read_state()?;

    // 停止
    gripper.stop()?;
    ```
