# Hans 华沿（原名大族）

[libhans](https://github.com/Robot-Exp-Platform/libhans-rs)

大族机器人被实例化为 `HansRobot` 在使用时需要先创建一个 `HansRobot` 实例。

=== "rust"
    ```rust
    use libhans::HansRobot;

    let mut robot = HansRobot::new("172.168.1.1");
    ```
    [libhans](https://crates.io/crates/libhans)
=== "python"
    ```python
    from libhans import HansRobot

    robot = HansRobot("172.168.1.1")
    ```
    [libhans](https://pypi.org/project/libhans/)
