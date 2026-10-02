# Using a Driver

Users usually do not implement traits directly. They receive a concrete driver instance, here named `robot`, and call the blanket `Motion` / `Control` methods.

## Lifecycle

Every device capability starts with `Robot`:

```rust
use robot_behavior::{Robot, RobotResult};

fn bringup<R: Robot>(robot: &mut R) -> RobotResult<()> {
    robot.init()?;
    robot.enable()?;
    let _state = robot.read_state()?;
    Ok(())
}
```

`shutdown`, `disable`, `stop` and `emergency_stop` are also part of `Robot`. A concrete driver may keep irrelevant lifecycle methods as their default no-op implementations.

## Point Motion

`Motion` is a blanket trait. Select the motion space with turbofish syntax:

```rust
use robot_behavior::{FlangeSpace, JointSpace, Motion, Pose, RobotResult};

fn run<R>(robot: &mut R) -> RobotResult<()>
where
    R: robot_behavior::MoveTo<JointSpace<6>> + robot_behavior::MoveTo<FlangeSpace>,
{
    robot.move_to::<JointSpace<6>>([0.0; 6])?;
    robot.move_to::<FlangeSpace>(Pose::Position([0.4, 0.0, 0.3]))?;
    Ok(())
}
```

Common spaces:

| Space | Target | Meaning |
|---|---|---|
| `JointSpace<N>` | `[f64; N]` | joint position, usually radians |
| `FlangeSpace` | `Pose` | flange pose |
| `TcpSpace` | `Pose` | tool-center-point pose |
| `EndSpace` | `Pose` | endpoint space with `EndPoint` limits |
| `Relative<S>` | `S::Target` | wrapper interpreted relative to the current pose |
| `Inertial<S>` | `S::Target` | wrapper interpreted in a fixed inertial frame |

## Trajectory Motion

Once a driver implements `MoveTraj<S>`, users can submit dense trajectories, continuous paths or waypoints:

```rust
use robot_behavior::{JointSpace, Motion, RobotResult};

fn path<R>(robot: &mut R) -> RobotResult<()>
where
    R: robot_behavior::MoveTraj<JointSpace<6>>,
{
    robot.move_waypoints::<JointSpace<6>>(vec![[0.0; 6], [0.2; 6]])
}
```

`move_path` / `move_waypoints` must be implemented by the driver. Unsupported planning operations should return an explicit error; supported operations may funnel into `move_traj`.

## Control sessions

An ordinary controller still returns `(command, done)` through `Control::control_with`. Use `Control::control_with_flow` when the callback must stop without producing an algorithm command. This example demonstrates only the no-command exit path; it is not a hardware bring-up recipe:

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

For a valid final command, return `Continue((command, true))`; the driver sends it before normal completion. `Break(())` instead enters the backend's termination protocol. Neither path is a generic emergency stop. `hold_command(obs)` is a continuity fallback, not a safety guarantee.

The `_async` variants accept async per-cycle callbacks but still block for the whole session. See [Motion and Control](../concepts/motion-and-control.md) for the execution and runtime limits.

## Roplat applications

Enable the `roplat` feature explicitly and organize application execution with `#[roplat::system]`, including child rhythm domains. Keep ordinary device calls inside the relevant driver/Node/Rhythm implementation; do not replace the application's graph with a hand-written chain of `.process()` calls. See [Roplat integration](roplat-integration.md).
