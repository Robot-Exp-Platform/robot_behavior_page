# Ecosystem and Status

## Position in Roplat

`robot_behavior` sits at the bottom of the Roplat driver layer and provides shared interfaces. It usually has three kinds of consumers:

1. **Real robot drivers**: adapt vendor SDKs into common traits.
2. **Simulation and visualization**: use `PhysicsEngine`, `World`, `Renderer` and robot description interfaces.
3. **roplat adapters**: wrap driver behavior as roplat nodes or experiment components.

## Known Related Crates / Directories

| Directory | Role |
|---|---|
| `franka-rust` | Franka / Panda real arm driver |
| `libjaka-rs` | JAKA arm driver |
| `libhans-rs` | Hans arm driver |
| `libaubo-rs` | AUBO interface scaffold; several operations are not implemented |
| `rsbullet` | PyBullet / Bullet simulation wrapper |
| `roplat_exrobot` | mock/example robot and foreign-language demonstration |
| `roplat_rerun` | Rerun visualization adapter |
| `utils/rerun_urdf` | URDF loading and Rerun logging tool |

## Not Mainline Anymore

The source tree may keep `robot_old` or older FFI examples for migration reference. New code should prefer:

- `Robot`
- `Joints<N>` / `EndPoint` / `Arm<N>`
- `MotionSpace` / `MoveTo` / `MoveTraj` / `Motion`
- `ControlSpace` / `ControlWith` / `Control`

## Stability Notes

The current version is `0.6.x`, and trait details may still change across minor versions. Before submitting a driver, verify at least:

```powershell
cargo check -p robot_behavior --no-default-features
cargo check -p robot_behavior --features roplat
cargo test -p robot_behavior --lib
```

If a public trait changes, also check real drivers, simulators and `roplat-exp` consumers.

These are verification commands, not a claim that every platform, feature or device was tested. The current alignment preserves blocking 0.6 control sessions. Truly asynchronous sessions, state-validity redesign and simulator timing are separate tasks. For adapter limits, see [Roplat integration](../guide/roplat-integration.md).
