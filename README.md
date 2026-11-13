# Autonomous-Robot-Navigation-and-Perception

A ROS 2 and Gazebo-based autonomous mobile robot simulation that enables a differential-drive robot to perceive its environment, build a map, localize itself, plan paths, avoid obstacles, and navigate autonomously to target locations. The project also integrates camera-based object detection using YOLO to demonstrate vision-based robot perception.
Project Overview
Autonomous mobile robots need to answer several fundamental questions:
- Where am I? → Localization
- What does my environment look like? → Mapping / SLAM
- Where should I go? → Navigation goal
- How should I get there? → Path planning
- Is there anything blocking my path? → Obstacle detection
- What is that object? → Computer vision
- How should the robot react? → Control / navigation behavior
This project combines these components into a complete simulated autonomous navigation pipeline using ROS 2 and Gazebo.

The robot operates inside a simulated warehouse environment and uses virtual sensors such as LiDAR, camera, IMU, and wheel odometry.
Objectives
The main objectives are:
1. Build a differential-drive mobile robot in simulation.
2. Integrate simulated LiDAR, camera, IMU, and wheel odometry.
3. Enable communication between robot components using ROS 2.
4. Generate a map of the environment using SLAM.
5. Localize the robot within the generated map.
6. Plan collision-free paths to navigation goals.
7. Detect and avoid obstacles.
8. Navigate autonomously through a warehouse environment.
9. Perform object detection using YOLO and the robot's camera.
10. Connect visual perception with robot navigation behavior.
System Architecture
                       Gazebo Simulation
                              │
                              ▼
                    Differential Drive Robot
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
          ▼                   ▼                   ▼
       LiDAR                Camera               IMU
          │                   │                   │
       /scan               /image               /imu
          │                   │                   │
          │                   ▼                   │
          │             OpenCV + YOLO            │
          │                   │                   │
          │              Object Detection        │
          │                   │                   │
          └───────────────┐   │   ┌───────────────┘
                          ▼   ▼   ▼
                         ROS 2
                          │
             ┌────────────┼────────────┐
             ▼            ▼            ▼
           SLAM       Localization   Perception
             │            │            │
             └────────────┼────────────┘
                          ▼
                        Nav2
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
       Global Path Planner       Local Controller
              │                       │
              └───────────┬───────────┘
                          ▼
                  Velocity Commands
                          │
                       /cmd_vel
                          │
                          ▼
                  Differential Drive
                          │
                          ▼
                    Robot Movement

Technologies Used
Technology	Purpose
ROS 2	Robot communication and system integration
Gazebo	Robot and environment simulation
Python	ROS 2 nodes, perception and automation
C++	Optional ROS 2/control components
Nav2	Autonomous navigation
SLAM Toolbox	Mapping and SLAM
OpenCV	Image processing
YOLO	Object detection
LiDAR	Distance and obstacle perception
IMU	Orientation and motion sensing
Wheel Odometry	Robot movement estimation
RViz2	Visualization and debugging


1. Robot Model
The robot is modeled as a differential-drive mobile robot.

It consists of:
- chassis
- left wheel
- right wheel
- caster/support wheel
- LiDAR
- RGB camera
- IMU
- wheel odometry
The differential-drive mechanism allows the robot to:
- move forward
- move backward
- rotate
- follow curved trajectories
Differential Drive
The robot's movement is controlled using linear and angular velocity:
v = linear velocity
ω = angular velocity

For a differential-drive robot:
v = (vR + vL) / 2

ω = (vR - vL) / L

where:
- vR = right wheel velocity
- vL = left wheel velocity
- L = distance between the wheels
These commands are ultimately used to control the robot's motion.
2. ROS 2 Architecture
ROS 2 provides the communication layer between different parts of the robot.

Instead of writing one large program, the system is divided into independent nodes.

For example:
Robot State Publisher
        │
        ├── Robot description
        │
        ▼
     Sensors
        │
        ▼
    Sensor Nodes
        │
        ▼
      SLAM
        │
        ▼
  Localization
        │
        ▼
      Nav2
        │
        ▼
  Robot Controller

Important ROS 2 concepts used
Nodes

Individual processes responsible for specific tasks.

Topics

Used for continuous data communication.

Examples:
/scan
/image_raw
/imu
/odom
/cmd_vel

TF

Maintains relationships between coordinate frames.

Services

Used for request/response operations.

Actions

Useful for long-running tasks such as navigation.
3. Sensor Integration
The robot uses multiple simulated sensors.
LiDAR
The simulated LiDAR measures distances between the robot and surrounding objects.
LiDAR
   ↓
LaserScan
   ↓
/scan
   ↓
Navigation / SLAM

LiDAR data is used for:
- obstacle detection
- mapping
- localization
- navigation costmaps
Camera
The simulated RGB camera provides images from the robot's viewpoint.
Camera
   ↓
Image
   ↓
/image_raw
   ↓
OpenCV / YOLO
   ↓
Object Detection

The camera is primarily used for vision-based perception.
IMU
The IMU provides information such as:
- angular velocity
- linear acceleration
- orientation-related information
ROS 2 publishes this information through an IMU message topic.

It can be used as part of the robot's state estimation pipeline.
Wheel Odometry
Wheel movement is used to estimate how far the robot has moved.
Wheel Rotation
      ↓
Odometry Calculation
      ↓
/odom
      ↓
Robot Pose Estimation

Odometry provides an estimate of the robot's position and orientation over time.

However, odometry accumulates error, which is why localization and SLAM are important.
4. Coordinate Frames and TF
Robotics systems need a common coordinate system.

A typical TF tree for this project can look like:
map
 │
 ▼
odom
 │
 ▼
base_footprint
 │
 ▼
base_link
 ├── laser_link
 ├── camera_link
 └── imu_link

Meaning
map

Global environment coordinate system.

odom

Continuous local odometry frame.

base_link

Main robot body frame.

laser_link

Coordinate frame of the LiDAR.

camera_link

Coordinate frame of the camera.

imu_link

Coordinate frame of the IMU.

Correct TF relationships are critical for:
- SLAM
- localization
- sensor visualization
- navigation
- sensor transformations
5. SLAM
SLAM stands for:

Simultaneous Localization and Mapping

The robot needs to create a map while determining where it is inside that map.
LiDAR + Odometry
       ↓
      SLAM
       ↓
 ┌─────┴─────┐
 ▼           ▼
Map       Robot Pose

The robot moves through an unknown environment while collecting sensor measurements.

The SLAM system uses those measurements to construct a map.

For this project, SLAM Toolbox is used.
Output
The result is a 2D occupancy map representing:
- free space
- occupied space
- unknown space
Conceptually:
████████████████████
█                  █
█    ███           █
█    ███      R    █
█                  █
█        ███       █
█              G   █
████████████████████

Where:
- R = robot
- G = goal
- ███ = obstacles/walls
6. Localization
After creating a map, the robot needs to determine:
"Where am I on this map?"

Localization estimates the robot's pose:
x, y, θ

where:
- x = position along X
- y = position along Y
- θ = orientation
A localization system compares sensor observations with the known map to estimate the robot's current pose.

This allows Nav2 to know:
Current Position → Target Position

7. Path Planning
Once the robot receives a goal, it needs to determine a route.

For example:
Start
  │
  ▼
  ────────────────┐
                  │
        █████     │
        █████     │
                  ▼
                Goal

The global planner generates a route through the environment while considering obstacles.

The resulting path can be represented as a sequence of poses:
(x1,y1)
(x2,y2)
(x3,y3)
...
(xn,yn)

Nav2 provides the navigation framework for this process.
8. Obstacle Avoidance
Path planning alone isn't enough.

Suppose the robot has planned this:
Robot ───────────────→ Goal

A new obstacle appears:
Robot ─────── X ─────→ Goal
              ↑
           obstacle

The robot's LiDAR detects the obstacle.

The navigation system updates its costmap and determines a safer motion.

The robot may:
- slow down
- stop
- steer around the obstacle
- re-plan if necessary
This provides dynamic obstacle response during navigation.
9. Costmaps
Nav2 uses costmaps to represent how safe different regions are for the robot.

Conceptually:
Low cost       → Safe area
High cost      → Near obstacle
Occupied       → Collision region

Sensor information such as LiDAR observations is incorporated into the costmap.

This allows the navigation system to avoid areas where the robot could collide.

Two important concepts are:
Global Costmap
Used primarily for global route planning.
Local Costmap
Used for short-range navigation and obstacle avoidance around the robot.
10. Nav2 Navigation Pipeline
The complete navigation pipeline is:
Navigation Goal
       ↓
Goal / Behavior System
       ↓
Global Planner
       ↓
Global Path
       ↓
Local Costmap
       ↓
Local Controller
       ↓
Velocity Command
       ↓
Robot

The robot continuously updates its motion based on the environment.
11. Camera-Based Perception
The robot's camera provides visual information.

The image is processed using:

OpenCV + YOLO
Camera
   ↓
RGB Image
   ↓
OpenCV
   ↓
YOLO
   ↓
Object Detection

YOLO identifies objects within the camera frame.

For example:
┌─────────────────────────┐
│                         │
│       Person            │
│       █████             │
│       █████             │
│                         │
│       Robot             │
└─────────────────────────┘

The detection output can include:
- object class
- bounding box
- confidence score
12. Vision-Driven Robot Behavior
The important part is that YOLO is not only used to display bounding boxes.

Its output can be connected to robot behavior.

For example:
Camera
   ↓
YOLO
   ↓
Person Detected
   ↓
ROS 2 Decision Node
   ↓
Safety Check
   ↓
Stop / Slow Robot

When the detected object is no longer blocking the robot's path, the navigation system can continue.

This creates a connection between:

Perception → Decision → Action

which is a key concept in autonomous robotics.
13. Complete Autonomous Navigation
The final system combines everything:
                 User
                  │
             Navigation Goal
                  │
                  ▼
                Nav2
                  │
                  ▼
            Path Planning
                  │
                  ▼
             Robot Motion
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
      LiDAR              Camera
        │                   │
        ▼                   ▼
      SLAM              YOLO/OpenCV
        │                   │
        ▼                   ▼
  Localization         Object Detection
        │                   │
        └─────────┬─────────┘
                  ▼
             Robot Decision
                  │
                  ▼
              /cmd_vel
                  │
                  ▼
            Mobile Robot

The complete system enables the robot to:

Map → Localize → Plan → Detect → Avoid → Navigate
Project Workflow
The project is developed incrementally.
Phase 1 — Robot Simulation
Create:
- robot model
- differential-drive wheels
- sensors
- Gazebo environment
Phase 2 — ROS 2 Integration
Connect:
- robot description
- sensor topics
- TF
- odometry
- velocity commands
Phase 3 — Mapping
Add:
- LiDAR
- SLAM Toolbox
- map generation
Phase 4 — Localization
Use the generated map to estimate the robot's pose.
Phase 5 — Autonomous Navigation
Configure:
- Nav2
- planner
- controller
- global costmap
- local costmap
- navigation behaviors
Phase 6 — Obstacle Avoidance
Test:
- static obstacles
- narrow corridors
- unexpected obstacles
- dynamic obstacles
Phase 7 — Computer Vision
Add:
- camera
- OpenCV
- YOLO
- object detection
Phase 8 — Perception + Navigation
Connect object detections to navigation behavior.
Example Mission
A typical demonstration can be:
1. Start robot in warehouse
        ↓
2. Launch Gazebo
        ↓
3. Start ROS 2 sensors
        ↓
4. Generate map using SLAM
        ↓
5. Save map
        ↓
6. Start localization
        ↓
7. Launch Nav2
        ↓
8. Give navigation goal
        ↓
9. Robot plans a path
        ↓
10. Robot follows the path
        ↓
11. LiDAR detects obstacle
        ↓
12. Robot avoids/replans
        ↓
13. Camera detects object
        ↓
14. Robot performs defined behavior
        ↓
15. Robot reaches goal

ROS 2 Topics
The project can contain topics similar to:
Topic	Purpose
/scan	LiDAR measurements
/image_raw	Camera images
/camera_info	Camera calibration information
/imu	IMU measurements
/odom	Wheel odometry
/cmd_vel	Robot velocity commands
/tf	Coordinate transformations
/tf_static	Static transformations
/map	Generated map
/goal_pose	Navigation goal


The exact topic names may vary depending on your implementation.
ROS 2 Nodes
The project can be organized into separate nodes/packages for:
robot_description
        ↓
robot_bringup
        ↓
robot_control
        ↓
robot_navigation
        ↓
robot_perception

This modular structure makes the system easier to maintain and debug.

## Package Structure
A possible project structure is:
autonomous_mobile_robot/
│
├── src/
│   ├── robot_description/
│   │   ├── urdf/
│   │   ├── meshes/
│   │   └── launch/
│   │
│   ├── robot_bringup/
│   │   ├── launch/
│   │   └── config/
│   │
│   ├── robot_control/
│   │   └── robot_control/
│   │
│   ├── robot_navigation/
│   │   ├── config/
│   │   ├── maps/
│   │   └── launch/
│   │
│   └── robot_perception/
│       ├── robot_perception/
│       └── launch/
│
├── worlds/
│   └── warehouse.world
│
├── maps/
│   └── warehouse_map.yaml
│
├── README.md
└── requirements.txt


### Limitations
This project is a simulation-based robotics project.
The sensors are simulated rather than physical:
- virtual LiDAR
- virtual camera
- virtual IMU
- simulated odometry
Therefore, the project does not cover some challenges encountered with physical robots, such as:
- real sensor noise
- hardware calibration
- communication latency
- motor inaccuracies
- battery limitations
- mechanical failures
- real-world lighting variations
- physical sensor mounting
- hardware-level debugging
However, the project demonstrates the software architecture and algorithms used in autonomous mobile robot systems.
