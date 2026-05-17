# Go for CI

This Docker image is designed for continuous integration (CI) workflows with
Go. It is built from the official `golang` image on Debian Bookworm.

## Features

- **Go** 1.25.0 runtime and toolchain (`golang:1.25.0-bookworm` base image)
- [**Task**](https://github.com/go-task/task) for running project commands (`task …`),
  pinned via `${TASK_VERSION}` (defaults to **v3.49.1** so builds succeed on Go 1.25.0;
  newer releases may bump the minimum Go toolchain)
- [**golangci-lint**](https://github.com/golangci/golangci-lint) for static analysis (`golangci-lint …`),
  pinned via `${GOLANGCI_LINT_VERSION}` (matching official release semver and tarball names)
- **git**
- Multi-architecture support (amd64/arm64)

Pinned tool versions default to reproducible installs and can be overridden at
build time (see Dockerfile `ARG` declarations for `${TASK_VERSION}` and
`${GOLANGCI_LINT_VERSION}`).

## Usage

Use this image as a base in CI pipelines or locally when you want Go, Task,
and golangci-lint without installing them on the runner.

Debian/Ubuntu login shells often reset `PATH` via `/etc/profile`. This image
adds `/etc/profile.d/docker-golang-path.sh` so `/go/bin` (Task) and
`/usr/local/go/bin` (the `go` toolchain) remain available for patterns like
`bash -lc` / `sh -lc` in CI scripts.
