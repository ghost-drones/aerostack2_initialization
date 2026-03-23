# Aerostack2 Initialization

Docker-based environment setup for autonomous drone development using [Aerostack2](https://aerostack2.github.io/), ROS2 Humble, and Gazebo Harmonic.

## Overview

This repository provides Docker configurations and an initialization script to quickly set up isolated development environments for drone robotics. It supports both simulation and hardware workflows with two autopilot backends.

## Supported Configurations

| Platform | Mode       | Simulator      |
|----------|------------|----------------|
| PX4      | Simulation | Gazebo Harmonic |
| PX4      | Hardware   | —              |
| Mavlink  | Simulation | Gazebo Harmonic |
| Mavlink  | Hardware   | —              |

## Prerequisites

- Docker
- (Optional) NVIDIA GPU with [nvidia-container-toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) for GPU acceleration

## Quick Start

```bash
bash first_run.bash
```

The script will prompt you for:
1. **Project/drone name** — used as the container name
2. **Platform** — `PX4` or `Mavlink`
3. **Mode** — `Hardware` or `Simulation`
4. **Simulator** — `GZ-Harmonic` or `Isaac-Sim` (simulation mode only)
5. **GPU availability** — enables NVIDIA GPU passthrough if available

The script then builds the appropriate Docker image and launches the container with:
- X11 forwarding for GUI applications (Gazebo, RViz, etc.)
- SSH agent forwarding for GitHub access
- Privileged mode for hardware access
- A `goto_<name>` bash alias added to your `~/.bash_aliases`

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

| Alias / Function | Description |
|------------------|-------------|
| `build_aerostack2` | Build the Aerostack2 workspace (Release, symlink-install, 3 parallel workers) |
| `kill_ros2` | Kill all running ROS2 nodes |
| `waitForRos` | Wait until ROS2 topics are available |
| `killp <pid>` | Recursively kill a process tree |
| `refresh` | Re-source `~/.bashrc` |

## Tech Stack

- **ROS2 Humble** — robotics middleware
- **Aerostack2** — autonomous aerial systems framework
- **PX4 Autopilot v1.16** — flight controller firmware
- **Gazebo Harmonic** — physics simulation
- **Docker** — containerized environments
- **Micro-XRCE-DDS-Agent** — DDS bridge for PX4 offboard communication
- **MAVROS** — MAVLink to ROS bridge
