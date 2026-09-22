# Docker Interview Questions and Answers

> Simple language, but with the important concepts needed for a fresher DevOps interview.

---

## 1. What is Docker?

**Answer:**

Docker is a **containerization platform**.

It helps us package an application together with everything it needs, such as libraries and dependencies, into a **Docker image**.

That image can then be used to create and run a **container**.

Docker also provides features like:

- Container networking
- Volumes
- Image building
- Container management
- Image registries

### Remember

```text
Dockerfile → Image → Container
```

- **Dockerfile** = instructions
- **Image** = packaged application
- **Container** = running instance of an image
- **Registry** = stores and distributes images

---

## 2. How are Containers different from VMs?

**Answer:**

The main difference is how they use the operating system.

A **VM (Virtual Machine)** runs a complete guest operating system.

A **container** shares the host machine's OS kernel and only contains the application and the user-space files it needs.

### VM

```text
Hardware
   ↓
Hypervisor
   ↓
Guest OS
   ↓
Application
```

### Container

```text
Hardware
   ↓
Host OS
   ↓
Container Engine
   ↓
Container
   ↓
Application
```

Because containers do not need a separate OS kernel, they are usually:

- Smaller
- Faster to start
- Less resource-heavy

VMs usually provide a stronger isolation boundary because each VM has its own guest OS.

---

## 3. What is the Docker lifecycle?

**Answer:**

A simple Docker lifecycle looks like this:

```text
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
Start / Stop / Restart
    ↓
Remove
```

We can also push an image to a registry:

```text
Image
  ↓
docker push
  ↓
Registry
  ↓
docker pull
  ↓
Another machine
```

### Common commands

```bash
docker build
docker run
docker start
docker stop
docker restart
docker rm
docker push
docker pull
```

---

## 4. What are the main Docker components?

**Answer:**

Important Docker components are:

### Docker CLI

The command-line tool we use to talk to Docker.

Example:

```bash
docker ps
docker run nginx
```

### Docker Daemon

A background service that manages Docker objects such as:

- Containers
- Images
- Networks
- Volumes

### Docker Image

A read-only package containing the application and its dependencies.

### Docker Container

A running instance of a Docker image.

### Docker Registry

A place used to store and distribute Docker images.

Examples:

- Docker Hub
- Amazon ECR
- GitHub Container Registry

### Docker Network

Allows containers to communicate with each other or with external systems.

### Docker Volume

Used for persistent data.

---

## 5. What is the difference between COPY and ADD?

**Answer:**

Both are used in a Dockerfile to put files into an image.

### COPY

Used mainly to copy files and folders from the build context.

```dockerfile
COPY package.json /app/
```

### ADD

Can do more than COPY. For example, it can:

- Copy files
- Extract local tar archives
- Handle remote URLs in supported Dockerfile usage

### Interview answer

**COPY is simpler and more predictable, so we normally prefer COPY unless we specifically need an ADD feature.**

---

## 6. What is the difference between CMD and ENTRYPOINT?

**Answer:**

Both are used to define what runs when a container starts.

### CMD

Provides the **default command or arguments**.

Example:

```dockerfile
CMD ["node", "server.js"]
```

CMD can easily be overridden when running the container.

### ENTRYPOINT

Defines the **main executable** of the container.

Example:

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

This runs:

```bash
python app.py
```

If we run:

```bash
docker run myimage other.py
```

it can run:

```bash
python other.py
```

### Easy way to remember

**ENTRYPOINT = main program**

**CMD = default arguments/command**

They can also be used together.

---

## 7. What are the networking types in Docker?

**Answer:**

Common Docker network drivers are:

1. **Bridge**
2. **Host**
3. **None**
4. **Overlay**
5. **Macvlan**

### Bridge

Used for communication between containers on the same Docker host.

It is the **default network driver** for containers on a normal Docker Engine installation.

### Host

The container uses the host's network namespace.

### None

The container has no normal network connectivity.

### Overlay

Used for communication across multiple Docker hosts, commonly in orchestrated environments.

### Macvlan

Gives containers their own MAC address and can make them appear as physical devices on the network.

---

## 8. How can you isolate networking between containers?

**Answer:**

We can create separate Docker networks and only connect the required containers to each network.

Example:

```bash
docker network create backend
```

Then:

```bash
docker run --network backend app1
docker run --network backend app2
```

`app1` and `app2` can communicate through the `backend` network.

Another container on a different network will not automatically be part of that network.

We should also expose only the ports that are actually needed.

---

## 9. What is a multi-stage build?

**Answer:**

A multi-stage build uses **more than one stage** in a Dockerfile.

The first stage is used to **build the application**.

It may contain:

- Compilers
- Build tools
- Development dependencies

The final stage contains only what is needed to **run the application**.

Example:

```dockerfile
FROM node:22 AS builder

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build


FROM node:22-alpine AS runner

WORKDIR /app

COPY --from=builder /app/dist ./dist

CMD ["node", "dist/server.js"]
```

### Why use it?

It can:

- Reduce image size
- Remove unnecessary build tools
- Reduce the attack surface
- Make the final image cleaner

### Easy idea

```text
Big Builder Image
       ↓
    Build App
       ↓
Copy only required files
       ↓
Small Runtime Image
```

---

## 10. What are Distroless images?

**Answer:**

Distroless images are very minimal container images.

They contain the **runtime components needed by the application**, but usually do not contain many normal OS utilities such as:

- Shells
- Package managers
- Debugging tools

This makes the image:

- Smaller
- Cleaner
- Less exposed to unnecessary vulnerabilities

They are often used as the final stage of a multi-stage build.

### Important

Do not say:

> "Distroless images contain nothing."

Instead say:

> **They contain only the minimum runtime components needed by the application.**

### Disadvantage

Debugging can be harder because tools such as a shell may not be available.

---

## 11. What are some real-world challenges with Docker?

**Answer:**

Some common challenges are:

### 1. Resource usage

Too many containers on one machine can use too much:

- CPU
- RAM
- Disk

This can cause slow performance or crashes.

### 2. Security

Running containers with too many privileges or as root can increase security risk.

### 3. Large images

Large images take more:

- Disk space
- Network bandwidth
- Time to download

Multi-stage builds and minimal images can help.

### 4. Networking

As the number of containers increases, managing communication between them can become more complex.

### 5. Storage

Containers are usually temporary, so persistent data should be stored using volumes or external storage.

### 6. Docker daemon

Traditional Docker Engine uses a daemon to manage containers and other Docker resources. Problems with the daemon can affect Docker management and operations.

Tools such as Podman use a daemonless architecture and can also support rootless containers.

---

## 12. What steps would you take to secure containers?

**Answer:**

I would follow these steps:

### 1. Use small and trusted images

Use minimal images such as slim or distroless images when appropriate.

### 2. Do not run as root

Run the application as a non-root user.

Example:

```dockerfile
USER node
```

### 3. Scan images

Use security scanning tools such as:

- Trivy
- Snyk
- Docker Scout

These tools can find known vulnerabilities.

### 4. Do not put secrets inside images

Avoid things like:

```dockerfile
ENV DATABASE_PASSWORD=12345
```

Use proper secret management or runtime environment configuration.

### 5. Use network isolation

Use separate Docker networks and expose only the ports that are required.

### 6. Keep images and dependencies updated

Regularly update:

- Base images
- Application dependencies
- Security patches

### 7. Use trusted images

Use official or trusted images and control the versions you deploy.

---

# Quick Revision Sheet

## Docker Basics

```text
Docker = Containerization platform

Dockerfile = Instructions
Image = Packaged application
Container = Running image
Registry = Stores/distributes images
Daemon = Manages Docker resources
CLI = Commands used to interact with Docker
Volume = Persistent data
Network = Container communication
```

## Container vs VM

```text
VM
→ Complete guest OS
→ More resources
→ Stronger isolation
→ Usually slower startup

Container
→ Shares host kernel
→ Lightweight
→ Fast startup
→ Uses fewer resources
```

## Docker Build Flow

```text
Dockerfile
    ↓
docker build
    ↓
Image
    ↓
docker run
    ↓
Container
```

## Networking

```text
Bridge   → Default, same Docker host
Host     → Uses host network
None     → No normal network
Overlay  → Multiple Docker hosts
Macvlan  → Container gets its own MAC address
```

## CMD vs ENTRYPOINT

```text
ENTRYPOINT → Main executable
CMD        → Default command/arguments
```

## Multi-stage Build

```text
Builder Stage
    ↓
Build application
    ↓
Copy required artifacts
    ↓
Small Runtime Stage
```

## Container Security

```text
Minimal image
      +
Non-root user
      +
Image scanning
      +
Network isolation
      +
No secrets in image
      +
Updated dependencies
      =
More secure container
```

---

# Interview Tip

As a fresher, you do **not** need to explain every Docker topic for 5 minutes.

Give a clear answer in **30-60 seconds**, then wait.

If the interviewer asks a follow-up, go deeper.

A good pattern is:

```text
Definition
   ↓
How it works
   ↓
Why we use it
   ↓
Small example
```

For example:

> **"A multi-stage build allows us to use separate build and runtime stages. We can keep compilers and development dependencies in the builder stage and copy only the required application artifacts into a smaller runtime image. This reduces image size and attack surface."**

That is short, technically correct, and enough to start the discussion.
