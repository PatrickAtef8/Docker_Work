# 01. The Problem Docker Solves

## The "Works on My Machine" Problem

You and your friend both run Ubuntu.

**Your laptop:**
- gcc 11
- cmake 3.22
- Qt 6.6

**Friend's laptop:**
- gcc 9
- cmake 3.16
- Qt 5

**Result:**
- Works on your machine
- Fails on friend's machine

**This happens ALL THE TIME in software development.**

## Why Traditional Approaches Fail

### Option A: Send the Binary

You compile your app and send the binary file:

```bash
# You send: my_app
```

**Will it work?** Usually NO.

**Why?** The binary depends on:
- Specific Qt version
- Specific glibc version
- System libraries
- OS version
- Architecture

**Common error your friend sees:**
```
error while loading shared libraries:
libQt6Core.so.6: cannot open shared object file
```

**When it works:**
- Same OS
- Same Qt version
- Same architecture
- Same library paths

**In real life:** This is extremely fragile and unreliable.

### Option B: Send Source Code

You send the source files:

```bash
# You send: main.cpp, CMakeLists.txt
# Friend runs: cmake .. && make
```

**Will it work?** Maybe, maybe not.

**It fails if your friend has:**
- Qt5 instead of Qt6
- Different CMake version
- Missing packages
- Different compiler behavior
- Different library versions

**This is the famous "Works on my machine" syndrome.**

### Option C: Write Installation Instructions

You create a README:

```markdown
## Setup Instructions

1. Install Ubuntu 22.04
2. Install gcc 11:
   sudo apt install gcc-11
3. Install CMake 3.22:
   sudo apt install cmake
4. Install Qt 6.6:
   sudo apt install qt6-base-dev
5. Set environment variables:
   export QT_DIR=/usr/lib/qt6
   export PATH=$PATH:/usr/bin/gcc-11
6. Build:
   cmake ..
   make
```

**Problems:**
- Pollutes the system
- Version conflicts with existing software
- Hard to undo/remove
- Different on every OS
- Easy to make mistakes
- Time-consuming
- Doesn't guarantee exact versions

## The Real Problem

The core issue is: **environment inconsistency**

- Different OS versions
- Different tool versions
- Different library versions
- Different configurations
- Different paths

**Every developer has a slightly different environment.**

## What We Actually Need

An ideal solution would:

1. Package the exact environment (OS + tools + versions)
2. Work the same on every machine
3. Not pollute the host system
4. Be easy to create and share
5. Be fast and lightweight
6. Be easy to delete/reset

**This is exactly what Docker provides.**

## The Docker Solution

Instead of sending:
- Binary (fragile)
- Source + instructions (unreliable)

You send:
- Source code
- **Dockerfile** (environment contract)

**Dockerfile example:**
```dockerfile
FROM ubuntu:22.04

RUN apt update && apt install -y \
    build-essential \
    cmake=3.22.* \
    qt6-base-dev=6.4.*

COPY . /app
WORKDIR /app

RUN cmake -B build
RUN cmake --build build
```

**Key insight:** Versions are NOT random. YOU specify exact versions.

### How Your Friend Uses It

Your friend runs:
```bash
docker build -t myapp .
docker run myapp
```

**Result:**
- Same Ubuntu 22.04
- Same Qt 6.4
- Same CMake 3.22
- Same GCC
- Build succeeds
- **Guaranteed to work**

## The Key Realization

**Docker replaces "README install steps" with a machine-enforced environment.**

Instead of:
- Human reads instructions
- Human follows steps
- Human makes mistakes
- Environment varies slightly

Docker does:
- Machine reads Dockerfile
- Machine executes exactly
- No human error
- Identical environment every time

## Summary

**The problem:** "Works on my machine" - inconsistent environments break builds.

**Traditional solutions fail because:**
- Binaries are environment-dependent
- Source code assumes specific tools
- Installation instructions are error-prone

**Docker solves it by:**
- Packaging the exact environment
- Making builds reproducible
- Keeping host system clean
- Being fast and easy to share

---

**Next:** [02. What Docker Actually Is](../02-what-is-docker/README.md) - Learn what Docker really does under the hood.

