# syntax = docker/dockerfile:1.2
# CI image with Docker CLI + Compose from ci-docker-client.

# Build argument for Go version (official image tag suffix, e.g. 1.26.3 → golang:1.26.3-bookworm).
ARG GO_VERSION=1.26.3
# Digest of the golang:${GO_VERSION}-bookworm manifest list, pinned so the tag can't be
# silently repointed. Update together with GO_VERSION (`docker pull` prints the new digest).
ARG GO_IMAGE_DIGEST=sha256:386d475a660466863d9f8c766fec64d7fdad3edac2c6a05020c09534d71edb4b

# Platform args for multi-arch builds
ARG TARGETPLATFORM=linux/amd64
# Digest pinned alongside the tag; ci-docker-client is a third-party image so its tag alone
# isn't trustworthy as an immutable reference. Update together when bumping the tag.
ARG DOCKER_CLIENT_IMAGE=dockerpaps/ci-docker-client:27.5.1-2.32.4-latest@sha256:8285527d01b6db55cf3c0fac26d2cf0e8af1dd3198ae19c227bc69e236d6a810

FROM --platform=$TARGETPLATFORM ${DOCKER_CLIENT_IMAGE} AS dockercli

FROM --platform=$TARGETPLATFORM golang:${GO_VERSION}-bookworm@${GO_IMAGE_DIGEST} AS toolchain

ARG TARGETARCH

# Release tag matching assets under https://github.com/go-task/task/releases
ARG TASK_VERSION=v3.49.1
# Semantic version matching release assets under https://github.com/golangci/golangci-lint/releases
ARG GOLANGCI_LINT_VERSION=2.12.2

RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates curl tar \
    && rm -rf /var/lib/apt/lists/*

RUN set -ex \
    && cd /tmp \
    && GCL_ASSET="golangci-lint-${GOLANGCI_LINT_VERSION}-linux-${TARGETARCH}.tar.gz" \
    && GCL_BASE_URL="https://github.com/golangci/golangci-lint/releases/download/v${GOLANGCI_LINT_VERSION}" \
    && curl -fsSL -o "${GCL_ASSET}" "${GCL_BASE_URL}/${GCL_ASSET}" \
    && curl -fsSL -o checksums.txt "${GCL_BASE_URL}/golangci-lint-${GOLANGCI_LINT_VERSION}-checksums.txt" \
    && grep -E " ${GCL_ASSET}\$" checksums.txt | sha256sum -c - \
    && tar -xzf "${GCL_ASSET}" -C /usr/local/bin --strip-components=1 \
        "golangci-lint-${GOLANGCI_LINT_VERSION}-linux-${TARGETARCH}/golangci-lint" \
    && rm -f "${GCL_ASSET}" checksums.txt \
    && golangci-lint version

RUN set -ex \
    && cd /tmp \
    && TASK_ASSET="task_linux_${TARGETARCH}.tar.gz" \
    && TASK_BASE_URL="https://github.com/go-task/task/releases/download/${TASK_VERSION}" \
    && curl -fsSL -o "${TASK_ASSET}" "${TASK_BASE_URL}/${TASK_ASSET}" \
    && curl -fsSL -o checksums.txt "${TASK_BASE_URL}/task_checksums.txt" \
    && grep -E " ${TASK_ASSET}\$" checksums.txt | sha256sum -c - \
    && tar -xzf "${TASK_ASSET}" -C /usr/local/bin task \
    && rm -f "${TASK_ASSET}" checksums.txt \
    && task --version

FROM --platform=$TARGETPLATFORM golang:${GO_VERSION}-bookworm@${GO_IMAGE_DIGEST}

ENV PATH=/go/bin:/usr/local/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates git \
    && rm -rf /var/lib/apt/lists/*

# Debian/Ubuntu login shells reset PATH via /etc/profile unless we prepend in profile.d,
# unlike `docker inspect` IMAGE PATH (upstream golang images behave this way too).
RUN printf '%s\n' \
        'PATH=/go/bin:/usr/local/go/bin:$${PATH:-/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin}' \
        'export PATH' \
    > /etc/profile.d/docker-golang-path.sh \
    && chmod 644 /etc/profile.d/docker-golang-path.sh

COPY --from=toolchain /usr/local/bin/golangci-lint /usr/local/bin/golangci-lint
COPY --from=toolchain /usr/local/bin/task /usr/local/bin/task

COPY --from=dockercli /usr/local/bin/docker /usr/local/bin/docker
COPY --from=dockercli /usr/local/lib/docker/cli-plugins/ /usr/local/lib/docker/cli-plugins/

RUN golangci-lint version \
    && task --version \
    && docker --version && docker compose version

CMD ["bash"]
