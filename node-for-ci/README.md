# Node for CI

This Docker image is designed for continuous integration (CI) workflows with
Node.js. It provides a lightweight environment based on Node slim images with
essential tools pre-installed.

## Features

- **Node.js** with version configurable via build arg `${NODE_VERSION}`
- [**pnpm**](https://pnpm.io/) for dependency management (`pnpm install`, `pnpm run …`),
  pinned via `${PNPM_VERSION}` (default **10.32.1**)
- [**just**](https://github.com/casey/just) task runner for project commands
  (`just …`), pinned via `${JUST_VERSION}` (default **1.52.0**)
- **git**
- Multi-architecture support (amd64/arm64)

Pinned tool versions default to reproducible installs and can be overridden at
build time (see Dockerfile `ARG` declarations for `${PNPM_VERSION}` and
`${JUST_VERSION}`).

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
