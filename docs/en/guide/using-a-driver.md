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
use robot_behavior::{FlangeSpace, JointSpace, Motion, MoveTo, Pose, RobotResult};

fn run<R>(robot: &mut R) -> RobotResult<()>
where
    R: MoveTo<JointSpace<6>> + MoveTo<FlangeSpace>,
{
    robot.move_to::<JointSpace<6>>([0.0; 6])?;
    robot.move_to_sync::<FlangeSpace>(Pose::Position([0.4, 0.0, 0.3]))?;
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
use robot_behavior::{JointSpace, Motion, MoveTraj, RobotResult};

fn path<R>(robot: &mut R) -> RobotResult<()>
where
    R: MoveTraj<JointSpace<6>>,
{
    robot.move_waypoints::<JointSpace<6>>(vec![[0.0; 6], [0.2; 6]])
}
```

Default `move_path` / `move_waypoints` mean that no planner is wired. A driver that supports these entry points should override them and eventually funnel into `move_traj`.

## Realtime Control

Realtime control is selected by channel type:

```rust
use robot_behavior::{Control, RealtimeControl, RobotResult, TorqueControl};

fn hold<R>(robot: &mut R) -> RobotResult<()>
where
    R: RealtimeControl<TorqueControl<7>>,
{
    robot.control_with_closure::<TorqueControl<7>, _>(|_state, _dt| {
        ([0.0; 7], true)
    })
}
```

`RealtimeControl<S>::safe_command()` provides a safe fallback for disconnects or abnormal exits, such as zero torque or hold-position commands.
