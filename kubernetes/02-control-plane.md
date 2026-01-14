# Kubernetes Control Plane Components

> **What it is:** The Control Plane is the "brain" of Kubernetes. It makes all decisions about the cluster - what runs where, how to respond to changes, and maintaining desired state. In production, control plane components run on dedicated master nodes for high availability.

## Components Overview

| Component | What It Does | Analogy |
|-----------|--------------|---------|
| **kube-apiserver** | Gateway to cluster, validates and processes all requests | Reception desk |
| **etcd** | Stores all cluster data | Database/filing cabinet |
| **kube-scheduler** | Decides which node runs each pod | Job placement officer |
| **kube-controller-manager** | Runs background control loops | Building maintenance |
| **cloud-controller-manager** | Manages cloud provider resources | Cloud liaison |

---

## 1. kube-apiserver

> **What it is:** The API Server is the central hub of Kubernetes - the only component that directly communicates with etcd. Every operation (kubectl, controllers, kubelet) goes through the API server. It handles authentication, authorization, and validation.

### Key Functions

| Function | Description |
|----------|-------------|
| **REST API** | Exposes Kubernetes API (pods, services, etc.) |
| **Authentication** | Validates WHO you are (certs, tokens) |
| **Authorization** | Validates WHAT you can do (RBAC) |
| **Admission Control** | Validates/modifies requests before storage |
| **etcd Gateway** | Only component that talks to etcd |

### Request Flow

```
kubectl get pods
      │
      ▼
┌─────────────────────────────────────────────────────────────┐
│                      API SERVER                              │
│                                                              │
│  1. Authentication  ─── "Who is this?" (cert/token)         │
│          │                                                   │
│          ▼                                                   │
│  2. Authorization   ─── "Can they do this?" (RBAC)          │
│          │                                                   │
│          ▼                                                   │
│  3. Admission       ─── "Is request valid? Modify?"         │
│          │               (Mutating → Validating)            │
│          ▼                                                   │
│  4. etcd            ─── Read/Write data                     │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### RBAC Example

```yaml
# Role: Define what actions are allowed
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

---
# RoleBinding: Assign role to user
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: production
subjects:
- kind: User
  name: alice
roleRef:
  kind: Role
  name: pod-reader
```

---

## 2. etcd

> **What it is:** etcd is a distributed, reliable key-value store that holds ALL cluster state. Every Kubernetes object (pods, services, secrets) is stored in etcd. It uses the Raft consensus algorithm for high availability across multiple members.

### What's Stored

```
/registry/pods/default/nginx-pod
/registry/services/default/kubernetes
/registry/secrets/default/my-secret
/registry/deployments/production/web-app
```

### High Availability

```
┌─────────────────────────────────────────────────────────────┐
│                   etcd Cluster (3 members)                   │
│                                                              │
│   ┌────────┐      ┌────────┐      ┌────────┐               │
│   │ etcd-1 │◄────►│ etcd-2 │◄────►│ etcd-3 │               │
│   │(Leader)│      │(Follow)│      │(Follow)│               │
│   └────────┘      └────────┘      └────────┘               │
│                                                              │
│   Quorum: 2 of 3 must agree for writes                      │
│   Can survive: 1 node failure                                │
└─────────────────────────────────────────────────────────────┘
```

### Critical Operations

```bash
# Check cluster health
etcdctl endpoint health --cluster

# Backup (DO THIS REGULARLY!)
etcdctl snapshot save /backup/etcd-$(date +%Y%m%d).db

# Restore from backup
etcdctl snapshot restore /backup/etcd-backup.db
```

---

## 3. kube-scheduler

> **What it is:** The Scheduler watches for newly created pods that have no node assigned, and selects a node for them to run on. It considers resource requirements, constraints, affinity rules, and data locality when making decisions.

### Scheduling Process

```
New Pod Created (no node assigned)
         │
         ▼
┌─────────────────────────────────────────────────────────────┐
│  1. FILTER - Remove nodes that CAN'T run the pod            │
│     - Not enough CPU/memory? ❌                              │
│     - Taints not tolerated? ❌                               │
│     - Node selector mismatch? ❌                             │
└─────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────┐
│  2. SCORE - Rank remaining nodes (0-100)                    │
│     - More free resources = higher score                    │
│     - Pod affinity matches = higher score                   │
│     - Image already present = higher score                  │
└─────────────────────────────────────────────────────────────┘
         │
         ▼
   Select highest score → Bind pod to node
```

### Scheduling Controls

| Control | Set On | Purpose |
|---------|--------|---------|
| **nodeSelector** | Pod | Simple label matching |
| **nodeAffinity** | Pod | Advanced node selection rules |
| **podAffinity** | Pod | Run near specific pods |
| **podAntiAffinity** | Pod | Run away from specific pods |
| **taints** | Node | Repel pods unless tolerated |
| **tolerations** | Pod | Accept tainted nodes |

---

## 4. kube-controller-manager

> **What it is:** The Controller Manager runs a collection of controller loops that watch the cluster state and make changes to move toward the desired state. Each controller handles a specific type of resource.

### Controller Pattern

```
        ┌──────────────────────────────────────────────────────┐
        │                   CONTROL LOOP                        │
        │                                                       │
        │    ┌───────┐     ┌─────────┐     ┌─────────┐        │
        │    │ Watch │────►│ Compare │────►│  Act    │        │
        │    │ State │     │ Actual  │     │ (fix    │        │
   ─────┼───►│       │     │   vs    │     │  diff)  │────────┼───►
        │    │       │     │ Desired │     │         │        │
        │    └───────┘     └─────────┘     └─────────┘        │
        │                                                       │
        └──────────────────────────────────────────────────────┘

Example: ReplicaSet Controller
- Watch: ReplicaSet says replicas=3
- Compare: Only 2 pods running
- Act: Create 1 more pod
```

### Key Controllers

| Controller | What It Manages |
|------------|-----------------|
| **Node Controller** | Monitors nodes, marks unhealthy, evicts pods |
| **ReplicaSet Controller** | Ensures correct number of pod replicas |
| **Deployment Controller** | Manages rollouts and rollbacks |
| **Service Controller** | Creates cloud load balancers |
| **Endpoint Controller** | Updates service endpoints |
| **Job Controller** | Runs pods to completion |
| **CronJob Controller** | Creates jobs on schedule |

---

## Troubleshooting

```bash
# Check control plane pods
kubectl get pods -n kube-system

# API server logs
kubectl logs -n kube-system kube-apiserver-<node>

# Scheduler logs (why pod not scheduled?)
kubectl logs -n kube-system kube-scheduler-<node>

# Controller manager logs
kubectl logs -n kube-system kube-controller-manager-<node>

# etcd health
kubectl get pods -n kube-system | grep etcd
```

