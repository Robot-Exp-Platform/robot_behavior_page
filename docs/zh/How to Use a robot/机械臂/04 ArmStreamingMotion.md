# 04 机械臂流式控制接口

流式控制接口用于运动目标在执行前未知，在执行过程中由上位机实时下发的场景。流式控制的核心是通过 `start_streaming` 获取一个流式控制句柄（Handle），然后通过句柄不断设置新的运动/控制目标。同时也提供了基于共享对象的方式，通过线程安全的共享变量来传递目标。

## 使用方式

流式控制有两种使用方式：

**方式一：句柄模式**

通过 `start_streaming()` 获取句柄，然后通过句柄下发运动目标。

=== "Rust"
    ```rust
    let mut handle = robot.start_streaming()?;
    for target in trajectory {
        handle.move_to(MotionType::Joint(target))?;
        sleep(Duration::from_millis(1));
    }
    robot.end_streaming()?;
    ```

=== "Python"
    ```python
    handle = robot.start_streaming()
    for target in trajectory:
        handle.move_to(MotionType.Joint(target))
        sleep(0.001)
    robot.end_streaming()
    ```

**方式二：共享对象模式**

通过 `move_to_target()` 等方法获取线程安全的共享对象，在其他线程中修改目标值。

## ArmStreamingMotion 接口

=== "Rust"
    ```rust
    pub trait ArmStreamingMotion<const N: usize>: Arm<N> {
        type Handle: ArmStreamingHandle<N>;

        fn start_streaming(&mut self) -> RobotResult<Self::Handle>;
        fn end_streaming(&mut self) -> RobotResult<()>;
        fn move_to_target(&mut self) -> Arc<Mutex<Option<MotionType<N>>>>;
        fn control_with_target(&mut self) -> Arc<Mutex<Option<ControlType<N>>>>;
    }
    ```

=== "Python"
    ```python
    class ArmStreamingMotion:
        def start_streaming(self) -> ArmStreamingHandle: ...
        def end_streaming(self) -> None: ...
        def move_to_target(self) -> object: ...
        def control_with_target(self) -> object: ...
    ```

## ArmStreamingMotionExt 扩展接口

扩展接口提供各种具体类型的共享对象访问：

=== "Rust"
    ```rust
    pub trait ArmStreamingMotionExt<const N: usize>: ArmStreamingMotion<N> {
        fn move_joint_target(&mut self) -> Arc<Mutex<Option<[f64; N]>>>;
        fn move_joint_vel_target(&mut self) -> Arc<Mutex<Option<[f64; N]>>>;
        fn move_joint_acc_target(&mut self) -> Arc<Mutex<Option<[f64; N]>>>;
        fn move_cartesian_target(&mut self) -> Arc<Mutex<Option<Pose>>>;
        fn move_cartesian_vel_target(&mut self) -> Arc<Mutex<Option<[f64; 6]>>>;
        fn move_cartesian_euler_target(&mut self) -> Arc<Mutex<Option<[f64; 6]>>>;
        fn move_cartesian_quat_target(&mut self) -> Arc<Mutex<Option<na::Isometry3<f64>>>>;
        fn move_cartesian_homo_target(&mut self) -> Arc<Mutex<Option<[f64; 16]>>>;
        fn control_tau_target(&mut self) -> Arc<Mutex<Option<[f64; N]>>>;
    }
    ```

=== "Python"
    ```python
    class ArmStreamingMotionExt:
        def move_joint_target(self) -> object: ...
        def move_joint_vel_target(self) -> object: ...
        def move_joint_acc_target(self) -> object: ...
        def move_cartesian_target(self) -> object: ...
        def move_cartesian_vel_target(self) -> object: ...
        def move_cartesian_euler_target(self) -> object: ...
        def move_cartesian_quat_target(self) -> object: ...
        def move_cartesian_homo_target(self) -> object: ...
        def control_tau_target(self) -> object: ...
    ```

## 流式控制句柄

`ArmStreamingHandle` 是流式控制的操作句柄，提供上一帧运动/控制目标的读取和新目标的设置：

=== "Rust"
    ```rust
    pub trait ArmStreamingHandle<const N: usize> {
        fn last_motion(&self) -> Option<MotionType<N>>;
        fn move_to(&mut self, target: MotionType<N>) -> RobotResult<()>;
        fn last_control(&self) -> Option<ControlType<N>>;
        fn control_with(&mut self, control: ControlType<N>) -> RobotResult<()>;
    }
    ```

=== "Python"
    ```python
    class ArmStreamingHandle:
        def last_motion(self) -> MotionType: ...
        def move_to(self, target: MotionType) -> None: ...
        def last_control(self) -> ControlType: ...
        def control_with(self, control: ControlType) -> None: ...
    ```
