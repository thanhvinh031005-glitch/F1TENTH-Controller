# F1TENTH-Controller
# F1TENTH Reactive Follow-the-Gap Controller

## 1. Project Overview
This project implements a reactive autonomous driving controller for the F1TENTH platform using ROS 2 Humble and Python.
The controller uses the **Follow-the-Gap algorithm** to process LiDAR data, detect available free space, and calculate steering and speed commands. It aims to drive safely around the race track while maintaining high speed on straight sections and reducing speed when approaching corners or obstacles.

The controller also includes:
- Automatic lap counting.
- Lap time measurement.
- Adaptive speed control.
- Steering smoothing.
- LiDAR-based obstacle avoidance.
## 2. Technologies Used
- Ubuntu Linux
- ROS 2 Humble
- Python 3
- NumPy
- F1TENTH Simulator
- LiDAR sensor
- Ackermann steering control
## 3. Algorithm: Follow-the-Gap
The Follow-the-Gap algorithm allows the vehicle to navigate around obstacles using LiDAR measurements.
The main steps are:
### Step 1: LiDAR Data Processing
The controller subscribes to the `/scan` topic to receive LiDAR measurements.
The raw measurements are processed by:
- Replacing invalid values.
- Limiting measurements to the maximum detection range.
- Applying a smoothing filter to reduce measurement noise.
### Step 2: Detecting Obstacles
The controller identifies the closest obstacle in front of the vehicle.
A safety bubble is created around the closest obstacle. LiDAR measurements inside this region are removed from the available free space.
This helps prevent the vehicle from driving too close to obstacles.
### Step 3: Finding the Largest Free Gap
The controller identifies continuous regions where the LiDAR distance is greater than the minimum safe distance.
Each available gap is evaluated based on:
- Gap width.
- Average distance.
- Direction relative to the vehicle.
- Difference from the previous target direction.
The controller selects a suitable gap to follow.
### Step 4: Selecting the Target Point
A target point is selected inside the chosen gap.
The target point is used to calculate the desired steering direction.
### Step 5: Steering Control
The target angle is limited by the maximum steering angle.
A smoothing function is applied to reduce sudden steering changes and improve driving stability.
### Step 6: Adaptive Speed Control
The vehicle adjusts its speed according to:
- Steering angle.
- Distance to obstacles in front.
- Distance in the selected turning direction.
The maximum configured speed is **20 m/s**.
The controller reduces speed when approaching obstacles or making sharper turns.
## 4. Code Structure
The main controller is implemented in:
`gap_node.py`
The main functions are:
| Function | Description |
|---|---|
| `__init__()` | Initializes ROS 2 publishers, subscribers, and controller parameters. |
| `odom_callback()` | Processes odometry data and manages lap counting and timing. |
| `preprocess_lidar()` | Filters and processes raw LiDAR measurements. |
| `find_max_gap()` | Detects and selects a suitable free-space gap. |
| `find_best_point()` | Selects the target point inside the selected gap. |
| `smooth_steering()` | Smooths steering commands to reduce sudden direction changes. |
| `calculate_speed()` | Calculates adaptive vehicle speed. |
| `publish_drive()` | Publishes speed and steering commands. |
| `lidar_callback()` | Main callback that processes LiDAR data and controls the vehicle. |
| `main()` | Initializes and runs the ROS 2 node. |

## 5. ROS 2 Communication
### Subscribers
| Topic | Message Type | Purpose |
|---|---|---|
| `/scan` | `sensor_msgs/msg/LaserScan` | Receives LiDAR measurements. |
| `/ego_racecar/odom` | `nav_msgs/msg/Odometry` | Receives vehicle position for lap counting and timing. |
### Publisher
| Topic | Message Type | Purpose |
|---|---|---|
| `/drive` | `ackermann_msgs/msg/AckermannDriveStamped` | Publishes steering and speed commands. |
## 6. Lap Counter Implementation
The controller uses odometry information to track the vehicle's position relative to the starting line.
The lap counter:
1. Records the starting position and direction.
2. Monitors the vehicle's movement.
3. Detects when the vehicle crosses the starting line.
4. Increments the lap count after a completed lap.
5. Displays the completed lap number in the terminal.
```
The lap counter is intended to support the 10-lap competition challenge.
## 7. Lap Timer Implementation

The controller uses Python's `time.monotonic()` function to measure elapsed time.
When the vehicle starts moving, the timer begins.
After each completed lap:
- The elapsed lap time is calculated.
- The result is displayed in the terminal.
- The timer is reset for the next lap.
Lap times are displayed in seconds with three decimal places.
## 8. Installation and Execution
### Requirements
Make sure ROS 2 Humble and the required F1TENTH packages are installed.
Install NumPy if necessary:

pip3 install numpy

### Step 1: Source ROS 2

source /opt/ros/humble/setup.bash

### Step 2: Build the Controller Package

Navigate to the ROS 2 workspace:
cd ~/YOUR_WORKSPACE
Build the controller package:
colcon build --packages-select controllers

### Step 3: Source the Workspace

source install/setup.bash

### Step 4: Start the F1TENTH Simulator
Launch the simulator using the launch command provided by your course or simulator installation.
### Step 5: Run the Controller
Open another terminal and execute:

source /opt/ros/humble/setup.bash
source ~/YOUR_WORKSPACE/install/setup.bash
ros2 run controllers reactive_node

Make sure that `reactive_node` matches the executable name configured in `setup.py`.
## 9. Controller Parameters
The main parameters can be adjusted in the `__init__()` function.
| Parameter | Value | Description |
|---|---:|---|
| Maximum speed | 20.0 m/s | Maximum configured vehicle speed. |
| Maximum steering angle | 0.4189 rad | Maximum steering angle. |
| Safety bubble radius | 0.60 m | Safety region around the closest obstacle. |
| Minimum gap distance | 1.2 m | Minimum distance required for free space. |
| Steering smoothing factor | 0.45 | Controls steering response smoothing. |
| Maximum steering rate | 2.5 rad/s | Limits steering changes. |
These parameters can be tuned according to track conditions and vehicle behavior.
## 10. Competition Objectives
The controller is designed for the following challenges:
### Challenge 1: 10 Laps Without Collisions
The objective is to complete ten consecutive laps without hitting track boundaries or obstacles.
The controller uses LiDAR-based obstacle detection, safety bubbles, and adaptive speed control to help avoid collisions.
### Challenge 2: Fastest Lap
The objective is to achieve the shortest possible lap time while maintaining control of the vehicle.
The controller allows higher speeds on straight sections and reduces speed when approaching corners.
## 11. Conclusion
This project demonstrates a reactive autonomous driving system using the Follow-the-Gap algorithm on the F1TENTH platform.
By combining LiDAR processing, obstacle avoidance, steering smoothing, adaptive speed control, lap counting, and lap timing, the controller provides a foundation for autonomous racing.
Further parameter tuning and simulation testing can improve driving stability, cornering performance, and lap time.
