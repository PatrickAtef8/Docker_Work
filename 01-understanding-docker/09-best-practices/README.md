# 09. Best Practices and Workflows

**Reading time:** 10 minutes

Professional patterns, workflows, and quick reference for daily Docker usage.

## Core Best Practices

### 1. Always Name Your Containers

**Bad:**
```bash
docker run -it myimage bash
# Creates: cranky_ptolemy (random name)
```

**Good:**
```bash
docker run -it --name myproject --hostname myproject myimage bash
# Creates: myproject (memorable name)
```

**Why:**
- Easy to reference later
- Clear purpose
- Easier to manage
- Better for team communication

### 2. Reuse Containers Instead of Creating New Ones

**Bad (creates many containers):**
```bash
docker run -it myimage bash   # Container 1
# work, exit
docker run -it myimage bash   # Container 2
# work, exit
docker run -it myimage bash   # Container 3
# ...containers accumulate
```

**Good (reuse one container):**
```bash
# Create once
docker run -it --name dev myimage bash

# Reuse every time
docker start -ai dev
docker start -ai dev
docker start -ai dev
```

**Benefits:**
- No container bloat
- Persistent changes
- Faster startup
- Cleaner system

### 3. Clean Up Regularly

**Check what's accumulated:**
```bash
docker ps -a
# Shows: 20+ old containers
```

**Clean up:**
```bash
# Remove specific containers
docker rm container1 container2

# Remove all stopped containers
docker container prune

# Remove unused images
docker image prune
```

### 4. Use `--rm` for Temporary Tasks

**For one-off commands:**
```bash
docker run --rm ubuntu echo "Hello"
docker run --rm myimage pytest
docker run --rm node:18 npm test
```

**Container automatically deleted after execution.**

### 5. Fix Versions in Dockerfiles

**Bad (unpredictable):**
```dockerfile
FROM ubuntu:latest
RUN apt install cmake qt6-base-dev
```

**Good (explicit):**
```dockerfile
FROM ubuntu:22.04
RUN apt install -y \
    cmake=3.22.* \
    qt6-base-dev=6.4.*
```

**Why:**
- Reproducible builds
- No surprises
- Clear dependencies
- Better debugging

## Volume Management for Persistent Data

### Problem: Container Data is Temporary

```bash
# Create container
docker run -it --name mycontainer ubuntu bash

# Inside: create files
echo "important data" > /data/file.txt

# Exit and remove
exit
docker rm mycontainer

# Data is GONE
```

### Solution: Mount Host Directory

```bash
docker run -it \
  --name myproject \
  -v $(pwd):/workspace \
  ubuntu bash
```

**Now:**
- Files in `/workspace` (container) = files in `$(pwd)` (host)
- Changes sync both ways
- Data persists even if container is deleted

### Practical Example

**Directory structure:**
```
~/myproject/
├── src/
│   ├── main.cpp
│   └── header.h
└── CMakeLists.txt
```

**Mount it:**
```bash
cd ~/myproject

docker run -it \
  --name devenv \
  -v $(pwd):/workspace \
  -w /workspace \
  myimage bash
```

**Inside container:**
```bash
ls /workspace
# src  CMakeLists.txt

# Build
cmake -B build
cmake --build build

# Exit
exit
```

**On host:**
```bash
ls ~/myproject
# src  CMakeLists.txt  build/
# Build artifacts persist!
```

## Docker + SDK for Embedded Development

### Professional Yocto Workflow

**Dockerfile:**
```dockerfile
FROM ubuntu:22.04

# Install Yocto dependencies
RUN apt update && apt install -y \
    build-essential \
    chrpath \
    cpio \
    diffstat \
    gawk \
    git \
    python3 \
    python3-pip \
    wget

# Copy SDK installer
COPY poky-sdk-installer.sh /tmp/

# Install SDK
RUN chmod +x /tmp/poky-sdk-installer.sh && \
    /tmp/poky-sdk-installer.sh -y -d /opt/poky

# Set up environment
ENV PATH="/opt/poky/sysroots/x86_64-pokysdk-linux/usr/bin:${PATH}"
ENV SDKTARGETSYSROOT="/opt/poky/sysroots/aarch64-poky-linux"

WORKDIR /workspace
```

**Build environment:**
```bash
docker build -t yocto-builder .
```

**Use it:**
```bash
docker run -it \
  --name yocto-dev \
  -v $(pwd):/workspace \
  yocto-builder bash

# Inside container
source /opt/poky/environment-setup-aarch64-poky-linux
make
```

**Benefits:**
- Same build environment for entire team
- No SDK pollution on host
- Easy CI/CD integration
- Reproducible builds

## Workflow Patterns

### Pattern 1: Development Environment

**Setup:**
```bash
# Create persistent dev environment
docker run -it \
  --name dev \
  --hostname dev \
  -v ~/projects:/projects \
  -w /projects \
  myimage bash
```

**Daily usage:**
```bash
# Morning
docker start -ai dev

# Work...

# Evening
exit

# Next day
docker start -ai dev
```

**Cleanup:**
```bash
docker rm dev
```

### Pattern 2: Quick Testing

**Test something quickly:**
```bash
docker run --rm -it ubuntu bash
# Try commands, experiment
exit
# Container auto-deleted
```

### Pattern 3: CI/CD Build

**Build script:**
```bash
#!/bin/bash

# Build Docker image
docker build -t myapp-builder .

# Run build inside container
docker run --rm \
  -v $(pwd):/workspace \
  -w /workspace \
  myapp-builder \
  bash -c "cmake -B build && cmake --build build"

# Build artifacts in ./build/
```

### Pattern 4: Isolated Tools

**Run tools without installing:**
```bash
# Node.js
docker run --rm -v $(pwd):/app -w /app node:18 npm install

# Python
docker run --rm -v $(pwd):/app -w /app python:3.11 pip install -r requirements.txt

# Rust
docker run --rm -v $(pwd):/app -w /app rust:latest cargo build
```

## Multi-Container Project Example

**Scenario:** Qt application + testing environment

```bash
# Development container
docker run -it \
  --name qt-dev \
  --hostname qt-dev \
  -v $(pwd):/workspace \
  -w /workspace \
  qt6-image bash

# Testing container (separate)
docker run -it \
  --name qt-test \
  --hostname qt-test \
  -v $(pwd):/workspace \
  -w /workspace \
  qt6-test-image bash
```

**Switch between them:**
```bash
# Develop
docker start -ai qt-dev

# Test
docker start -ai qt-test

# Both share same source via volume
```

## Debugging Tips

### Check Container Status

```bash
docker ps -a
# Shows all containers and their status
```

### View Last Logs

```bash
docker logs --tail 50 mycontainer
```

### Enter Running Container

```bash
docker exec -it mycontainer bash
```

### Copy Files Out

```bash
# From container to host
docker cp mycontainer:/app/output.log ./

# From host to container
docker cp ./file.txt mycontainer:/app/
```

### Inspect What Changed

```bash
docker diff mycontainer
```

**Output:**
```
A /app/newfile.txt
C /app/modified.txt
D /app/deleted.txt
```

## Common Mistakes to Avoid

### Mistake 1: Not Naming Containers

**Problem:** Dozens of random names accumulate
```bash
docker ps -a
# cranky_ptolemy, hungry_tesla, boring_morse...
```

**Solution:** Always use `--name`

### Mistake 2: Creating New Containers Instead of Reusing

**Problem:** Disk fills with old containers
```bash
docker ps -a | wc -l
# 47 containers!
```

**Solution:** Use `docker start` or `--rm`

### Mistake 3: Not Cleaning Up

**Problem:** System resources wasted
```bash
docker system df
# Images:     15GB
# Containers: 8GB
# Volumes:    12GB
```

**Solution:** Regular cleanup
```bash
docker system prune -a
```

### Mistake 4: Using `:latest` Tag

**Problem:** Builds change over time
```dockerfile
FROM ubuntu:latest  # Bad: what version?
```

**Solution:** Pin versions
```dockerfile
FROM ubuntu:22.04  # Good: explicit
```

### Mistake 5: Not Using Volumes

**Problem:** Losing work when container deleted
```bash
docker rm mycontainer
# All changes gone!
```

**Solution:** Mount volumes
```bash
docker run -v $(pwd):/workspace ...
```

## Quick Reference: Essential Commands

### Container Management

```bash
# Create and start
docker run -it --name NAME IMAGE

# List running
docker ps

# List all
docker ps -a

# Start stopped
docker start -ai NAME

# Stop running
docker stop NAME

# Remove
docker rm NAME

# Remove all stopped
docker container prune
```

### Image Management

```bash
# Build from Dockerfile
docker build -t NAME .

# List images
docker images

# Remove image
docker rmi NAME

# Pull from registry
docker pull ubuntu:22.04

# Remove unused images
docker image prune
```

### Working with Containers

```bash
# Run command in container
docker exec NAME COMMAND

# Enter container
docker exec -it NAME bash

# View logs
docker logs NAME

# Follow logs
docker logs -f NAME

# Copy files
docker cp NAME:/path ./local
docker cp ./local NAME:/path

# Inspect
docker inspect NAME

# Monitor resources
docker stats
```

### Volumes

```bash
# Mount directory
docker run -v /host/path:/container/path IMAGE

# Mount current directory
docker run -v $(pwd):/workspace IMAGE

# Set working directory
docker run -w /workspace IMAGE
```

### Cleanup

```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove everything unused
docker system prune -a
```

## Quick Start Templates

### Development Environment

```bash
docker run -it \
  --name dev \
  --hostname dev \
  -v $(pwd):/workspace \
  -w /workspace \
  ubuntu:22.04 bash
```

### Build Environment

```bash
docker run --rm \
  -v $(pwd):/build \
  -w /build \
  gcc:11 \
  bash -c "make clean && make"
```

### Testing Environment

```bash
docker run --rm \
  -v $(pwd):/test \
  -w /test \
  python:3.11 \
  pytest tests/
```

## Summary: Golden Rules

1. **Always name important containers**
   - Use `--name` and `--hostname`

2. **Reuse containers, don't create new ones**
   - Use `docker start -ai`

3. **Use volumes for persistent data**
   - Mount with `-v`

4. **Clean up regularly**
   - `docker container prune`

5. **Fix versions explicitly**
   - No `:latest` in production

6. **Use `--rm` for temporary tasks**
   - Auto-cleanup

7. **One purpose per container**
   - Dev environment, test environment, build environment

8. **Match name and hostname**
   - Consistency and clarity

9. **Keep Dockerfiles simple**
   - One concern at a time

10. **Document your workflow**
    - Team members thank you

## Final Thoughts

Docker is a simple tool that solves one problem: **environment consistency**.

It's not about:
- Changing your CPU architecture
- Replacing virtual machines
- Replacing SDKs
- Being a Raspberry Pi simulator

It's about:
- Clean environments
- Reproducible builds
- Team collaboration
- CI/CD pipelines

**Master these workflows and Docker becomes second nature.**

---

**Congratulations!** You now have a complete understanding of Docker from fundamentals to professional usage.

## Next Steps

- Practice the workflows with your own projects
- Experiment with different base images
- Create Dockerfiles for your build environments
- Share this knowledge with your team
- Apply Docker to your CI/CD pipelines

**Happy Dockerizing!**