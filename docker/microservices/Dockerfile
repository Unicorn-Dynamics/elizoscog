# Multi-stage Dockerfile for ElizaOS-OpenCog-GnuCash Microservices
# Supports both cognitive services and GGML-optimized ML services

FROM python:3.12-slim as base

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    make \
    cmake \
    git \
    curl \
    wget \
    pkg-config \
    libssl-dev \
    libffi-dev \
    && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Copy requirements
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Production stage
FROM base as production

# Copy application code
COPY src/ ./src/
COPY *.py ./

# Create non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser
RUN chown -R appuser:appuser /app

USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Default command
CMD ["python", "-m", "src.microservices.service_discovery"]

# GGML-optimized stage for ML services
FROM base as ggml

# Install additional ML dependencies
RUN pip install --no-cache-dir \
    torch \
    transformers \
    accelerate \
    bitsandbytes

# Download and compile GGML (placeholder - would need actual GGML setup)
RUN mkdir -p /app/ggml

# Copy ML-specific configurations
COPY docker/configs/ggml/ ./configs/

# Copy application code
COPY src/ ./src/
COPY *.py ./

# Create non-root user
RUN groupadd -r mluser && useradd -r -g mluser mluser
RUN chown -R mluser:mluser /app

USER mluser

# Health check for ML services
HEALTHCHECK --interval=60s --timeout=30s --start-period=60s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# Command for GGML services
CMD ["python", "-m", "src.microservices.ggml_optimization"]

# Service discovery stage
FROM production as discovery

EXPOSE 8001

ENV SERVICE_TYPE=discovery
ENV SERVICE_PORT=8001

CMD ["python", "-c", "from src.microservices import ServiceRegistry; import asyncio; asyncio.run(ServiceRegistry().start())"]

# Load balancer stage  
FROM production as loadbalancer

EXPOSE 8002

ENV SERVICE_TYPE=loadbalancer
ENV SERVICE_PORT=8002

CMD ["python", "-c", "from src.microservices import LoadBalancer; print('Load balancer ready')"]

# Orchestration stage
FROM production as orchestrator

EXPOSE 8003

ENV SERVICE_TYPE=orchestrator
ENV SERVICE_PORT=8003

CMD ["python", "-c", "from src.microservices import ServiceOrchestrator; print('Orchestrator ready')"]