# Robot Behavior

This library is part of the [Universal Robot Driver Project](https://github.com/Robot-Exp-Platform/robot_behavior)! We are committed to providing Rust driver support for more robotic platforms! **Unifying driver interfaces across different robot models, reducing the learning curve for robotics, and delivering more efficient robot control solutions!**

This library is a general robot driven feature library, used to describe the characteristics of robot behavior. It provides some common feature descriptors and implementations for use by other robot driver libraries. At the same time, the signature database also implements automatic derivation macros for common interfaces, which can be used to derive secure interface implementations.

We aim to ensure consistent behavior of driver libraries across various operating platforms, as well as compatibility among different driver libraries. We hope that through the use of this library, we can minimize the learning curve associated with robot operations and achieve seamless integration.

## The principles of interface design

- **Complete interface description**. During the usage of each interface, the behavior executed by that interface should be clearly and completely expressed
- **Semantic consistency**: Function parameters/return values and function names should have consistent semantics
- **Consistent behavior**: The behavior of the interface should remain consistent across different driver libraries

## How to use the robot driven by this library

[robot_behavior](https://robot-exp-platform.github.io/robot_behavior_page/)。

All robots derived from this library follow the same interface specification. For consistency, we usually name the instantiated driver `robot`, and the rest of this document assumes that name.

Here is a minimal example of commanding a robot:

```rust
robot.move_to(MotionType::Joint([0.; 6]))?;
robot.move_to(MotionType::Cartensian(Pose::Euler([0; 3], [0.; 3])))?;
```

We also provide helper shortcuts that perform the same actions:

```rust
robot.move_joint([0.; 6])?;
robot.move_cartesian_euler([0; 3], [0.; 3])?;
```

Conceptually, the interfaces fall into three categories:

1. **Preplanned interfaces** – trajectories are known at the time of submission.
2. **Streaming interfaces** – trajectories are generated online and streamed while the robot is moving.
3. **Closure/real-time interfaces** – control commands are computed inside a user-provided closure during execution.

Together these categories cover most control scenarios. We are continuously adding support for more robots and friendlier APIs. See [robots](https://robot-exp-platform.github.io/robot_behavior_page/) for the latest list, and reach out if you have a new driver to add.
