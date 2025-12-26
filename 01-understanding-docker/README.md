# Understanding Docker: Complete Guide

A comprehensive, step-by-step guide from zero to mastery. Perfect for developers, especially those working with embedded systems, cross-compilation, and SDKs.

## About This Guide

This guide takes you through Docker systematically, clearing up common misconceptions and building deep understanding. Each chapter is standalone but builds on previous concepts.


## Learning Path

### Part 1: Foundation (20 minutes)

**[01. The Problem Docker Solves](01-the-problem.md)** - 5 min
- The "Works on my machine" syndrome
- Why sending binaries fails
- Why sending source code fails
- The Docker solution

**[02. What Docker Actually Is](02-what-is-docker.md)** - 5 min
- One-sentence definition
- What Docker is NOT
- What Docker IS
- Core mental model

**[03. Docker vs Virtual Machines](03-docker-vs-vms.md)** - 10 min
- Why containers are NOT VMs
- Architecture comparison
- Kernel sharing explained
- Performance differences

### Part 2: Docker in Context (15 minutes)

**[04. Docker vs SDK and Cross-Compilation](04-docker-vs-sdk.md)** - 10 min
- What SDKs actually do
- What Docker does differently
- Common misconceptions about ARM/Raspberry Pi
- Professional embedded workflow

**[05. Images and Containers](05-images-containers.md)** - 5 min
- The fundamental distinction
- Perfect analogies
- One image, many containers

### Part 3: Deep Dive (25 minutes)

**[06. What Happens Behind the Scenes](06-behind-scenes.md)** - 15 min
- Step-by-step build process
- What happens during `docker run`
- Isolation mechanisms
- Where Docker stores files

**[07. Complete Practical Example](07-practical-example.md)** - 10 min
- Building a C++ application
- Exploring the container filesystem
- Why it works regardless of host

### Part 4: Daily Usage (20 minutes)

**[08. Container Management](08-management.md)** - 10 min
- Listing and inspecting containers
- Container lifecycle
- Naming vs hostname
- Cleaning up

**[09. Best Practices and Workflows](09-best-practices.md)** - 10 min
- Professional patterns
- Reusing containers
- Volume management
- Docker + SDK for embedded
- Quick reference commands

## Quick Start

If you're in a hurry:
1. Read [01-the-problem.md](01-the-problem.md) - understand WHY
2. Read [02-what-is-docker.md](02-what-is-docker.md) - understand WHAT
3. Read [07-practical-example.md](07-practical-example.md) - see it WORK
4. Use [09-best-practices.md](09-best-practices.md) - as a reference

## Prerequisites

- Basic Linux command line knowledge
- Basic understanding of compilation (gcc, make)
- No prior Docker experience needed

## Who This Guide Is For

- Developers frustrated with "works on my machine" problems
- Embedded engineers using Yocto/SDK wondering about Docker
- Anyone confused about Docker vs VMs
- Teams wanting reproducible builds

## What Makes This Guide Different

Unlike typical Docker tutorials, this guide:
- Addresses real confusion (Docker vs VM, Docker vs SDK, Docker + ARM)
- Explains what happens behind the scenes
- Focuses on understanding, not just commands
- Includes embedded/cross-compilation context
- Clears up the "Docker makes my PC into Raspberry Pi" misconception

## Navigation Tips

- Each chapter is self-contained
- Read in order for complete understanding
- Jump to specific chapters for quick answers
- Code examples are in every chapter
- Tables and comparisons throughout

## After This Guide

You will understand:
- What Docker actually does (and doesn't do)
- Why containers are different from VMs
- How Docker fits with SDKs and cross-compilation
- When to use Docker (and when not to)
- How to use Docker professionally

Start with: **[01. The Problem Docker Solves](01-the-problem.md)**