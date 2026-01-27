# must-gather (OpenShift Troubleshooting)

> **What it is:** `must-gather` is an OpenShift tool that collects cluster information, logs, and diagnostic data for troubleshooting. It creates a tarball for Red Hat support or local analysis.

## Basic Usage

```bash
# Default collection
oc adm must-gather

# Specify output directory
oc adm must-gather --dest-dir=/tmp/must-gather

# With timeout
oc adm must-gather --timeout=30m
```

## Specialized Images

```bash
# Default cluster info
oc adm must-gather

# Network diagnostics (OVN/SDN)
oc adm must-gather --image=registry.redhat.io/openshift4/ose-cluster-network-operator -- gather_network_logs

# Storage (OCS/ODF)
oc adm must-gather --image=registry.redhat.io/ocs4/ocs-must-gather-rhel8

# Virtualization (CNV)
oc adm must-gather --image=registry.redhat.io/container-native-virtualization/cnv-must-gather-rhel8

# Logging
oc adm must-gather --image=registry.redhat.io/openshift-logging/cluster-logging-operator-bundle

# Service Mesh
oc adm must-gather --image=registry.redhat.io/openshift-service-mesh/istio-must-gather-rhel8

# GitOps/ArgoCD
oc adm must-gather --image=registry.redhat.io/openshift-gitops-1/must-gather-rhel8

# ACM (Advanced Cluster Management)
oc adm must-gather --image=registry.redhat.io/rhacm2/acm-must-gather-rhel8

# Multiple at once
oc adm must-gather \
  --image=registry.redhat.io/openshift4/ose-must-gather \
  --image=registry.redhat.io/openshift4/ose-cluster-network-operator -- gather_network_logs
```

## Options

```bash
# Specific node
oc adm must-gather --node-name=master-0

# Time-based collection
oc adm must-gather --since=1h
oc adm must-gather --since-time="2024-01-15T10:00:00Z"

# Specific namespaces
oc adm must-gather -- /usr/bin/gather --namespaces=my-namespace
```

## What It Collects

```
must-gather/
├── cluster-scoped-resources/
│   ├── clusteroperators.yaml
│   ├── clusterversions.yaml
│   └── nodes/
├── namespaces/
│   └── openshift-*/
│       ├── pods/
│       ├── configmaps/
│       └── events.yaml
├── host_service_logs/
│   └── masters/<node>/
│       ├── kubelet.log
│       └── crio.log
└── audit_logs/
```

## Analyzing must-gather

```bash
# Extract
tar -xvf must-gather.tar.gz
cd must-gather.local.*

# Cluster operators
cat cluster-scoped-resources/config.openshift.io/clusteroperators.yaml

# Cluster version
cat cluster-scoped-resources/config.openshift.io/clusterversions/version.yaml

# Node info
cat cluster-scoped-resources/core/nodes/<node>.yaml

# Operator logs
cat namespaces/openshift-kube-apiserver/pods/*/kube-apiserver/*.log | grep -i error

# Events
cat namespaces/openshift-*/events.yaml | grep -i warning

# Search for errors
grep -ri "error\|failed" . | head -50
```

## For Specific Issues

| Issue | What to Check |
|-------|---------------|
| Upgrade stuck | `clusterversions/version.yaml` |
| Node NotReady | `host_service_logs/<node>/kubelet.log` |
| Pod crashes | `namespaces/<ns>/pods/<pod>/*.log` |
| Network issues | Run with network operator image |
| Certificate issues | `openshift-kube-apiserver-operator/` |

## Upload to Red Hat Support

```bash
# Compress
tar -czvf must-gather-$(date +%Y%m%d).tar.gz must-gather.local.*

# Upload via CLI
redhat-support-tool addattachment -c <case-number> must-gather-*.tar.gz

# Or via Customer Portal
```

## Quick Reference

```bash
# Standard collection
oc adm must-gather

# With network logs
oc adm must-gather --image=registry.redhat.io/openshift4/ose-cluster-network-operator -- gather_network_logs

# Specific namespace
oc adm must-gather -- /usr/bin/gather --namespaces=my-app

# Last hour only
oc adm must-gather --since=1h
```

---

*Part of the [Kubernetes Documentation](README.md)*


