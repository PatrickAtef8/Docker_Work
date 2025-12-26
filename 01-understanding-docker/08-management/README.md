# 08. Container Management

**Reading time:** 10 minutes

Learn how to effectively list, start, stop, name, and clean up containers.

## Listing Containers

### Show Running Containers

```bash
docker ps
```

**Example output:**
```
CONTAINER ID   IMAGE                 COMMAND   STATUS        NAMES
8c9b34b33dac   patrick-first-image   "bash"    Up 2 minutes  mycontainer
```

### Show All Containers (Including Stopped)

```bash
docker ps -a
```

**Example output:**
```
CONTAINER ID   IMAGE                 COMMAND    CREATED         STATUS                     NAMES
8c9b34b33dac   patrick-first-image   "bash"     5 minutes ago   Up 2 minutes              mycontainer
336225de8d12   patrick-first-image   "bash"     10 minutes ago  Exited (0) 8 minutes ago  test1
691867bfe6b6   patrick-first-image   "./app"    15 minutes ago  Exited (0) 14 minutes ago interesting_bardeen
```

### Column Meanings

| Column | Meaning | Example |
|--------|---------|---------|
| CONTAINER ID | Unique 12-character ID | 8c9b34b33dac |
| IMAGE | Source image | patrick-first-image |
| COMMAND | Command being run | bash |
| CREATED | When container was created | 5 minutes ago |
| STATUS | Running or exited | Up 2 minutes / Exited (0) |
| NAMES | Human-friendly name | mycontainer |

## Container Lifecycle

### Creating and Starting

**Create and start:**
```bash
docker run -it --name mycontainer patrick-first-image bash
```

**What happens:**
1. Creates container from image
2. Starts it immediately
3. Attaches your terminal
4. Runs bash

### Stopping

**From inside container:**
```bash
exit
```

**From another terminal:**
```bash
docker stop mycontainer
```

**Force stop (if unresponsive):**
```bash
docker kill mycontainer
```

### Restarting Existing Container

**Start stopped container:**
```bash
docker start mycontainer
```

**Start and attach (interactive):**
```bash
docker start -ai mycontainer
```

**Flags:**
- `-a`: Attach stdout/stderr
- `-i`: Attach stdin (interactive)

### Removing

**Remove stopped container:**
```bash
docker rm mycontainer
```

**Remove running container (force):**
```bash
docker rm -f mycontainer
```

**Remove multiple:**
```bash
docker rm container1 container2 container3
```

## Container Names

### Auto-Generated Names

When you don't specify `--name`, Docker generates random names:

```bash
docker run -it patrick-first-image bash
```

**Possible names:**
- interesting_bardeen
- busy_pascal
- tender_chatelet
- happy_keldysh
- dreamy_jemison

**Format:** `adjective_scientist_surname`

### Specifying Names

**Good practice:**
```bash
docker run -it --name myproject patrick-first-image bash
```

**Benefits:**
- Easy to remember
- Easy to reference
- Clear purpose

**Requirements:**
- Must be unique
- Can contain: letters, numbers, hyphens, underscores
- Cannot contain spaces

### Name Conflicts

**Error:**
```bash
docker run --name mycontainer patrick-first-image
# Error: name "/mycontainer" already in use
```

**Solutions:**

**Option 1: Remove old container**
```bash
docker rm mycontainer
docker run --name mycontainer patrick-first-image
```

**Option 2: Use different name**
```bash
docker run --name mycontainer2 patrick-first-image
```

**Option 3: Reuse existing container**
```bash
docker start -ai mycontainer
```

## Container Naming vs Hostname

### Two Different Concepts

#### Container Name (`--name`)

**Purpose:** How Docker refers to the container

**Set with:**
```bash
docker run --name mycontainer patrick-first-image
```

**Used in:**
```bash
docker start mycontainer
docker stop mycontainer
docker rm mycontainer
docker logs mycontainer
```

**Scope:** Docker commands on host

#### Hostname (`--hostname`)

**Purpose:** Linux hostname inside the container

**Set with:**
```bash
docker run --hostname mycontainer patrick-first-image
```

**Used by:**
- Shell prompt
- `hostname` command
- Network identification
- Application configuration

**Scope:** Inside container only

### The Difference Illustrated

**Container name only:**
```bash
docker run -it --name mycontainer patrick-first-image bash
```

**You see:**
```
root@d136419d05e2:/app#
         ^^^^^^^^^^^
         Random hostname
```

**Container name + hostname:**
```bash
docker run -it --name mycontainer --hostname mycontainer patrick-first-image bash
```

**You see:**
```
root@mycontainer:/app#
     ^^^^^^^^^^^
     Your chosen hostname
```

### Comparison Table

| Aspect | `--name` | `--hostname` |
|--------|----------|--------------|
| **Scope** | Docker host | Inside container |
| **Purpose** | Docker command reference | Linux system identity |
| **Visible in** | `docker ps` | Shell prompt, `hostname` cmd |
| **Used by** | Docker CLI | Linux programs |
| **Example** | `docker start mycontainer` | `root@mycontainer:/app#` |

### Best Practice

**Make them match:**
```bash
docker run -it \
  --name myproject \
  --hostname myproject \
  patrick-first-image bash
```

**Benefits:**
- Consistency
- Clear identification
- Less confusion

## Managing Multiple Containers

### Scenario: Multiple Containers from Same Image

```bash
# Create multiple containers
docker run --name dev1 patrick-first-image
docker run --name dev2 patrick-first-image
docker run --name test1 patrick-first-image
docker run --name prod patrick-first-image
```

**List them:**
```bash
docker ps -a
```

**Output:**
```
CONTAINER ID   IMAGE                 NAMES
abc123def456   patrick-first-image   dev1
789abc012def   patrick-first-image   dev2
345ghi678jkl   patrick-first-image   test1
901mno234pqr   patrick-first-image   prod
```

**All share the same image but are independent instances.**

## Cleaning Up

### Remove Single Container

```bash
docker rm mycontainer
```

### Remove Multiple Containers

**By name:**
```bash
docker rm container1 container2 container3
```

**By ID:**
```bash
docker rm 8c9b34b33dac 336225de8d12
```

### Remove All Stopped Containers

```bash
docker container prune
```

**Output:**
```
WARNING! This will remove all stopped containers.
Are you sure? [y/N] y

Deleted Containers:
8c9b34b33dac...
336225de8d12...
691867bfe6b6...

Total reclaimed space: 45.2 MB
```

### Remove Container Automatically After Exit

**Run with `--rm` flag:**
```bash
docker run --rm patrick-first-image
```

**After the command exits, container is automatically deleted.**

**Useful for:**
- Quick tests
- One-off commands
- CI/CD pipelines

## Inspecting Containers

### Basic Information

```bash
docker inspect mycontainer
```

**Output:** JSON with complete container details

### Specific Fields

**Get IP address:**
```bash
docker inspect -f '{{.NetworkSettings.IPAddress}}' mycontainer
# 172.17.0.2
```

**Get status:**
```bash
docker inspect -f '{{.State.Status}}' mycontainer
# running
```

**Get image used:**
```bash
docker inspect -f '{{.Config.Image}}' mycontainer
# patrick-first-image
```

## Container Logs

### View Output

```bash
docker logs mycontainer
```

**Example:**
```
Hello Docker is Working
Application started
Processing data...
Done
```

### Follow Logs (Real-time)

```bash
docker logs -f mycontainer
```

**Like `tail -f`, shows new output as it happens.**

### Show Last N Lines

```bash
docker logs --tail 50 mycontainer
```

## Executing Commands in Running Container

### Run Command Without Entering

```bash
docker exec mycontainer ls /app
```

**Output:**
```
main.cpp
app
```

### Enter Running Container

```bash
docker exec -it mycontainer bash
```

**Now you have a shell inside the running container.**

**Difference from `docker run`:**
- `docker run`: Creates new container
- `docker exec`: Runs in existing container

## Container Resource Limits

### Limit Memory

```bash
docker run --memory="512m" patrick-first-image
```

### Limit CPU

```bash
docker run --cpus="1.5" patrick-first-image
```

### Combined

```bash
docker run \
  --memory="1g" \
  --cpus="2" \
  patrick-first-image
```

### Monitor Resource Usage

```bash
docker stats
```

**Output:**
```
CONTAINER ID   NAME          CPU %   MEM USAGE / LIMIT   MEM %
8c9b34b33dac   mycontainer   0.50%   45MB / 512MB       8.79%
```

## Common Workflows

### Development Workflow

**Create once:**
```bash
docker run -it --name dev --hostname dev patrick-first-image bash
```

**Work, exit:**
```bash
exit
```

**Resume later:**
```bash
docker start -ai dev
```

**Clean up when done:**
```bash
docker rm dev
```

### Testing Workflow

**Run test, auto-remove:**
```bash
docker run --rm patrick-first-image pytest
```

### Quick Experiments

**Try something:**
```bash
docker run -it ubuntu bash
# experiment...
exit
```

**Clean up:**
```bash
docker container prune
```

## Practical Example: Managing a Project

```bash
# Start project container
docker run -it \
  --name myproject \
  --hostname myproject \
  -v $(pwd):/workspace \
  patrick-first-image bash

# Work on project...
exit

# Continue next day
docker start -ai myproject

# Check what's running
docker ps

# View logs
docker logs myproject

# Enter for quick command
docker exec myproject ls /workspace

# Finished with project
docker stop myproject
docker rm myproject
```

## Summary: Key Commands

| Task | Command |
|------|---------|
| List running | `docker ps` |
| List all | `docker ps -a` |
| Create & start | `docker run --name NAME IMAGE` |
| Start stopped | `docker start NAME` |
| Start interactive | `docker start -ai NAME` |
| Stop running | `docker stop NAME` |
| Remove stopped | `docker rm NAME` |
| Remove all stopped | `docker container prune` |
| Auto-remove on exit | `docker run --rm IMAGE` |
| View logs | `docker logs NAME` |
| Execute command | `docker exec NAME COMMAND` |
| Enter container | `docker exec -it NAME bash` |
| Inspect | `docker inspect NAME` |
| Monitor resources | `docker stats` |

## Best Practices

1. **Always name important containers**
   ```bash
   docker run --name myproject ...
   ```

2. **Match name and hostname**
   ```bash
   docker run --name dev --hostname dev ...
   ```

3. **Use `--rm` for temporary containers**
   ```bash
   docker run --rm ubuntu echo "test"
   ```

4. **Clean up regularly**
   ```bash
   docker container prune
   ```

5. **Use `docker start` instead of creating new containers**
   ```bash
   docker start -ai existing_container
   # Instead of: docker run --name new_container
   ```

---

**Next:** [09. Best Practices and Workflows](09-best-practices.md) - Professional patterns, volume management, and quick reference.