# 01 机械臂基础接口

机械臂基础行为接口，包括状态查询、负载与坐标系设置、运动参数设置等。

其中函数主要分为三类：状态读取接口、参数设置接口（前缀`set`）、行为附加参数接口（前缀`with`）。其中行为附加参数接口被推荐使用链式或管道式的调用方式：

=== "Rust"
    ```rust
    let arm = Arm::new();
    arm.with_coord(Coord::Relative)
       .with_scale(0.5)
       .move_to(target_position);
    ```

=== "Python"
    ```python
    arm = Arm()
    arm.with_coord("Shot") \
       .with_speed(0.5) \
       .move_to(target_position)
    ```

## 坐标系 Coord

坐标系使用 `Coord` 枚举表示，定义了运动指令的参考坐标系：

=== "Rust"
    ```rust
    pub enum Coord {
        OCS,          // 默认坐标系（操作坐标系）
        Relative,     // 相对当前位姿
        Inertial,     // 惯性坐标系（与基坐标系方向一致，原点在当前位置）
        Other(Pose),  // 自定义坐标系
    }
    ```

=== "Python"
    ```python
    # Python 中通过字符串传入
    "OCS"       # 默认坐标系
    "Shot"      # 相对当前位姿
    "Inertial"  # 惯性坐标系
    ```

## 接口列表

- `state`

获取机械臂当前状态。

=== "Rust"
    ```rust
    pub fn state(&mut self) -> RobotResult<ArmState<N>>
    ```

=== "Python"
    ```python
    def state(self) -> ArmState: ...
    ```

- `set_load`
参数设置接口，设置末端执行器的负载状态。

=== "Rust"
    ```rust
    pub fn set_load(&mut self, load: LoadState) -> RobotResult<()>
    ```

=== "Python"
    ```python
    def set_load(self, load: LoadState) -> None: ...
    ```

- `set_coord`
参数设置接口，设置机械臂的坐标系。

=== "Rust"
    ```rust
    pub fn set_coord(&mut self, coord: Coord) -> RobotResult<()>
    ```

=== "Python"
    ```python
    def set_coord(self, coord: str) -> None: ...
    ```

- `with_coord`
行为附加参数接口，设置下一个运动指令的坐标系。

=== "Rust"
    ```rust
    pub fn with_coord(&mut self, coord: Coord) -> &mut Self
    ```

=== "Python"
    ```python
    def with_coord(self, coord: str) -> 'Arm': ...
    ```

- `set_scale`
参数设置接口，设置机械臂的速度比。

参数 `scale` 的值通常在 0.0 到 1.0 之间，表示速度的百分比。

=== "Rust"
    ```rust
    pub fn set_scale(&mut self, scale: f64) -> RobotResult<()>
    ```

=== "Python"
    ```python
    def set_speed(self, speed: float) -> None: ...
    ```

- `with_scale`
行为附加参数接口，设置下一个运动指令的速度比。

=== "Rust"
    ```rust
    pub fn with_scale(&mut self, scale: f64) -> &mut Self
    ```

=== "Python"
    ```python
    def with_speed(self, speed: float) -> 'Arm': ...
    ```

- `with_velocity`
行为附加参数接口，设置下一个运动指令的关节速度。单位：[rad/s]

=== "Rust"
    ```rust
    pub fn with_velocity(&mut self, joint_vel: &[f64; N]) -> &mut Self
    ```

=== "Python"
    ```python
    def with_velocity(self, joint_vel: list[float]) -> 'Arm': ...
    ```

- `with_acceleration`
行为附加参数接口，设置下一个运动指令的关节加速度。单位：[rad/s²]

=== "Rust"
    ```rust
    pub fn with_acceleration(&mut self, joint_acc: &[f64; N]) -> &mut Self
    ```

=== "Python"
    ```python
    def with_acceleration(self, joint_acc: list[float]) -> 'Arm': ...
    ```

- `with_jerk`
行为附加参数接口，设置下一个运动指令的关节加加速度。单位：[rad/s³]

=== "Rust"
    ```rust
    pub fn with_jerk(&mut self, joint_jerk: &[f64; N]) -> &mut Self
    ```

=== "Python"
    ```python
    def with_jerk(self, joint_jerk: list[float]) -> 'Arm': ...
    ```

- `with_cartesian_velocity`
行为附加参数接口，设置下一个运动指令的笛卡尔速度。单位：[m/s]

=== "Rust"
    ```rust
    pub fn with_cartesian_velocity(&mut self, cartesian_vel: f64) -> &mut Self
    ```

=== "Python"
    ```python
    def with_cartesian_velocity(self, cartesian_vel: float) -> 'Arm': ...
    ```

- `with_cartesian_acceleration`
行为附加参数接口，设置下一个运动指令的笛卡尔加速度。单位：[m/s²]

=== "Rust"
    ```rust
    pub fn with_cartesian_acceleration(&mut self, cartesian_acc: f64) -> &mut Self
    ```

=== "Python"
    ```python
    def with_cartesian_acceleration(self, cartesian_acc: float) -> 'Arm': ...
    ```

- `with_cartesian_jerk`
行为附加参数接口，设置下一个运动指令的笛卡尔加加速度。单位：[m/s³]

=== "Rust"
    ```rust
    pub fn with_cartesian_jerk(&mut self, cartesian_jerk: f64) -> &mut Self
    ```

=== "Python"
    ```python
    def with_cartesian_jerk(self, cartesian_jerk: float) -> 'Arm': ...
    ```

- `with_rotation_velocity`
行为附加参数接口，设置下一个运动指令的旋转速度。单位：[rad/s]

=== "Rust"
    ```rust
    pub fn with_rotation_velocity(&mut self, rotation_vel: f64) -> &mut Self
    ```

- `with_rotation_acceleration`
行为附加参数接口，设置下一个运动指令的旋转加速度。单位：[rad/s²]

=== "Rust"
    ```rust
    pub fn with_rotation_acceleration(&mut self, rotation_acc: f64) -> &mut Self
    ```

- `with_rotation_jerk`
行为附加参数接口，设置下一个运动指令的旋转加加速度。单位：[rad/s³]

=== "Rust"
    ```rust
    pub fn with_rotation_jerk(&mut self, rotation_jerk: f64) -> &mut Self
    ```

## 机械臂状态

`ArmState` 描述了机械臂当前的完整状态：

=== "Rust"
    ```rust
    pub struct ArmState<const N: usize> {
        pub joint: Option<[f64; N]>,          // 关节位置 [rad]
        pub joint_vel: Option<[f64; N]>,      // 关节速度 [rad/s]
        pub joint_acc: Option<[f64; N]>,      // 关节加速度 [rad/s²]
        pub torque: Option<[f64; N]>,         // 关节力矩 [Nm]
        pub pose_o_to_ee: Option<Pose>,       // 基座到末端的位姿
        pub pose_ee_to_k: Option<Pose>,       // 末端到法兰的位姿
        pub cartesian_vel: Option<[f64; 6]>,  // 笛卡尔速度
        pub load: Option<LoadState>,          // 负载状态
    }
    ```

=== "Python"
    ```python
    class ArmState:
        joint: list[float] | None
        joint_vel: list[float] | None
        joint_acc: list[float] | None
        tau: list[float] | None
        pose_o_to_ee: Pose | None
        pose_ee_to_k: Pose | None
        cartesian_vel: list[float] | None
        load: LoadState | None
    ```
