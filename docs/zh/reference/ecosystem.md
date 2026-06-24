# 生态与状态

## 在 Roplat 中的位置

`robot_behavior` 是驱动接口层。典型依赖关系：

```text
roplat / roplat-exp
        |
roplat_exrobot
        |
具体驱动或仿真器
        |
robot_behavior
```

## 相关实现

| crate / 目录 | 类型 | 说明 |
|---|---|---|
| `franka-rust` | 真机驱动 | Franka / Panda Rust SDK 与 PyO3 绑定 |
| `libjaka-rs` | 真机驱动 | JAKA 机械臂驱动 |
| `libhans-rs` | 真机驱动 | Hans 机械臂驱动 |
| `libaubo-rs` | 真机驱动 | Aubo 机械臂驱动 |
| `rsbullet` | 仿真 | PyBullet Rust 包装 |
| `roplat_exrobot` | 适配层 | 将外部机器人适配为 roplat 节点 |
| `roplat_rerun` | 可视化 | Rerun 与 roplat 节律对齐的发送适配 |
| `utils/rerun_urdf` | 可视化 | URDF 加载与 Rerun 推送 |

## 当前稳定性

- 核心 Rust trait 是当前主线：`Robot`、`Joints`、`EndPoint`、`Arm`、`MoveTo`、`MoveTraj`、`RealtimeControl`。
- `robot_old/` 不是新驱动的目标接口。
- FFI feature 存在，但部分宏和示例仍在迁移中；新文档不再把它作为主入口。
- 由于 `generic_const_exprs`，仍需 nightly Rust。
- `utils::trajectory` 当前依赖 `copp`，用于带速度/加速度/jerk 限位的时间参数化。

## 贡献建议

新增驱动时：

1. 先实现核心 Rust trait。
2. 明确写出限位单位和控制周期。
3. 对 Cartesian API 说明目标是 flange 还是 TCP。
4. 若使用 `MoveTraj` 的默认 path/waypoint 语义，需要说明是否接入规划器。
5. 真机相关功能应保留安全限位和急停语义。

`robot_behavior` 在 `drives` 仓内作为上游子项目存在，修改后应同步到对应上游仓库，避免只停留在父仓工作树中。
