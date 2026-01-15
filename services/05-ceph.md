# Ceph Storage

> **What it is:** Ceph is a distributed storage system that provides object, block, and file storage in a single unified platform. It's highly scalable, self-healing, and commonly used with Kubernetes/OpenShift (via Rook or ODF - OpenShift Data Foundation).

## Ceph Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Ceph Architecture                                    │
│                                                                              │
│   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐                      │
│   │   CephFS    │   │     RBD     │   │    RGW      │                      │
│   │   (File)    │   │   (Block)   │   │  (Object)   │                      │
│   └──────┬──────┘   └──────┬──────┘   └──────┬──────┘                      │
│          │                 │                 │                              │
│          └────────────┬────┴────────┬────────┘                              │
│                       │             │                                        │
│                       ▼             ▼                                        │
│              ┌─────────────────────────────┐                                │
│              │         RADOS               │                                │
│              │  (Reliable Autonomic        │                                │
│              │   Distributed Object Store) │                                │
│              └─────────────────────────────┘                                │
│                            │                                                 │
│         ┌──────────────────┼──────────────────┐                             │
│         ▼                  ▼                  ▼                             │
│   ┌───────────┐     ┌───────────┐     ┌───────────┐                        │
│   │    OSD    │     │    OSD    │     │    OSD    │   Object Storage       │
│   │  (disk 1) │     │  (disk 2) │     │  (disk N) │   Daemons              │
│   └───────────┘     └───────────┘     └───────────┘                        │
│                                                                              │
│   ┌───────────┐     ┌───────────┐     ┌───────────┐                        │
│   │    MON    │     │    MON    │     │    MON    │   Monitors             │
│   └───────────┘     └───────────┘     └───────────┘   (cluster state)      │
│                                                                              │
│   ┌───────────┐                       ┌───────────┐                        │
│   │    MGR    │                       │    MDS    │   Metadata Server      │
│   └───────────┘                       └───────────┘   (for CephFS)         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Ceph Components

| Component | Description |
|-----------|-------------|
| **MON** (Monitor) | Maintains cluster state, maps, authentication |
| **OSD** (Object Storage Daemon) | Stores data, handles replication, recovery |
| **MGR** (Manager) | Cluster monitoring, orchestration, web dashboard |
| **MDS** (Metadata Server) | Manages metadata for CephFS (file storage) |
| **RGW** (RADOS Gateway) | RESTful gateway for S3/Swift compatible object storage |
| **RBD** (RADOS Block Device) | Block storage (like virtual disks) |
| **CephFS** | POSIX-compliant distributed filesystem |

---

## Installation (Cephadm)

> **Cephadm** is the recommended deployment tool for Ceph (Octopus 15.2+).

### Prerequisites

```bash
# Install cephadm
curl --silent --remote-name --location https://github.com/ceph/ceph/raw/quincy/src/cephadm/cephadm
chmod +x cephadm
./cephadm add-repo --release quincy
./cephadm install

# Or via package manager (RHEL/CentOS)
dnf install cephadm

# Install ceph-common (CLI tools)
cephadm install ceph-common
```

### Bootstrap First Node

```bash
# Bootstrap (creates first MON and MGR)
cephadm bootstrap --mon-ip <MON_IP>

# Example:
cephadm bootstrap --mon-ip 192.168.1.10

# With cluster network
cephadm bootstrap \
  --mon-ip 192.168.1.10 \
  --cluster-network 10.0.0.0/24
```

### Add Hosts and OSDs

```bash
# Copy SSH key to other nodes
ssh-copy-id -f -i /etc/ceph/ceph.pub root@node2
ssh-copy-id -f -i /etc/ceph/ceph.pub root@node3

# Add hosts to cluster
ceph orch host add node2 192.168.1.11
ceph orch host add node3 192.168.1.12

# View available devices
ceph orch device ls

# Add all available devices as OSDs
ceph orch apply osd --all-available-devices

# Add specific device
ceph orch daemon add osd node1:/dev/sdb
```

---

## Ceph Status and Health

```bash
# Cluster status
ceph status
ceph -s

# Health detail
ceph health detail

# OSD status
ceph osd status
ceph osd tree
ceph osd df

# Pool status
ceph osd pool ls
ceph df

# Real-time watch
ceph -w
```

---

## RBD (Block Storage)

> **RBD** provides block devices that can be mapped to hosts or used as Kubernetes PVs.

### Create and Map

```bash
# Create pool
ceph osd pool create rbd-pool 128
ceph osd pool application enable rbd-pool rbd
rbd pool init rbd-pool

# Create RBD image (10GB virtual disk)
rbd create rbd-pool/myimage --size 10G

# List images
rbd ls -l rbd-pool

# Map to block device
rbd map rbd-pool/myimage
# Output: /dev/rbd0

# Create filesystem and mount
mkfs.xfs /dev/rbd0
mount /dev/rbd0 /mnt/myimage

# Unmap
umount /mnt/myimage
rbd unmap /dev/rbd0
```

### Snapshots

```bash
rbd snap create rbd-pool/myimage@snapshot1
rbd snap ls rbd-pool/myimage
rbd snap rollback rbd-pool/myimage@snapshot1
rbd snap rm rbd-pool/myimage@snapshot1
```

---

## CephFS (File Storage)

> **CephFS** provides a POSIX-compliant distributed filesystem.

### Create CephFS

```bash
# Create pools
ceph osd pool create cephfs_data 64
ceph osd pool create cephfs_metadata 32

# Create filesystem
ceph fs new mycephfs cephfs_metadata cephfs_data

# Deploy MDS
ceph orch apply mds mycephfs --placement="3"
```

### Mount on Client

```bash
# Install ceph-common on client
dnf install ceph-common

# Mount
mount -t ceph mon1,mon2,mon3:/ /mnt/cephfs -o name=admin

# Add to fstab
echo "mon1,mon2,mon3:/ /mnt/cephfs ceph name=admin,secretfile=/etc/ceph/admin.secret,noatime 0 0" >> /etc/fstab
```

---

## RGW (S3 Compatible Object Storage)

### Deploy RGW

```bash
ceph orch apply rgw myrgw --placement="2"
```

### Create User

```bash
radosgw-admin user create \
  --uid=myuser \
  --display-name="My User"
```

### Use S3 Client

```bash
# Configure AWS CLI
aws configure

# Create bucket
aws --endpoint-url http://rgw-endpoint:80 s3 mb s3://mybucket

# Upload file
aws --endpoint-url http://rgw-endpoint:80 s3 cp file.txt s3://mybucket/
```

---

## Ceph with Kubernetes/OpenShift

### Using Rook (Kubernetes)

```bash
kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/deploy/examples/crds.yaml
kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/deploy/examples/common.yaml
kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/deploy/examples/operator.yaml
kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/deploy/examples/cluster.yaml
```

### Using ODF (OpenShift)

```bash
# Check ODF status
oc get storagecluster -n openshift-storage

# Get Ceph tools pod
oc rsh -n openshift-storage $(oc get pods -n openshift-storage -l app=rook-ceph-tools -o name)
ceph status
```

### StorageClass for RBD

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rook-ceph-block
provisioner: rook-ceph.rbd.csi.ceph.com
parameters:
  clusterID: rook-ceph
  pool: replicapool
reclaimPolicy: Delete
allowVolumeExpansion: true
```

---

## Troubleshooting

```bash
ceph health detail              # Detailed health
ceph osd tree                   # OSD hierarchy
ceph pg dump_stuck              # Stuck PGs
ceph log last 100               # Recent logs

# Mark OSD out for maintenance
ceph osd out <osd-id>
ceph osd in <osd-id>
```

---

## Quick Reference

```bash
# Status
ceph -s
ceph health detail
ceph osd tree

# RBD
rbd create <pool>/<image> --size <size>
rbd map <pool>/<image>
rbd snap create <pool>/<image>@<snap>

# CephFS
mount -t ceph mon1:/ /mnt/cephfs -o name=admin

# RGW
radosgw-admin user create --uid=<user> --display-name="<name>"
```

---

*Part of the [Services Documentation](README.md)*

