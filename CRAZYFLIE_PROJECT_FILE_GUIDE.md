# Crazyflie Gazebo PD Demo: Project File Guide

This guide explains the purpose of every important file in the Crazyflie Gazebo PD project. It also shows students which file to inspect or edit for each type of change.

> This project uses ROS 2 Jazzy, Gazebo Harmonic, Python, Docker, and a simplified PD controller. The current student Docker image is `yujietang/crazyflie-gazebo-pd:v3`.

## 1. Project structure

```text
crazyflie-gazebo-pd-demo/
├── README.md
├── Dockerfile
├── compose.yaml
├── start_demo.sh
├── docs/
│   ├── 01_BUILD_THE_DEMO.md
│   ├── 02_STUDENT_TUTORIAL.md
│   └── ROS2_NODE_LIST_EMPTY_DOCKER_FIX.md
└── ros2_ws/
    └── src/
        └── crazyflie_demo/
            ├── package.xml
            ├── setup.py
            ├── setup.cfg
            ├── resource/
            │   └── crazyflie_demo
            ├── crazyflie_demo/
            │   ├── __init__.py
            │   ├── trajectory_node.py
            │   └── pd_controller.py
            ├── urdf/
            │   ├── crazyflie_description.urdf
            │   ├── cf2_assembly.stl
            │   └── cf2_assembly_with_props.dae
            ├── worlds/
            │   └── crazyflie_world.sdf
            ├── config/
            │   ├── controller.yaml
            │   └── bridge.yaml
            └── launch/
                └── demo.launch.py
```

> Older documentation may refer to `crazyflie.urdf.xacro` or `demo_world.sdf`. In the current project, the correct files are `crazyflie_description.urdf` and `crazyflie_world.sdf`.

## 2. Quick file-selection table

| What you want to do | File to edit |
| --- | --- |
| Change the project introduction or commands shown to students | `README.md` |
| Change packages installed in the Docker image | `Dockerfile` |
| Change container networking, volumes, display, or startup behaviour | `compose.yaml` |
| Change the automatic build and launch sequence | `start_demo.sh` |
| Change ROS 2 package dependencies | `package.xml` |
| Install Python nodes, launch files, URDFs, worlds, or configuration files | `setup.py` |
| Change Python command entry points | `setup.py` |
| Change Python package installation behaviour | `setup.cfg` |
| Change the circular or figure-eight reference trajectory | `crazyflie_demo/trajectory_node.py` |
| Change the PD control law, timer, saturation, or force command | `crazyflie_demo/pd_controller.py` |
| Change the drone body, mass, inertia, collision, mesh, or attached frames | `urdf/crazyflie_description.urdf` |
| Change physics step, gravity, ground, lighting, or fixed world objects | `worlds/crazyflie_world.sdf` |
| Change the default `kp`, `kd`, mass, or acceleration limit | `config/controller.yaml` |
| Change ROS–Gazebo topic mappings | `config/bridge.yaml` |
| Change which nodes start or the initial drone position | `launch/demo.launch.py` |
| Explain setup and troubleshooting to students | Files under `docs/` |

## 3. Root-level files

### `README.md`

**Purpose:** The main entrance to the GitHub repository.

It should explain:

- what the project demonstrates;
- required software;
- how to pull the Docker image;
- how to run the simulation;
- links to student, terminal, and debugging guides.

Edit this file when the public instructions or Docker image tag changes.

Example:

```bash
docker pull yujietang/crazyflie-gazebo-pd:v3
```

### `Dockerfile`

**Purpose:** Defines how the Docker image is built.

It normally installs:

- Ubuntu packages;
- ROS 2 Jazzy;
- Gazebo Harmonic;
- `ros_gz` packages;
- Python dependencies;
- build tools such as `colcon`.

Edit this file when a dependency must be installed permanently in the image.

Do not edit the Dockerfile just to change `kp`, `kd`, a trajectory, or a Gazebo world.

### `compose.yaml`

**Purpose:** Describes how Docker should run the project during development.

It can configure:

- container name;
- host networking;
- display variables;
- software or GPU rendering;
- mounted project folders;
- working directory;
- startup command.

Example responsibilities:

```yaml
container_name: crazyflie_pd_demo
network_mode: host
working_dir: /workspace
```

Edit this file when Docker cannot access the GUI, network, files, or startup script.

### `start_demo.sh`

**Purpose:** Starts the complete demonstration.

It normally:

1. sources ROS 2;
2. builds the workspace;
3. sources the workspace;
4. launches Gazebo and the ROS 2 nodes.

Edit this file when the startup order, build command, or launch command must change.

If the script works, run it only once:

```bash
/workspace/start_demo.sh
```

## 4. Documentation files

### `docs/01_BUILD_THE_DEMO.md`

**Purpose:** Instructor or developer guide.

It explains how the project was created, built, tested, and packaged.

### `docs/02_STUDENT_TUTORIAL.md`

**Purpose:** Beginner operating guide.

It should explain:

- WSL and Docker Desktop setup;
- pulling image v3;
- starting the container;
- using the four terminals;
- changing parameters;
- stopping the experiment.

### `docs/ROS2_NODE_LIST_EMPTY_DOCKER_FIX.md`

**Purpose:** Troubleshoots ROS 2 discovery problems inside Docker.

Use it when:

```bash
ros2 node list
```

returns no nodes even though the simulation appears to be running.

## 5. ROS 2 package files

### `package.xml`

**Purpose:** Declares ROS 2 package metadata and dependencies.

Typical dependencies include:

- `rclpy`;
- `geometry_msgs`;
- `nav_msgs`;
- `ros_gz_bridge`;
- `ros_gz_sim`;
- `ros_gz_interfaces`.

Edit this file when a new ROS 2 message type, node dependency, or runtime package is introduced.

### `setup.py`

**Purpose:** Installs the Python ROS 2 package.

It defines:

- package name;
- Python modules;
- console scripts;
- installation of launch, URDF, world, and configuration files.

The data-file section should use the current folders:

```python
(
    os.path.join('share', package_name, 'urdf'),
    glob('urdf/*'),
),
(
    os.path.join('share', package_name, 'worlds'),
    glob('worlds/*'),
),
(
    os.path.join('share', package_name, 'config'),
    glob('config/*'),
),
```

The console entry points may look like:

```python
'trajectory_node = crazyflie_demo.trajectory_node:main',
'pd_controller = crazyflie_demo.pd_controller:main',
```

If a new Python node is created, add its console entry point here.

### `setup.cfg`

**Purpose:** Tells Python and ROS 2 where executable scripts should be installed.

This file is normally small and rarely needs modification.

### `resource/crazyflie_demo`

**Purpose:** Registers the package with the ROS 2 ament index.

This file is normally empty. Do not delete it and do not add controller code to it.

### `crazyflie_demo/__init__.py`

**Purpose:** Marks `crazyflie_demo` as a Python package.

It is normally empty. Do not remove it.

## 6. ROS 2 Python nodes

### `crazyflie_demo/trajectory_node.py`

**Purpose:** Generates the desired drone position and velocity.

It publishes:

```text
/cf/reference
```

Typical changes include:

- circle radius;
- figure-eight shape;
- target altitude;
- angular speed `omega`;
- initial hovering time;
- adding a new trajectory.

Edit this file if you want to change where the drone should go.

Runtime example:

```bash
ros2 param set /trajectory_node trajectory circle
ros2 param set /trajectory_node omega 0.2
```

### `crazyflie_demo/pd_controller.py`

**Purpose:** Calculates the force required to follow the reference trajectory.

It subscribes to:

```text
/cf/reference
/cf/odom
```

It publishes:

```text
/cf/wrench
```

The simplified control law is:

```text
position_error = reference_position - current_position
velocity_error = reference_velocity - current_velocity

acceleration_command =
    kp * position_error
  + kd * velocity_error
  + gravity_compensation
```

Edit this file when changing:

- the PD control equation;
- the controller timer;
- acceleration saturation;
- gravity compensation;
- world-frame versus body-frame force conversion.

For the WSL 500 Hz configuration:

```python
self.timer = self.create_timer(0.002, self.control_loop)
```

## 7. Drone model files

### `urdf/crazyflie_description.urdf`

**Purpose:** Defines the Crazyflie body and frames attached to it.

It can contain:

- `base_link`;
- mass and inertia;
- visual mesh;
- collision geometry;
- body coordinate axes;
- camera, IMU, or sensor links;
- fixed joints;
- Gazebo model plugins.

Edit this file when a component must move and rotate with the drone.

Examples:

- body X, Y and Z axes;
- `camera_link`;
- `imu_link`;
- sensor mounting offsets.

To add a real child frame:

```xml
<link name="camera_link"/>

<joint name="camera_joint" type="fixed">
  <parent link="base_link"/>
  <child link="camera_link"/>
  <origin xyz="0.05 0 0.02" rpy="0 0 0"/>
</joint>
```

### Mesh files

```text
urdf/cf2_assembly.stl
urdf/cf2_assembly_with_props.dae
```

**Purpose:** Store the 3D geometry used to display the Crazyflie.

- `.stl` normally contains geometry only;
- `.dae` can contain colors and material information.

Do not edit mesh files to change mass, inertia, controller gains, or trajectory behaviour.

## 8. Gazebo world file

### `worlds/crazyflie_world.sdf`

**Purpose:** Defines the environment surrounding the drone.

It can contain:

- physics system plugins;
- physics time step;
- gravity;
- lighting;
- ground plane;
- fixed world coordinate axes;
- obstacles;
- landing zones;
- walls and tables.

Edit this file when an object should remain fixed in the Gazebo world.

For the WSL 500 Hz configuration:

```xml
<physics name="500hz_physics" type="ignored">
  <max_step_size>0.002</max_step_size>
  <real_time_factor>1.0</real_time_factor>
</physics>
```

A fixed world coordinate frame belongs in this file:

```xml
<model name="world_coordinate_frame">
  <static>true</static>
  ...
</model>
```

The model must be placed inside:

```xml
<sdf>
  <world>
    <!-- Fixed model goes here -->
  </world>
</sdf>
```

## 9. Configuration files

### `config/controller.yaml`

**Purpose:** Stores the controller's default parameters.

Example:

```yaml
pd_controller:
  ros__parameters:
    kp: 2.0
    kd: 2.8
    mass: 0.038
    max_acceleration: 3.0
```

Edit this file when a parameter should remain the default after restarting.

For temporary experiments, use:

```bash
ros2 param set /pd_controller kp 2.0
ros2 param set /pd_controller kd 2.8
```

### `config/bridge.yaml`

**Purpose:** Connects ROS 2 topics to Gazebo Transport topics.

Important topic paths include:

```text
Gazebo odometry -> /cf/odom
/cf/wrench      -> Gazebo wrench command
```

Edit this file when:

- a topic is missing;
- a topic name is incorrect;
- the ROS and Gazebo message types do not match;
- the bridge direction is incorrect.

Do not edit `bridge.yaml` to change physics frequency, trajectory speed, or PD gains.

## 10. Launch file

### `launch/demo.launch.py`

**Purpose:** Starts and connects the complete ROS 2 system.

It normally launches:

- Gazebo server;
- Crazyflie model spawning;
- ROS–Gazebo parameter bridge;
- trajectory node;
- PD controller.

It may also define:

- the world-file path;
- the URDF-file path;
- the initial drone position;
- parameter-file paths;
- node output and startup arguments.

Edit this file when changing what starts or where the drone initially appears.

If the world file was renamed, make sure the launch file uses:

```python
'crazyflie_world.sdf'
```

## 11. Coordinate-frame file guide

| Coordinate or frame change | Correct file |
| --- | --- |
| Fixed world coordinate visual | `worlds/crazyflie_world.sdf` |
| Body coordinate visual moving with the drone | `urdf/crazyflie_description.urdf` |
| Camera or IMU frame attached to the drone | `urdf/crazyflie_description.urdf` |
| ROS TF relationship between robot links | URDF links and joints, published by `robot_state_publisher` |
| Static ROS TF such as `world -> odom` | A static-transform command or launch node |
| Initial drone coordinates | `launch/demo.launch.py` |
| Reference-trajectory coordinates | `crazyflie_demo/trajectory_node.py` |
| Convert controller force between frames | `crazyflie_demo/pd_controller.py` |

## 12. Visual coordinates versus real TF frames

A colored coordinate visual only appears in Gazebo:

```text
Visible in Gazebo: yes
Available in ROS TF: no
```

A URDF link connected by a joint defines a real robot frame:

```text
Visible in Gazebo: yes, if visual geometry is included
Available in ROS TF: yes, if robot_state_publisher is running
```

For future computer-vision work, use actual links and joints for frames such as:

```text
base_link
└── camera_link
    └── camera_optical_frame
```

Check a published transform with:

```bash
ros2 run tf2_ros tf2_echo base_link camera_link
```

## 13. Rebuild rules

Rebuild after changing:

- Python source code;
- `setup.py`;
- `package.xml`;
- launch files;
- URDF files;
- SDF world files;
- configuration files.

```bash
cd /workspace/ros2_ws
source /opt/ros/jazzy/setup.bash
colcon build --symlink-install
source install/setup.bash
```

If stale build metadata refers to an old filename:

```bash
cd /workspace/ros2_ws
rm -rf build/crazyflie_demo
rm -rf install/crazyflie_demo
rm -rf src/crazyflie_demo/crazyflie_demo.egg-info
colcon build --symlink-install
```

## 14. Recommended debugging order

1. Read the error and identify the subsystem.
2. Check ROS 2 nodes with `ros2 node list`.
3. Check topics with `ros2 topic list`.
4. Inspect `/cf/odom` and `/cf/wrench`.
5. Select the responsible file using this guide.
6. Change one item at a time.
7. Validate URDF or SDF syntax.
8. Rebuild and restart the simulation.
9. Compare the new result with the previous measurement.

This prevents students from changing unrelated files and makes the debugging process reproducible.
