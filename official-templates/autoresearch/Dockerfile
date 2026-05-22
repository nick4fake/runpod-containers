ARG BASE_IMAGE=non-existing
FROM ${BASE_IMAGE}

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Install runpodctl for pod management (scaling up GPUs)
ARG RUNPODCTL_VERSION=v2.3.0
RUN wget -qO- https://github.com/runpod/runpodctl/releases/download/"${RUNPODCTL_VERSION}"/runpodctl-linux-amd64.tar.gz | \
    tar -xz -C /usr/local/bin runpodctl

# Clone autoresearch to /opt (safe from volume mounts)
ARG AUTORESEARCH_REF=master
RUN git clone --branch "${AUTORESEARCH_REF}" --depth 1 \
    https://github.com/runpod/autoresearch.git /opt/autoresearch

WORKDIR /opt/autoresearch

# Install Python dependencies.
# --no-build refuses source distributions (which can execute arbitrary code
# via setup.py during install). If a dep lacks a wheel for linux/amd64, this
# build will fail loudly — that's intentional. Add per-package exceptions with
# --no-build-package=<name> only after auditing the source.
RUN uv sync --no-build && \
    uv run prepare.py # Download data and train tokenizer (~2 min)

# On first boot: copy source files to /workspace (lightweight, persists edits)
# and symlink .venv back to /opt (12GB, stays on fast container layer)
COPY pre_start.sh /pre_start.sh

# Override the NVIDIA CUDA entrypoint which blocks boot on hosts with older
# drivers (e.g. driver 550 only supports CUDA 12.4, but this image has 12.8).
ENTRYPOINT ["/start.sh"]
