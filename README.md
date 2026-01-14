# Technical Documentation

A comprehensive guide covering Kubernetes, Containers, and Linux System Administration.

## Table of Contents

### Part 1: Kubernetes
> **What it is:** Kubernetes (K8s) is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications across clusters of machines.

| Topic | Description |
|-------|-------------|
| [Architecture Overview](kubernetes/01-architecture.md) | Control plane, data plane, and how components interact |
| [Control Plane Components](kubernetes/02-control-plane.md) | API Server, etcd, Scheduler, Controller Manager |
| [Data Plane Components](kubernetes/03-data-plane.md) | Kubelet, kube-proxy, Container Runtime |
| [Networking](kubernetes/04-networking.md) | Pod networking, Services, DNS, CNI |
| [OpenShift Specifics](kubernetes/05-openshift.md) | Red Hat OpenShift additions to Kubernetes |
| [Troubleshooting](kubernetes/06-troubleshooting.md) | Commands and techniques for debugging |

### Part 2: Containers & Docker
> **What it is:** Containers are lightweight, isolated environments for running applications. Docker is the most popular tool for building and running containers.

| Topic | Description |
|-------|-------------|
| [Container Fundamentals](containers/01-fundamentals.md) | Namespaces, cgroups, union filesystems |
| [Container Registries](containers/03-registries.md) | Docker Hub, Quay.io, push/pull, robot accounts |
| [Example Dockerfiles](containers/04-dockerfiles.md) | Nginx, MySQL, PostgreSQL, Apache, Grafana, Kibana |
| [Salt for Containers](containers/05-salt.md) | SaltStack for container management & orchestration |

### Part 3: Linux System Administration
> **What it is:** Linux system administration involves managing Linux servers - storage, networking, services, security, and troubleshooting.

| Topic | Description |
|-------|-------------|
| [Filesystem Hierarchy](linux/01-filesystem.md) | Directory structure: /etc, /var, /proc, /sys |
| [Storage Fundamentals](linux/02-storage.md) | Block devices, partitions, device naming |
| [Disk Management](linux/03-disks.md) | fdisk, parted, gdisk |
| [LVM](linux/04-lvm.md) | Physical Volumes, Volume Groups, Logical Volumes |
| [Filesystems](linux/05-filesystems.md) | ext4, xfs, btrfs, mounting, fstab |
| [Hardware Information](linux/06-hardware.md) | CPU, memory, disk, PCI, USB info |
| [System Configuration](linux/07-system-config.md) | Hostname, timezone, sysctl, kernel modules |
| [Networking](linux/08-networking.md) | ip commands, NetworkManager, DNS, firewall |
| [Logs and Logging](linux/09-logs.md) | journalctl, dmesg, log files, rotation |
| [Systemd and Services](linux/10-systemd.md) | Service management, timers, boot targets |
| [Performance Monitoring](linux/11-performance.md) | top, htop, iostat, vmstat |
| [Security & SELinux](linux/12-security.md) | SELinux, permissions, ACLs |
| [Bash Scripting](linux/13-bash.md) | Loops, awk, sed, cat EOF, file operations |
| [Shell Configuration](linux/14-shell-config.md) | .bashrc, .bash_profile, aliases, functions |
| [Password Recovery](linux/15-password-recovery.md) | Reset root password via GRUB single user mode |

### Part 4: Git Version Control
> **What it is:** Git is a distributed version control system for tracking changes, collaborating on code, and maintaining project history.

| Topic | Description |
|-------|-------------|
| [Git Guide](git/README.md) | Clone, pull, push, fetch, branches, merge, stash, history |

### Part 5: Common Services
> **What it is:** Configuration and management of common Linux services - DNS, web servers, and databases.

| Topic | Description |
|-------|-------------|
| [dnsmasq](services/01-dnsmasq.md) | DNS, DHCP, and TFTP server for local networks |
| [Apache HTTP Server](services/02-apache.md) | Web server - virtual hosts, SSL, reverse proxy |
| [SQL Basics](services/03-sql.md) | MySQL, PostgreSQL queries and administration |
| [Ansible](services/04-ansible.md) | Agentless automation - playbooks, roles, inventory |

### Part 6: Programming & Scripting
> **What it is:** Programming languages commonly used in DevOps, system administration, and automation.

| Topic | Description |
|-------|-------------|
| [Python](programming/01-python.md) | Data types, loops, functions, file operations, practical examples |

---

## Quick Start

```bash
# Clone this repository
git clone https://github.com/yourusername/docs.git

# Browse by topic
cd docs/kubernetes    # Kubernetes topics
cd docs/containers    # Container topics
cd docs/linux         # Linux admin topics
cd docs/git           # Git version control
```

## Full Document

For the complete guide in a single file, see:
- [kubernetes-architecture-deep-dive.md](../../kubernetes-architecture-deep-dive.md)

---

*Last updated: January 2026*

