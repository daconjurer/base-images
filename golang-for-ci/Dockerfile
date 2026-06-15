# syntax = docker/dockerfile:1.2

# Build argument for Go version (official image tag suffix, e.g. 1.26.3 → golang:1.26.3-bookworm).
ARG GO_VERSION=1.26.3

# Let BuildKit choose the runtime platform per invocation (`docker build/buildx --platform`).
FROM golang:${GO_VERSION}-bookworm

ARG TARGETARCH

# Module path is github.com/go-task/task/v3 (see https://github.com/go-task/task)
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

RUN go install github.com/go-task/task/v3/cmd/task@${TASK_VERSION} \
    && go clean -cache -modcache \
    && rm -rf /tmp/*

CMD ["bash"]
