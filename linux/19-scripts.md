# Practical Shell Scripts

> **What it is:** Ready-to-use scripts for common system administration tasks.

---

## Script 1: Find Variable in Files

```bash
#!/bin/bash
# find_var.sh - Search for pattern and count occurrences
# Usage: ./find_var.sh <pattern> <directory> [extension]

PATTERN="${1:?Usage: $0 <pattern> <directory> [extension]}"
DIRECTORY="${2:-.}"
EXTENSION="${3:-*}"

echo "Searching for '$PATTERN' in $DIRECTORY"
echo "=========================================="

total_count=0
total_files=0

while IFS= read -r file; do
    count=$(grep -c "$PATTERN" "$file" 2>/dev/null)
    if [ "$count" -gt 0 ]; then
        echo "$file: $count occurrences"
        total_count=$((total_count + count))
        total_files=$((total_files + 1))
    fi
done < <(find "$DIRECTORY" -type f -name "*.$EXTENSION" 2>/dev/null)

echo "=========================================="
echo "Total: $total_count occurrences in $total_files files"
```

---

## Script 2: Check Disk Space with Alerts

```bash
#!/bin/bash
# check_space.sh - Monitor disk space
# Usage: ./check_space.sh [threshold_percent]

THRESHOLD=${1:-80}

echo "Disk Space Check - Threshold: ${THRESHOLD}%"
echo "============================================"

while read -r line; do
    [[ "$line" == Filesystem* ]] && continue
    
    filesystem=$(echo "$line" | awk '{print $1}')
    usage=$(echo "$line" | awk '{print $5}' | tr -d '%')
    mountpoint=$(echo "$line" | awk '{print $6}')
    
    [[ "$filesystem" == tmpfs* ]] && continue
    
    if [ "$usage" -ge "$THRESHOLD" ]; then
        status="⚠️  WARNING"
    else
        status="✓ OK"
    fi
    
    printf "%-30s %5s%% %s\n" "$mountpoint" "$usage" "$status"
done < <(df -h)
```

---

## Script 3: Find Large Files

```bash
#!/bin/bash
# find_large_files.sh - Find files larger than specified size
# Usage: ./find_large_files.sh [size] [directory]

SIZE=${1:-100M}
DIRECTORY=${2:-/}

echo "Finding files larger than $SIZE in $DIRECTORY"
echo "=============================================="

find "$DIRECTORY" -type f -size +$SIZE -exec ls -lh {} \; 2>/dev/null | \
    awk '{print $5, $9}' | sort -rh | head -50
```

---

## Script 4: Log Analyzer

```bash
#!/bin/bash
# log_analyzer.sh - Analyze logs for errors
# Usage: ./log_analyzer.sh <logfile> [pattern]

LOGFILE="${1:?Usage: $0 <logfile> [pattern]}"
PATTERN="${2:-error|fail|critical}"

echo "=== Log Analysis: $LOGFILE ==="
echo "Pattern: $PATTERN"

total_lines=$(wc -l < "$LOGFILE")
matching_lines=$(grep -Eic "$PATTERN" "$LOGFILE")

echo "Total lines: $total_lines"
echo "Matching lines: $matching_lines"

echo -e "\n=== Errors by Type ==="
grep -Eio "$PATTERN" "$LOGFILE" | sort | uniq -c | sort -rn

echo -e "\n=== Last 10 Errors ==="
grep -Ei "$PATTERN" "$LOGFILE" | tail -10
```

---

## Script 5: System Health Check

```bash
#!/bin/bash
# health_check.sh - System health check

echo "=========================================="
echo "System Health Check - $(hostname)"
echo "Date: $(date)"
echo "=========================================="

# CPU Load
echo -n "CPU Load: "
cat /proc/loadavg | awk '{print $1, $2, $3}'

# Memory
echo -n "Memory: "
free -h | grep Mem | awk '{print $3 "/" $2 " used"}'

# Disk
echo "Disk Usage:"
df -h | grep -v tmpfs | grep -v "^Filesystem" | \
    awk '{printf "  %-20s %s used\n", $6, $5}'

# Failed services
failed=$(systemctl --failed --no-legend | wc -l)
echo "Failed Services: $failed"

# Network
echo -n "Network: "
ping -c 1 8.8.8.8 &>/dev/null && echo "Connected" || echo "No connectivity"
```

---

## Script 6: Backup with Rotation

```bash
#!/bin/bash
# backup_rotate.sh - Backup with rotation
# Usage: ./backup_rotate.sh <source> <backup_dir> [retention_days]

SOURCE="${1:?Usage: $0 <source> <backup_dir> [days]}"
BACKUP_DIR="${2:?}"
RETENTION=${3:-7}

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_NAME=$(basename "$SOURCE")_$DATE.tar.gz

mkdir -p "$BACKUP_DIR"

echo "Creating backup: $BACKUP_NAME"
tar -czf "$BACKUP_DIR/$BACKUP_NAME" -C "$(dirname "$SOURCE")" "$(basename "$SOURCE")"

echo "Removing backups older than $RETENTION days..."
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +$RETENTION -delete

echo "Current backups:"
ls -lht "$BACKUP_DIR"/*.tar.gz | head -5
```

---

## Script 7: Service Monitor

```bash
#!/bin/bash
# service_monitor.sh - Monitor and restart services

SERVICES="nginx sshd crond"
LOG_FILE="/var/log/service_monitor.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

for service in $SERVICES; do
    if ! systemctl is-active --quiet "$service"; then
        log "WARNING: $service is down, restarting..."
        if systemctl restart "$service"; then
            log "SUCCESS: $service restarted"
        else
            log "ERROR: Failed to restart $service"
        fi
    fi
done
```

---

## More Script Ideas

| Script | Purpose |
|--------|---------|
| `user_audit.sh` | List users, last login, sudo access |
| `port_scan.sh` | Check open/listening ports |
| `ssl_check.sh` | Check SSL certificate expiration |
| `cleanup_docker.sh` | Remove unused Docker resources |
| `security_audit.sh` | Check SUID files, world-writable |
| `db_backup.sh` | Database backup with rotation |

---

*Part of the [Linux Documentation](README.md)*

