# Docker Base Images

A collection of Docker images for continuous integration (CI) pipelines.

## Images

| Image | Purpose |
|-------|---------|
| [`ci-docker-client`](ci-docker-client) | Docker CLI and Docker Compose for building and pushing images in CI |
| [`golang-for-ci`](golang-for-ci) | Go toolchain with Task, golangci-lint, and Docker CLI |
| [`node-for-ci`](node-for-ci) | Node.js with pnpm, just, and Docker CLI |
| [`python-for-ci`](python-for-ci) | Python with uv, just, and Docker CLI |
| [`ros2-for-ci`](ros2-for-ci) | ROS 2 with uv |
| [`rust-for-ci`](rust-for-ci) | Rust toolchain with just and Docker CLI; includes a separate frontend variant |

## Build and Publish

GitHub Actions workflows build and push images to Docker Hub and GitHub Container Registry on changes to `main`.

- `.github/workflows/docker-publish.yml` - builds and publishes images on merge to `main`
- `.github/workflows/docker-build-verify.yml` - builds images for verification on pull requests
- `.github/workflows/reusable-docker-build-push.yml` - reusable publish workflow
- `.github/workflows/reusable-docker-build-verify.yml` - reusable verification workflow

All images support `linux/amd64` and `linux/arm64` unless noted otherwise.

## Supply Chain Security

- Base images are pinned to their content digest to prevent silent tag repointing.
- Binary downloads are verified against published SHA256 checksums before extraction.
- Tool versions are parameterized as Dockerfile ARGs so they can be overridden at build time.
