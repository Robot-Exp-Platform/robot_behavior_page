# 机器人基础接口

以下的接口为机器人基础接口，主要为基础的上下电上下使能等操作，不包含危险指令，但是并非所有的机器人厂商都支持完整的操作流程。一般认为的机器人操作流程见下图：

![机器人操作流程](../asserts/机器人运行流程.png)

- `version`

获取驱动版本和机器人版本，一般来说是一样的，但是因厂商而异。如 Franka 存在[版本兼容管理](https://www.franka.cn/FCI/compatibility.html)， 系统版本、驱动版本、机器人版本应当挂钩，否则会出现指令问题。

=== "Rust"
    ```rust
    pub fn version(&self) -> String
    ```

=== "Python"
    ```python
    def version(self) -> str: ...
    ```

=== "C++"
    ```cpp
    std::string version()
    ```

- `init`

初始化机器人的操作，一般来说初始化完成的工作主要在于驱动和控制柜的初始化，初始化后的机器人一般是去使能的，避免突然启动导致的安全问题。

=== "Rust"
    ```rust
    pub fn init(&mut self) -> RobotResult<()>
    ```

=== "Python"
    ```python
    def init(self) -> None: ...
    ```

=== "C++"
    ```cpp
    void init()
    ```

- `shutdown`

关闭机器人的操作，一般来说关闭完成的工作主要在于驱动和控制柜的关闭，关闭过程中会自动对机器人去使能，但是为了安全还是建议在关闭前手动去使能。

=== "Rust"
    ```rust
    pub fn shutdown(&mut self) -> RobotResult<()>
    ```

=== "Python"
    ```python
    def shutdown(self) -> None: ...
    ```

=== "C++"
    ```cpp
    void shutdown()
    ```

- `enable`

使能机器人的操作，使能后的机器人会解开关节锁，此时可能会导致安全问题，使能过程的机器人一般应该保证周围空间环境的安全。使能后机器人可以进行运动操作。

=== "Rust"
    ```rust
    pub fn enable(&mut self) -> RobotResult<()>
    ```

=== "Python"
    ```python
    def enable(self) -> None: ...
    ```

=== "C++"
    ```cpp
    void enable()
    ```

- `disable`

去使能机器人的操作，去使能后的机器人会上锁关节，此时机器人会停止运动操作。

=== "Rust"
    ```rust
    pub fn disable(&mut self) -> RobotResult<()>
    ```

=== "Python"
    ```python
    def disable(self) -> None: ...
    ```

=== "C++"
    ```cpp
    void disable()
    ```

- `reset`

重置机器人的操作，重置后的机器人会回到初始状态，恢复初始状态过程中可能会导致运动安全问题，部分运动行为要求在重置后才能执行。

=== "Rust"
    ```rust
    pub fn reset(&mut self) -> RobotResult<()>
    ```

=== "Python"
    ```python
    def reset(self) -> None: ...
    ```

=== "C++"
    ```cpp
    void reset()
    ```

- `is_moving`

回答机器人是否在运动的问题，大多数机器人不能同时执行多个运动指令和控制指令，此时就需要判断机器人运动状态作为指令执行条件。

=== "Rust"
    ```rust
    pub fn is_moving(&mut self) -> bool
    ```

=== "Python"
    ```python
    def is_moving(self) -> bool: ...
    ```

=== "C++"
    ```cpp
    bool is_moving()
    ```

- `stop`

停止机器人的运动，停止后的机器人会立即停止运动，本指令并不会自动去使能

=== "Rust"
    ```rust
    pub fn stop(&mut self) -> RobotResult<()>
    ```

=== "Python"
    ```python
    def stop(self) -> None: ...
    ```

=== "C++"
    ```cpp
    void stop()
    ```

- `resume`

恢复运动状态，由 `stop` 指令停止后的机器人可以通过 `resume` 恢复运动，但是触发 `emergency_stop` 后的机器人需要先 `clear_emergency_stop` 才能恢复运动。

=== "Rust"
    ```rust
    pub fn resume(&mut self) -> RobotResult<()>
    ```

=== "Python"
    ```python
    def resume(self) -> None: ...
    ```

=== "C++"
    ```cpp
    void resume()
    ```

- `emergency_stop`

急停！急停！急停！无论是来源于主动控制还是由于机器人自身的安全保护，这都代表机器人遇见了无法处理的问题，一般来说会对机器人去使能，部分机器人会直接下电。触发急停后的机器人需要先 `clear_emergency_stop` 才能恢复运动。

=== "Rust"
    ```rust
    pub fn emergency_stop(&mut self) -> RobotResult<()>
    ```

=== "Python"
    ```python
    def emergency_stop(self) -> None: ...
    ```

=== "C++"
    ```cpp
    void emergency_stop()
    ```

- `clear_emergency_stop`

=== "Rust"
    ```rust
    pub fn clear_emergency_stop(&mut self) -> RobotResult<()>
    ```

=== "Python"
    ```python
    def clear_emergency_stop(self) -> None: ...
    ```

=== "C++"
    ```cpp
    void clear_emergency_stop()
    ```

- `read_state`

读取由机器人厂商提供的原生状态结构体

=== "Rust"
    ```rust
    pub fn read_state(&mut self) -> RobotResult<Self::State>
    ```

=== "Python"
    ```python
    def read_state(self) -> Self.State: ...
    ```

=== "C++"
    ```cpp
    Self::State read_state()
    ```
