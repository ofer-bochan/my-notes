# Container Fundamentals

> **What it is:** Containers are isolated processes that use Linux kernel features (namespaces and cgroups) to separate applications from each other and from the host system. Understanding these fundamentals helps you troubleshoot and secure containers.

## How Containers Work

Containers are NOT mini-VMs. They're regular Linux processes with isolation provided by:

1. **Namespaces** - What the process can SEE (isolation)
2. **Cgroups** - What the process can USE (resource limits)
3. **Union Filesystems** - Layered, efficient storage

---

## Namespaces (Isolation)

> **What it is:** Namespaces limit what a process can see. Each namespace type isolates a specific system resource, making the container think it's alone on the system.

| Namespace | Isolates | Example |
|-----------|----------|---------|
| **pid** | Process IDs | Container sees PID 1 for its main process |
| **net** | Network stack | Container has its own IP, ports, routes |
| **mnt** | Filesystem mounts | Container has its own root filesystem |
| **uts** | Hostname | Container can have different hostname |
| **ipc** | Inter-process communication | Isolated shared memory |
| **user** | User/group IDs | UID 0 in container ≠ UID 0 on host |

### Example: PID Namespace

```bash
# On host: see all processes
ps aux | wc -l
# 250

# Inside container: only container processes
docker exec nginx ps aux
# PID 1 nginx
# PID 20 nginx: worker
```

### Viewing Namespaces

```bash
# List namespaces of a process
ls -la /proc/<PID>/ns/

# Output:
lrwxrwxrwx 1 root root 0 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 ipc -> 'ipc:[4026532278]'
lrwxrwxrwx 1 root root 0 mnt -> 'mnt:[4026532276]'
lrwxrwxrwx 1 root root 0 net -> 'net:[4026532281]'
lrwxrwxrwx 1 root root 0 pid -> 'pid:[4026532279]'
lrwxrwxrwx 1 root root 0 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 uts -> 'uts:[4026532277]'

# Enter a container's namespace
nsenter -t <PID> -n ip addr    # Enter network namespace
```

---

## Cgroups (Resource Limits)

> **What it is:** Control Groups (cgroups) limit and track resource usage. They're how Kubernetes enforces CPU and memory limits, and how Docker restricts container resources.

| Controller | Controls | Example |
|------------|----------|---------|
| **cpu** | CPU time | Container gets 50% of one core |
| **cpuset** | CPU pinning | Container only uses cores 0,1 |
| **memory** | RAM limit | Container limited to 512MB |
| **blkio** | Disk I/O | Container limited to 100MB/s reads |
| **pids** | Process count | Container can only create 100 processes |

### Cgroups v2 Location

```bash
# All cgroups in unified hierarchy
ls /sys/fs/cgroup/

# Container's cgroup
cat /sys/fs/cgroup/system.slice/docker-<id>.scope/memory.max
# 536870912 (512MB in bytes)

cat /sys/fs/cgroup/system.slice/docker-<id>.scope/cpu.max
# 50000 100000 (50% of one CPU)
```

### Kubernetes Resource Limits Use Cgroups

```yaml
# This Pod definition:
resources:
  limits:
    memory: "512Mi"
    cpu: "500m"

# Creates these cgroup limits:
# memory.max = 536870912 (512MB)
# cpu.max = 50000 100000 (50% CPU)
```

---

## Union Filesystems (Layers)

> **What it is:** Container images are built in layers. Each Dockerfile instruction creates a new layer. Layers are read-only and shared between containers. Only the top layer is writable.

```
┌─────────────────────────────────────────────────────────────┐
│  Container (Read-Write layer)                                │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  /var/log/app.log (new file)                        │   │
│  │  /etc/config (modified file)                        │   │
│  └─────────────────────────────────────────────────────┘   │
│                        │                                     │
│                  Copy-on-Write                               │
│                        │                                     │
│  Image Layers (Read-Only)                                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Layer 3: COPY app.py (your code)                   │   │
│  └─────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Layer 2: RUN pip install (dependencies)            │   │
│  └─────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Layer 1: FROM python:3.11 (base image)             │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘

Key points:
- Layers are shared (saves disk space)
- Only changes are stored in new layers
- Container writes go to top writable layer
- Deleting a container only deletes the writable layer
```

### Viewing Layers

```bash
# See image layers
docker history nginx:latest

IMAGE          CREATED       CREATED BY                                      SIZE
a8b8d...       2 weeks ago   CMD ["nginx" "-g" "daemon off;"]               0B
<missing>      2 weeks ago   STOPSIGNAL SIGQUIT                             0B
<missing>      2 weeks ago   EXPOSE 80                                       0B
<missing>      2 weeks ago   ENTRYPOINT ["/docker-entrypoint.sh"]           0B
<missing>      2 weeks ago   COPY 30-tune-worker-processes.sh ...           4.62kB
<missing>      2 weeks ago   COPY 20-envsubst-on-templates.sh ...           3.02kB
```

---

## Container Runtime Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Docker/Podman                           │
│                           │                                  │
│                           │ API                              │
│                           ▼                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │          containerd / CRI-O                          │   │
│  │    (High-level runtime - manages lifecycle)          │   │
│  │                                                      │   │
│  │    - Pull images                                     │   │
│  │    - Create containers                               │   │
│  │    - Manage networking                               │   │
│  │    - Storage                                         │   │
│  └──────────────────────┬───────────────────────────────┘   │
│                         │                                    │
│                         │ OCI Runtime Spec                   │
│                         ▼                                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              runc / crun                             │   │
│  │    (Low-level runtime - sets up isolation)          │   │
│  │                                                      │   │
│  │    - Create namespaces                              │   │
│  │    - Configure cgroups                              │   │
│  │    - Set up filesystem                              │   │
│  │    - Apply security (SELinux, seccomp)             │   │
│  │    - Start process                                  │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## Hands-On Exploration

```bash
# See container's cgroup limits
docker run -d --name test --memory=256m nginx
cat /sys/fs/cgroup/system.slice/docker-$(docker inspect test --format '{{.Id}}').scope/memory.max

# See container's namespaces
docker inspect test --format '{{.State.Pid}}'
ls -la /proc/<PID>/ns/

# See container's processes from host
docker top test

# See overlay mount
docker inspect test --format '{{.GraphDriver.Data.MergedDir}}'
ls /var/lib/docker/overlay2/<id>/merged/
```

