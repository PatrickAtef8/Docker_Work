# 05. Images and Containers

**Reading time:** 5 minutes

Understanding the difference between images and containers is crucial to mastering Docker.

## The Fundamental Distinction

### Image

**Definition:** A static, read-only template containing everything needed to run an application.

**Characteristics:**
- Static snapshot
- Read-only (cannot be modified directly)
- Stored on disk
- Template/Blueprint
- Can be shared and downloaded

**Example:**
- `ubuntu:22.04`
- `node:18-alpine`
- `my-qt-app:v1.0`

**Think of it as:**
- A class definition (in programming)
- A recipe (in cooking)
- A blueprint (in construction)
- A master copy (in manufacturing)

### Container

**Definition:** A running instance of an image.

**Characteristics:**
- Dynamic and running
- Has a process ID
- Can be started, stopped, paused
- Temporary (unless committed)
- Has writable layer on top of image

**Example:**
- Container ID: `8c9b34b33dac`
- Container name: `mycontainer`

**Think of it as:**
- An object instance (in programming)
- A cooked meal (in cooking)
- A built building (in construction)
- A manufactured product (in manufacturing)

## Perfect Analogies

| Concept | Image | Container |
|---------|-------|-----------|
| **Programming** | Class | Object |
| **Cooking** | Recipe | Cooked meal |
| **Construction** | Blueprint | Building |
| **Music** | Sheet music | Performance |
| **Photography** | Negative | Printed photo |

## Visualization

```
┌─────────────────────────────┐
│   Docker Image              │
│   (patrick-first-image)     │
│   - Ubuntu 22.04            │
│   - g++ compiler            │
│   - Compiled app            │
│   READ-ONLY                 │
└─────────────────────────────┘
         │
         │ docker run (creates instance)
         │
         ├─────────────────────────────┐
         │                             │
         ▼                             ▼
┌──────────────────┐         ┌──────────────────┐
│   Container 1    │         │   Container 2    │
│   (mycontainer)  │         │   (test1)        │
│   RUNNING        │         │   STOPPED        │
│   Writable layer │         │   Writable layer │
└──────────────────┘         └──────────────────┘
```

## One Image, Many Containers

This is a key concept:

```bash
# One image
docker images
# REPOSITORY            TAG       IMAGE ID
# patrick-first-image   latest    abc123def456

# Multiple containers from same image
docker run --name container1 patrick-first-image
docker run --name container2 patrick-first-image
docker run --name container3 patrick-first-image

# All three containers share the same base image
# But each has its own writable layer
```

## Lifecycle Comparison

### Image Lifecycle

```bash
# Create image
docker build -t myimage .

# Image exists on disk
docker images
# Shows: myimage

# Share image
docker push myimage:latest

# Download image
docker pull ubuntu:22.04

# Delete image
docker rmi myimage
```

**Images are persistent until explicitly deleted.**

### Container Lifecycle

```bash
# Create and start container
docker run --name mycontainer myimage

# Container is running
docker ps
# Shows: mycontainer (Status: Up)

# Stop container
docker stop mycontainer
# Container still exists, but stopped

# Container is stopped
docker ps -a
# Shows: mycontainer (Status: Exited)

# Start again
docker start mycontainer

# Remove container
docker rm mycontainer
```

**Containers are ephemeral unless you keep them.**

## Storage Layers

### How Images Store Data

Images are built in **layers**:

```dockerfile
FROM ubuntu:22.04           # Layer 1: Base Ubuntu
RUN apt update              # Layer 2: Package cache
RUN apt install gcc         # Layer 3: GCC installed
COPY main.cpp /app/         # Layer 4: Your source
RUN gcc main.cpp -o app     # Layer 5: Compiled binary
```

Each instruction creates a new read-only layer.

### How Containers Store Data

Containers add a **writable layer** on top:

```
┌─────────────────────────┐
│  Writable Layer         │  ← Container changes here
├─────────────────────────┤
│  Layer 5: Compiled app  │  ↑
├─────────────────────────┤  │
│  Layer 4: Source code   │  │ Read-only
├─────────────────────────┤  │ (from image)
│  Layer 3: GCC           │  │
├─────────────────────────┤  │
│  Layer 2: Package cache │  │
├─────────────────────────┤  │
│  Layer 1: Ubuntu base   │  ↓
└─────────────────────────┘
```

**Changes in container:**
- New files → writable layer
- Modified files → copy to writable layer
- Deleted files → marked as deleted in writable layer

**When container is deleted:**
- Writable layer is destroyed
- Base image layers remain unchanged

## Practical Examples

### Example 1: Creating from Image

```bash
# Build image once
docker build -t myapp .

# Create multiple containers
docker run --name test1 myapp
docker run --name test2 myapp
docker run --name test3 myapp

# All containers share the same image
# But each has independent writable layer
```

### Example 2: Image Reusability

```bash
# Developer 1
docker build -t myapp:v1.0 .
docker push myapp:v1.0

# Developer 2 (different machine)
docker pull myapp:v1.0
docker run myapp:v1.0

# Same image, different container
```

### Example 3: Container Independence

```bash
# Start two containers
docker run -it --name c1 ubuntu bash
docker run -it --name c2 ubuntu bash

# In container 1
echo "Hello from C1" > /tmp/file.txt
exit

# In container 2
cat /tmp/file.txt
# Error: No such file or directory

# Why? Different containers, different writable layers
```

## Commands Comparison

| Operation | Image Command | Container Command |
|-----------|---------------|-------------------|
| List | `docker images` | `docker ps -a` |
| Remove | `docker rmi myimage` | `docker rm mycontainer` |
| Create | `docker build -t myimage .` | `docker run myimage` |
| Inspect | `docker inspect myimage` | `docker inspect mycontainer` |
| Save/Export | `docker save myimage` | `docker export mycontainer` |

## Common Confusion: Saving Container Changes

If you modify a container, those changes are NOT in the image:

```bash
# Start container
docker run -it --name mycontainer ubuntu bash

# Inside container: install something
apt update && apt install vim

# Exit container
exit

# Start again - vim is still there
docker start -ai mycontainer

# BUT: If you create a new container from the same image
docker run -it ubuntu bash
# vim is NOT installed here
```

**To save container changes to a new image:**
```bash
docker commit mycontainer mynewimage:v2
```

## Storage Efficiency

Multiple containers from the same image are space-efficient:

```bash
# Image size: 500 MB

# Run 10 containers
docker run --name c1 myimage
docker run --name c2 myimage
# ... up to c10

# Disk usage:
# - Image: 500 MB (stored once)
# - Containers: 10 x ~1 MB = 10 MB (only writable layers)
# Total: ~510 MB (not 5000 MB!)
```

## Summary Table

| Aspect | Image | Container |
|--------|-------|-----------|
| **Nature** | Static template | Running instance |
| **State** | Read-only | Has writable layer |
| **Lifetime** | Persistent | Ephemeral |
| **Can run** | No | Yes |
| **Can modify** | No (create new) | Yes |
| **Storage** | Layered filesystem | Image layers + writable layer |
| **Sharing** | Easy (push/pull) | Not typical |
| **Analogy** | Class/Recipe | Object/Meal |

## Key Takeaways

1. **Image = Template, Container = Instance**
   - One image can create many containers
   - Each container is independent

2. **Images are permanent, Containers are temporary**
   - Images persist until deleted
   - Containers can be stopped and removed

3. **Images are read-only, Containers are writable**
   - Changes in container don't affect image
   - Need to commit to create new image

4. **Images are shared, Containers are local**
   - Push/pull images to registries
   - Containers exist only on local machine

**Mental model:**
- Image = Master template you keep
- Container = Working copy you use and throw away

---

**Next:** [06. What Happens Behind the Scenes](06-behind-scenes.md) - See exactly what Docker does during build and run.