# API 映射

本页按源码模块列出当前主线公共 API。

## crate root

| 项 | 来源 | 说明 |
|---|---|---|
| `RobotException`、`RobotResult` | `exception.rs` | 统一错误类型 |
| `Robot`、`RobotDescription` | `robot/mod.rs` | 生命周期与描述 |
| `Arm`、`ArmState`、`ArmStateSample` | `robot/arm.rs` | 机械臂能力表面 |
| `Joints` | `robot/joint.rs` | 关节模型 |
| `EndPoint` | `robot/endpoint.rs` | 笛卡尔末端能力 |
| `JointSpace`、`EndSpace`、`FlangeSpace`、`TcpSpace`、`Relative`、`Inertial` | `robot/spaces.rs` | 空间 |
| `MotionSpace`、`MoveTo`、`MoveTraj`、`Motion`、`MotionFile` | `robot/motion.rs` | 运动命令 |
| `ControlSpace`、`RealtimeControl`、`Control` | `robot/control.rs` | 实时控制 |
| `Pose`、`Coord` | `robot/types.rs` | 位姿与坐标系 |
| `DhParam`、`dh_param!`、`mdh_param!` | `robot/dh.rs` | DH 参数 |
| `ArmForwardKinematics`、`ArmInverseKinematics`、`ArmKineCache` | `robot/kinematics_dynamics.rs` | FK/IK 与雅可比 |
| `LoadState` | `robot/load.rs` | 负载信息 |
| `PhysicsEngine`、`AddSearchPath` | `physics_engine.rs` | 仿真后端接口 |
| `Renderer`、`AttachFrom` | `renderer.rs` | 渲染/可视化接口 |
| `World`、`Entity`、`EntityBuilder`、`AddRobot`、`AddCollision`、`AddVisual` | `world.rs` | 场景构建接口 |

## utils

| 模块 | 公共项 | 说明 |
|---|---|---|
| `utils::types` | `homo_to_isometry`、`combine_array`、`rad_to_deg`、`to_radians_array`、`isometry_*` | 数组与 SE(3) 转换 |
| `utils::once` | `OverrideOnce`、`RobotBound`、`ArmBound` | 一次性覆盖限位 |
| `utils::limit` | `limit`、`clamp`、`limit_dot`、`difference`、`update` | 固定长度数组限位与差分 |
| `utils::path_generate` | `joint_linear`、`cartesian_quat_linear`、`joint_trapezoid`、`joint_s_curve` | 简单 path 生成 |
| `utils::trajectory` | `plan_*_via_copp`、`COPP_PATH_SAMPLES` | COPP 轨迹规划 |
| `utils::inverse` | `ik_planar_2r_all` | 平面 2R 解析 IK |
| `utils::controller` | PID、阻抗控制辅助 | 控制律工具 |

## Prelude

`robot_behavior::behavior::*` 重导出常用机器人行为项和仿真/渲染/world trait。

`robot_behavior::controller::*` 重导出 `utils::controller::pid` 与 `utils::controller::impedance`。

## 非主线目录

源码树中可能存在 `robot_old/` 或旧 FFI 宏文件，它们用于迁移和兼容，不应作为新驱动的主文档依据。新驱动优先实现本页列出的当前 Rust trait。
