# Docker Interview Questions & Answers
> Covers Basics · Images · Containers · Networking · Storage · Docker Compose · Security · Scenarios

---

## 🟢 SECTION 1: BASICS & CORE CONCEPTS

---

### 1. What is Docker and why is it used?
**Answer:**
Docker is an open-source platform that uses OS-level virtualization to deliver software in packages called **containers**. Containers bundle an application's code, runtime, libraries, and dependencies into a single portable unit.

**Why it's used:**
- **Consistency** — "Works on my machine" becomes "works everywhere"
- **Isolation** — Apps run in separate environments without conflicts
- **Portability** — Run the same image on laptop, server, or cloud
- **Speed** — Containers start in milliseconds vs minutes for VMs
- **Efficiency** — Containers share the host OS kernel; no full OS per app
- **Scalability** — Works seamlessly with orchestrators like Kubernetes

---

### 2. What is the difference between a Container and a Virtual Machine?
**Answer:**

| Feature | Container | Virtual Machine |
|---|---|---|
| OS | Shares host OS kernel | Full OS per VM |
| Size | MBs | GBs |
| Startup time | Milliseconds | Minutes |
| Isolation | Process-level | Hardware-level |
| Performance | Near-native | Overhead from hypervisor |
| Portability | High | Medium |
| Use case | Microservices, apps | Full OS environments |

**Containers** use Linux namespaces and cgroups for isolation. **VMs** use a hypervisor (VMware, VirtualBox, KVM) to emulate hardware.

---

### 3. What is a Docker Image vs a Docker Container?
**Answer:**

- **Docker Image** — A read-only, layered template that contains everything needed to run an application (code, runtime, config, libraries). Images are built from a `Dockerfile`. They are immutable.

- **Docker Container** — A running (or stopped) instance of an image. It adds a writable layer on top of the image. Multiple containers can run from the same image independently.

```
Dockerfile → docker build → Image → docker run → Container
```

An image is like a class in OOP; a container is like an object (instance).

---

### 4. What is a Dockerfile?
**Answer:**
A Dockerfile is a text file with a sequence of instructions that Docker uses to build an image. Each instruction creates a new layer in the image.

```dockerfile
# Base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy dependency files first (layer caching optimization)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Default command
CMD ["node", "server.js"]
```

---

### 5. What is Docker Hub?
**Answer:**
Docker Hub is the default public container registry where Docker images are stored and shared. It hosts official images (nginx, node, postgres, alpine) and community/private images.

```bash
docker pull nginx:latest          # Pull from Docker Hub
docker push myuser/myapp:v1.0     # Push to Docker Hub
docker search nginx               # Search images
```

Other registries: Amazon ECR, Google Container Registry (GCR), GitHub Container Registry (GHCR), Azure Container Registry (ACR), self-hosted (Harbor).

---

### 6. What are the most commonly used Docker commands?
**Answer:**

```bash
# Image commands
docker images                         # List local images
docker pull nginx:alpine              # Download image
docker build -t myapp:v1 .            # Build image
docker push myuser/myapp:v1           # Push to registry
docker rmi myapp:v1                   # Remove image
docker image prune                    # Remove unused images

# Container commands
docker run -d -p 8080:80 nginx        # Run container in background
docker run -it ubuntu bash            # Run interactive shell
docker ps                             # List running containers
docker ps -a                          # List all containers
docker stop <container>               # Graceful stop
docker kill <container>               # Force stop
docker rm <container>                 # Remove stopped container
docker exec -it <container> bash      # Shell into running container
docker logs <container>               # View logs
docker logs -f <container>            # Follow logs
docker inspect <container>            # Detailed JSON info
docker stats                          # Live resource usage
docker cp file.txt <container>:/path  # Copy file into container
```

---

### 7. What is `docker run` and its important flags?
**Answer:**

```bash
docker run [OPTIONS] IMAGE [COMMAND]

# Key flags:
-d                    # Detached mode (background)
-it                   # Interactive + TTY (for shell)
-p 8080:80            # Port mapping host:container
-v /host:/container   # Volume mount
-e KEY=VALUE          # Set environment variable
--env-file .env       # Load env vars from file
--name my-container   # Assign container name
--rm                  # Auto-remove when stopped
--network my-network  # Attach to network
--restart always      # Restart policy
-m 512m               # Memory limit
--cpus 1.5            # CPU limit
--user 1000:1000      # Run as specific user

# Example
docker run -d \
  --name my-app \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -v /data:/app/data \
  --restart unless-stopped \
  myapp:v1.0
```

---

### 8. What is a Docker Registry?
**Answer:**
A Docker Registry is a storage and distribution system for Docker images. It stores image layers and makes them accessible via push/pull.

- **Public Registry** — Docker Hub, GitHub Container Registry (open to all)
- **Private Registry** — Self-hosted (Harbor, Nexus) or cloud-managed (ECR, GCR, ACR)

```bash
# Login to a registry
docker login registry.example.com

# Tag image for a specific registry
docker tag myapp:v1 registry.example.com/team/myapp:v1

# Push to private registry
docker push registry.example.com/team/myapp:v1

# Pull from private registry
docker pull registry.example.com/team/myapp:v1
```

---

### 9. What are Docker layers and how do they work?
**Answer:**
Docker images are made up of read-only layers, where each layer corresponds to a Dockerfile instruction that modifies the filesystem (`FROM`, `RUN`, `COPY`, `ADD`).

- Layers are **cached** — if a layer hasn't changed, Docker reuses the cached version
- Layers are **shared** — if two images share the same base layers, they share storage
- Only the container's writable layer (Copy-on-Write) differs between containers from the same image

```
┌──────────────────────┐  ← Writable container layer
├──────────────────────┤  ← COPY . .
├──────────────────────┤  ← RUN npm install
├──────────────────────┤  ← COPY package.json .
├──────────────────────┤  ← WORKDIR /app
└──────────────────────┘  ← FROM node:18-alpine (base)
```

**Build cache optimization:** Put rarely changing instructions first, frequently changing ones last.

---

### 10. What is the difference between `CMD` and `ENTRYPOINT`?
**Answer:**

| Instruction | Purpose | Can be overridden at runtime? |
|---|---|---|
| `CMD` | Default command/args; easily replaced | Yes (`docker run image <new-cmd>`) |
| `ENTRYPOINT` | Main executable; not easily replaced | Requires `--entrypoint` flag |

```dockerfile
# CMD only — easily overridden
CMD ["npm", "start"]

# ENTRYPOINT + CMD — ENTRYPOINT is the executable, CMD provides defaults
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080"]   # default args, overridable

# docker run myimage --port 9090
# Runs: python app.py --port 9090
```

**Best practice:** Use `ENTRYPOINT` for the main executable, `CMD` for default arguments that users might want to override.

---

## 🔵 SECTION 2: DOCKERFILE & IMAGE BEST PRACTICES

---

### 11. What is a Multi-Stage Build and why is it important?
**Answer:**
Multi-stage builds use multiple `FROM` statements in one Dockerfile. Each stage can copy artifacts from previous stages. The final image only contains what's needed at runtime — drastically reducing image size.

```dockerfile
# Stage 1: Build (has compilers, dev tools)
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build      # Produces /app/dist

# Stage 2: Production (minimal image, no build tools)
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
USER node
CMD ["node", "dist/server.js"]
```

**Result:** Build image might be 1.2GB; production image is ~200MB.

---

### 12. What are Dockerfile best practices?
**Answer:**

1. **Use specific image tags** — `node:18-alpine` not `node:latest`
2. **Use slim/alpine base images** — Smaller attack surface, faster pulls
3. **Order layers by change frequency** — Dependencies before source code (cache optimization)
4. **Combine RUN commands** to reduce layers:
   ```dockerfile
   RUN apt-get update && \
       apt-get install -y curl git && \
       rm -rf /var/lib/apt/lists/*
   ```
5. **Use `.dockerignore`** — Exclude `node_modules`, `.git`, `*.log`
6. **Use non-root user** for security:
   ```dockerfile
   RUN addgroup -S appgroup && adduser -S appuser -G appgroup
   USER appuser
   ```
7. **Use COPY over ADD** — `ADD` has extra magic (untar, URL fetch) that's often unexpected
8. **Multi-stage builds** — Keep production images lean
9. **Use `--no-cache`** in CI to avoid stale layers
10. **HEALTHCHECK instruction** — Docker knows if your app is healthy

---

### 13. What is `.dockerignore`?
**Answer:**
`.dockerignore` tells Docker which files and directories to exclude from the build context sent to the Docker daemon. This speeds up builds and prevents secrets from being accidentally included.

```
node_modules/
.git/
*.log
.env
.env.*
dist/
coverage/
*.md
Dockerfile
docker-compose*.yml
.DS_Store
__pycache__/
*.pyc
```

Without `.dockerignore`, Docker sends the entire project directory to the daemon — including `node_modules` which can be hundreds of MB.

---

### 14. What is the difference between `COPY` and `ADD` in Dockerfile?
**Answer:**

| Instruction | Copies local files | Supports URLs | Auto-extracts archives |
|---|---|---|---|
| `COPY` | Yes | No | No |
| `ADD` | Yes | Yes | Yes (tar.gz, etc.) |

**Best practice:** Always use `COPY` unless you specifically need `ADD`'s URL or auto-extract functionality. `COPY` is more transparent and predictable.

```dockerfile
COPY src/ /app/src/           # Clear and explicit
ADD https://... /tmp/file     # Only when fetching remote URL
ADD archive.tar.gz /app/      # Only when auto-extract needed
```

---

### 15. What is the difference between `RUN`, `CMD`, and `ENTRYPOINT`?
**Answer:**

| Instruction | When it runs | Purpose |
|---|---|---|
| `RUN` | During `docker build` | Execute commands to build the image (install packages, compile code) |
| `CMD` | During `docker run` (default) | Default command to run when container starts; easily overridden |
| `ENTRYPOINT` | During `docker run` (fixed) | Main process of the container; not easily overridden |

```dockerfile
RUN apt-get install -y python3   # Runs at BUILD time
ENTRYPOINT ["python3"]           # Runs at RUN time (fixed)
CMD ["app.py"]                   # Runs at RUN time (default args)
```

---

### 16. What is Docker image tagging and versioning strategy?
**Answer:**
Tags are labels on images that typically represent versions. Without a tag, Docker uses `latest` by default (avoid in production).

```bash
# Tag at build time
docker build -t myapp:1.0.0 .
docker build -t myapp:1.0 .
docker build -t myapp:latest .

# Tag existing image
docker tag myapp:1.0.0 registry.io/team/myapp:1.0.0

# Recommended versioning strategy
myapp:1.2.3              # Exact semantic version (pin in production)
myapp:1.2                # Minor version (auto-patch updates)
myapp:1                  # Major version
myapp:latest             # Never use in production manifests
myapp:git-abc1234        # Git commit SHA (traceable)
myapp:20240315-abc1234   # Date + SHA (common in CI)
```

---

### 17. How do you reduce Docker image size?
**Answer:**

1. **Use Alpine or Distroless base images:**
   ```dockerfile
   FROM node:18-alpine     # ~180MB vs node:18 ~900MB
   FROM gcr.io/distroless/nodejs18  # Even smaller, no shell
   ```

2. **Multi-stage builds** — only copy final artifacts

3. **Minimize layers:**
   ```dockerfile
   RUN apt-get update && apt-get install -y pkg && rm -rf /var/lib/apt/lists/*
   ```

4. **Don't install dev dependencies:**
   ```dockerfile
   RUN npm ci --only=production
   ```

5. **Use `.dockerignore`** to exclude unnecessary files

6. **Use `docker image prune`** to remove dangling images

7. **Squash layers** (experimental):
   ```bash
   docker build --squash -t myapp .
   ```

---

### 18. What is a dangling image?
**Answer:**
A dangling image is an image that has no tag and is not referenced by any container. They accumulate after repeated builds when a tag is reassigned to a new image. The old one becomes untagged (`<none>:<none>`).

```bash
docker images -f dangling=true     # List dangling images
docker image prune                 # Remove all dangling images
docker image prune -a              # Remove ALL unused images
docker system prune                # Remove all unused data (images, containers, networks, volumes)
docker system prune -a --volumes   # Nuclear cleanup
```

---

## 🟡 SECTION 3: NETWORKING

---

### 19. What are Docker network types?
**Answer:**

| Network Driver | Description | Use Case |
|---|---|---|
| **bridge** | Default. Virtual network on host. Containers get their own IPs. | Single host apps, most common |
| **host** | Container shares host's network stack. No isolation. | Performance-critical apps |
| **none** | No network. Fully isolated. | Batch jobs, security |
| **overlay** | Multi-host networking. Used in Docker Swarm. | Distributed apps |
| **macvlan** | Container gets its own MAC address on the network. | Legacy apps needing direct network access |

```bash
docker network ls                                    # List networks
docker network create my-network                     # Create bridge network
docker network create --driver overlay my-overlay    # Overlay network
docker run --network my-network myapp               # Attach container
docker network inspect my-network                    # Inspect network
docker network connect my-network <container>        # Connect running container
```

---

### 20. How do containers communicate with each other?
**Answer:**

**On the same custom bridge network:** Containers can reach each other by container name (Docker provides DNS resolution).

```bash
# Create network
docker network create app-network

# Run containers on same network
docker run -d --name db --network app-network postgres
docker run -d --name app --network app-network myapp

# app container can reach db by name:
# POSTGRES_HOST=db (Docker DNS resolves "db" to its IP)
```

**On the default bridge network:** Containers can only communicate by IP address (no DNS). Use custom networks instead.

**Between different networks:** Containers can't communicate unless explicitly connected to both networks.

---

### 21. What is port mapping in Docker?
**Answer:**
Port mapping (`-p`) binds a port on the host to a port on the container. Traffic on the host port is forwarded to the container port.

```bash
-p 8080:80          # host_port:container_port
-p 127.0.0.1:8080:80  # Bind to specific host IP (localhost only)
-p 80               # Random host port → container port 80
-P                  # Map ALL exposed ports to random host ports

docker port <container>   # Show port mappings
```

The container `EXPOSE` instruction is documentation only — it doesn't publish the port. You need `-p` to actually map it.

---

### 22. What is the difference between `EXPOSE` and `-p` flag?
**Answer:**

- **`EXPOSE` in Dockerfile** — Documents that the container listens on a port. Informational only. Does NOT publish the port to the host.
- **`-p` in `docker run`** — Actually maps a host port to a container port, making it accessible from outside.

```dockerfile
EXPOSE 8080    # Documents the port, no real effect on networking
```

```bash
docker run -p 3000:8080 myapp   # ACTUALLY publishes port 8080 as 3000 on host
```

---

### 23. What is DNS resolution in Docker networks?
**Answer:**
Docker's embedded DNS server (`127.0.0.11`) provides automatic name resolution for containers on user-defined networks. Containers can reach each other by:
- Container name
- Service name (in Docker Compose)
- Network alias

```bash
# Container "web" can ping "db" by name
docker exec web ping db

# DNS lookup
docker exec web nslookup db
```

**Note:** The default bridge network does NOT provide DNS. Always use custom networks for service discovery.

---

## 🟠 SECTION 4: STORAGE & VOLUMES

---

### 24. What are Docker Volumes?
**Answer:**
Volumes are the preferred mechanism for persisting data generated and used by Docker containers. Unlike bind mounts, volumes are managed by Docker and stored in Docker's storage area (`/var/lib/docker/volumes/`).

```bash
# Create a volume
docker volume create my-data

# Use volume in container
docker run -v my-data:/app/data myapp
# or
docker run --mount source=my-data,target=/app/data myapp

# List volumes
docker volume ls

# Inspect volume
docker volume inspect my-data

# Remove unused volumes
docker volume prune
```

---

### 25. What is the difference between Volumes, Bind Mounts, and tmpfs?
**Answer:**

| Type | Storage Location | Managed By | Use Case |
|---|---|---|---|
| **Volume** | Docker's area (`/var/lib/docker/volumes`) | Docker | Persistent app data, DBs |
| **Bind Mount** | Any host path | Host OS | Dev (live code reload), sharing host files |
| **tmpfs** | Host memory only (RAM) | Host OS | Temporary, sensitive data |

```bash
# Volume (recommended for production)
docker run -v my-volume:/data myapp

# Bind mount (dev-friendly)
docker run -v /home/user/code:/app myapp
docker run -v $(pwd):/app myapp    # Mount current directory

# tmpfs (in-memory only)
docker run --tmpfs /tmp myapp
```

---

### 26. When should you use volumes vs bind mounts?
**Answer:**

**Use Volumes when:**
- Data needs to persist beyond container lifecycle (databases)
- Sharing data between multiple containers
- Backing up, restoring, or migrating data
- On non-Linux hosts where host filesystem performance is poor

**Use Bind Mounts when:**
- Developing locally (mount source code for hot reload)
- Reading host-specific config files
- Writing logs to host-specific paths

---

### 27. How do you backup and restore Docker volumes?
**Answer:**
```bash
# Backup: tar the volume contents into a file on the host
docker run --rm \
  -v my-volume:/source \
  -v $(pwd):/backup \
  alpine tar czf /backup/volume-backup.tar.gz -C /source .

# Restore: extract tar back into a volume
docker run --rm \
  -v my-volume:/target \
  -v $(pwd):/backup \
  alpine tar xzf /backup/volume-backup.tar.gz -C /target

# For databases (PostgreSQL example)
docker exec postgres pg_dump -U postgres mydb > backup.sql
docker exec -i postgres psql -U postgres mydb < backup.sql
```

---

## 🔴 SECTION 5: DOCKER COMPOSE

---

### 28. What is Docker Compose?
**Answer:**
Docker Compose is a tool for defining and running multi-container Docker applications using a YAML file (`docker-compose.yml`). It manages the entire lifecycle of your application stack with a single command.

```yaml
# docker-compose.yml
version: '3.9'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=db
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data

volumes:
  postgres-data:
  redis-data:
```

```bash
docker compose up -d           # Start in background
docker compose down            # Stop and remove containers
docker compose down -v         # Also remove volumes
docker compose logs -f         # Follow logs
docker compose ps              # List services
docker compose exec web bash   # Shell into service
docker compose build           # Rebuild images
docker compose pull            # Pull latest images
```

---

### 29. What is the difference between `docker-compose up` and `docker-compose start`?
**Answer:**

| Command | Creates containers? | Starts existing? |
|---|---|---|
| `docker compose up` | Yes (if not exist) | Yes |
| `docker compose start` | No | Yes (only existing) |
| `docker compose run` | Yes (one-off) | — |

- `up` — Full lifecycle: creates networks, volumes, builds images if needed, starts containers
- `start` — Only starts already-created but stopped containers
- `run` — Runs a one-off command in a new container (for scripts, migrations)

```bash
docker compose run web python manage.py migrate   # One-off command
```

---

### 30. What is `depends_on` in Docker Compose?
**Answer:**
`depends_on` controls the startup **order** of services. By default, it only waits for the container to start — not for the application inside to be ready.

```yaml
services:
  app:
    depends_on:
      db:
        condition: service_healthy    # Wait for healthcheck to pass
      redis:
        condition: service_started    # Just wait for start (default)
```

Without `condition: service_healthy`, `depends_on` doesn't wait for PostgreSQL to be accepting connections — just for the container to start. Always combine with `healthcheck`.

---

### 31. What are Docker Compose profiles?
**Answer:**
Profiles let you selectively enable services for different environments (dev, test, prod) in a single Compose file.

```yaml
services:
  app:
    image: myapp
    # No profile = always starts

  db:
    image: postgres
    # No profile = always starts

  pgadmin:
    image: dpage/pgadmin4
    profiles: ["dev"]        # Only starts with dev profile

  tests:
    image: myapp
    command: npm test
    profiles: ["test"]       # Only starts with test profile
```

```bash
docker compose --profile dev up     # Start app + db + pgadmin
docker compose --profile test up    # Start app + db + tests
docker compose up                   # Start only app + db
```

---

### 32. What is the difference between Docker Compose v2 and v3?
**Answer:**
Docker Compose v3 was designed for Docker Swarm with added `deploy` key support. Compose v2 (and the newer format without `version:`) is for standalone Docker. Since Docker Compose v2 as a CLI plugin, the `version` field is optional/deprecated.

**Key changes:**
- `version` field no longer required (and ignored in latest Compose)
- `deploy` key only relevant for Swarm deployments
- New `depends_on` with `condition: service_healthy`
- Better secret and config management

```bash
# Old: docker-compose (v1 binary)
docker-compose up

# New: docker compose (v2 plugin, built into Docker)
docker compose up
```

---

## 🟣 SECTION 6: SECURITY

---

### 33. What are Docker security best practices?
**Answer:**

1. **Run as non-root user:**
   ```dockerfile
   RUN addgroup -S app && adduser -S app -G app
   USER app
   ```

2. **Use read-only filesystem:**
   ```bash
   docker run --read-only myapp
   ```

3. **Drop Linux capabilities:**
   ```bash
   docker run --cap-drop ALL --cap-add NET_BIND_SERVICE myapp
   ```

4. **Use Distroless or scratch images** — No shell, minimal attack surface

5. **Scan images for vulnerabilities:**
   ```bash
   docker scout cves myapp:latest
   trivy image myapp:latest
   ```

6. **Don't store secrets in images** — Use env vars, Docker secrets, or external vaults

7. **Use specific image digests in production:**
   ```dockerfile
   FROM node@sha256:abc123...   # Immutable reference
   ```

8. **Enable Docker Content Trust (DCT):**
   ```bash
   export DOCKER_CONTENT_TRUST=1
   ```

9. **Limit container resources:**
   ```bash
   docker run -m 512m --cpus 1.0 myapp
   ```

10. **Use network segmentation** — Custom networks, don't expose unnecessary ports

---

### 34. What are Docker Secrets?
**Answer:**
Docker Secrets is a mechanism for securely storing and distributing sensitive data (passwords, tokens, keys) to containers. Secrets are stored in Docker's encrypted Raft store (Swarm) or mounted as files in containers.

```bash
# Create a secret
echo "mypassword" | docker secret create db_password -
docker secret create ssl_cert ./cert.pem

# Use in Docker Compose
services:
  db:
    image: postgres
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password

secrets:
  db_password:
    external: true   # Already created via docker secret create
```

Secrets are mounted at `/run/secrets/<secret_name>` inside the container and never stored in the image or environment variables.

---

### 35. What is Docker Content Trust (DCT)?
**Answer:**
Docker Content Trust uses digital signatures to ensure you only pull/run signed, verified images. It prevents tampering or substitution of images.

```bash
# Enable DCT globally
export DOCKER_CONTENT_TRUST=1

# Sign and push image
docker trust sign myrepo/myimage:v1.0

# View signatures
docker trust inspect myrepo/myimage:v1.0

# Pull only verified images (DCT enforced)
docker pull myrepo/myimage:v1.0
```

---

### 36. How do you prevent privilege escalation in containers?
**Answer:**

```bash
# Prevent privilege escalation via setuid binaries
docker run --security-opt no-new-privileges myapp

# Use a seccomp profile to restrict system calls
docker run --security-opt seccomp=seccomp-profile.json myapp

# Use AppArmor profile
docker run --security-opt apparmor=docker-default myapp

# Run as non-root (in Dockerfile)
USER 1000:1000

# Read-only root filesystem
docker run --read-only --tmpfs /tmp myapp

# Drop all capabilities
docker run --cap-drop ALL myapp
```

---

## ⚫ SECTION 7: ADVANCED CONCEPTS

---

### 37. What is Docker Swarm?
**Answer:**
Docker Swarm is Docker's native container orchestration system. It turns a pool of Docker hosts into a single virtual host. While largely replaced by Kubernetes in production, it's simpler to set up.

```bash
# Initialize swarm
docker swarm init --advertise-addr <manager-ip>

# Join a worker node (copy token from init output)
docker swarm join --token <token> <manager-ip>:2377

# Deploy a stack
docker stack deploy -c docker-compose.yml myapp

# Scale a service
docker service scale myapp_web=5

# List services
docker service ls
docker service ps myapp_web
```

---

### 38. What is the Docker build cache and how does it work?
**Answer:**
Docker caches each layer during a build. If a layer's instruction and all previous layers haven't changed, Docker reuses the cached layer instead of rebuilding it. This makes subsequent builds much faster.

**Cache invalidation rules:**
- `FROM` — Cache invalidated if base image changes
- `RUN` — Cache invalidated if the instruction string changes
- `COPY`/`ADD` — Cache invalidated if any copied file changes

```dockerfile
# WRONG ORDER — code change invalidates npm install cache
COPY . .
RUN npm install

# CORRECT ORDER — dependencies cached separately
COPY package*.json ./
RUN npm install      # Only re-runs when package.json changes
COPY . .             # Source changes don't affect npm install layer
```

```bash
docker build --no-cache -t myapp .   # Force full rebuild
```

---

### 39. What is `docker inspect`?
**Answer:**
`docker inspect` returns detailed JSON information about Docker objects (containers, images, networks, volumes).

```bash
docker inspect <container>                     # Full JSON
docker inspect <container> -f '{{.NetworkSettings.IPAddress}}'  # Extract IP
docker inspect <container> -f '{{.State.Status}}'              # Container status
docker inspect <image>                         # Image details, layers
docker inspect <network>                       # Network details
docker inspect <volume>                        # Volume details

# Useful fields
docker inspect <container> -f '{{json .Mounts}}'        # Volume mounts
docker inspect <container> -f '{{json .HostConfig}}'    # Resource limits
```

---

### 40. What is `docker exec` vs `docker attach`?
**Answer:**

- **`docker exec`** — Runs a NEW process inside an already running container. Doesn't affect the main process.
- **`docker attach`** — Attaches your terminal to the container's MAIN process (stdin/stdout). Exiting will stop the container.

```bash
# exec — Safe, runs new process
docker exec -it my-container bash
docker exec my-container ls /app

# attach — Dangerous for stopping containers
docker attach my-container   # Ctrl+C stops the container!
# Use Ctrl+P, Ctrl+Q to detach without stopping
```

**Always use `exec` for debugging.** Use `attach` only when you specifically need to interact with the main process.

---

### 41. What is Docker's overlay2 storage driver?
**Answer:**
`overlay2` is the default and recommended storage driver for Docker on modern Linux kernels. It implements a layered filesystem where:
- Lower layers (image layers) are read-only
- Upper layer (container layer) is writable
- A merged view shows both layers as one filesystem

```bash
docker info | grep "Storage Driver"   # Check current driver

# overlay2 stores layers in:
# /var/lib/docker/overlay2/
```

Other drivers: `aufs` (older Ubuntu), `devicemapper` (RHEL/CentOS old), `btrfs`, `zfs`.

---

### 42. How does Docker handle logging?
**Answer:**
Docker captures stdout/stderr from container processes. Log drivers control where logs go.

```bash
docker logs <container>            # View logs
docker logs -f <container>         # Follow logs
docker logs --tail 100 <container> # Last 100 lines
docker logs --since 1h <container> # Logs from last hour

# Log drivers (set in daemon.json or docker run)
--log-driver json-file     # Default, stored locally
--log-driver syslog        # System syslog
--log-driver journald      # systemd journal
--log-driver fluentd       # Fluentd log collector
--log-driver awslogs       # AWS CloudWatch
--log-driver splunk        # Splunk

# Set log options
docker run --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  myapp
```

---

### 43. What is a HEALTHCHECK instruction?
**Answer:**
`HEALTHCHECK` tells Docker how to test if a container is still working correctly. Docker uses this to report container health status (healthy, unhealthy, starting).

```dockerfile
HEALTHCHECK --interval=30s \
            --timeout=10s \
            --start-period=40s \
            --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

```bash
docker inspect <container> | grep -A5 '"Health"'
docker ps    # Shows (healthy) or (unhealthy) status
```

---

### 44. What is the difference between `docker stop` and `docker kill`?
**Answer:**

| Command | Signal sent | Behavior |
|---|---|---|
| `docker stop` | SIGTERM first, then SIGKILL after timeout (default 10s) | Graceful shutdown |
| `docker kill` | SIGKILL immediately (or custom signal) | Forceful termination |

```bash
docker stop my-container                   # Graceful (SIGTERM → SIGKILL after 10s)
docker stop -t 30 my-container             # 30s grace period
docker kill my-container                   # Immediate SIGKILL
docker kill --signal SIGUSR1 my-container  # Send custom signal
```

Always prefer `stop` to allow the application to clean up connections and flush data.

---

### 45. What is `docker system df`?
**Answer:**
`docker system df` shows disk usage by Docker objects — images, containers, volumes, build cache.

```bash
docker system df           # Summary
docker system df -v        # Verbose, shows each item

# Output:
# TYPE            TOTAL  ACTIVE  SIZE     RECLAIMABLE
# Images          15     3       4.2GB    3.1GB (73%)
# Containers      5      2       120MB    80MB  (66%)
# Local Volumes   8      3       2.1GB    1.4GB (66%)
# Build Cache     42     0       800MB    800MB
```

---

## 🔶 SECTION 8: SCENARIO-BASED QUESTIONS

---

### 46. SCENARIO: Container exits immediately after `docker run`. How do you debug?

**Answer:**
```bash
# Step 1: See exit code
docker ps -a    # Shows exited containers with status

# Step 2: Check logs of the exited container
docker logs <container-id>

# Step 3: Run interactively to see what happens
docker run -it myapp bash

# Step 4: Override entrypoint to prevent auto-exit
docker run -it --entrypoint bash myapp

# Step 5: Check exit code
docker inspect <container> -f '{{.State.ExitCode}}'
# 0  = Success (CMD ran and finished normally — maybe CMD is not a daemon?)
# 1  = App error
# 127 = Command not found
# 126 = Permission denied

# Common fix: app isn't running as a foreground daemon
# Wrong:  CMD ["nginx"]          starts and exits
# Right:  CMD ["nginx", "-g", "daemon off;"]  foreground
```

---

### 47. SCENARIO: Your Docker image is 3GB. How do you reduce it?

**Answer:**
```bash
# Step 1: Analyze layers
docker history myapp:latest --no-trunc
docker image inspect myapp:latest

# Step 2: Use dive tool to find waste
docker run --rm -it -v /var/run/docker.sock:/var/run/docker.sock \
  wagoodman/dive myapp:latest
```

**Fixes:**
```dockerfile
# Before: 3GB
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y python3 pip
COPY . .
RUN pip install -r requirements.txt

# After: ~200MB
# 1. Use slim base image
FROM python:3.11-slim

# 2. Combine RUN commands and clean up
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc && \
    rm -rf /var/lib/apt/lists/*

# 3. Multi-stage build
FROM python:3.11-slim AS builder
COPY requirements.txt .
RUN pip install --user -r requirements.txt

FROM python:3.11-slim
COPY --from=builder /root/.local /root/.local
COPY . .
CMD ["python", "app.py"]
```

---

### 48. SCENARIO: Container can't connect to the database container. How do you fix it?

**Answer:**
```bash
# Step 1: Verify both containers are on the same network
docker network inspect <network-name>
docker inspect app-container | grep -A5 Networks
docker inspect db-container | grep -A5 Networks

# Step 2: If using default bridge network, switch to custom
docker network create app-net
docker run -d --name db --network app-net postgres
docker run -d --name app --network app-net myapp

# Step 3: Test connectivity from app container
docker exec app-container ping db
docker exec app-container nc -zv db 5432

# Step 4: Check if DB is actually listening
docker exec db-container netstat -tlnp | grep 5432

# Step 5: Verify environment variables
docker exec app-container env | grep DB_

# Step 6: Check container logs
docker logs db-container    # Is DB fully started?
```

**Common fix:** The app container starts before the DB is ready. Use health checks and `depends_on: condition: service_healthy` in Compose.

---

### 49. SCENARIO: How do you update a running container without downtime?

**Answer:**
```bash
# Strategy 1: Blue-Green with Docker Compose
# 1. Build new image
docker build -t myapp:v2 .

# 2. Start new container on same network
docker run -d --name myapp-v2 --network app-net myapp:v2

# 3. Update load balancer / nginx upstream to point to new container

# 4. Stop old container
docker stop myapp-v1
docker rm myapp-v1

# Strategy 2: Docker Compose rolling update
docker compose up -d --no-deps --build web
# --no-deps: don't restart dependencies
# --build: rebuild the image

# Strategy 3: Docker Swarm rolling update
docker service update \
  --image myapp:v2 \
  --update-parallelism 1 \
  --update-delay 10s \
  myapp_web
```

---

### 50. SCENARIO: You need to pass secrets to a container without hardcoding them. How?

**Answer:**

**Option 1 — Environment variables (basic):**
```bash
docker run -e DB_PASSWORD=$DB_PASSWORD myapp
docker run --env-file .env myapp
```

**Option 2 — Docker Secrets (Swarm):**
```bash
echo "mysecretpassword" | docker secret create db_pass -
docker service create --secret db_pass myapp
# Accessible at /run/secrets/db_pass
```

**Option 3 — Docker Compose secrets:**
```yaml
services:
  app:
    secrets:
      - db_password
    environment:
      DB_PASSWORD_FILE: /run/secrets/db_password
secrets:
  db_password:
    file: ./secrets/db_password.txt
```

**Option 4 — External secrets manager (production best practice):**
```bash
# AWS Secrets Manager + inject at runtime
aws secretsmanager get-secret-value --secret-id prod/db/password \
  --query SecretString --output text | docker run -e DB_PASS=$(cat) myapp

# HashiCorp Vault Agent sidecar
# Azure Key Vault / GCP Secret Manager
```

---

### 51. SCENARIO: Docker build is very slow in CI/CD. How do you speed it up?

**Answer:**

**1. Use layer caching properly:**
```dockerfile
# Dependencies first (rarely change)
COPY package*.json ./
RUN npm ci

# Source code last (changes often)
COPY . .
```

**2. Use BuildKit (enabled by default in Docker 23+):**
```bash
DOCKER_BUILDKIT=1 docker build -t myapp .
```

**3. Inline cache (store cache in registry):**
```bash
docker build \
  --cache-from myrepo/myapp:cache \
  --build-arg BUILDKIT_INLINE_CACHE=1 \
  -t myrepo/myapp:latest .

docker push myrepo/myapp:latest
docker tag myrepo/myapp:latest myrepo/myapp:cache
docker push myrepo/myapp:cache
```

**4. Use GitHub Actions/GitLab CI cache:**
```yaml
# GitHub Actions
- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v3

- name: Build and push
  uses: docker/build-push-action@v5
  with:
    cache-from: type=gha
    cache-to: type=gha,mode=max
```

**5. Use `.dockerignore`** to reduce build context size.

---

### 52. SCENARIO: How do you run different environments (dev/staging/prod) with Docker Compose?

**Answer:**
```bash
# File structure
docker-compose.yml          # Base config (shared)
docker-compose.dev.yml      # Development overrides
docker-compose.staging.yml  # Staging overrides
docker-compose.prod.yml     # Production overrides
```

```yaml
# docker-compose.yml (base)
services:
  web:
    image: myapp:${TAG:-latest}
    environment:
      - NODE_ENV=${NODE_ENV}

# docker-compose.dev.yml
services:
  web:
    build: .
    volumes:
      - .:/app          # Hot reload
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development

# docker-compose.prod.yml
services:
  web:
    restart: always
    deploy:
      replicas: 3
    environment:
      - NODE_ENV=production
```

```bash
# Merge files
docker compose -f docker-compose.yml -f docker-compose.dev.yml up
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

---

### 53. SCENARIO: A container is consuming too much memory and causing OOM on the host. What do you do?

**Answer:**
```bash
# Step 1: Check current memory usage
docker stats
docker stats --no-stream    # One-time snapshot

# Step 2: Inspect container for memory limits
docker inspect <container> | grep -i memory

# Step 3: Set memory limit on restart
docker run -m 512m --memory-swap 512m myapp
# --memory-swap == memory means no swap

# Step 4: Update limits for running container (Docker 1.10+)
docker update --memory 512m --memory-swap 512m <container>

# Step 5: Set limits in Docker Compose
services:
  app:
    mem_limit: 512m
    mem_reservation: 256m    # Soft limit
    memswap_limit: 512m      # No swap
```

```bash
# Investigate what's consuming memory inside the container
docker exec -it <container> top
docker exec -it <container> cat /proc/meminfo
```

---

### 54. SCENARIO: How do you monitor Docker containers in production?

**Answer:**

**Built-in:**
```bash
docker stats                     # Live CPU, memory, network, I/O
docker events                    # Real-time event stream
docker logs --since 1h myapp     # Recent logs
```

**Prometheus + cAdvisor stack:**
```yaml
# docker-compose.yml
services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    ports:
      - "8080:8080"

  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
```

**Other tools:**
- **ELK Stack** — Elasticsearch, Logstash, Kibana for log aggregation
- **Datadog / New Relic** — Full APM and container monitoring
- **Portainer** — GUI for Docker management
- **Loki + Grafana** — Log aggregation lightweight alternative

---

### 55. SCENARIO: You accidentally deleted an important container. Can you recover data?

**Answer:**
```bash
# 1. Check if container still exists (even stopped)
docker ps -a | grep <container-name>

# 2. If container exists but stopped, you can still access files
docker start <container>
docker cp <container>:/app/data ./recovered-data

# 3. Copy from stopped container (without starting)
docker cp <stopped-container>:/app/data ./recovered-data

# 4. If container is fully removed but volume exists
docker volume ls    # Check if named volume still exists
docker run --rm -v <volume-name>:/data alpine tar czf - /data > backup.tar.gz

# 5. Inspect what volumes existed (from image metadata)
docker image inspect <image> | grep -A5 Volumes
```

**Lesson:** Always use **named volumes** not anonymous volumes. Named volumes persist even after `docker rm`.

---

### 56. SCENARIO: How do you copy files between host and container?

**Answer:**
```bash
# Copy from host to container
docker cp ./local-file.txt my-container:/app/file.txt
docker cp ./local-directory my-container:/app/

# Copy from container to host
docker cp my-container:/app/logs/error.log ./error.log
docker cp my-container:/app/data ./local-data/

# Copy works on running AND stopped containers

# Alternative: use volume mount for live sync (dev)
docker run -v $(pwd)/src:/app/src myapp

# Tar pipe for large directories
docker exec my-container tar czf - /app/data | tar xzf - -C ./local-backup/
```

---

### 57. SCENARIO: How do you do a health check for a service that doesn't have an HTTP endpoint?

**Answer:**
```dockerfile
# TCP check — is the port open?
HEALTHCHECK CMD nc -z localhost 5432 || exit 1

# Script-based check
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD pg_isready -U postgres || exit 1

# Redis check
HEALTHCHECK CMD redis-cli ping | grep PONG || exit 1

# File-based check (app writes a heartbeat file)
HEALTHCHECK CMD test -f /tmp/healthy || exit 1

# Database connectivity check
HEALTHCHECK CMD mysql -u root -p${MYSQL_ROOT_PASSWORD} -e "SELECT 1" || exit 1
```

In Docker Compose:
```yaml
healthcheck:
  test: ["CMD", "pg_isready", "-U", "postgres"]
  interval: 10s
  timeout: 5s
  retries: 5
  start_period: 30s
```

---

### 58. SCENARIO: Docker build fails due to network issues when running `apt-get install`. How do you fix?

**Answer:**
```bash
# Error: "Could not resolve 'archive.ubuntu.com'"

# Fix 1: Configure Docker DNS
# Edit /etc/docker/daemon.json
{
  "dns": ["8.8.8.8", "8.8.4.4"]
}
systemctl restart docker

# Fix 2: Use --network host during build
docker build --network=host -t myapp .

# Fix 3: Use a specific apt mirror
RUN sed -i 's/archive.ubuntu.com/mirrors.ubuntu.com/g' /etc/apt/sources.list

# Fix 4: Pre-download packages and COPY them
COPY ./packages/*.deb /tmp/
RUN dpkg -i /tmp/*.deb

# Fix 5: Use a corporate proxy
docker build \
  --build-arg HTTP_PROXY=http://proxy:3128 \
  --build-arg HTTPS_PROXY=http://proxy:3128 \
  -t myapp .
```

---

### 59. SCENARIO: How do you run a Docker container as a background service that restarts automatically?

**Answer:**
```bash
# Restart policies:
# no           = Never restart (default)
# on-failure   = Restart only on non-zero exit code
# always       = Always restart (even after docker restart)
# unless-stopped = Always restart unless manually stopped

docker run -d \
  --name my-service \
  --restart unless-stopped \
  -p 8080:80 \
  myapp:latest

# Update restart policy on existing container
docker update --restart unless-stopped my-container

# In Docker Compose
services:
  web:
    restart: unless-stopped
```

---

### 60. SCENARIO: How do you set up a private Docker registry?

**Answer:**
```bash
# Option 1: Simple registry (no auth)
docker run -d \
  --name registry \
  -p 5000:5000 \
  -v registry-data:/var/lib/registry \
  --restart always \
  registry:2

# Push to local registry
docker tag myapp localhost:5000/myapp:v1
docker push localhost:5000/myapp:v1
docker pull localhost:5000/myapp:v1

# Option 2: Registry with basic authentication
docker run -d \
  --name registry \
  -p 5000:5000 \
  -v auth:/auth \
  -e REGISTRY_AUTH=htpasswd \
  -e REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry:2

# Option 3: Harbor (enterprise-grade, recommended)
# - Role-based access control
# - Vulnerability scanning
# - Content trust
# - Multi-tenant
# Install via Helm: helm install harbor harbor/harbor

# Option 4: Cloud registries (managed, production-ready)
# AWS ECR, Google GCR, Azure ACR, GitHub GHCR
```

---

### 61. SCENARIO: How do you handle configuration that differs between environments in Docker?

**Answer:**

**Method 1 — Environment variables (12-factor app):**
```bash
docker run -e DB_HOST=prod-db -e LOG_LEVEL=warn myapp
docker run --env-file .env.production myapp
```

**Method 2 — Config files via volume mount:**
```bash
docker run -v ./config/prod.yml:/app/config.yml myapp
```

**Method 3 — Build args for compile-time config:**
```dockerfile
ARG ENV=production
RUN if [ "$ENV" = "development" ]; then npm install; else npm ci --only=production; fi
```
```bash
docker build --build-arg ENV=development -t myapp:dev .
```

**Method 4 — Docker Compose `.env` file:**
```bash
# .env file (auto-loaded by Compose)
NODE_ENV=production
DB_HOST=prod-db
TAG=v2.1.0
```
```yaml
services:
  web:
    image: myapp:${TAG}
    environment:
      - NODE_ENV=${NODE_ENV}
      - DB_HOST=${DB_HOST}
```

---

### 62. SCENARIO: How do you debug a Dockerfile build that fails midway?

**Answer:**
```bash
# Step 1: Note which step failed in the build output
# STEP 7/12 : RUN npm run build
# ERROR: ...

# Step 2: Run the last successful layer interactively
# Find the ID of the last successful layer
docker images    # Look for <none>:<none> layers

# Step 3: Use --target to build up to a specific stage
docker build --target builder -t debug-image .
docker run -it debug-image bash

# Step 4: Add debug RUN commands
RUN ls -la /app        # Check what's there
RUN cat /etc/os-release  # Check OS
RUN which node && node --version  # Check tool availability

# Step 5: Use BuildKit --progress=plain for full output
DOCKER_BUILDKIT=1 docker build --progress=plain -t myapp . 2>&1 | tee build.log

# Step 6: Check build context size
du -sh .   # If huge, check .dockerignore
```

---

### 63. SCENARIO: How do you clean up Docker resources to free disk space?

**Answer:**
```bash
# Check disk usage
docker system df -v

# Remove stopped containers
docker container prune

# Remove unused images (not referenced by any container)
docker image prune
docker image prune -a    # Remove ALL unused images (not just dangling)

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune

# Nuclear option: clean everything at once
docker system prune                         # containers + networks + dangling images
docker system prune -a                      # + all unused images
docker system prune -a --volumes            # + volumes (DATA LOSS!)
docker system prune -a --volumes --force    # No confirmation prompt

# Cleanup specific images older than 24h
docker image prune -a --filter "until=24h"

# Regular automated cleanup (add to cron)
0 2 * * * docker system prune -f >> /var/log/docker-cleanup.log 2>&1
```

---

### 64. SCENARIO: How do you push an image to AWS ECR?

**Answer:**
```bash
# Step 1: Authenticate Docker to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789.dkr.ecr.us-east-1.amazonaws.com

# Step 2: Create ECR repository (if not exists)
aws ecr create-repository \
  --repository-name my-app \
  --region us-east-1

# Step 3: Build and tag image
docker build -t my-app .
docker tag my-app:latest \
  123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
docker tag my-app:latest \
  123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:v1.2.3

# Step 4: Push to ECR
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:v1.2.3

# Step 5: Pull from ECR
docker pull 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:v1.2.3
```

---

### 65. SCENARIO: How do you implement a CI/CD pipeline using Docker?

**Answer:**

**GitHub Actions example:**
```yaml
name: Build, Test, Push

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Extract metadata (tags)
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: myuser/myapp
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=sha

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Run Trivy vulnerability scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myuser/myapp:${{ github.sha }}
          exit-code: '1'          # Fail pipeline on critical CVEs
          severity: 'CRITICAL'
```

---

*© Docker Interview Q&A — 65 Questions*
*Sections: Basics · Dockerfile · Networking · Storage · Docker Compose · Security · Advanced · 20 Scenarios*
