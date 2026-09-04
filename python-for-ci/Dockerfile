# syntax = docker/dockerfile:1.2
# CI image with Docker CLI + Compose from ci-docker-client.

# Build argument for Python version (official image tag suffix, e.g. 3.12 → python:3.12-slim).
ARG PYTHON_VERSION=3.12
# Digest of the python:${PYTHON_VERSION}-slim manifest list, pinned so the tag can't be
# silently repointed. Update together with PYTHON_VERSION (`docker pull` prints the new digest).
ARG PYTHON_IMAGE_DIGEST=sha256:229a2c5bfa27522db7815ea81f9bed70af17ccb9de9fc7ad142b1877b5830d36

# Version of the ghcr.io/astral-sh/uv image to copy the uv/uvx binaries from.
ARG UV_VERSION=0.11.21
# Digest pinned alongside the tag since it's a third-party image. Update together with UV_VERSION.
ARG UV_IMAGE_DIGEST=sha256:ff07b86af50d4d9391d9daf4ff89ce427bc544f9aae87057e69a1cc0aa369946

# Release tag matching assets under https://github.com/casey/just/releases
ARG JUST_VERSION=1.52.0

# Platform args for multi-arch builds
ARG TARGETPLATFORM=linux/amd64
# Digest pinned alongside the tag; ci-docker-client is a third-party image so its tag alone
# isn't trustworthy as an immutable reference. Update together when bumping the tag.
ARG DOCKER_CLIENT_IMAGE=dockerpaps/ci-docker-client:27.5.1-2.32.4-latest@sha256:8285527d01b6db55cf3c0fac26d2cf0e8af1dd3198ae19c227bc69e236d6a810

FROM --platform=$TARGETPLATFORM ${DOCKER_CLIENT_IMAGE} AS dockercli

FROM --platform=$TARGETPLATFORM python:${PYTHON_VERSION}-slim@${PYTHON_IMAGE_DIGEST} AS python-base

FROM ghcr.io/astral-sh/uv:${UV_VERSION}@${UV_IMAGE_DIGEST} AS uv

# Builder stage — just prebuilt binary (multi-arch)
FROM python-base AS builder

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

# Final stage
FROM python-base AS final

RUN apt-get update && apt-get install -y --no-install-recommends \
    libatomic1 \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder /usr/local/bin/just /usr/local/bin/just
COPY --from=uv /uv /uvx /usr/local/bin/
COPY --from=dockercli /usr/local/bin/docker /usr/local/bin/docker
COPY --from=dockercli /usr/local/lib/docker/cli-plugins/ /usr/local/lib/docker/cli-plugins/

WORKDIR /app
RUN uv venv /app/.venv
ENV PATH="/app/.venv/bin:$PATH"

RUN just --version
RUN uv --version
RUN docker --version && docker compose version

CMD ["bash"]
