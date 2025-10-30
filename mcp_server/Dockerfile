# syntax=docker/dockerfile:1.9
FROM python:3.12-slim
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user EARLY
RUN groupadd -r app && useradd -r -d /app -g app app

# Install uv using the installer script as app user
USER app
ADD --chown=app:app https://astral.sh/uv/install.sh /tmp/uv-installer.sh
RUN sh /tmp/uv-installer.sh && rm /tmp/uv-installer.sh

# Add uv to PATH for app user
ENV PATH="/app/.local/bin:${PATH}"

# Configure uv for optimal Docker usage
ENV UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy \
    UV_PYTHON_DOWNLOADS=never \
    MCP_SERVER_HOST="0.0.0.0" \
    PYTHONUNBUFFERED=1

# Copy project files for dependency installation (better caching)
COPY --chown=app:app pyproject.toml uv.lock ./

# Install dependencies first (better layer caching)
ARG INSTALL_FALKORDB=false
RUN if [ "$INSTALL_FALKORDB" = "true" ]; then \
        uv sync --frozen --no-dev --extra falkordb; \
    else \
        uv sync --frozen --no-dev; \
    fi

# Copy application code
COPY --chown=app:app graphiti_mcp_server.py ./

# Expose port
EXPOSE $PORT

# Command to run the application
CMD ["uv", "run", "graphiti_mcp_server.py"]
