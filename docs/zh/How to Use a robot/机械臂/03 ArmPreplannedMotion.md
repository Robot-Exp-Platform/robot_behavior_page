# 03 机械臂预规划接口

TODO 将 `ArmPreplannedMotion` 接口整理为更详细的文档

=== "Python"
    ```python
    class ArmPreplannedMotion:
        """
        ArmPreplannedMotion

        Interface for preplanned motion of the robot arm, supporting absolute/relative/inertial moves and path operations.

        机械臂预规划运动接口，支持绝对/相对/惯性移动及路径操作。
        """
        def move_to(self, target: 'MotionType') -> None:
            """
            Move to the target position.

            移动到目标位置。
            """
            ...
        def move_to_async(self, target: 'MotionType') -> None:
            """
            Move to the target position asynchronously.

            异步移动到目标位置。
            """
            ...
        def move_rel(self, target: 'MotionType') -> None:
            """
            Move to the target position relative to the current pose.

            相对当前位置移动到目标位置。
            """
            ...
        def move_rel_async(self, target: 'MotionType') -> None:
            """
            Move to the target position asynchronously in relative mode.

            以相对模式异步移动到目标位置。
            """
            ...
        def move_int(self, target: 'MotionType') -> None:
            """
            Move to the target position in inertial coordinate system.

            在惯性坐标系下移动到目标位置。
            """
            ...
        def move_int_async(self, target: 'MotionType') -> None:
            """
            Move to the target position asynchronously in inertial coordinate system.

            在惯性坐标系下异步移动到目标位置。
            """
            ...
        def move_path(self, path: list['MotionType']) -> None:
            """
            Move along a given path.

            按给定路径移动。
            """
            ...
        def move_path_async(self, path: list['MotionType']) -> None:
            """
            Move along a given path asynchronously.

            异步按给定路径移动。
            """
            ...
        def move_path_prepare(self, path: list['MotionType']) -> None:
            """
            Prepare for path motion.

            准备路径运动。
            """
            ...
        def move_path_start(self, start: 'MotionType') -> None:
            """
            Start path motion from a given start point.

            从指定起点开始路径运动。
            """
            ...

    class ArmPreplannedMotionImpl:
        """
        ArmPreplannedMotionImpl

        Implementation interface for preplanned motion, providing joint and Cartesian space motion methods.

        机械臂预规划运动实现接口，提供关节空间和笛卡尔空间的运动方法。
        """
        def move_joint(self, target: list[float]) -> None:
            """
            Move in joint space to the target.

            关节空间移动到目标。
            """
            ...
        def move_joint_async(self, target: list[float]) -> None:
            """
            Move in joint space to the target asynchronously.

            关节空间异步移动到目标。
            """
            ...
        def move_cartesian(self, target: 'Pose') -> None:
            """
            Move in Cartesian space to the target pose.

            笛卡尔空间移动到目标位姿。
            """
            ...
        def move_cartesian_async(self, target: 'Pose') -> None:
            """
            Move in Cartesian space to the target pose asynchronously.

            笛卡尔空间异步移动到目标位姿。
            """
            ...

    class ArmPreplannedMotionExt:
        """
        ArmPreplannedMotionExt

        Extension interface for preplanned motion, supporting relative, path, and various linear moves.

        机械臂预规划运动扩展接口，支持相对、路径及多种直线运动方式。
        """
        def move_joint_rel(self, target: list[float]) -> None:
            """
            Move relatively in joint space.

            关节空间相对移动。
            """
            ...
        def move_joint_rel_async(self, target: list[float]) -> None:
            """
            Move relatively in joint space asynchronously.

            关节空间异步相对移动。
            """
            ...
        def move_joint_path(self, path: list[list[float]]) -> None:
            """
            Move along a joint space path.

            按关节空间路径移动。
            """
            ...
        def move_cartesian_rel(self, target: 'Pose') -> None:
            """
            Move relatively in Cartesian space.

            笛卡尔空间相对移动。
            """
            ...
        def move_cartesian_rel_async(self, target: 'Pose') -> None:
            """
            Move relatively in Cartesian space asynchronously.

            笛卡尔空间异步相对移动。
            """
            ...
        def move_cartesian_int(self, target: 'Pose') -> None:
            """
            Move in inertial Cartesian space.

            惯性坐标系下笛卡尔空间移动。
            """
            ...
        def move_cartesian_int_async(self, target: 'Pose') -> None:
            """
            Move in inertial Cartesian space asynchronously.

            惯性坐标系下笛卡尔空间异步移动。
            """
            ...
        def move_cartesian_path(self, path: list['Pose']) -> None:
            """
            Move along a Cartesian space path.

            按笛卡尔空间路径移动。
            """
            ...
        def move_linear_with_euler(self, pose: list[float]) -> None:
            """
            Move linearly using Euler angles.

            以欧拉角方式直线移动。
            """
            ...
        def move_linear_with_euler_async(self, pose: list[float]) -> None:
            """
            Move linearly using Euler angles asynchronously.

            以欧拉角方式异步直线移动。
            """
            ...
        def move_linear_with_euler_rel(self, pose: list[float]) -> None:
            """
            Move linearly and relatively using Euler angles.

            以欧拉角方式相对直线移动。
            """
            ...
        def move_linear_with_euler_rel_async(self, pose: list[float]) -> None:
            """
            Move linearly and relatively using Euler angles asynchronously.

            以欧拉角方式异步相对直线移动。
            """
            ...
        def move_linear_with_euler_int(self, pose: list[float]) -> None:
            """
            Move linearly in inertial coordinates using Euler angles.

            以欧拉角方式惯性直线移动。
            """
            ...
        def move_linear_with_euler_int_async(self, pose: list[float]) -> None:
            """
            Move linearly in inertial coordinates using Euler angles asynchronously.

            以欧拉角方式惯性异步直线移动。
            """
            ...
        def move_linear_with_quat(self, target) -> None:
            """
            Move linearly using quaternion.

            以四元数方式直线移动。
            """
            ...
        def move_linear_with_quat_async(self, target) -> None:
            """
            Move linearly using quaternion asynchronously.

            以四元数方式异步直线移动。
            """
            ...
        def move_linear_with_quat_rel(self, target) -> None:
            """
            Move linearly and relatively using quaternion.

            以四元数方式相对直线移动。
            """
            ...
        def move_linear_with_quat_rel_async(self, target) -> None:
            """
            Move linearly and relatively using quaternion asynchronously.

            以四元数方式异步相对直线移动。
            """
            ...
        def move_linear_with_quat_int(self, target) -> None:
            """
            Move linearly in inertial coordinates using quaternion.

            以四元数方式惯性直线移动。
            """
            ...
        def move_linear_with_quat_int_async(self, target) -> None:
            """
            Move linearly in inertial coordinates using quaternion asynchronously.

            以四元数方式惯性异步直线移动。
            """
            ...
        def move_linear_with_homo(self, target: list[float]) -> None:
            """
            Move linearly using homogeneous matrix.

            以齐次矩阵方式直线移动。
            """
            ...
        def move_linear_with_homo_async(self, target: list[float]) -> None:
            """
            Move linearly using homogeneous matrix asynchronously.

            以齐次矩阵方式异步直线移动。
            """
            ...
        def move_linear_with_homo_rel(self, target: list[float]) -> None:
            """
            Move linearly and relatively using homogeneous matrix.

            以齐次矩阵方式相对直线移动。
            """
            ...
        def move_linear_with_homo_rel_async(self, target: list[float]) -> None:
            """
            Move linearly and relatively using homogeneous matrix asynchronously.

            以齐次矩阵方式异步相对直线移动。
            """
            ...
        def move_linear_with_homo_int(self, target: list[float]) -> None:
            """
            Move linearly in inertial coordinates using homogeneous matrix.

            以齐次矩阵方式惯性直线移动。
            """
            ...
        def move_linear_with_homo_int_async(self, target: list[float]) -> None:
            """
            Move linearly in inertial coordinates using homogeneous matrix asynchronously.

            以齐次矩阵方式惯性异步直线移动。
            """
            ...
        def move_path_prepare_from_file(self, path: str) -> None:
            """
            Prepare path motion from file.

            从文件准备路径运动。
            """
            ...
        def move_path_from_file(self, path: str) -> None:
            """
            Execute path motion from file.

            从文件执行路径运动。
            """
            ...
    ```

=== "Rust"
    ```rust
    pub trait ArmPreplannedMotion<const N: usize>: ArmPreplannedMotionImpl<N> {
        fn move_to(&mut self, target: MotionType<N>) -> RobotResult<()> {
            match target {
                MotionType::Joint(target) => self.move_joint(&target),
                MotionType::Cartesian(target) => self.move_cartesian(&target),
                _ => Err(RobotException::ConflictingInstruction(
                    "ArmPreplannedMotion does not support this motion type".to_string(),
                )),
            }
        }
        fn move_to_async(&mut self, target: MotionType<N>) -> RobotResult<()> {
            match target {
                MotionType::Joint(target) => self.move_joint_async(&target),
                MotionType::Cartesian(target) => self.move_cartesian_async(&target),
                _ => Err(RobotException::ConflictingInstruction(
                    "ArmPreplannedMotion does not support this motion type".to_string(),
                )),
            }
        }
        fn move_rel(&mut self, target: MotionType<N>) -> RobotResult<()> {
            self.with_coord(Coord::Shot).move_to(target)
        }
        fn move_rel_async(&mut self, target: MotionType<N>) -> RobotResult<()> {
            self.with_coord(Coord::Shot).move_to_async(target)
        }
        fn move_int(&mut self, target: MotionType<N>) -> RobotResult<()> {
            self.with_coord(Coord::Interial).move_to(target)
        }
        fn move_int_async(&mut self, target: MotionType<N>) -> RobotResult<()> {
            self.with_coord(Coord::Interial).move_to_async(target)
        }

        fn move_path(&mut self, path: Vec<MotionType<N>>) -> RobotResult<()>;
        fn move_path_async(&mut self, path: Vec<MotionType<N>>) -> RobotResult<()>;
        fn move_path_prepare(&mut self, path: Vec<MotionType<N>>) -> RobotResult<()>;
        fn move_path_start(&mut self, start: MotionType<N>) -> RobotResult<()>;
    }

    pub trait ArmPreplannedMotionImpl<const N: usize>: ArmBehavior<N> {
        fn move_joint(&mut self, target: &[f64; N]) -> RobotResult<()>;
        fn move_joint_async(&mut self, target: &[f64; N]) -> RobotResult<()>;

        fn move_cartesian(&mut self, target: &Pose) -> RobotResult<()>;
        fn move_cartesian_async(&mut self, target: &Pose) -> RobotResult<()>;
    }

    pub trait ArmPreplannedMotionExt<const N: usize>: ArmPreplannedMotion<N> {
        fn move_joint_rel(&mut self, target: &[f64; N]) -> RobotResult<()> {
            self.with_coord(Coord::Shot).move_joint(target)
        }
        fn move_joint_rel_async(&mut self, target: &[f64; N]) -> RobotResult<()> {
            self.with_coord(Coord::Shot).move_joint_async(target)
        }
        fn move_joint_path(&mut self, path: Vec<[f64; N]>) -> RobotResult<()> {
            self.with_coord(Coord::Shot)
                .move_path(path.into_iter().map(MotionType::Joint).collect())
        }

        fn move_cartesian_rel(&mut self, target: &Pose) -> RobotResult<()> {
            self.with_coord(Coord::Shot).move_cartesian(target)
        }
        fn move_cartesian_rel_async(&mut self, target: &Pose) -> RobotResult<()> {
            self.with_coord(Coord::Shot).move_cartesian_async(target)
        }
        fn move_cartesian_int(&mut self, target: &Pose) -> RobotResult<()> {
            self.with_coord(Coord::Interial).move_cartesian(target)
        }
        fn move_cartesian_int_async(&mut self, target: &Pose) -> RobotResult<()> {
            self.with_coord(Coord::Interial)
                .move_cartesian_async(target)
        }
        fn move_cartesian_path(&mut self, path: Vec<Pose>) -> RobotResult<()> {
            self.with_coord(Coord::Shot)
                .move_path(path.into_iter().map(MotionType::Cartesian).collect())
        }

        fn move_linear_with_euler(&mut self, pose: [f64; 6]) -> RobotResult<()> {
            self.move_cartesian(&pose.into())
        }
        fn move_linear_with_euler_async(&mut self, pose: [f64; 6]) -> RobotResult<()> {
            self.move_cartesian_async(&pose.into())
        }
        fn move_linear_with_euler_rel(&mut self, pose: [f64; 6]) -> RobotResult<()> {
            self.with_coord(Coord::Shot).move_cartesian(&pose.into())
        }
        fn move_linear_with_euler_rel_async(&mut self, pose: [f64; 6]) -> RobotResult<()> {
            self.with_coord(Coord::Shot)
                .move_cartesian_async(&pose.into())
        }
        fn move_linear_with_euler_int(&mut self, pose: [f64; 6]) -> RobotResult<()> {
            self.with_coord(Coord::Interial)
                .move_cartesian(&pose.into())
        }
        fn move_linear_with_euler_int_async(&mut self, pose: [f64; 6]) -> RobotResult<()> {
            self.with_coord(Coord::Interial)
                .move_cartesian_async(&pose.into())
        }

        fn move_linear_with_quat(&mut self, target: &na::Isometry3<f64>) -> RobotResult<()> {
            self.move_cartesian(&Pose::Quat(*target))
        }
        fn move_linear_with_quat_async(&mut self, target: &na::Isometry3<f64>) -> RobotResult<()> {
            self.move_cartesian_async(&Pose::Quat(*target))
        }
        fn move_linear_with_quat_rel(&mut self, target: &na::Isometry3<f64>) -> RobotResult<()> {
            self.with_coord(Coord::Shot)
                .move_cartesian(&Pose::Quat(*target))
        }
        fn move_linear_with_quat_rel_async(&mut self, target: &na::Isometry3<f64>) -> RobotResult<()> {
            self.with_coord(Coord::Shot)
                .move_cartesian_async(&Pose::Quat(*target))
        }
        fn move_linear_with_quat_int(&mut self, target: &na::Isometry3<f64>) -> RobotResult<()> {
            self.with_coord(Coord::Interial)
                .move_cartesian(&Pose::Quat(*target))
        }
        fn move_linear_with_quat_int_async(&mut self, target: &na::Isometry3<f64>) -> RobotResult<()> {
            self.with_coord(Coord::Interial)
                .move_cartesian_async(&Pose::Quat(*target))
        }

        fn move_linear_with_homo(&mut self, target: [f64; 16]) -> RobotResult<()> {
            self.move_cartesian(&target.into())
        }
        fn move_linear_with_homo_async(&mut self, target: [f64; 16]) -> RobotResult<()> {
            self.move_cartesian_async(&target.into())
        }
        fn move_linear_with_homo_rel(&mut self, target: [f64; 16]) -> RobotResult<()> {
            self.with_coord(Coord::Shot).move_cartesian(&target.into())
        }
        fn move_linear_with_homo_rel_async(&mut self, target: [f64; 16]) -> RobotResult<()> {
            self.with_coord(Coord::Shot)
                .move_cartesian_async(&target.into())
        }
        fn move_linear_with_homo_int(&mut self, target: [f64; 16]) -> RobotResult<()> {
            self.with_coord(Coord::Interial)
                .move_cartesian(&target.into())
        }
        fn move_linear_with_homo_int_async(&mut self, target: [f64; 16]) -> RobotResult<()> {
            self.with_coord(Coord::Interial)
                .move_cartesian_async(&target.into())
        }

        fn move_path_prepare_from_file(&mut self, path: &str) -> RobotResult<()> {
            let file = File::open(path)?;
            let reader = BufReader::new(file);
            let path: Vec<MotionType<N>> = from_reader(reader).unwrap();
            self.move_path_prepare(path)
        }
        fn move_path_from_file(&mut self, path: &str) -> RobotResult<()> {
            let file = File::open(path)?;
            let reader = BufReader::new(file);
            let path: Vec<MotionType<N>> = from_reader(reader).unwrap();
            self.move_path(path)
        }
    }
    ```
