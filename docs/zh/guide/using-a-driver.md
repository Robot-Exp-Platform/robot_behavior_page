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
robot.move_to::<FlangeSpace>(Pose::Position([0.4, 0.0, 0.3]))?;
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

`move_path` / `move_waypoints` 需要具体驱动实现；不支持的规划入口应显式返回错误。可复用 `utils::trajectory` 或 `utils::path_generate`。

## 控制会话

普通控制器仍可通过 `Control::control_with` 返回 `(command, done)`。当本次回调需要在没有算法指令时结束，使用 `Control::control_with_flow`。以下只展示无指令退出路径，不是真机启动流程：

```rust
use std::ops::ControlFlow;
use robot_behavior::{Control, RobotResult, TorqueControl};

fn finish_session<R>(robot: &mut R) -> RobotResult<()>
where
    R: robot_behavior::ControlWith<TorqueControl<7>>,
{
    robot.control_with_flow::<TorqueControl<7>, _>(|_state, _dt| {
        ControlFlow::Break(())
    })
}
```

需要发送有效的最后一条指令时，返回 `Continue((command, true))`；驱动先发送，再正常结束。`Break(())` 则进入设备结束协议，两者均不能解释为通用急停。`hold_command(obs)` 是连续性回退，不是安全保证。

`_async` 版本只是接收异步周期闭包，整个控制 session 仍阻塞。通道的观测类型、三种返回值和 runtime 边界见 [运动与控制](../concepts/motion-and-control.md)。

## Roplat 应用

显式启用 `roplat` feature，应用通过 `#[roplat::system]` 组织节点和多层节律域。设备调用放在对应驱动、Node 或 Rhythm 实现中；不要把应用图改成手写 `.process()` 调用链。具体执行状态与资源边界见 [Roplat 集成](roplat-integration.md)。

## 错误处理

所有 fallible API 使用：

```rust
pub type RobotResult<T> = Result<T, RobotException>;
```

常见错误包括网络错误、命令错误、反序列化错误、无法处理的指令和 FFI 数据错误。
