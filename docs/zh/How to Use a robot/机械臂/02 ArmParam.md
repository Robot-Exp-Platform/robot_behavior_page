# 02 机械臂参数接口及派生函数

TODO 将 `ArmParam` 接口整理为文档形式

=== "Rust"
    ```rust
    pub trait ArmParam<const N: usize> {
        const DH: [[f64; 4]; N];
        const JOINT_DEFAULT: [f64; N] = [f64::MAX; N];
        const JOINT_MIN: [f64; N];
        const JOINT_MAX: [f64; N];
        const JOINT_VEL_BOUND: [f64; N] = [f64::MAX; N];
        const JOINT_ACC_BOUND: [f64; N] = [f64::MAX; N];
        const JOINT_JERK_BOUND: [f64; N] = [f64::MAX; N];
        const CARTESIAN_VEL_BOUND: f64 = f64::MAX;
        const CARTESIAN_ACC_BOUND: f64 = f64::MAX;
        const CARTESIAN_JERK_BOUND: f64 = f64::MAX;
        const ROTATION_VEL_BOUND: f64 = f64::MAX;
        const ROTATION_ACC_BOUND: f64 = f64::MAX;
        const ROTATION_JERK_BOUND: f64 = f64::MAX;
        const TORQUE_BOUND: [f64; N] = [f64::MAX; N];
        const TORQUE_DOT_BOUND: [f64; N] = [f64::MAX; N];

        #[inline(always)]
        fn forward_kinematics(q: &[f64; N]) -> Pose {
            let mut isometry = na::Isometry3::identity();
            for (dh, q) in Self::DH.iter().zip(q) {
                let (s, c) = dh[3].sin_cos();
                let translation = na::Translation3::new(dh[2], -dh[1] * s, dh[1] * c);
                let rotation = na::UnitQuaternion::from_euler_angles(*q, 0.0, dh[3]);
                let isometry_increment = na::Isometry::from_parts(translation, rotation);
                isometry *= isometry_increment;
            }
            Pose::Quat(isometry)
        }
    
        #[deprecated(
            note = "inverse kinematics is not implemented because the representation method of the robotic arm configuration has not    been determined"
        )]
        #[inline(always)]
        fn inverse_kinematics(_pose: Pose) -> RobotResult<[f64; N]> {
            unimplemented!(
                "inverse kinematics is not implemented because the representation method of the robotic arm configuration has not been  determined"
            )
        }
    
        #[inline(always)]
        fn limit_joint_jerk(q_dddot: &mut [f64; N]) -> &mut [f64; N] {
            limit(q_dddot, &Self::JOINT_JERK_BOUND)
        }
    
        #[inline]
        fn limit_joint_acc<'a>(
            q_ddot: &'a mut [f64; N],
            q_ddot_last: &[f64; N],
            time: f64,
        ) -> &'a mut [f64; N] {
            q_ddot
                .pipe_mut(|q| limit_dot(q, q_ddot_last, time, &Self::JOINT_JERK_BOUND))
                .pipe_mut(|q| limit(q, &Self::JOINT_ACC_BOUND))
        }
    
        #[inline]
        fn limit_joint_vel<'a>(
            q_dot: &'a mut [f64; N],
            q_dot_last: &[f64; N],
            q_ddot_last: &[f64; N],
            time: f64,
        ) -> &'a mut [f64; N] {
            let mut q_ddot = difference(q_dot, q_dot_last, time);
            q_ddot.pipe_mut(|q| Self::limit_joint_acc(q, q_ddot_last, time));
    
            q_dot
                .pipe_mut(|q| update(q, &q_ddot, time))
                .pipe_mut(|q| limit(q, &Self::JOINT_VEL_BOUND))
        }
    
        #[inline]
        fn limit_joint<'a>(
            q: &'a mut [f64; N],
            q_last: &[f64; N],
            q_dot_last: &[f64; N],
            q_ddot_last: &[f64; N],
            time: f64,
        ) -> &'a mut [f64; N] {
            let mut q_dot = difference(q, q_last, time);
            q_dot.pipe_mut(|q| Self::limit_joint_vel(q, q_dot_last, q_ddot_last, time));
    
            q.pipe_mut(|q| update(q, &q_dot, time))
                .pipe_mut(|q| clamp(q, &Self::JOINT_MIN, &Self::JOINT_MAX))
        }
    
        #[inline]
        fn limit_torque<'a>(tau: &'a mut [f64; N], tau_last: &[f64; N], time: f64) -> &'a mut [f64; N] {
            tau.pipe_mut(|t| limit_dot(t, tau_last, time, &Self::TORQUE_BOUND))
                .pipe_mut(|t| limit(t, &Self::TORQUE_BOUND))
        }
    }
    ```
    