# Multi-Stage Docker Build Handbook

## 1) What is a basic Docker build?

A normal Docker build creates an image from a Dockerfile. The image usually contains:
- the OS base image
- application source code
- runtime dependencies
- build tools
- package managers
- sometimes debugging tools

This is simple, but it often creates large images.

### Basic Dockerfile example

```dockerfile
# Basic Dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

### Commands

```bash
docker build -t myapp:latest .
docker run -d -p 3000:3000 --name myapp myapp:latest
```

### Why this is basic

This image contains:
- Node runtime
- npm
- build tools
- source files
- package manager

That is fine for development, but not ideal for production.

---

## 2) What is a multi-stage Docker build?

A multi-stage build uses more than one `FROM` instruction in a single Dockerfile.

The idea is:
- Stage 1: install dependencies and compile/build the app
- Stage 2: take only the necessary final files and copy them into a smaller runtime image

This results in:
- smaller final image
- less attack surface
- faster deployments
- cleaner production image

### Simple idea

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/server.js"]
```

### Why it is useful

The final image does not contain:
- source code
- dev dependencies
- package manager
- compiler tools
- build cache

Only the app runtime files remain.

---

## 3) Two-stage Docker build (very easy example)

This is the most common and easiest example to understand.

### Example: Go application

```dockerfile
# Stage 1: Builder
FROM golang:1.22 AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build -o app .

# Stage 2: Runtime
FROM gcr.io/distroless/base-debian12
WORKDIR /app
COPY --from=builder /app/app /app/app
EXPOSE 8080
ENTRYPOINT ["/app/app"]
```

### Build command

```bash
docker build -t goapp:2stage .
```

### Run command

```bash
docker run -d -p 8080:8080 --name goapp goapp:2stage
```

### Step-by-step explanation

1. `FROM golang:1.22 AS builder`
   - starts with a Go build environment
   - includes compiler, Go toolchain, and necessary libraries

2. `COPY . .`
   - copies the project code into the build stage

3. `RUN go build -o app .`
   - compiles the Go code into a binary named `app`

4. `FROM gcr.io/distroless/base-debian12`
   - starts a very minimal runtime image
   - no shell, no package manager, no extra OS tools

5. `COPY --from=builder /app/app /app/app`
   - copies only the compiled binary to the final image

6. `ENTRYPOINT ["/app/app"]`
   - runs the app directly

This final image is much smaller and more secure.

---

## 4) Three-stage Docker build (simple and clear)

A 3-stage build is useful when you want to separate:
- dependency installation
- app build
- runtime

### Example: Node.js application

```dockerfile
# Stage 1: Install dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Stage 2: Build the application
FROM node:20-alpine AS build
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Stage 3: Production runtime
FROM gcr.io/distroless/nodejs20-debian12
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/package.json ./package.json
COPY --from=build /app/node_modules ./node_modules
EXPOSE 3000
CMD ["dist/server.js"]
```

### Build command

```bash
docker build -t mynodeapp:3stage .
```

### Run command

```bash
docker run -d -p 3000:3000 --name mynodeapp mynodeapp:3stage
```

### Step-by-step explanation

- Stage 1 installs dependencies only
- Stage 2 builds the application using those dependencies
- Stage 3 creates a minimal final runtime image with only required files

This helps keep the final container lean while still building cleanly.

---

## 5) What are distroless images?

Distroless images are minimal container images that contain only the application and its runtime dependencies.

They usually do not contain:
- package manager like `apt`, `yum`, or `apk`
- shell like `bash` or `sh`
- debugging utilities
- OS package tools
- unneeded libraries

### Distroless image idea

Instead of a full Linux base image, you get a minimal runtime environment.

Example images:
- `gcr.io/distroless/base-debian12`
- `gcr.io/distroless/static-debian12`
- `gcr.io/distroless/python3-debian12`
- `gcr.io/distroless/nodejs20-debian12`

### Where to find them

Official project:
- https://github.com/GoogleContainerTools/distroless

Official README and usage details:
- https://github.com/GoogleContainerTools/distroless/blob/main/README.md

Common registry references:
- https://console.cloud.google.com/gcr/images/distroless/GLOBAL

Example image pulls:

```bash
docker pull gcr.io/distroless/base-debian12
docker pull gcr.io/distroless/static-debian12
docker pull gcr.io/distroless/nodejs20-debian12
```

---

## 6) Why are distroless images more secure than normal images?

This is a very common interview question.

### Answer in simple words

Normal images often include extra system tools, shells, and package managers. This increases the attack surface.

A distroless image reduces the number of components inside the container. That means:
- fewer vulnerabilities
- fewer packages to patch
- fewer entry points for attackers
- smaller attack surface
- better production security posture

### Example comparison

Normal image may contain:
- OS packages
- `bash`
- `apt`
- `curl`
- shell utilities
- debug tools

Distroless image may contain only:
- your compiled binary
- required runtime libraries
- nothing extra

This makes it harder for an attacker to run malicious commands or explore the system.

### Interview-style answer

> Distroless images are more secure because they remove unnecessary OS components, package managers, and shells. Since there are fewer tools and fewer packages installed, the attack surface is reduced and the container is harder to exploit. This makes distroless images a better choice for production-grade security.

---

## 7) Why multi-stage builds matter in production

Multi-stage builds are important because they help you:
- reduce final image size
- avoid shipping build tools
- keep runtime minimal and secure
- improve performance and deployment speed
- follow best practices for production containers

A large Docker image is not only slower to push/pull, but also more vulnerable because it contains more components.

---

## 8) Easy command summary

### Basic single-stage build

```bash
docker build -t app:latest .
docker run -d -p 8080:8080 --name app app:latest
```

### Two-stage build

```bash
docker build -t app:2stage .
docker run -d -p 8080:8080 --name app app:2stage
```

### Three-stage build

```bash
docker build -t app:3stage .
docker run -d -p 8080:8080 --name app app:3stage
```

---

## 9) Important points to remember

- A basic Docker build is a simple image build from one stage.
- A multi-stage build uses multiple `FROM` commands.
- The first stage is usually for building or compiling.
- The final stage is usually for running the app.
- Distroless images contain almost no OS utilities.
- Smaller images are usually more secure and efficient.
- Production images should avoid shipping build dependencies.

---

## 10) Short interview questions and answers

### Q1: What is a multi-stage Docker build?
A: It is a Dockerfile with multiple `FROM` stages, where one stage builds the app and another stage copies only the required runtime files.

### Q2: Why use multi-stage builds?
A: To reduce image size, improve security, and avoid including build tools in the final image.

### Q3: What is a distroless image?
A: It is a minimal container image without package managers, shells, or extra OS utilities.

### Q4: Why are distroless images more secure?
A: Because they have a smaller attack surface and fewer installed packages/tools to exploit.

### Q5: What is the main benefit of multi-stage builds?
A: Final image is lean, secure, and production-ready.

---

## 11) Final takeaway

Use a multi-stage Docker build when you want a clean and secure production image.

The basic pattern is:

```dockerfile
FROM build-image AS builder
# install dependencies and compile

FROM distroless-or-minimal-runtime
# copy only the final app artifact
```

This is the standard modern approach for production containerization.

---

## 12) Quick cheat sheet

- Single-stage build = build and run in one image
- Multi-stage build = build in one stage, run in another
- Distroless = minimal runtime, small and secure
- Best practice = compile in builder stage, run in final minimal stage

```bash
docker images
docker ps
docker build -t app .
docker run -p 8080:8080 app
```

This is the practical mindset used in DevOps and cloud-native projects.
