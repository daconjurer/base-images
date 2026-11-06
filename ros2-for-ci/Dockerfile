# Generic CI image for ROS 2 Python projects managed with uv.
# Consumers install their own ROS apt packages and Python deps.

# ROS distro (official image tag, e.g. lyrical → ros:lyrical-ros-base).
ARG ROS_DISTRO=lyrical
# Digest of the ros:${ROS_DISTRO}-ros-base manifest list, pinned so the tag can't be
# silently repointed. Update together with ROS_DISTRO (`docker buildx imagetools inspect` prints the new digest).
ARG ROS_IMAGE_DIGEST=sha256:d9078270abaeae65497036085f1654c75ec0a95e2aca5ecf7ddc0846a6cd69e1

# Version of the ghcr.io/astral-sh/uv image to copy the uv/uvx binaries from.
ARG UV_VERSION=0.11.21
# Digest pinned alongside the tag since it's a third-party image. Update together with UV_VERSION.
ARG UV_IMAGE_DIGEST=sha256:ff07b86af50d4d9391d9daf4ff89ce427bc544f9aae87057e69a1cc0aa369946

FROM ghcr.io/astral-sh/uv:${UV_VERSION}@${UV_IMAGE_DIGEST} AS uv

FROM ros:${ROS_DISTRO}-ros-base@${ROS_IMAGE_DIGEST}

# Package installs in CI jobs never have a terminal to prompt on.
ENV DEBIAN_FRONTEND=noninteractive

# git lets actions/checkout clone instead of downloading a tarball.
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        git \
    && rm -rf /var/lib/apt/lists/*

COPY --from=uv /uv /uvx /usr/local/bin/

# The app venv uses the system Python and sees system site-packages so rclpy and the
# ROS message packages stay importable. UV_PROJECT_ENVIRONMENT makes `uv sync` install
# into it; UV_PYTHON_DOWNLOADS=never stops uv from swapping in a managed Python, which
# would recreate the venv without ROS.
ENV UV_PROJECT_ENVIRONMENT=/opt/venv \
    UV_PYTHON_DOWNLOADS=never \
    VIRTUAL_ENV=/opt/venv \
    PATH=/opt/venv/bin:$PATH
RUN uv venv --python /usr/bin/python3 --system-site-packages /opt/venv

RUN uv --version \
    && /ros_entrypoint.sh python -c "import rclpy"
