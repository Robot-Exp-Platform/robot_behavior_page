# 运动与控制

## MotionSpace

`MotionSpace<R>` 是一个类型级映射：

```rust
pub trait MotionSpace<R: ?Sized> {
    type Target;
}
```

它没有方法，只说明某个空间在某个机器人上携带什么目标类型。

内置空间：

| 空间 | Target |
|---|---|
| `JointSpace<N>` | `[f64; N]` |
| `FlangeSpace` | `Pose` |
| `TcpSpace` | `Pose` |
| `EndSpace` | `Pose`，要求机器人实现 `EndPoint` |
| `Relative<S>` | `S::Target` |
| `Inertial<S>` | `S::Target` |

## MoveTo 与 MoveTraj

`MoveTo<S>` 表达单点目标：

```rust
fn move_to(&mut self, target: S::Target) -> RobotResult<()>;
fn move_to_sync(&mut self, target: S::Target) -> RobotResult<()>;
```

`MoveTraj<S>` 表达密集轨迹、连续 path 或 sparse waypoint：

```rust
fn move_traj(&mut self, traj: Vec<S::Target>) -> RobotResult<()>;
fn move_path<F>(&mut self, path: F) -> RobotResult<()>
where
    F: Fn(f64) -> Option<S::Target>;
fn move_waypoints(&mut self, waypoints: Vec<S::Target>) -> RobotResult<()>;
```

`Motion` 是 blanket trait，用户调用它，驱动不需要实现它。

## ControlSpace

`ControlSpace<R>` 是控制侧的类型级映射：

```rust
pub trait ControlSpace<R: ?Sized> {
    type Obs;
    type Command;
}
```

内置通道全部观察 `ArmState<N>`：

| 通道 | Command |
|---|---|
| `TorqueControl<N>` | `[f64; N]` |
| `JointPositionControl<N>` | `[f64; N]` |
| `JointVelocityControl<N>` | `[f64; N]` |
| `CartesianVelocityControl<N>` | `[f64; 6]` |

虽然前三个命令类型都是 `[f64; N]`，但由于通道类型不同，Rust 可以区分不同 impl。

## RealtimeControl

驱动实现：

```rust
fn safe_command() -> S::Command;

fn control_with_closure<F>(&mut self, closure: F) -> RobotResult<()>
where
    F: FnMut(S::Obs, Duration) -> (S::Command, bool) + Send + 'static;
```

闭包每个控制周期收到观测和 `dt`，返回命令和 `done` 标志。`safe_command` 用于断开或退出时的安全回退，例如零力矩或保持命令。
