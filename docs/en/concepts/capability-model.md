# Capability Model

`robot_behavior` expresses capabilities through composable traits. A type does not need to implement everything; it exposes only the behavior it actually supports.

## Core Layers

```text
Robot
├── Joints<N>
├── EndPoint
├── MoveTo<S>
├── MoveTraj<S>
├── ControlWith<S>
└── Arm<N> = Robot + Joints<N> + EndPoint + MoveTo<JointSpace<N>> + MoveTo<FlangeSpace>
```

`Arm<N>` is the aggregate arm trait, but it is not the only entry point. A simulation-only type may implement `RobotDescription`; a driver supporting one control channel may implement just that `ControlWith<S>`.

## Data and State

- `Robot::State` is the driver's native state and may mirror a vendor SDK type.
- `ArmState<N>` contains `joint: JointState<N>`, a flange `StateView<SpatialSample>`, optional TCP/stiffness views, and optional load data. Each `StateView<T>` preserves `meas`, `cmd` and `des`.
- `LoadState` describes payload mass, center of mass and inertia.

## Why Not Runtime Enums

Older APIs used runtime enums such as `MotionType` / `ControlType`. The current mainline uses typed spaces:

```rust
robot.move_to::<JointSpace<6>>([0.0; 6])?;
robot.control_with::<TorqueControl<7>, _>(...)?;
```

Benefits:

- The compiler can distinguish joint position and joint torque even if both are `[f64; N]`.
- A driver can implement each supported space/channel separately.
- Generic upper layers can state their required capability exactly through trait bounds.
