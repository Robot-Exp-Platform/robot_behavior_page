# 运动与控制

## 运动空间

`MotionSpace<R>` 把类型级空间映射为 `Target`：

| 空间 | Target |
|---|---|
| `JointSpace<N>` | `[f64; N]`，要求 `R: Joints<N>` |
| `EndSpace` | `Pose`，要求 `R: EndPoint` |
| `FlangeSpace`、`TcpSpace` | `Pose` |
| `Relative<S>`、`Inertial<S>` | `S::Target` |

驱动实现 `MoveTo<S>` / `MoveTraj<S>`；应用通过 blanket `Motion` 接口调用，用 turbofish 选择空间。

`move_to` 阻塞到完成或失败。`move_to_async` 返回惰性 Future，默认返回“不支持”；只有真实异步后端才应实现。它与下面“接受异步控制闭包”的契约不同。

`MoveTraj<S>` 要求实现 `move_traj`、`move_path` 和 `move_waypoints`。驱动可以把规划归约成密集轨迹，也可以对不支持的 path/waypoint 显式返回 `UnprocessableInstructionError`。仿真器的入队/注册语义需要单独查看。

## 控制通道

`ControlSpace<R>` 固定 `Obs` 与 `Command`；负载类型相同不代表通道相同。

| 通道 | Obs | Command |
|---|---|---|
| `TorqueControl<N>` | `JointState<N>` | `[f64; N]` |
| `JointPositionControl<N>` | `JointState<N>` | `[f64; N]` |
| `JointVelocityControl<N>` | `JointState<N>` | `[f64; N]` |
| `ArmTorqueControl<N>` | `ArmState<N>` | `[f64; N]` |
| `CartesianPoseControl<N>` | `ArmState<N>` | `Pose` |
| `CartesianVelocityControl<N>` | `ArmState<N>` | `[f64; 6]` |
| `BaseVelocityControl` | `BaseState` | `[f64; 6]` |

## 回调可以产生指令，也可以结束会话

驱动实现 `ControlWith<S>::control_with_flow`。以下列出关键签名，省略外围 trait 约束：

```rust
pub type ControlStep<Command> = std::ops::ControlFlow<(), (Command, bool)>;

fn control_with_flow<F>(&mut self, closure: F) -> RobotResult<()>
where
    F: FnMut(S::Obs, Duration) -> ControlStep<S::Command>;
```

| 回调返回值 | 含义 |
|---|---|
| `ControlFlow::Continue((command, false))` | 发送本次计算出的指令，继续控制。 |
| `ControlFlow::Continue((command, true))` | 发送最后一条有效指令，然后正常结束。 |
| `ControlFlow::Break(())` | 本周期没有算法指令，进入设备协议规定的会话结束流程。 |

`Break(())` 不等于急停，也不代表设备不会再发送任何协议报文。设备可以按其协议进行结束握手或发送连续性报文；不能为了填充返回值而捏造零指令。进入任一种结束路径后，驱动都不能再次调用控制闭包。

原有 `control_with` 保留：其闭包返回 `(command, done)`，默认包装为 `Continue((command, done))`，已有控制器生成函数仍可使用这个入口。闭包只属于本次调用，没有统一的 `Send + 'static` 要求，驱动不能在会话返回后继续保存它。

`hold_command(obs)` 是基于当前通道观测的连续性回退，不是通用安全命令，也不能替代设备安全机制。

## 异步闭包与阻塞会话

`control_with_flow_async` 接受 `async FnMut(...) -> ControlStep<Command>`；`control_with_async` 接受 `async FnMut(...) -> (Command, bool)`。**两者返回的仍是 `RobotResult<()>`，不是 Future；整个会话仍然阻塞。** 默认适配每周期通过 `futures::executor::block_on` 完成回调 Future。

因此，闭包可以写 async 不代表已经建立 Tokio reactor，也不保证调用线程上的定时器、网络 I/O 或其他 Future 得到推进。依赖 runtime 的操作必须核对具体驱动的执行模型。真正异步的设备会话是独立的后续设计任务。

可选图适配层见 [Roplat 集成](../guide/roplat-integration.md)。
