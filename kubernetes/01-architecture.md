# Kubernetes Architecture Overview

> **What it is:** Kubernetes architecture consists of two main parts - the **Control Plane** (brain of the cluster that makes decisions) and the **Data Plane** (worker nodes that run your applications). The control plane manages the cluster state, while worker nodes execute the actual workloads.

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              CONTROL PLANE                                       │
│                                                                                  │
│   ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐            │
│   │  kube-apiserver │◄──►│  kube-scheduler │    │   controller    │            │
│   │                 │    │                 │    │    manager      │            │
│   │  - REST API     │    │  - Pod placing  │    │                 │            │
│   │  - AuthN/AuthZ  │    │  - Node scoring │    │  - Node ctrl    │            │
│   │  - Admission    │    │  - Affinity     │    │  - Replication  │            │
│   └────────┬────────┘    └─────────────────┘    │  - Endpoints    │            │
│            │                                     └─────────────────┘            │
│            ▼                                                                     │
│   ┌─────────────────┐                                                           │
│   │      etcd       │   Distributed key-value store - cluster state             │
│   └─────────────────┘                                                           │
└─────────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        │ HTTPS/gRPC
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                               DATA PLANE (Workers)                               │
│                                                                                  │
│  ┌───────────────────────────────────────────────────────────────────────────┐ │
│  │  WORKER NODE                                                               │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────────────┐   │ │
│  │  │   kubelet   │  │ kube-proxy  │  │     Container Runtime           │   │ │
│  │  │  (agent)    │  │ (networking)│  │     (CRI-O / containerd)        │   │ │
│  │  └─────────────┘  └─────────────┘  └─────────────────────────────────┘   │ │
│  └───────────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## Key Principles

| Principle | Description |
|-----------|-------------|
| **Declarative** | You declare desired state, Kubernetes makes it happen |
| **Control Loops** | Controllers continuously reconcile actual vs desired state |
| **API-Centric** | All operations go through the API server |
| **Distributed** | Components are loosely coupled, can run anywhere |

## Component Summary

### Control Plane Components

| Component | Role | What Happens If It Fails |
|-----------|------|-------------------------|
| **API Server** | Front door to cluster | No cluster operations possible |
| **etcd** | Cluster database | Cluster state is lost |
| **Scheduler** | Assigns pods to nodes | New pods stay pending |
| **Controller Manager** | Runs control loops | No automatic healing |

### Data Plane Components

| Component | Role | What Happens If It Fails |
|-----------|------|-------------------------|
| **Kubelet** | Node agent | Node marked NotReady |
| **Kube-proxy** | Network rules | Services don't work |
| **Container Runtime** | Runs containers | Containers can't start |

## Communication Flow

```
User Request: kubectl create deployment nginx --image=nginx

1. kubectl → API Server (authenticated, authorized)
2. API Server → etcd (store Deployment object)
3. Controller Manager sees new Deployment → creates ReplicaSet
4. Controller Manager sees ReplicaSet → creates Pod objects
5. Scheduler sees unscheduled Pods → assigns to nodes
6. Kubelet on node sees assigned Pod → starts containers
7. Containers running, Pod status updated
```

## See Also

- [Control Plane Components](02-control-plane.md)
- [Data Plane Components](03-data-plane.md)

