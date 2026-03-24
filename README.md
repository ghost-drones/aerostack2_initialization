# Aerostack2 Initialization

Docker-based environment setup for autonomous drone development using [Aerostack2](https://aerostack2.github.io/), ROS2 Humble, and Gazebo Harmonic.

## Overview

This repository provides Docker configurations and an initialization script to quickly set up isolated development environments for drone robotics. It supports both simulation and hardware workflows with two autopilot backends.

## Supported Configurations

| Platform | Mode       | Simulator       |
|----------|------------|-----------------|
| PX4      | Simulation | Gazebo Harmonic |
| PX4      | Hardware   | —               |
| Mavlink  | Simulation | Gazebo Harmonic |
| Mavlink  | Hardware   | —               |

## Prerequisites

- Docker Engine (see [installation guide](#docker-engine-installation) below)
- (Optional) NVIDIA GPU with [nvidia-container-toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) for GPU acceleration (see [installation guide](#nvidia-container-toolkit-installation-optional) below)

---

## Docker Engine Installation

<details>
<summary><b>Click here if you need to install Docker Engine.</b></summary>
<br>

Follow the steps below to install Docker Engine on Ubuntu.

Source: [Docker Engine Installation](https://docs.docker.com/engine/install/ubuntu/)

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

---

### Running Docker without sudo

After installation, configure Docker to run without `sudo`.

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
sudo reboot
```

After rebooting:

```bash
newgrp docker
```

</details>

---

## NVIDIA Container Toolkit Installation (Optional)

<details>
<summary><b>Click here if you have an NVIDIA GPU and want GPU acceleration.</b></summary>
<br>

Source: [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg && \
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

**Enable experimental packages (optional):**

```bash
sed -i -e '/experimental/ s/^#//g' /etc/apt/sources.list.d/nvidia-container-toolkit.list
```

**Install NVIDIA Container Toolkit:**

```bash
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
```

**Configure Docker to use NVIDIA runtime:**

```bash
sudo nvidia-ctk runtime configure --runtime=docker
```

**Restart the Docker daemon:**

```bash
sudo systemctl restart docker
```

</details>

---

## Enabling Graphical Applications in the Container

Before running any container with GUI support (Gazebo, RViz, etc.), allow Docker to access your display:

```bash
xhost +local:docker
```

To avoid running this every time:

```bash
echo "xhost +local:docker" >> ~/.bashrc
```

---

## Quick Start

1. Clone the repository:

```bash
git clone git@github.com:ghost-drones/aerostack2_tutorial.git
```

2. Enable BuildKit:

```bash
export DOCKER_BUILDKIT=1
```

3. Run the initialization script:

```bash
bash first_run.bash
```

The script will prompt you for:

1. **Project/drone name** — used as the container name
2. **Platform** — `PX4` or `Mavlink`
3. **Mode** — `Hardware` or `Simulation`
4. **Simulator** — `GZ-Harmonic` or `Isaac-Sim` (simulation mode only)
5. **GPU availability** — enables NVIDIA GPU passthrough if available

The script then builds the appropriate Docker image (approximately 40 minutes — cloning PX4 takes a while) and launches the container with:

- X11 forwarding for GUI applications (Gazebo, RViz, etc.)
- SSH agent forwarding for GitHub access
- Privileged mode for hardware access
- A `goto_<name>` bash alias added to your `~/.bash_aliases`

---

## Managing the Container

> `first_run.bash` handles container creation automatically. The commands below are for reference and day-to-day use.
 
Your container will be named `<CONT_NAME>_drone_env_cont`, where `<CONT_NAME>` is the project/drone name you provided during setup.

**Reuse the container after the first run:**

```bash
docker start -i <CONT_NAME>_drone_env_cont && docker exec -it <CONT_NAME>_drone_env_cont /bin/bash
```

**Add a convenient alias to** `.bashrc`:

```bash
echo "alias aerostack2_tutorial='docker start -i <CONT_NAME>_drone_env_cont && docker exec -it <CONT_NAME>_drone_env_cont /bin/bash'" >> ~/.bashrc
```

<details>
<summary><b>Other useful Docker commands</b></summary>
<br>

**Exit the container:**

```bash
exit
```

**Open another shell in a running container:**

```bash
docker exec -it <CONT_NAME>_drone_env_cont bash
```

**Stop the container:**

```bash
docker stop <CONT_NAME>_drone_env_cont
```

**Remove the container:**

```bash
docker rm <CONT_NAME>_drone_env_cont
```

> If the container is removed, recreate it by running `first_run.bash` again.

</details>

---

## Repository Structure

```
.
├── first_run.bash                        # Main initialization script
├── Dockerfile_PX4_Simulation_GZ-Harmonic
├── Dockerfile_PX4_Hardware
├── Dockerfile_Mavlink_Simulation_GZ-Harmonic
├── Dockerfile_Mavlink_Hardware
├── models/                               # Gazebo drone models
│   ├── x500/                            # Generic NXP HoverGames x500
│   ├── x500_as2/                        # Aerostack2-specific variant
│   └── x500_px4/                        # PX4-specific variant
└── to_copy/                              # Config files copied into containers
    ├── aliases                           # Bash aliases and helper functions
    └── tmux                              # Tmux configuration
```

## What Gets Installed

All Docker images include:

- ROS2 Humble
- [Aerostack2](https://github.com/ghost-drones/aerostack2) (ghost-drones fork)
- Colcon build tools
- tmux / tmuxinator
- Code quality tools: flake8, pylint, cppcheck, lcov

**PX4 images** additionally include:

- PX4 Autopilot v1.16 (simulation images)
- Micro-XRCE-DDS-Agent
- px4_msgs, px4_ros_com
- as2_platform_pixhawk

**Mavlink images** additionally include:

- MAVROS
- GeographicLib datasets
- as2_platform_mavlink

**Simulation images** additionally include:

- Gazebo Harmonic
- ros-gzharmonic bridge

## Drone Models

Three variants of the [NXP HoverGames x500](https://www.nxp.com/design/designs/nxp-hovergames-drone-kit-including-rddrone-fmuk66-and-peripherals:KIT-HGDRONEK66) quadcopter are provided as Gazebo SDF models:

- `x500` — base model
- `x500_as2` — configured for Aerostack2
- `x500_px4` — configured for PX4 SITL

## In-Container Aliases

| Alias / Function    | Description                                                                   |
|---------------------|-------------------------------------------------------------------------------|
| `build_aerostack2`  | Build the Aerostack2 workspace (Release, symlink-install, 3 parallel workers) |
| `kill_ros2`         | Kill all running ROS2 nodes                                                   |
| `waitForRos`        | Wait until ROS2 topics are available                                          |
| `killp <pid>`       | Recursively kill a process tree                                               |
| `refresh`           | Re-source `~/.bashrc`                                                         |

---

## VS Code — Dev Containers

Install the [Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers) extension in VS Code to improve the development experience inside the container.

To hide unwanted dotfiles in the file explorer:

1. Right-click on the file explorer tab → **Open Folder Settings**
2. Search for **"Files: Exclude"** and add:

```json
"**/.*"
```

---

## Tech Stack

- **ROS2 Humble** — robotics middleware
- **Aerostack2** — autonomous aerial systems framework
- **PX4 Autopilot v1.16** — flight controller firmware
- **Gazebo Harmonic** — physics simulation
- **Docker** — containerized environments
- **Micro-XRCE-DDS-Agent** — DDS bridge for PX4 offboard communication
- **MAVROS** — MAVLink to ROS bridge