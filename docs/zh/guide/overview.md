# 总览

`robot_behavior` 位于驱动层的“接口契约”位置：

```text
应用 / roplat 节点
        |
具体驱动或仿真器（franka-rust / libjaka-rs / rsbullet / roplat_exrobot）
        |
robot_behavior trait 与工具
```

它的目标是让不同机器人在 Rust 侧拥有相同的行为语义：生命周期、状态读取、运动命令、实时控制、运动学、负载配置和仿真/可视化接入。

## 不做什么

`robot_behavior` 不负责：

- 连接某个厂商 SDK。
- 编写轨迹规划业务逻辑。
- 处理 roplat 系统调度。
- 直接保证真机安全策略。

这些职责分别属于具体驱动、实验工程、roplat 主仓和设备安全层。

## 当前主 API

| 层级 | 关键项 | 作用 |
|---|---|---|
| 生命周期 | `Robot` | 初始化、使能、停止、状态读取、控制周期 |
| 模型参数 | `Joints<N>`、`EndPoint`、`RobotDescription` | 关节限位、末端限位、URDF 信息 |
| 机械臂表面 | `Arm<N>`、`ArmState<N>`、`LoadState` | 统一机械臂状态、负载、临时限位覆盖 |
| 运动命令 | `MotionSpace`、`MoveTo`、`MoveTraj`、`Motion` | 单点、轨迹、路径、waypoint 命令 |
| 实时控制 | `ControlSpace`、`RealtimeControl`、`Control` | 闭包式控制循环 |
| 运动学 | `Pose`、`DhParam`、`ArmKineCache`、`IKMethod` | SE(3)、DH、FK、IK、雅可比 |
| 场景扩展 | `PhysicsEngine`、`Renderer`、`World` | 仿真、渲染、实体构建接口 |

## 为什么使用类型级空间

旧接口把命令塞进 `MotionType` / `ControlType` 枚举。当前实现改为：

```rust
robot.move_to::<JointSpace<6>>([0.0; 6])?;
robot.move_to::<FlangeSpace>(Pose::Position([0.4, 0.0, 0.3]))?;
robot.control_with_closure::<TorqueControl<7>, _>(|state, dt| { /* ... */ })?;
```

这样做的好处是：同样都是 `[f64; N]` 的命令，关节位置、关节速度、关节力矩可以通过类型区分；驱动也可以为不同空间分别实现能力，而不是在一个枚举分支里做运行时检查。
