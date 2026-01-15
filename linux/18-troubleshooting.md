# Linux Troubleshooting: Disk Space & sosreport

> **What it is:** Essential commands and tools for diagnosing system issues, disk space problems, and collecting diagnostic data.

---

## Disk Space Troubleshooting

### Check Disk Space

```bash
df -h                           # All filesystems
df -hT                          # With filesystem type
df -i                           # Inode usage (important!)
watch -d df -h                  # Real-time monitoring
```

### Check Directory Sizes

```bash
du -sh .                        # Current directory
du -sh *                        # All items in directory
du -sh * | sort -h              # Sorted by size
du -h --max-depth=1 /var        # Subdirectory sizes
```

### Find Large Files

```bash
# Files larger than 100MB
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null

# Top 20 largest files in /var
find /var -type f -exec du -h {} + 2>/dev/null | sort -rh | head -20

# Files larger than 1GB
find / -type f -size +1G 2>/dev/null
```

### Find Old Files

```bash
find /tmp -type f -atime +30              # Not accessed in 30 days
find /var/log -type f -mtime +90          # Not modified in 90 days
find /var -type f -mtime +30 -size +10M   # Old AND large
```

### Cleanup Commands

```bash
# Journal logs
journalctl --vacuum-time=7d
journalctl --vacuum-size=500M

# Package cache
dnf clean all                             # RHEL/CentOS
apt clean && apt autoremove               # Ubuntu

# Truncate large log (keep file, empty content)
> /var/log/large.log
truncate -s 0 /var/log/large.log

# Delete old logs
find /var/log -name "*.gz" -mtime +30 -delete
```

### Quick Disk Report Script

```bash
#!/bin/bash
echo "=== DISK USAGE ==="
df -hT | grep -v tmpfs

echo -e "\n=== TOP 10 DIRECTORIES IN /var ==="
du -h /var --max-depth=1 2>/dev/null | sort -rh | head -10

echo -e "\n=== FILES > 100MB ==="
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | head -10
```

---

## sosreport (System Diagnostic Collection)

> **What it is:** `sosreport` collects system configuration, logs, and diagnostic information. Standard tool for Red Hat support cases.

### Installation

```bash
sudo dnf install sos              # RHEL/CentOS/Fedora
sudo apt install sosreport        # Ubuntu/Debian
```

### Basic Usage

```bash
# Standard collection
sudo sosreport

# Batch mode (no prompts)
sudo sosreport --batch

# With case ID
sudo sosreport --case-id=12345678 --batch

# Quick/minimal collection
sudo sosreport --preset=minimal --batch

# To specific directory
sudo sosreport --tmp-dir=/var/tmp/sos
```

### Plugin Selection

```bash
# List all plugins
sosreport --list-plugins

# Run specific plugins only
sudo sosreport -o networking,logs,systemd --batch

# Exclude plugins (faster)
sudo sosreport -n docker,kubernetes --batch
```

### Common Plugins

| Plugin | Collects |
|--------|----------|
| `kernel` | Kernel info, dmesg, modules |
| `networking` | Network config, firewall, routes |
| `systemd` | Services, units, journal |
| `logs` | System and application logs |
| `filesys` | Filesystem, mounts, LVM |
| `selinux` | SELinux config, denials |
| `podman` | Podman containers |
| `openshift` | OCP node diagnostics |

### Analyzing sosreport

```bash
# Extract
tar -xvf sosreport-*.tar.xz
cd sosreport-*/

# System info
cat sos_commands/hardware/dmidecode
cat sos_commands/kernel/uname_-a

# Network
cat sos_commands/networking/ip_addr
cat sos_commands/networking/ip_route

# Services
cat sos_commands/systemd/systemctl_--failed

# Logs
cat var/log/messages | grep -i error

# Search for errors
grep -ri "error\|fail" sos_commands/ | head -50
```

### For Specific Issues

```bash
# Network issues
sudo sosreport -o networking,firewalld,selinux --batch

# Storage issues
sudo sosreport -o filesys,lvm2,block --batch

# Performance issues
sudo sosreport -o kernel,memory,processor --batch

# Container issues
sudo sosreport -o podman,crio,container_log --batch
```

### Obfuscate Sensitive Data

```bash
sos clean sosreport-*.tar.xz
```

### Multi-host Collection

```bash
sos collect --nodes=server1,server2 --ssh-user=root
```

### Upload to Red Hat Support

```bash
redhat-support-tool addattachment -c <case-number> sosreport-*.tar.xz
```

---

## Quick Reference

```bash
# Disk space
df -h                                    # Filesystem usage
du -sh * | sort -h                       # Directory sizes
find / -size +100M                       # Large files

# sosreport
sudo sosreport --batch                   # Standard collection
sudo sosreport --preset=minimal          # Quick collection
sudo sosreport -o networking,logs        # Specific plugins
sos clean sosreport-*.tar.xz             # Obfuscate data
```

---

*Part of the [Linux Documentation](README.md)*

