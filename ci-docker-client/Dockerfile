# syntax = docker/dockerfile:1.2
# Reusable Docker CLI + Compose client image for CI workloads.

ARG TARGETPLATFORM=linux/amd64
FROM --platform=$TARGETPLATFORM debian:bookworm-slim
ARG TARGETARCH
ARG DOCKER_CLI_VERSION=27.5.1
ARG COMPOSE_VERSION=2.32.4
RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates curl \
    && rm -rf /var/lib/apt/lists/* \
    && set -ex \
    && case "${TARGETARCH}" in \
        amd64) DOCKER_ARCH=x86_64; COMPOSE_ARCH=x86_64 ;; \
        arm64) DOCKER_ARCH=aarch64; COMPOSE_ARCH=aarch64 ;; \
        *) DOCKER_ARCH="${TARGETARCH}"; COMPOSE_ARCH="${TARGETARCH}" ;; \
       esac \
    && curl -fsSL "https://download.docker.com/linux/static/stable/${DOCKER_ARCH}/docker-${DOCKER_CLI_VERSION}.tgz" \
        | tar -xz -C /usr/local/bin --strip-components=1 docker/docker \
    && mkdir -p /usr/local/lib/docker/cli-plugins \
    && curl -fsSL \
        "https://github.com/docker/compose/releases/download/v${COMPOSE_VERSION}/docker-compose-linux-${COMPOSE_ARCH}" \
        -o /usr/local/lib/docker/cli-plugins/docker-compose \
    && chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
