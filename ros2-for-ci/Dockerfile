# Generic CI image for ROS 2 Python projects managed with uv and just.
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

# Release tag matching assets under https://github.com/casey/just/releases
ARG JUST_VERSION=1.52.0

FROM ghcr.io/astral-sh/uv:${UV_VERSION}@${UV_IMAGE_DIGEST} AS uv

FROM ros:${ROS_DISTRO}-ros-base@${ROS_IMAGE_DIGEST} AS ros-base

# Builder stage - just prebuilt binary (multi-arch)
FROM ros-base AS builder

ARG JUST_VERSION

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    tar \
    && rm -rf /var/lib/apt/lists/*

RUN set -ex \
    && cd /tmp \
    && case "$(dpkg --print-architecture)" in \
        amd64) just_arch=x86_64-unknown-linux-musl ;; \
        arm64) just_arch=aarch64-unknown-linux-musl ;; \
        *) echo "unsupported architecture: $(dpkg --print-architecture)" >&2; exit 1 ;; \
       esac \
    && JUST_ASSET="just-${JUST_VERSION}-${just_arch}.tar.gz" \
    && JUST_BASE_URL="https://github.com/casey/just/releases/download/${JUST_VERSION}" \
    && curl -fsSL -o "${JUST_ASSET}" "${JUST_BASE_URL}/${JUST_ASSET}" \
    && curl -fsSL -o checksums.txt "${JUST_BASE_URL}/SHA256SUMS" \
    && grep -E " ${JUST_ASSET}\$" checksums.txt | sha256sum -c - \
    && tar -xzf "${JUST_ASSET}" -C /usr/local/bin just \
    && rm -f "${JUST_ASSET}" checksums.txt \
    && just --version

FROM ros-base AS final

# Package installs in CI jobs never have a terminal to prompt on.
ENV DEBIAN_FRONTEND=noninteractive

# git lets actions/checkout clone instead of downloading a tarball.
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        git \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder /usr/local/bin/just /usr/local/bin/just
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

RUN just --version \
    && uv --version \
    && /ros_entrypoint.sh python -c "import rclpy"
