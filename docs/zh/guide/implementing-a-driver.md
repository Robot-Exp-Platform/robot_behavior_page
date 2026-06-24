# 实现驱动

一个机械臂驱动通常从小到大实现 trait。下面是当前源码对应的推荐顺序。

## 1. 实现 Robot

```rust
use robot_behavior::{ArmState, Robot, RobotResult};

struct MyArm {
    q: [f64; 6],
}

impl Robot for MyArm {
    type State = ArmState<6>;
    const CONTROL_PERIOD: f64 = 0.001;

    fn version() -> String {
        "MyArm 0.1".to_string()
    }

    fn read_state(&mut self) -> RobotResult<Self::State> {
        Ok(ArmState::default())
    }
}
```

`init`、`shutdown`、`enable`、`disable`、`stop` 等生命周期方法都有安全的 no-op 默认实现；真实驱动应按硬件语义覆盖。

## 2. 描述关节和末端限位

```rust
use robot_behavior::{EndPoint, Joints};

impl Joints<6> for MyArm {
    const JOINT_MIN: [f64; 6] = [-3.14; 6];
    const JOINT_MAX: [f64; 6] = [ 3.14; 6];
    const JOINT_VEL_BOUND: [f64; 6] = [2.0; 6];
}

impl EndPoint for MyArm {
    const CARTESIAN_VEL_BOUND: f64 = 1.0;
    const ROTATION_VEL_BOUND: f64 = 1.57;
}
```

`JOINT_MIN` / `JOINT_MAX` 必填，其余限位默认是 `f64::MAX`。

## 3. 实现运动空间

```rust
use robot_behavior::{FlangeSpace, JointSpace, MoveTo, Pose, RobotResult};

impl MoveTo<JointSpace<6>> for MyArm {
    fn move_to(&mut self, target: [f64; 6]) -> RobotResult<()> {
        self.q = target;
        Ok(())
    }
}

impl MoveTo<FlangeSpace> for MyArm {
    fn move_to(&mut self, target: Pose) -> RobotResult<()> {
        let _target = target;
        // 通常：Pose -> IK -> JointSpace -> hardware
        Ok(())
    }
}
```

`MoveTraj<S>` 是轨迹入口。若硬件只接受单点目标，可以在驱动中采样后逐点下发；若硬件支持原生轨迹，则直接映射到厂商 SDK。

## 4. 实现 Arm<N>

`Arm<N>` 统一状态、负载和临时限位覆盖：

```rust
use robot_behavior::{Arm, ArmState, LoadState, Pose};

impl Arm<6> for MyArm {
    fn state(&mut self) -> RobotResult<ArmState<6>> {
        self.read_state()
    }

    fn set_load(&mut self, load: LoadState) -> RobotResult<()> {
        let _ = load;
        Ok(())
    }

    fn get_joint(&self) -> [f64; 6] { self.q }
    fn get_endpoint(&self) -> Pose { Pose::default() }

    fn with_joint_vel(self, _v: [f64; 6]) -> Self { self }
    fn with_joint_acc(self, _a: [f64; 6]) -> Self { self }
    fn with_joint_jerk(self, _j: [f64; 6]) -> Self { self }
    fn with_torque(self, _t: [f64; 6]) -> Self { self }
    fn with_torque_dot(self, _td: [f64; 6]) -> Self { self }
    fn with_cartesian_vel(self, _v: f64) -> Self { self }
    fn with_cartesian_acc(self, _a: f64) -> Self { self }
    fn with_cartesian_jerk(self, _j: f64) -> Self { self }
    fn with_rotation_vel(self, _v: f64) -> Self { self }
    fn with_rotation_acc(self, _a: f64) -> Self { self }
    fn with_rotation_jerk(self, _j: f64) -> Self { self }
}
```

真实驱动通常用 `OverrideOnce` 存储 `with_*` 的一次性覆盖值。

## 5. 实现运动学

若驱动需要 FK/IK：

```rust
use robot_behavior::{ArmForwardKinematics, DhParam, dh_param};

impl ArmForwardKinematics<6> for MyArm {
    const DH: [DhParam; 6] = [dh_param!(0.0, 0.0, 0.0, 0.0); 6];
}
```

如果有解析 IK，可实现 `ArmInverseKinematics::ik_analytic_all`；否则可以复用默认的 DLS、JT、Newton、LM 单步更新。
