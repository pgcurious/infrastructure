# Chapter 5: The Container Revolution

*Problem: VMs solved the utilization crisis, but each VM runs a complete operating system. When you're deploying microservices—perhaps dozens of small applications—the overhead of a full OS per service becomes painful. Boot times are slow. Memory is wasted. There must be a lighter way.*

---

## The Weight of Virtual Machines

Let's revisit what a VM actually contains:

```
┌─────────────────────────────────────┐
│         Your Application            │  ~100 MB
│            (Java JAR)               │
├─────────────────────────────────────┤
│         JVM Runtime                 │  ~200 MB
├─────────────────────────────────────┤
│         Dependencies                │  ~500 MB
│   (libraries, tools you need)       │
├─────────────────────────────────────┤
│      Guest Operating System         │  ~1-4 GB
│   (kernel, init, system services)   │
├─────────────────────────────────────┤
│         Virtual Hardware            │
│   (emulated CPU, RAM, disk, NIC)    │
└─────────────────────────────────────┘
         Total: 2-5 GB per VM
         Boot time: 30-60 seconds
```

If your microservice needs 100 MB of memory to run, you're carrying 2+ GB of baggage.

What if you could skip the guest operating system entirely?

## Containers: OS-Level Virtualization

A **container** is an isolated environment that shares the host operating system's kernel. It provides process isolation without the overhead of a complete OS.

```
┌─────────────────────────────────────┐
│         Your Application            │  ~100 MB
│            (Java JAR)               │
├─────────────────────────────────────┤
│         JVM Runtime                 │  ~200 MB
├─────────────────────────────────────┤
│         Dependencies                │  ~100 MB
│   (only what your app needs)        │
└─────────────────────────────────────┘
     Total: 200-400 MB per container
     Start time: <1 second
```

No guest kernel. No system services you don't need. Just your application and its dependencies.

### How Containers Achieve Isolation

Containers use Linux kernel features to isolate processes:

**Namespaces**: Partition kernel resources so each container sees its own isolated view of:
- Process IDs (PID namespace)
- Network interfaces (Network namespace)
- File system mounts (Mount namespace)
- User IDs (User namespace)
- Hostnames (UTS namespace)

**Control Groups (cgroups)**: Limit and account for resource usage:
- CPU time
- Memory
- Disk I/O
- Network bandwidth

**Union File Systems**: Layer file systems on top of each other. Changes in a container don't affect the base image—they go into a new layer.

```
Container's View:           Reality:
┌─────────────────┐         ┌─────────────────┐
│   /app          │         │  Container Layer │ (read-write)
│   /etc          │  ───►   ├─────────────────┤
│   /usr          │         │   Image Layer 3  │ (read-only)
│   /bin          │         ├─────────────────┤
└─────────────────┘         │   Image Layer 2  │ (read-only)
Looks like a                ├─────────────────┤
complete filesystem         │   Image Layer 1  │ (read-only)
                            └─────────────────┘
                            Layered filesystem
```

> **Think About It:** If containers share the host kernel, what does that mean for running Windows containers on Linux? Or Linux containers on Windows?

## Docker: Containers Made Accessible

Container technology existed before Docker (LXC, Solaris Zones, FreeBSD Jails). But Docker, released in 2013, made containers accessible to everyone.

Docker provided:
- **Simple CLI**: `docker run nginx` instead of complex namespace configuration
- **Dockerfile**: A recipe for building images, version-controllable
- **Docker Hub**: A registry to share images publicly
- **Portable images**: Build once, run anywhere Docker runs

### The Dockerfile: A Recipe

A Dockerfile describes how to build an image:

```dockerfile
# Start from an official base image
FROM eclipse-temurin:17-jdk-alpine

# Set working directory
WORKDIR /app

# Copy application JAR
COPY target/myapp.jar app.jar

# Expose the port your app listens on
EXPOSE 8080

# Command to run when container starts
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Build and run:
```bash
# Build the image
docker build -t myapp:1.0 .

# Run a container from the image
docker run -p 8080:8080 myapp:1.0
```

### Image vs. Container: Recipe vs. Running Kitchen

This distinction is crucial:

**Image**: A read-only template with your application and dependencies. Like a recipe or a class definition.
- Built from a Dockerfile
- Stored in a registry (Docker Hub, ECR, GCR)
- Has a tag (e.g., `myapp:1.0`, `myapp:latest`)
- Immutable—you don't modify images, you build new ones

**Container**: A running instance of an image. Like a cooked meal or an object instance.
- Created from an image
- Has a read-write layer for runtime changes
- Has a lifecycle (created, running, stopped, removed)
- Ephemeral—data in containers is lost when they're removed

```
Image (myapp:1.0)
       │
       │ docker run
       ▼
┌─────────────────┐
│   Container A   │  (instance 1, running)
└─────────────────┘
       │
       │ docker run (again)
       ▼
┌─────────────────┐
│   Container B   │  (instance 2, running)
└─────────────────┘

Same image, different containers
```

## Container Lifecycle

```bash
# Pull an image from registry
docker pull nginx:latest

# Create and start a container
docker run -d --name web nginx:latest

# View running containers
docker ps

# View all containers (including stopped)
docker ps -a

# Stop a container
docker stop web

# Start a stopped container
docker start web

# Remove a container
docker rm web

# Remove an image
docker rmi nginx:latest
```

## Volumes: Persistent Data

Containers are ephemeral. When a container is removed, its data is gone. But what about databases? Log files? Uploaded content?

**Volumes** provide persistent storage that survives container lifecycles.

```bash
# Create a named volume
docker volume create mydata

# Mount volume in container
docker run -v mydata:/app/data myapp:1.0

# Or use bind mount (host directory)
docker run -v /host/path:/container/path myapp:1.0
```

```
┌─────────────────────────────────┐
│     Container (ephemeral)       │
│  ┌───────────────────────────┐  │
│  │  /app/data ───────────────┼──┼───► Volume (persistent)
│  │  /app/logs ───────────────┼──┼───► Host: /var/log/myapp
│  │  /app/code                │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

### Volume Types

| Type | Use Case |
|------|----------|
| **Named Volume** | Docker manages the location; easy backup/restore |
| **Bind Mount** | Mount specific host directory; good for development |
| **tmpfs Mount** | In-memory storage; fast, but data lost on container stop |

## Container Networking

Containers need to communicate—with each other and the outside world.

### Network Modes

**Bridge (default)**: Containers get IPs on a virtual network. They can communicate with each other by name or IP.

```bash
# Create a network
docker network create mynet

# Run containers on that network
docker run -d --name db --network mynet postgres
docker run -d --name app --network mynet myapp:1.0

# Inside 'app' container, 'db' hostname resolves
# jdbc:postgresql://db:5432/mydb
```

**Host**: Container shares the host's network stack. No isolation, but maximum performance.

```bash
docker run --network host myapp:1.0
# App is directly accessible on host's IP
```

**None**: No networking. Container is completely isolated.

### Port Mapping

To expose a container to the outside world:

```bash
# Map host port 80 to container port 8080
docker run -p 80:8080 myapp:1.0

# Now: http://host-ip:80 → container:8080
```

```
            Internet
                │
         ┌──────▼──────┐
         │    Host     │
         │  port 80    │
         └──────┬──────┘
                │ NAT
         ┌──────▼──────┐
         │  Container  │
         │  port 8080  │
         └─────────────┘
```

## Container Registries

Images need a home. A **container registry** stores and distributes images.

**Public registries**:
- Docker Hub (hub.docker.com)
- GitHub Container Registry (ghcr.io)

**Private registries**:
- AWS ECR (Elastic Container Registry)
- Google Container Registry (GCR)
- Self-hosted (Harbor, Nexus)

```bash
# Tag for a registry
docker tag myapp:1.0 mycompany.azurecr.io/myapp:1.0

# Push to registry
docker push mycompany.azurecr.io/myapp:1.0

# Pull from registry (on production server)
docker pull mycompany.azurecr.io/myapp:1.0
```

## Containers in Practice: A Spring Boot Example

Here's a complete Dockerfile for a Spring Boot application:

```dockerfile
# Build stage
FROM eclipse-temurin:17-jdk-alpine AS builder
WORKDIR /app
COPY . .
RUN ./mvnw package -DskipTests

# Runtime stage
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

# Create non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Copy JAR from builder
COPY --from=builder /app/target/*.jar app.jar

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget -q --spider http://localhost:8080/actuator/health || exit 1

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

This Dockerfile demonstrates:
- **Multi-stage build**: Build with JDK, run with smaller JRE
- **Non-root user**: Security best practice
- **Health check**: Orchestrators can verify container health
- **Layer optimization**: Copy dependencies before code for better caching

```bash
# Build
docker build -t mycompany/orders-service:1.0 .

# Run with environment variables
docker run -d \
  --name orders \
  -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=production \
  -e DATABASE_URL=jdbc:postgresql://db:5432/orders \
  mycompany/orders-service:1.0
```

## When Containers Aren't the Right Choice

Containers are powerful, but not universal:

- **GUI applications**: Containers are designed for server workloads
- **Windows-kernel-dependent software**: Can't run on Linux hosts
- **Maximum kernel isolation**: For true multi-tenant security, VMs may be required
- **Legacy applications** that expect a full OS environment

> **Think About It:** Why do many organizations run containers inside VMs in the cloud? What does each layer provide?

---

## Key Takeaways

1. **Containers share the host kernel** - This eliminates OS overhead and enables sub-second startup times. But it also means containers provide less isolation than VMs and can only run workloads compatible with the host kernel.

2. **Images are immutable recipes; containers are running instances** - Build images through Dockerfiles, store them in registries, and run them as containers. Treat containers as disposable—any data you care about must be in volumes.

3. **Container networking and storage require explicit configuration** - Unlike VMs, containers don't get their own filesystem or network by default. Use volumes for persistence and Docker networks for service discovery.

---

*Next Chapter: [Orchestration](06-orchestration.md) - Managing hundreds of containers without losing your mind*
