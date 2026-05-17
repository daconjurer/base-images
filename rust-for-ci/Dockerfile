# syntax = docker/dockerfile:1.2

# Platform args for multi-arch builds
ARG TARGETARCH
ARG TARGETPLATFORM=linux/amd64

FROM --platform=$TARGETPLATFORM rust:1.85-slim

# git is required for tests that shell out to `git` (e.g. git init steps).
RUN apt-get update \
    && apt-get install -y --no-install-recommends git \
    && rm -rf /var/lib/apt/lists/*

RUN rustup component add rustfmt clippy
