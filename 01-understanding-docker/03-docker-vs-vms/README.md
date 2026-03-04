# 03. Docker vs Virtual Machines


## The Core Difference

**Virtual Machines virtualize hardware. Docker virtualizes the user-space environment.**

This one distinction explains everything else.

## Architecture Comparison

### Virtual Machine Architecture

```
┌─────────────────────────────────┐
│         Your Application        │
├─────────────────────────────────┤
│     Guest OS (Full Linux)       │
│     - Own kernel                │
│     - Own init system           │
│     - Own drivers               │
├─────────────────────────────────┤
│     Virtual Hardware            │
│     - Virtual CPU               │
│     - Virtual RAM               │
│     - Virtual Disk              │
├─────────────────────────────────┤
│     Hypervisor                  │
│     (VirtualBox, VMware, KVM)   │
├─────────────────────────────────┤
│     Host Operating System       │
└─────────────────────────────────┘
```

**Key characteristics:**
- Complete operating system inside
- Has its own kernel
- Emulates hardware
- Heavy (GBs of disk space)
- Slow startup (minutes)
- Strong isolation

### Docker Container Architecture

```
┌─────────────────────────────────┐
│         Your Application        │
├─────────────────────────────────┤
│     Container User Space        │
│     - Libraries                 │
│     - Binaries                  │
│     - Your files                │
├─────────────────────────────────┤
│     Docker Engine               │
├─────────────────────────────────┤
│     Host OS Kernel (SHARED)     │
├─────────────────────────────────┤
│     Host Operating System       │
└─────────────────────────────────┘
```

**Key characteristics:**
- No guest kernel
- Uses host kernel
- No hardware emulation
- Lightweight (MBs of disk space)
- Fast startup (milliseconds)
- Process-level isolation

## How Docker Achieves Isolation

Docker uses Linux kernel features:

### 1. Namespaces
Isolate what processes can see:

- **PID namespace:** Container sees only its own processes
- **Network namespace:** Container has its own network stack
- **Mount namespace:** Container has its own filesystem view
- **User namespace:** Container has its own user IDs
- **IPC namespace:** Container has its own inter-process communication

**Example:**
```bash
# On host
ps aux
# Shows all processes: 1000+ processes

# Inside container
ps aux
# Shows only container processes: 5 processes
```

### 2. Control Groups (cgroups)
Limit resource usage:

- CPU limits
- Memory limits
- Disk I/O limits
- Network bandwidth limits

**Example:**
```bash
docker run --memory="512m" --cpus="1.5" myapp
# Container limited to 512MB RAM and 1.5 CPU cores
```

### 3. OverlayFS (Filesystem Layers)
Efficient storage:

- Base image is read-only
- Each change creates a new layer
- Multiple containers share base layers
- Only differences are stored

## Detailed Comparison

| Feature | Virtual Machine | Docker Container |
|---------|----------------|------------------|
| **Kernel** | Own kernel (guest) | Shared kernel (host) |
| **Boot time** | 30 seconds - 2 minutes | Milliseconds |
| **Size** | GBs (2-20 GB typical) | MBs (50-500 MB typical) |
| **Memory overhead** | High (512MB - 2GB+) | Low (few MB) |
| **Performance** | Slower (virtualization overhead) | Near-native |
| **Isolation** | Complete (hardware level) | Process-level |
| **OS diversity** | Can run different OS families | Must be same kernel |
| **Portability** | Heavy to move | Lightweight to share |
| **Startup** | Full OS boot | Process start |
| **Density** | 10s per host | 100s per host |

## Practical Examples

### Example 1: Starting Time

**Virtual Machine:**
```bash
# Start VM
VBoxManage startvm "Ubuntu"
# Time: 45 seconds to 2 minutes
# Why: Full OS boot, init system, services
```

**Docker Container:**
```bash
# Start container
docker run ubuntu echo "Hello"
# Time: < 1 second
# Why: Just starts a process
```

### Example 2: Resource Usage

**Virtual Machine:**
```bash
# Running Ubuntu VM
Memory: 2 GB (allocated, even if unused)
Disk: 10 GB (full OS installation)
CPU: Always some overhead
```

**Docker Container:**
```bash
# Running Ubuntu container
Memory: ~10 MB (only what's actually used)
Disk: ~200 MB (only needed binaries)
CPU: No overhead
```

### Example 3: Multiple Instances

**Virtual Machines:**
```bash
# Run 3 Ubuntu VMs
Memory: 6 GB (3 x 2 GB)
Disk: 30 GB (3 x 10 GB)
Time to start all: ~3 minutes
```

**Docker Containers:**
```bash
# Run 3 Ubuntu containers
Memory: ~30 MB (shared base + differences)
Disk: ~200 MB (shared layers)
Time to start all: < 3 seconds
```

## When Containers Share the Kernel

This is the most important concept:

```
Host: Linux kernel 5.15
└─ Container 1: Ubuntu 20.04 userspace
└─ Container 2: Ubuntu 22.04 userspace
└─ Container 3: Debian 11 userspace
└─ Container 4: Alpine 3.18 userspace
```

**All containers use the SAME Linux 5.15 kernel.**

What differs:
- Filesystem layout
- Installed packages
- Libraries
- System configuration

What's shared:
- Kernel
- Hardware access
- System calls

## Critical Limitations

Because containers share the kernel:

### Cannot Run Different Kernel OS

**Works:**
```bash
# Host: Linux
docker run ubuntu        # Linux userspace
docker run debian        # Linux userspace
docker run alpine        # Linux userspace
```

**Does NOT work (natively):**
```bash
# Host: Linux kernel
docker run windows       # Windows kernel - NO
docker run freebsd       # BSD kernel - NO
```

### Cannot Run Different Architecture (Without Emulation)

**Works:**
```bash
# Host: x86_64
docker run ubuntu:amd64  # Same architecture
```

**Does NOT work (without QEMU):**
```bash
# Host: x86_64
docker run ubuntu:arm64  # Different architecture - NO
```

## Why VMs Still Matter

Docker doesn't replace VMs. They serve different purposes:

### Use Virtual Machines When:
- Need to run different OS (Windows on Linux)
- Need complete isolation (security-critical)
- Need different kernel versions
- Testing OS-level features
- Running on different CPU architecture

### Use Docker Containers When:
- Need fast startup
- Need high density (many instances)
- Need reproducible builds
- Need lightweight packaging
- Need to share environments

## Hybrid Approach: Docker on Non-Linux

**On Windows/Mac:**
Docker actually uses a lightweight VM underneath:

```
Windows/Mac Host
└─ Lightweight Linux VM
   └─ Docker Engine
      └─ Linux Containers
```

But you don't manage the VM directly - Docker handles it.

## Performance Reality

**Virtual Machine:**
- System call: goes through hypervisor
- Memory access: through virtualization layer
- I/O operations: virtualized drivers

**Docker Container:**
- System call: directly to host kernel
- Memory access: direct (with namespaces)
- I/O operations: direct to host

**Result:** Containers have nearly zero overhead.

## Security Comparison

**Virtual Machines:**
- Stronger isolation (hardware boundaries)
- Guest escape is harder
- Complete OS separation

**Docker Containers:**
- Process-level isolation
- Kernel shared (potential attack surface)
- Need proper configuration for security

**Important:** For most development/build use cases, Docker isolation is sufficient.

## Summary

**Virtual Machines:**
- Full OS with own kernel
- Heavy but strongly isolated
- Slow startup, high overhead
- Can run different OS families

**Docker Containers:**
- Shared kernel, isolated userspace
- Lightweight and fast
- Millisecond startup, low overhead
- Must be same kernel family

**Key insight:** Containers isolate the environment, not the hardware.

**Mental model:** 
- VM = Complete computer inside your computer
- Container = Isolated process with its own filesystem

---

**Next:** [04. Docker vs SDK and Cross-Compilation](../04-docker-vs-sdk/README.md) - Learn how Docker fits with embedded development and SDKs.