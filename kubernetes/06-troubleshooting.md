# Kubernetes Troubleshooting

> **What it is:** A collection of commands and techniques for debugging Kubernetes clusters. Covers common issues with pods, nodes, networking, and cluster components.

## Troubleshooting Workflow

```
Problem Detected
      │
      ▼
┌─────────────────────────────────────────────────────────────┐
│ 1. Check Events                                              │
│    kubectl get events --sort-by='.lastTimestamp'            │
└───────────────────────┬─────────────────────────────────────┘
                        │
      ▼─────────────────┴─────────────────▼
┌─────────────────────────┐  ┌─────────────────────────────────┐
│ 2. Pod Issues?          │  │ 3. Node Issues?                 │
│    kubectl describe pod │  │    kubectl describe node        │
│    kubectl logs         │  │    kubectl get nodes            │
└─────────────────────────┘  └─────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. Component Issues?                                         │
│    Check control plane pods/logs                            │
└─────────────────────────────────────────────────────────────┘
```

---

## Pod Troubleshooting

### Pod Status Reference

| Status | Meaning | Action |
|--------|---------|--------|
| **Pending** | Not scheduled yet | Check resources, node selector, taints |
| **ContainerCreating** | Setting up | Check events, image pull |
| **Running** | Containers are up | Check readiness, logs |
| **CrashLoopBackOff** | Keeps crashing | Check logs, command, resources |
| **ImagePullBackOff** | Can't pull image | Check image name, registry auth |
| **Completed** | Finished (Job) | Normal for Jobs |
| **Error** | Container error | Check logs |

### Commands

```bash
# Get pod status
kubectl get pods -o wide
kubectl get pods -A                    # All namespaces

# Detailed pod info (MOST USEFUL!)
kubectl describe pod <pod-name>
# Look at: Events section at bottom

# Pod logs
kubectl logs <pod>
kubectl logs <pod> -c <container>      # Specific container
kubectl logs <pod> --previous          # Previous crash
kubectl logs <pod> -f                  # Follow

# Shell into pod
kubectl exec -it <pod> -- /bin/sh
kubectl exec -it <pod> -c <container> -- /bin/bash

# Debug running pod (copy with debug tools)
kubectl debug <pod> -it --image=busybox
```

### Common Pod Issues

**Issue: Pod stuck in Pending**
```bash
# Check why
kubectl describe pod <pod> | grep -A 20 Events

# Common causes:
# - "Insufficient cpu/memory" → Node doesn't have resources
# - "0/3 nodes available: 1 Taints" → Node taints not tolerated
# - "no nodes match pod selector" → nodeSelector mismatch
```

**Issue: CrashLoopBackOff**
```bash
# Check logs from crashed container
kubectl logs <pod> --previous

# Common causes:
# - Application error (check logs)
# - Missing config/secrets
# - Wrong command
# - OOMKilled (out of memory)
```

**Issue: ImagePullBackOff**
```bash
# Check events
kubectl describe pod <pod> | grep -i image

# Common causes:
# - Image name typo
# - Image doesn't exist
# - Private registry needs imagePullSecrets
```

---

## Node Troubleshooting

```bash
# List nodes
kubectl get nodes -o wide

# Node details
kubectl describe node <node>

# Check node conditions
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.conditions[?(@.type=="Ready")].status}{"\n"}{end}'

# What's running on a node
kubectl get pods --field-selector spec.nodeName=<node> -A

# Resource usage
kubectl top nodes
kubectl describe node <node> | grep -A 10 "Allocated resources"
```

### Node Conditions

| Condition | Description |
|-----------|-------------|
| Ready=True | Node is healthy |
| Ready=False | Node has problems |
| MemoryPressure | Low memory |
| DiskPressure | Low disk space |
| PIDPressure | Too many processes |
| NetworkUnavailable | Network not configured |

---

## Control Plane Troubleshooting

### OpenShift

```bash
# Cluster operators (overall health)
oc get clusteroperators
oc get co                              # Short form

# Find broken operators
oc get co | grep -v "True.*False.*False"

# Operator details
oc describe co <operator>

# Control plane pods
oc get pods -n openshift-kube-apiserver
oc get pods -n openshift-kube-scheduler
oc get pods -n openshift-kube-controller-manager
oc get pods -n openshift-etcd

# API server logs
oc logs -n openshift-kube-apiserver kube-apiserver-<node>
```

### Generic Kubernetes

```bash
# Control plane pods
kubectl get pods -n kube-system

# API server health
kubectl get --raw='/readyz?verbose'

# Component logs
kubectl logs -n kube-system kube-apiserver-<master>
kubectl logs -n kube-system kube-scheduler-<master>
kubectl logs -n kube-system kube-controller-manager-<master>
```

### etcd

```bash
# OpenShift etcd status
oc get etcd cluster -o yaml
oc get pods -n openshift-etcd

# etcd health check
oc rsh -n openshift-etcd etcd-<master>
etcdctl endpoint health --cluster
etcdctl member list
```

---

## Networking Troubleshooting

```bash
# Check service endpoints
kubectl get endpoints <service>
# Empty endpoints = no pods match selector

# DNS test
kubectl run tmp --image=busybox --rm -it -- nslookup kubernetes

# Connectivity test
kubectl run tmp --image=busybox --rm -it -- wget -qO- http://my-service

# Check network policies
kubectl get networkpolicies -A

# kube-proxy logs
kubectl logs -n kube-system -l k8s-app=kube-proxy

# Debug with network tools
kubectl run debug --image=nicolaka/netshoot --rm -it -- /bin/bash
# Then: curl, dig, ping, tcpdump, etc.
```

---

## Events

```bash
# All events (sorted by time)
kubectl get events --sort-by='.lastTimestamp'

# Events in a namespace
kubectl get events -n <namespace>

# Only warnings
kubectl get events --field-selector type=Warning

# Events for specific resource
kubectl get events --field-selector involvedObject.name=<pod-name>
```

---

## Resource Usage

```bash
# Node resources
kubectl top nodes

# Pod resources
kubectl top pods
kubectl top pods -A                    # All namespaces

# Container resources
kubectl top pods --containers
```

---

## Quick Debug Commands

```bash
# Why can't I schedule?
kubectl describe pod <pod> | tail -20

# What's on this node?
kubectl get pods -A --field-selector spec.nodeName=<node>

# What just happened?
kubectl get events --sort-by='.lastTimestamp' | tail -20

# Is my service working?
kubectl get endpoints <service>

# What's broken?
kubectl get pods -A | grep -v Running
kubectl get pods -A | grep -v "1/1\|2/2\|3/3"

# Cluster overview
kubectl cluster-info
kubectl get nodes
kubectl get pods -A
```

