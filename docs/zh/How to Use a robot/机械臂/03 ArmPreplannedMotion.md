# 03 机械臂预规划接口

## 接口组成

预规划类接口要求在动作执行前就已知运动目标和轨迹。其本质上为两种类型的运动 `move_to` 和 `move_traj`/`move_waypoints`，其他指令都是在此基础上扩展的。拓展有两种方式：使用自带的拓展指令如 `move_joint_rel` 等，或者使用机械臂[基础指令](./01%20ArmBehavior.md)中的 `set_` 或 `with_` 指令修饰。

当前预规划接口由三个特征组成：

- `ArmPreplannedMotion` - 单点运动（`move_joint`、`move_cartesian` 及其 `move_to` / `move_rel` / `move_int` 等衍生）
- `ArmPreplannedPath` - 路径/轨迹运动（`move_traj`、`move_waypoints` 及其准备/启动接口）
- `ArmPreplannedMotionExt` - 扩展接口（各种便捷函数，自动由上述两个特征派生）

例如：以下这两种语法具有相同的功能，都表示在笛卡尔空间中相对于当前位置位移。

=== "Rust"
    ```rust
    // 使用拓展指令
    robot.move_cartesian_rel(&target_pose);

    // 使用基础指令修饰
    robot.with_coord(Coord::Relative)
        .move_cartesian(&target_pose);
    ```

=== "Python"
    ```python
    # 使用拓展指令
    robot.move_cartesian_rel(target_pose)

    # 使用基础指令修饰
    robot.with_coord("Shot") \
        .move_cartesian(target_pose)
    ```

我们将拓展指令涉及到的前后缀进行整理。

- `joint` 在关节空间中运动
- `cartesian` 在笛卡尔空间中运动
- `rel` 相对于当前位姿运动
- `int` 相对于当前惯性坐标系运动（和基坐标系方向一致，原点位置在当前位置）
- `async` 异步运动（机器人运动不会阻塞代码运行，可以多机器人一起动，但是要注意在机器人运动完成前不能下发其他运动指令，最好用 `is_moving()` 函数来判断）

所有拓展指令的命名都是上述前后缀的组合。如要求机器人在关节空间中直线运动到某处，并不阻塞电脑程序运行，则可以使用：

=== "Rust"
    ```rust
    robot.move_joint_async(&target_joint);
    ```

=== "Python"
    ```python
    robot.move_joint_async(target_joint)
    ```

全部可用的拓展指令见本页最后

## 使用样例

在完成机器人实例化并上使能后即可使用运动接口，以下是一些使用样例：

关节空间运动到指定位置(非阻塞)

=== "Rust"
    ```rust
    let target_joint = [0.0, 0.5, 0.0, -1.0, 0.0, 1.5];
    robot.move_joint_async(&target_joint)?;

    robot.waiting_for_finish()?;
    ```

=== "Python"
    ```python
    target_joint = [0.0, 0.5, 0.0, -1.0, 0.0, 1.5]
    robot.move_joint_async(target_joint)

    while robot.is_moving():
        sleep(0.01)
    ```

关节空间相对运动到指定位置（阻塞）

=== "Rust"
    ```rust
    let target_joint = [0.0, 0.5, 0.0, -1.0, 0.0, 1.5];
    robot.move_joint_rel(&target_joint);
    ```

=== "Python"
    ```python
    target_joint = [0.0, 0.5, 0.0, -1.0, 0.0, 1.5]
    robot.move_joint_rel(target_joint)
    ```

从文件中读取路径并沿轨迹运动(阻塞)

=== "Rust"
    ```rust
    robot.move_traj_from_file("path.json")?;
    ```

=== "Python"
    ```python
    robot.move_path_from_file("path.json")
    ```

=== "使用到的Json文件"
    ```json
    [
        {"Joint": [0.0, 0.5, 0.0, -1.0, 0.0, 1.5]},
        {"Joint": [0.1, 0.6, 0.1, -1.1, 0.1, 1.6]},
        {"Joint": [0.2, 0.7, 0.2, -1.2, 0.2, 1.7]}
    ]
    ```

## 完整可用接口

### ArmPreplannedMotion 单点运动

`ArmPreplannedMotion` 包含了单点运动的核心接口和其衍生接口。`move_joint` 和 `move_cartesian` 为需要机器人厂商实现的核心接口，其余接口由默认实现自动派生。

=== "Rust"
    ```rust
    pub trait ArmPreplannedMotion<const N: usize>: Arm<N> {
        // 需要实现
        fn move_joint(&mut self, target: &[f64; N]) -> RobotResult<()>;
        fn move_joint_async(&mut self, target: &[f64; N]) -> RobotResult<()>;
        fn move_cartesian(&mut self, target: &Pose) -> RobotResult<()>;
        fn move_cartesian_async(&mut self, target: &Pose) -> RobotResult<()>;

        // 自动派生
        fn move_to(&mut self, target: MotionType<N>) -> RobotResult<()>;
        fn move_to_async(&mut self, target: MotionType<N>) -> RobotResult<()>;
        fn move_rel(&mut self, target: MotionType<N>) -> RobotResult<()>;
        fn move_rel_async(&mut self, target: MotionType<N>) -> RobotResult<()>;
        fn move_int(&mut self, target: MotionType<N>) -> RobotResult<()>;
        fn move_int_async(&mut self, target: MotionType<N>) -> RobotResult<()>;
    }
    ```

=== "Python"
    ```python
    class ArmPreplannedMotion:
        def move_joint(self, target: list[float]) -> None: ...
        def move_joint_async(self, target: list[float]) -> None: ...
        def move_cartesian(self, target: Pose) -> None: ...
        def move_cartesian_async(self, target: Pose) -> None: ...
        def move_to(self, target: MotionType) -> None: ...
        def move_to_async(self, target: MotionType) -> None: ...
        def move_rel(self, target: MotionType) -> None: ...
        def move_rel_async(self, target: MotionType) -> None: ...
        def move_int(self, target: MotionType) -> None: ...
        def move_int_async(self, target: MotionType) -> None: ...
    ```

### ArmPreplannedPath 路径运动

`ArmPreplannedPath` 定义了路径/轨迹运动的接口。其中 `move_traj` 为密集轨迹运动（机器人按给定轨迹点逐点运动），`move_waypoints` 为路点运动（机器人自行规划经过路点的轨迹）。

=== "Rust"
    ```rust
    pub trait ArmPreplannedPath<const N: usize> {
        fn move_traj(&mut self, path: Vec<MotionType<N>>) -> RobotResult<()>;
        fn move_traj_async(&mut self, path: Vec<MotionType<N>>) -> RobotResult<()>;
        fn move_waypoints(&mut self, path: Vec<MotionType<N>>) -> RobotResult<()>;
        fn move_waypoints_async(&mut self, path: Vec<MotionType<N>>) -> RobotResult<()>;
        fn move_waypoints_prepare(&mut self, path: Vec<MotionType<N>>) -> RobotResult<()>;
        fn move_waypoints_start(&mut self, start: MotionType<N>) -> RobotResult<()>;
    }
    ```

=== "Python"
    ```python
    class ArmPreplannedPath:
        def move_traj(self, path: list[MotionType]) -> None: ...
        def move_traj_async(self, path: list[MotionType]) -> None: ...
        def move_waypoints(self, path: list[MotionType]) -> None: ...
        def move_waypoints_async(self, path: list[MotionType]) -> None: ...
        def move_waypoints_prepare(self, path: list[MotionType]) -> None: ...
        def move_waypoints_start(self, start: MotionType) -> None: ...
    ```

### ArmPreplannedMotionExt 扩展接口

当一个机器人同时实现了 `ArmPreplannedMotion` 和 `ArmPreplannedPath` 后，`ArmPreplannedMotionExt` 会自动派生，提供丰富的便捷函数。

=== "Rust"
    ```rust
    pub trait ArmPreplannedMotionExt<const N: usize>:
        ArmPreplannedMotion<N> + ArmPreplannedPath<N>
    {
        // 关节空间拓展
        fn move_joint_rel(&mut self, target: &[f64; N]) -> RobotResult<()>;
        fn move_joint_rel_async(&mut self, target: &[f64; N]) -> RobotResult<()>;
        fn move_joint_traj(&mut self, path: Vec<[f64; N]>) -> RobotResult<()>;

        // 笛卡尔空间拓展
        fn move_cartesian_rel(&mut self, target: &Pose) -> RobotResult<()>;
        fn move_cartesian_rel_async(&mut self, target: &Pose) -> RobotResult<()>;
        fn move_cartesian_int(&mut self, target: &Pose) -> RobotResult<()>;
        fn move_cartesian_int_async(&mut self, target: &Pose) -> RobotResult<()>;
        fn move_cartesian_traj(&mut self, path: Vec<Pose>) -> RobotResult<()>;

        // 欧拉角便捷接口
        fn move_linear_with_euler(&mut self, pose: [f64; 6]) -> RobotResult<()>;
        fn move_linear_with_euler_async(&mut self, pose: [f64; 6]) -> RobotResult<()>;
        fn move_linear_with_euler_rel(&mut self, pose: [f64; 6]) -> RobotResult<()>;
        fn move_linear_with_euler_rel_async(&mut self, pose: [f64; 6]) -> RobotResult<()>;
        fn move_linear_with_euler_int(&mut self, pose: [f64; 6]) -> RobotResult<()>;
        fn move_linear_with_euler_int_async(&mut self, pose: [f64; 6]) -> RobotResult<()>;

        // 四元数便捷接口
        fn move_linear_with_quat(&mut self, target: &na::Isometry3<f64>) -> RobotResult<()>;
        fn move_linear_with_quat_async(&mut self, target: &na::Isometry3<f64>) -> RobotResult<()>;
        fn move_linear_with_quat_rel(&mut self, target: &na::Isometry3<f64>) -> RobotResult<()>;
        fn move_linear_with_quat_rel_async(&mut self, target: &na::Isometry3<f64>) -> RobotResult<()>;
        fn move_linear_with_quat_int(&mut self, target: &na::Isometry3<f64>) -> RobotResult<()>;
        fn move_linear_with_quat_int_async(&mut self, target: &na::Isometry3<f64>) -> RobotResult<()>;

        // 齐次矩阵便捷接口
        fn move_linear_with_homo(&mut self, target: [f64; 16]) -> RobotResult<()>;
        fn move_linear_with_homo_async(&mut self, target: [f64; 16]) -> RobotResult<()>;
        fn move_linear_with_homo_rel(&mut self, target: [f64; 16]) -> RobotResult<()>;
        fn move_linear_with_homo_rel_async(&mut self, target: [f64; 16]) -> RobotResult<()>;
        fn move_linear_with_homo_int(&mut self, target: [f64; 16]) -> RobotResult<()>;
        fn move_linear_with_homo_int_async(&mut self, target: [f64; 16]) -> RobotResult<()>;

        // 文件加载
        fn move_waypoints_prepare_from_file(&mut self, path: &str) -> RobotResult<()>;
        fn move_traj_from_file(&mut self, path: &str) -> RobotResult<()>;
        fn move_waypoints_from_file(&mut self, path: &str) -> RobotResult<()>;
    }
    ```

=== "Python"
    ```python
    class ArmPreplannedMotionExt:
        def move_joint_rel(self, target: list[float]) -> None: ...
        def move_joint_rel_async(self, target: list[float]) -> None: ...
        def move_joint_traj(self, path: list[list[float]]) -> None: ...
        def move_cartesian_rel(self, target: Pose) -> None: ...
        def move_cartesian_rel_async(self, target: Pose) -> None: ...
        def move_cartesian_int(self, target: Pose) -> None: ...
        def move_cartesian_int_async(self, target: Pose) -> None: ...
        def move_cartesian_traj(self, path: list[Pose]) -> None: ...
        def move_linear_with_euler(self, pose: list[float]) -> None: ...
        def move_linear_with_euler_async(self, pose: list[float]) -> None: ...
        def move_linear_with_euler_rel(self, pose: list[float]) -> None: ...
        def move_linear_with_euler_rel_async(self, pose: list[float]) -> None: ...
        def move_linear_with_euler_int(self, pose: list[float]) -> None: ...
        def move_linear_with_euler_int_async(self, pose: list[float]) -> None: ...
        def move_linear_with_homo(self, target: list[float]) -> None: ...
        def move_linear_with_homo_async(self, target: list[float]) -> None: ...
        def move_linear_with_homo_rel(self, target: list[float]) -> None: ...
        def move_linear_with_homo_rel_async(self, target: list[float]) -> None: ...
        def move_linear_with_homo_int(self, target: list[float]) -> None: ...
        def move_linear_with_homo_int_async(self, target: list[float]) -> None: ...
        def move_waypoints_prepare_from_file(self, path: str) -> None: ...
        def move_traj_from_file(self, path: str) -> None: ...
        def move_waypoints_from_file(self, path: str) -> None: ...
    ```
