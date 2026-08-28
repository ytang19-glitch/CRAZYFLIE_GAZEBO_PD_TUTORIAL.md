# Crazyflie Gazebo PD — Simple Student Tutorial

This tutorial shows how to install **WSL 2**, **Ubuntu**, and **Docker Desktop**, and then run the Crazyflie simulation with four terminals.

# Part 1 — Install WSL 2 and Ubuntu

## 1. Open PowerShell as Administrator

Open the Windows Start menu, search for **PowerShell**, right-click it, and select **Run as administrator**.

Run:

```powershell
wsl --install
```

Restart Windows if requested.

## 2. Start Ubuntu

Open **Ubuntu** from the Windows Start menu.

The first time Ubuntu starts, create a Linux username and password. The password will not appear while you type; this is normal.

## 3. Check WSL 2

In PowerShell, run:

```powershell
wsl -l -v
```

Confirm that Ubuntu shows `VERSION 2`:

```text
NAME      STATE      VERSION
Ubuntu    Running    2
```

If Ubuntu is not using version 2, run:

```powershell
wsl --set-version Ubuntu 2
```

---

# Part 2 — Install and Configure Docker Desktop

## 1. Install Docker Desktop

Download and install **Docker Desktop for Windows** from the official Docker website.

During installation, use the **WSL 2 backend** when the option is shown. Restart Windows if requested.

## 2. Start Docker Desktop

Open Docker Desktop and wait until the Docker engine is running.

Open:

```text
Docker Desktop → Settings → General
```

Enable **Use the WSL 2 based engine** if the option is available.

## 3. Enable Ubuntu integration

Open:

```text
Docker Desktop → Settings → Resources → WSL Integration
```

Enable **Ubuntu**, and then select **Apply & restart**.

If this page is missing, make sure Docker Desktop is using Linux containers.

## 4. Test Docker from WSL

Open Ubuntu and run:

```bash
docker version
docker run --rm hello-world
```

If you see `Hello from Docker!`, Docker Desktop and WSL are connected correctly.

> Do not install a second Docker Engine inside Ubuntu. Docker Desktop already provides Docker to WSL.

---

# Part 3 — Start the Experiment

## 1. Start Docker Desktop

Open Docker Desktop in Windows and wait until the Docker engine is running.

Keep Docker Desktop open during the experiment.

## 2. Open WSL Ubuntu

Open PowerShell or Windows Terminal and run:

```powershell
wsl
```

The WSL prompt looks similar to:

```text
yujietang@TANGPC:~$
```

## 3. Pull the image — first time only

From the WSL prompt, run:

```bash
docker pull yujietang/crazyflie-gazebo-pd:v3
```

After the image is downloaded, use the following four terminals.

---

# Terminal 1 — Run Docker and the main simulation

Purpose: Start the container, Gazebo server, ROS 2 nodes, trajectory generator, and PD controller.

First, confirm that you are inside WSL Ubuntu. Your prompt should look similar to:
```bash
yujietang@TANGPC:~$
```

Important: Run wsl only from Windows PowerShell. Do not type wsl again after entering Ubuntu.

## Step 1: Check for an old container

Before creating the container, check whether a container named crazyflie_pd_demo already exists:
```bash
docker ps -a --filter name=crazyflie_pd_demo
```
If no container appears, continue to Step 2.

If the container already exists, remove it:
```bash
docker rm -f crazyflie_pd_demo
```
This prevents the following error:

Conflict. The container name "/crazyflie_pd_demo" is already in use.

Removing the old container does not remove the downloaded Docker image. You do not need to pull the image again.

## Step 2: Run the container and simulation

**Purpose:** Start the container, Gazebo server, ROS 2 nodes, trajectory generator, and PD controller.

Open a WSL terminal and run:

```bash
docker run --rm -it \
  --name crazyflie_pd_demo \
  --network host \
  --gpus all \
  -e DISPLAY=$DISPLAY \
  -e LIBGL_ALWAYS_SOFTWARE=1 \
  -v /tmp/.X11-unix:/tmp/.X11-unix:rw \
  yujietang/crazyflie-gazebo-pd:v3
```

Keep Terminal 1 open. Pressing `Ctrl+C` here stops the simulation.

---

# Terminal 3 — Open the Gazebo Window

**Purpose:** Display the Crazyflie simulation.

Open another WSL terminal and enter the running container:

```bash
docker exec -it crazyflie_pd_demo bash
```

Wait until the prompt looks similar to:

```text
root@docker-desktop:/workspace#
```

Then run:

```bash
source /opt/ros/jazzy/setup.bash
export QT_X11_NO_MITSHM=1
export LIBGL_ALWAYS_SOFTWARE=1
gz sim -g
```

Keep Terminal 2 open while using the Gazebo window.

> Do not run `source /opt/ros/jazzy/setup.bash` or `gz sim -g` at the WSL prompt. These commands must run inside the Docker container.

---

# Terminal 4 — Set Kp and Kd

**Purpose:** Change the PD controller gains while the simulation is running.

Open another WSL terminal and enter the same container:

```bash
docker exec -it crazyflie_pd_demo bash
```

Inside the container, run:

```bash
source /opt/ros/jazzy/setup.bash
```

Set the controller gains:

```bash
ros2 param set /pd_controller kp 2.0
ros2 param set /pd_controller kd 2.8
```

Check the current gains:

```bash
ros2 param get /pd_controller kp
ros2 param get /pd_controller kd
```

The expected response after changing a value is:

```text
Set parameter successful
```

---

# Terminal 5 — Debug the Simulation

**Purpose:** Check the container, ROS 2 nodes, topics, drone position, and controller output.

Open another WSL terminal and enter the same container:

```bash
docker exec -it crazyflie_pd_demo bash
```

Inside the container, run:

```bash
source /opt/ros/jazzy/setup.bash
```

Use these debugging commands:

```bash
ros2 node list
ros2 topic list
ros2 topic echo /cf/odom --field pose.pose.position
ros2 topic echo /cf/wrench --once
```

Press `Ctrl+C` to stop a continuously running `ros2 topic echo` command.

---

# Correct Operating Order

1. **Terminal 1:** Start Docker and the main simulation.
2. **Terminal 2:** Open the Gazebo window.
3. **Terminal 3:** Set or inspect `Kp` and `Kd`.
4. **Terminal 4:** Debug nodes, topics, position, and controller output.

Terminals 2–4 connect to the same container. Do not run the main Docker command again in those terminals.

# Stop the Experiment

Return to Terminal 1 and press:

```text
Ctrl+C
```

Because the Docker command uses `--rm`, the stopped container is removed automatically. The downloaded image remains available for the next experiment.
