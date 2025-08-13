# Introduction

本通用驱动的设计是面向对象的，不同的机器人需要先获得对应的对象实例才能进行操作。具体的实例化方法随机器人的不同而有所不同，在教程剩余部分中，统一使用 `robot` 作为对象实例的名称。

当前可以使用的对象实例包括：

样例机器人

- [exrobot](../Robots%20Derived/exrobot.md)

实物机器人

- [Franka](../Robots%20Derived/实物机器人/franka%20emika.md)
- [Hans](../Robots%20Derived/实物机器人/hans.md)
- [Jaka](../Robots%20Derived/实物机器人/jaka.md)

仿真机器人

- []

---

在完成实例化之后，可以通过调用 `robot` 对象的方法来控制机器人执行相应的操作。具体的操作方法和参数设置请参考各个机器人的文档说明。

[机器人基础接口](./机器人基础接口/RobotBehavior.md)
[机械臂相关接口](./机械臂/00%20Introduction.md)
