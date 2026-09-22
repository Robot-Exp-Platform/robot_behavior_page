# Roplat Integration

The current development source keeps device traits independent of roplat. Enable the `robot_behavior` **`roplat` feature** to use `robot_behavior::roplat::*`; default driver builds do not enable it.

## Compose applications with System

Use `#[roplat::system]` to organize the application's graph and child rhythm domains. Node authors implement `process`; driver authors implement device sessions and rhythm adapters. A hand-written chain of `.process()` calls in an application bypasses the graph's generated lifecycle and execution handling.

Use the [core syntax documentation](https://github.com/Robot-Exp-Platform/roplat/blob/main/docs/system-syntax.md) matching your checkout, and combine [roplat-skills](https://github.com/Robot-Exp-Platform/roplat-skills) with the device capability guidance in [robot_behavior-skills](https://github.com/Robot-Exp-Platform/robot_behavior-skills). Do not invent a new DSL spelling to work around a driver compile error.

## Data and execution state are separate

| Adapter | Business input | Business output |
|---|---|---|
| `MotionNode<R, S>` | `(RobotResult<R>, S::Target)` | `RobotResult<R>` |
| `SpaceMapNode<M, From, To>` | model input | `RobotResult<M::Output>` |
| `SafetyNode<F, Command>` | `(Command, bool)` | `(Command, bool)` |
| `ControlRhythm<R, S>` | `RobotResult<R>` | `R` on completed execution |

`ControlRhythm` yields `(S::Obs, Duration)` and receives business `Feed = (S::Command, bool)`. The surrounding execution result is separate: `Execution<T>` represents completed output, cooperative stop, or execution error. A normal business `Result` from a Node does not automatically fail every surrounding domain.

Existing `ControlRhythm` calls blocking `ControlWith::control_with_flow_async`; `AsyncControlRhythm` awaits the complete session from `AsyncControlWith::control_native_async`. Both share these exit rules:

- A completed domain produces a command/done tuple and becomes `Continue((command, done))`.
- A stopped or failed domain has no algorithm command and becomes `Break(())`.
- The device then completes its own termination protocol before the rhythm returns.
- Domain errors and device termination errors must both remain observable; a later cleanup error must not replace the original failure.

`ControlRhythm` remains a **blocking device session with an async per-cycle callback**. It does not make transport truly asynchronous, and it cannot promise that sibling work on the calling executor progresses while the session blocks. Use finite mock graphs to validate orchestration; do not infer concurrent hardware behavior or timing from compilation.

## Ownership and lifecycle limits

`Lifecycle::on_init` / `on_shutdown` belong to the layer that creates the Node or Rhythm. Passing an existing object into a child domain does not reinitialize or close it. Repeated drives preserve state; explicitly create or reset an object when that is intended.

Every cooperative `drive` exit returns the opaque node tuple `N`, including stop and error. Await the in-flight domain to recover `N`; do not drop it to implement fail-fast behavior.

**The robot passed as `ControlRhythm::Input` is not automatically part of `N`.** `Completed(R)` returns it as output. `Stopped` and `Err` do not contain that `R`, so this adapter does not guarantee that the caller receives the input robot on those paths. The device session's termination is distinct from final `Robot::shutdown` or a creating scope's lifecycle hooks. Likewise, `MotionNode` returns its input robot only on success. Applications that require recovery of input-owned resources must account for this boundary explicitly.

Panic, dropping the outer Future, forced task abort, and uncooperative blocking code are outside cooperative return guarantees. A test watchdog timing out is a failed test, not proof of safe shutdown. Driver-specific hold/stop, sample validity and simulator timing require separate validation; no universal zero-command policy is implied.

## Driver runtime boundary

Franka currently creates a local Tokio runtime inside its blocking async-callback facade. Calling it from an already entered Tokio runtime can panic. A successful mock ControlRhythm/System test does not establish that the same calling context works for Franka. This restriction remains on the old blocking facade; use the native entry point below for asynchronous composition.


## Native asynchronous control rhythm

Use `AsyncControlRhythm<R, S>` in async System graphs when the driver implements `AsyncControlWith<S>`. Its Input, Output, Yield, Feed and cooperative N-return contracts match ControlRhythm. Device waits allow the outer task to poll ready siblings. Each cycle still completes its domain before sending a command, and a dependent parent successor still waits for the whole drive.

Franka native sessions require an application Tokio runtime with I/O and time enabled. Entry, UDP cycles and TCP termination await I/O. Synchronous move_to keeps its ordinary kernel. Async alone does not guarantee fairness during CPU-heavy work or hardware deadlines.

`AsyncControlCallback::call(&mut self, ...)` returns a Send Future and may borrow callback state across await. Ordinary FnMut-to-Send-Future callbacks have a blanket implementation. Stateful lending controllers can implement the trait directly. The driver borrows the callback for the session, without requiring per-cycle boxing, spawning or node copies.

Native behavior traits need no roplat feature. Enable `robot_behavior/roplat` for graph adapters and `rsbullet/roplat` for SimRhythm. Independent repositories retain full dependency declarations with pinned Git revisions. Local integration uses source patches at the workspace root.
