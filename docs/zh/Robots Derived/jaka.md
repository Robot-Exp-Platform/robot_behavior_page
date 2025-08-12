# jaka 节卡

[libjaka](https://github.com/Robot-Exp-Platform/libjaka-rs)

节卡机器人被实例化为 `JakaRobot` 在使用时需要先创建一个 `JakaRobot` 实例。

=== "rust"
    ```rust
    use libjaka::JakaRobot;

    let mut robot = JakaRobot::new("172.168.1.1");
    ```
    [libjaka-rs](https://crates.io/crates/libjaka-rs)
=== "python"
    ```python
    from libjaka import JakaRobot

    robot = JakaRobot("172.168.1.1")
    ```
    [libjaka-python](https://pypi.org/project/libjaka/)
