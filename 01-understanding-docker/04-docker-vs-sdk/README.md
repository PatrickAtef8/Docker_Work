# 04. Docker vs SDK and Cross-Compilation

This chapter is especially important for embedded developers working with Yocto, cross-compilation toolchains, and target hardware like Raspberry Pi.

## What an SDK Actually Does

### Example: Yocto SDK

When you run:
```bash
bitbake meta-toolchain-qt6
source environment-setup-aarch64-poky-linux
```

You get a **Software Development Kit** containing:

1. **Cross compiler**
   - `aarch64-poky-linux-gcc` (compiles for ARM on x86)
   - `aarch64-poky-linux-g++`
   - Target-specific compiler flags

2. **Sysroot**
   - Target system headers
   - Target libraries
   - Target filesystem structure

3. **Target libraries**
   - Qt libraries compiled for ARM
   - System libraries for target
   - Architecture-specific binaries

4. **Build tools configuration**
   - CMake toolchain file
   - qmake configuration
   - pkg-config settings

5. **Environment variables**
   - `CC`, `CXX` point to cross-compiler
   - `PKG_CONFIG_PATH` points to target libs
   - `SDKTARGETSYSROOT` set

### When You Build

```bash
# After sourcing SDK environment
make myapplication
```

**What happens:**
- Compiler runs on **host** (x86)
- Binary is for **target** (ARM)
- Output is architecture-specific

**This is cross-compilation.**

## What Docker Does (Differently)

Docker focuses on:

1. **Build environment isolation**
   - Fixed Ubuntu version
   - Fixed tool versions
   - Clean, reproducible environment

2. **Dependency management**
   - All tools in one place
   - No host pollution
   - Easy to share

3. **Reproducibility**
   - Same result on every machine
   - No "works on my machine"
   - CI/CD friendly

**Docker does NOT change target architecture.**

## Critical Comparison

| Aspect | Yocto SDK | Docker |
|--------|-----------|---------|
| **Purpose** | Cross-compile for different CPU | Isolate build environment |
| **Changes output architecture** | YES - x86 to ARM | NO - same as host |
| **Depends on host OS** | YES - needs specific libraries | NO - brings own environment |
| **Uses host kernel** | YES | YES |
| **Reproducible builds** | Limited (host dependencies) | Strong (containerized) |
| **Replaces the other** | NO | NO |
| **Installed where** | Host system | Isolated container |

## One-Sentence Truth

**Docker isolates WHERE you build. SDK defines WHAT you build for.**

They solve two completely different problems.

## Common Misconceptions

### Misconception 1: "Docker makes my PC into a Raspberry Pi"

**Reality:**
```bash
# You're on x86 PC
# Start Docker container
docker run -it ubuntu bash

# Inside container
uname -m
# Output: x86_64  (NOT arm64!)

# You're STILL on x86
# Container shares your x86 kernel
# Docker did NOT change your CPU
```

### Misconception 2: "I can run ARM binaries in Docker on x86"

**Reality:**

```bash
# Inside Docker container on x86
./my-arm-binary
# Error: cannot execute binary file: Exec format error

# Why? Because:
# - Container is still x86
# - CPU is still x86
# - ARM instructions don't work on x86
```

**To run ARM binaries on x86, you need:**
1. Real ARM hardware (Raspberry Pi), OR
2. QEMU emulation (slow), OR
3. Docker + QEMU (advanced setup)

### Misconception 3: "Docker replaces my SDK"

**Reality:**

Docker doesn't cross-compile by itself.

**Without SDK:**
```dockerfile
FROM ubuntu:22.04
RUN apt install gcc

# gcc compiles for x86 (host architecture)
# No cross-compilation happening
```

**With SDK in Docker:**
```dockerfile
FROM ubuntu:22.04
COPY yocto-sdk.sh /tmp/
RUN /tmp/yocto-sdk.sh -y -d /opt/sdk

# NOW you have cross-compilation
# SDK provides aarch64-poky-linux-gcc
```

## Correct Understanding: Docker + SDK

The professional embedded workflow:

```
Host Machine (x86)
└─ Docker Container (clean Ubuntu 22.04)
   └─ Yocto SDK (aarch64 toolchain)
      └─ Build Qt Application
         └─ Output: ARM binary
            └─ Deploy to Raspberry Pi
```

### Example Dockerfile

```dockerfile
FROM ubuntu:22.04

# Install base dependencies
RUN apt update && apt install -y \
    build-essential \
    cmake \
    python3 \
    git

# Copy and install Yocto SDK
COPY poky-glibc-x86_64-meta-toolchain-qt6-aarch64-toolchain-4.0.sh /tmp/sdk.sh
RUN chmod +x /tmp/sdk.sh && \
    /tmp/sdk.sh -y -d /opt/poky

# Set up SDK environment
ENV PATH="/opt/poky/sysroots/x86_64-pokysdk-linux/usr/bin:${PATH}"
ENV SDKTARGETSYSROOT="/opt/poky/sysroots/aarch64-poky-linux"

WORKDIR /workspace
```

### Building with Docker + SDK

```bash
# Build the environment (once)
docker build -t yocto-qt6-builder .

# Use it to cross-compile
docker run -v $(pwd):/workspace yocto-qt6-builder \
    bash -c "source /opt/poky/environment-setup-aarch64-poky-linux && make"
```

**Result:**
- Docker: Clean, reproducible Ubuntu 22.04 environment
- SDK: Cross-compiles for ARM
- Output: ARM binary (runs on Raspberry Pi)

## Why Use Both?

### Problem Without Docker

Team member 1:
```bash
# Ubuntu 20.04, SDK installed in /home/user/sdk
source /home/user/sdk/environment-setup-*
make
```

Team member 2:
```bash
# Ubuntu 22.04, SDK installed in /opt/sdk
source /opt/sdk/environment-setup-*
make
# ERROR: Different Python version, missing library
```

CI server:
```bash
# CentOS, SDK location different
# Different system libraries
# Build fails
```

### Solution With Docker + SDK

Everyone runs:
```bash
docker run yocto-builder make
```

**Same:**
- Ubuntu version
- SDK location
- System libraries
- Python version
- Build succeeds everywhere

## Real-World Automotive Example

```dockerfile
FROM ubuntu:22.04

# Install Qt 6 build dependencies
RUN apt update && apt install -y \
    build-essential \
    cmake \
    python3 \
    libgl1-mesa-dev

# Install Yocto SDK for automotive ECU (ARM)
COPY automotive-sdk-installer.sh /tmp/
RUN /tmp/automotive-sdk-installer.sh -d /opt/automotive-sdk

# Set up environment
ENV CMAKE_TOOLCHAIN_FILE="/opt/automotive-sdk/cmake-toolchain.cmake"

WORKDIR /workspace
```

Usage:
```bash
# Developer 1 (Ubuntu)
docker run -v $(pwd):/workspace automotive-builder cmake --build build

# Developer 2 (Arch Linux)
docker run -v $(pwd):/workspace automotive-builder cmake --build build

# CI Server (CentOS)
docker run -v $(pwd):/workspace automotive-builder cmake --build build

# All produce IDENTICAL ARM binaries
```

## Flow Diagram

```
┌─────────────────────────────────────────┐
│  Your x86 PC (Ubuntu/Windows/Mac)       │
│  ┌───────────────────────────────────┐  │
│  │  Docker Container                 │  │
│  │  (Fixed Ubuntu 22.04)             │  │
│  │  ┌─────────────────────────────┐  │  │
│  │  │  Yocto SDK                  │  │  │
│  │  │  (aarch64 cross-compiler)   │  │  │
│  │  │  ┌───────────────────────┐  │  │  │
│  │  │  │  Your Qt App          │  │  │  │
│  │  │  │  (cross-compiled)     │  │  │  │
│  │  │  │                       │  │  │  │
│  │  │  │  Output: ARM binary   │  │  │  │
│  │  │  └───────────────────────┘  │  │  │
│  │  └─────────────────────────────┘  │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
                    │
                    │ Deploy
                    ▼
┌─────────────────────────────────────────┐
│  Raspberry Pi / Automotive ECU (ARM)    │
│  Runs the ARM binary                    │
└─────────────────────────────────────────┘
```

## When You DON'T Need Docker

If you're only working alone and your environment is stable:
```bash
# Just use SDK directly
bitbake meta-toolchain-qt6
source environment-setup-*
make
```

Docker adds value when:
- Multiple developers
- CI/CD pipelines
- Different host systems
- Need reproducibility
- Want clean environments

## Summary

| What | Docker | SDK | Docker + SDK |
|------|--------|-----|--------------|
| **Isolates environment** | YES | NO | YES |
| **Cross-compiles** | NO | YES | YES |
| **Changes architecture** | NO | YES | YES |
| **Reproducible builds** | YES | Limited | YES |
| **Clean host system** | YES | NO | YES |

**Key insights:**
1. Docker does NOT make your x86 PC into ARM
2. Docker does NOT run ARM binaries on x86 (without emulation)
3. Docker DOES provide clean, reproducible environments
4. SDK DOES cross-compile for target architecture
5. Docker + SDK = Professional embedded workflow

**Think of it this way:**
- Docker = Clean workshop
- SDK = Tools for building for ARM
- Your app = What you're building
- Raspberry Pi = Where it runs

---

**Next:** [05. Images and Containers](../05-images-containers/README.md) - Understand the fundamental Docker concepts.