# syntax = docker/dockerfile:1.2

# Build argument for Rust version
ARG RUST_VERSION=1.85

# Platform args for multi-arch builds
ARG TARGETARCH
ARG TARGETPLATFORM=linux/amd64

FROM --platform=$TARGETPLATFORM rust:${RUST_VERSION}-slim

# git is required for tests that shell out to `git` (e.g. git init steps).
RUN apt-get update \
    && apt-get install -y --no-install-recommends git \
    && rm -rf /var/lib/apt/lists/*

RUN rustup component add rustfmt clippy
