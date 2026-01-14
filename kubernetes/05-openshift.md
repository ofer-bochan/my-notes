# OpenShift Specifics

> **What it is:** OpenShift is Red Hat's enterprise Kubernetes platform. It includes everything in Kubernetes plus additional features for security, developer experience, and operations. OpenShift extensively uses the Operator pattern to manage cluster components.

---

## OpenShift vs Kubernetes

| Feature | Kubernetes | OpenShift |
|---------|------------|-----------|
| **Installation** | DIY | Automated installer |
| **Ingress** | Ingress resource | Routes (+ Ingress) |
| **Security** | Basic | Enhanced (SCC, default restricted) |
| **Registry** | External | Built-in |
| **CI/CD** | External | Built-in (Builds, Pipelines) |
| **CLI** | kubectl | oc (+ kubectl) |
| **Projects** | Namespaces | Projects (enhanced namespaces) |
| **Monitoring** | Install yourself | Built-in (Prometheus, Grafana) |
| **Management** | Manual | Operators |

---

## Understanding Operators

> **What is an Operator?** An Operator is a method of packaging, deploying, and managing a Kubernetes application. It extends the Kubernetes API with Custom Resource Definitions (CRDs) and uses custom controllers to manage application lifecycle automatically.

### Operator Pattern

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         OPERATOR PATTERN                                     │
│                                                                              │
│  ┌─────────────────┐      ┌─────────────────┐      ┌─────────────────┐     │
│  │ Custom Resource │      │   Controller    │      │   Managed       │     │
│  │ Definition (CRD)│─────►│   (Operator)    │─────►│   Application   │     │
│  │                 │      │                 │      │                 │     │
│  │ "What I want"   │      │ "Make it happen"│      │ "Running app"   │     │
│  └─────────────────┘      └─────────────────┘      └─────────────────┘     │
│                                                                              │
│  Example: PostgreSQL Operator                                                │
│  1. You create: PostgreSQL CR (database: mydb, replicas: 3)                 │
│  2. Operator sees CR, creates: StatefulSet, Services, Secrets               │
│  3. Operator manages: Backups, failover, upgrades automatically             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### OpenShift Operator Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Cluster Version Operator (CVO)                            │
│                    "The operator that manages operators"                     │
│                              │                                               │
│         ┌────────────────────┼────────────────────┐                         │
│         ▼                    ▼                    ▼                         │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                   │
│  │   Cluster   │     │   Cluster   │     │   Cluster   │                   │
│  │  Operators  │     │  Operators  │     │  Operators  │                   │
│  │ (etcd, API) │     │ (ingress)   │     │ (MCO, net)  │                   │
│  └─────────────┘     └─────────────┘     └─────────────┘                   │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │              Operator Lifecycle Manager (OLM)                        │   │
│  │              "Manages user-installed operators"                      │   │
│  │                              │                                       │   │
│  │         ┌────────────────────┼────────────────────┐                 │   │
│  │         ▼                    ▼                    ▼                 │   │
│  │  ┌───────────┐       ┌───────────┐       ┌───────────┐             │   │
│  │  │ PostgreSQL│       │   Redis   │       │  Logging  │             │   │
│  │  │ Operator  │       │ Operator  │       │ Operator  │             │   │
│  │  └───────────┘       └───────────┘       └───────────┘             │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Cluster Version Operator (CVO)

> **What it does:** The CVO manages OpenShift cluster upgrades and ensures all cluster operators are running and healthy. It's responsible for the overall cluster version.

```bash
# Check cluster version
oc get clusterversion

# Detailed info
oc describe clusterversion

# View available upgrades
oc adm upgrade

# Start upgrade
oc adm upgrade --to-latest
oc adm upgrade --to=4.14.10

# Watch upgrade
watch oc get clusterversion
watch oc get co
```

---

## Cluster Operators (CO)

> **What they are:** Core operators that manage essential OpenShift components.

```bash
# List all cluster operators
oc get co

# Find unhealthy operators
oc get co | grep -v "True.*False.*False"

# Operator details
oc describe co <name>
```

### Key Cluster Operators

| Operator | Manages | Namespace |
|----------|---------|-----------|
| `etcd` | etcd cluster | openshift-etcd |
| `kube-apiserver` | API server | openshift-kube-apiserver |
| `machine-config` | Node OS configuration | openshift-machine-config-operator |
| `ingress` | Routes/Router | openshift-ingress-operator |
| `dns` | CoreDNS | openshift-dns-operator |
| `network` | SDN/OVN | openshift-network-operator |
| `storage` | Storage classes | openshift-cluster-storage-operator |
| `authentication` | OAuth | openshift-authentication-operator |
| `console` | Web console | openshift-console-operator |
| `monitoring` | Prometheus | openshift-monitoring |

---

## Machine Config Operator (MCO)

> **What it does:** Manages the operating system configuration on OpenShift nodes - kernel parameters, systemd units, files, and other OS-level settings.

### How MCO Works

```
MachineConfig              MachineConfigPool              Nodes
┌────────────┐            ┌────────────┐              ┌────────────┐
│ 00-master  │──┐         │   master   │──────────────│  master-0  │
│ 00-worker  │  │         │ Rendered   │              │  master-1  │
│ 99-custom  │──┼────────►│ Config     │              └────────────┘
└────────────┘  │         └────────────┘
                │         ┌────────────┐              ┌────────────┐
                └────────►│   worker   │──────────────│  worker-0  │
                          │ Rendered   │              │  worker-1  │
                          │ Config     │              └────────────┘
                          └────────────┘
```

### MachineConfig Example - Add File

```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  name: 99-worker-custom-sysctl
  labels:
    machineconfiguration.openshift.io/role: worker
spec:
  config:
    ignition:
      version: 3.2.0
    storage:
      files:
      - path: /etc/sysctl.d/99-custom.conf
        mode: 0644
        contents:
          source: data:,net.ipv4.ip_forward%3D1
```

### MachineConfig Example - Kernel Arguments

```yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfig
metadata:
  name: 99-worker-kernel-args
  labels:
    machineconfiguration.openshift.io/role: worker
spec:
  kernelArguments:
    - nosmt
    - hugepagesz=1G
    - hugepages=16
```

### MCO Commands

```bash
# Check MachineConfigPools
oc get mcp

# Check if updating
oc get mcp worker -o yaml | grep -A 5 status

# List MachineConfigs
oc get mc

# View rendered config
oc get mc rendered-worker-<id> -o yaml

# MCO logs
oc logs -n openshift-machine-config-operator -l k8s-app=machine-config-daemon
```

⚠️ **Warning:** MachineConfig changes cause node reboots!

---

## Operator Lifecycle Manager (OLM)

> **What it does:** Manages user-installed operators from catalogs like OperatorHub.

### OLM Components

| Resource | Purpose |
|----------|---------|
| `CatalogSource` | Operator catalog location |
| `Subscription` | Install + update request |
| `ClusterServiceVersion` | Operator deployment info |
| `InstallPlan` | Approval for install/upgrade |

### Installing an Operator

```bash
# Find available operators
oc get packagemanifests | grep postgresql

# View details
oc describe packagemanifest postgresql

# Create subscription
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: postgresql
  namespace: openshift-operators
spec:
  channel: stable
  name: postgresql
  source: community-operators
  sourceNamespace: openshift-marketplace
  installPlanApproval: Automatic
EOF

# Check installation
oc get csv -n openshift-operators
```

### OLM Commands

```bash
# List catalogs
oc get catalogsource -n openshift-marketplace

# List available operators
oc get packagemanifests

# List subscriptions
oc get subscriptions -A

# List installed operators
oc get csv -A

# Check install plans
oc get installplan -A

# Delete operator
oc delete subscription <name> -n <namespace>
oc delete csv <name> -n <namespace>
```

---

## Routes

> **What it does:** Routes expose services externally with HTTP/HTTPS.

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: my-route
spec:
  host: myapp.apps.cluster.example.com
  to:
    kind: Service
    name: my-service
  port:
    targetPort: 8080
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
```

### TLS Termination Types

| Type | Description |
|------|-------------|
| `edge` | TLS at router, HTTP to pod |
| `passthrough` | TLS direct to pod |
| `reencrypt` | TLS at router, new TLS to pod |

---

## Security Context Constraints (SCC)

> **What it does:** Controls what pods can do - run as root, privileged mode, host access.

```bash
# List SCCs
oc get scc

# View SCC
oc describe scc restricted

# Check pod's SCC
oc get pod <pod> -o yaml | grep scc
```

### Common SCCs

| SCC | Allows |
|-----|--------|
| `restricted` | Non-root only (default) |
| `anyuid` | Any user including root |
| `privileged` | Full access |
| `hostnetwork` | Host network access |

---

## OpenShift CLI (oc)

```bash
# Login
oc login -u kubeadmin -p <password>
oc login                        # Interactive

# Projects
oc projects                     # List
oc project <name>               # Switch
oc new-project dev              # Create

# Apps
oc new-app nginx                # Create from image
oc expose svc/nginx             # Create route

# Debug
oc debug node/<node>            # Node shell
oc debug pod/<pod>              # Pod debug copy
oc rsh <pod>                    # Pod shell

# Cluster status
oc get clusterversion
oc get co
oc whoami
```

---

## Troubleshooting

```bash
# Cluster operators
oc get co | grep -v "True.*False.*False"
oc describe co <name>

# Operator logs
oc logs -n <operator-namespace> -l name=<operator>

# Events
oc get events -A --sort-by='.lastTimestamp'

# MCO issues
oc get mcp
oc logs -n openshift-machine-config-operator -l k8s-app=machine-config-daemon

# OLM issues
oc logs -n openshift-operator-lifecycle-manager -l app=olm-operator
```
