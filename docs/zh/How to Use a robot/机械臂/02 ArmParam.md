# 02 机械臂参数接口及运动学

## ArmParam 参数常量

`ArmParam` 特征定义了机械臂的物理参数常量，包括关节类型、关节极限、速度/加速度/加加速度边界等。这些常量用于运动限幅和安全检查。

=== "Rust"
    ```rust
    pub trait ArmParam<const N: usize> {
        const JOINT_TYPES: [JointType; N];      // 关节类型（旋转 / 移动）
        const JOINT_DEFAULT: [f64; N];           // 默认关节位置
        const JOINT_PACKED: [f64; N];            // 收拢位姿关节位置
        const JOINT_MIN: [f64; N];               // 关节下限 [rad]
        const JOINT_MAX: [f64; N];               // 关节上限 [rad]
        const JOINT_VEL_BOUND: [f64; N];         // 关节速度边界 [rad/s]
        const JOINT_ACC_BOUND: [f64; N];         // 关节加速度边界 [rad/s²]
        const JOINT_JERK_BOUND: [f64; N];        // 关节加加速度边界 [rad/s³]
        const CARTESIAN_VEL_BOUND: f64;          // 笛卡尔速度边界 [m/s]
        const CARTESIAN_ACC_BOUND: f64;          // 笛卡尔加速度边界 [m/s²]
        const CARTESIAN_JERK_BOUND: f64;         // 笛卡尔加加速度边界 [m/s³]
        const ROTATION_VEL_BOUND: f64;           // 旋转速度边界 [rad/s]
        const ROTATION_ACC_BOUND: f64;           // 旋转加速度边界 [rad/s²]
        const ROTATION_JERK_BOUND: f64;          // 旋转加加速度边界 [rad/s³]
        const TORQUE_BOUND: [f64; N];            // 力矩边界 [Nm]
        const TORQUE_DOT_BOUND: [f64; N];        // 力矩变化率边界 [Nm/s]
    }
    ```

=== "Python"
    ```python
    class ArmParam:
        @staticmethod
        def joint_default() -> list[float]: ...
        @staticmethod
        def joint_min() -> list[float]: ...
        @staticmethod
        def joint_max() -> list[float]: ...
        @staticmethod
        def joint_vel_bound() -> list[float]: ...
        @staticmethod
        def joint_acc_bound() -> list[float]: ...
        @staticmethod
        def joint_jerk_bound() -> list[float]: ...
        @staticmethod
        def cartesian_vel_bound() -> float: ...
        @staticmethod
        def cartesian_acc_bound() -> float: ...
        @staticmethod
        def cartesian_jerk_bound() -> float: ...
        @staticmethod
        def rotation_vel_bound() -> float: ...
        @staticmethod
        def rotation_acc_bound() -> float: ...
        @staticmethod
        def rotation_jerk_bound() -> float: ...
        @staticmethod
        def torque_bound() -> list[float]: ...
        @staticmethod
        def torque_dot_bound() -> list[float]: ...
    ```

### 关节类型

```rust
pub enum JointType {
    Revolute,   // 旋转关节
    Prismatic,  // 移动关节
}
```

### 限幅函数

`ArmParam` 同时提供了一组限幅函数，用于在实时控制中对关节位置、速度、加速度和力矩进行安全限制。这些函数遵循逐级限幅的思想：先限制高阶量（加加速度），再逐级传递到低阶量（位置）。

| 函数 | 说明 |
|------|------|
| `limit_joint_jerk` | 限制关节加加速度 |
| `limit_joint_acc` | 限制关节加速度（考虑加加速度约束） |
| `limit_joint_vel` | 限制关节速度（考虑加速度和加加速度约束） |
| `limit_joint` | 限制关节位置（考虑速度、加速度和加加速度约束，并裁剪到关节范围） |
| `limit_torque` | 限制力矩（考虑力矩变化率约束） |

## ArmForwardKinematics 正向运动学

正向运动学已从 `ArmParam` 中分离为独立的 `ArmForwardKinematics` 特征，使用 DH 参数进行正运动学计算。

=== "Rust"
    ```rust
    pub trait ArmForwardKinematics<const N: usize> {
        const DH: [DhParam; N];

        fn kine_cache(q: &[f64; N], q_dot: &[f64; N]) -> ArmKineCache<N>;
        fn fk_end_pose(q: &[f64; N]) -> Pose;
    }
    ```

=== "Python"
    ```python
    class ArmParam:
        @staticmethod
        def dh() -> list[list[float]]: ...

        @staticmethod
        def forward_kinematics(q: list[float]) -> Pose: ...
    ```

### DH 参数

`DhParam` 枚举支持多种 DH 参数表示方式：

```rust
pub enum DhParam {
    DH { theta: f64, d: f64, r: f64, alpha: f64 },           // 标准 DH 参数
    MDH { alpha: f64, a: f64, theta: f64, d: f64 },          // 修正 DH 参数
    Iso3Quat { pos: [f64; 3], quat: [f64; 4] },              // 四元数 SE(3) 变换
    Iso3RPY { pos: [f64; 3], rpy: [f64; 3] },                // RPY 欧拉角 SE(3) 变换
}
```

### ArmKineCache 运动学缓存

`ArmKineCache` 是正向运动学的缓存结构，一次计算后可以高效地获取各种运动学量：

```rust
pub struct ArmKineCache<const N: usize> {
    pub q_dot: [f64; N],                          // 关节速度
    pub t_prefix: [na::Isometry3<f64>; N + 1],    // 各连杆累积变换
    pub origins: [na::Vector3<f64>; N + 1],        // 各连杆原点位置
    pub z_axes: [na::Vector3<f64>; N + 1],         // 各连杆 z 轴方向
}
```

`ArmKineCache` 提供的方法：

| 方法 | 说明 |
|------|------|
| `build(dh, q, q_dot)` | 构建运动学缓存 |
| `end_effector_pose()` | 获取末端执行器位姿 |
| `link_pose(link)` | 获取指定连杆位姿 |
| `relative_pose(a, b)` | 获取两个连杆之间的相对位姿 |
| `jacobian()` | 计算几何雅可比矩阵 |
| `jacobian_at_link(link_index)` | 计算到指定连杆的雅可比矩阵 |
| `ee_twist()` | 获取末端执行器的速度旋量 |

## ArmInverseKinematics 逆向运动学

逆向运动学特征支持解析解和数值迭代方法：

=== "Rust"
    ```rust
    pub trait ArmInverseKinematics<const N: usize>:
        ArmForwardKinematics<N> + ArmParam<N>
    {
        const ANALYTIC_FAMILY: Option<AnalyticFamily> = None;

        fn ik_analytic_all(target: &Pose) -> Option<Vec<JVec<N>>>;
        fn ik_analytic_best(q_seed: &JVec<N>, target: &Pose) -> Option<JVec<N>>;
        fn task_error_and_jacobian(q: &JVec<N>, target: &Pose) -> (Twist, Jaco<N>);
        fn ik_step(q: &JVec<N>, target: &Pose, method: &IKMethod) -> JVec<N>;
        fn ik_solve(q0: &JVec<N>, target: &Pose, method: IKMethod) -> JVec<N>;
    }
    ```

### IK 求解方法

`IKMethod` 枚举支持多种逆运动学求解算法：

```rust
pub enum IKMethod {
    Analytic { fallback: Box<IKMethod> },       // 解析解（如有），否则回退到 fallback
    DLS { lambda: f64, stop: CommonStop },      // 阻尼最小二乘
    JT { gain: f64, stop: CommonStop },         // 雅可比转置法
    Newton { stop: CommonStop },                // 牛顿-拉夫森法
    LM { lambda0: f64, stop: CommonStop },      // Levenberg-Marquardt 法
}
```

### 解析解族

```rust
pub enum AnalyticFamily {
    Planar2R,      // 平面 2R 机械臂
    Planar3R,      // 平面 3R 机械臂
    ScaraRRPR,     // SCARA RRPR 机械臂
    Spherical6R,   // 球腕 6R 机械臂
    Scara,         // SCARA 机械臂
    Custom,        // 自定义解析实现
}
```
