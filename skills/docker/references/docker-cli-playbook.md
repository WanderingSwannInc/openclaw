---
name: docker-cli
description: Use this skill when working with Docker containers, images, Dockerfiles, or Docker Compose. Triggers include creating or optimizing Dockerfiles, setting up multi-container applications with Docker Compose, debugging container issues, implementing Docker security best practices, containerizing applications, optimizing image sizes or build times, or troubleshooting Docker networking/volumes. Provides systematic workflows for Docker tasks with language and framework-specific guidance.
---

# Docker CLI Skill

## Core Principles

1. **Security First** - Always implement security best practices (non-root users, minimal images, no secrets)
2. **Optimization by Default** - Minimize image size and build time through multi-stage builds and layer caching
3. **Production-Ready** - All configurations should work in real production environments
4. **Clear Explanations** - Help users understand the "why" behind Docker patterns

---

## Decision Tree: What Does the User Need?

### User wants to Dockerize an application
→ Go to **Application Dockerization Workflow**

### User has an existing Dockerfile to optimize
→ Go to **Dockerfile Optimization Workflow**

### User needs Docker Compose setup
→ Go to **Docker Compose Patterns**

### User is debugging a Docker issue
→ Go to **Troubleshooting Workflows**

### User asks about Docker security
→ Go to **Security Hardening Checklist**

---

## Application Dockerization Workflow

When a user wants to containerize their application:

1. **Identify the Stack**
   - What language/framework? (Node.js, Python, Go, Java, etc.)
   - What are the dependencies?
   - Are there any build steps?
   - Does it need any system packages?

2. **Determine the Environment**
   - Development only?
   - Production deployment?
   - Both? (provide separate configs or multi-stage approach)

3. **Select Base Image Strategy**
   - **Alpine**: Smallest, best for production (~5MB base)
   - **Debian Slim**: Good balance, more compatibility (~30MB base)
   - **Official language images**: Convenience but larger
   - **Distroless**: Maximum security, no shell (advanced)

4. **Implement Multi-Stage Build** (if applicable)
   - Separate build stage from runtime stage
   - Only copy necessary artifacts to final image
   - See Language-Specific Patterns below

5. **Apply Security Hardening**
   - Run as non-root user
   - Use .dockerignore
   - Don't include secrets
   - Pin dependency versions
   - Minimize layers and attack surface

6. **Optimize Layer Caching**
   - Copy dependency files first
   - Install dependencies
   - Then copy application code
   - This ensures dependencies are cached

7. **Create Docker Compose if Multi-Service**
   - See Docker Compose Patterns section

---

## Language-Specific Patterns

### Node.js Applications

**Best Practices:**
- Use official Node Alpine images
- Multi-stage build for development dependencies
- Leverage npm/yarn cache layers
- Use node user (UID 1000)

**Example Dockerfile:**
```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder
WORKDIR /app

# Copy package files and install dependencies
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Stage 2: Runtime
FROM node:20-alpine
WORKDIR /app

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

# Create non-root user (node user already exists in official image)
USER node

# Copy dependencies from builder
COPY --from=builder --chown=node:node /app/node_modules ./node_modules

# Copy application code
COPY --chown=node:node . .

EXPOSE 3000

# Use dumb-init to handle signals properly
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "server.js"]
```

**Development Dockerfile (dockerfile.dev):**
```dockerfile
FROM node:20-alpine
WORKDIR /app

RUN apk add --no-cache dumb-init

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

ENTRYPOINT ["dumb-init", "--"]
CMD ["npm", "run", "dev"]
```

---

### Python Applications

**Best Practices:**
- Use official Python Alpine or Slim images
- Multi-stage build for compiled dependencies
- Use virtual environment or pip install with --user
- Run as non-root user
- Use .dockerignore to exclude __pycache__, .pyc files

**Example Dockerfile (Flask/Django):**
```dockerfile
# Stage 1: Build
FROM python:3.12-slim AS builder
WORKDIR /app

# Install system dependencies if needed
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Runtime
FROM python:3.12-slim
WORKDIR /app

# Create non-root user
RUN useradd -m -u 1000 appuser

# Copy installed packages from builder
COPY --from=builder /root/.local /home/appuser/.local

# Copy application code
COPY --chown=appuser:appuser . .

# Make sure scripts in .local are usable
ENV PATH=/home/appuser/.local/bin:$PATH

USER appuser

EXPOSE 8000

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "app:app"]
```

**For Data Science/ML (heavier dependencies):**
```dockerfile
FROM python:3.12-slim
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    libgomp1 \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user
RUN useradd -m -u 1000 appuser

# Install Python packages
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY --chown=appuser:appuser . .

USER appuser

CMD ["python", "main.py"]
```

---

### Go Applications

**Best Practices:**
- Excellent for multi-stage builds (compiled binary)
- Final image can be minimal (scratch or alpine)
- Static compilation for maximum portability
- Very small final images (<10MB possible)

**Example Dockerfile:**
```dockerfile
# Stage 1: Build
FROM golang:1.22-alpine AS builder
WORKDIR /app

# Copy go mod files
COPY go.mod go.sum ./
RUN go mod download

# Copy source code
COPY . .

# Build static binary
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Stage 2: Runtime (minimal)
FROM alpine:latest
WORKDIR /app

# Install ca-certificates for HTTPS
RUN apk --no-cache add ca-certificates

# Create non-root user
RUN adduser -D -u 1000 appuser

# Copy binary from builder
COPY --from=builder --chown=appuser:appuser /app/main .

USER appuser

EXPOSE 8080

CMD ["./main"]
```

**Ultra-minimal version (using scratch):**
```dockerfile
# Stage 1: Build
FROM golang:1.22-alpine AS builder
WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .

# Build completely static binary
RUN CGO_ENABLED=0 GOOS=linux go build -a -ldflags '-extldflags "-static"' -o main .

# Stage 2: Minimal runtime
FROM scratch

# Copy CA certificates for HTTPS
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Copy binary
COPY --from=builder /app/main /main

EXPOSE 8080

USER 1000:1000

ENTRYPOINT ["/main"]
```

---

### Java / Spring Boot Applications

**Best Practices:**
- Use official Eclipse Temurin or Adoptium images
- Multi-stage build to separate build from runtime
- Layer JAR files for better caching
- Use JRE (not JDK) in final image

**Example Dockerfile (Maven):**
```dockerfile
# Stage 1: Build
FROM maven:3.9-eclipse-temurin-21 AS builder
WORKDIR /app

# Copy pom.xml and download dependencies (cached layer)
COPY pom.xml .
RUN mvn dependency:go-offline

# Copy source and build
COPY src ./src
RUN mvn package -DskipTests

# Stage 2: Runtime
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

# Create non-root user
RUN adduser -D -u 1000 appuser

# Copy JAR from builder
COPY --from=builder --chown=appuser:appuser /app/target/*.jar app.jar

USER appuser

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Spring Boot layered JAR approach (optimal caching):**
```dockerfile
# Stage 1: Build
FROM maven:3.9-eclipse-temurin-21 AS builder
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests
RUN java -Djarmode=layertools -jar target/*.jar extract

# Stage 2: Runtime
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

RUN adduser -D -u 1000 appuser

# Copy layers in order of least to most frequently changing
COPY --from=builder --chown=appuser:appuser /app/dependencies/ ./
COPY --from=builder --chown=appuser:appuser /app/spring-boot-loader/ ./
COPY --from=builder --chown=appuser:appuser /app/snapshot-dependencies/ ./
COPY --from=builder --chown=appuser:appuser /app/application/ ./

USER appuser

EXPOSE 8080

ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

---

## Dockerfile Optimization Workflow

When optimizing an existing Dockerfile:

1. **Analyze Current State**
   - What's the current image size?
   - How long does build take?
   - Is it using multi-stage builds?
   - What's the base image?

2. **Apply Multi-Stage Build**
   - Separate build dependencies from runtime dependencies
   - Only copy necessary artifacts to final stage

3. **Switch to Smaller Base Image**
   - Alpine variants are usually smallest
   - Ensure compatibility with dependencies
   - Test thoroughly after switching

4. **Optimize Layer Caching**
   - Move dependency installation before code copy
   - Order commands from least to most frequently changing

5. **Remove Unnecessary Files**
   - Use .dockerignore
   - Clean up package manager caches
   - Remove build artifacts not needed at runtime

6. **Combine Commands**
   - Fewer RUN commands = fewer layers
   - Chain commands with && and clean up in same layer

**Example Optimization:**

**Before (500MB+):**
```dockerfile
FROM node:20
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["node", "server.js"]
```

**After (50-100MB):**
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

FROM node:20-alpine
WORKDIR /app
RUN apk add --no-cache dumb-init
USER node
COPY --from=builder --chown=node:node /app/node_modules ./node_modules
COPY --chown=node:node . .
EXPOSE 3000
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "server.js"]
```

---

## Docker Compose Patterns

### Development Environment (Multi-Service)

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules  # Prevent overwriting node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:password@db:5432/myapp
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-network

  db:
    image: postgres:16-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - app-network

volumes:
  postgres-data:

networks:
  app-network:
    driver: bridge
```

### Production Configuration

```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:password@db:5432/myapp
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    networks:
      - app-network
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M

  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
      - POSTGRES_DB=myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data
    secrets:
      - db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

volumes:
  postgres-data:

networks:
  app-network:
    driver: bridge

secrets:
  db_password:
    file: ./secrets/db_password.txt
```

---

## Security Hardening Checklist

When creating or reviewing a Dockerfile, ensure:

- [ ] **Non-root user**: Create and use a non-root user (UID 1000 is standard)
- [ ] **Minimal base image**: Use Alpine, Slim, or Distroless variants
- [ ] **No secrets in image**: Never include passwords, keys, or tokens in Dockerfile or image
- [ ] **.dockerignore exists**: Exclude sensitive files, .git, tests, etc.
- [ ] **Pin versions**: Use specific versions for base images and dependencies
- [ ] **Scan for vulnerabilities**: Run `docker scan` or use Trivy/Snyk
- [ ] **Minimal layers**: Combine commands to reduce attack surface
- [ ] **Read-only filesystem**: Where possible, use `--read-only` flag
- [ ] **No unnecessary packages**: Only install what's needed
- [ ] **Clean package caches**: Remove apt/apk caches in same RUN layer

**Example .dockerignore:**
```
node_modules
npm-debug.log
.git
.gitignore
.env
.env.local
*.md
.vscode
.idea
coverage
.pytest_cache
__pycache__
*.pyc
.DS_Store
```

---

## Troubleshooting Workflows

### Build Failures

**Symptom**: Docker build fails with errors

**Diagnostic Steps:**
1. Check error message carefully - what's the last successful step?
2. Verify base image exists and is accessible
3. Check network connectivity if downloading dependencies
4. Look for syntax errors in Dockerfile
5. Verify file paths are correct (case-sensitive)
6. Check if required files exist (COPY commands)

**Common Issues:**
- **"COPY failed: no source files"**: File doesn't exist or .dockerignore is blocking it
- **"unable to select packages"**: Package name wrong or not available in base image
- **"npm ERR!"**: Check package.json syntax, network issues, or npm registry problems
- **"permission denied"**: Trying to access files as wrong user

---

### Container Won't Start

**Symptom**: Container exits immediately or fails to start

**Diagnostic Steps:**
1. Run `docker logs <container-id>` to see error messages
2. Check if port is already in use: `docker ps` or `netstat -an | grep <port>`
3. Verify command in CMD/ENTRYPOINT is correct
4. Try running container interactively: `docker run -it <image> /bin/sh`
5. Check health checks if configured

**Common Issues:**
- **"address already in use"**: Port conflict, change host port or stop other service
- **"exec: not found"**: Binary doesn't exist or wrong path
- **"permission denied"**: User doesn't have access to file/port
- **"no such file or directory"**: File missing or wrong working directory

---

### Networking Problems

**Symptom**: Containers can't communicate with each other

**Diagnostic Steps:**
1. Verify containers are on same network: `docker network inspect <network-name>`
2. Use service name as hostname (not container name or ID)
3. Check if ports are exposed in Dockerfile
4. Test connectivity: `docker exec <container> ping <other-service>`
5. Check firewall rules on host
6. Verify depends_on and healthchecks in Compose

**Common Issues:**
- **Can't reach other container**: Not on same network or using wrong hostname
- **Connection refused**: Service not listening or wrong port
- **Connection timeout**: Network misconfigured or firewall blocking
- **"name or service not known"**: DNS resolution failing, check network config

**Solution Pattern:**
```bash
# Check what networks exist
docker network ls

# Inspect a network
docker network inspect <network-name>

# Create custom network
docker network create my-network

# Connect container to network
docker network connect my-network <container-name>
```

---

### Volume/Permission Issues

**Symptom**: Can't read/write files in mounted volumes

**Diagnostic Steps:**
1. Check volume mount syntax: `-v host:container` or `-v named-volume:container`
2. Verify file ownership on host: `ls -la /path/to/volume`
3. Check UID/GID of user in container: `docker exec <container> id`
4. Look at permission errors in logs
5. Test with `chmod 777` temporarily to isolate permission issues (never use in production)

**Common Issues:**
- **"permission denied"** when accessing files: UID mismatch between host and container
- **Files owned by root**: Container running as root created files
- **Can't see files**: Wrong mount path or anonymous volume overriding bind mount

**Solution Patterns:**
```dockerfile
# Match UID/GID with host user
RUN useradd -u 1000 -m appuser
USER appuser
```

```yaml
# Docker Compose with user specification
services:
  app:
    user: "1000:1000"
    volumes:
      - ./data:/app/data
```

```bash
# Fix ownership on host
sudo chown -R 1000:1000 /path/to/volume
```

---

## Optimization Techniques

### Layer Caching Optimization

**Principle**: Docker caches each layer. If a layer hasn't changed, it reuses the cached version.

**Best Practice Order:**
1. Base image (FROM)
2. System packages (RUN apt-get install)
3. Dependency manifest (COPY package.json)
4. Install dependencies (RUN npm install)
5. Application code (COPY . .)
6. Build application (if needed)

**Why**: Dependencies change less frequently than code. By copying package.json first and installing, that layer is cached and reused unless package.json changes.

**Bad Example:**
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY . .              # Copies everything - cache busts on any code change
RUN npm install      # Has to reinstall every time code changes
CMD ["node", "app.js"]
```

**Good Example:**
```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./    # Only dependency manifest
RUN npm ci               # Cached unless package.json changes
COPY . .                 # Code changes don't affect above layers
CMD ["node", "app.js"]
```

---

### Image Size Reduction

**Techniques:**

1. **Use Alpine base images**
   - `node:20-alpine` instead of `node:20` (900MB → 130MB)
   - `python:3.12-alpine` instead of `python:3.12` (900MB → 50MB)

2. **Multi-stage builds**
   - Build in large image, copy artifacts to small runtime image
   - Can reduce from 1GB+ to <100MB

3. **Clean up in same layer**
   ```dockerfile
   RUN apt-get update && apt-get install -y package \
       && apt-get clean \
       && rm -rf /var/lib/apt/lists/*
   ```

4. **Use .dockerignore**
   - Prevent copying unnecessary files into build context

5. **Remove development dependencies**
   - Use `npm ci --only=production`
   - Use `pip install --no-cache-dir`

---

### Build Performance

**Techniques:**

1. **Parallel stages**: Use BuildKit and `--platform` for multi-platform builds
2. **Cache mounts**: Use `--mount=type=cache` for package manager caches
3. **Order matters**: Put frequently changing steps last
4. **Minimize context**: Use .dockerignore to exclude large unnecessary files
5. **Use layer caching**: Structure Dockerfile to maximize cache hits

**BuildKit Cache Mounts:**
```dockerfile
# syntax=docker/dockerfile:1
FROM node:20-alpine

RUN --mount=type=cache,target=/root/.npm \
    npm install -g npm@latest
```

---

## Common Patterns & Recipes

### Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

Or in Docker Compose:
```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
  interval: 30s
  timeout: 3s
  retries: 3
  start_period: 40s
```

### Signal Handling (Node.js)

Use `dumb-init` or `tini` to properly handle signals:
```dockerfile
RUN apk add --no-cache dumb-init
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "server.js"]
```

### Environment-Specific Configs

```dockerfile
ARG BUILD_ENV=production
ENV NODE_ENV=${BUILD_ENV}

# Different commands based on environment
CMD if [ "$NODE_ENV" = "development" ]; then \
        npm run dev; \
    else \
        npm start; \
    fi
```

### Secrets Management

Never in Dockerfile:
```dockerfile
# NEVER DO THIS
ENV API_KEY=secret123
```

Use environment variables at runtime:
```bash
docker run -e API_KEY=secret123 myapp
```

Or Docker secrets (Swarm/Compose):
```yaml
secrets:
  api_key:
    file: ./secrets/api_key.txt

services:
  app:
    secrets:
      - api_key
```

---

## Quick Reference: Common Commands

```bash
# Build image
docker build -t myapp:latest .

# Build with specific Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Run container
docker run -d -p 8080:8080 --name myapp myapp:latest

# Run with environment variables
docker run -e DB_HOST=localhost -e DB_PORT=5432 myapp

# Run with volume mount
docker run -v $(pwd)/data:/app/data myapp

# View logs
docker logs myapp
docker logs -f myapp  # Follow logs

# Execute command in container
docker exec -it myapp /bin/sh

# Stop and remove container
docker stop myapp && docker rm myapp

# Remove image
docker rmi myapp:latest

# Prune unused images/containers/volumes
docker system prune -a

# Docker Compose
docker compose up -d          # Start services
docker compose down           # Stop services
docker compose logs -f app    # View logs
docker compose exec app sh    # Execute command
docker compose ps             # List services
```

---

## When to Use This Skill

Trigger this skill whenever the user:
- Mentions creating a Dockerfile
- Wants to containerize an application
- Asks about Docker best practices
- Has a Docker-related error or problem
- Wants to optimize Docker images
- Needs a Docker Compose setup
- Asks about container security
- Wants to reduce image size
- Has networking or volume issues

This skill provides systematic, production-ready Docker guidance with security and optimization built in by default.
