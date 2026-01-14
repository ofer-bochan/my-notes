# Kubernetes Data Plane Components

> **What it is:** The Data Plane consists of worker nodes that actually run your containerized applications. Each worker node has three essential components: kubelet (node agent), kube-proxy (networking), and a container runtime (runs containers).

## Components Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            WORKER NODE                                       │
│                                                                              │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────────┐ │
│  │    KUBELET      │  │   KUBE-PROXY    │  │    CONTAINER RUNTIME        │ │
│  │                 │  │                 │  │    (CRI-O/containerd)       │ │
│  │ • Node agent    │  │ • Network rules │  │                             │ │
│  │ • Pod lifecycle │  │ • Service IPs   │  │  ┌─────┐ ┌─────┐ ┌─────┐  │ │
│  │ • Health probes │  │ • Load balance  │  │  │ Pod │ │ Pod │ │ Pod │  │ │
│  │ • Volume mounts │  │ • iptables/IPVS │  │  └─────┘ └─────┘ └─────┘  │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 1. Kubelet

> **What it is:** Kubelet is the primary node agent that runs on every node. It watches for Pods assigned to its node by the API server, then ensures those containers are running and healthy. Think of it as the "manager" of a single node.

### Responsibilities

| Task | Description |
|------|-------------|
| **Pod Lifecycle** | Start, stop, restart containers based on pod specs |
| **Health Probes** | Run liveness, readiness, startup probes |
| **Volume Management** | Mount volumes into containers |
| **Resource Enforcement** | Enforce CPU/memory limits via cgroups |
| **Status Reporting** | Report node and pod status to API server |
| **Image Management** | Pull container images (via runtime) |

### Pod Lifecycle Flow

```
API Server: "Run this pod on your node"
                    │
                    ▼
              ┌──────────┐
              │  Kubelet │
              └────┬─────┘
                   │
    ┌──────────────┼──────────────┐
    │              │              │
    ▼              ▼              ▼
Pull Image    Create Pod     Mount Volumes
    │              │              │
    └──────────────┼──────────────┘
                   │
                   ▼
           Start Containers
                   │
                   ▼
           Run Health Probes ←──────┐
                   │                │
                   ▼                │
           Report Status ───────────┘
```

### Health Probes

| Probe Type | Purpose | On Failure |
|------------|---------|------------|
| **Startup** | Check if app has started | Block other probes |
| **Liveness** | Check if app is alive | Restart container |
| **Readiness** | Check if app is ready for traffic | Remove from Service |

```yaml
# Example: All three probe types
spec:
  containers:
  - name: app
    image: myapp
    
    startupProbe:           # Wait for slow-starting apps
      httpGet:
        path: /health
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
    
    livenessProbe:          # Restart if unhealthy
      httpGet:
        path: /health
        port: 8080
      periodSeconds: 10
      failureThreshold: 3
    
    readinessProbe:         # Don't send traffic until ready
      httpGet:
        path: /ready
        port: 8080
      periodSeconds: 5
```

---

## 2. Kube-proxy

> **What it is:** Kube-proxy runs on every node and maintains network rules that allow network communication to pods. It implements Kubernetes Services by managing iptables rules (or IPVS) to route traffic to the correct pods.

### How It Works

```
Client Pod wants to reach "my-service" (ClusterIP: 10.96.0.10)
                    │
                    ▼
┌─────────────────────────────────────────────────────────────┐
│                      KUBE-PROXY                              │
│                                                              │
│   iptables rules:                                           │
│   10.96.0.10:80 → DNAT to one of:                          │
│     • 10.0.0.5:8080 (pod-1)                                │
│     • 10.0.0.6:8080 (pod-2)                                │
│     • 10.0.0.7:8080 (pod-3)                                │
│                                                              │
│   Load balancing: Random selection                          │
└─────────────────────────────────────────────────────────────┘
                    │
                    ▼
              Backend Pod
```

### Proxy Modes

| Mode | Description | Best For |
|------|-------------|----------|
| **iptables** | Uses iptables rules | Default, small clusters |
| **IPVS** | Uses Linux IPVS | Large clusters (1000s of services) |
| **nftables** | Uses nftables | New, replacing iptables |

---

## 3. Container Runtime

> **What it is:** The container runtime is the software that actually runs containers. Kubernetes uses the Container Runtime Interface (CRI) to communicate with any compatible runtime. The two main options are containerd and CRI-O.

### Runtime Comparison

| Feature | CRI-O | containerd |
|---------|-------|------------|
| **Focus** | Kubernetes-only | General purpose |
| **Used by** | OpenShift, RHEL | Docker, most K8s distros |
| **Size** | Lightweight | Heavier |
| **Image format** | OCI only | OCI + Docker |

### CRI Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         Kubelet                              │
│                            │                                 │
│                            │ CRI (gRPC)                     │
│                            ▼                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           Container Runtime (CRI-O/containerd)       │   │
│  │                                                      │   │
│  │   • Pull images                                      │   │
│  │   • Create/start/stop containers                     │   │
│  │   • Manage container lifecycle                       │   │
│  └──────────────────────┬───────────────────────────────┘   │
│                         │                                    │
│                         │ OCI Runtime Spec                   │
│                         ▼                                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Low-level Runtime (runc)                │   │
│  │                                                      │   │
│  │   • Create namespaces                               │   │
│  │   • Set up cgroups                                  │   │
│  │   • Apply security (SELinux, seccomp)              │   │
│  │   • Start container process                         │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Runtime Commands

```bash
# List containers (works with CRI-O and containerd)
crictl ps

# List pods
crictl pods

# List images
crictl images

# View container logs
crictl logs <container-id>

# Exec into container
crictl exec -it <container-id> /bin/sh

# Pull image
crictl pull nginx:latest
```

---

## Troubleshooting Data Plane

```bash
# Check node status
kubectl get nodes -o wide
kubectl describe node <node-name>

# Check kubelet status
systemctl status kubelet
journalctl -u kubelet -f

# Check kubelet on a node (OpenShift)
oc debug node/<node-name>
chroot /host
journalctl -u kubelet --since "10 min ago"

# Check kube-proxy
kubectl logs -n kube-system -l k8s-app=kube-proxy

# Check container runtime
crictl info
crictl ps -a   # Include stopped containers
```

