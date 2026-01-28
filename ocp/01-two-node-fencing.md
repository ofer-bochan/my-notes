# Two-Node OpenShift Cluster with Fencing (HA)

> **Technology Preview Feature** - This feature is NOT supported for production use. Red Hat production SLAs do not apply. Use for testing and evaluation only.

## Overview

A two-node OpenShift cluster with fencing provides **high availability (HA) with a reduced hardware footprint**, designed for:
- Distributed environments
- Edge deployments
- Locations where a full three-node control plane is impractical

**Documentation Sources:**
- [OKD 4.20 - Two-Node Fencing Installation](https://docs.okd.io/4.20/installing/installing_two_node_cluster/installing_tnf/install-tnf.html)
- [Red Hat OCP 4.20 - Two Node Cluster](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html/installing_a_two_node_openshift_cluster/index)

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   Two-Node Fencing Cluster                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────────┐         ┌─────────────────┐          │
│   │  Control Plane  │◄───────►│  Control Plane  │          │
│   │     Node 0      │ Pacemaker│     Node 1      │          │
│   │                 │ Quorum   │                 │          │
│   │  - etcd         │         │  - etcd         │          │
│   │  - API Server   │         │  - API Server   │          │
│   │  - Scheduler    │         │  - Scheduler    │          │
│   │  - Workloads    │         │  - Workloads    │          │
│   └────────┬────────┘         └────────┬────────┘          │
│            │                           │                    │
│            │  BMC/Redfish (Fencing)    │                    │
│            ▼                           ▼                    │
│   ┌─────────────────┐         ┌─────────────────┐          │
│   │   BMC/iDRAC/    │         │   BMC/iDRAC/    │          │
│   │     iLO         │         │     iLO         │          │
│   └─────────────────┘         └─────────────────┘          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Key Characteristics

| Aspect | Details |
|--------|---------|
| Control Plane Nodes | 2 (run both K8s services AND user workloads) |
| Worker Nodes | 0 (control planes handle workloads) |
| Bootstrap Node | 1 (temporary, removed after installation) |
| HA Mechanism | Pacemaker-based fencing via BMC |
| Feature Status | Technology Preview (requires `TechPreviewNoUpgrade`) |

---

## Fencing Mechanism (Split-Brain Prevention)

**Pacemaker** manages fencing by isolating unresponsive nodes using the **Baseboard Management Console (BMC)**:

1. When a node becomes unresponsive, Pacemaker detects the failure
2. The healthy node issues a fencing command via BMC (Redfish API)
3. The unresponsive node is forcibly powered off/isolated
4. The remaining node safely takes over all resources
5. This prevents **split-brain scenarios** and **data corruption**

---

## Minimum Requirements

### Hardware Requirements (Per Node)

| Component | vCPU | RAM | Storage | IOPS |
|-----------|------|-----|---------|------|
| Bootstrap (temporary) | 4 | 16 GB | 120 GB | 300 |
| Control Plane (x2) | 4 | 16 GB | 120 GB | 300 |

### Operating System

All machines **must** use **Red Hat Enterprise Linux CoreOS (RHCOS)**

### BMC Requirements

Each control plane node requires:
- BMC with **Redfish API** support (iDRAC, iLO, etc.)
- Network-accessible BMC endpoint
- BMC credentials for each node

### Networking Requirements

| Component | Description |
|-----------|-------------|
| **API VIP** | Virtual IP for Kubernetes API access |
| **Ingress VIP** | Virtual IP for application ingress traffic |
| **BMC Network** | Network connectivity to BMC interfaces |
| **DNS** | Forward and reverse DNS for all nodes |
| **NTP** | Time synchronization (critical for certificates) |

---

## Deployment Methods

### Option 1: Installer-Provisioned Infrastructure (IPI)

- Automated provisioning of bootstrap and control plane machines
- No manual RHCOS installation required
- BMC credentials provided in `install-config.yaml`

### Option 2: User-Provisioned Infrastructure (UPI)

- Manual provisioning of machines
- Manual RHCOS installation required
- More flexibility for custom environments

---

## Deployment Steps

### Step 1: Prerequisites

```bash
# Download OpenShift installation program
tar xvf openshift-install-linux.tar.gz

# Verify installation
./openshift-install version
```

**Gather Required Information:**
- BMC hostname/IP address for each node
- Redfish API URL (e.g., `https://bmc-node0.example.com/redfish/v1/Systems/1`)
- BMC username and password
- API VIP and Ingress VIP addresses
- Pull secret from console.redhat.com
- SSH public key for node access

### Step 2: Create install-config.yaml

#### IPI (Installer-Provisioned Infrastructure)

```yaml
apiVersion: v1
baseDomain: example.com
compute:
- name: worker
  replicas: 0                          # No dedicated workers
controlPlane:
  name: master
  replicas: 2                          # Two control plane nodes
  fencing:
    credentials:
      - hostname: control-0.example.com
        address: https://bmc-control-0.example.com/redfish/v1/Systems/1
        username: admin
        password: <bmc_password>
        certificateVerification: Disabled    # Use for self-signed certs
      - hostname: control-1.example.com
        address: https://bmc-control-1.example.com/redfish/v1/Systems/1
        username: admin
        password: <bmc_password>
        certificateVerification: Enabled     # Use for CA-signed certs
metadata:
  name: my-two-node-cluster
featureSet: TechPreviewNoUpgrade       # REQUIRED to enable this feature
platform:
  baremetal:
    apiVIPs:
      - 192.168.1.100                   # API virtual IP
    ingressVIPs:
      - 192.168.1.101                   # Ingress virtual IP
    hosts:
      - name: control-0.example.com
        role: master
        bmc:
          address: ipmi://bmc-control-0.example.com
          username: admin
          password: <bmc_password>
        bootMACAddress: aa:bb:cc:dd:ee:00
      - name: control-1.example.com
        role: master
        bmc:
          address: ipmi://bmc-control-1.example.com
          username: admin
          password: <bmc_password>
        bootMACAddress: aa:bb:cc:dd:ee:01
pullSecret: '<your_pull_secret>'
sshKey: '<your_ssh_public_key>'
```

#### UPI (User-Provisioned Infrastructure)

```yaml
apiVersion: v1
baseDomain: example.com
compute:
- name: worker
  replicas: 0
controlPlane:
  name: master
  replicas: 2
  fencing:
    credentials:
      - hostname: control-0.example.com
        address: https://bmc-control-0.example.com/redfish/v1/Systems/1
        username: admin
        password: <bmc_password>
      - hostname: control-1.example.com
        address: https://bmc-control-1.example.com/redfish/v1/Systems/1
        username: admin
        password: <bmc_password>
metadata:
  name: my-two-node-cluster
featureSet: TechPreviewNoUpgrade
platform:
  none: {}                              # UPI uses platform: none
pullSecret: '<your_pull_secret>'
sshKey: '<your_ssh_public_key>'
```

### Step 3: Generate Manifests and Ignition Configs

```bash
# Create installation directory
mkdir ~/ocp-install && cd ~/ocp-install

# Copy install-config.yaml (backup first - it gets consumed!)
cp install-config.yaml install-config.yaml.backup

# Generate manifests (optional - for customization)
./openshift-install create manifests --dir ~/ocp-install

# Generate ignition configs
./openshift-install create ignition-configs --dir ~/ocp-install
```

**Generated Files:**
```
~/ocp-install/
├── auth/
│   ├── kubeconfig
│   └── kubeadmin-password
├── bootstrap.ign
├── master.ign
├── worker.ign
└── metadata.json
```

### Step 4: Deploy the Cluster

#### For IPI:

```bash
./openshift-install create cluster --dir ~/ocp-install --log-level=info
```

#### For UPI:

1. **Deploy bootstrap node** with `bootstrap.ign`
2. **Deploy control plane nodes** with `master.ign`
3. **Boot nodes via PXE or ISO**
4. **Wait for bootstrap to complete**:
   ```bash
   ./openshift-install wait-for bootstrap-complete --dir ~/ocp-install --log-level=info
   ```
5. **Remove bootstrap node** after successful completion
6. **Wait for installation to complete**:
   ```bash
   ./openshift-install wait-for install-complete --dir ~/ocp-install --log-level=info
   ```

### Step 5: Post-Installation Verification

```bash
# Export kubeconfig
export KUBECONFIG=~/ocp-install/auth/kubeconfig

# Verify nodes are ready
oc get nodes
# Expected output:
# NAME                      STATUS   ROLES                         AGE   VERSION
# control-0.example.com     Ready    control-plane,master,worker   1h    v1.29.x
# control-1.example.com     Ready    control-plane,master,worker   1h    v1.29.x

# Verify all cluster operators are available
oc get clusteroperators
# All operators should show AVAILABLE=True, PROGRESSING=False, DEGRADED=False

# Verify cluster version
oc get clusterversion

# Check Pacemaker/fencing status (SSH to control plane node)
ssh core@control-0.example.com "sudo pcs status"

# Verify etcd health
oc get etcd -o=jsonpath='{range .items[0].status.conditions[?(@.type=="EtcdMembersAvailable")]}{.message}{"\n"}{end}'

# Check etcd member list
oc rsh -n openshift-etcd etcd-control-0.example.com etcdctl member list -w table
```

---

## Key Configuration Parameters

| Parameter | Value | Description |
|-----------|-------|-------------|
| `controlPlane.replicas` | `2` | **Must be 2** for two-node fencing |
| `compute.replicas` | `0` | No dedicated worker nodes |
| `featureSet` | `TechPreviewNoUpgrade` | **Required** to enable this feature |
| `fencing.credentials` | (array) | BMC credentials for each control plane |
| `certificateVerification` | `Disabled`/`Enabled` | `Disabled` for self-signed BMC certs |

---

## Troubleshooting

### Check Pacemaker Status

```bash
# SSH to a control plane node
ssh core@control-0.example.com

# Check cluster status
sudo pcs status

# Check fencing devices
sudo pcs stonith status

# View Pacemaker logs
sudo journalctl -u pacemaker -f
```

### Check Fencing Configuration

```bash
# List stonith devices
sudo pcs stonith config

# Test fencing (CAUTION: will fence the node!)
# sudo pcs stonith fence <node-name>
```

### etcd Health

```bash
# Check etcd pod status
oc get pods -n openshift-etcd

# Check etcd member health
oc rsh -n openshift-etcd etcd-control-0.example.com etcdctl endpoint health --cluster

# Check etcd alarms
oc rsh -n openshift-etcd etcd-control-0.example.com etcdctl alarm list
```

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Fencing not working | BMC unreachable | Verify BMC network connectivity |
| Split-brain | Fencing failed | Check BMC credentials and Redfish API |
| etcd unhealthy | Network partition | Verify network between nodes |
| Bootstrap timeout | DNS issues | Verify forward/reverse DNS |

---

## Important Considerations

### Before Deployment
- ✅ **Backup etcd** before making any changes
- ✅ **Test BMC connectivity** from both nodes
- ✅ **Verify Redfish API** endpoints are accessible
- ✅ **Ensure time synchronization** (NTP) is working
- ✅ **Configure DNS** with forward and reverse records

### Limitations
- ⚠️ **Technology Preview** - not for production
- ⚠️ **No upgrade path** (due to `TechPreviewNoUpgrade`)
- ⚠️ **Requires BMC/Redfish** support on all nodes
- ⚠️ **Network partitions** require fencing to resolve
- ⚠️ **No additional workers** can be added

### Two-Node Options Comparison

| Feature | Two-Node with Fencing | Two-Node with Arbiter |
|---------|----------------------|----------------------|
| HA Mechanism | Pacemaker + BMC | etcd arbiter node |
| Extra Hardware | BMC on each node | Lightweight arbiter VM |
| Split-Brain Prevention | Fencing (power off) | Quorum via arbiter |
| Status | Technology Preview | Technology Preview |

---

## DNS Records Example

```
# Forward DNS (A records)
api.my-cluster.example.com.        IN  A  192.168.1.100
api-int.my-cluster.example.com.    IN  A  192.168.1.100
*.apps.my-cluster.example.com.     IN  A  192.168.1.101
control-0.my-cluster.example.com.  IN  A  192.168.1.10
control-1.my-cluster.example.com.  IN  A  192.168.1.11

# Reverse DNS (PTR records)
100.1.168.192.in-addr.arpa.  IN  PTR  api.my-cluster.example.com.
10.1.168.192.in-addr.arpa.   IN  PTR  control-0.my-cluster.example.com.
11.1.168.192.in-addr.arpa.   IN  PTR  control-1.my-cluster.example.com.

# SRV records for etcd
_etcd-server-ssl._tcp.my-cluster.example.com.  86400 IN SRV 0 10 2380 etcd-0.my-cluster.example.com.
_etcd-server-ssl._tcp.my-cluster.example.com.  86400 IN SRV 0 10 2380 etcd-1.my-cluster.example.com.
```

---

## References

- [OKD 4.20 - Installing Two-Node with Fencing](https://docs.okd.io/4.20/installing/installing_two_node_cluster/installing_tnf/install-tnf.html)
- [OKD 4.20 - Preparing for Two-Node Installation](https://docs.okd.io/4.20/installing/installing_two_node_cluster/installing_tnf/installing-two-node-fencing.html)
- [Red Hat OCP 4.20 - Two Node Cluster Documentation](https://docs.redhat.com/en/documentation/openshift_container_platform/4.20/html/installing_a_two_node_openshift_cluster/index)
- [Two-Node with Arbiter (Alternative)](https://docs.okd.io/4.20/installing/installing_two_node_cluster/about-two-node-arbiter-installation.html)

---

*Last updated: January 2026*

