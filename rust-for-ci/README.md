# Rust for CI

This directory contains Docker images designed for continuous integration (CI)
workflows with Rust. All images are built from the official `rust` image on
Debian Slim.

## Variants

### Backend (`Dockerfile`)

For CI workflows that build, test, and package Rust backend services or
libraries.

- **Rust** 1.85 toolchain (`rust:1.85-slim` base image), including `rustup`, `cargo`, and `rustc`
- **rustfmt** and **clippy** components for formatting and linting
- **Docker CLI** and **Docker Compose** for building/pushing container images
- **git** for source access
- Multi-architecture support (amd64/arm64)

### Frontend (`Dockerfile.frontend`)

For CI workflows that build Rust/WASM web apps with Trunk and a Node.js
frontend toolchain.

- Everything from the backend variant except Docker CLI/Compose
- **`wasm32-unknown-unknown`** target for WASM builds
- **Trunk** for building and bundling Rust/WASM projects
- **Node.js** 20 and **pnpm** 9

Base image digest is pinned to prevent tag repointing; update it alongside `RUST_VERSION` when upgrading.

### Supply Chain Security

Base images (rust, and ci-docker-client for the backend variant) are pinned to their content digest to ensure immutability.

## Usage

Use the backend image as a base in CI pipelines or locally when you want Rust,
rustfmt, clippy, and Docker without installing them on the runner.

Use the frontend image for Rust/WASM projects that also need Node.js and pnpm.
