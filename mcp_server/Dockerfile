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

# Install dependencies first (better layer caching)
ARG INSTALL_FALKORDB=false
RUN if [ "$INSTALL_FALKORDB" = "true" ]; then \
        uv sync --frozen --no-dev --extra falkordb; \
    else \
        uv sync --frozen --no-dev; \
    fi

# Copy application code
COPY graphiti_mcp_server.py ./

# Expose port
EXPOSE $PORT

# Command to run the application
CMD ["uv", "run", "graphiti_mcp_server.py"]
```

**3. Dina Environment Variables i Railway**

Sätt dessa i Railway → Variables:
```
OPENAI_API_KEY=din-nyckel
NEO4J_URI=bolt://din-neo4j-url:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=ditt-lösenord
PORT=8000
```

Om du använder FalkorDB istället för Neo4j:
```
FALKORDB_HOST=din-falkordb-host
FALKORDB_PORT=6379
