# Overview

`robot_behavior` keeps the call surface of different robot drivers consistent while leaving each device free to implement its own transport and firmware details.

It is not a runtime and does not schedule systems. Concrete drivers, simulators and roplat own those responsibilities. This crate provides behavior traits: once a driver implements the relevant traits, upper layers can call it through the same API shape.

## Design Principles

- **Capabilities compose as traits**: a type implements only the abilities it supports.
- **Command spaces are types**: `JointSpace<6>`, `FlangeSpace` and `TcpSpace` are selected at compile time.
- **Realtime channels are types**: `TorqueControl<7>` and `JointPositionControl<7>` do not collide even if both command `[f64; 7]`.
- **Fixed DoF first**: arms use const generic `N`, so limits, Jacobians and trajectories stay statically sized.
- **Utilities are optional**: the crate offers limits, FK/IK and trajectory tools, but a driver may choose its own implementation.

## Relationship to Roplat

| Repository / directory | Relationship |
|---|---|
| `drives/franka-rust`, `libjaka-rs`, `libhans-rs`, `libaubo-rs` | real robot drivers consuming these traits |
| `drives/rsbullet` | simulation backend consuming world / physics / robot description interfaces |
| `drives/roplat_exrobot` | adapter from behavior traits to roplat nodes |
| `roplat` | scheduling and node framework, not implemented here |
| `roplat-exp` | experiment workspace using this crate through patches |

## Version Status

The crate is still evolving. The core Rust traits are the mainline API; `to_py`, `to_cxx` and `to_c` are optional FFI gates and should be checked against the current source before use.
