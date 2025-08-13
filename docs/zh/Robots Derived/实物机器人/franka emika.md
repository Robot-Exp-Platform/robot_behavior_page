# Franka

[franka](https://github.com/Robot-Exp-Platform/franka-rust)

Franka 机器人被实例化为 `FrankaRobot` 在使用时需要先创建一个 `FrankaRobot` 实例。

=== "rust"
    ```rust
    use libfranka::FrankaRobot;

    let mut robot = FrankaRobot::new("172.168.1.1");
    ```
    [libfranka-rs](https://crates.io/crates/libfranka)
=== "python"
    ```python
    from libfranka import FrankaRobot

    robot = FrankaRobot("172.168.1.1")
    ```
    [libfranka-python](https://pypi.org/project/libfranka/)
