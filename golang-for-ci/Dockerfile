# syntax = docker/dockerfile:1.2

# Build argument for Go version (official image tag suffix, e.g. 1.26.3 → golang:1.26.3-bookworm).
ARG GO_VERSION=1.26.3

# Platform args for multi-arch builds
ARG TARGETPLATFORM=linux/amd64
ARG DOCKER_CLIENT_IMAGE=dockerpaps/ci-docker-client:27.5.1-2.32.4-latest

FROM --platform=$TARGETPLATFORM ${DOCKER_CLIENT_IMAGE} AS dockercli

FROM --platform=$TARGETPLATFORM golang:${GO_VERSION}-bookworm

ARG TARGETARCH

# Release tag matching assets under https://github.com/go-task/task/releases
ARG TASK_VERSION=v3.49.1
# Semantic version matching release assets under https://github.com/golangci/golangci-lint/releases
ARG GOLANGCI_LINT_VERSION=2.12.2

ENV PATH=/go/bin:/usr/local/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates curl git tar \
    && rm -rf /var/lib/apt/lists/*

# Debian/Ubuntu login shells reset PATH via /etc/profile unless we prepend in profile.d,
# unlike `docker inspect` IMAGE PATH (upstream golang images behave this way too).
RUN printf '%s\n' \
        'PATH=/go/bin:/usr/local/go/bin:$${PATH:-/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin}' \
        'export PATH' \
    > /etc/profile.d/docker-golang-path.sh \
    && chmod 644 /etc/profile.d/docker-golang-path.sh

RUN set -ex \
    && curl -fsSL \
        "https://github.com/golangci/golangci-lint/releases/download/v${GOLANGCI_LINT_VERSION}/golangci-lint-${GOLANGCI_LINT_VERSION}-linux-${TARGETARCH}.tar.gz" \
        -o /tmp/gcl.tgz \
    && tar -xzf /tmp/gcl.tgz -C /usr/local/bin --strip-components=1 \
        "golangci-lint-${GOLANGCI_LINT_VERSION}-linux-${TARGETARCH}/golangci-lint" \
    && rm /tmp/gcl.tgz \
    && golangci-lint version

RUN set -ex \
    && curl -fsSL \
        "https://github.com/go-task/task/releases/download/${TASK_VERSION}/task_linux_${TARGETARCH}.tar.gz" \
        -o /tmp/task.tgz \
    && tar -xzf /tmp/task.tgz -C /go/bin task \
    && rm /tmp/task.tgz \
    && task --version

COPY --from=dockercli /usr/local/bin/docker /usr/local/bin/docker
COPY --from=dockercli /usr/local/lib/docker/cli-plugins/ /usr/local/lib/docker/cli-plugins/

RUN docker --version && docker compose version

CMD ["bash"]
