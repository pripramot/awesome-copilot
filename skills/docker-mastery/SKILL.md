---
name: docker-mastery
description: 'Create optimized, production-ready Docker configurations including multi-stage builds, Docker Compose for local development, container security hardening, health checks, and deployment best practices for various application types.'
---

# Docker Mastery (Docker อย่างมืออาชีพ)

Build efficient, secure, and production-ready Docker configurations for any application type.

## Multi-Stage Dockerfile Patterns

### Node.js Application

```dockerfile
# Stage 1: Dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Stage 2: Builder
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 3: Production
FROM node:20-alpine AS runner
WORKDIR /app

# Security: run as non-root user
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs

COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./

USER nextjs
EXPOSE 3000
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -qO- http://localhost:3000/health || exit 1

CMD ["node", "dist/index.js"]
```

### Python FastAPI Application

```dockerfile
FROM python:3.12-slim AS base

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    && rm -rf /var/lib/apt/lists/*

FROM base AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

FROM base AS runner
WORKDIR /app

# Security: non-root user
RUN useradd -r -u 1001 -g root appuser

COPY --from=builder /root/.local /root/.local
COPY . .

USER appuser
ENV PATH=/root/.local/bin:$PATH

EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=5s \
  CMD curl -f http://localhost:8000/health || exit 1

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Docker Compose for Local Development

```yaml
# docker-compose.yml
version: '3.9'

services:
  app:
    build:
      context: .
      target: builder  # Use builder stage for hot-reload
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules  # Preserve container's node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./scripts/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydb"]
      interval: 10s
      timeout: 5s
      retries: 5
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass yourredispassword
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

volumes:
  postgres_data:
```

## Container Security Best Practices

```dockerfile
# 1. Use specific version tags, never 'latest'
FROM node:20.11.0-alpine3.19

# 2. Minimize attack surface
RUN apk add --no-cache curl && \
    apk upgrade --no-cache

# 3. Remove package manager cache
RUN npm ci --only=production && \
    npm cache clean --force

# 4. Run as non-root
RUN adduser -D -u 1001 appuser
USER appuser

# 5. Read-only filesystem where possible
# Run with: docker run --read-only --tmpfs /tmp myapp

# 6. No sensitive data in image layers
# Use secrets or environment variables instead
ARG BUILD_VERSION
ENV APP_VERSION=$BUILD_VERSION
# Never: ARG API_KEY (would be visible in image history)
```

## Useful Docker Commands

```bash
# Build with build args
docker build \
  --build-arg BUILD_VERSION=$(git rev-parse --short HEAD) \
  --tag myapp:latest \
  --file Dockerfile .

# Inspect image layers and sizes
docker image history myapp:latest

# Run with security options
docker run \
  --read-only \
  --tmpfs /tmp \
  --security-opt no-new-privileges:true \
  --cap-drop ALL \
  --user 1001:1001 \
  myapp:latest

# Check container resource usage
docker stats --no-stream

# Clean up unused resources
docker system prune --volumes --filter "until=24h"
```

## .dockerignore Template

```
# Dependencies
node_modules/
__pycache__/
*.pyc
.venv/

# Build artifacts
dist/
build/
*.egg-info/

# Development files
.env
.env.local
.env.*.local
*.log

# Version control
.git/
.gitignore

# IDE files
.vscode/
.idea/
*.swp

# Documentation
docs/
README.md
```

## Health Check Patterns

```python
# FastAPI health endpoint
from fastapi import FastAPI
from datetime import datetime

app = FastAPI()

@app.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "timestamp": datetime.utcnow().isoformat(),
        "version": os.getenv("APP_VERSION", "unknown")
    }
```

## Tips for Production

1. **Pin versions** - Always specify exact base image versions
2. **Layer caching** - Put rarely-changing layers first (e.g., OS packages, then deps, then code)
3. **Multi-stage** - Keep final image slim by discarding build tools
4. **Health checks** - Always define HEALTHCHECK for container orchestration
5. **Non-root** - Run containers as non-root users for security
6. **Secrets** - Use Docker secrets or environment variables, never bake secrets into images
