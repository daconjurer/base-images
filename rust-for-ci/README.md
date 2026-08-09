# Rust for CI

This Docker image is designed for continuous integration (CI) workflows with
Rust. It is built from the official `rust` image on Debian Slim.

## Features

- **Rust** 1.85 toolchain (`rust:1.85-slim` base image), including `rustup`, `cargo`, and `rustc`
- **rustfmt** and **clippy** components for formatting and linting
- **Docker CLI** and **Docker Compose** for building/pushing container images
- **git** for source access
- Multi-architecture support (amd64/arm64)

Base image digest is pinned to prevent tag repointing; update it alongside `RUST_VERSION` when upgrading.

### Supply Chain Security

Base images (rust, ci-docker-client) are pinned to their content digest to ensure immutability.

## Usage

Use this image as a base in CI pipelines or locally when you want Rust, rustfmt, clippy, and Docker without installing them on the runner.
