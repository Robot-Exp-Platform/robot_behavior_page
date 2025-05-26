# 04 机械臂流式控制接口

TODO 将 `ArmStreamingMotion` 接口整理为更详细的文档

=== "Python"
    ```python
    class ArmStreamingMotion:
        """
        ArmStreamingMotion

        Streaming motion interface for robot arm, supports starting/stopping streaming and accessing shared targets.
    
        机械臂流式运动接口，支持流式运动的启动/停止及目标共享对象的访问。
        """
        def start_streaming(self) -> 'ArmStreamingHandle':
            """
            Start streaming motion.
    
            开始流式运动。
            """
            ...
        def end_streaming(self) -> None:
            """
            End streaming motion.
    
            结束流式运动。
            """
            ...
        def move_to_target(self) -> object:
            """
            Get the shared object for motion target.
    
            获取运动目标的共享对象。
            """
            ...
        def control_with_target(self) -> object:
            """
            Get the shared object for control target.
    
            获取控制目标的共享对象。
            """
            ...
    
    class ArmStreamingMotionExt:
        """
        ArmStreamingMotionExt
    
        Extension interface for streaming motion, provides access to various shared targets.
    
        机械臂流式运动扩展接口，提供多种目标的共享对象访问。
        """
        def move_joint_target(self) -> object:
            """
            Get the shared object for joint target.
    
            获取关节目标的共享对象。
            """
            ...
        def move_joint_vel_target(self) -> object:
            """
            Get the shared object for joint velocity target.
    
            获取关节速度目标的共享对象。
            """
            ...
        def move_joint_acc_target(self) -> object:
            """
            Get the shared object for joint acceleration target.
    
            获取关节加速度目标的共享对象。
            """
            ...
        def move_cartesian_target(self) -> object:
            """
            Get the shared object for Cartesian target.
    
            获取笛卡尔目标的共享对象。
            """
            ...
        def move_cartesian_vel_target(self) -> object:
            """
            Get the shared object for Cartesian velocity target.
    
            获取笛卡尔速度目标的共享对象。
            """
            ...
        def move_cartesian_euler_target(self) -> object:
            """
            Get the shared object for Euler angle target.
    
            获取欧拉角目标的共享对象。
            """
            ...
        def move_cartesian_quat_target(self) -> object:
            """
            Get the shared object for quaternion target.
    
            获取四元数目标的共享对象。
            """
            ...
        def move_cartesian_homo_target(self) -> object:
            """
            Get the shared object for homogeneous matrix target.
    
            获取齐次矩阵目标的共享对象。
            """
            ...
        def control_tau_target(self) -> object:
            """
            Get the shared object for torque target.
    
            获取力矩目标的共享对象。
            """
            ...
    ```

## 流式控制句柄

=== "Python"
    ```python
    class ArmStreamingHandle:
        """
        ArmStreamingHandle

        Streaming handle for robot arm, provides access to last motion/control and allows setting new targets.
    
        机械臂流式运动句柄，提供上一个运动/控制目标的访问与新目标的设置。
        """
        def last_motion(self) -> 'MotionType':
            """
            Get the last motion target.
    
            获取上一个运动目标。
            """
            ...
        def move_to(self, target: 'MotionType') -> None:
            """
            Set a new motion target for streaming.
    
            设置新的流式运动目标。
            """
            ...
        def last_control(self) -> 'ControlType':
            """
            Get the last control target.
    
            获取上一个控制目标。
            """
            ...
        def control_with(self, control: 'ControlType') -> None:
            """
            Set a new control target for streaming.
    
            设置新的流式控制目标。
            """
            ...
    ```
