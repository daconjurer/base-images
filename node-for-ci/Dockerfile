# syntax = docker/dockerfile:1.2
# CI image with Docker CLI + Compose from ci-docker-client.
# Retrigger GitLab CI publish pipeline.

# Build argument for Node.js version (official image tag, e.g. 22 → node:22-slim).
ARG NODE_VERSION=22

# Tool versions
ARG PNPM_VERSION=10.32.1
ARG JUST_VERSION=1.52.0

# Platform args for multi-arch builds
ARG TARGETPLATFORM=linux/amd64
ARG DOCKER_CLIENT_IMAGE=dockerpaps/ci-docker-client:27.5.1-2.32.4-latest

FROM --platform=$TARGETPLATFORM ${DOCKER_CLIENT_IMAGE} AS dockercli

FROM --platform=$TARGETPLATFORM node:${NODE_VERSION}-slim AS node-base

# Builder stage — just prebuilt binary (multi-arch)
FROM node-base AS builder

ARG JUST_VERSION

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    tar \
    && rm -rf /var/lib/apt/lists/*

RUN set -ex \
    && case "$(dpkg --print-architecture)" in \
        amd64) just_arch=x86_64-unknown-linux-musl ;; \
        arm64) just_arch=aarch64-unknown-linux-musl ;; \
        *) echo "unsupported architecture: $(dpkg --print-architecture)" >&2; exit 1 ;; \
       esac \
    && curl -fsSL \
        "https://github.com/casey/just/releases/download/${JUST_VERSION}/just-${JUST_VERSION}-${just_arch}.tar.gz" \
        -o /tmp/just.tar.gz \
    && tar -xzf /tmp/just.tar.gz -C /usr/local/bin just \
    && rm /tmp/just.tar.gz \
    && just --version

# Final stage
FROM node-base AS final

ARG PNPM_VERSION

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    git \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder /usr/local/bin/just /usr/local/bin/just
COPY --from=dockercli /usr/local/bin/docker /usr/local/bin/docker
COPY --from=dockercli /usr/local/lib/docker/cli-plugins/ /usr/local/lib/docker/cli-plugins/

RUN corepack enable \
    && corepack prepare "pnpm@${PNPM_VERSION}" --activate

WORKDIR /app

RUN node --version && pnpm --version && just --version
RUN docker --version && docker compose version

CMD ["bash"]
