# 05 机械臂闭包控制与阻抗控制接口

## 闭包控制接口

闭包控制（实时控制）接口用于运动目标在执行前未知，需要在执行过程中根据机器人当前状态和运行时间实时计算控制量的场景。用户提供一个闭包函数，驱动层在每个控制周期调用该闭包，根据返回值更新运动/控制目标。

闭包函数的签名为：接收当前机械臂状态 `ArmState` 和运行时长 `Duration`，返回一个元组 `(目标, 是否结束)`。当返回的 `finished` 为 `true` 时，实时控制终止。

### 使用样例

=== "Rust"
    ```rust
    // 关节空间正弦运动
    let q0 = robot.state()?.joint.unwrap();
    robot.move_joint_with_closure(move |state, duration| {
        let t = duration.as_secs_f64();
        let mut q = q0;
        q[0] += 0.1 * (2.0 * std::f64::consts::PI * t).sin();
        (q, t > 5.0)  // 5秒后停止
    })?;
    ```

=== "Python"
    ```python
    import math

    q0 = robot.state().joint
    def closure(state, duration):
        t = duration
        q = list(q0)
        q[0] += 0.1 * math.sin(2 * math.pi * t)
        return (q, t > 5.0)

    robot.move_joint_with_closure(closure)
    ```

### ArmRealtimeControl 接口

=== "Rust"
    ```rust
    pub trait ArmRealtimeControl<const N: usize>: Arm<N> {
        fn move_with_closure<FM>(&mut self, closure: FM) -> RobotResult<()>
        where
            FM: FnMut(ArmState<N>, Duration) -> (MotionType<N>, bool) + Send + 'static;

        fn control_with_closure<FC>(&mut self, closure: FC) -> RobotResult<()>
        where
            FC: FnMut(ArmState<N>, Duration) -> (ControlType<N>, bool) + Send + 'static;
    }
    ```

=== "Python"
    ```python
    class ArmRealtimeControl:
        def move_with_closure(
            self,
            closure: Callable[[ArmState, float], tuple[MotionType, bool]]
        ) -> None: ...

        def control_with_closure(
            self,
            closure: Callable[[ArmState, float], tuple[ControlType, bool]]
        ) -> None: ...
    ```

### ArmRealtimeControlExt 扩展接口

扩展接口为常见的控制模式提供了便捷的闭包包装：

=== "Rust"
    ```rust
    pub trait ArmRealtimeControlExt<const N: usize>: ArmRealtimeControl<N> {
        fn move_joint_with_closure<FM>(&mut self, closure: FM) -> RobotResult<()>
        where FM: FnMut(ArmState<N>, Duration) -> ([f64; N], bool) + Send + Sync + 'static;

        fn move_joint_vel_with_closure<FM>(&mut self, closure: FM) -> RobotResult<()>
        where FM: FnMut(ArmState<N>, Duration) -> ([f64; N], bool) + Send + Sync + 'static;

        fn move_cartesian_with_closure<FM>(&mut self, closure: FM) -> RobotResult<()>
        where FM: FnMut(ArmState<N>, Duration) -> (Pose, bool) + Send + Sync + 'static;

        fn move_cartesian_vel_with_closure<FM>(&mut self, closure: FM) -> RobotResult<()>
        where FM: FnMut(ArmState<N>, Duration) -> ([f64; 6], bool) + Send + Sync + 'static;

        fn move_cartesian_euler_with_closure<FM>(&mut self, closure: FM) -> RobotResult<()>
        where FM: FnMut(ArmState<N>, Duration) -> ([f64; 3], [f64; 3], bool) + Send + Sync + 'static;

        fn move_cartesian_quat_with_closure<FM>(&mut self, closure: FM) -> RobotResult<()>
        where FM: FnMut(ArmState<N>, Duration) -> (na::Isometry3<f64>, bool) + Send + Sync + 'static;

        fn move_cartesian_homo_with_closure<FM>(&mut self, closure: FM) -> RobotResult<()>
        where FM: FnMut(ArmState<N>, Duration) -> ([f64; 16], bool) + Send + Sync + 'static;
    }
    ```

=== "Python"
    ```python
    class ArmRealtimeControlExt:
        def move_joint_with_closure(
            self, closure: Callable[[ArmState, float], tuple[list[float], bool]]
        ) -> None: ...

        def move_joint_vel_with_closure(
            self, closure: Callable[[ArmState, float], tuple[list[float], bool]]
        ) -> None: ...

        def move_cartesian_with_closure(
            self, closure: Callable[[ArmState, float], tuple[Pose, bool]]
        ) -> None: ...

        def move_cartesian_vel_with_closure(
            self, closure: Callable[[ArmState, float], tuple[list[float], bool]]
        ) -> None: ...

        def move_cartesian_euler_with_closure(
            self, closure: Callable[[ArmState, float], tuple[list[float], list[float], bool]]
        ) -> None: ...

        def move_cartesian_quat_with_closure(
            self, closure: Callable[[ArmState, float], tuple[object, bool]]
        ) -> None: ...

        def move_cartesian_homo_with_closure(
            self, closure: Callable[[ArmState, float], tuple[list[float], bool]]
        ) -> None: ...
    ```

## 阻抗控制接口

`ArmImpedance` 特征提供了关节空间和笛卡尔空间的阻抗控制接口。阻抗控制通过设置刚度（stiffness）和阻尼（damping）参数来实现柔顺控制，适用于接触力控制、装配等场景。

阻抗控制返回一个句柄（Handle），用户可以通过句柄动态调整刚度、阻尼和目标位置。

### ArmImpedance 接口

=== "Rust"
    ```rust
    pub trait ArmImpedance<const N: usize> {
        /// 异步启动关节阻抗控制，返回控制句柄
        fn joint_impedance_async(
            &mut self,
            stiffness: &[f64; N],
            damping: &[f64; N],
        ) -> RobotResult<JointImpedanceHandle<N>>;

        /// 异步启动笛卡尔阻抗控制，返回控制句柄
        fn cartesian_impedance_async(
            &mut self,
            stiffness: (f64, f64),  // (平移刚度, 旋转刚度)
            damping: (f64, f64),    // (平移阻尼, 旋转阻尼)
        ) -> RobotResult<CartesianImpedanceHandle>;

        /// 启动关节阻抗控制，返回句柄和异步终止函数
        fn joint_impedance_control(
            &mut self,
            stiffness: &[f64; N],
            damping: &[f64; N],
        ) -> RobotResult<(
            JointImpedanceHandle<N>,
            impl FnMut() -> BoxFuture<'static, RobotResult<()>> + Send + 'static,
        )>;

        /// 启动笛卡尔阻抗控制，返回句柄和异步终止函数
        fn cartesian_impedance_control(
            &mut self,
            stiffness: (f64, f64),
            damping: (f64, f64),
        ) -> RobotResult<(
            CartesianImpedanceHandle,
            impl FnMut() -> BoxFuture<'static, RobotResult<()>> + Send + 'static,
        )>;
    }
    ```

### 阻抗控制句柄

关节阻抗控制句柄和笛卡尔阻抗控制句柄提供了动态调整控制参数的接口：

=== "Rust"
    ```rust
    // 关节阻抗句柄
    pub trait JointImpedance<const N: usize> {
        fn set_stiffness(&self, stiffness: [f64; N]);
        fn set_damping(&self, damping: [f64; N]);
        fn set_target(&self, target: Option<[f64; N]>);
        fn finish(&self);
    }

    // 笛卡尔阻抗句柄
    pub trait CartesianImpedance {
        fn set_stiffness(&self, stiffness: (f64, f64));
        fn set_damping(&self, damping: (f64, f64));
        fn set_target(&self, target: Option<Pose>);
        fn finish(&self);
    }
    ```

### 使用样例

=== "Rust"
    ```rust
    // 关节阻抗控制
    let stiffness = [100.0; 6];
    let damping = [10.0; 6];
    let handle = robot.joint_impedance_async(&stiffness, &damping)?;

    // 动态调整目标
    handle.set_target(Some([0.0, 0.5, 0.0, -1.0, 0.0, 1.5]));

    // 结束控制
    handle.finish();
    ```
