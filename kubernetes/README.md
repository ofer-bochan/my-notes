# Kubernetes Documentation

> **What is Kubernetes?**  
> Kubernetes (K8s) is an open-source container orchestration platform originally designed by Google. It automates the deployment, scaling, and management of containerized applications across clusters of machines. Kubernetes provides a framework for running distributed systems resiliently, handling failover, scaling, and deployment patterns.

## Topics

| File | Topic | Description |
|------|-------|-------------|
| [01-architecture.md](01-architecture.md) | Architecture Overview | How Kubernetes is structured - control plane, data plane |
| [02-control-plane.md](02-control-plane.md) | Control Plane | API Server, etcd, Scheduler, Controller Manager |
| [03-data-plane.md](03-data-plane.md) | Data Plane | Kubelet, kube-proxy, Container Runtime |
| [04-networking.md](04-networking.md) | Networking | Pod networking, Services, DNS |
| [05-openshift.md](05-openshift.md) | OpenShift | Red Hat's enterprise Kubernetes platform |
| [06-troubleshooting.md](06-troubleshooting.md) | Troubleshooting | Debug commands and techniques |
| [07-must-gather.md](07-must-gather.md) | must-gather | OpenShift diagnostic data collection |

## Key Concepts

- **Pod**: Smallest deployable unit, one or more containers
- **Service**: Stable network endpoint for accessing pods
- **Deployment**: Manages pod replicas and updates
- **Namespace**: Virtual cluster for resource isolation
- **ConfigMap/Secret**: Configuration and sensitive data storage

## Quick Reference

```bash
# Common kubectl commands
kubectl get pods                    # List pods
kubectl get pods -A                 # All namespaces
kubectl describe pod <name>         # Pod details
kubectl logs <pod>                  # View logs
kubectl exec -it <pod> -- /bin/sh   # Shell into pod
kubectl apply -f manifest.yaml      # Apply configuration
kubectl delete pod <name>           # Delete pod
```

