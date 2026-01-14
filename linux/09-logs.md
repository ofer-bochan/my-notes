# Logs and Logging

> **What it is:** Linux systems continuously generate logs about what's happening - services starting, users logging in, errors occurring, security events. Logs are essential for troubleshooting problems and monitoring system health.

---

## Logging Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Linux Logging Systems                                     │
│                                                                              │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐                       │
│  │ Applications│   │   Kernel    │   │  Services   │                       │
│  │   (apps)    │   │  (dmesg)    │   │ (systemd)   │                       │
│  └──────┬──────┘   └──────┬──────┘   └──────┬──────┘                       │
│         │                 │                 │                                │
│         └─────────────────┼─────────────────┘                                │
│                           ▼                                                  │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                    systemd-journald                                  │   │
│  │              (Central logging daemon)                                │   │
│  │                                                                      │   │
│  │  Stores: /var/log/journal/ (binary format)                          │   │
│  │  Tool: journalctl                                                   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                           │                                                  │
│                           ▼                                                  │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      rsyslog (optional)                              │   │
│  │              (Writes to text files)                                  │   │
│  │                                                                      │   │
│  │  Files: /var/log/messages, /var/log/secure, etc.                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## journalctl - Primary Log Tool

> **What it is:** `journalctl` queries the systemd journal, which captures logs from all systemd services, the kernel, and applications. It's the modern way to view logs on Linux.

### Basic Usage

```bash
# View all logs
journalctl

# Follow logs in real-time (like tail -f)
journalctl -f

# Show newest entries first
journalctl -r

# Show only last N lines
journalctl -n 100

# Show logs since last boot
journalctl -b

# Show logs from previous boot
journalctl -b -1
```

### Filter by Service

```bash
# All logs from a specific service
journalctl -u nginx

# Follow specific service
journalctl -u nginx -f

# Multiple services
journalctl -u nginx -u php-fpm
```

**Real Example - Troubleshoot failing nginx:**
```bash
# 1. Check service status
systemctl status nginx
# Shows: Failed

# 2. View nginx logs
journalctl -u nginx -n 50

# Output shows:
# nginx[1234]: [emerg] bind() to 0.0.0.0:80 failed (98: Address in use)

# 3. Find what's using port 80
ss -tlnp | grep :80
# LISTEN  0  128  *:80  *:*  users:(("httpd",pid=5678))

# 4. Problem found: httpd is using port 80
```

### Filter by Time

```bash
# Since specific time
journalctl --since "2024-01-15 09:00"

# Until specific time
journalctl --until "2024-01-15 17:00"

# Time range
journalctl --since "2024-01-15 09:00" --until "2024-01-15 17:00"

# Relative time
journalctl --since "1 hour ago"
journalctl --since "30 min ago"
journalctl --since today
journalctl --since yesterday
```

### Filter by Priority

```bash
# Only errors
journalctl -p err

# Errors and above (emerg, alert, crit, err)
journalctl -p err

# Warnings and above
journalctl -p warning

# Priority levels:
# 0 = emerg   (system unusable)
# 1 = alert   (immediate action needed)
# 2 = crit    (critical)
# 3 = err     (errors)
# 4 = warning (warnings)
# 5 = notice  (normal but significant)
# 6 = info    (informational)
# 7 = debug   (debug messages)
```

**Real Example - Morning Health Check:**
```bash
# Check for errors since midnight
journalctl --since today -p err

# Check for failed SSH logins
journalctl -u sshd | grep -i "fail"

# Check for out of memory events
journalctl | grep -i "oom\|out of memory"
```

### Kernel Messages

```bash
# Kernel messages only
journalctl -k

# Kernel errors
journalctl -k -p err
```

### Journal Management

```bash
# Check journal disk usage
journalctl --disk-usage
# Archived and active journals take up 1.5G

# Clean up old logs
journalctl --vacuum-size=500M     # Keep only 500MB
journalctl --vacuum-time=7d       # Keep only 7 days

# Rotate journal now
journalctl --rotate
```

---

## dmesg - Kernel Ring Buffer

> **What it is:** `dmesg` shows the kernel ring buffer - messages from the kernel about hardware, drivers, and low-level events. Essential for troubleshooting hardware issues.

### Basic Usage

```bash
# View kernel messages
dmesg

# With human-readable timestamps
dmesg -T

# Follow in real-time
dmesg -w

# Show only errors and warnings
dmesg -l err,warn

# Clear buffer (requires root)
dmesg -c
```

**Real Example - New disk not showing:**
```bash
# 1. Plug in disk and check dmesg immediately
dmesg -T | tail -30

# Good output:
# [Jan 15 10:30] usb 2-1: new SuperSpeed USB device
# [Jan 15 10:30] scsi 6:0:0:0: Direct-Access  Samsung  Portable SSD
# [Jan 15 10:30] sd 6:0:0:0: [sdc] 976773168 512-byte sectors

# Bad output (problem):
# [Jan 15 10:30] usb 2-1: device descriptor read/64, error -71
# This means cable or port issue

# 2. Find the device name
dmesg | grep "sd\[" | tail
# [sdc] Attached SCSI disk
```

**Real Example - USB device not working:**
```bash
dmesg -T | grep -i usb | tail -20

# Look for errors like:
# "device descriptor read/64, error -71"  → bad cable/port
# "not enough bandwidth"                  → too many USB devices
# "device not accepting address"          → device problem
```

---

## Traditional Log Files

> **What it is:** Many systems still write logs to text files in `/var/log/`. Some applications only write here.

### Important Log Files

| File | Contents | Used For |
|------|----------|----------|
| `/var/log/messages` | General system messages | System troubleshooting (RHEL) |
| `/var/log/syslog` | General system messages | System troubleshooting (Debian) |
| `/var/log/secure` | Authentication logs | Security auditing (RHEL) |
| `/var/log/auth.log` | Authentication logs | Security auditing (Debian) |
| `/var/log/dmesg` | Boot-time kernel messages | Hardware issues |
| `/var/log/cron` | Cron job logs | Scheduled task issues |
| `/var/log/boot.log` | Boot messages | Boot problems |
| `/var/log/audit/audit.log` | SELinux audit events | Security compliance |

### Viewing Log Files

```bash
# View entire log
less /var/log/messages

# Follow log (real-time)
tail -f /var/log/messages

# Last 100 lines
tail -100 /var/log/messages

# Search for errors
grep -i error /var/log/messages

# Search with context
grep -B 5 -A 5 "error" /var/log/messages
```

**Real Example - Find failed SSH attempts:**
```bash
# RHEL/CentOS
grep "Failed password" /var/log/secure

# Count attempts by IP
grep "Failed password" /var/log/secure | \
    awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head

# Output:
# 1523 192.168.1.100    ← Someone is brute-forcing!
#   45 10.0.0.5
#   12 172.16.0.1
```

---

## Log Rotation

> **What it is:** `logrotate` prevents logs from filling disk by rotating (archiving) old logs and deleting ancient ones.

### Configuration

```bash
# Main config
cat /etc/logrotate.conf

# Per-application configs
ls /etc/logrotate.d/
```

**Real Example - Configure rotation for your app:**
```bash
# Create /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily              # Rotate daily
    rotate 7           # Keep 7 days
    compress           # Gzip old logs
    delaycompress      # Don't compress yesterday's
    missingok          # Don't error if log missing
    notifempty         # Don't rotate empty logs
    create 0644 appuser appgroup
    postrotate
        systemctl reload myapp
    endscript
}

# Test config
logrotate -d /etc/logrotate.d/myapp   # Dry run

# Force rotation now
logrotate -f /etc/logrotate.d/myapp
```

---

## Quick Reference

```bash
# === journalctl ===
journalctl -f                       # Follow all
journalctl -u nginx -f              # Follow nginx
journalctl -b -p err                # Errors since boot
journalctl --since "1 hour ago"     # Last hour
journalctl -k                       # Kernel messages

# === dmesg ===
dmesg -T                            # Human timestamps
dmesg -l err,warn                   # Errors/warnings
dmesg | grep -i usb                 # USB issues

# === Files ===
tail -f /var/log/messages           # Follow messages
grep -i error /var/log/messages     # Find errors
```

