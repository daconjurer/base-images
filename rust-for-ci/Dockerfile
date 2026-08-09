# syntax = docker/dockerfile:1.2
# CI image with Docker CLI + Compose from ci-docker-client.

# Build argument for Rust version (official image tag suffix, e.g. 1.85 → rust:1.85-slim).
ARG RUST_VERSION=1.85
# Digest of the rust:${RUST_VERSION}-slim manifest list, pinned so the tag can't be
# silently repointed. Update together with RUST_VERSION (`docker pull` prints the new digest).
ARG RUST_IMAGE_DIGEST=sha256:9f841bbe9e7d8e37ceb96ed907265a3a0df7f44e3737d0b100e7907a679acb36

# Platform args for multi-arch builds
ARG TARGETARCH
ARG TARGETPLATFORM=linux/amd64
# Digest pinned alongside the tag; ci-docker-client is a third-party image so its tag alone
# isn't trustworthy as an immutable reference. Update together when bumping the tag.
ARG DOCKER_CLIENT_IMAGE=dockerpaps/ci-docker-client:27.5.1-2.32.4-latest@sha256:8285527d01b6db55cf3c0fac26d2cf0e8af1dd3198ae19c227bc69e236d6a810

FROM --platform=$TARGETPLATFORM ${DOCKER_CLIENT_IMAGE} AS dockercli

FROM --platform=$TARGETPLATFORM rust:${RUST_VERSION}-slim@${RUST_IMAGE_DIGEST}

# git is required for tests that shell out to `git` (e.g. git init steps).
RUN apt-get update \
    && apt-get install -y --no-install-recommends git \
    && rm -rf /var/lib/apt/lists/*

RUN rustup component add rustfmt clippy

COPY --from=dockercli /usr/local/bin/docker /usr/local/bin/docker
COPY --from=dockercli /usr/local/lib/docker/cli-plugins/ /usr/local/lib/docker/cli-plugins/

RUN rustup --version && cargo --version && rustfmt --version && cargo clippy --version \
    && docker --version && docker compose version
