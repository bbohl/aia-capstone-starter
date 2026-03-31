FROM mcr.microsoft.com/devcontainers/base:bookworm

# Install prerequisites
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        zstd curl ca-certificates python3 python3-venv python3-pip && \
    rm -rf /var/lib/apt/lists/*

# Pre-build the Python virtual environment
# Install CPU-only PyTorch FIRST to avoid pulling in GPU/triton packages
COPY requirements/requirements.txt /tmp/requirements.txt
RUN python3 -m venv /opt/py_env && \
    /opt/py_env/bin/pip install --no-cache-dir \
        torch torchvision torchaudio \
        --index-url https://download.pytorch.org/whl/cpu && \
    grep -ivE '^torch(vision|audio)?>=' /tmp/requirements.txt > /tmp/requirements_filtered.txt && \
    /opt/py_env/bin/pip install --no-cache-dir -r /tmp/requirements_filtered.txt && \
    /opt/py_env/bin/pip freeze | grep -iE '^(nvidia|cuda|triton)' \
        | cut -d= -f1 | xargs -r /opt/py_env/bin/pip uninstall -y 2>/dev/null || true && \
    find /opt/py_env -type d -name "tests" -exec rm -rf {} + 2>/dev/null || true && \
    find /opt/py_env -name "*.pyc" -delete 2>/dev/null || true
