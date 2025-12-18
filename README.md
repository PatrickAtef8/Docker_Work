# Docker First Trial – From Zero to Understanding

This document explains **everything that happened** in the first Docker experiment, **step by step**, from the very beginning. It is written to build *real understanding*, not just to run commands.

---

## 1. What Problem Does Docker Solve?

Before Docker:

* Applications depend on the operating system
* Each machine may have different library versions
* "It works on my machine" problems are common
* Setting up environments takes time

Docker solves this by:

* Packaging the application **with its environment**
* Making the runtime predictable and repeatable
* Isolating applications from the host OS

Docker does **not** replace Linux. It **uses Linux features** to isolate applications.

---

## 2. What Docker Is (Conceptually)

Docker allows you to run applications inside **containers**.

A container is:

* A process running on your system
* Isolated using Linux kernel features
* Having its own filesystem, network, and process tree

Important clarification:

* Containers are **not virtual machines**
* Containers share the host kernel
* They are lightweight and fast

---

## 3. Key Docker Components

### 3.1 Docker Engine

* A background service (`dockerd`)
* Manages images, containers, networks, volumes
* Talks to the Linux kernel

### 3.2 Docker CLI

* The `docker` command
* Sends requests to Docker Engine

### 3.3 Docker Hub

* A public image registry
* Stores base images (ubuntu, alpine, etc.)

---

## 4. Images vs Containers (Critical Concept)

| Image          | Container                     |
| -------------- | ----------------------------- |
| Blueprint      | Running instance              |
| Read-only      | Writable                      |
| Built once     | Created many times            |
| Stored on disk | Lives in memory while running |

Analogy:

* Image = executable file
* Container = running process

---

## 5. What Happened During Installation

When Docker was installed using `apt`:

* Docker Engine was installed
* Docker CLI was installed
* A system service (`dockerd`) was registered

The Docker service:

* Runs in the background
* Waits for commands from the CLI

Adding the user to the `docker` group allows running Docker commands **without sudo**.

---

## 6. What Happened When Running hello-world

Command:

```
docker run hello-world
```

Internal steps:

1. Docker searched for `hello-world` image locally
2. Image was not found
3. Docker pulled it from Docker Hub
4. Docker created a container from the image
5. The container executed its default command
6. The container printed a message and exited

This proved:

* Docker Engine works
* Image pulling works
* Container execution works

---

## 7. Understanding the Dockerfile

The Dockerfile is a **build recipe** for creating an image.

It describes:

* Base operating system
* Dependencies
* Application files
* Default execution command

### Dockerfile Used

```
FROM ubuntu:22.04

RUN apt update && apt install -y g++

WORKDIR /app

COPY main.cpp .

RUN g++ main.cpp -o app

CMD ["./app"]
```

---

## 8. Dockerfile Instruction Breakdown

### FROM ubuntu:22.04

* Uses Ubuntu 22.04 as the base filesystem
* This is not a full OS installation
* It is a minimal root filesystem

---

### RUN apt update && apt install -y g++

* Executes commands inside the image during build
* Installs the C++ compiler inside the image
* Does not affect the host system

Each RUN creates a **new immutable layer**.

---

### WORKDIR /app

* Sets the working directory inside the image
* Automatically creates the directory if it does not exist
* All following commands use this directory

---

### COPY main.cpp .

* Copies `main.cpp` from the host filesystem
* Places it inside the image at `/app/main.cpp`
* This is a snapshot at build time

---

### RUN g++ main.cpp -o app

* Compiles the program inside the image
* Output binary exists only inside the image

---

### CMD ["./app"]

* Defines the default command for the container
* Executed when `docker run` is called
* Can be overridden at runtime

---

## 9. What Happened During docker build

Command:

```
docker build -t patrick-first-image .
```

Internal process:

1. Docker reads Dockerfile line by line
2. Each instruction creates a layer
3. Layers are cached and reusable
4. Final result is an immutable image

Docker images are **layered filesystems**.

---

## 10. What Happened During docker run

Command:

```
docker run patrick-first-image
```

Internal process:

1. Docker creates a writable container layer
2. Image layers are mounted read-only
3. CMD instruction is executed
4. Program runs and exits
5. Container stops

Stopped containers still exist until removed.

---

## 11. Why Containers Stop

A container runs **as long as its main process runs**.

In this case:

* The program prints text
* The program exits
* The container stops

This is expected behavior.

---

## 12. Why Docker Is Important for DevOps

Docker enables:

* Consistent environments
* Automated CI/CD pipelines
* Reproducible builds
* Easy deployment

Docker is the foundation for:

* Kubernetes
* Cloud-native systems
* Microservices

---

## 13. Docker vs Virtual Machines

| Docker               | Virtual Machine     |
| -------------------- | ------------------- |
| Shares host kernel   | Own kernel          |
| Lightweight          | Heavy               |
| Starts in seconds    | Starts in minutes   |
| Lower resource usage | High resource usage |

Docker focuses on **process isolation**, not hardware virtualization.

---

## 14. Key Takeaways

* Docker packages applications with their environment
* Images are immutable blueprints
* Containers are running instances
* Dockerfiles define reproducible builds
* Docker is essential for modern DevOps

---

## 15. Understanding the Image Name (patrick-first-image)

### What is `patrick-first-image`?

`patrick-first-image` is simply the **name (tag)** given to a Docker image.

It is **not a file**, **not source code**, and **not a container**. It is a label that refers to a built Docker image stored inside Docker’s internal storage.

The name was created by this command:

```
docker build -t patrick-first-image .
```

Here, `-t` means **tag** (name), and `patrick-first-image` is an arbitrary name chosen by the user.

---

### Where did `patrick-first-image` come from?

It was produced as the **output of the Docker build process**.

The full chain is:

```
main.cpp  --> used by --> Dockerfile
Dockerfile --> used by --> docker build
Docker build --> produces --> Docker IMAGE
Docker IMAGE --> named --> patrick-first-image
```

So:

* `main.cpp` is an input
* `Dockerfile` is the recipe
* `docker build` is the build tool
* `patrick-first-image` is the final product

---

### How do `main.cpp` and `Dockerfile` know each other?

They are connected through this line in the Dockerfile:

```
COPY main.cpp .
```

This instruction tells Docker:

* Take `main.cpp` from the current directory on the host
* Copy it into the image filesystem during build time

Because `docker build` was run in the same directory (`.`), Docker could see both `Dockerfile` and `main.cpp`.

---

### Is `patrick-first-image` related to `main.cpp`?

Yes, **indirectly**.

* `main.cpp` is source code
* The Dockerfile describes how to compile it
* The resulting compiled program is embedded inside the image

So the image contains:

* A minimal Ubuntu filesystem
* The C++ compiler (g++)
* The compiled binary (`app`)

---

### Is `patrick-first-image` a normal file?

No.

* It does not appear in `ls`
* It does not exist in your project directory
* It is stored internally by Docker

You can see it using:

```
docker images
```

---

### What happens when `docker run patrick-first-image` is executed?

Docker performs the following steps:

1. Finds the image named `patrick-first-image`
2. Creates a new container from that image
3. Adds a writable layer on top of the image
4. Executes the default command defined in the Dockerfile

The default command was defined as:

```
CMD ["./app"]
```

This means the compiled program is executed automatically when the container starts.

When the program finishes execution, the container stops.

---

### Analogy with C/C++ Compilation

| C/C++ Concept       | Docker Concept          |
| ------------------- | ----------------------- |
| `main.c`            | `main.cpp`              |
| `gcc main.c -o app` | `docker build`          |
| `app` executable    | Docker image            |
| `./app`             | `docker run image-name` |

A Docker image is like a **compiled artifact**, but instead of compiling only code, Docker compiles an entire runtime environment.

---

### Why the Image Is Not Named After main.cpp

The image is not named `main.cpp` because it contains much more than just the source code:

* Operating system files
* Installed dependencies
* Compiled binaries
* Runtime configuration

Images are named like **products or services**, not source files.

---

### Final Mental Model

* Dockerfile explains **how** to build
* `docker build` creates **what** to run
* Image is the immutable result
* Image name is just a label
* `docker run` turns an image into a running container

---

## 16. Next Learning Steps

Recommended progression:

1. Volumes (persistent data)
2. Networking and ports
3. Docker Compose
4. Multi-stage builds
5. CI/CD integration

This document represents the foundation of Docker understanding.

Recommended progression:

1. Volumes (persistent data)
2. Networking and ports
3. Docker Compose
4. Multi-stage builds
5. CI/CD integration

This document represents the foundation of Docker understanding.
