---
name: docker
description: Create, fix, and optimize Dockerfiles and Docker Compose setups. Use when users ask to containerize apps, debug container startup/network/volume issues, harden Docker security, or reduce image size/build time.
---

# Docker

Use this skill for production-ready Docker work.

## Workflow
1. Identify app stack/runtime and deployment target (dev, prod, or both).
2. Build/adjust Dockerfile with:
   - minimal base image
   - layer-cache-friendly ordering
   - non-root runtime user
   - multi-stage build when useful
3. Add/adjust Docker Compose for multi-service apps.
4. Validate with run logs, health checks, and connectivity tests.
5. Apply hardening and optimization before final handoff.

## Non-negotiables
- No secrets in images or committed compose files.
- Use non-root user in runtime containers.
- Prefer smaller secure base images.

## References
- Full playbook, patterns, and troubleshooting: `references/docker-cli-playbook.md`
