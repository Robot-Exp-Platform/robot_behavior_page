# 01 机械臂基础接口

机械臂基础行为接口，包括状态查询、负载与坐标系设置、运动参数设置等。

其中函数主要分为三类：状态读取接口、参数设置接口（前缀`set`）、行为附加参数接口（前缀`with`）。其中行为附加参数接口被推荐使用链式或管道式的调用方式：

=== "Rust"
    ```rust
    let arm = Arm::new();
    arm.with_coord("base")
       .with_speed(0.5)
       .move_to(target_position);
    ```

=== "Python"
    ```python
    arm = Arm()
    arm.with_coord("base") \
       .with_speed(0.5) \
       .move_to(target_position)
    ```

- `state`

获取机械臂当前状态。

=== "Rust"
    ```rust
    pub fn state(&self) -> ArmState
    ```

=== "Python"
    ```python
    def state(self) -> ArmState
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
    pub fn set_coord(&mut self, coord: &str) -> RobotResult<()>
    ```

=== "Python"
    ```python
    def set_coord(self, coord: str) -> None: ...
    ```

- `with_coord`
行为附加参数接口，设置下一个运动指令的坐标系。

=== "Rust"
    ```rust
    pub fn with_coord(&mut self, coord: &str) -> RobotResult<&mut Self>
    ```

=== "Python"
    ```python
    def with_coord(self, coord: str) -> 'Arm': ...
    ```

- `set_speed`
参数设置接口，设置机械臂的速度比。

参数 `speed` 的值通常在 0.0 到 1.0 之间，表示速度的百分比。

=== "Rust"
    ```rust
    pub fn set_speed(&mut self, speed: f64) -> RobotResult<()>
    ```

=== "Python"
    ```python
    def set_speed(self, speed: float) -> None: ...
    ```

- `with_speed`
行为附加参数接口，设置下一个运动指令的速度比。

=== "Rust"
    ```rust
    pub fn with_speed(&mut self, speed: f64) -> &mut Self
    ```

=== "Python"
    ```python
    def with_speed(self, speed: float) -> 'Arm': ...
    ```

- `with_velocity`
行为附加参数接口，设置下一个运动指令的关节速度。

=== "Rust"
    ```rust
    pub fn with_velocity(&mut self, joint_vel: [f64; N]) -> &mut Self
    ```

=== "Python"
    ```python
    def with_velocity(self, joint_vel: list[float]) -> 'Arm': ...
    ```

- `with_acceleration`
行为附加参数接口，设置下一个运动指令的关节加速度。

=== "Rust"
    ```rust
    pub fn with_acceleration(&mut self, joint_acc: [f64; N]) -> &mut Self
    ```

=== "Python"
    ```python
    def with_acceleration(self, joint_acc: list[float]) -> 'Arm': ...
    ```

- `with_jerk`
行为附加参数接口，设置下一个运动指令的关节加加速度。

=== "Rust"
    ```rust
    pub fn with_jerk(&mut self, joint_jerk: [f64; N]) -> &mut Self
    ```

=== "Python"
    ```python
    def with_jerk(self, joint_jerk: list[float]) -> 'Arm': ...
    ```

- `with_cartesian_velocity`
行为附加参数接口，设置下一个运动指令的笛卡尔速度。

=== "Rust"
    ```rust
    pub fn with_cartesian_velocity(&mut self, cartesian_vel: f64) -> &mut Self
    ```

=== "Python"
    ```python
    def with_cartesian_velocity(self, cartesian_vel: float) -> 'Arm': ...
    ```

- `with_cartesian_acceleration`
行为附加参数接口，设置下一个运动指令的笛卡尔加速度。

=== "Rust"
    ```rust
    pub fn with_cartesian_acceleration(&mut self, cartesian_acc: f64) -> &mut Self
    ```

=== "Python"
    ```python
    def with_cartesian_acceleration(self, cartesian_acc: float) -> 'Arm': ...
    ```

- `with_cartesian_jerk`
行为附加参数接口，设置下一个运动指令的笛卡尔加加速度。

=== "Rust"
    ```rust
    pub fn with_cartesian_jerk(&mut self, cartesian_jerk: f64) -> &mut Self
    ```

=== "Python"
    ```python
    def with_cartesian_jerk(self, cartesian_jerk: float) -> 'Arm': ...
    ```

## 机械臂状态
