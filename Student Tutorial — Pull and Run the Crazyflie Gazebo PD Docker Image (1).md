# Student Tutorial — Pull and Run the Crazyflie Gazebo PD Docker Image

This tutorial is for **students**.

The instructor has already prepared the Docker image. You do **not** need to:

- clone the GitHub repository;
- build the Docker image;
- install ROS 2 manually;
- install Gazebo manually.

You only need:

```text
Windows
↓
WSL 2 Ubuntu
↓
Docker Desktop
↓
Pull instructor Docker image
↓
Run container
↓
Enter container
↓
Run Crazyflie simulation
```

The instructor's Docker Hub account is:

```text
yujitang
```

The Docker image used in this tutorial is:

```text
yujitang/crazyflie-gazebo-pd:latest
```

---

# Part 1 — Install WSL 2 and Ubuntu

Open **PowerShell as Administrator** and run:

```powershell
wsl --install
```

Restart Windows if requested.

Then open **Ubuntu** from the Windows Start menu.

The first time Ubuntu starts, it will ask you to create a Linux username and password.

You can check WSL with:

```powershell
wsl -l -v
```

You should see something similar to:

```text
NAME      STATE      VERSION
Ubuntu    Running    2
```

The important part is:

```text
VERSION 2
```

---

# Part 2 — Configure Docker Desktop for WSL

## 1. Start Docker Desktop

Open:

```text
Windows Start Menu
→ Docker Desktop
```

Wait until Docker Desktop reports that the Docker engine is running.

Keep Docker Desktop running while using the simulation.

---

## 2. Enable the WSL 2 engine

Open:

```text
Docker Desktop
→ Settings
→ General
```

Enable:

```text
Use the WSL 2 based engine
```

On some versions of Docker Desktop, this option is already enabled automatically.

---

## 3. Enable Ubuntu integration

Open:

```text
Docker Desktop
→ Settings
→ Resources
→ WSL Integration
```

Enable your Ubuntu distribution.

For example:

```text
Ubuntu       ON
```

Then select:

```text
Apply & Restart
```

If the **WSL Integration** page is missing, make sure Docker Desktop is using **Linux containers**, not Windows containers.

---

# Part 3 — Test Docker from WSL Ubuntu

Open **Ubuntu** from the Windows Start menu.

Your terminal may look similar to:

```text
student@DESKTOP-123456:~$
```

Run:

```bash
docker version
```

If Docker Desktop and WSL are connected correctly, Docker client and server information should appear.

Then test Docker with:

```bash
docker run --rm hello-world
```

You should see:

```text
Hello from Docker!
```

This means WSL Ubuntu can communicate with Docker Desktop.

> Do not install another Docker Engine inside Ubuntu. Docker Desktop already provides the Docker engine to WSL.

---

# Part 4 — Pull the Instructor's Docker Image

The instructor has already uploaded the Crazyflie Gazebo PD environment to Docker Hub.

The image is:

```text
yujitang/crazyflie-gazebo-pd:latest
```

Students only need to download this image once.

## Recommended method — pull from WSL

Inside Ubuntu, run:

```bash
docker pull yujitang/crazyflie-gazebo-pd:latest
```

Docker will begin downloading several layers.

You may see output similar to:

```text
latest: Pulling from yujitang/crazyflie-gazebo-pd
...
Download complete
...
Status: Downloaded newer image for yujitang/crazyflie-gazebo-pd:latest
```

When it finishes, check the image:

```bash
docker image ls
```

You should see something similar to:

```text
REPOSITORY                        TAG       IMAGE ID       SIZE
yujitang/crazyflie-gazebo-pd     latest    xxxxxxxxxxxx   ...
```

You can also verify it directly:

```bash
docker image inspect yujitang/crazyflie-gazebo-pd:latest > /dev/null && echo "IMAGE READY"
```

Expected output:

```text
IMAGE READY
```

---

# Part 5 — Pull the Image Using Docker Desktop Instead

If you prefer the graphical interface, you can also use Docker Desktop.

Open:

```text
Docker Desktop
→ Images
```

Search for:

```text
yujitang/crazyflie-gazebo-pd
```

Select the repository and choose the:

```text
latest
```

tag.

Then select:

```text
Pull
```

After downloading, check:

```text
Docker Desktop
→ Images
→ Local
```

You should see:

```text
yujitang/crazyflie-gazebo-pd
```

with tag:

```text
latest
```

Docker Desktop and WSL Ubuntu use the same Docker engine, so you **do not need to pull the image twice**.

---

# Part 6 — Allow Gazebo GUI to Open on Windows

The Crazyflie simulation uses Gazebo, which opens a graphical window.

Modern Windows 11 systems with WSLg normally support Linux GUI applications automatically.

First, check that the display variable exists:

```bash
echo $DISPLAY
```

You may see something such as:

```text
:0
```

or:

```text
:0.0
```

You can also check:

```bash
echo $WAYLAND_DISPLAY
```

If WSLg is working correctly, you normally do not need to install an additional X server.

---

# Part 7 — Run the Crazyflie Docker Container

Students should run the container from **WSL Ubuntu**, not from inside Docker Desktop.

Open Ubuntu.

First make sure the image exists:

```bash
docker image ls
```

Then run:

```bash
docker run -it \
  --name crazyflie_pd_demo \
  --network host \
  --ipc host \
  -e DISPLAY=$DISPLAY \
  -e WAYLAND_DISPLAY=$WAYLAND_DISPLAY \
  -e XDG_RUNTIME_DIR=$XDG_RUNTIME_DIR \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  yujitang/crazyflie-gazebo-pd:latest
```

This creates a container called:

```text
crazyflie_pd_demo
```

After entering the container, the terminal prompt may change from:

```text
student@DESKTOP:~$
```

to something similar to:

```text
root@DESKTOP:/workspace#
```

This means you are now **inside the Docker container**.

---

# Part 8 — Start the Crazyflie Simulation

Inside the Docker container, run:

```bash
/start_demo.sh
```

If the image uses the script from `/workspace`, the command may instead be:

```bash
cd /workspace
./start_demo.sh
```

The script should start:

```text
ROS 2
Gazebo
Crazyflie model
trajectory node
PD controller
ROS ↔ Gazebo bridge
```

Gazebo should open in a separate window.

You should see the Crazyflie attempting to follow the reference trajectory.

---

# Part 9 — Why Three Terminals Are Used

During the experiment, keep **three WSL Ubuntu terminals** open. Each terminal has a different purpose.

| Terminal | Purpose | Keep it open? |
|---|---|---|
| Terminal 1 | Create/start the Docker container and run the main ROS 2 simulation | Yes |
| Terminal 2 | Enter the same container and open the Gazebo graphical window | Yes |
| Terminal 3 | Enter the same container and change or inspect ROS 2 parameters | Yes |

All three terminals connect to the **same** container named:

```text
crazyflie_pd_demo
```

They are not three different simulations.

## How to recognize where you are

The text before the cursor tells you whether you are in WSL or Docker.

### WSL Ubuntu host prompt

It looks similar to:

```text
yujietang@TANGPC:/mnt/c/Users/t3426$
```

Run Docker-management commands here, including:

```bash
docker run ...
docker start crazyflie_pd_demo
docker exec -it crazyflie_pd_demo bash
docker stop crazyflie_pd_demo
```

### Docker container prompt

It looks similar to:

```text
root@docker-desktop:/workspace#
```

Run ROS 2, Gazebo, and experiment commands here, including:

```bash
source /opt/ros/jazzy/setup.bash
source /workspace/ros2_ws/install/setup.bash
gz sim -g
ros2 node list
ros2 param set ...
```

> **Important:** open window linux system by type:
> wsl


Terminal 1 — Main Simulation Terminal

Purpose: This terminal creates the Docker container and automatically starts the Crazyflie simulation backend, ROS 2 nodes, PD controller, trajectory generator, and Gazebo server.

From the WSL prompt, run:
```bash
docker run --rm -it \
  --name crazyflie_pd_demo \
  --network host \
  --gpus all \
  -e DISPLAY=$DISPLAY \
  -e LIBGL_ALWAYS_SOFTWARE=1 \
  -v /tmp/.X11-unix:/tmp/.X11-unix:rw \
  yujietang/crazyflie-gazebo-pd:latest
```
When you see the Crazyflie demo logs, the simulation is running. Keep Terminal 1 open. You will not get a command prompt in this terminal while the simulation is running.

Do not use this terminal to change parameters while the launch process is occupying it. Pressing `Ctrl + C` here stops the main simulation.

## Terminal 2 — Gazebo GUI terminal

**Purpose:** This terminal opens the Gazebo graphical client so the student can watch the Crazyflie.

Open a second WSL Ubuntu terminal and enter the already-running container:

```bash
docker exec -it crazyflie_pd_demo bash
```

Inside Docker, run:

```bash
source /opt/ros/jazzy/setup.bash
export QT_X11_NO_MITSHM=1
export LIBGL_ALWAYS_SOFTWARE=1
gz sim -g
```

Leave this terminal open while using the Gazebo window.

The message below is normally only a warning:

```text
QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-root'
```

If the Gazebo window opens, the warning can be ignored.

> Run `gz sim -g` only if `start_demo.sh` starts the Gazebo server without its GUI. If the Gazebo window already opened from Terminal 1, do not launch a second GUI.

## Terminal 3 — Controller and trajectory terminal

**Purpose:** This terminal lets the student change the experiment while the simulation continues to run.

Open a third WSL Ubuntu terminal and enter the same container:

```bash
docker exec -it crazyflie_pd_demo bash
```

Inside Docker, source ROS 2 and the built workspace:

```bash
source /opt/ros/jazzy/setup.bash
source /workspace/ros2_ws/install/setup.bash
```

Change the controller gains:

```bash
ros2 param set /pd_controller kp 2.0
ros2 param set /pd_controller kd 2.8
```

Select the figure-eight trajectory:

```bash
ros2 param set /trajectory_node trajectory figure8
```

Return to the circle trajectory:

```bash
ros2 param set /trajectory_node trajectory circle
```

The expected response is:

```text
Set parameter successful
```

You can also inspect the current values:

```bash
ros2 param get /pd_controller kp
ros2 param get /pd_controller kd
ros2 param get /trajectory_node trajectory
```

## Correct operating order

```text
Terminal 1: start the container and main simulation
                    ↓
Terminal 2: enter the same container and open Gazebo GUI
                    ↓
Terminal 3: enter the same container and change Kp, Kd, or trajectory
```

Start Terminal 2 and Terminal 3 only after `crazyflie_pd_demo` is running. Confirm this from WSL with:

```bash
docker ps
```

---

# Part 10 — Open Another Terminal and Enter the Running Container

Do **not** run another copy of the simulation.

Instead, open a second Ubuntu terminal.

Check the running container:

```bash
docker ps
```

You should see:

```text
crazyflie_pd_demo
```

Then enter it:

```bash
docker exec -it crazyflie_pd_demo bash
```

Your prompt should change to something similar to:

```text
root@DESKTOP:/workspace#
```

You are now inside the **same running simulation container**.

This second terminal can be used to inspect ROS 2 topics, nodes, parameters, and controller settings.

For example:

```bash
ros2 node list
```

You may see:

```text
/pd_controller
/trajectory_node
/ros_gz_bridge
```

Check ROS topics:

```bash
ros2 topic list
```

---

# Part 11 — Modify the PD Controller Parameters

Enter the running container:

```bash
docker exec -it crazyflie_pd_demo bash
```

Then go to the workspace:

```bash
cd /workspace/ros2_ws
```

The controller configuration should be located somewhere similar to:

```text
src/crazyflie_demo/config/controller.yaml
```

Open it with:

```bash
nano src/crazyflie_demo/config/controller.yaml
```

If `nano` is unavailable, use:

```bash
vi src/crazyflie_demo/config/controller.yaml
```

You may see parameters such as:

```yaml
kp: 2.0
kd: 1.0
```

The PD controller follows approximately:

```text
a_cmd = Kp × position_error + Kd × velocity_error
```

where:

```text
Kp = proportional gain
Kd = derivative gain
```

Try changing the values gradually.

For example:

```yaml
kp: 1.5
kd: 0.8
```

Avoid making extremely large changes.

---

# Part 12 — Important: What If the Drone Crashes or Falls?

Changing `Kp` and `Kd` can make the controller unstable.

For example, the Crazyflie may:

```text
fall to the ground
fly along the ground
oscillate violently
shoot upward
move very quickly
leave the visible area
```

This does **not necessarily mean Gazebo is broken**.

It usually means the controller parameters are poor or the simulation needs to be restarted.

## Step 1 — Stop the simulation

In the terminal running the simulation, press:

```text
Ctrl + C
```

Do not repeatedly launch another simulation while the old one is still running.

Otherwise, duplicate ROS nodes can appear.

---

## Step 2 — Check for remaining ROS nodes

Run:

```bash
ros2 node list
```

If you still see:

```text
/pd_controller
/trajectory_node
/ros_gz_bridge
```

after stopping the simulation, exit and restart the container.

---

## Step 3 — Restart the container

From the WSL Ubuntu host:

```bash
docker stop crazyflie_pd_demo
```

Then:

```bash
docker start crazyflie_pd_demo
```

Enter it again:

```bash
docker exec -it crazyflie_pd_demo bash
```

Then start the simulation again:

```bash
/start_demo.sh
```

---

# Part 13 — Starting the Container on Another Day

You only need to use:

```bash
docker run ...
```

the **first time** you create the container.

After the container already exists, do not create another container with the same name.

Check all containers:

```bash
docker ps -a
```

If you see:

```text
crazyflie_pd_demo
```

start it with:

```bash
docker start crazyflie_pd_demo
```

Then enter it:

```bash
docker exec -it crazyflie_pd_demo bash
```

Then run:

```bash
/start_demo.sh
```

Therefore, your normal workflow after the first installation is:

```bash
docker start crazyflie_pd_demo
docker exec -it crazyflie_pd_demo bash
```

Then, inside Docker:

```bash
/start_demo.sh
```

---

# Part 14 — Stop the Container

When you finish the experiment, first stop the simulation with:

```text
Ctrl + C
```

Then exit Docker:

```bash
exit
```

You will return to the WSL terminal:

```text
student@DESKTOP:~$
```

Stop the container:

```bash
docker stop crazyflie_pd_demo
```

---

# Part 15 — If You Want a Completely Fresh Container

If your experiment becomes badly configured and you want to return to the instructor's original environment, remove the container.

Stop it:

```bash
docker stop crazyflie_pd_demo
```

Remove it:

```bash
docker rm crazyflie_pd_demo
```

This does **not** delete the Docker image.

Check:

```bash
docker image ls
```

The image should still exist:

```text
yujitang/crazyflie-gazebo-pd:latest
```

You can then create a fresh container again using the `docker run` command from Part 7.

---

# Student Quick Reference

## First installation

Pull the image:

```bash
docker pull yujitang/crazyflie-gazebo-pd:latest
```

Check it:

```bash
docker image ls
```

Create the container:

```bash
docker run -it \
  --name crazyflie_pd_demo \
  --network host \
  --ipc host \
  -e DISPLAY=$DISPLAY \
  -e WAYLAND_DISPLAY=$WAYLAND_DISPLAY \
  -e XDG_RUNTIME_DIR=$XDG_RUNTIME_DIR \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  yujitang/crazyflie-gazebo-pd:latest
```

Start the experiment inside Docker:

```bash
/start_demo.sh
```

---

## Every later session

Open WSL Ubuntu:

```bash
docker start crazyflie_pd_demo
```

Enter the container:

```bash
docker exec -it crazyflie_pd_demo bash
```

Start the experiment:

```bash
/start_demo.sh
```

---

## Open a second terminal for debugging

```bash
docker exec -it crazyflie_pd_demo bash
```

Then:

```bash
ros2 node list
ros2 topic list
```

---

## Finish the experiment

Stop ROS/Gazebo:

```text
Ctrl + C
```

Exit Docker:

```bash
exit
```

Stop the container:

```bash
docker stop crazyflie_pd_demo
```

---

# Observation table

| Experiment | $K_p$ | $K_d$ | Response | Oscillation | Smoothness |
|---|---:|---:|---|---|---|
| Recommended | 2.0 | 2.8 |  |  |  |
| Slow response | 0.5 | 1.0 |  |  |  |
| Aggressive | 6.0 | 0.5 |  |  |  |
| More damping | 6.0 | 4.0 |  |  |  |

# Questions

1. What happened when you increased $K_p$?
2. What happened when you increased $K_d$?
3. Which values produced the smoothest circle?
4. Did the same values work equally well for the figure-eight?

# Main conclusion

> $K_p$ determines how strongly the Crazyflie follows the target. $K_d$ suppresses oscillation and makes the movement smoother.


# Important Student Rule

You do **not** need to clone the instructor's GitHub repository.

You do **not** need to build the Docker image.

The instructor has already prepared the complete environment.

Your workflow is simply:

```text
Docker Desktop
      ↓
WSL Ubuntu
      ↓
docker pull yujitang/crazyflie-gazebo-pd:latest
      ↓
docker run
      ↓
docker exec
      ↓
/start_demo.sh
      ↓
Gazebo + Crazyflie PD experiment


```
