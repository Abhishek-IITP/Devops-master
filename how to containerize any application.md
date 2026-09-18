# How to Containerize Any Application

Containerization packages an application, its dependencies, and its runtime configuration into an isolated image. The same image can run on a laptop, CI server, EC2 instance, or Kubernetes cluster.

## 1. Basic Workflow

1. Identify the application start command and the port on which it listens.
2. Create a `Dockerfile` in the project root.
3. Add a `.dockerignore` file.
4. Build the image:

	```bash
	docker build -t my-app:1.0 .
	```

5. Run it and map the host port to the container port:

	```bash
	docker run -d --name my-app -p 8080:8080 my-app:1.0
	```

6. Test `http://localhost:8080` and inspect failures with `docker logs -f my-app`.
7. Tag and push the image to Docker Hub, Amazon ECR, or GitHub Container Registry.
8. Pull and run the same image on the target server.

### Port Rule

The application must listen on `0.0.0.0` inside the container, not only on `localhost`.

```text
-p HOST_PORT:CONTAINER_PORT
```

For example, `-p 80:8080` sends host port `80` to container port `8080`. `EXPOSE` documents a port; it does not publish it or open an AWS firewall rule.

## 2. Recommended `.dockerignore`

```text
.git
.github
.env
.venv
venv
node_modules
__pycache__
*.pyc
target
dist
build
coverage
*.log
Dockerfile*
docker-compose*.yml
```

Never copy secrets into an image. Pass them at runtime or use a secrets manager.

## 3. Dockerfile Examples

### Python

Assumes `requirements.txt` and `app.py`, listening on port `8000`.

```dockerfile
FROM python:3.12-slim
WORKDIR /app
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

For FastAPI, use `CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]`.

### Node.js

Assumes `package.json`, `package-lock.json`, and `server.js`, listening on port `3000`.

```dockerfile
FROM node:22-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
ENV NODE_ENV=production
EXPOSE 3000
CMD ["node", "server.js"]
```

### Go

Assumes a Go module and a `main.go` package, listening on port `8080`.

```dockerfile
FROM golang:1.24-alpine AS builder
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /app ./...

FROM alpine:3.21
WORKDIR /app
COPY --from=builder /app /app
EXPOSE 8080
CMD ["/app"]
```

### Rust

Assumes a Cargo project with `src/main.rs`, listening on port `8080`.

```dockerfile
FROM rust:1.86 AS builder
WORKDIR /src
COPY Cargo.toml Cargo.lock ./
COPY src ./src
RUN cargo build --release

FROM debian:bookworm-slim
WORKDIR /app
COPY --from=builder /src/target/release/my-app /app/my-app
EXPOSE 8080
CMD ["/app/my-app"]
```

Replace `my-app` with the Rust binary name.

## 4. Expose the Application on an EC2 Instance

You need both Docker port publishing and an EC2 Security Group rule.

### On the EC2 Instance

1. Install Docker, start the Docker service, and authenticate to your image registry.
2. Pull the image:

	```bash
	docker pull <registry>/<repository>:<tag>
	```

3. Run it with a host-to-container port mapping:

	```bash
	docker run -d --name my-app \
	  -p 80:8080 \
	  --restart unless-stopped \
	  <registry>/<repository>:<tag>
	```

4. Verify the container and local response:

	```bash
	docker ps
	curl http://localhost
	```

### In AWS

1. Open **EC2 > Instances**, select the instance, and open its **Security** tab.
2. Open the attached Security Group and edit **Inbound rules**.
3. Add TCP port `80` for HTTP or `443` for HTTPS.
4. Use `0.0.0.0/0` only for intentionally public services. Restrict SSH (`22`) to your own IP or trusted network.
5. Confirm the instance has a public IPv4 address or DNS name, and that its subnet route table and network ACL allow the traffic.
6. Browse to `http://<ec2-public-ip>`.

For production HTTPS, put Nginx or an Application Load Balancer in front of the container and keep the application port private where possible.

## 5. Docker Command Handsheet

### Images

```bash
docker images                         # List local images
docker build -t name:tag .             # Build an image
docker build --no-cache -t name:tag .   # Build without cache
docker pull name:tag                    # Download an image
docker push name:tag                    # Upload an image
docker tag name:tag registry/name:tag   # Add a registry tag
docker rmi name:tag                     # Remove an image
docker image prune                      # Remove dangling images
docker history name:tag                 # Show image layers
```

### Containers

```bash
docker ps                              # Running containers
docker ps -a                           # All containers
docker run --name app image:tag        # Create and start a container
docker run -d -p 8080:8080 image:tag    # Detached mode with port mapping
docker start app                       # Start an existing container
docker stop app                        # Gracefully stop a container
docker restart app                     # Restart a container
docker rm app                          # Remove a stopped container
docker rm -f app                       # Stop and remove it
docker logs app                        # View logs
docker logs -f --tail 100 app          # Follow recent logs
docker exec -it app sh                 # Open a shell inside a container
docker inspect app                     # Show configuration and network details
docker stats                           # Live CPU and memory usage
docker cp app:/app/file.txt .          # Copy a file from a container
docker cp ./file.txt app:/app/         # Copy a file into a container
docker top app                         # Show processes in a container
docker port app                        # Show published ports
```

### Volumes and Networks

```bash
docker volume ls
docker volume create app-data
docker volume inspect app-data
docker volume rm app-data
docker network ls
docker network create app-network
docker network inspect app-network
```

Example with persistent data and a custom network:

```bash
docker run -d --name database \
	--network app-network \
	-v app-data:/var/lib/data \
	database-image:tag
```

### Cleanup

```bash
docker container prune                   # Remove stopped containers
docker image prune                       # Remove dangling images
docker volume prune                      # Remove unused volumes
docker system df                         # Show Docker disk usage
docker system prune                      # Remove unused Docker data
docker system prune -a                   # Also remove unused images
```

Review cleanup prompts carefully: removing volumes can permanently delete data.

## 6. Docker Compose Example

Use Compose for local multi-container development:

```yaml
services:
	app:
		build: .
		ports:
			- "8080:8080"
		environment:
			DATABASE_URL: postgres://user:password@db:5432/app
		depends_on:
			- db

	db:
		image: postgres:17-alpine
		environment:
			POSTGRES_USER: user
			POSTGRES_PASSWORD: password
			POSTGRES_DB: app
		volumes:
			- postgres-data:/var/lib/postgresql/data

volumes:
	postgres-data:
```

```bash
docker compose up -d --build
docker compose ps
docker compose logs -f app
docker compose exec app sh
docker compose down
docker compose down -v  # Also remove named volumes
```

## 7. Practical Best Practices

- Use small, supported base images and pin versions instead of relying on `latest`.
- Use multi-stage builds to keep compilers and development tools out of the final image.
- Run as a non-root user when the base image supports it.
- Keep containers stateless; store durable data in volumes or managed services.
- Pass configuration through environment variables and secrets through a secrets manager.
- Add a health check for services that need reliable orchestration.
- Log to standard output and standard error so the hosting platform can collect logs.
- Scan images for vulnerabilities and rebuild them regularly.
- Document the expected port, start command, environment variables, and health endpoint.