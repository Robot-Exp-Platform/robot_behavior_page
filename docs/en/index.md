# Robot Behavior

> **This site is frozen.** On 2026-10-26 `robot_behavior` was renamed `roplat_behavior` and split into `roplat_behavior`, `roplat_kinematics`, `roplat_control` and `roplat_trajectory`. This site documents robot_behavior 0.8 and is no longer updated. `robot_behavior` 0.8.1 on crates.io is the final release (same API as 0.8.0). Code and migration notes: [roplat_behavior](https://github.com/Robot-Exp-Platform/roplat_behavior); as of 2026-10-26, 0.9 is not on crates.io yet.

`robot_behavior` is the shared behavior-contract crate for the Roplat driver stack. It is not tied to one robot model; it extracts the behavior interfaces that real arms, simulators, visualizers and Roplat adapters need to share.

The current version is centered on **typed motion spaces**, **typed realtime control channels**, **fixed-DoF arm modeling**, and reusable kinematics / trajectory utilities.

## Who Should Read This

- Driver users: start with [Using a Driver](guide/using-a-driver.md).
- Driver authors: start with [Implementing a Driver](guide/implementing-a-driver.md).
- API/design readers: read [Capability Model](concepts/capability-model.md) and [Motion and Control](concepts/motion-and-control.md).
- API lookup: see [API Map](reference/api-map.md).

## Main Interfaces

| Layer | Key types | Purpose |
|---|---|---|
| Lifecycle | `Robot` | connect, enable, read state, stop, shutdown |
| Arm description | `Joints<N>`, `EndPoint`, `Arm<N>` | joint limits, Cartesian limits, state and payload |
| Motion | `MotionSpace`, `MoveTo`, `MoveTraj`, `Motion` | typed motion spaces and a uniform call surface |
| Realtime control | `ControlSpace`, `ControlWith`, `Control` | typed control channels and closure-driven loops |
| Geometry / kinematics | `Pose`, `DhParam`, `ArmKineCache`, `IKMethod` | pose, DH, FK/IK, Jacobian |
| Utilities | `utils::*` | limits, interpolation, trajectory generation, copp retiming |

!!! note
    `src/robot_old` is not the current public interface. This documentation follows the items re-exported by `src/lib.rs` and `src/robot/mod.rs`.

For the optional `roplat` feature, lifecycle ownership and cooperative exits, read [Roplat integration](guide/roplat-integration.md). This development documentation does not certify hardware operation or realtime timing.
