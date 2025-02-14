# Python for CI

This Docker image is designed for continuous integration (CI) workflows with
Python. It provides a lightweight environment based on Python slim images with
essential tools pre-installed.

## Features

- Python ${PYTHON_VERSION} (configurable via build arg)
- Poetry 2.0.0 for dependency management
- Multi-architecture support (amd64/arm64)
- Virtual environment enabled by default

## Usage

The main idea is to use the image as a base for CI pipelines but it can be
useful for development environments too.
