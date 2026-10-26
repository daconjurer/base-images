# Generic CI image for ROS 2 Python projects managed with Poetry.
# Consumers install their own ROS apt packages and Python deps.

# ROS distro (official image tag, e.g. lyrical → ros:lyrical-ros-base).
ARG ROS_DISTRO=lyrical
# Digest of the ros:${ROS_DISTRO}-ros-base manifest list, pinned so the tag can't be
# silently repointed. Update together with ROS_DISTRO (`docker buildx imagetools inspect` prints the new digest).
ARG ROS_IMAGE_DIGEST=sha256:d9078270abaeae65497036085f1654c75ec0a95e2aca5ecf7ddc0846a6cd69e1

FROM ros:${ROS_DISTRO}-ros-base@${ROS_IMAGE_DIGEST}

ARG POETRY_VERSION=2.1.1

# Package installs in CI jobs never have a terminal to prompt on.
ENV DEBIAN_FRONTEND=noninteractive

# git lets actions/checkout clone instead of downloading a tarball.
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        git \
        python3-venv \
    && rm -rf /var/lib/apt/lists/*

# Poetry in its own venv; the app venv sees system site-packages so rclpy and the
# ROS message packages stay importable. VIRTUAL_ENV makes `poetry sync` install into it.
RUN python3 -m venv /opt/poetry \
    && /opt/poetry/bin/pip install --no-cache-dir poetry==${POETRY_VERSION} \
    && python3 -m venv --system-site-packages /opt/venv
ENV VIRTUAL_ENV=/opt/venv \
    PATH=/opt/venv/bin:/opt/poetry/bin:$PATH

RUN poetry --version \
    && /ros_entrypoint.sh python -c "import rclpy"
