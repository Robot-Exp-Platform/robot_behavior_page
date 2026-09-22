# Motion and Control

## Motion spaces

`MotionSpace<R>` maps a typed space to its `Target`:

| Space | Target |
|---|---|
| `JointSpace<N>` | `[f64; N]`, with `R: Joints<N>` |
| `EndSpace` | `Pose`, with `R: EndPoint` |
| `FlangeSpace`, `TcpSpace` | `Pose` |
| `Relative<S>`, `Inertial<S>` | `S::Target` |

Drivers implement `MoveTo<S>` and `MoveTraj<S>`. Applications use the blanket `Motion` facade and choose the space with turbofish syntax.

`move_to` blocks until completion or failure. `move_to_async` returns an inert Future and defaults to an unsupported error; a driver must provide a real asynchronous backend to support it. It is a different contract from an async *control callback* below.

`MoveTraj<S>` requires `move_traj`, `move_path` and `move_waypoints`. Drivers can funnel planning into dense trajectories, or explicitly return `UnprocessableInstructionError` for unsupported path/waypoint operations. Inspect simulator-specific queue semantics separately.

## Control spaces

`ControlSpace<R>` fixes `Obs` and `Command`. Equal payload types do not make two channels equivalent.

| Channel | Obs | Command |
|---|---|---|
| `TorqueControl<N>` | `JointState<N>` | `[f64; N]` |
| `JointPositionControl<N>` | `JointState<N>` | `[f64; N]` |
| `JointVelocityControl<N>` | `JointState<N>` | `[f64; N]` |
| `ArmTorqueControl<N>` | `ArmState<N>` | `[f64; N]` |
| `CartesianPoseControl<N>` | `ArmState<N>` | `Pose` |
| `CartesianVelocityControl<N>` | `ArmState<N>` | `[f64; 6]` |
| `BaseVelocityControl` | `BaseState` | `[f64; 6]` |

## A callback can produce a command or end the session

Drivers implement `ControlWith<S>::control_with_flow`. These are the relevant signatures, with the surrounding trait bounds omitted:

```rust
pub type ControlStep<Command> = std::ops::ControlFlow<(), (Command, bool)>;

fn control_with_flow<F>(&mut self, closure: F) -> RobotResult<()>
where
    F: FnMut(S::Obs, Duration) -> ControlStep<S::Command>;
```

| Callback result | Meaning |
|---|---|
| `ControlFlow::Continue((command, false))` | Send this computed command and continue. |
| `ControlFlow::Continue((command, true))` | Send this final computed command, then finish normally. |
| `ControlFlow::Break(())` | No algorithm command was produced for this cycle; enter the device's session termination protocol. |

`Break(())` is not an emergency stop and does not mean that no protocol packet can be sent. The device determines its required termination handshake or continuity packet. Do not fabricate a zero command to satisfy the callback return type. After either ending path the driver must not invoke the callback again.

The existing `control_with` interface remains available. Its callback returns `(command, done)`; the default wrapper maps it to `Continue((command, done))`. Existing controller helpers can continue to use that interface. The callback is scoped to the session, without a blanket `Send + 'static` requirement, and must not be stored after the call returns.

`hold_command(obs)` is a continuity fallback based on the current channel's observation. It is not a universal safe command or a substitute for vendor safety mechanisms.

## Async callback, blocking session

`control_with_flow_async` accepts `async FnMut(...) -> ControlStep<Command>`. `control_with_async` accepts `async FnMut(...) -> (Command, bool)`. **Both methods return `RobotResult<()>`, not a Future; the whole session still blocks.** The default adapter runs each callback Future to completion with `futures::executor::block_on`.

An async callback therefore does not establish a Tokio reactor or guarantee that timers, sockets, or sibling futures on the calling thread will progress. Check the driver's execution model before using runtime-dependent operations. A truly asynchronous device session is a separate design task.

For the optional graph adapter, see [Roplat integration](../guide/roplat-integration.md).
