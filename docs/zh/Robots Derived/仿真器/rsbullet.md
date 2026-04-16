# RsBullet

[rsbullet](https://github.com/Robot-Exp-Platform/rsbullet)

RsBullet 是基于 Bullet Physics 的 Rust 物理仿真引擎封装。它实现了 `robot_behavior` 中的 `PhysicsEngine`、`AddRobot`、`AddCollision`、`AddVisual` 等特征，可以在仿真环境中加载和控制机器人。

## 创建仿真引擎

=== "Rust"
    ```rust
    use rsbullet::{RsBullet, Mode};

    // GUI 模式（带可视化窗口）
    let mut engine = RsBullet::new(Mode::Gui)?;

    // Direct 模式（无 GUI，纯物理计算）
    let mut engine = RsBullet::new(Mode::Direct)?;
    ```

### 连接模式

`Mode` 枚举定义了仿真引擎的运行模式：

| 模式 | 说明 |
|------|------|
| `Mode::Direct` | 无 GUI，纯物理计算，性能最高 |
| `Mode::Gui` | 带 OpenGL 可视化窗口 |
| `Mode::SharedMemory` | 共享内存模式（默认） |
| `Mode::Tcp { .. }` / `Mode::Udp { .. }` | 网络连接模式 |
| `Mode::GuiServer` | GUI + 服务器模式 |

## 已实现的通用接口

### PhysicsEngine — 物理引擎核心

=== "Rust"
    ```rust
    use robot_behavior::PhysicsEngine;
    use std::time::Duration;

    // 设置物理参数
    engine.set_gravity([0., 0., -9.8])?
          .set_step_time(Duration::from_secs_f64(1.0 / 240.0))?;

    // 仿真循环
    loop {
        engine.step()?;  // 推进一步仿真
    }

    // 重置仿真
    engine.reset()?;

    // 关闭
    engine.shutdown();
    ```

`PhysicsEngine` 接口：

| 方法 | 说明 |
|------|------|
| `reset()` | 重置仿真（清空所有命令和物体） |
| `step()` | 推进一步仿真（处理排队命令 + 执行物理步进） |
| `shutdown()` | 关闭引擎 |
| `set_step_time(Duration)` | 设置仿真步长（默认 1/240 秒） |
| `set_gravity([f64; 3])` | 设置重力加速度 |

### AddRobot — 加载机器人

通过 Builder 模式加载 URDF 格式的机器人模型。机器人类型需要实现 `RobotFile` 特征（提供 URDF 路径）和 `ArmParam<N>` 特征（提供运动学参数）。其中配置指令为可选项：

=== "Rust"
    ```rust
    use robot_behavior::{AddRobot, EntityBuilder};
    use nalgebra as na;

    // 使用 robot_builder 加载机器人
    let mut robot = engine
        .robot_builder::<MyRobot>("robot1")
        .base(na::Isometry3::identity())  // 基座位姿
        .base_fixed(true)                  // 固定基座
        .scaling(1.0)                      // 缩放比例
        .load()?;
    ```

Builder 方法链：

| 方法 | 说明 |
|------|------|
| `base(Isometry3)` | 设置基座位姿 |
| `base_fixed(bool)` | 是否固定基座 |
| `scaling(f64)` | 模型缩放 |
| `name(String)` | 设置名称（当前未使用） |
| `use_maximal_coordinates(bool)` | 是否使用最大坐标模式 |
| `flags(LoadModelFlags)` | 加载选项标志 |
| `load()` | 执行加载，返回 `RsBulletRobot<R>` |

`LoadModelFlags` 常用标志：

- `URDF_ENABLE_CACHED_GRAPHICS_SHAPES` — 缓存图形网格，减少重复加载
- `URDF_USE_INERTIA_FROM_FILE` — 使用 URDF 文件中的惯量参数

### AddCollision & AddVisual — 添加碰撞体与可视体

=== "Rust"
    ```rust
    use robot_behavior::{AddCollision, AddVisual, Entity, EntityBuilder};

    let mesh_scale = [0.1, 0.1, 0.1];

    // 添加可视形状
    let visual = engine
        .visual(Entity::MeshFile { file: "duck.obj", scale: mesh_scale })
        .base([0., -0.02, 0.])
        .load()?;

    // 添加碰撞形状
    let collision = engine
        .collision(Entity::MeshFile { file: "duck_vhacd.obj", scale: mesh_scale })
        .base([0., -0.02, 0.])
        .load()?;
    ```

### AddSearchPath — 资源搜索路径

=== "Rust"
    ```rust
    use robot_behavior::AddSearchPath;

    engine.add_search_path("path/to/assets")?;
    ```

## 仿真机器人接口

加载后的 `RsBulletRobot<R>` 自动实现以下通用机械臂接口：

| 特征 | 说明 |
|------|------|
| `Arm<N>` | 机械臂基础接口（state 读取关节状态） |
| `ArmParam<N>` | 关节参数（代理到加载时指定的机器人类型 `R`） |
| `ArmPreplannedMotion<N>` | 预规划运动（move_joint/move_cartesian，内部通过 S 曲线/4 次曲线规划） |
| `ArmPreplannedPath<N>` | 路径运动（move_traj 逐帧执行） |
| `ArmRealtimeControl<N>` | 闭包实时控制 |
| `ArmRealtimeControlExt<N>` | 闭包控制扩展 |

!!! note
    `ArmStreamingMotion` 尚未实现（代码中已注释）。`set_coord`、`set_scale` 等参数设置方法当前为空操作。

### 使用样例

=== "Rust"
    ```rust
    use rsbullet::{RsBullet, Mode};
    use robot_behavior::*;
    use std::time::Duration;
    use nalgebra as na;

    // 1. 创建引擎
    let mut engine = RsBullet::new(Mode::Gui)?;
    engine
        .set_gravity([0., 0., -9.8])?
        .set_step_time(Duration::from_secs_f64(1.0 / 240.0))?;

    // 2. 加载机器人
    let mut robot = engine
        .robot_builder::<MyRobot>("panda")
        .base(na::Isometry3::identity())
        .base_fixed(true)
        .load()?;

    // 3. 预规划运动
    robot.move_joint(&[0.0, 0.5, 0.0, -1.0, 0.0, 1.5, 0.0])?;

    // 4. 闭包实时控制
    let q0 = robot.state()?.joint.unwrap();
    robot.move_joint_with_closure(move |state, duration| {
        let t = duration.as_secs_f64();
        let mut q = q0;
        q[0] += 0.1 * (2.0 * std::f64::consts::PI * t).sin();
        (q, t > 5.0)
    })?;

    // 5. 仿真循环（需要持续调用 step 驱动物理引擎）
    for _ in 0..10000 {
        engine.step()?;
    }

    // 6. 关闭
    engine.shutdown();
    ```

### 命令执行机制

`RsBulletRobot` 的运动命令不会立即执行，而是通过内部的命令队列（channel）排队，在每次 `engine.step()` 时被处理。这意味着：

1. `move_joint_async` 等异步方法会将轨迹闭包入队后立即返回
2. 需要在主循环中持续调用 `engine.step()` 驱动仿真和命令执行
3. 同步方法（如 `move_joint`）会阻塞直到运动完成

### 底层访问

通过 `body_id` 和 `joint_indices` 字段可直接访问 Bullet 底层：

=== "Rust"
    ```rust
    // 获取 Bullet 物体 ID
    let body_id = robot.body_id;

    // 获取可控关节索引列表
    let joints = &robot.joint_indices;

    // 获取末端执行器 link 索引
    let ee_link = robot.end_effector_link;
    ```

## 更多接口

你可以在 `Rsbullet.client` 访问物理引擎，在其中包含与 `Pybullet` 中完全一致的接口，这容许你完成与之类似的所有功能。可以参考 [Pybullet 官方文档](https://pybullet.org/wordpress/) 来使用这些接口。
