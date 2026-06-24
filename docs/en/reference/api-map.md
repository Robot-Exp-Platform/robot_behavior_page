# API Map

This page lists the current public API by source module. Most items are re-exported at the crate root and common behavior items are also available through `robot_behavior::behavior::*`.

## Crate Root

| Item | Description |
|---|---|
| `RobotException`, `RobotResult<T>` | unified error and result alias |
| `Robot`, `RobotDescription` | lifecycle and URDF description |
| `behavior` | behavior prelude |
| `controller` | controller utility prelude |
| `roplat_data_dir()` | roplat asset directory helper |

## robot

| Module | Public items |
|---|---|
| `arm` | `Arm`, `ArmState`, `ArmStateSample` |
| `joint` | `Joints` |
| `endpoint` | `EndPoint` |
| `motion` | `MotionSpace`, `MoveTo`, `MoveTraj`, `Motion`, `MotionFile` |
| `control` | `ControlSpace`, `RealtimeControl`, `Control`, `TorqueControl`, `JointPositionControl`, `JointVelocityControl`, `CartesianVelocityControl` |
| `types` | `Coord`, `Pose` |
| `dh` | `DhParam`, `dh_param!`, `mdh_param!` |
| `kinematics_dynamics` | `Iso3`, `Twist`, `Wrench`, `JVec`, `JMat`, `Jaco`, `Link`, `Joint`, `JointType`, `ArmKineCache`, `ArmForwardKinematics`, `ArmInverseKinematics`, `IKMethod`, `CommonStop`, `AnalyticFamily`, `ArmDynamics` |
| `load` | `LoadState` |
| `spaces` | `JointSpace`, `EndSpace`, `FlangeSpace`, `TcpSpace`, `BasePoseSpace`, `BaseVelocitySpace`, `WholeBodyJointSpace`, `WholeBodyVelocitySpace`, `WholeBodyTorqueSpace`, `CenterOfMassSpace`, `GaitCommand`, `GaitSpace`, `FootSpace`, `HandSpace`, `JacobianSpace`, `MassMatrixSpace`, `JointTorqueSpace`, `CoriolisInputSpace`, `GravityInputSpace`, `Relative`, `Inertial` |

## utils

| Module | Description |
|---|---|
| `types` | `homo_to_isometry`, `combine_array`, `rad_to_deg`, `to_radians_array`, isometry raw/frame helpers |
| `once` | `OverrideOnce`, `RobotBound`, `ArmBound` |
| `limit` | `limit`, `clamp`, `limit_dot`, `difference`, `update` |
| `path_generate` | `joint_linear`, `cartesian_quat_linear`, `joint_trapezoid`, `t_curve`, `joint_s_curve` |
| `trajectory` | `plan_s_t_via_copp`, `plan_waypoints_traj_via_copp`, `plan_path_traj_via_copp` |
| `inverse` | `ik_planar_2r_all` |
| `controller::impedance` | joint-space impedance helpers |
| `controller::pid` | PID helpers |

## world / physics / renderer

| Item | Description |
|---|---|
| `PhysicsEngine`, `AddSearchPath` | simulation backend interfaces |
| `World`, `Entity`, `EntityBuilder`, `AddRobot`, `AddCollision`, `AddVisual` | world entity loading interfaces |
| `Renderer`, `AttachFrom<T>` | visualization / rendering adapters |

## FFI

`ffi`, `to_py`, `to_cxx` and `to_c` are feature-gated adapters. They are not required by the current core Rust API and should be checked separately with the target feature before use.
