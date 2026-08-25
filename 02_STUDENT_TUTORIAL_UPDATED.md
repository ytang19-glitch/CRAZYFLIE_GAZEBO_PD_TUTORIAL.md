# Student Tutorial: Run the Crazyflie PD Demo on Windows

## What you will do

You will install WSL 2 and Docker Desktop, download a prepared Crazyflie demo image, run Gazebo from WSL, adjust $K_p$ and $K_d$, and switch between circle and figure-eight trajectories.

You will **not** build the image or modify ROS 2 source code.

## Requirements

- Windows 11 or a supported Windows 10 version
- About 10 GB of free disk space
- Internet access for the first download
- Hardware virtualization enabled

This tutorial uses the instructor's published Docker image:

```text
yujietang/crazyflie-gazebo-pd:latest
```

> **Important:** Run all `docker pull`, `docker run`, and `docker exec` commands in the **WSL Ubuntu terminal**, not directly in Windows PowerShell. PowerShell and Linux Bash use different multiline syntax.

## Know which terminal you are using

Windows PowerShell normally looks like this:

```text
PS C:\Users\student>
```

WSL Ubuntu normally looks like this:

```text
student@computer:~$
```

Commands marked `powershell` must run in PowerShell. Commands marked `bash` must run in WSL Ubuntu.

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

# Part 3: Download the prepared image

## Option A: WSL terminal — recommended

In Ubuntu:

```bash
docker pull yujietang/crazyflie-gazebo-pd:latest
docker image ls
```

A public image does not require a Docker Hub login.

## Option B: Docker Desktop

1. Open Docker Desktop.
2. Open **Images**.
3. Search for `yujietang/crazyflie-gazebo-pd`.
4. Select **Pull**.
5. Return to Ubuntu for the remaining commands.

The downloaded image is shared between Docker Desktop and the integrated WSL terminal.

---

# Part 4: Run the Gazebo demo

## 1. Check WSL graphics

In Ubuntu:

```bash
echo $DISPLAY
```

It should return a value. Windows 11 normally provides Linux GUI support through WSLg.

## 2. Start the demo

Confirm that your prompt looks like `student@computer:~$`. If it still shows `C:\Users\...>`, enter WSL first:

```powershell
wsl
```

Now copy and paste the **entire command below at once** into WSL Ubuntu:

```bash
docker run --rm -it \
  --name crazyflie_pd_demo \
  --network host \
  -e DISPLAY=$DISPLAY \
  -e QT_X11_NO_MITSHM=1 \
  -e LIBGL_ALWAYS_SOFTWARE=1 \
  -v /tmp/.X11-unix:/tmp/.X11-unix:rw \
  yujietang/crazyflie-gazebo-pd:latest
```

The backslash (`\`) means “continue this command on the next line” in Linux Bash. Do not run its individual lines separately.

The prepared image automatically starts:

- ROS 2 Jazzy
- Gazebo Harmonic
- the Crazyflie model
- the trajectory generator
- the PD controller

Wait for Gazebo to open. The Crazyflie should begin following a circle. Keep this terminal open.

> Instructor requirement: the published image must define an entrypoint that launches the complete demo automatically.

---

# Part 5: Change $K_p$ and $K_d$

Open a second Ubuntu terminal:

```bash
docker exec -it crazyflie_pd_demo bash
```

If the image does not automatically source ROS 2:

```bash
source /opt/ros/jazzy/setup.bash
source /workspace/ros2_ws/install/setup.bash
```

## Safety rule before changing gains

The Crazyflie may fall when $K_p$ or $K_d$ is too small, too large, or badly balanced. Gazebo may show the fall without printing an error because the software has not necessarily crashed—the simulated drone simply became physically unstable or could no longer produce enough corrective force.

Before every experiment:

1. Change only to the values listed in this tutorial.
2. Keep the first Ubuntu terminal visible so you can press `Ctrl+C` quickly.
3. Change one experiment at a time and observe the drone for several seconds.
4. Record the gain values before trying another pair.

> A fallen drone usually cannot recover just because safe gains are restored. Its position, velocity, or orientation may already be outside the controller's recoverable range. Restart the simulation to reset the drone.

## Emergency recovery: drone falls, disappears, or moves violently

If the drone drops or becomes uncontrollable:

1. **Stop the simulation immediately.** Return to the first Ubuntu terminal and press:

   ```text
   Ctrl+C
   ```

2. Wait until the command finishes. Because the container uses `--rm`, it should be removed automatically.

3. Confirm that it stopped:

   ```bash
   docker ps
   ```

   `crazyflie_pd_demo` should not be listed.

4. If the container is still running, stop it from another WSL terminal:

   ```bash
   docker stop crazyflie_pd_demo
   ```

5. Start a fresh simulation using the complete `docker run` command from **Part 4**. This resets the drone and reloads the default safe gains.

6. After the new container starts, verify the gains:

   ```bash
   docker exec -it crazyflie_pd_demo bash
   ros2 param get /pd_controller kp
   ros2 param get /pd_controller kd
   ```

   The recommended values are:

   ```text
   kp = 2.0
   kd = 2.8
   ```

7. Write down the values that caused the fall. Do not repeat them until you understand why the response became unstable.

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

> **Caution:** This pair is intentionally aggressive and may make the drone oscillate or fall. Be ready to follow the emergency recovery procedure above.

```bash
ros2 param set /pd_controller kp 6.0
ros2 param set /pd_controller kd 0.5
```

Does it respond faster but overshoot or oscillate?

## Experiment 4: Add damping

```bash
ros2 param set /pd_controller kd 4.0
```

Does the movement become smoother?

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

---

# Part 7: Stop and restart

Return to the first terminal and press `Ctrl+C`.

The stopped container is removed automatically because it was started with `--rm`. The downloaded image remains available.

Run the same `docker run` command to start again. You only need to pull again when the instructor publishes an updated image.

---

# Quick troubleshooting

## `docker` is not found in Ubuntu

Start Docker Desktop, enable **Settings → Resources → WSL Integration → Ubuntu**, select **Apply & restart**, and reopen Ubuntu.

## Cannot connect to Docker

If you see an error mentioning `dockerDesktopLinuxEngine` or `The system cannot find the file specified`, the Docker command is installed but the Docker Desktop engine is not running.

1. Open Docker Desktop from the Windows Start menu.
2. Wait until Docker Desktop reports that the engine is running.
3. Test in WSL Ubuntu:

```bash
docker version
docker run --rm hello-world
```

If it still cannot connect, close WSL and run this in PowerShell:

```powershell
wsl --shutdown
```

Restart Docker Desktop, reopen Ubuntu, and test again:

```bash
docker version
```

## PowerShell says `-e` or `-v` is not recognized

This happens when a Linux multiline Docker command is pasted into Windows PowerShell. PowerShell treats each new line as a new command, and `\` is not PowerShell's continuation character.

Do not execute lines such as these by themselves:

```text
-e LIBGL_ALWAYS_SOFTWARE=1 \
-v /tmp/.X11-unix:/tmp/.X11-unix:rw \
```

Enter WSL Ubuntu first:

```powershell
wsl
```

Then paste the complete `docker run` command from **Part 4**. Verify that the prompt changed from `C:\Users\...>` to something like `student@computer:~$`.

## Container name is already in use

Check existing containers:

```bash
docker ps -a
```

If an old container named `crazyflie_pd_demo` is stopped, remove only that container:

```bash
docker rm crazyflie_pd_demo
```

Then run the Part 4 command again.

## Gazebo does not appear

Check in Ubuntu:

```bash
echo $DISPLAY
```

Then update and restart WSL from PowerShell:

```powershell
wsl --update
wsl --shutdown
```

Open Ubuntu and run the container again.

## Image cannot be pulled

Check the complete image name. If the instructor uses a private image:

```bash
docker login
```

## Drone becomes unstable

If the drone is still flying and only oscillating slightly, restore the safe gains:

```bash
ros2 param set /pd_controller kp 2.0
ros2 param set /pd_controller kd 2.8
```

If the drone has already hit the ground, flipped, disappeared, or stopped responding, restoring the gains is not enough. Press `Ctrl+C` in the first terminal and restart the container using the Part 4 command.

## Drone falls but no error is printed

This can be normal in a physics simulation. ROS 2 and Gazebo may still be running correctly while the controller produces an unstable physical response. A fall is therefore an **experiment result**, not always a software error.

Common causes include:

- $K_p$ is too low, so the controller cannot correct position error quickly enough.
- $K_p$ is too high, causing overshoot or violent motion.
- $K_d$ is too low, so there is not enough damping.
- $K_d$ is too high, making the response excessively resistant or noisy.
- The selected $K_p$ and $K_d$ are poorly balanced.
- The command reaches its force or acceleration limit, so the drone cannot recover.

Record the gain values, restart the simulation, and return to `kp = 2.0` and `kd = 2.8`.

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
