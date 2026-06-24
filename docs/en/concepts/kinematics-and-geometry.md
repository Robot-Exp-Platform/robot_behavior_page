# Geometry and Kinematics

## Pose

`Pose` represents a rigid-body pose in several equivalent forms:

- `Euler([x, y, z], [roll, pitch, yaw])`
- `Quat(Isometry3<f64>)`
- `Homo([f64; 16])`
- `AxisAngle([x, y, z], [ax, ay, az], angle)`
- `Position([x, y, z])`

Common conversions:

```rust
use robot_behavior::Pose;

let pose = Pose::Position([0.4, 0.0, 0.3]);
let iso = pose.quat();
let homo = pose.homo();
let xyz = pose.position();
```

## DH Parameters

`DhParam` supports standard DH, modified DH, and direct `Iso3Quat` / `Iso3RPY` transforms:

```rust
use robot_behavior::{DhParam, dh_param, mdh_param};

let a: DhParam = dh_param! {theta: 0.0, d: 0.0, r: 0.3, alpha: 0.0};
let b: DhParam = mdh_param! {0.0, 0.0, 0.3, 0.0};
```

`to_se3(q)` applies the joint variable to the link transform.

## FK Cache

`ArmKineCache<N>` caches, in one build step:

- `t_prefix[0..=N]`: cumulative transforms from base to each link.
- `origins` / `z_axes`: link origins and z axes.
- `joint_z` / `joint_p`: joint axes and axis origins.
- `jacobian()`: geometric end-effector Jacobian.
- `ee_twist()`: end-effector twist from `q_dot`.

`ArmForwardKinematics<N>` only requires `const DH: [DhParam; N]` and then provides default `kine_cache` and `fk_end_pose` methods.

## IK Methods

`ArmInverseKinematics<N>` provides:

- `ANALYTIC_FAMILY`: optional analytic family marker.
- `ik_analytic_all` / `ik_analytic_best`: analytic solver entry points.
- `task_error_and_jacobian`: default geometric residual and Jacobian.
- `ik_step`: one DLS, JT, Newton, LM or analytic-fallback update.

`utils::inverse::ik_planar_2r_all` is a compact planar 2R analytic IK example.

## Trajectory Utilities

- `utils::path_generate`: linear, trapezoid and S-curve paths.
- `utils::limit`: clamping, rate limits, finite differences and integration update.
- `utils::trajectory`: copp-based waypoint/path retiming.
