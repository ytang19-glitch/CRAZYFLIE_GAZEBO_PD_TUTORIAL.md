# Student Tutorial: Run the Crazyflie PD Demo on Windows

## What you will do

You will install WSL 2 and Docker Desktop, download a prepared Crazyflie demo image, run Gazebo from WSL, adjust $K_p$ and $K_d$, and switch between circle and figure-eight trajectories.

You will **not** build the image or modify ROS 2 source code.

The instructor has already packaged the following inside the image:

- ROS 2 Jazzy
- Gazebo Harmonic
- the Crazyflie model and mesh files
- the ROS–Gazebo bridge
- the circle and figure-eight trajectory node
- the Python PD controller
- the automatic startup script

## Requirements

- Windows 11 or a supported Windows 10 version
- About 10 GB of free disk space
- Internet access for the first download
- Hardware virtualization enabled
- Permission to install WSL and Docker Desktop

Replace this placeholder in every command with the image name supplied by the instructor:

```text
DOCKERHUB_USERNAME/crazyflie-gazebo-pd:latest
```

Example only:

```text
yujietang/crazyflie-gazebo-pd:latest
```

Do not use the example unless it is the exact image name supplied by the instructor.

---

# Part 1: Install WSL 2 and Ubuntu

## 1. Open PowerShell as Administrator

Open the Windows Start menu, search for **PowerShell**, right-click it, and select **Run as administrator**.

## 2. Install WSL and Ubuntu

```powershell
wsl --install
```

Restart Windows when requested. The first time Ubuntu opens, create a Linux username and password. The password does not appear while you type; this is normal.

Official guide: [Install WSL](https://learn.microsoft.com/windows/wsl/install)

## 3. Confirm WSL 2

In PowerShell:

```powershell
wsl -l -v
```

Expected example:

```text
NAME      STATE     VERSION
Ubuntu    Stopped   2
```

If Ubuntu shows version `1`:

```powershell
wsl --set-version Ubuntu 2
```

Update WSL:

```powershell
wsl --update
```

Restart WSL after updating:

```powershell
wsl --shutdown
```

If `wsl --install` only displays help, list the available distributions and install Ubuntu explicitly:

```powershell
wsl --list --online
wsl --install -d Ubuntu
```

---

# Part 2: Install Docker Desktop

## 1. Download Docker Desktop

Download and install [Docker Desktop for Windows](https://docs.docker.com/desktop/setup/install/windows-install/). Use the WSL 2 backend when that option is shown.

## 2. Enable the WSL 2 engine

Start Docker Desktop and open:

```text
Settings → General
```

Enable **Use the WSL 2 based engine**. On current installations it may already be enabled, so the option may not appear.

## 3. Enable Ubuntu integration

Open:

```text
Settings → Resources → WSL Integration
```

Enable Ubuntu and select **Apply & restart**.

If this page is missing, Docker Desktop may be using Windows containers. Switch it to Linux containers.

Official guide: [Docker Desktop WSL 2 backend](https://docs.docker.com/desktop/features/wsl/)

## 4. Test the integration

Open **Ubuntu** from the Windows Start menu and run:

```bash
docker version
docker run --rm hello-world
```

Do not install another Docker Engine inside Ubuntu. Docker Desktop supplies the engine through WSL integration.

---

# Part 3: Pull the instructor's image from Docker Desktop

For this tutorial, use the **Docker Desktop graphical interface** to download the prepared image. Do not clone the source repository and do not build a Docker image.

Before starting, copy the complete image name supplied by the instructor. It should look similar to:

```text
DOCKERHUB_USERNAME/crazyflie-gazebo-pd:latest
```
The username, repository name and tag must all be correct.
Eg: account name is yujietang in this experiment.So:

```bash
yujietang/crazyflie-gazebo-pd:latest
```

## 1. Start Docker Desktop

Open **Docker Desktop** from the Windows Start menu and wait until the Docker engine reports that it is running.

Do not close Docker Desktop while using the simulation.

## 2. Sign in when required

- A public image can normally be pulled without signing in.
- For a private image, select **Sign in** and use the Docker Hub account that the instructor has authorized.

## 3. Find the instructor's image

1. Select **Images** in the Docker Desktop sidebar.
2. Open the **Docker Hub repositories** or image search view.
3. Paste the complete image name supplied by the instructor into the search box.
4. Select the correct repository and the `latest` tag, unless the instructor supplied a different tag.

Do not select another image with a similar name.

## 4. Pull the image

Select **Pull** beside the required tag. Wait until every layer finishes downloading.

If Docker Desktop displays **Run** instead of **Pull**, the image may already be downloaded. Check **Images → Local**.

## 5. Confirm that the image is local

Under **Images → Local**, confirm that the repository and tag match the instructor's image, for example:

```text
Repository: DOCKERHUB_USERNAME/crazyflie-gazebo-pd
Tag:        latest
```

## 6. Confirm from WSL Ubuntu

Open Ubuntu and run:

```bash
docker image ls
```

The same image should appear in the list. Docker Desktop and an integrated WSL Ubuntu distribution share the same Docker images, so there is no need to download it again inside WSL.

You can also check the exact image directly:

```bash
docker image inspect DOCKERHUB_USERNAME/crazyflie-gazebo-pd:latest > /dev/null && echo "IMAGE READY"
```

Expected result:

```text
IMAGE READY
```

If Docker Desktop cannot find the image, check the exact spelling and tag with the instructor. As a command-line fallback, run this once in Ubuntu:

```bash
docker pull DOCKERHUB_USERNAME/crazyflie-gazebo-pd:latest
```

Students do not need to download the GitHub source repository, edit ROS 2 files or run `docker build`.

---

# Part 4: Run the Gazebo demo

## 1. Check WSL graphics

In Ubuntu:

```bash
echo $DISPLAY
```

It should return a value. Windows 11 normally provides Linux GUI support through WSLg.

Also check:

```bash
echo $WAYLAND_DISPLAY
```

If both variables are empty, run the following in Administrator PowerShell and then reopen Ubuntu:

```powershell
wsl --update
wsl --shutdown
```

Official guide: [Run Linux GUI applications with WSL](https://learn.microsoft.com/windows/wsl/tutorials/gui-apps)

## 2. Start the demo

```bash
docker run --rm -it \
  --name crazyflie_pd_demo \
  --network host \
  --ipc host \
  -e DISPLAY=$DISPLAY \
  -e QT_X11_NO_MITSHM=1 \
  -e LIBGL_ALWAYS_SOFTWARE=1 \
  -v /tmp/.X11-unix:/tmp/.X11-unix:rw \
  DOCKERHUB_USERNAME/crazyflie-gazebo-pd:latest
```

The prepared image automatically starts:

- ROS 2 Jazzy
- Gazebo Harmonic
- the Crazyflie model
- the ROS–Gazebo bridge
- the trajectory generator
- the PD controller

Keep this terminal open. Successful startup should include:

```text
Entity creation successful
Crazyflie trajectory publisher started
Crazyflie PD controller started
```

## 3. Open the Gazebo window

If the Gazebo window opens automatically, continue to Part 5.

If the image starts the Gazebo server without opening the window, open a second Ubuntu terminal:

```bash
docker exec -it crazyflie_pd_demo bash
```

Inside the container:

```bash
source /opt/ros/jazzy/setup.bash
export QT_X11_NO_MITSHM=1
export LIBGL_ALWAYS_SOFTWARE=1
gz sim -g
```

Keep this terminal open. The `-g` option opens only the graphical client and connects it to the Gazebo server that is already running.

---

# Part 5: Change $K_p$ and $K_d$

Open another Ubuntu terminal:

```bash
docker exec -it crazyflie_pd_demo bash
```

If the image does not automatically source ROS 2:

```bash
source /opt/ros/jazzy/setup.bash
source /workspace/ros2_ws/install/setup.bash
```

Check the nodes before changing parameters:

```bash
ros2 node list
```

Expected output:

```text
/pd_controller
/ros_gz_bridge
/trajectory_node
```

If the node list is empty but the simulation processes are running, reset the ROS 2 discovery cache:

```bash
ros2 daemon stop
ros2 daemon start
sleep 5
ros2 node list
```

Do not run `start_demo.sh` or `ros2 launch` again. The prepared image has already started the complete launch file.

## Experiment 1: Recommended values

```bash
ros2 param set /pd_controller kp 2.0
ros2 param set /pd_controller kd 2.8
```

## Experiment 2: Slow response

```bash
ros2 param set /pd_controller kp 0.5
ros2 param set /pd_controller kd 1.0
```

Does the drone follow the target slowly or fall behind it?

## Experiment 3: Fast but oscillating

```bash
ros2 param set /pd_controller kp 6.0
ros2 param set /pd_controller kd 0.5
```

Does it respond faster but overshoot or oscillate?

## Experiment 4: Add damping

Keep $K_p=6.0$ and increase $K_d$:

```bash
ros2 param set /pd_controller kd 4.0
```

Does the movement become smoother?

## Restore recommended values

After each unstable experiment, restore:

```bash
ros2 param set /pd_controller kp 2.0
ros2 param set /pd_controller kd 2.8
```

---

# Part 6: Change the trajectory

Figure-eight:

```bash
ros2 param set /trajectory_node trajectory figure8
```

Circle:

```bash
ros2 param set /trajectory_node trajectory circle
```

Each successful command should display:

```text
Set parameter successful
```

---

# Part 7: Observe the simulation data

Actual Crazyflie position:

```bash
ros2 topic echo /cf/odom --field pose.pose.position
```

Desired trajectory position:

```bash
ros2 topic echo /cf/reference --field pose.pose.position
```

Current controller parameters:

```bash
ros2 param get /pd_controller kp
ros2 param get /pd_controller kd
```

Press `Ctrl+C` to stop a topic display. This does not stop the complete simulation.

---

# Part 8: Stop and restart

Return to the first terminal and press `Ctrl+C`.

The stopped container is removed automatically because it was started with `--rm`. The downloaded image remains available.

Run the same `docker run` command to start again. You only need to pull again when the instructor publishes an updated image.

To download an updated image, open Docker Desktop, go to the instructor's repository under **Images**, and select **Pull** for the required tag again. Then start a new container using the same `docker run` command.

---

# Quick troubleshooting

## `docker` is not found in Ubuntu

Start Docker Desktop, enable **Settings → Resources → WSL Integration → Ubuntu**, select **Apply & restart**, and reopen Ubuntu.

## Cannot connect to Docker

Make sure Docker Desktop is running. In PowerShell:

```powershell
wsl --shutdown
```

Reopen Ubuntu and test:

```bash
docker version
```

## Gazebo does not appear

Check in Ubuntu:

```bash
echo $DISPLAY
echo $WAYLAND_DISPLAY
```

Then update and restart WSL from PowerShell:

```powershell
wsl --update
wsl --shutdown
```

Open Ubuntu and run the container again.

If the server starts but the window is missing, use Part 4 to enter the container and run `gz sim -g`.

## Image cannot be pulled

In Docker Desktop, check that:

- the complete image name and tag are correct;
- Docker Desktop shows that its engine is running;
- the computer has internet access; and
- you are signed in to an authorized account if the image is private.

For a private image, you may also sign in from Ubuntu:

```bash
docker login
```

Then retry in Docker Desktop, or use:

```bash
docker pull DOCKERHUB_USERNAME/crazyflie-gazebo-pd:latest
```

## Drone becomes unstable

Restore safe gains:

```bash
ros2 param set /pd_controller kp 2.0
ros2 param set /pd_controller kd 2.8
```

## `Node not found`

Check the node list:

```bash
ros2 node list
```

If it is empty:

```bash
ros2 daemon stop
ros2 daemon start
sleep 5
ros2 node list
```

---

# Observation table

Students should use this table to record how different parameter combinations affect the drone.

For each parameter set:

1. Select the circle trajectory.
2. Observe the drone for approximately 20 seconds.
3. Record speed, overshoot, oscillation and smoothness.
4. Repeat using the figure-eight trajectory.
5. Compare whether the same gains work equally well for both paths.

| Experiment | Trajectory | $K_p$ | $K_d$ | Response speed | Overshoot | Oscillation | Smoothness |
|---|---|---:|---:|---|---|---|---|
| Recommended | Circle | 2.0 | 2.8 | | | | |
| Recommended | Figure eight | 2.0 | 2.8 | | | | |
| Slow response | Circle | 0.5 | 1.0 | | | | |
| Slow response | Figure eight | 0.5 | 1.0 | | | | |
| Aggressive | Circle | 6.0 | 0.5 | | | | |
| Aggressive | Figure eight | 6.0 | 0.5 | | | | |
| More damping | Circle | 6.0 | 4.0 | | | | |
| More damping | Figure eight | 6.0 | 4.0 | | | | |

Suggested rating scale:

```text
1 = very low / very poor
2 = low
3 = moderate
4 = high / good
5 = very high / very good
```

# Questions

1. What happened when you increased $K_p$?
2. What happened when you increased $K_d$?
3. Which values produced the smoothest circle?
4. Which values followed the target most quickly?
5. Which values produced the largest oscillation?
6. Did the same values work equally well for the figure-eight?
7. Why might the figure-eight require more damping than the circle?

# Main conclusion

> $K_p$ determines how strongly the Crazyflie corrects position error and follows the target. Increasing $K_p$ usually makes the response faster, but excessive $K_p$ can cause overshoot and oscillation. $K_d$ responds to velocity error, adds damping, suppresses oscillation and can make the movement smoother.
