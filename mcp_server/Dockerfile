# syntax=docker/dockerfile:1.9
FROM python:3.12-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Install uv using the installer script
ADD https://astral.sh/uv/install.sh /uv-installer.sh
RUN sh /uv-installer.sh && rm /uv-installer.sh

# Add uv to PATH
ENV PATH="/root/.local/bin:${PATH}"

# Configure uv for optimal Docker usage
ENV UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy \
    UV_PYTHON_DOWNLOADS=never \
    MCP_SERVER_HOST="0.0.0.0" \
    PYTHONUNBUFFERED=1

# Copy project files for dependency installation (better caching)
COPY pyproject.toml uv.lock ./

# Install dependencies with providers extra for Neo4j and other LLM providers
RUN uv sync --frozen --no-dev --extra providers

# Copy application code
COPY graphiti_mcp_server.py ./

# Expose port
EXPOSE 8000

# Healthcheck for Railway
HEALTHCHECK --interval=30s --timeout=10s --start-period=40s --retries=3 \
    CMD curl -f http://localhost:${PORT:-8000}/health || exit 1

# Command to run with Neo4j database provider
CMD ["sh", "-c", "uv run graphiti_mcp_server.py --database-provider neo4j --transport http --port ${PORT:-8000}"]
