# Docker Skill - Evaluation Set

This evaluation set tests the Docker skill across different use cases and complexity levels.

## Eval 1: Basic Node.js API Dockerization

**Prompt:**
```
I have a simple Node.js Express API. Here's my package.json:

{
  "name": "my-api",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  }
}

And server.js just runs on port 3000. Can you create a production-ready Dockerfile for this?
```

**Expected Behaviors:**
- Uses multi-stage build
- Uses Alpine base image
- Installs only production dependencies
- Runs as non-root user
- Implements proper layer caching (package.json before code)
- Includes dumb-init or similar for signal handling
- Final image should be small (<100MB)
- Explains key decisions

**Assertions:**
- [ ] Multi-stage build used
- [ ] Alpine base image used
- [ ] Non-root user configured
- [ ] `npm ci --only=production` or equivalent
- [ ] Layer caching optimized (COPY package*.json before code)
- [ ] Mentions expected image size
- [ ] Security considerations addressed

---

## Eval 2: Python Flask + PostgreSQL Docker Compose

**Prompt:**
```
I need to set up a development environment for my Flask API that connects to PostgreSQL. The Flask app is in app.py and uses these requirements:

Flask==3.0.0
psycopg2-binary==2.9.9
python-dotenv==1.0.0

Can you create a Docker Compose setup with both services? I want hot-reload for development.
```

**Expected Behaviors:**
- Creates docker-compose.yml with both services
- Flask service has volume mounts for hot-reload
- PostgreSQL service with health check
- Proper depends_on configuration
- Environment variables for database connection
- Network configuration
- Volume for PostgreSQL data persistence
- May also provide Dockerfile for Flask app
- Development-optimized configuration

**Assertions:**
- [ ] Docker Compose with 2 services created
- [ ] Volume mounts for Flask code (hot-reload)
- [ ] PostgreSQL health check configured
- [ ] depends_on with condition: service_healthy
- [ ] Database credentials configured
- [ ] Named volume for database persistence
- [ ] Network configuration present
- [ ] Explains the setup

---

## Eval 3: Optimize Large Existing Dockerfile

**Prompt:**
```
My Docker image is 1.2GB and takes forever to build. Here's my Dockerfile:

FROM node:20
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]

How can I make this better?
```

**Expected Behaviors:**
- Identifies multiple optimization opportunities
- Suggests multi-stage build
- Recommends Alpine base image
- Optimizes layer caching
- Separates build from runtime dependencies
- Provides optimized Dockerfile
- Estimates size reduction
- Explains each improvement

**Assertions:**
- [ ] Multi-stage build recommended
- [ ] Alpine variant suggested
- [ ] Layer caching optimization (package.json first)
- [ ] Separate build and runtime stages
- [ ] Build artifacts optimization mentioned
- [ ] Provides complete optimized Dockerfile
- [ ] Explains size/speed improvements
- [ ] Estimates reduction (should mention ~90% reduction possible)

---

## Eval 4: Debug Container Connection Issue

**Prompt:**
```
I'm trying to connect my Node.js app to PostgreSQL in Docker Compose, but I keep getting:

Error: connect ECONNREFUSED 127.0.0.1:5432

Here's my docker-compose.yml:

version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DB_HOST=127.0.0.1
      - DB_PORT=5432
  
  db:
    image: postgres:16
    environment:
      - POSTGRES_PASSWORD=password

What's wrong?
```

**Expected Behaviors:**
- Identifies the issue (using 127.0.0.1 instead of service name)
- Explains Docker networking concepts
- Provides corrected configuration
- Suggests using depends_on
- Recommends adding health check
- May suggest other improvements (networks, volumes)
- Provides debugging steps for future issues

**Assertions:**
- [ ] Identifies 127.0.0.1 issue (should use service name "db")
- [ ] Explains Docker Compose networking
- [ ] Provides corrected DB_HOST=db
- [ ] Suggests depends_on configuration
- [ ] Recommends health check for database
- [ ] Mentions other improvements (named volume, etc.)
- [ ] Provides testing/debugging steps

---

## Eval 5: Security Hardening Review

**Prompt:**
```
Can you review this Dockerfile for security issues and improve it?

FROM ubuntu:latest
WORKDIR /app
COPY . .
RUN apt-get update && apt-get install -y python3 python3-pip
RUN pip3 install -r requirements.txt
ENV SECRET_KEY=my-secret-key-123
EXPOSE 8000
CMD ["python3", "app.py"]
```

**Expected Behaviors:**
- Identifies multiple security issues
- Recommends specific base image (Python official or Alpine)
- Flags secret in ENV
- Suggests non-root user
- Recommends pinning versions
- Suggests creating .dockerignore
- Provides hardened Dockerfile
- Explains security best practices

**Assertions:**
- [ ] Identifies "latest" tag issue (should pin version)
- [ ] Identifies ENV SECRET_KEY as security risk
- [ ] Recommends non-root user
- [ ] Suggests proper base image (python:3.x-alpine or slim)
- [ ] Recommends .dockerignore file
- [ ] Suggests cleaning apt cache
- [ ] Mentions vulnerability scanning
- [ ] Provides secure Dockerfile rewrite
- [ ] Explains each security improvement

---

## Eval 6: Go Application with Ultra-Small Image

**Prompt:**
```
I have a Go REST API. I want the smallest possible Docker image. Here's my main.go - it's just a simple HTTP server. How should I Dockerize this?
```

**Expected Behaviors:**
- Recognizes Go's static compilation advantage
- Suggests multi-stage build with scratch base
- Includes ca-certificates for HTTPS
- Static binary compilation flags
- Non-root user configuration
- Explains why this works for Go
- Mentions expected image size (<10MB)

**Assertions:**
- [ ] Multi-stage build with scratch or alpine
- [ ] CGO_ENABLED=0 mentioned
- [ ] Static compilation flags explained
- [ ] CA certificates copied for HTTPS
- [ ] Non-root user (USER 1000:1000 or equivalent)
- [ ] Explains Go's advantage for minimal images
- [ ] Mentions expected tiny size (<10MB)
- [ ] Complete working Dockerfile provided

---

## Eval 7: Development vs Production Dockerfiles

**Prompt:**
```
I need both development and production Docker setups for my Python Django app. In development, I want hot-reload and debugging tools. In production, I want it optimized and secure. How should I structure this?
```

**Expected Behaviors:**
- Provides separate Dockerfiles or multi-stage approach
- Development: volume mounts, dev dependencies, debugger support
- Production: minimal image, security hardening, optimized
- May suggest Docker Compose for both environments
- Explains trade-offs and when to use each
- Provides practical examples

**Assertions:**
- [ ] Provides both development and production configurations
- [ ] Development includes volume mounts for hot-reload
- [ ] Production excludes dev dependencies
- [ ] Production uses smaller base image
- [ ] Production includes security hardening
- [ ] Explains differences and rationale
- [ ] Provides complete examples for both
- [ ] May include compose files for both environments

---

## Success Criteria Summary

A successful Docker skill should:

1. **Provide complete, working configurations** - Not just explanations but actual Dockerfiles/compose files
2. **Apply security by default** - Non-root users, minimal images, no secrets
3. **Optimize automatically** - Multi-stage builds, layer caching, small images
4. **Explain the why** - Help users understand Docker concepts
5. **Be language-aware** - Different patterns for Node.js, Python, Go, Java
6. **Handle troubleshooting systematically** - Clear diagnostic approaches
7. **Consider context** - Development vs production, single vs multi-service

## Metrics to Track

- **Image size reduction**: Should achieve 50-90% reduction in optimization cases
- **Security coverage**: All major security items addressed (non-root, secrets, scanning)
- **Completeness**: Provides full working examples, not fragments
- **Education**: Explains concepts, not just gives answers
- **Best practices**: Follows Docker official recommendations
