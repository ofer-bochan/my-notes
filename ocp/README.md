# OpenShift Container Platform Documentation

> **What is OpenShift?**  
> OpenShift Container Platform (OCP) is Red Hat's enterprise Kubernetes distribution with additional features for security, developer productivity, and operational management. It extends Kubernetes with built-in CI/CD, monitoring, logging, and service mesh capabilities.

## Topics

| File | Topic | Description |
|------|-------|-------------|
| [01-two-node-fencing.md](01-two-node-fencing.md) | Two-Node HA with Fencing | High availability cluster with Pacemaker fencing for edge deployments |

## Key Concepts

- **Control Plane**: Master nodes running API Server, etcd, Scheduler, Controllers
- **Compute Nodes**: Worker nodes running user workloads
- **Operators**: Kubernetes-native applications that extend cluster functionality
- **GitOps**: Declarative infrastructure management via Git repositories
- **Machine Config**: RHCOS configuration management via MachineConfig Operator

## OpenShift-Specific Features

| Feature | Description |
|---------|-------------|
| **Routes** | External access to services (alternative to Ingress) |
| **BuildConfigs** | Source-to-image builds, Dockerfile builds |
| **DeploymentConfigs** | Legacy deployment strategy (use Deployments now) |
| **Projects** | Namespaces with additional metadata and RBAC |
| **SCC** | Security Context Constraints for pod security |
| **OAuth** | Built-in identity provider integration |

## Quick Reference

```bash
# OpenShift CLI (oc) commands
oc login https://api.cluster.example.com:6443    # Login to cluster
oc new-project myproject                          # Create project
oc get clusterversion                             # Check cluster version
oc get clusteroperators                           # List cluster operators
oc adm node-logs <node> -u kubelet               # View node logs
oc debug node/<node>                             # Debug node shell
oc adm top nodes                                 # Node resource usage
oc get mcp                                       # MachineConfigPools
oc get mc                                        # MachineConfigs
```

## Installation Methods

| Method | Use Case |
|--------|----------|
| **IPI** (Installer-Provisioned) | Automated infrastructure provisioning |
| **UPI** (User-Provisioned) | Pre-existing infrastructure, custom networks |
| **Agent-Based** | Disconnected environments, bare metal |
| **Assisted Installer** | Web-based installation wizard |
| **SNO** | Single Node OpenShift for edge |
| **Two-Node** | HA with minimal footprint (Arbiter or Fencing) |


