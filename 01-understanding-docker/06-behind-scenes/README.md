# 06. What Happens Behind the Scenes

This chapter reveals exactly what Docker does during `build` and `run` operations - no hand-waving, step by step.

## Scenario Setup

Your friend has:
- Different GCC version
- Different Qt version
- Different OS
- **Only Docker installed**

You send them:
- Source code
- Dockerfile

**That's enough for identical builds.**

## Step-by-Step: `docker build`

### Example Dockerfile

```dockerfile
FROM ubuntu:22.04

RUN apt update && apt install -y \
    build-essential \
    cmake \
    qt6-base-dev

COPY . /app
WORKDIR /app

RUN cmake -B build
RUN cmake --build build

CMD ["./build/myapp"]
```

### What Happens Line by Line

#### Line 1: `FROM ubuntu:22.04`

```bash
docker build -t myapp .
```

**Docker checks:**
```
"Do I have ubuntu:22.04 locally?"
```

**If NO:**
- Connects to Docker Hub
- Downloads Ubuntu 22.04 filesystem (~80 MB compressed)
- Extracts to `/var/lib/docker/overlay2/`
- This is a **full Linux filesystem** with:
  ```
  /bin    /lib    /usr
  /etc    /var    /home
  /boot   /dev    /proc
  ```

**If YES:**
- Uses cached copy (instant)

**Important:**
- This is NOT your friend's OS
- His Ubuntu/Arch/Fedora doesn't matter
- This is a fresh, fixed Ubuntu 22.04

#### Line 2: `RUN apt update && apt install ...`

**Docker:**
1. Creates a **temporary container** from current image state
2. Runs the command **inside that container**:
   ```bash
   apt update
   apt install -y build-essential cmake qt6-base-dev
   ```
3. Downloads packages from Ubuntu repos
4. Installs **inside container filesystem only**
5. Takes a **snapshot** of changes
6. Creates a new image layer
7. Destroys temporary container

**Critical:**
- Nothing installed on host
- Host `/usr/bin/gcc` unchanged
- Host `/usr/lib/` unchanged
- Everything in container only

**Isolation point:**

```
Friend's machine:
/usr/bin/gcc         ← Unused, untouched
/usr/lib/qt          ← Unused, untouched

Container filesystem:
/usr/bin/gcc         ← Used (newly installed)
/usr/lib/x86_64-linux-gnu/libQt6Core.so ← Used
```

#### Line 3: `COPY . /app`

**Docker:**
1. Reads files from **build context** (current directory on host)
2. Copies into **container filesystem** at `/app/`
3. Creates new layer

**Example:**
```
Host machine:
/home/you/project/main.cpp
/home/you/project/CMakeLists.txt

Container filesystem:
/app/main.cpp
/app/CMakeLists.txt
```

#### Line 4: `WORKDIR /app`

**Docker:**
- Sets working directory to `/app` for subsequent commands
- If `/app` doesn't exist, creates it
- Equivalent to `cd /app`

#### Line 5: `RUN cmake -B build`

**Docker:**
1. Creates temporary container
2. Runs command **inside container**:
   ```bash
   cd /app
   cmake -B build
   ```
3. CMake uses **container cmake** (not host cmake)
4. CMake uses **container libraries** (not host libraries)
5. Creates build directory in **container filesystem**
6. Generates Makefiles
7. Takes snapshot
8. Creates new layer

#### Line 6: `RUN cmake --build build`

**Docker:**
1. Creates temporary container
2. Runs:
   ```bash
   cd /app
   cmake --build build
   ```
3. Compiler used: **container gcc** (not host gcc)
4. Libraries linked: **container Qt** (not host Qt)
5. Output binary: `/app/build/myapp` (in container)
6. Takes snapshot
7. Creates final layer

#### Final Result

Docker produces an **image** named `myapp` containing:

```
Layer 1: Ubuntu 22.04 base          (80 MB)
Layer 2: build-essential, cmake, Qt (300 MB)
Layer 3: Your source code           (1 MB)
Layer 4: CMake configuration        (5 MB)
Layer 5: Compiled application       (10 MB)
────────────────────────────────────────────
Total: ~396 MB
```

**Stored:** `/var/lib/docker/overlay2/...`

**Important:** 
- Your friend's system is completely untouched
- No Qt on host
- No cmake on host
- Everything isolated in image

## Step-by-Step: `docker run`

Now your friend runs:
```bash
docker run myapp
```

### What Happens

#### 1. Container Creation

**Docker:**
- Creates a **new container** from the `myapp` image
- Assigns container ID: `8c9b34b33dac`
- Assigns random name: `interesting_bardeen` (if no `--name`)

#### 2. Namespace Setup

**Docker creates isolated namespaces:**

**PID namespace:**
```
Host processes:
PID 1: systemd
PID 2: kthreadd
PID 100: dockerd
...
PID 5000: your-other-apps

Container processes:
PID 1: ./build/myapp    ← Isolated view
PID 2: bash (if any)
```

From inside container: only sees its own processes.

**Network namespace:**
- Container gets its own network stack
- Own IP address (172.17.0.2 for example)
- Own routing table
- Isolated from host network (unless bridged)

**Mount namespace:**
- Container sees its own filesystem
- Host filesystem invisible (unless mounted with `-v`)

**User namespace:**
- Container has its own user IDs
- `root` in container ≠ `root` on host

#### 3. Filesystem Setup

**Docker uses OverlayFS:**

```
┌─────────────────────────────────────┐
│  Writable Layer (Container)         │  ← New files go here
├─────────────────────────────────────┤
│  Layer 5: Compiled app (Read-only)  │
├─────────────────────────────────────┤
│  Layer 4: CMake files (Read-only)   │
├─────────────────────────────────────┤
│  Layer 3: Source code (Read-only)   │
├─────────────────────────────────────┤
│  Layer 2: Tools (Read-only)         │
├─────────────────────────────────────┤
│  Layer 1: Ubuntu base (Read-only)   │
└─────────────────────────────────────┘
```

Container sees a unified filesystem but can only write to top layer.

#### 4. Command Execution

**Dockerfile had:**
```dockerfile
CMD ["./build/myapp"]
```

**Docker:**
1. Changes to working directory (set by `WORKDIR`)
2. Executes: `./build/myapp`
3. Process runs **inside container**
4. Uses **container libraries**:
   ```
   ./build/myapp
   └─ loads libQt6Core.so.6 (from container)
   └─ loads libstdc++.so.6 (from container)
   └─ makes syscalls to host kernel
   ```

#### 5. Process Lifecycle

```
Container starts
    ↓
CMD runs (./build/myapp)
    ↓
App executes
    ↓
App prints output to stdout
    ↓
App exits (return 0)
    ↓
Container stops
```

**Container lived only while the process was running.**

#### 6. What Your Friend Sees

```bash
docker run myapp
# Output:
Hello from Qt Application!
Application running...
Done.

# Container stops automatically
```

## The Kernel Magic

This is the most important part:

### Container Does NOT Have

- Its own kernel
- Its own init system
- Its own device drivers
- Its own kernel modules

### Container DOES Have

- Its own filesystem
- Its own processes (isolated view)
- Its own network stack
- Its own users

### Container SHARES

- **Host kernel** (same Linux kernel)
- Hardware access (through host kernel)
- System calls (to host kernel)

### Visualization

```
┌──────────────────────────────────────────┐
│         Your Friend's PC                 │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │  Linux Kernel 5.15 (HOST KERNEL)   │ │
│  │  - Manages CPU                     │ │
│  │  - Manages memory                  │ │
│  │  - Manages hardware                │ │
│  └────────────────────────────────────┘ │
│           ▲              ▲                │
│           │              │                │
│     ┌─────┴──────┐ ┌────┴────────┐      │
│     │ Host       │ │ Container   │      │
│     │ Processes  │ │ Processes   │      │
│     │            │ │  - myapp    │      │
│     │ (Isolated) │ │  (Isolated) │      │
│     └────────────┘ └─────────────┘      │
│                                          │
└──────────────────────────────────────────┘
```

**Both host and container processes use the SAME kernel.**

### Why This Matters

**System call example:**
```c
// Inside container
open("/app/myfile.txt", O_RDONLY);
```

**Flow:**
1. Container process makes syscall
2. Syscall goes to **host kernel** (not container kernel)
3. Host kernel checks namespace
4. Host kernel translates `/app/myfile.txt` to actual location
5. Host kernel performs operation
6. Returns result to container

**This is why containers are fast:** Direct kernel access, no emulation.

## Where Files Actually Live

### Docker's Storage

```bash
# On host (your friend's machine)
/var/lib/docker/
├── overlay2/
│   ├── abc123.../  ← Image layer 1
│   ├── def456.../  ← Image layer 2
│   ├── ghi789.../  ← Image layer 3
│   └── jkl012.../  ← Container writable layer
├── containers/
│   └── 8c9b34b33dac.../  ← Container metadata
└── images/
    └── myapp/  ← Image metadata
```

### Accessing Storage

**Wrong way (don't do this):**
```bash
cd /var/lib/docker
# Permission denied (owned by root)
```

**Right way:**
```bash
# Enter container
docker run -it myapp bash

# Now you're inside the container filesystem
ls /
ls /app
```

## Summary: Build vs Run

### `docker build` Creates:
- Image (static template)
- Stored on disk
- Contains: OS + tools + your app
- Can be shared

### `docker run` Creates:
- Container (running instance)
- Has process ID
- Uses image + writable layer
- Temporary (stops when process ends)

### Key Flow

```
Dockerfile
    ↓ docker build
Docker Image
    ↓ docker run
Container (running)
    ↓ app exits
Container (stopped)
    ↓ docker rm
Container deleted
    ↓
Image remains
```

## Performance Reality

**Why containers are fast:**

1. **No hypervisor:** Direct kernel access
2. **No hardware emulation:** Native CPU instructions
3. **Shared kernel:** No OS boot time
4. **Copy-on-write:** Efficient storage
5. **Process-level:** Just another process

**Startup time comparison:**
- VM: 30-120 seconds
- Container: <1 second

**Why?**
- VM: Must boot entire OS
- Container: Just starts a process

## What Your Friend's System Looks Like After

```bash
# Host system
/usr/bin/gcc         ← Unchanged (maybe gcc 9)
/usr/lib/qt          ← Unchanged (maybe Qt5)
/home/friend/        ← No build artifacts

# Docker storage
/var/lib/docker/
└── Images and containers (isolated)
```

**Host system completely clean.**

## Summary

**Docker build:**
1. Downloads base OS image
2. Executes each instruction in temporary containers
3. Takes snapshots after each step
4. Creates layered image
5. Nothing touches host system

**Docker run:**
1. Creates container from image
2. Sets up isolated namespaces
3. Mounts filesystem layers
4. Runs command inside container
5. Uses host kernel for all operations
6. Stops when process exits

**Key insight:** Docker is just a process isolation system using Linux kernel features. No magic, no emulation, just clever use of namespaces and layered filesystems.

---

**Next:** [07. Complete Practical Example](07-practical-example.md) - Walk through a real build and explore the container.