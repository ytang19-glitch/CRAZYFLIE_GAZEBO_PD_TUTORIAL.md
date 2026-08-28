# Upload and Verify the Crazyflie Docker Image (v3)

This guide explains how to upload the prepared Crazyflie Gazebo image to Docker Hub and verify that students can download it without building locally.

## Image used by this tutorial

```text
yujietang/crazyflie-gazebo-pd:v3
```

> Run each command separately. Wait until the prompt returns before running the next command.

## 1. Start Docker Desktop

Open Docker Desktop in Windows and wait until it reports that the Docker Engine is running.

## 2. Confirm that v3 exists locally

Open PowerShell and run:

```powershell
docker images yujietang/crazyflie-gazebo-pd
```

Confirm that the output contains the `v3` tag.

> `docker images` only confirms that the image exists on the computer. It does not prove that the image has been uploaded to Docker Hub.

## 3. Sign in to Docker Hub

```powershell
docker login -u yujietang
```

Complete the sign-in process. A successful login displays:

```text
Login Succeeded
```

## 4. Upload v3

```powershell
docker push yujietang/crazyflie-gazebo-pd:v3
```

The image is large, so the upload may take a long time. Do not close Docker Desktop or press `Ctrl+C`.

A successful upload ends with output similar to:

```text
v3: digest: sha256:...
```

## 5. Verify the upload from the command line

```powershell
docker manifest inspect yujietang/crazyflie-gazebo-pd:v3
```

If Docker returns manifest information in JSON format, `v3` exists on Docker Hub.

If Docker displays `manifest unknown`, the upload did not finish successfully. Run the push command again:

```powershell
docker push yujietang/crazyflie-gazebo-pd:v3
```

## 6. Verify the upload on Docker Hub

Open:

https://hub.docker.com/r/yujietang/crazyflie-gazebo-pd/tags

Confirm that the tag list contains:

```text
v3
```

## 7. Student Compose file (no build)

Save the following as `compose.student.yaml`:

```yaml
services:
  crazyflie_demo:
    image: yujietang/crazyflie-gazebo-pd:v3
    pull_policy: always
    container_name: crazyflie_pd_demo

    network_mode: host
    ipc: host

    environment:
      DISPLAY: ${DISPLAY}
      QT_X11_NO_MITSHM: "1"
      LIBGL_ALWAYS_SOFTWARE: "1"

    volumes:
      - /tmp/.X11-unix:/tmp/.X11-unix:rw

    working_dir: /workspace

    stdin_open: true
    tty: true

    command: bash /workspace/start_demo.sh
```

This student file intentionally does not contain:

```yaml
build: .
```

It also does not mount the student's local directory over `/workspace`.

## 8. Student download and launch commands

Students should open PowerShell in the directory containing `compose.student.yaml` and run each command separately:

```powershell
docker compose -f .\compose.student.yaml config
```

```powershell
docker compose -f .\compose.student.yaml pull
```

```powershell
docker compose -f .\compose.student.yaml up --force-recreate
```

To stop and remove the tutorial container:

```powershell
docker compose -f .\compose.student.yaml down
```

## Common mistake: combining commands

Do not paste two commands on the same line. For example, this is incorrect:

```text
docker compose ... up --force-recreatedocker compose ... config
```

Docker interprets `--force-recreatedocker` as one invalid option. Run one command, press Enter, wait for it to finish, and then run the next command.

