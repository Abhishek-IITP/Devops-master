# Docker Bind Mounts and Volumes

## Interview Summary

Containers are **ephemeral by design**. A container is a running instance of an image, and its container-writable layer is disposable. Recreating a container from the same image gives a clean writable layer; it does not restore files written only inside the old container.

This separation is intentional:

- Images provide immutable, repeatable application code and dependencies.
- Containers provide isolated runtime processes.
- Persistent data belongs outside the container writable layer.
- Configuration and secrets should normally be injected at runtime, not baked into the image.

The correct interview answer is: **a container can be replaced at any time, so stateful data must be stored in a volume, bind mount, or external service such as a managed database or object storage.**

## What Happens to Files and Logs?

### Container writable layer

Files written to a path that is not mounted are stored in the container writable layer.

- `docker stop` stops the process but keeps the container and its writable layer.
- `docker start` starts the same container, so those files are still present.
- `docker rm` removes the container and its writable layer.
- A replacement container starts with a new writable layer, so the old files are not available.

Do not use this layer for databases, uploads, queues, or other business data.

### Container logs

By default, Docker captures data written to the container's `stdout` and `stderr`. With the default `json-file` logging driver, logs are stored by the Docker daemon on the host and are normally available after a container stops. They are not a durable application backup and can be lost when the container is removed, the host fails, or log rotation is misconfigured.

Production practice:

- Write application logs to `stdout` and `stderr`.
- Configure a logging driver such as `local`, `syslog`, `journald`, `fluentd`, or a cloud logging driver.
- Configure rotation and retention; never allow logs to fill the host disk.
- Send long-term logs to a centralized system such as CloudWatch, Elasticsearch/OpenSearch, Loki, or another supported platform.

Useful commands:

```bash
docker logs <container>
docker logs --tail 100 -f <container>
docker inspect --format '{{.HostConfig.LogConfig}}' <container>
docker info --format '{{.LoggingDriver}}'
```

## Storage Options

### Named volumes

A named volume is managed by Docker and stored in Docker's data directory on the host. Docker controls its lifecycle and can use a volume driver to connect to external storage. This is usually the preferred option for containerized state that should be managed separately from a container.

Advantages: Docker lifecycle management, easy reuse, less coupling to host paths, and support for volume drivers.

Limitations: data is still tied to the host or volume backend, and a volume is not a backup. Backups and restore testing are still required.

### Bind mounts

A bind mount maps an existing host path into a container. The host path is the source of truth, so it is useful for source code during development, configuration files, certificates, and files that must be directly managed by the host.

Advantages: direct host access and precise control over the host path.

Risks: stronger coupling to host layout, permission/ownership problems, and possible host data exposure. Prefer read-only mounts whenever the application does not need write access.

### tmpfs mounts

A tmpfs mount stores data in host memory and does not persist across container removal or host restart. It is useful for short-lived sensitive or temporary data, not for durable storage.

## `docker volume` Management Commands

```bash
# Create a named volume
docker volume create abhi

# List all named volumes
docker volume ls

# Filter the list
docker volume ls --filter name=abhi

# Display driver, mountpoint, labels, and options
docker volume inspect abhi

# Remove one unused volume
docker volume rm abhi

# Remove all unused local volumes; review carefully before using in production
docker volume prune
docker volume prune --filter label=environment=dev
```

`docker volume rm` refuses to remove a volume referenced by a container, including a stopped container. That protection prevents accidental data loss. The safe sequence is:

```bash
docker ps -a --filter volume=abhi
docker stop <container>
docker rm <container>
docker volume rm abhi
```

Removing a container does not remove a named volume unless `docker rm -v` is used. `docker compose down -v` explicitly removes Compose-managed volumes, so use it only when data deletion is intended.

## Mounting a Named Volume

The short syntax is `-v SOURCE:TARGET[:OPTIONS]`. The long syntax is `--mount` with explicit key-value fields. Long syntax is easier to review in production scripts because a missing or malformed field is more visible.

```bash
# Create and mount the volume using long syntax
docker volume create abhi
docker run -d --name web-volume \
	--mount type=volume,source=abhi,target=/data \
	nginx:latest

# The equivalent short syntax
docker run -d --name web-volume-short \
	-v abhi:/data \
	nginx:latest

# Write data and verify it
docker exec web-volume sh -c 'date > /data/created-by-container.txt'
docker exec web-volume ls -la /data
docker volume inspect abhi

# Reuse the same data from another container
docker run --rm -v abhi:/data busybox cat /data/created-by-container.txt
```

The `source` is the volume name and `target` is the path inside the container. If a named volume does not exist, `docker run -v name:/path` can create it automatically; using `docker volume create` first makes the lifecycle explicit.

## Bind Mount Examples

```bash
# Linux/macOS: mount a host directory
mkdir -p "$PWD/site"
echo 'hello from the host' > "$PWD/site/index.html"
docker run --rm -v "$PWD/site:/usr/share/nginx/html:ro" nginx:latest

# Equivalent long syntax
docker run --rm \
	--mount type=bind,source="$PWD/site",target=/usr/share/nginx/html,readonly \
	nginx:latest

# Mount one host file as read-only
docker run --rm \
	--mount type=bind,source="$PWD/app.conf",target=/etc/app/app.conf,readonly \
	my-app:latest
```

On Windows PowerShell, use an absolute path or `${PWD}` and quote paths that contain spaces:

```powershell
docker run --rm -v "${PWD}\site:/usr/share/nginx/html:ro" nginx:latest
```

The host path must exist and the Docker daemon must be allowed to access it. On Docker Desktop, file-sharing permissions can affect bind mounts.

## Read-Only and Advanced Mount Options

```bash
# Read-only named volume
docker run --rm --mount type=volume,source=abhi,target=/data,readonly nginx:latest
docker run --rm -v abhi:/data:ro nginx:latest

# Temporary in-memory filesystem
docker run --rm \
	--mount type=tmpfs,target=/run/secrets,tmpfs-size=1048576,tmpfs-mode=0700 \
	nginx:latest

# Equivalent short syntax for tmpfs
docker run --rm --tmpfs /run/secrets:rw,noexec,nosuid,size=1m nginx:latest

# Bind mount with shared propagation, useful for specialized host mount workflows
docker run --rm \
	--mount type=bind,source=/mnt/host,target=/mnt/container,bind-propagation=rshared \
	my-tool:latest
```

Use propagation only when the workload requires nested mount events to move between host and container. It is not a general persistence setting.

## Backup and Restore a Volume

A volume is persistent storage, not a backup. A common portable approach is to mount the volume into a short-lived helper container and archive it:

```bash
# Create a compressed backup in the current host directory
docker run --rm \
	-v abhi:/data:ro \
	-v "$PWD:/backup" \
	busybox tar czf /backup/abhi-backup.tgz -C /data .

# Restore into an existing or empty volume
docker run --rm \
	-v abhi:/data \
	-v "$PWD:/backup" \
	busybox sh -c 'tar xzf /backup/abhi-backup.tgz -C /data'
```

For databases, prefer a database-aware dump such as `pg_dump`, `mysqldump`, or the vendor's backup tool. Files copied while a database is actively writing may be inconsistent.

## Practical Investigation Workflow

```bash
# Identify mounts attached to a container
docker inspect --format '{{json .Mounts}}' <container>

# Check whether a volume is still referenced
docker ps -a --filter volume=abhi

# See container state, exit code, and error
docker inspect --format 'status={{.State.Status}} exit={{.State.ExitCode}} error={{.State.Error}}' <container>

# Check application output and recent events
docker logs --tail 200 <container>
docker events --filter container=<container>
```

When a container goes down, first distinguish **stopped** from **removed**, then check whether the data path is a volume or bind mount. Finally check exit code, logs, host disk capacity, permissions, and the health check before recreating it.

## Interview Questions and Model Answers

### 1. Why are containers ephemeral?

Containers are designed to be replaceable and reproducible. Their writable layer belongs to one container instance, while the image and external storage are separate. This supports immutable deployments, fast rollback, horizontal scaling, and recovery by replacement.

### 2. What happens to data when a container stops?

Stopping preserves the container and its writable layer. Data remains until the container is removed. Data in a named volume or bind mount remains independently of the container, subject to the lifecycle of that storage.

### 3. What happens when a container is deleted?

The container writable layer is deleted. Named volumes normally remain, but `docker rm -v` can remove anonymous volumes associated with the container. Bind-mounted data remains on the host, although the mount itself is gone.

### 4. How would you persist PostgreSQL data?

Mount a named volume at PostgreSQL's data directory, use a database-aware backup strategy, and test restore. In production, a managed database may be preferable because replication, failover, patching, and backups are handled by the service.

### 5. `-v` versus `--mount`?

Both can create volumes and bind mounts. `-v` is concise and common for interactive use. `--mount` is explicit, supports clearer options, and is generally easier to audit in automation.

### 6. Named volume versus bind mount?

Use a named volume when Docker should manage application data and portability matters. Use a bind mount when a specific host path must be visible or edited directly, such as local source code or a host-managed configuration file.

### 7. Why does `docker volume rm abhi` say “volume is in use”?

At least one container still references the volume, even if that container is stopped. Find it with `docker ps -a --filter volume=abhi`, remove or recreate the container after confirming the data is backed up, and then remove the volume.

### 8. A new container cannot see data from the old one. What do you check?

Check that both containers use the same volume name, that the mount target is correct, and that the application is writing to that target rather than another path. Use `docker inspect` on both containers and verify permissions and ownership inside the container.

### 9. The container is healthy but the host disk is full. What do you investigate?

Check container logs and rotation, unused images, stopped containers, build cache, volumes, and Docker's data-root filesystem. Use `docker system df` and remove resources only after confirming they are unused and backed up.

```bash
docker system df
docker system df -v
```

### 10. How do you make a mount read-only?

Use `:ro` with short syntax or `readonly` with long syntax. This reduces accidental writes and limits the impact of a compromised process.

### 11. Can multiple containers use one volume?

Yes, provided the application and storage semantics support it. Read-only sharing is usually straightforward. Multiple writers require application-level coordination and a storage backend that supports the required consistency and locking behavior.

### 12. Are volumes replicated automatically?

No. A local volume is usually local to one Docker host. Replication, snapshots, cross-zone durability, and disaster recovery require a volume driver, storage platform, orchestrator feature, or external service.

### 13. How are volumes handled in Docker Compose?

Declare named volumes under `volumes` and mount them in the service. `docker compose down` keeps named volumes by default; `docker compose down -v` removes them and can destroy data.

```yaml
services:
	db:
		image: postgres:16
		environment:
			POSTGRES_PASSWORD: change-me-in-a-lab-only
		volumes:
			- postgres-data:/var/lib/postgresql/data

volumes:
	postgres-data:
```

### 14. What is the production approach for logs?

Emit structured logs to `stdout` and `stderr`, configure a bounded logging driver, and ship logs to centralized storage with retention, access control, and alerting. Do not depend on a container's writable layer for log retention.

### 15. What is the difference between persistence and backup?

Persistence keeps data available across container replacement. A backup is a separate recoverable copy that protects against corruption, deletion, host failure, and disaster. A volume provides persistence, not backup by itself.

## Final Interview Checklist

- State belongs outside the container writable layer.
- A stopped container is not the same as a deleted container.
- Named volumes are Docker-managed; bind mounts are host-path-managed.
- Prefer `--mount` for readable automation and `:ro` where possible.
- Logs need rotation and centralized shipping.
- Volumes need backups, restore tests, access control, and a disaster-recovery plan.
- Always inspect mounts and container state before deleting anything.
