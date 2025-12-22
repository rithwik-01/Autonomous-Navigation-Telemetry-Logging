# TurtleTrack - Autonomous Navigation & Telemetry Logger

**ROS-based waypoint navigation with PID control and real-time telemetry logging**

[![ROS Version](https://img.shields.io/badge/ROS-Kinetic-blue.svg)](http://wiki.ros.org/kinetic)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.x-yellow.svg)](https://www.python.org/)

---

## Overview

TurtleTrack enables the Turtlebot robot to autonomously navigate through a sequence of waypoints using ROS (Robot Operating System) and the Turtlebot simulator. The system implements precise PID control for steering, odometry-based position tracking, and comprehensive telemetry logging for analysis and verification.

### Features

| Feature | Description |
|---------|-------------|
| **Waypoint Navigation** | Define and execute complex multi-point trajectories |
| **PID Control** | Discrete PID controller for accurate steering and movement |
| **Odometry Tracking** | Real-time position and orientation monitoring |
| **Telemetry Logging** | CSV export of trajectory and control parameters |
| **Visualization** | Real-time trajectory plotting and orientation analysis |

---

## Architecture

### High-Level System Design

```
TURTLETRACK NAVIGATION SYSTEM
===========================================================================================

  [TURTLEBOT]            [CONTROLLER]            [TURTLEBOT]
  SIMULATOR      ODOM    (waypoint)     TWIST    ACTUATORS
                            |                    |
                            | Telemetry          |
                            V                    |

                       [DATA LOGGER]
                      + Visualizer


ROS Communication Layer:
===========================================================================================

                    /odom (Odometry)
                          |
                          | subscribe
                          V
                   waypoint.py
                          |
                          |  • PID Control
                          |  • Navigation
                          |  • Logging
                          |
                          | publish
                          V
                  /cmd_vel_mux/input/navi (Twist)

Output Files:
  - coordinates.csv  (trajectory: x, y, theta)
  - PID.csv          (control parameters)
  - Plots            (matplotlib visualization)
```

### Component Breakdown

**PID Controller (`PID` class)**
- Discrete PID control for angular velocity regulation
- Parameters: Kp, Ki, Kd with anti-windup integrator clamping
- Steering mode: Pure P control (Kp=1, Ki=0, Kd=0)
- Movement mode: Full PID (Kp=1, Ki=0.02, Kd=0.2)

**Navigation Controller (`turtlebot_move` class)**
- Odometry Subscriber: Receives `/odom` messages and extracts x, y, theta
- Velocity Publisher: Sends `/cmd_vel_mux/input/navi` (Twist) commands
- Waypoint Sequencer: Iterates through `WAYPOINTS` list
- Data Logger: Records trajectory and PID values to CSV

**Data Flow**
```
Odometry Callback
    |-> Update position (x, y)
    |-> Convert quaternion to Euler angles (theta)
    |-> Append to trajectory buffer
    |-> Store PID parameters
    V
move_to_point(target_x, target_y)
    |-> Phase 1: Rotate to face target (P control)
    |-> Phase 2: Move to target (PID control)
    |-> Stop at waypoint
    V
Save to CSV + Visualize
```

---

## Installation

### Prerequisites

| Requirement | Details |
|-------------|---------|
| ROS | Kinetic or later |
| Turtlebot Simulator | [turtlebot_simulator](https://github.com/turtlebot/turtlebot_simulator) |
| Python | 3.x |
| Dependencies | rospy, geometry_msgs, nav_msgs, tf, matplotlib, numpy |

### Setup

```bash
# 1. Clone the repository
git clone https://github.com/rithwik-01/Autonomous-Navigation-Telemetry-Logging.git

# 2. Install ROS dependencies (if not already installed)
sudo apt-get install ros-$(rosversion -d)-geometry-msgs \
                      ros-$(rosversion -d)-nav-msgs \
                      ros-$(rosversion -d)-tf

# 3. Build the package
cd Autonomous-Navigation-Telemetry-Logging
catkin_make

# 4. Source the workspace
source devel/setup.bash

# 5. Launch the Turtlebot simulator
roslaunch turtlebot_gazebo turtlebot_world.launch
```

---

## Usage

### Running the Navigation

```bash
# In a new terminal (after sourcing workspace):
rosrun turtlebot_waypoint waypoint.py
```

The Turtlebot will:
1. Start at origin (0, 0)
2. Navigate through each waypoint in sequence
3. Pause briefly at each waypoint
4. Log all telemetry data

### Output Files

| File | Description | Format |
|------|-------------|--------|
| `coordinates.csv` | Trajectory data (x, y, theta) | CSV |
| `PID.csv` | PID parameters over time | CSV |
| Plot display | Real-time visualization | Matplotlib |

### Visualization

After execution, two plots are displayed:

1. **Trajectory Plot**: X-Y path visualization
2. **Theta Plot**: Orientation over time

---

## Configuration

### Modify Waypoints

Edit the `WAYPOINTS` list in `waypoint.py`:

```python
WAYPOINTS = [
    [0.5, 0],    # [x, y] coordinates
    [1, 0],
    [1, 0.5],
    [1, 1],
    # Add more waypoints...
]
```

### Tune PID Parameters

Adjust control gains in the `move_to_point()` method:

```python
# Steering phase (rotation only)
self.pid_theta.setPID(Kp=1.0, Ki=0.0, Kd=0.0)

# Movement phase (rotation + translation)
self.pid_theta.setPID(Kp=1.0, Ki=0.02, Kd=0.2)
```

**Tuning Guidelines**:
- **Kp**: Higher values provide faster response but may cause oscillation
- **Ki**: Eliminates steady-state error but can cause overshoot
- **Kd**: Reduces overshoot but amplifies noise

---

## Project Structure

```
turtlebot_waypoint/
├── src/
│   └── script/
│       └── waypoint.py          # Main navigation script
├── CMakeLists.txt               # Build configuration
├── package.xml                  # Package metadata and dependencies
├── LICENSE                      # MIT License
└── README.md                    # This file
```

---

## Contributing

Contributions are welcome! Please feel free to submit a pull request or open an issue for bugs and feature requests.

**Development Workflow**:
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Acknowledgements

- **Turtlebot** by [Open Robotics](https://www.openrobotics.org/)
- **Turtlebot Simulator** - [turtlebot_simulator](https://github.com/turtlebot/turtlebot_simulator) package
- **ROS (Robot Operating System)** - [ros.org](http://www.ros.org/)

---

**Built with ROS**
