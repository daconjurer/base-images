# Go for CI

This Docker image is designed for continuous integration (CI) workflows with
Go. It is built from the official `golang` image on Alpine Linux.

## Features

- **Go** 1.26.3 runtime and toolchain (`golang:1.26.3-alpine` base image)
- [**Task**](https://github.com/go-task/task) for running project commands, pinned to **v3.49.1** by default
- [**golangci-lint**](https://github.com/golangci/golangci-lint) for static analysis, pinned to **2.12.2** by default
- **Docker CLI** and **Docker Compose** for building/pushing container images
- **git** for source access
- Multi-architecture support (amd64/arm64)

Pinned tool versions and base image digest default to reproducible, verified installs.
Versions can be overridden at build time via Dockerfile `ARG` declarations.
Base image digest is pinned to prevent tag repointing; update it alongside `GO_VERSION` when upgrading.

### Supply Chain Security

Binary downloads (golangci-lint, task) are verified against their published SHA256 checksums before extraction.
Base images (golang, ci-docker-client) are pinned to their content digest to ensure immutability.

## Usage

Use this image as a base in CI pipelines or locally when you want Go, Task, golangci-lint, and Docker without installing them on the runner.

Alpine login shells do not reset `PATH` via `/etc/profile` the way Debian does, so `/go/bin` (Task) and `/usr/local/go/bin` (the `go` toolchain) are available directly.
This image adds `/etc/profile.d/docker-golang-path.sh` for explicit login-shell initialization when needed.
