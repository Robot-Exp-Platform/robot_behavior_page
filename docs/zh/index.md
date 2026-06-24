# Robot Behavior

`robot_behavior` 是 Roplat 驱动体系的行为契约 crate。它定义“机器人应该暴露什么能力”，而不是直接实现某一个硬件驱动。

当前实现已经从旧的运行时 `MotionType` / `ControlType` 枚举，切换为类型级空间模型：

- `Robot` 管生命周期和基础状态读取。
- `Joints<N>` / `EndPoint` 管模型级限位。
- `MotionSpace` / `MoveTo` / `MoveTraj` 管预规划运动。
- `ControlSpace` / `RealtimeControl` 管闭包式实时控制。
- `Pose`、`DhParam`、`ArmKineCache`、`IKMethod` 管几何和运动学。
- `utils` 提供限位、插值、COPP 轨迹规划、PID/阻抗控制等辅助工具。

```rust
use robot_behavior::{JointSpace, Motion, MoveTo, RobotResult};

fn home<R>(robot: &mut R) -> RobotResult<()>
where
    R: MoveTo<JointSpace<6>>,
{
    robot.move_to::<JointSpace<6>>([0.0; 6])
}
```

## 阅读路径

1. [总览](guide/overview.md)：了解 crate 的职责边界。
2. [安装与特性](guide/install.md)：确认 nightly、feature 和 workspace 用法。
3. [使用驱动](guide/using-a-driver.md)：从用户角度调用统一运动/控制接口。
4. [实现驱动](guide/implementing-a-driver.md)：从驱动作者角度实现 trait。
5. [能力模型](concepts/capability-model.md)：理解 trait 分层。
6. [运动与控制](concepts/motion-and-control.md)：理解类型级空间。
7. [几何与运动学](concepts/kinematics-and-geometry.md)：理解 `Pose`、DH、FK/IK 与轨迹规划。
8. [API 映射](reference/api-map.md)：按源码模块查找公共项。
9. [生态与状态](reference/ecosystem.md)：了解相关驱动、仿真器和当前状态。

## 当前边界

核心 Rust API 是当前主线。`ffi` / `to_py` / `to_cxx` / `to_c` 是可选门控，部分适配层仍处于迁移期；写新驱动时优先实现 Rust trait，再按需要补外语绑定。
