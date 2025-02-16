# Python for CI

This Docker image is designed for continuous integration (CI) workflows with
Python. It provides a lightweight environment based on Python slim images with
essential tools pre-installed.

## Features

- **Python** with version configurable via build arg `${PYTHON_VERSION}`
- **Poetry** 2.0.0 for dependency management
- **Make** for running the different scripts in a convenient way
- Multi-architecture support (amd64/arm64)
- Virtual environment enabled by default

## Usage

The main idea is to use the image as a base for CI pipelines but it can be
useful for development environments too.
