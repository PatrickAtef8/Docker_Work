# 02. What Docker Actually Is

## One-Sentence Definition

**Docker is a clean, throw-away Linux environment with fixed tools.**

That's it. Nothing more complicated.

## What Docker Is NOT

Let's clear up common misconceptions:

### NOT a Virtual Machine
- VMs run a complete OS with its own kernel
- Docker containers share the host kernel
- Containers are much lighter and faster

### NOT Another Computer
- It doesn't create new hardware
- It doesn't simulate a CPU
- It uses your existing CPU and kernel

### NOT a Raspberry Pi Simulator
- Running Docker on x86 doesn't make it ARM
- Docker doesn't change your CPU architecture
- You're still on your host CPU

### NOT a Replacement for SDKs
- SDKs cross-compile for different architectures
- Docker isolates the build environment
- You still need SDKs for cross-compilation

## What Docker IS

### A Boxed Linux Environment

Think of Docker as creating a "box" containing:
- A specific Linux filesystem (Ubuntu, Debian, etc.)
- Specific versions of tools (gcc, cmake, Qt)
- Your application code
- Everything needed to build/run

### With Fixed Versions

When you write:
```dockerfile
FROM ubuntu:22.04
RUN apt install gcc=11.* cmake=3.22.*
```

You're saying:
- "Use exactly Ubuntu 22.04"
- "Use exactly gcc 11.x"
- "Use exactly cmake 3.22.x"

Not "latest" or "whatever is installed" - **exact versions**.

### That Runs on Your Host Kernel

The container:
- Uses your Linux kernel
- Uses your CPU
- Uses your hardware

But has:
- Its own filesystem
- Its own processes
- Its own users
- Its own network

### Isolated from Your System

Changes inside container don't affect your host:
- Install packages inside: host stays clean
- Delete container: everything disappears
- No pollution, no conflicts

## The Mental Model

Think of Docker like this:

```
Docker = ZIP file + script + isolation

Where:
- ZIP file = Linux filesystem snapshot
- Script = commands to run (build, install, etc.)
- Isolation = container can't mess with host
```

## Comparison: Without vs With Docker

### Without Docker

```bash
# On your system
sudo apt install gcc cmake qt6
```

**Results in:**
- System polluted with packages
- Potential version conflicts
- Hard to remove cleanly
- Affects all projects
- Different on every machine

### With Docker

```dockerfile
# In Dockerfile
FROM ubuntu:22.04
RUN apt install gcc cmake qt6
```

**Results in:**
- Host system stays clean
- No version conflicts
- Delete container, everything gone
- Only affects this project
- Same on every machine

## What Docker Makes

When you build a Docker image, you get:

1. **A clean Linux filesystem**
   - Complete directory structure (/bin, /lib, /usr, etc.)
   - Just like a fresh Linux installation

2. **With exact tool versions**
   - The specific gcc you specified
   - The specific cmake you specified
   - The specific Qt you specified

3. **That runs isolated**
   - Own processes
   - Own filesystem
   - Own users
   - But shares kernel with host

4. **And can be deleted anytime**
   - Remove container: instant cleanup
   - No traces left on host
   - Can recreate from Dockerfile

## A Simple Example

**Traditional way:**
```bash
# Install system-wide
sudo apt install nodejs npm

# Now nodejs is on your system forever
# Might conflict with other projects
# Hard to have multiple versions
```

**Docker way:**
```dockerfile
FROM ubuntu:22.04
RUN apt install nodejs=14.*

# Runs in container
# Different project can use nodejs=16.*
# Delete container when done
# Host system untouched
```

## Why This Matters

Imagine you have 3 projects:
- Project A needs Qt 5
- Project B needs Qt 6
- Project C needs Qt 6.4 specifically

**Without Docker:**
- Install Qt 5: breaks Project B
- Install Qt 6: breaks Project A
- Version chaos

**With Docker:**
```dockerfile
# Project A
FROM ubuntu:20.04
RUN apt install qt5-default

# Project B
FROM ubuntu:22.04
RUN apt install qt6-base-dev

# Project C
FROM ubuntu:22.04
RUN apt install qt6-base-dev=6.4.*
```

Each project has its own isolated environment. No conflicts.

## The Core Insight

Docker doesn't do anything magical. It simply:

1. Creates an isolated Linux environment
2. Lets you specify exact versions
3. Keeps your host system clean
4. Makes it reproducible everywhere

**That's the entire concept.**

## Common Questions

**Q: If Docker uses my kernel, how is it isolated?**  
A: It uses Linux namespaces. The kernel is shared, but processes/filesystem/users are isolated.

**Q: Is it slower than running directly on host?**  
A: Almost no overhead. Containers are nearly as fast as native.

**Q: Can I run Windows apps in Docker?**  
A: Docker runs Linux containers. On Windows, Docker uses a Linux VM underneath.

**Q: Do I need to learn Linux to use Docker?**  
A: Basic Linux knowledge helps, but Docker simplifies many things.

## Summary

**Docker is:** A tool that creates clean, isolated Linux environments with fixed tool versions.

**Docker is NOT:** A VM, another computer, a CPU emulator, or an SDK replacement.

**Docker's purpose:** Eliminate "works on my machine" by making environments reproducible.

**How it works:** Creates isolated user-space environments that share the host kernel.

---

**Next:** [03. Docker vs Virtual Machines](03-docker-vs-vms.md) - Understand the crucial differences between containers and VMs.