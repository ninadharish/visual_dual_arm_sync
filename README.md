# Visual Dual-Arm Synchronization

Dual-arm manipulation and visual synchronization in **NVIDIA Isaac Sim 6.0.1** with **ROS 2 Jazzy**.

- **Robot A:** UR10e; picks up an object and follows prescribed Cartesian paths.
- **Robot B:** Franka Emika Panda Robot; visually tracks the object and maintains a desired relative pose.
- Robot B's control loop must use camera feedback only; it must not consume Robot A's joint states, trajectory, or simulator ground-truth object pose.
- Robot A's baseline motion will later be compared with a time-optimized motion using task time, tracking error, and energy/control-effort metrics.

## Architecture

The project uses two Docker Compose services:

- `isaac`: Isaac Sim, physics, rendering, sensors, and the ROS 2 bridge.
- `ros`: ROS 2 Jazzy development environment and workspace.

Both containers start idle with `sleep infinity`. Isaac Sim is launched manually in either direct GUI or headless/WebRTC mode.

## Repository layout

```text
visual_dual_arm_sync/
├── compose.yaml
├── .env.example
├── .gitignore
├── docker/
│   └── ros/
│       └── Dockerfile
├── isaac-sim/
│   ├── cache/                 # generated; not committed
│   ├── config/                # generated; not committed
│   ├── data/                  # generated; not committed
│   ├── hub/                   # generated; not committed
│   ├── logs/                  # generated; not committed
│   ├── pkg/                   # generated; not committed
│   └── project/
│       ├── assets/            # custom/imported USD assets and textures
│       ├── scenes/            # saved Isaac Sim stages
│       └── scripts/           # Isaac Sim Python scripts
├── ros_ws/
│   └── src/                   # ROS 2 source packages
├── config/                    # DDS and project configuration
└── results/                   # selected plots, CSV summaries, and reports
```

## Prerequisites

- Ubuntu 24.04 with an X11 desktop session
- NVIDIA GPU and compatible host driver
- Docker Engine and Docker Compose
- NVIDIA Container Toolkit

Verify the GPU is available to Docker:

```bash
docker run --rm --gpus all \
  nvcr.io/nvidia/cuda:12.8.0-base-ubuntu24.04 \
  nvidia-smi
```

## Configure the environment

Create the local environment file:

```bash
cp .env.example .env
```

Set `HOST_UID`, `HOST_GID`, and `HOST_USER` to the output of:

```bash
id -u
id -g
id -un
```

`ISAACSIM_HOST` is only needed for WebRTC mode. Set it to the host's active LAN IP when using WebRTC.

## Build and start the containers

From the repository root:

```bash
docker compose pull isaac
docker compose build ros
docker compose up -d
```

Check service status:

```bash
docker compose ps
```

## Run Isaac Sim in direct GUI mode

Allow the container to access the local X11 display:

```bash
xhost +local:
```

Open a shell in the Isaac container:

```bash
docker compose exec isaac bash
```

Inside the container:

```bash
./runapp.sh -v \
  --/isaac/startup/ros_bridge_extension=isaacsim.ros2.bridge
```

The Compose configuration already sets the ROS 2 Jazzy library path required by the Isaac Sim ROS bridge.

## Optional: run headless with WebRTC

Ensure `ISAACSIM_HOST` in `.env` matches an active host network interface, then open an Isaac shell:

Inside the container:

```bash
./runheadless.sh -v \
  --/isaac/startup/ros_bridge_extension=isaacsim.ros2.bridge
```

After the streaming application is ready, launch the WebRTC client on the host and connect to `ISAACSIM_HOST`.

## ROS 2 workspace

Open a ROS development shell:

```bash
docker compose exec ros bash
```

Build the workspace:

```bash
source /opt/ros/jazzy/setup.bash
cd /workspace/project/ros_ws
rosdep install --from-paths src --ignore-src --rosdistro jazzy -r -y
colcon build --symlink-install
source install/setup.bash
```

## Saving Isaac Sim work

Save project-owned stages inside the mounted project directory:

```text
/workspace/project/isaac/scenes/
```

This maps to:

```text
isaac-sim/project/scenes/
```

Recommended organization:

```text
isaac-sim/project/
├── scenes/
│   ├── robot_a_test.usda
│   └── dual_arm_scene.usda
├── assets/
│   ├── robots/
│   ├── objects/
│   ├── materials/
│   └── textures/
└── scripts/
```

Keep lightweight root scenes separate from reusable robot, object, material, and texture assets. Use relative USD references where possible.

## Stop the environment

Stop the existing containers without removing them:

```bash
docker compose stop
```

Restart the same containers later:

```bash
docker compose start
```

To remove the containers while preserving bind-mounted project files:

```bash
docker compose down
```