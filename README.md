# 🏎️ F1TENTH Autonomous Racing Controller
### Reactive Follow-the-Gap Algorithm | ROS 2 Humble | Python

## 📌 1. Project Overview

This project implements a reactive autonomous driving controller for the **F1TENTH platform** using ROS 2 Humble and Python.
The controller uses the **Follow-the-Gap algorithm** to process LiDAR measurements, detect obstacles, identify available free space, and calculate steering and speed commands in real time.
The main objective is to enable the autonomous vehicle to navigate a race track safely while maintaining high speed on straight sections and reducing speed when approaching corners or obstacles.

### Key Features

- LiDAR-based obstacle detection and avoidance.
- Follow-the-Gap algorithm for free-space navigation.
- Adaptive vehicle speed control.
- Steering angle smoothing.
- Automatic lap counting.
- Lap time measurement.
- Real-time Ackermann steering control.

---

## 🛠️ 2. Technologies Used

| Technology | Description |
|---|---|
| Ubuntu Linux | Development environment |
| ROS 2 Humble | Robot communication framework |
| Python 3 | Controller implementation |
| NumPy | Numerical calculations and data processing |
| F1TENTH Simulator | Autonomous vehicle simulation |
| LiDAR | Obstacle detection and distance measurement |
| Ackermann Steering | Vehicle motion control |

---

## 🧠 3. Follow-the-Gap Algorithm

The Follow-the-Gap algorithm is a reactive navigation method that allows the vehicle to avoid obstacles by identifying available free space using LiDAR data.
Instead of following a predefined path, the vehicle continuously analyzes its surroundings and selects a suitable direction to drive.
### Algorithm Workflow
#### Step 1: LiDAR Data Processing

The controller subscribes to the `/scan` topic to receive LiDAR measurements.
The raw measurements are processed through the following operations:
- Replace invalid values (NaN and infinity).
- Limit measurements to the maximum detection range.
- Apply a smoothing filter to reduce measurement noise.
**Purpose:** Obtain reliable LiDAR data for obstacle detection and free-space analysis.

#### Step 2: Obstacle Detection and Safety Bubble

The controller identifies the closest obstacle in front of the vehicle.
A safety bubble is created around the closest obstacle. LiDAR measurements inside this region are removed from the available free space.
This prevents the vehicle from selecting a target point that is too close to an obstacle.
**Purpose:** Maintain a safe distance from obstacles and reduce collision risks.

#### Step 3: Finding the Largest Free Gap

After removing unsafe regions, the controller searches for continuous areas of available free space.
These areas are called **gaps**.
The available gaps are evaluated based on:
- Gap width.
- Average distance.
- Direction relative to the vehicle.
- Difference from the previous target direction.
The controller selects a suitable gap for navigation.
**Purpose:** Identify a safe and practical driving direction.

#### Step 4: Selecting the Target Point

After selecting a suitable gap, the controller determines a target point inside that gap.
The target point represents the desired direction of travel.
The angle between the vehicle's forward direction and the target point is calculated and used to determine the steering command.

**Purpose:** Convert the selected free space into a specific driving direction.

#### Step 5: Steering Control

The controller calculates the steering angle based on the selected target point.
The steering angle is limited by the maximum configured steering angle.
A smoothing function is also applied to reduce sudden steering changes and improve vehicle stability.
**Purpose:** Maintain smooth steering and prevent aggressive direction changes.

#### Step 6: Adaptive Speed Control

The vehicle dynamically adjusts its speed according to the surrounding environment.
The main factors include:
- Steering angle.
- Distance to obstacles in front.
- Distance in the selected turning direction.
The maximum configured speed is **15.0 m/s**.
The controller maintains higher speeds on straight sections and reduces speed when approaching obstacles or making sharper turns.
**Purpose:** Balance driving speed, safety, and vehicle stability.

---

## 📂 4. Code Structure

The main controller is implemented in:

`gap_node.py`

### Main Functions

| Function | Description |
|---|---|
| `__init__()` | Initializes ROS 2 publishers, subscribers, and controller parameters. |
| `odom_callback()` | Processes odometry data for lap counting and lap timing. |
| `preprocess_lidar()` | Filters and processes raw LiDAR measurements. |
| `find_max_gap()` | Identifies and selects a suitable free-space gap. |
| `find_best_point()` | Selects the target point inside the chosen gap. |
| `smooth_steering()` | Smooths steering commands to reduce sudden changes. |
| `calculate_speed()` | Calculates adaptive vehicle speed. |
| `publish_drive()` | Publishes steering and speed commands. |
| `lidar_callback()` | Main callback for LiDAR processing and vehicle control. |
| `main()` | Initializes and runs the ROS 2 node. |

---

## 🔄 5. ROS 2 Communication

The controller communicates with the F1TENTH simulator through ROS 2 topics.
### Subscribers

| Topic | Message Type | Purpose |
|---|---|---|
| `/scan` | `sensor_msgs/msg/LaserScan` | Receives LiDAR measurements for obstacle detection and free-space analysis. |
| `/ego_racecar/odom` | `nav_msgs/msg/Odometry` | Receives vehicle position and movement information for lap counting and timing. |

### Publisher

| Topic | Message Type | Purpose |
|---|---|---|
| `/drive` | `ackermann_msgs/msg/AckermannDriveStamped` | Publishes steering angle and speed commands to the vehicle. |

### Communication Workflow

```text
       LiDAR Sensor
            |
            v
        /scan Topic
            |
            v
    LiDAR Preprocessing
            |
            v
    Obstacle Detection
            |
            v
     Find Free Gap
            |
            v
    Select Target Point
            |
            v
   Steering & Speed Control
            |
            v
       /drive Topic
            |
            v
     F1TENTH Vehicle
```

Odometry data is received separately through `/ego_racecar/odom` and used by the lap counter and lap timer.

---

## 🏁 6. Automatic Lap Counter

The controller includes an automatic lap counting system based on odometry information.
The system monitors the vehicle's position and detects when it crosses the starting line.

### Lap Counting Process

1. Record the initial vehicle position and starting direction.
2. Continuously monitor vehicle movement using odometry.
3. Detect when the vehicle crosses the starting line.
4. Increase the lap counter after a valid crossing.
5. Display the completed lap number in the terminal.
This feature supports the **10-lap competition challenge**, where the vehicle must complete ten consecutive laps without collisions.

## ⏱️ 7. Lap Timer

The controller uses Python's `time.monotonic()` function to measure lap times.

### Timing Process

1. Start the timer when the vehicle begins moving.
2. Continuously measure elapsed time.
3. Calculate the lap time when a lap is completed.
4. Display the result in the terminal.
5. Reset the timer for the next lap.
Lap times are displayed in seconds with three decimal places.

## ⚙️ 8. Controller Parameters

The following parameters are configured in the `__init__()` function.
| Parameter | Value | Description |
|---|---:|---|
| Maximum Speed | 15.0 m/s | Maximum configured vehicle speed. |
| Maximum Steering Angle | 0.4189 rad | Maximum steering angle. |
| Safety Bubble Radius | 0.60 m | Safety region around the closest obstacle. |
| Minimum Gap Distance | 1.2 m | Minimum distance required for free-space identification. |
| Steering Smoothing Factor | 0.45 | Controls steering response smoothing. |
| Maximum Steering Rate | 2.5 rad/s | Limits the rate of steering changes. |
These parameters can be adjusted according to track conditions and vehicle behavior.

---

## 🚀 9. Installation and Execution

### Requirements

Make sure the following software is installed:
- Ubuntu Linux
- ROS 2 Humble
- F1TENTH Simulator
- Python 3
- NumPy
Install NumPy if necessary:

```bash
pip3 install numpy
```

### Step 1: Source ROS 2

```bash
source /opt/ros/humble/setup.bash
```

### Step 2: Navigate to the ROS 2 Workspace

```bash
cd ~/YOUR_WORKSPACE
```

### Step 3: Build the Controller Package

```bash
colcon build --packages-select controllers
```

### Step 4: Source the Workspace

```bash
source install/setup.bash
```

### Step 5: Start the F1TENTH Simulator

Launch the simulator using the launch command provided by your course or simulator installation.
Wait until the simulation environment is ready.

### Step 6: Run the Controller

Open another terminal and execute:

```bash
source /opt/ros/humble/setup.bash
source ~/YOUR_WORKSPACE/install/setup.bash

ros2 run controllers reactive_node
```

**Note:** Make sure `reactive_node` matches the executable name configured in `setup.py`.

---

## 🏆 10. Competition Objectives

The controller is designed to address two main competition challenges.

### Challenge 1: Complete 10 Laps Without Collisions

**Objective:** Complete ten consecutive laps without hitting track boundaries or obstacles.

The controller uses:
- LiDAR-based obstacle detection.
- Safety bubble obstacle filtering.
- Free-space identification.
- Adaptive speed control.
- Steering moothing.
These features help the vehicle navigate the track safely.

### Challenge 2: Achieve the Fastest Lap Time

**Objective:** Achieve the shortest possible lap time while maintaining vehicle stability.
The controller aims to:
- Maintain high speed on straight sections.
- Reduce speed before sharp corners.
- Select suitable driving directions.
- Minimize unnecessary steering changes.
- Maintain smooth vehicle movement.
The lap timer is used to evaluate performance and compare different controller configurations.

---

## 📌 11. Conclusion

This project demonstrates a reactive autonomous driving system using the Follow-the-Gap algorithm on the F1TENTH platform.
By combining LiDAR processing, obstacle detection, free-space identification, steering control, and adaptive speed adjustment, the controller enables the vehicle to navigate a race track without relying on a predefined path.
Additional features, including automatic lap counting and lap timing, provide useful information for evaluating racing performance.
Further improvements through parameter tuning and simulation testing can enhance driving stability, reduce collision risks, and improve lap times.

---

**Project:** F1TENTH Autonomous Racing Controller  
**Framework:** ROS 2 Humble  
**Algorithm:** Follow-the-Gap  
**Programming Language:** Python 3
