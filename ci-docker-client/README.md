# CI Docker Client

This Docker image provides the Docker CLI and Docker Compose for CI
workflows that need to build or push container images. It is built from
the official `debian` image (`bookworm-slim` variant).

Other images in this repository (golang-for-ci, python-for-ci, rust-for-ci,
node-for-ci) copy the CLI binaries out of this image at build time, so it
has no upstream CI base image of its own to pin; its own supply chain
consists of the Debian base image and the Docker CLI and Compose binary
downloads.

## Features

- **Docker CLI**, pinned to **27.5.1** by default (`debian:bookworm-slim` base image)
- **Docker Compose**, pinned to **2.32.4** by default
- Multi-architecture support (amd64/arm64)

Pinned tool versions and base image digest default to reproducible, verified installs.
Versions can be overridden at build time via Dockerfile `ARG` declarations.
Base image digest is pinned to prevent tag repointing; update it alongside `DEBIAN_VERSION` when upgrading.

### Supply Chain Security

The Docker Compose binary download is verified against its published SHA256 checksum
file before installation. The Docker CLI static binary tarball is verified against a
SHA256 checksum pinned directly in the Dockerfile, computed from the official
`download.docker.com` release artifacts, since Docker does not publish a checksums
file alongside its static CLI binaries.
The base image (`debian:bookworm-slim`) is pinned to its content digest to ensure
immutability.

## Usage

Use this image as a build stage to copy the Docker CLI and Compose plugin into other
CI images, or run it directly when a pipeline step only needs the Docker client tools.

```dockerfile
FROM dockerpaps/ci-docker-client:27.5.1-2.32.4-latest AS dockercli
...
COPY --from=dockercli /usr/local/bin/docker /usr/local/bin/docker
COPY --from=dockercli /usr/local/lib/docker/cli-plugins/ /usr/local/lib/docker/cli-plugins/
```

This image does not include a Docker daemon; it is intended to be paired with a
daemon reachable via `DOCKER_HOST` (e.g. Docker-in-Docker or a mounted socket) in CI.
