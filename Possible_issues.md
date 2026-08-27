Possible issues that can cause the drone to drop or fly unstably in Gazebo under WSL:

1. **Physics rate is too high**
   `max_step_size = 0.001` requires approximately 1000 physics updates per second, which WSL may not maintain.

2. **Controller update rate is too low**
   The `/cf/wrench` rate may be lower than the Gazebo physics rate, causing insufficient or inconsistent force application.

3. **Timing mismatch**
   The Gazebo step size and Python controller timer may use different periods, such as 1 ms versus 2 ms.

4. **WSL scheduling jitter**
   Even when the average rate is sufficient, irregular message intervals can make the PD output fluctuate.

5. **Gazebo GUI consumes excessive CPU**
   Software rendering, especially with `LIBGL_ALWAYS_SOFTWARE=1`, competes with physics and control calculations.

6. **Docker and WSL overhead**
   Virtualization and container communication can introduce latency compared with native Ubuntu.

7. **Trajectory moves too quickly**
   A high `omega` value makes the reference position change faster than the controller can follow.

8. **Trajectory starts immediately**
   The drone must take off and follow a moving reference simultaneously. An initial hovering period would reduce the starting error.

9. **PD gains are unsuitable for WSL**
   Gains tuned on native Ubuntu may cause oscillation or poor tracking when WSL introduces additional delay.

10. **Acceleration or force saturation**
    The controller may reach `max_acceleration`, preventing it from generating enough corrective force.

11. **Incorrect gravity compensation**
    If the controller does not continuously include approximately \(mg\), the average upward force may be insufficient.

12. **Incorrect drone mass**
    A mismatch between the mass in the URDF and the controller configuration produces incorrect gravity compensation.

13. **Wrong entity name or type**
    The wrench must target the correct Gazebo entity, such as the `crazyflie` model rather than an incorrect link.

14. **Bridge configuration problems**
    Incorrect topic names, message types, or bridge directions can prevent odometry or wrench messages from reaching Gazebo.

15. **Odometry delay or low update rate**
    Delayed state feedback means the PD controller calculates force from outdated position and velocity data.

16. **ROS 2 message loss or queueing**
    Messages may be dropped or delayed when the system is overloaded.

17. **Real-time factor below 1.0**
    Gazebo may be unable to simulate one second of simulation time within one second of real time.

18. **Collision with the ground**
    A poor collision model or low initial height may cause the drone to contact the ground and fail to recover.

19. **Large initial tracking error**
    Immediately commanding a distant or moving reference may trigger force saturation and oscillation.

20. **Simplified physical model**
    Directly applying a world-frame wrench without attitude control, motor dynamics, or thrust persistence makes the simulation especially sensitive to update rates.
