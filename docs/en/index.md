# Robot Behavior

This library is part of the [Universal Robot Driver Project](https://github.com/Robot-Exp-Platform/robot_behavior)! We are committed to providing Rust driver support for more robotic platforms! **Unifying driver interfaces across different robot models, reducing the learning curve for robotics, and delivering more efficient robot control solutions!**

This library is a general robot driven feature library, used to describe the characteristics of robot behavior. It provides some common feature descriptors and implementations for use by other robot driver libraries. At the same time, the signature database also implements automatic derivation macros for common interfaces, which can be used to derive secure interface implementations.

We aim to ensure consistent behavior of driver libraries across various operating platforms, as well as compatibility among different driver libraries. We hope that through the use of this library, we can minimize the learning curve associated with robot operations and achieve seamless integration.

## The principles of interface design

- **Complete interface description**. During the usage of each interface, the behavior executed by that interface should be clearly and completely expressed
- **Semantic consistency**: Function parameters/return values and function names should have consistent semantics
- **Consistent behavior**: The behavior of the interface should remain consistent across different driver libraries
