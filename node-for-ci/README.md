# Node for CI

This Docker image is designed for continuous integration (CI) workflows with
Node.js. It is built from the official `node` image on Debian slim
(`node:${NODE_VERSION}-slim`), with Docker CLI and Compose layered in from
`ci-docker-client`.

## Features

- **Node.js** 22 runtime by default (`node:22-slim` base image), with 20 also supported via build arg `${NODE_VERSION}`
- [**pnpm**](https://pnpm.io/) for dependency management (`pnpm install`, `pnpm run …`), activated via Corepack and pinned to **10.32.1** by default
- [**just**](https://github.com/casey/just) task runner for project commands (`just …`), pinned to **1.52.0** by default
- **Docker CLI** and **Docker Compose** for building/pushing container images
- **git** for source access
- Multi-architecture support (amd64/arm64)

Pinned tool versions and base image digest default to reproducible, verified installs.
Versions can be overridden at build time via Dockerfile `ARG` declarations.
Base image digest is pinned to prevent tag repointing; when building a `NODE_VERSION`
other than the default, pass the matching `NODE_IMAGE_DIGEST` too (see the CI workflows
for the digests paired with each supported version).

### Supply Chain Security

Binary downloads (just) are verified against their published SHA256 checksums before extraction.
Base images (node, ci-docker-client) are pinned to their content digest to ensure immutability.

## Usage

Use this image as a base in CI pipelines or locally when you want Node.js, pnpm,
and just without installing them on the runner.

Typical workflow in a TypeScript project with `package.json`, `pnpm-lock.yaml`,
and a `justfile`:

```bash
pnpm install --frozen-lockfile
just test
just lint
just typecheck
```

The default working directory is `/app`.
