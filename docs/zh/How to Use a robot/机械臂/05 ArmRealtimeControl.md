# 05 机械臂闭包控制接口

TODO 将 `ArmRealtimeControl` 接口整理为更详细的文档

=== "Python"
    ```python
    class ArmRealtimeControl:
        """
        ArmRealtimeControl

        Real-time control interface for robot arm, supports closure-based real-time motion and control.
    
        机械臂实时控制接口，支持基于闭包的实时运动与控制。
        """
        def move_with_closure(self, closure: callable[['ArmState', float], tuple['MotionType', bool]]) -> None:
            """
            Real-time motion using a closure, returns (motion target, finished).
    
            以闭包方式实时运动，返回(运动目标, 是否结束)。
            """
            ...
        def control_with_closure(self, closure: callable[['ArmState', float], tuple['ControlType', bool]]) -> None:
            """
            Real-time control using a closure, returns (control target, finished).
    
            以闭包方式实时控制，返回(控制目标, 是否结束)。
            """
            ...
    
    class ArmRealtimeControlExt:
        """
        ArmRealtimeControlExt
    
        Extension interface for real-time control, supports closure-based joint/Cartesian/velocity control.
    
        机械臂实时控制扩展接口，支持基于闭包的关节/笛卡尔/速度控制。
        """
        def move_joint_with_closure(self, closure: callable[['ArmState', float], tuple[list[float], bool]]) -> None:
            """
            Real-time joint space motion using a closure, returns (joint target, finished).
    
            以闭包方式实时关节空间运动，返回(关节目标, 是否结束)。
            """
            ...
        def move_joint_vel_with_closure(self, closure: callable[['ArmState', float], tuple[list[float], bool]]) -> None:
            """
            Real-time joint velocity motion using a closure, returns (joint velocity target, finished).
    
            以闭包方式实时关节速度运动，返回(关节速度目标, 是否结束)。
            """
            ...
        def move_cartesian_with_closure(self, closure: callable[['ArmState', float], tuple['Pose', bool]]) -> None:
            """
            Real-time Cartesian space motion using a closure, returns (pose target, finished).
    
            以闭包方式实时笛卡尔空间运动，返回(位姿目标, 是否结束)。
            """
            ...
        def move_cartesian_vel_with_closure(self, closure: callable[['ArmState', float], tuple[list[float], bool]]) -> None:
            """
            Real-time Cartesian velocity motion using a closure, returns (velocity target, finished).
    
            以闭包方式实时笛卡尔速度运动，返回(速度目标, 是否结束)。
            """
            ...
        def move_cartesian_euler_with_closure(self, closure: callable[['ArmState', float], tuple[list[float], list[float], bool]]) ->   None:
            """
            Real-time Euler angle motion using a closure, returns (translation, rotation, finished).
    
            以闭包方式实时欧拉角运动，返回(平移, 旋转, 是否结束)。
            """
            ...
        def move_cartesian_quat_with_closure(self, closure: callable[['ArmState', float], tuple[object, bool]]) -> None:
            """
            Real-time quaternion motion using a closure, returns (quaternion target, finished).
    
            以闭包方式实时四元数运动，返回(四元数目标, 是否结束)。
            """
            ...
        def move_cartesian_homo_with_closure(self, closure: callable[['ArmState', float], tuple[list[float], bool]]) -> None:
            """
            Real-time homogeneous matrix motion using a closure, returns (homogeneous matrix target, finished).
    
            以闭包方式实时齐次矩阵运动，返回(齐次矩阵目标, 是否结束)。
            """
    ```

=== "Rust"
    ```rust
    pub trait ArmRealtimeControl<const N: usize>: ArmBehavior<N> {
        fn move_with_closure<FM>(&mut self, closure: FM) -> RobotResult<()>
        where
            FM: FnMut(ArmState<N>, Duration) -> (MotionType<N>, bool) + Send + 'static;
        fn control_with_closure<FC>(&mut self, closure: FC) -> RobotResult<()>
        where
            FC: FnMut(ArmState<N>, Duration) -> (ControlType<N>, bool) + Send + 'static;
    }

    pub trait ArmRealtimeControlExt<const N: usize>: ArmRealtimeControl<N> {
        fn move_joint_with_closure<FM>(&mut self, mut closure: FM) -> RobotResult<()>
        where
            FM: FnMut(ArmState<N>, Duration) -> ([f64; N], bool) + Send + Sync + 'static,
        {
            self.move_with_closure(move |state, duration| {
                let (joint, finished) = closure(state, duration);
                (MotionType::Joint(joint), finished)
            })
        }
    
        fn move_joint_vel_with_closure<FM>(&mut self, mut closure: FM) -> RobotResult<()>
        where
            FM: FnMut(ArmState<N>, Duration) -> ([f64; N], bool) + Send + Sync + 'static,
        {
            self.move_with_closure(move |state, duration| {
                let (joint_vel, finished) = closure(state, duration);
                (MotionType::JointVel(joint_vel), finished)
            })
        }
    
        fn move_cartesian_with_closure<FM>(&mut self, mut closure: FM) -> RobotResult<()>
        where
            FM: FnMut(ArmState<N>, Duration) -> (Pose, bool) + Send + Sync + 'static,
        {
            self.move_with_closure(move |state, duration| {
                let (pose, finished) = closure(state, duration);
                (MotionType::Cartesian(pose), finished)
            })
        }
    
        fn move_cartesian_vel_with_closure<FM>(&mut self, mut closure: FM) -> RobotResult<()>
        where
            FM: FnMut(ArmState<N>, Duration) -> ([f64; 6], bool) + Send + Sync + 'static,
        {
            self.move_with_closure(move |state, duration| {
                let (vel, finished) = closure(state, duration);
                (MotionType::CartesianVel(vel), finished)
            })
        }
    
        fn move_cartesian_euler_with_closure<FM>(&mut self, mut closure: FM) -> RobotResult<()>
        where
            FM: FnMut(ArmState<N>, Duration) -> ([f64; 3], [f64; 3], bool) + Send + Sync + 'static,
        {
            self.move_cartesian_with_closure(move |state, duration| {
                let (tran, rot, finished) = closure(state, duration);
                (Pose::Euler(tran, rot), finished)
            })
        }
    
        fn move_cartesian_quat_with_closure<FM>(&mut self, mut closure: FM) -> RobotResult<()>
        where
            FM: FnMut(ArmState<N>, Duration) -> (na::Isometry3<f64>, bool) + Send + Sync + 'static,
        {
            self.move_cartesian_with_closure(move |state, duration| {
                let (quat, finished) = closure(state, duration);
                (Pose::Quat(quat), finished)
            })
        }
    
        fn move_cartesian_homo_with_closure<FM>(&mut self, mut closure: FM) -> RobotResult<()>
        where
            FM: FnMut(ArmState<N>, Duration) -> ([f64; 16], bool) + Send + Sync + 'static,
        {
            self.move_cartesian_with_closure(move |state, duration| {
                let (homo, finished) = closure(state, duration);
                (Pose::Homo(homo), finished)
            })
        }
    }
    ```
