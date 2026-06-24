# 使用驱动

本页站在“已经拿到一个驱动实例”的用户视角。假设变量名为 `robot`。

## 生命周期

任何驱动至少实现 `Robot`：

```rust
robot.init()?;
robot.enable()?;
let state = robot.read_state()?;
robot.stop()?;
robot.shutdown()?;
```

`Robot::CONTROL_PERIOD` 是驱动声明的控制周期，实时控制和 COPP 轨迹采样会使用它。

## 单点运动

运动接口通过空间类型选择命令含义：

```rust
use robot_behavior::{FlangeSpace, JointSpace, Motion, Pose};

robot.move_to::<JointSpace<6>>([0.0; 6])?;
robot.move_to_sync::<FlangeSpace>(Pose::Position([0.4, 0.0, 0.3]))?;
```

常见空间：

| 空间 | 目标类型 | 语义 |
|---|---|---|
| `JointSpace<N>` | `[f64; N]` | 关节位置，单位通常为 rad |
| `FlangeSpace` | `Pose` | 法兰位姿 |
| `TcpSpace` | `Pose` | 工具中心点位姿 |
| `EndSpace` | `Pose` | 带 `EndPoint` 限位语义的末端空间 |
| `Relative<S>` | `S::Target` | 相对当前位姿解释 |
| `Inertial<S>` | `S::Target` | 固定惯性系解释 |

## 轨迹、路径和 waypoint

如果驱动实现了 `MoveTraj<S>`，就可以使用：

```rust
robot.move_traj::<JointSpace<6>>(vec![[0.0; 6], [0.2; 6]])?;

robot.move_path::<JointSpace<6>, _>(|s| {
    Some([s; 6])
})?;

robot.move_waypoints::<JointSpace<6>>(vec![[0.0; 6], [0.4; 6], [0.0; 6]])?;
```

默认 `move_path` / `move_waypoints` 不会自动规划，具体驱动需要显式实现。可复用 `utils::trajectory` 或 `utils::path_generate`。

## 实时控制

实时控制通过 `ControlSpace` 类型选择控制通道：

```rust
use robot_behavior::{Control, TorqueControl};

robot.control_with_closure::<TorqueControl<7>, _>(|state, dt| {
    let _ = (state, dt);
    ([0.0; 7], true)
})?;
```

内置通道：

| 通道 | 观测 | 命令 |
|---|---|---|
| `TorqueControl<N>` | `ArmState<N>` | `[f64; N]` torque |
| `JointPositionControl<N>` | `ArmState<N>` | `[f64; N]` joint position |
| `JointVelocityControl<N>` | `ArmState<N>` | `[f64; N]` joint velocity |
| `CartesianVelocityControl<N>` | `ArmState<N>` | `[f64; 6]` spatial velocity |

## 错误处理

所有 fallible API 使用：

```rust
pub type RobotResult<T> = Result<T, RobotException>;
```

常见错误包括网络错误、命令错误、反序列化错误、无法处理的指令和 FFI 数据错误。
