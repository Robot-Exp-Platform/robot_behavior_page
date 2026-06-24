# 几何与运动学

## Pose

`Pose` 是笛卡尔位姿的统一表示：

| 变体 | 内容 |
|---|---|
| `Euler([f64; 3], [f64; 3])` | 平移 + roll/pitch/yaw |
| `Quat(Isometry3<f64>)` | nalgebra isometry |
| `Homo([f64; 16])` | column-major 齐次矩阵 |
| `AxisAngle([f64; 3], [f64; 3], f64)` | 平移 + 轴角 |
| `Position([f64; 3])` | 纯位置，姿态为单位旋转 |

常用转换：

```rust
let pose = Pose::Position([0.4, 0.0, 0.3]);
let iso = pose.quat();
let homo = pose.homo();
let xyz = pose.position();
```

## DH 参数

`DhParam` 支持标准 DH、改进 DH，以及直接给定位姿：

```rust
use robot_behavior::{DhParam, dh_param, mdh_param};

let standard: DhParam = dh_param!(0.0, 0.2, 0.3, 1.57);
let modified: DhParam = mdh_param!(0.0, 0.2, 0.3, 1.57);
```

`to_se3(q)` 会把关节变量应用到当前连杆，返回 `Isometry3<f64>`。

## ArmKineCache

`ArmKineCache<N>` 从 `[DhParam; N]`、`q`、`q_dot` 构建，并缓存：

- `t_prefix[0..=N]`：base 到各 link 的累计变换。
- `origins[0..=N]`：各 link 原点。
- `z_axes[0..=N]`：各 link 的 z 轴。
- `joint_z[0..N]` / `joint_p[0..N]`：世界系下的关节轴和轴原点。

常用方法：

```rust
let cache = MyArm::kine_cache(&q, &q_dot);
let ee = cache.end_effector_pose();
let j = cache.jacobian();
let twist = cache.ee_twist();
```

## 正逆运动学

`ArmForwardKinematics<N>` 由驱动声明：

```rust
const DH: [DhParam; N];
```

默认提供 `kine_cache` 与 `fk_end_pose`。

`ArmInverseKinematics<N>` 在 FK 和 `Joints<N>` 基础上提供：

- `ANALYTIC_FAMILY`：可选解析解族。
- `ik_analytic_all` / `ik_analytic_best`：解析解入口。
- `task_error_and_jacobian`：任务空间残差和雅可比。
- `ik_step`：DLS、JT、Newton、LM 或解析回退的一步更新。

## 轨迹辅助

`utils::path_generate` 提供简单 path：

- `joint_linear`
- `cartesian_quat_linear`
- `joint_trapezoid`
- `joint_s_curve`

`utils::trajectory` 使用 `copp` 进行更严格的限位轨迹规划：

- `plan_waypoints_traj_via_copp`
- `plan_path_traj_via_copp`
- `plan_s_t_via_copp`

这些函数会使用 `Joints<N>` 的速度、加速度、jerk 限位，并按 `Robot::CONTROL_PERIOD` 生成时间均匀采样。
