# Kubernetes Networking

> **What it is:** Kubernetes networking provides a flat network where every pod gets its own IP address and can communicate with any other pod without NAT. Services provide stable endpoints for accessing groups of pods. CNI plugins implement the actual networking.

## Core Networking Rules

| Rule | Description |
|------|-------------|
| Every pod gets a unique IP | No need to map ports |
| Pods can reach all other pods | Without NAT, using pod IPs |
| Nodes can reach all pods | Direct communication |
| Pod sees same IP internally | No translation confusion |

---

## Pod Networking

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Pod-to-Pod Communication                            │
│                                                                              │
│  Node 1 (192.168.1.10)               Node 2 (192.168.1.11)                  │
│  ┌────────────────────┐              ┌────────────────────┐                 │
│  │ ┌────────────────┐ │              │ ┌────────────────┐ │                 │
│  │ │ Pod A          │ │              │ │ Pod B          │ │                 │
│  │ │ 10.244.1.5     │ │              │ │ 10.244.2.3     │ │                 │
│  │ └───────┬────────┘ │              │ └───────▲────────┘ │                 │
│  │         │ veth     │              │         │ veth     │                 │
│  │ ┌───────▼────────┐ │              │ ┌───────┴────────┐ │                 │
│  │ │  cni0 bridge   │ │              │ │  cni0 bridge   │ │                 │
│  │ │  10.244.1.1    │ │              │ │  10.244.2.1    │ │                 │
│  │ └───────┬────────┘ │              │ └───────▲────────┘ │                 │
│  └─────────┼──────────┘              └─────────┼──────────┘                 │
│            │                                   │                             │
│            └───────────── Network ─────────────┘                             │
│                  (overlay/underlay)                                          │
└─────────────────────────────────────────────────────────────────────────────┘

Traffic: Pod A (10.244.1.5) → Pod B (10.244.2.3)
- Pod A sends packet to its gateway (cni0 bridge)
- Network routes packet to Node 2
- Node 2's bridge delivers to Pod B
```

---

## Services

> **What it is:** A Service provides a stable IP address and DNS name for accessing a group of pods. As pods come and go, the Service maintains a consistent endpoint for clients.

### Service Types

| Type | Description | Use Case |
|------|-------------|----------|
| **ClusterIP** | Internal IP only | Internal services |
| **NodePort** | Port on every node | Simple external access |
| **LoadBalancer** | External load balancer | Production external access |
| **ExternalName** | DNS CNAME | External services |

### Service Example

```yaml
# Scenario: Expose nginx pods internally

apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP              # Internal only (default)
  selector:
    app: nginx                 # Find pods with this label
  ports:
  - port: 80                   # Service port
    targetPort: 8080           # Pod port
```

```
┌─────────────────────────────────────────────────────────────┐
│                      nginx-service                           │
│                   ClusterIP: 10.96.10.50                    │
│                                                              │
│   Selector: app=nginx                                        │
│                    │                                         │
│      ┌─────────────┼─────────────┐                          │
│      ▼             ▼             ▼                          │
│  ┌───────┐    ┌───────┐    ┌───────┐                       │
│  │ nginx │    │ nginx │    │ nginx │                       │
│  │ pod-1 │    │ pod-2 │    │ pod-3 │                       │
│  │ :8080 │    │ :8080 │    │ :8080 │                       │
│  └───────┘    └───────┘    └───────┘                       │
│                                                              │
│  Any pod can reach: nginx-service:80                        │
│  or: nginx-service.namespace.svc.cluster.local:80          │
└─────────────────────────────────────────────────────────────┘
```

---

## DNS

> **What it is:** CoreDNS provides DNS service discovery within the cluster. Every Service automatically gets a DNS name, so pods can find services by name instead of IP address.

### DNS Names

```
<service>.<namespace>.svc.cluster.local

Examples:
- nginx.default.svc.cluster.local
- database.production.svc.cluster.local
- my-service.my-namespace.svc.cluster.local

Short names (within same namespace):
- nginx
- database
```

### DNS Resolution

```bash
# From a pod, you can reach services by:
curl http://my-service                        # Same namespace
curl http://my-service.other-namespace        # Different namespace
curl http://my-service.other-namespace.svc.cluster.local  # Full name
```

---

## CNI Plugins

> **What it is:** Container Network Interface (CNI) plugins implement the actual pod networking. Different plugins offer different features like network policies, encryption, or performance optimizations.

| Plugin | Features | Used By |
|--------|----------|---------|
| **Calico** | Network policies, BGP | Many distros |
| **Cilium** | eBPF, security, observability | High performance needs |
| **Flannel** | Simple overlay | Simple clusters |
| **OVN-Kubernetes** | Open Virtual Network | OpenShift |
| **Weave** | Simple, encrypted | Development |

---

## Network Policies

> **What it is:** Network Policies control traffic flow between pods. By default, all pods can communicate. Policies let you restrict this - like firewall rules for your cluster.

```yaml
# Example: Only allow traffic from frontend to backend

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      app: backend           # Apply to backend pods
  
  policyTypes:
  - Ingress
  
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend      # Only allow from frontend
    ports:
    - port: 8080
```

---

## Quick Reference

```bash
# View services
kubectl get svc
kubectl get svc -A              # All namespaces

# View endpoints (pod IPs behind service)
kubectl get endpoints <service>

# Test DNS
kubectl run tmp --image=busybox --rm -it -- nslookup my-service

# Debug networking
kubectl run tmp --image=nicolaka/netshoot --rm -it -- /bin/bash
# Then use: curl, ping, dig, tcpdump, etc.
```

