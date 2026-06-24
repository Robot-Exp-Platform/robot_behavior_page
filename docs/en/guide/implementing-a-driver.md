# Implementing a Driver

An arm driver normally implements traits in this order: lifecycle, physical description, motion spaces, realtime control, and finally the unified `Arm` surface.

## 1. Implement Robot

```rust
use robot_behavior::{ArmState, Robot, RobotResult};

struct MyArm {
    q: [f64; 6],
}

impl Robot for MyArm {
    type State = ArmState<6>;
    const CONTROL_PERIOD: f64 = 0.001;

    fn version() -> String {
        "my-arm 0.1".to_string()
    }

    fn read_state(&mut self) -> RobotResult<Self::State> {
        Ok(ArmState::default())
    }
}
```

`init`, `shutdown`, `enable`, `disable` and `stop` have defaults. Real drivers should override them according to the hardware semantics.

## 2. Declare Limits

```rust
use robot_behavior::{EndPoint, Joints, to_radians_array};

impl Joints<6> for MyArm {
    const JOINT_MIN: [f64; 6] = to_radians_array([-170.0; 6]);
    const JOINT_MAX: [f64; 6] = to_radians_array([170.0; 6]);
    const JOINT_VEL_BOUND: [f64; 6] = [2.0; 6];
}

impl EndPoint for MyArm {
    const CARTESIAN_VEL_BOUND: f64 = 1.0;
    const ROTATION_VEL_BOUND: f64 = 3.14;
}
```

`Joints<N>` requires only `JOINT_MIN` and `JOINT_MAX`; the rest default to `f64::MAX`.

## 3. Implement Motion Spaces

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
        let _pose = target.quat();
        // IK -> joint target -> hardware command
        Ok(())
    }
}
```

A common funnel is: `TcpSpace` applies the tool offset into `FlangeSpace`; `FlangeSpace` uses IK into `JointSpace<N>`; `JointSpace<N>` talks to the hardware.

## 4. Implement Arm

`Arm<N>` groups state, payload and temporary bound overrides. `with_*` methods commonly return `Self` with one-shot bounds stored through `OverrideOnce` / `ArmBound<N>`.

```rust
use robot_behavior::{Arm, ArmState, LoadState, Pose};

impl Arm<6> for MyArm {
    fn state(&mut self) -> RobotResult<ArmState<6>> {
        self.read_state()
    }

    fn set_load(&mut self, _load: LoadState) -> RobotResult<()> {
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

## 5. Optional Capabilities

- `MoveTraj<S>`: dense trajectories, paths and waypoints.
- `RealtimeControl<S>`: torque, position, velocity or Cartesian-velocity closure loops.
- `RobotDescription`: URDF asset path.
- `ArmForwardKinematics` / `ArmInverseKinematics`: DH, FK/IK, Jacobians and IK updates.
- `PhysicsEngine`, `AddRobot`, `Renderer`: simulation and visualization backends.
