# syntax = docker/dockerfile:1.2
# Reusable Docker CLI + Compose client image for CI workloads.

ARG TARGETPLATFORM=linux/amd64

# Debian release tag (matches tags under https://hub.docker.com/_/debian).
ARG DEBIAN_VERSION=bookworm-slim
# Digest of the debian:${DEBIAN_VERSION} manifest list, pinned so the tag can't be
# silently repointed. Update together with DEBIAN_VERSION (`docker pull` prints the new digest).
ARG DEBIAN_IMAGE_DIGEST=sha256:abd67ffcfa541b485a3dff59865ab629aa048a6c613e639d36e7456b0b229241

FROM --platform=$TARGETPLATFORM debian:${DEBIAN_VERSION}@${DEBIAN_IMAGE_DIGEST}

ARG TARGETARCH

# Docker CLI static binary release; matches assets under
# https://download.docker.com/linux/static/stable/<arch>/docker-<version>.tgz
ARG DOCKER_CLI_VERSION=27.5.1
# Docker Compose release; matches assets under
# https://github.com/docker/compose/releases/tag/v<version>
ARG COMPOSE_VERSION=2.32.4

RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates curl \
    && rm -rf /var/lib/apt/lists/*

# download.docker.com does not publish a checksums file alongside the static CLI
# tarballs, so the SHA256 sums below were computed directly from the official
# download.docker.com artifacts for DOCKER_CLI_VERSION and must be updated together
# whenever DOCKER_CLI_VERSION is bumped.
RUN set -ex \
    && cd /tmp \
    && case "${TARGETARCH}" in \
        amd64) DOCKER_ARCH=x86_64; DOCKER_SHA256=4f798b3ee1e0140eab5bf30b0edc4e84f4cdb53255a429dc3bbae9524845d640 ;; \
        arm64) DOCKER_ARCH=aarch64; DOCKER_SHA256=e6b53725a73763ab3f988c73f8772eaed429754c1a579db5ff11f21990fd1817 ;; \
        *) echo "Unsupported architecture: ${TARGETARCH}" >&2; exit 1 ;; \
       esac \
    && DOCKER_ASSET="docker-${DOCKER_CLI_VERSION}.tgz" \
    && curl -fsSL -o "${DOCKER_ASSET}" "https://download.docker.com/linux/static/stable/${DOCKER_ARCH}/${DOCKER_ASSET}" \
    && echo "${DOCKER_SHA256}  ${DOCKER_ASSET}" | sha256sum -c - \
    && tar -xzf "${DOCKER_ASSET}" -C /usr/local/bin --strip-components=1 docker/docker \
    && rm -f "${DOCKER_ASSET}"

RUN set -ex \
    && mkdir -p /usr/local/lib/docker/cli-plugins \
    && cd /tmp \
    && case "${TARGETARCH}" in \
        amd64) COMPOSE_ARCH=x86_64 ;; \
        arm64) COMPOSE_ARCH=aarch64 ;; \
        *) echo "Unsupported architecture: ${TARGETARCH}" >&2; exit 1 ;; \
       esac \
    && COMPOSE_ASSET="docker-compose-linux-${COMPOSE_ARCH}" \
    && COMPOSE_BASE_URL="https://github.com/docker/compose/releases/download/v${COMPOSE_VERSION}" \
    && curl -fsSL -o "${COMPOSE_ASSET}" "${COMPOSE_BASE_URL}/${COMPOSE_ASSET}" \
    && curl -fsSL -o "${COMPOSE_ASSET}.sha256" "${COMPOSE_BASE_URL}/${COMPOSE_ASSET}.sha256" \
    && sha256sum -c "${COMPOSE_ASSET}.sha256" \
    && install -m 0755 "${COMPOSE_ASSET}" /usr/local/lib/docker/cli-plugins/docker-compose \
    && rm -f "${COMPOSE_ASSET}" "${COMPOSE_ASSET}.sha256"

RUN docker --version && docker compose version
