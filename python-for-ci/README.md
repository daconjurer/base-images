# Python for CI

This Docker image is designed for continuous integration (CI) workflows with
Python. It provides a lightweight environment based on Python slim images with
essential tools pre-installed.

## Features

- **Python** with version configurable via build arg `${PYTHON_VERSION}`
- [**uv**](https://docs.astral.sh/uv/) for dependency and environment management
  (`uv sync`, `uv run`, …), pinned via `${UV_VERSION}` (default **0.11.21**)
- [**just**](https://github.com/casey/just) task runner for project commands
  (`just …`), pinned via `${JUST_VERSION}` (default **1.52.0**)
- Multi-architecture support (amd64/arm64)
- Virtual environment at `/app/.venv` enabled by default

Pinned tool versions default to reproducible installs and can be overridden at
build time (see Dockerfile `ARG` declarations for `${UV_VERSION}` and
`${JUST_VERSION}`).

## Usage

Use this image as a base in CI pipelines or locally when you want Python, uv,
and just without installing them on the runner.

Typical workflow in a project with `pyproject.toml`, `uv.lock`, and a
`justfile`:

```bash
uv sync --frozen
just test
just lint
```

The default working directory is `/app` and the pre-created venv is on `PATH`,
so `python` and installed packages resolve from `/app/.venv` after `uv sync`.
