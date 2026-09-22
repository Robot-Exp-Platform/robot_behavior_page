# Roplat 集成

当前开发源码把设备 trait 与 roplat 分开。使用 `robot_behavior::roplat::*` 时显式启用 **`robot_behavior` 的 `roplat` feature**；默认驱动构建不启用它。

## 应用使用 System 组图

应用通过 `#[roplat::system]` 组织节点和子节律域。Node 作者实现 `process`，驱动作者实现设备会话和节律适配；应用改成手写 `.process()` 调用链会绕开图生成的生命周期与执行处理。

查看与当前 checkout 对应的 [核心语法文档](https://github.com/Robot-Exp-Platform/roplat/blob/main/docs/system-syntax.md)，并结合 [roplat-skills](https://github.com/Robot-Exp-Platform/roplat-skills) 的组图规则与 [robot_behavior-skills](https://github.com/Robot-Exp-Platform/robot_behavior-skills) 的设备能力规则。不要为绕开驱动编译错误而自行发明 DSL。

## 业务数据与执行状态分离

| 适配器 | 业务输入 | 业务输出 |
|---|---|---|
| `MotionNode<R, S>` | `(RobotResult<R>, S::Target)` | `RobotResult<R>` |
| `SpaceMapNode<M, From, To>` | 模型输入 | `RobotResult<M::Output>` |
| `SafetyNode<F, Command>` | `(Command, bool)` | `(Command, bool)` |
| `ControlRhythm<R, S>` | `RobotResult<R>` | 完成执行时的 `R` |

`ControlRhythm` 的 `Yield = (S::Obs, Duration)`，业务 `Feed = (S::Command, bool)`。外层 `Execution<T>` 分别表示完成输出、协作停止、执行错误，与业务数据分开。Node 输出的普通 `Result` 不会自动让所有外层域失败。

控制适配器使用 `ControlWith::control_with_flow_async`：

- 域正常完成并给出指令/done，转换为 `Continue((command, done))`。
- 域停止或失败，没有算法指令，转换为 `Break(())`。
- 设备完成自身的会话结束协议后，节律才返回。
- 域错误与设备结束错误都应保留，后来的收尾错误不能覆盖原始失败。

这仍是**具有异步周期闭包的阻塞设备会话**。它不使传输自动成为真正异步，也不能保证同一执行器上的其他任务在会话阻塞时继续推进。有限 mock 图可以验证编排，编译通过不能证明并发真机行为或周期性能。

## 所有权与生命周期边界

`Lifecycle::on_init` / `on_shutdown` 由 Node 或 Rhythm 的创建层负责。传入子域不会重新启用或关闭已有对象；重复 drive 保留状态，需要刷新时由用户显式创建或重置。

所有协作退出路径都必须归还不透明节点元组 `N`，包括停止与错误。正在执行的域必须等待完成以收回 `N`，不能通过丢弃它实现快速失败。

**作为 `ControlRhythm::Input` 传入的机器人并不自动属于 `N`。** `Completed(R)` 会通过输出归还机器人；`Stopped` 与 `Err` 不携带 `R`，该适配器不保证这些路径会把输入机器人交回调用方。设备 session 的结束也不同于最终 `Robot::shutdown` 或创建层的生命周期钩子。`MotionNode` 同样仅在成功时通过输出归还输入机器人。需要失败后继续取得输入资源的应用，必须明确处理这一边界。

panic、丢弃外层 Future、强制 abort、不配合返回的阻塞代码不在协作归还保证之内。测试 watchdog 超时表示失败，不表示已经安全关闭。设备 hold/stop、状态有效性和仿真时序应单独验证，不能推导出统一发送零命令的策略。

## 驱动 runtime 边界

Franka 当前在阻塞的 async 回调入口内创建本地 Tokio runtime；从已进入的 Tokio runtime 调用可能因嵌套 runtime 而 panic。mock ControlRhythm/System 的成功不能证明相同调用上下文适用于 Franka 真机。此已有约束需要单独设计，本轮回调失败处理没有消除它。
