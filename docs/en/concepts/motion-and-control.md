# Motion and Control

## MotionSpace

`MotionSpace<R>` has one associated type, `Target`. It maps a command space to the data carried by a command in that space.

| Space | Target | Requirement |
|---|---|---|
| `JointSpace<N>` | `[f64; N]` | `R: Joints<N>` |
| `EndSpace` | `Pose` | `R: EndPoint` |
| `FlangeSpace` | `Pose` | any `R` |
| `TcpSpace` | `Pose` | any `R` |
| `Relative<S>` | `S::Target` | `S: MotionSpace<R>` |
| `Inertial<S>` | `S::Target` | `S: MotionSpace<R>` |

`MoveTo<S>` is for one target. `MoveTraj<S>` is for dense trajectories, continuous paths and waypoints. `Motion` is a blanket trait that forwards user calls to the concrete `MoveTo` / `MoveTraj` implementations.

## MoveTraj Funnel

```text
move_waypoints -> driver interpolation/planning -> move_traj
move_path      -> driver sampling/planning       -> move_traj
move_traj      -> hardware/simulator command
```

Default `move_path` / `move_waypoints` mean that no planner is installed. Drivers that support them should override the methods and eventually call their own `move_traj`.

## ControlSpace

`ControlSpace<R>` is the control-side counterpart of `MotionSpace<R>`:

| Channel | Obs | Command |
|---|---|---|
| `TorqueControl<N>` | `ArmState<N>` | `[f64; N]` |
| `JointPositionControl<N>` | `ArmState<N>` | `[f64; N]` |
| `JointVelocityControl<N>` | `ArmState<N>` | `[f64; N]` |
| `CartesianVelocityControl<N>` | `ArmState<N>` | `[f64; 6]` |

Several channels share `[f64; N]` as the command type, but their marker types remain distinct.

## RealtimeControl

Drivers implement `RealtimeControl<S>`:

```rust
fn control_with_closure<F>(&mut self, closure: F) -> RobotResult<()>
where
    F: FnMut(S::Obs, Duration) -> (S::Command, bool) + Send + 'static;
```

The closure receives one observation and `dt` per control cycle, then returns the next command and a done flag. `safe_command()` provides the fallback command for disconnects or abnormal exits.
