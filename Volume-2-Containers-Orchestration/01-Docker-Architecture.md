# Docker Architecture

> AWS DevOps Playbook
>
> Volume 2 – Containers & Orchestration
>
> Chapter 01

---

# Chapter Objective

After completing this chapter, you should be able to:

- Explain why Docker was created.
- Explain the complete Docker architecture.
- Understand Docker Engine.
- Understand Images, Containers, Layers.
- Explain OverlayFS.
- Explain the Docker lifecycle.
- Understand namespaces and cgroups.
- Answer Docker architecture interview questions.
- Troubleshoot common Docker issues.

---

# 1. Why Docker?

Before Docker, applications were deployed directly on servers.

Example:

```
Server

├── Java 8
├── Python 2
├── NodeJS 10
├── MySQL
└── Apache
```

Now another application needs:

```
Java 17
Python 3.11
NodeJS 22
```

Problems:

- Dependency conflicts
- Library version conflicts
- Difficult upgrades
- Difficult rollbacks
- "Works on my machine" syndrome
- Resource wastage

Virtual Machines solved isolation but introduced a new problem.

---

# 2. Virtual Machines vs Docker

## Virtual Machine Architecture

```
Application A
Guest OS
------------------

Application B
Guest OS
------------------

Application C
Guest OS
------------------

Hypervisor
------------------

Host OS

------------------

Hardware
```

Each VM contains a full operating system.

Advantages

- Strong isolation
- Different operating systems

Disadvantages

- Heavy
- Slow boot
- Large storage
- High RAM usage

---

## Docker Architecture

```
Application A

↓

Container

------------------

Application B

↓

Container

------------------

Application C

↓

Container

------------------

Docker Engine

------------------

Host OS

------------------

Hardware
```

Containers share the host kernel.

Advantages

- Lightweight
- Fast startup
- Less memory
- Smaller images
- Better density

---

# 3. What is Docker?

Docker is a container platform that packages an application along with all its dependencies into a portable image.

Build once.

Run anywhere.

---

# 4. Docker Architecture

```
Developer

↓

Docker CLI

↓

Docker Daemon

↓

Docker Engine

↓

Images

↓

Containers

↓

Linux Kernel

↓

Hardware
```

---

# 5. Components of Docker

Docker consists of:

```
Docker CLI

Docker Daemon

Docker Engine

Images

Containers

Registry

Volumes

Networks
```

Let's study each one.

---

# 6. Docker CLI

The Docker CLI is what users interact with.

Examples:

```bash
docker build
docker run
docker ps
docker images
docker logs
docker exec
docker stop
```

The CLI itself does not create containers.

It simply sends API requests to the Docker Daemon.

---

# 7. Docker Daemon (dockerd)

⭐⭐⭐⭐⭐

The Docker Daemon is the brain of Docker.

It:

- Builds images
- Starts containers
- Stops containers
- Creates networks
- Creates volumes
- Pulls images
- Pushes images

Flow:

```
Docker CLI

↓

Docker Daemon

↓

Docker Engine
```

Linux Service:

```bash
systemctl status docker
```

If the daemon is stopped, Docker commands will fail.

---

# 8. Docker Engine

Docker Engine consists of:

- Docker Daemon
- REST API
- Docker CLI

It is responsible for managing the entire container lifecycle.

---

# 9. Docker Registry

Where images are stored.

Examples:

- Docker Hub
- Amazon ECR
- Azure Container Registry
- Google Artifact Registry
- GitHub Container Registry
- Nexus Registry

Example:

```bash
docker pull nginx
```

Docker contacts the registry and downloads the image.

---

# 10. Docker Image

⭐⭐⭐⭐⭐

An image is a **read-only template** used to create containers.

Think of it as a blueprint.

Example:

```
Ubuntu Image

↓

Container 1

Container 2

Container 3
```

An image contains:

- Application
- Runtime
- Libraries
- Dependencies
- Configuration
- Metadata

Images are immutable.

---

# 11. Docker Container

A container is a **running instance of an image**.

Example:

```
nginx Image

↓

Container A

Container B

Container C
```

Each container has:

- Own process space
- Own filesystem view
- Own network stack
- Own hostname

Containers share the host kernel.

---

# 12. Image Layers

⭐⭐⭐⭐⭐

Docker images are built in layers.

Example Dockerfile:

```dockerfile
FROM ubuntu:24.04

RUN apt-get update

RUN apt-get install nginx

COPY app /app

CMD ["nginx","-g","daemon off;"]
```

Layers:

```
Layer 5 CMD

↓

Layer 4 COPY

↓

Layer 3 Install nginx

↓

Layer 2 apt update

↓

Layer 1 Ubuntu Base
```

Each instruction creates a new immutable layer.

Benefits:

- Faster builds
- Layer reuse
- Smaller downloads
- Efficient caching

---

# 13. Copy-on-Write

Images are read-only.

Containers add one writable layer.

```
Image

(Read Only)

↓

Writable Layer

(Container)
```

Changes made inside the container exist only in the writable layer.

When the container is deleted, that layer is removed unless data is stored externally.

---

# 14. OverlayFS

⭐⭐⭐⭐⭐

Docker commonly uses OverlayFS as a storage driver.

OverlayFS merges multiple read-only image layers with one writable container layer.

```
Layer 5

↓

Layer 4

↓

Layer 3

↓

Layer 2

↓

Layer 1

↓

Writable Layer
```

The container sees a single unified filesystem.

Advantages:

- Fast
- Space efficient
- Layer sharing
- Copy-on-write support

---

# 15. Docker Lifecycle

```
Dockerfile

↓

docker build

↓

Image

↓

docker run

↓

Container

↓

docker stop

↓

Stopped Container

↓

docker start

↓

Running Container

↓

docker rm

↓

Deleted Container
```

---

# 16. Docker Networking

Every container gets a network interface.

Default network:

```
bridge
```

We'll study networking in the next chapter.

---

# 17. Docker Volumes

Containers are ephemeral.

If you store data inside the writable layer:

```
Container Deleted

↓

Data Deleted
```

Volumes solve this problem.

We'll cover them in detail later.

---

# 18. Namespaces

⭐⭐⭐⭐⭐

Namespaces provide isolation.

Types:

- PID Namespace
- Network Namespace
- Mount Namespace
- UTS Namespace
- IPC Namespace
- User Namespace

Example:

```
Container A

PID 1

Container B

PID 1
```

Each container believes it owns the system.

---

# 19. cgroups

⭐⭐⭐⭐⭐

Control Groups limit resource usage.

Example:

```
CPU

2 vCPU

Memory

2 GB
```

Even if the host has more resources, the container cannot exceed these limits.

This prevents one container from affecting others.

---

# 20. Docker Build Process

```
Dockerfile

↓

Read Instructions

↓

Create Layers

↓

Image

↓

Store Locally

↓

Push to Registry (optional)
```

---

# 21. Docker Run Process

```
docker run nginx

↓

Check Local Image

↓

Image Found?

↓

No

↓

Pull Image

↓

Create Writable Layer

↓

Create Namespaces

↓

Apply cgroups

↓

Start Container
```

---

# 22. Production Architecture

```
Developer

↓

GitHub

↓

Jenkins

↓

docker build

↓

Docker Image

↓

Amazon ECR

↓

Amazon ECS / Amazon EKS

↓

Application
```

---

# 23. Common Docker Commands

```bash
docker images
docker ps
docker ps -a
docker run nginx
docker stop <container>
docker start <container>
docker restart <container>
docker logs <container>
docker exec -it <container> bash
docker inspect <container>
docker rm <container>
docker rmi <image>
```

---

# 24. Production Scenarios

## Scenario 1

Problem

Container exits immediately after starting.

Possible Causes:

- Main process exited
- Incorrect CMD
- Missing ENTRYPOINT
- Application crash

Investigation:

```bash
docker logs <container>
docker inspect <container>
```

---

## Scenario 2

Problem

Container restarts continuously.

Possible Causes:

- Application crash
- Out of Memory
- Failed health check
- Restart policy

Commands:

```bash
docker logs
docker inspect
docker events
```

---

## Scenario 3

Problem

Application works locally but not inside the container.

Possible Causes:

- Wrong port exposed
- Missing environment variables
- Missing dependency
- Wrong working directory

---

## Scenario 4

Problem

Container cannot reach the database.

Check:

- Docker network
- DNS resolution
- Firewall rules
- Database endpoint
- Security groups (if external)

---

# 25. Best Practices

- Use official base images.
- Keep images small.
- Use multi-stage builds.
- Run containers as a non-root user.
- Pin image versions instead of using `latest`.
- Use health checks.
- Store secrets outside images.
- Keep one main process per container.

---

# 26. Common Mistakes

❌ Using `latest` tag in production.

Use versioned tags instead.

---

❌ Storing secrets inside Dockerfile.

Use environment variables or a secrets manager.

---

❌ Running multiple unrelated services in one container.

Prefer one primary application process per container.

---

❌ Ignoring image size.

Large images increase deployment time and attack surface.

---

# 27. Interview Questions

## Question 1

What is Docker?

### Perfect Answer

Docker is a containerization platform that packages an application and its dependencies into a portable image. Containers share the host operating system kernel, making them lightweight and fast compared to virtual machines.

---

## Question 2

Difference between Docker Image and Container?

### Perfect Answer

A Docker image is a read-only template containing the application, dependencies, and configuration. A container is a running instance of that image with its own writable layer.

---

## Question 3

What is Docker Daemon?

### Perfect Answer

The Docker Daemon (`dockerd`) is the background service responsible for building images, running containers, managing networks, volumes, and communicating with registries.

---

## Question 4

Why are Docker images built in layers?

### Perfect Answer

Layers enable caching, efficient storage, and reuse. If only one instruction changes, Docker reuses the unchanged layers, making builds and downloads faster.

---

## Question 5

What are namespaces and cgroups?

### Perfect Answer

Namespaces provide isolation for resources such as processes, networking, and filesystems. cgroups control resource usage by limiting CPU, memory, and other resources available to a container.

---

# 28. Amazon Cross Questions

### Question

Does each Docker container have its own kernel?

### Perfect Answer

No. Containers share the host operating system kernel. Unlike virtual machines, they do not include a separate guest operating system.

---

### Question

Why do containers start faster than VMs?

### Perfect Answer

Containers reuse the host kernel and do not need to boot a full operating system, so startup is much faster.

---

### Question

If two containers are created from the same image, are they sharing the writable layer?

### Perfect Answer

No. Each container gets its own writable layer while sharing the read-only image layers.

---

### Question

Can one Docker Daemon manage multiple containers?

### Perfect Answer

Yes. A single Docker Daemon can manage many images, containers, networks, and volumes simultaneously.

---

# 29. One-Page Revision

```
Developer
     │
     ▼
Docker CLI
     │
     ▼
Docker Daemon
     │
     ▼
Docker Engine
     │
     ▼
Docker Image
     │
     ▼
Container
     │
     ▼
Linux Kernel
     │
     ▼
Hardware
```

Remember:

- Docker CLI
- Docker Daemon
- Docker Engine
- Registry
- Image
- Container
- Layers
- OverlayFS
- Copy-on-Write
- Namespaces
- cgroups

---

# 30. Think Like a Production Engineer

Don't think:

> "Docker runs containers."

Think:

> "Docker provides a complete runtime that packages applications, isolates workloads, optimizes resource usage, and enables consistent deployments across environments."

Whenever someone says:

```
Container Failed

↓

Application Crash

↓

High Memory

↓

Image Pull Error
```

Your troubleshooting flow should be:

```
Container Status

↓

docker logs

↓

docker inspect

↓

Resource Limits

↓

Network

↓

Application
```

---

# Key Takeaways

Docker is much more than a command-line tool. It is a complete container platform built around images, containers, layered filesystems, namespaces, and cgroups. Understanding these internals is essential for debugging production issues and succeeding in DevOps interviews.

# End of Chapter 01
