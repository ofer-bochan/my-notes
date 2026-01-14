# Containers & Docker Documentation

> **What are Containers?**  
> Containers are lightweight, isolated environments for running applications. They package an application with all its dependencies, ensuring it runs the same way everywhere. Unlike virtual machines, containers share the host's kernel, making them fast and efficient.

## Key Concepts

| Concept | Description |
|---------|-------------|
| **Image** | Read-only template for creating containers |
| **Container** | Running instance of an image |
| **Dockerfile** | Instructions for building an image |
| **Registry** | Storage for container images |
| **Layer** | Each instruction in Dockerfile creates a layer |

## Topics

| File | Topic | Description |
|------|-------|-------------|
| [01-fundamentals.md](01-fundamentals.md) | Fundamentals | Namespaces, cgroups, how containers work |
| [03-registries.md](03-registries.md) | Registries | Docker Hub, Quay.io, private registries, push/pull |
| [04-dockerfiles.md](04-dockerfiles.md) | Dockerfiles | Example Dockerfiles for Nginx, MySQL, Apache, Grafana, Kibana |
| [05-salt.md](05-salt.md) | Salt/SaltStack | Container management with Salt, states, orchestration |

## Container vs VM

```
┌─────────────────────┐   ┌─────────────────────┐
│    CONTAINER        │   │   VIRTUAL MACHINE   │
├─────────────────────┤   ├─────────────────────┤
│ App + Dependencies  │   │ App + Dependencies  │
│ (MBs, seconds)      │   │ Full Guest OS       │
├─────────────────────┤   │ (GBs, minutes)      │
│ Container Runtime   │   ├─────────────────────┤
├─────────────────────┤   │    Hypervisor       │
│    Host OS          │   ├─────────────────────┤
├─────────────────────┤   │    Host OS          │
│    Hardware         │   ├─────────────────────┤
└─────────────────────┘   │    Hardware         │
                          └─────────────────────┘
```

## Quick Reference

```bash
# Run a container
docker run -d -p 8080:80 nginx

# Build an image
docker build -t myapp:1.0 .

# List containers
docker ps -a

# View logs
docker logs -f <container>

# Shell into container
docker exec -it <container> /bin/sh

# Push to registry
docker push myuser/myapp:1.0
```

## Tools Comparison

| Tool | Description | Root Required |
|------|-------------|---------------|
| **Docker** | Most popular, daemon-based | Yes (usually) |
| **Podman** | Daemonless, rootless, drop-in replacement | No |
| **Buildah** | Build OCI images | No |
| **Skopeo** | Copy/inspect images between registries | No |

