# ROS 2 for CI

This directory contains a Docker image designed for continuous integration (CI)
workflows with ROS 2 and Python.
It is built from the official `ros` image on Ubuntu.

## Features

- **ROS 2** Lyrical (`ros:lyrical-ros-base` base image), including `rclpy`
- **uv** 0.11.21 (`uv`, `uvx`)
- App venv at `/opt/venv` built on the system Python with system site-packages, so `rclpy` is importable; `UV_PROJECT_ENVIRONMENT` points at it, so `uv sync` installs into it
- `UV_PYTHON_DOWNLOADS=never`, so uv never swaps in a managed Python that cannot see ROS
- **git** for source access
- Multi-architecture support (amd64/arm64)

Nothing project-specific is baked in: install extra ROS packages with `apt-get` and Python dependencies with `uv sync` in the job.

### Supply Chain Security

The base image is pinned to its content digest to ensure immutability.
Update `ROS_IMAGE_DIGEST` alongside `ROS_DISTRO` when upgrading.
The uv binaries are copied from `ghcr.io/astral-sh/uv`, pinned by tag and digest.
Update `UV_IMAGE_DIGEST` alongside `UV_VERSION` when upgrading.

## Usage

GitHub Actions ignores the image entrypoint, so source ROS through the shell:

```yaml
container: ghcr.io/<owner>/ros2-for-ci:lyrical-latest
defaults:
  run:
    shell: /ros_entrypoint.sh bash -e {0}
```
