#   СБОРКА

из корневой папки:
``` source /opt/ros/foxy/setup.bash ```
``` colcon build --symlink-install ```

#   ЗАПУСК

1-й терминал
Запускаем робота в rviz и gazebo
``` source /opt/ros/foxy/setup.bash ```
``` source install/setup.bash ```
``` bash ~/ros2_ws/src/mycobot_ros2/mycobot_bringup/scripts/mycobot_280_gazebo.sh ```

2-й терминал
Запускаем файл arm_gripper_loop_controller, который контролирует работу 6dof
``` source /opt/ros/foxy/setup.bash ```
``` source install/setup.bash ```
``` ros2 run mycobot_system_tests arm_gripper_loop_controller ```

# CPP code explanation:

# Arm Gripper Loop Controller Explanation

This ROS 2 node controls a robotic arm and gripper to perform repetitive movements between predefined positions. Let me break down how it works:

## 1. Node Initialization

When the node starts up (`ArmGripperLoopController` constructor):
- Creates two action clients:
  - `/arm_controller/follow_joint_trajectory` for arm movement
  - `/gripper_action_controller/gripper_cmd` for gripper control
- Waits up to 20 seconds for both action servers to become available
- Defines the joint names for the 6-DOF arm
- Sets two positions:
  - `target_pos_`: The destination position (angles in radians for each joint)
  - `home_pos_`: The starting/resting position (all zeros)
- Creates a timer that triggers the control loop every 100ms

## 2. Control Loop Sequence

The `controlLoopCallback()` executes this sequence repeatedly:

1. **Move to Target Position**
   - Calls `sendArmCommand(target_pos_)`
   - Waits 2.5 seconds for movement to complete

2. **Pause at Target**
   - Waits 1 second (simulating work at target position)

3. **Close Gripper**
   - Calls `sendGripperCommand(-0.7)` (closed position)
   - Waits 0.5 seconds

4. **Move to Home Position**
   - Calls `sendArmCommand(home_pos_)`
   - Waits 2.5 seconds for movement

5. **Pause at Home**
   - Waits 1 second (simulating work at home position)

6. **Open Gripper**
   - Calls `sendGripperCommand(0.0)` (open position)
   - Waits 0.5 seconds

7. **Final Pause**
   - Waits 1 second before repeating

## 3. Action Command Details

### Arm Movement (`sendArmCommand`)
- Creates a `FollowJointTrajectory` goal message
- Sets the joint names and target positions
- Configures the movement to take 2 seconds
- Sends the goal asynchronously to the arm controller

### Gripper Control (`sendGripperCommand`)
- Creates a `GripperCommand` goal message
- Sets the position (0.0 for open, -0.7 for closed)
- Sets a maximum effort of 5.0 (force limit)
- Sends the goal asynchronously to the gripper controller

## 4. Why This Structure?

The code is designed to:
- Be robust with server availability checks
- Use standard ROS 2 action interfaces for reliable command execution
- Have predictable timing with explicit waits
- Be easily modifiable for different positions or sequences
- Handle potential exceptions gracefully

## 5. Typical Use Case

This would be used for repetitive tasks like:
- Pick-and-place operations
- Assembly line tasks
- Testing robotic arm and gripper functionality
- Demonstrating coordinated arm/gripper control

The node will continue running this loop until shutdown, making it suitable for continuous operation scenarios.





