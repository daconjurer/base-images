# ROS 2 for CI

This directory contains a Docker image designed for continuous integration (CI)
workflows with ROS 2 and Python.
It is built from the official `ros` image on Ubuntu.

## Features

- **ROS 2** Lyrical (`ros:lyrical-ros-base` base image), including `rclpy`
- **Poetry** 2.1.1 in its own venv at `/opt/poetry`
- App venv at `/opt/venv` with system site-packages, so `rclpy` is importable; `VIRTUAL_ENV` points at it, so `poetry sync` installs into it
- **git** for source access
- Multi-architecture support (amd64/arm64)

Nothing project-specific is baked in: install extra ROS packages with `apt-get` and Python dependencies with `poetry install --no-root` in the job.

### Supply Chain Security

The base image is pinned to its content digest to ensure immutability.
Update `ROS_IMAGE_DIGEST` alongside `ROS_DISTRO` when upgrading.
Poetry is installed from PyPI at a pinned version; no other binaries are downloaded.

## Usage

GitHub Actions ignores the image entrypoint, so source ROS through the shell:

```yaml
container: ghcr.io/<owner>/ros2-for-ci:lyrical-latest
defaults:
  run:
    shell: /ros_entrypoint.sh bash -e {0}
```
