# syntax=docker/dockerfile:1.7

# Use NVIDIA PyTorch base image with CUDA support
FROM nvcr.io/nvidia/pytorch:26.01-py3 as base

# CDC-specific certs
COPY .devcontainer/certs/*.crt /usr/local/share/ca-certificates/

# Install libGL
RUN apt-get update \
    && apt-get install -y --no-install-recommends ca-certificates libgl1-mesa-dev \
    && update-ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# === Common deps ==========================================================
FROM base AS deps

WORKDIR /workspaces/TRIDENT

# NOTE: Stub install to cache package dependencies for fast load
COPY pyproject.toml .

# Create a minimal stub so pip can resolve and install all declared
# dependencies without the real source tree being present.
# The stub is removed afterwards; `postCreateCommand` does the real -e install
# against the bind-mounted source, which is now near-instant.
RUN mkdir -p trident \
    && touch trident/__init__.py \
    && pip install --constraint /etc/pip/constraint.txt -e . \
    && rm -rf trident/

# === VSCode interactive dev =======================================
FROM deps AS dev

# Set Node.js to trust the additional CA certificates
# NOTE: Fixes vscode extensions
ENV NODE_EXTRA_CA_CERTS=/usr/local/share/ca-certificates/CDC-G2.crt

# === Dist image ===================================================
FROM deps AS runtime

COPY . .
RUN pip install --constraint /etc/pip/constraint.txt --no-deps .

ENTRYPOINT ["python", "run_single_slide.py"]
CMD ["--help"]
