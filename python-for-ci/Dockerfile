# syntax = docker/dockerfile:1.2

# Build argument for Python version
ARG PYTHON_VERSION=3.12

# Platform args for multi-arch builds
ARG TARGETARCH
ARG TARGETPLATFORM=linux/amd64

FROM --platform=$TARGETPLATFORM python:${PYTHON_VERSION}-slim AS python-base

# Builder stage
FROM python-base AS builder

RUN apt-get update && apt-get install -y --no-install-recommends \
    make \
    curl \
    # Poetry
    && curl -sSL https://install.python-poetry.org | POETRY_HOME=/opt/poetry POETRY_VERSION=2.0.0 python3 - \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get remove -y curl \
    && apt-get autoremove -y

# Final stage
FROM python-base AS final

# Copy the installations
COPY --from=builder /opt/poetry /opt/poetry
COPY --from=builder /usr/bin/make /usr/bin/make

# Set working directory with Poetry and a venv enabled
WORKDIR /app
RUN python -m venv /app/.venv
ENV PATH="/opt/poetry/bin:/app/.venv/bin:$PATH"

# Default command
CMD ["bash"]
