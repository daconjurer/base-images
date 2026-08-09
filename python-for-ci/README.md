# Python for CI

This Docker image is designed for continuous integration (CI) workflows with
Python. It is built from the official `python` slim image on Debian.

## Features

- **Python** runtime, version configurable via build arg `${PYTHON_VERSION}` (`python:${PYTHON_VERSION}-slim` base image, default **3.12**)
- [**uv**](https://docs.astral.sh/uv/) for dependency and environment management
  (`uv sync`, `uv run`, …), copied from the `ghcr.io/astral-sh/uv` image, pinned to **0.11.21** by default
- [**just**](https://github.com/casey/just) task runner for project commands
  (`just …`), pinned to **1.52.0** by default
- **Docker CLI** and **Docker Compose** for building/pushing container images
- Multi-architecture support (amd64/arm64)
- Virtual environment at `/app/.venv` enabled by default

Pinned tool versions and base image digest default to reproducible, verified installs.
Versions can be overridden at build time via Dockerfile `ARG` declarations.
Base image digest is pinned to prevent tag repointing; update it alongside `PYTHON_VERSION` when upgrading.

### Supply Chain Security

Binary downloads (just) are verified against their published SHA256 checksums before extraction.
Base images (python, uv, ci-docker-client) are pinned to their content digest to ensure immutability.

## Usage

Use this image as a base in CI pipelines or locally when you want Python, uv,
just, and Docker without installing them on the runner.

Typical workflow in a project with `pyproject.toml`, `uv.lock`, and a
`justfile`:

```bash
uv sync --frozen
just test
just lint
```

The default working directory is `/app` and the pre-created venv is on `PATH`,
so `python` and installed packages resolve from `/app/.venv` after `uv sync`.
