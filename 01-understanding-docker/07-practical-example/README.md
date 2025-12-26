# 07. Complete Practical Example

This chapter walks through a complete, real example from writing code to running it in Docker.

## The Application

A simple C++ program that proves Docker is working.

### main.cpp

```cpp
#include <iostream>

int main() { 
    std::cout << "Hello Docker is Working" << std::endl;
    return 0;
}
```

### Dockerfile

```dockerfile
FROM ubuntu:22.04

RUN apt update && apt install -y g++

WORKDIR /app

COPY main.cpp .

RUN g++ main.cpp -o app

CMD ["./app"]
```

## Building the Image

### Command

```bash
docker build -t patrick-first-image .
```

**What this means:**
- `build`: Create image from Dockerfile
- `-t patrick-first-image`: Tag (name) the image
- `.`: Build context (current directory)

### Build Output (Annotated)

```bash
$ docker build -t patrick-first-image .

[1/6] FROM ubuntu:22.04
# Docker checks if ubuntu:22.04 exists locally
# If not, downloads from Docker Hub (~80 MB)
# This is a complete Ubuntu filesystem

[2/6] RUN apt update && apt install -y g++
# Creates temporary container
# Runs apt update and install
# Installs g++ and dependencies (~200 MB)
# Takes snapshot, creates layer
# Deletes temporary container

[3/6] WORKDIR /app
# Creates /app directory in image
# Sets it as working directory

[4/6] COPY main.cpp .
# Copies main.cpp from host to /app/ in container
# Creates layer (~1 KB)

[5/6] RUN g++ main.cpp -o app
# Creates temporary container
# Compiles: g++ main.cpp -o app
# Binary created at /app/app
# Takes snapshot
# Deletes temporary container

[6/6] CMD ["./app"]
# Sets default command to run when container starts
# Does not execute now, just stores instruction

Successfully built abc123def456
Successfully tagged patrick-first-image:latest
```

### What Was Created

```bash
docker images
```

**Output:**
```
REPOSITORY            TAG       IMAGE ID       SIZE
patrick-first-image   latest    abc123def456   350MB
```

**Image contents:**
- Ubuntu 22.04 base
- g++ compiler
- Standard library
- main.cpp source
- Compiled binary (app)

**Where stored:** `/var/lib/docker/` (managed by Docker)

## Running the Container

### Simple Run

```bash
docker run patrick-first-image
```

**Output:**
```
Hello Docker is Working
```

**What happened:**
1. Docker created container from image
2. Executed: `./app` (from CMD)
3. Program printed message
4. Program exited (return 0)
5. Container stopped

**Duration:** < 1 second

### Verification

```bash
docker ps -a
```

**Output:**
```
CONTAINER ID   IMAGE                 STATUS                  NAMES
8c9b34b33dac   patrick-first-image   Exited (0) 5 seconds ago   happy_keldysh
```

Container existed briefly, then exited successfully.

## Exploring the Container Filesystem

### Enter the Container

```bash
docker run -it patrick-first-image bash
```

**Flags:**
- `-i`: Interactive (keep stdin open)
- `-t`: Allocate pseudo-TTY (terminal)
- `bash`: Override CMD, run bash instead

**You'll see:**
```
root@1d9f3e9fa668:/app#
```

**You're now inside the container as root.**

### Navigate the Filesystem

```bash
# Check current directory
pwd
# /app

# List files
ls
# main.cpp  app

# Move to root
cd ..

# List root directory
ls /
```

**Output:**
```
app   bin   boot  dev  etc  home  lib  lib32  lib64  libx32
media mnt   opt   proc root run   sbin srv   sys   tmp
usr   var
```

**This is a complete Linux filesystem.**

### Key Observations

**1. Complete Linux structure**
```bash
ls /bin
# bash, cat, cp, ls, mkdir, rm, ...

ls /usr/bin
# g++, gcc, ld, ar, ...
```

**2. Your application is there**
```bash
ls /app
# main.cpp  app

# Run it
./app
# Hello Docker is Working
```

**3. It's Ubuntu 22.04**
```bash
cat /etc/os-release
```
**Output:**
```
NAME="Ubuntu"
VERSION="22.04.1 LTS (Jammy Jellyfish)"
```

**4. Tools are isolated**
```bash
which g++
# /usr/bin/g++

g++ --version
# g++ (Ubuntu 11.3.0-1ubuntu1) 11.3.0
```

This is the container's g++, not your host's.

### Understanding What You're Seeing

You are NOT in:
- A virtual machine
- A different computer
- Your host filesystem

You ARE in:
- An isolated filesystem view
- Using your host kernel
- Running as a process on your host

**Proof:**
```bash
# Inside container
uname -r
# 5.15.0-58-generic

# Exit container
exit

# On host
uname -r
# 5.15.0-58-generic

# SAME KERNEL
```

## Isolation Verification

### Test 1: Container Files Don't Exist on Host

**Inside container:**
```bash
docker run -it patrick-first-image bash

# Create a file
echo "Hello from container" > /tmp/test.txt
cat /tmp/test.txt
# Hello from container

exit
```

**On host:**
```bash
cat /tmp/test.txt
# cat: /tmp/test.txt: No such file or directory
```

**Why?** Different filesystem namespaces.

### Test 2: Host Files Don't Exist in Container

**On host:**
```bash
echo "Hello from host" > ~/host-file.txt
```

**Inside container:**
```bash
docker run -it patrick-first-image bash

ls ~
# (empty or different files)

cat ~/host-file.txt
# cat: cannot open: No such file
```

**Why?** Container doesn't have access to host filesystem.

## Why It Works Regardless of Host

Let's prove Docker's power:

### Your System

```bash
# Check your gcc
which gcc
# /usr/bin/gcc

gcc --version
# gcc version 9.4.0
```

### Container System

```bash
docker run -it patrick-first-image bash

which gcc
# /usr/bin/gcc (but different file!)

gcc --version
# gcc version 11.3.0
```

### The Application

```bash
docker run patrick-first-image
# Hello Docker is Working

# Works because:
# - Uses container's gcc 11.3.0
# - Uses container's libstdc++
# - Uses container's OS
# - Doesn't care about host gcc 9.4.0
```

**Your host could have:**
- gcc 8, 9, 10, 12, or none at all
- Qt 5, Qt 6, or no Qt
- Ubuntu, Fedora, Arch, or anything

**Container always has:**
- Exact versions specified in Dockerfile
- Same environment every time

## Container Lifecycle Demonstration

### Create Container

```bash
docker run -it --name mytest patrick-first-image bash
```

**Inside container:**
```bash
# Install something
apt update
apt install -y vim

# Create files
echo "persistent data" > /app/data.txt

# Exit
exit
```

### Container Still Exists

```bash
docker ps -a
```
**Output:**
```
CONTAINER ID   STATUS                     NAMES
d136419d05e2   Exited (0) 10 seconds ago  mytest
```

### Restart Container

```bash
docker start -ai mytest
```

**Inside container:**
```bash
# vim is still there
which vim
# /usr/bin/vim

# Files are still there
cat /app/data.txt
# persistent data
```

**Why?** Container's writable layer persists until you delete container.

### Delete Container

```bash
exit

docker rm mytest
```

Now create fresh container:
```bash
docker run -it patrick-first-image bash

# vim NOT installed
which vim
# (nothing)

# File NOT there
cat /app/data.txt
# cat: cannot open
```

**Why?** New container = fresh writable layer.

## Performance Comparison

### Startup Time

**Container:**
```bash
time docker run patrick-first-image
# Hello Docker is Working

# real    0m0.342s
# user    0m0.024s
# sys     0m0.018s
```

**Virtual Machine (for comparison):**
```bash
time VBoxManage startvm Ubuntu
# (waits for boot...)

# real    1m23.456s
```

**Container is 240x faster.**

### Resource Usage

**Check while running:**
```bash
# Terminal 1: Run container
docker run -it patrick-first-image bash

# Terminal 2: Check resources
docker stats
```

**Output:**
```
CONTAINER ID   CPU %   MEM USAGE / LIMIT   MEM %
8c9b34b33dac   0.00%   3.5MB / 16GB       0.02%
```

**Minimal overhead.**

## The "Aha" Moment Visualization

```
Your Host Machine
├─ Your OS: Ubuntu 20.04
├─ Your gcc: version 9
├─ Your Qt: version 5
│
└─ Docker Container (Running)
   ├─ Container OS: Ubuntu 22.04
   ├─ Container gcc: version 11
   ├─ Container Qt: version 6
   └─ Your app: Built with above
   
Shared:
└─ Linux Kernel: 5.15 (used by both)
```

**Host and container are isolated but share the kernel.**

## Summary of What We Proved

1. **Docker creates complete Linux environments**
   - Full filesystem
   - All necessary tools
   - Isolated from host

2. **Container shares host kernel**
   - Same `uname -r`
   - No separate kernel
   - Fast and efficient

3. **Changes are isolated**
   - Files in container ≠ files on host
   - Installing in container doesn't affect host
   - Clean and safe

4. **Reproducible**
   - Same Dockerfile = same environment
   - Works on any Linux host
   - No "works on my machine"

5. **Fast and lightweight**
   - Millisecond startup
   - Minimal resource usage
   - Many containers possible

---

**Next:** [08. Container Management](08-management.md) - Learn how to manage, name, and organize containers.