# Docker CLI Skill - Analysis & Proposal

## Executive Summary

A Docker CLI skill would be highly valuable for helping users work with Docker containers, images, and infrastructure. This analysis examines the feasibility, scope, and design considerations for creating such a skill.

## Current State Analysis

### What Claude Can Already Do
Without a specialized skill, Claude can:
- Write Dockerfiles with best practices
- Explain Docker concepts and commands
- Debug Docker-related issues
- Suggest Docker compose configurations
- Provide general Docker advice

### What's Missing
Claude struggles with:
- **Systematically following Docker best practices** across all interactions
- **Context-aware optimization** based on specific use cases (development vs production, language-specific patterns)
- **Consistent multi-stage build patterns** that are properly optimized
- **Security hardening** with comprehensive, up-to-date practices
- **Troubleshooting workflows** that follow a structured diagnostic approach
- **Project-specific Dockerization** that considers the entire application stack

## Proposed Docker Skill Scope

### Primary Use Cases

1. **Creating Dockerfiles**
   - Language/framework-specific best practices
   - Multi-stage builds for optimization
   - Security hardening (non-root users, minimal base images)
   - Layer caching optimization
   - Build argument and environment variable patterns

2. **Docker Compose Workflows**
   - Multi-service application orchestration
   - Development vs production configurations
   - Volume and network management
   - Environment variable management
   - Health checks and dependency ordering

3. **Container Optimization**
   - Image size reduction strategies
   - Build time optimization
   - Runtime performance tuning
   - Layer caching strategies

4. **Debugging & Troubleshooting**
   - Systematic diagnostic workflows
   - Common error patterns and solutions
   - Networking issues
   - Volume/permission problems
   - Container state investigation

5. **Security Best Practices**
   - Vulnerability scanning guidance
   - Secrets management
   - Base image selection
   - Non-root user patterns
   - Network isolation

### Out of Scope (Initially)

- Kubernetes/orchestration (too broad, could be separate skill)
- Docker Swarm specifics
- Docker registry management
- Advanced networking (overlay networks, service mesh)
- CI/CD pipeline integration (better suited for CI/CD skill)

## Skill Structure Proposal

### SKILL.md Structure

```markdown
---
name: docker-cli
description: Use this skill when working with Docker containers, images, Dockerfiles, or Docker Compose. Triggers include: creating/optimizing Dockerfiles, setting up multi-container applications, debugging container issues, implementing Docker security best practices, optimizing image sizes or build times, or containerizing applications. Provides systematic workflows for common Docker tasks with language/framework-specific guidance.
---

# Docker CLI Skill

## Core Principles

1. **Security First** - Always implement security best practices
2. **Optimization by Default** - Minimize image size and build time
3. **Production-Ready** - Configurations work in real environments
4. **Clear Explanations** - Help users understand the "why"

## Workflow Decision Tree

### When User Wants to Dockerize an Application
1. Identify the application stack (language, framework, dependencies)
2. Check for existing Dockerfile or compose files
3. Determine environment (development, production, or both)
4. Select appropriate base image strategy
5. Implement multi-stage build if applicable
6. Add security hardening
7. Configure for the specific use case

### When User Has Docker Problems
1. Identify the failure point (build, run, networking, volumes)
2. Check logs and error messages systematically
3. Verify basic prerequisites
4. Isolate the issue
5. Provide targeted solutions

## Language-Specific Patterns

### Node.js Applications
[Detailed best practices for Node.js Docker images]

### Python Applications
[Detailed best practices for Python Docker images]

### Go Applications
[Detailed best practices for Go Docker images]

### Java/Spring Boot
[Detailed best practices for Java Docker images]

## Multi-Stage Build Patterns

[Comprehensive examples and explanations]

## Security Hardening Checklist

- [ ] Use minimal base images (alpine, distroless)
- [ ] Run as non-root user
- [ ] Scan for vulnerabilities
- [ ] Don't include secrets in images
- [ ] Use .dockerignore
- [ ] Pin dependency versions
- [ ] Minimize attack surface

## Common Troubleshooting Workflows

### Build Failures
[Systematic diagnostic approach]

### Runtime Issues
[Container won't start, crashes, etc.]

### Networking Problems
[Service connectivity issues]

### Volume/Permission Issues
[File access problems]

## Optimization Techniques

### Layer Caching
[How to structure Dockerfiles for optimal caching]

### Image Size Reduction
[Specific techniques with examples]

### Build Performance
[Strategies for faster builds]

## Docker Compose Patterns

### Development Environment
[Multi-service setup with hot reload]

### Production Deployment
[Production-ready compose configurations]

### Testing Environments
[Isolated test setups]

## Reference Material

### Common Base Images
- Alpine: Minimal, security-focused
- Debian Slim: Good balance
- Distroless: Maximum security
- Official language images: Convenience

### Useful Commands Reference
[Quick reference for common CLI operations]
```

## Implementation Considerations

### Strengths of This Approach

1. **Systematic**: Provides clear workflows rather than ad-hoc advice
2. **Context-Aware**: Language and use-case specific guidance
3. **Security-Focused**: Bakes in best practices from the start
4. **Optimization-First**: Helps users avoid common pitfalls
5. **Troubleshooting Support**: Structured diagnostic approaches

### Potential Challenges

1. **Rapidly Evolving Best Practices**: Docker best practices change frequently
2. **Version-Specific Issues**: Different Docker versions have different features
3. **Platform Differences**: Windows vs Linux vs macOS containers
4. **Scope Creep**: Easy to expand into Kubernetes, CI/CD, etc.

### Mitigation Strategies

- Focus on timeless principles over version-specific features
- Clearly document assumptions (e.g., "assumes Docker 20.10+")
- Keep initial scope focused on core Dockerfile/Compose workflows
- Link to official Docker documentation for version-specific details

## Evaluation Framework

### Test Cases for Skill Development

1. **Basic Node.js API Dockerization**
   - Expected: Multi-stage build, npm install optimization, non-root user
   - Verify: Image size < 200MB, builds under 2 minutes on first run

2. **Python Flask App with PostgreSQL**
   - Expected: Docker Compose with two services, proper networking, volume mounts
   - Verify: Services communicate, data persists across restarts

3. **Optimize Existing Large Image**
   - Given: 1GB+ image with issues
   - Expected: Reduced to <300MB with multi-stage build

4. **Debug Container Connection Issue**
   - Given: Service can't connect to database
   - Expected: Systematic troubleshooting leading to network/DNS solution

5. **Security Hardening Request**
   - Given: Basic Dockerfile
   - Expected: Non-root user, .dockerignore, minimal base image, no secrets

### Success Metrics

- **Image Size**: 50-80% reduction in typical cases
- **Security**: All hardening checklist items addressed
- **Build Time**: Optimal layer caching demonstrated
- **Troubleshooting**: Issues resolved through systematic workflow
- **Education**: Users understand the "why" behind recommendations

## Comparison: Docker CLI vs Docker Compose Skills

### Option A: Combined "Docker" Skill
**Pros:**
- Unified experience
- Natural overlap (Dockerfiles often used with Compose)
- Single skill to maintain

**Cons:**
- Could become large and unwieldy
- Mix of different concerns (single container vs orchestration)

### Option B: Separate Skills
**Pros:**
- More focused and maintainable
- Clear separation of concerns
- Users only load what they need

**Cons:**
- Potential duplication
- Users might not know which to use

**Recommendation: Start with Combined, Split Later if Needed**

The overlap is significant enough that a combined skill makes sense initially. Monitor usage patterns and split if the skill becomes too large or users consistently need only one aspect.

## Related Skills & Integration

### Skills That Would Benefit from Docker Skill

1. **Backend Development Skills**
   - Automated containerization of APIs
   - Local development environment setup

2. **Database Skills**
   - Container-based database setup
   - Local testing environments

3. **Deployment Skills** (if created)
   - Production-ready container configurations
   - Cloud deployment patterns

### Skills This Depends On

- None strictly required
- Could reference general security skills if they exist

## Recommended Next Steps

1. **Write Initial SKILL.md**
   - Start with Node.js and Python patterns (most common)
   - Include core troubleshooting workflows
   - Add security hardening checklist

2. **Create Initial Eval Set** (3-5 test cases)
   - Node.js API containerization
   - Multi-service Docker Compose setup
   - Troubleshooting scenario
   - Image optimization task

3. **Test & Iterate**
   - Run through eval cases
   - Identify gaps in guidance
   - Refine workflows based on results

4. **Expand Language Coverage**
   - Add Java, Go, Ruby, PHP patterns
   - Include framework-specific guidance (Django, Spring Boot, etc.)

5. **Add Advanced Topics** (Later)
   - Multi-platform builds
   - BuildKit features
   - Advanced caching strategies

## Alternative: More Specific Skills

Instead of a general Docker skill, consider more targeted skills:

1. **dockerfile-optimizer**: Just for creating/optimizing Dockerfiles
2. **docker-troubleshooter**: Just for debugging container issues
3. **container-security**: Security hardening focus

This could work if users have very specific needs, but likely fragments the experience too much.

## Conclusion

A Docker CLI skill is highly viable and would provide significant value by:
- Systematizing Docker best practices
- Providing context-aware guidance
- Offering structured troubleshooting workflows
- Ensuring security and optimization by default

The key is keeping initial scope focused on core Dockerfile and Docker Compose workflows, with clear patterns for common languages and frameworks. Start simple, measure effectiveness, and expand based on user needs.

The skill should be triggered whenever users mention containers, images, Dockerfiles, or Docker Compose, ensuring consistent best practices across all Docker-related interactions.
