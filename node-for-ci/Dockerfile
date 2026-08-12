# syntax = docker/dockerfile:1.2
# CI image with Docker CLI + Compose from ci-docker-client.
# Retrigger GitLab CI publish pipeline.

# Build argument for Node.js version (official image tag, e.g. 22 → node:22-slim).
ARG NODE_VERSION=22
# Digest of the node:${NODE_VERSION}-slim manifest list, pinned so the tag can't be
# silently repointed. Update together with NODE_VERSION (`docker pull` prints the new digest).
# Defaults to the digest for NODE_VERSION=22; callers building other supported versions
# (e.g. 20) must pass the matching NODE_IMAGE_DIGEST.
ARG NODE_IMAGE_DIGEST=sha256:d649c27dae7ba0137b3cef5dd75baa422c08dc3d9e3fc0c23dfb172dc3cc6436

# Release version matching assets under https://github.com/casey/just/releases
ARG JUST_VERSION=1.52.0
# npm package version matching https://www.npmjs.com/package/pnpm
ARG PNPM_VERSION=10.32.1

# Platform args for multi-arch builds
ARG TARGETPLATFORM=linux/amd64
# Digest pinned alongside the tag; ci-docker-client is a third-party image so its tag alone
# isn't trustworthy as an immutable reference. Update together when bumping the tag.
ARG DOCKER_CLIENT_IMAGE=dockerpaps/ci-docker-client:27.5.1-2.32.4-latest@sha256:8285527d01b6db55cf3c0fac26d2cf0e8af1dd3198ae19c227bc69e236d6a810

FROM --platform=$TARGETPLATFORM ${DOCKER_CLIENT_IMAGE} AS dockercli

FROM --platform=$TARGETPLATFORM node:${NODE_VERSION}-slim@${NODE_IMAGE_DIGEST} AS node-base

# Builder stage — just prebuilt binary (multi-arch)
FROM node-base AS builder

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
    && curl -fsSL -o SHA256SUMS "${JUST_BASE_URL}/SHA256SUMS" \
    && grep -E "  ${JUST_ASSET}\$" SHA256SUMS | sha256sum -c - \
    && tar -xzf "${JUST_ASSET}" -C /usr/local/bin just \
    && rm -f "${JUST_ASSET}" SHA256SUMS \
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
