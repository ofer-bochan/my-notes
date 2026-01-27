# Linux System Administration

> **What is Linux System Administration?**  
> Linux system administration involves managing and maintaining Linux servers - configuring storage, networking, services, security, and troubleshooting issues. This guide covers essential skills for managing RHEL/CentOS/Fedora systems commonly used in enterprise environments.

## Topics

| File | Topic | Description |
|------|-------|-------------|
| [01-filesystem.md](01-filesystem.md) | Filesystem Hierarchy | Directory structure: /etc, /var, /proc, /sys |
| [02-storage.md](02-storage.md) | Storage | Block devices, partitions |
| [03-disks.md](03-disks.md) | Disk Management | fdisk, parted, gdisk |
| [04-lvm.md](04-lvm.md) | LVM | Logical Volume Manager |
| [05-filesystems.md](05-filesystems.md) | Filesystems | ext4, xfs, mounting, fstab |
| [06-hardware.md](06-hardware.md) | Hardware | CPU, memory, PCI, USB info |
| [07-system-config.md](07-system-config.md) | System Config | hostname, timezone, sysctl |
| [08-networking.md](08-networking.md) | Networking | ip, NetworkManager, firewall |
| [09-logs.md](09-logs.md) | Logging | journalctl, dmesg, log files |
| [10-systemd.md](10-systemd.md) | Systemd | Services, timers, boot |
| [11-performance.md](11-performance.md) | Performance | top, htop, iostat |
| [12-security.md](12-security.md) | Security | SELinux, permissions |
| [13-bash.md](13-bash.md) | Bash Scripting | Loops, awk, sed |
| [14-shell-config.md](14-shell-config.md) | Shell Config | .bashrc, .bash_profile, aliases |
| [15-password-recovery.md](15-password-recovery.md) | Password Recovery | Reset root password via GRUB |
| [16-find-grep-commands.md](16-find-grep-commands.md) | find & grep | Search files and text |
| [17-vi-editor.md](17-vi-editor.md) | Vi Editor | Basic vi/vim commands |
| [18-troubleshooting.md](18-troubleshooting.md) | Troubleshooting | Disk space, sosreport |
| [19-scripts.md](19-scripts.md) | Scripts | Practical shell scripts |
| [20-proc-and-modules.md](20-proc-and-modules.md) | /proc, Modules & Hardware | lsmod, lspci, dmidecode, /proc filesystem |

## Quick Reference

### Essential Commands

```bash
# System info
hostnamectl                     # Hostname and OS info
uname -a                        # Kernel info
uptime                          # System uptime and load

# Disk
df -h                           # Disk usage
lsblk                           # Block devices
mount                           # Mounted filesystems

# Memory
free -h                         # Memory usage

# Processes
ps aux                          # All processes
top                             # Real-time process monitor
systemctl status <service>      # Service status

# Network
ip addr                         # IP addresses
ss -tuln                        # Listening ports

# Logs
journalctl -f                   # Follow all logs
journalctl -u <service>         # Service logs
dmesg                           # Kernel messages
```

### Important Directories

| Directory | Purpose |
|-----------|---------|
| `/etc` | System configuration files |
| `/var` | Variable data (logs, mail, spool) |
| `/var/log` | Log files |
| `/home` | User home directories |
| `/root` | Root user's home |
| `/tmp` | Temporary files (cleared on reboot) |
| `/proc` | Process and kernel info (virtual) |
| `/sys` | Hardware and device info (virtual) |
| `/boot` | Kernel and bootloader |

### Service Management

```bash
systemctl start <service>       # Start service
systemctl stop <service>        # Stop service
systemctl restart <service>     # Restart service
systemctl enable <service>      # Enable at boot
systemctl status <service>      # Check status
systemctl list-units --failed   # List failed services
```

