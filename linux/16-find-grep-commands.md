# Essential Linux Commands: find, grep, touch

> **What it is:** Core Linux commands for searching files, searching text, and creating/modifying files.

---

## find Command

> **What it is:** `find` searches for files and directories based on name, size, time, permissions, and more.

### Find by Name

```bash
find /var/log -name "messages"          # Exact name
find /home -name "*.txt"                # Pattern (case sensitive)
find /home -iname "*.TXT"               # Pattern (case insensitive)
find /home -not -name "*.tmp"           # NOT matching
```

### Find by Type

```bash
find /etc -type f -name "*.conf"        # Files only
find /home -type d -name "Downloads"    # Directories only
find /usr -type l                        # Symbolic links

# Types: f=file, d=directory, l=symlink
```

### Find by Size

```bash
find /var -size +100M                   # Larger than 100MB
find /tmp -size -1k                     # Smaller than 1KB
find /var -size +10M -size -100M        # Between sizes

# Units: c=bytes, k=KB, M=MB, G=GB
```

### Find by Time

```bash
find /var/log -mtime -7                 # Modified in last 7 days
find /tmp -mtime +30                    # Modified more than 30 days ago
find /etc -mmin -60                     # Modified in last hour

# -mtime=modification, -atime=access, -ctime=change
# -mmin, -amin, -cmin for minutes
```

### Find by Permissions/Owner

```bash
find /home -perm 755                    # Exact permissions
find / -perm -4000 -type f              # SUID files
find / -user john                       # Owned by user
find / -nouser                          # No owner
```

### Execute Actions

```bash
find /tmp -name "*.tmp" -exec rm {} \;           # Delete each
find /tmp -name "*.log" -exec rm {} +            # Delete batch (faster)
find /var/log -name "*.old" -delete              # Delete directly
find /var/www -type f -exec chmod 644 {} \;      # Change permissions
```

### Real Examples

```bash
# Find large files
find /var -type f -size +100M -exec ls -lh {} \;

# Find and delete old temp files
find /tmp -type f -mtime +7 -delete

# Find config files modified recently
find /etc -type f -name "*.conf" -mtime -1

# Find empty files/directories
find /var/log -type f -empty
find /home -type d -empty
```

---

## grep Command

> **What it is:** `grep` searches for patterns in files and command output.

### Basic Usage

```bash
grep "error" /var/log/messages          # Search in file
grep -r "TODO" /home/user/project/      # Recursive search
cat /var/log/messages | grep "error"    # Search from pipe
```

### Common Options

```bash
grep -i "error" logfile                 # Case insensitive
grep -n "error" logfile                 # Show line numbers
grep -c "error" logfile                 # Count matches
grep -l "error" *.log                   # Files with matches
grep -L "error" *.log                   # Files without matches
grep -v "debug" logfile                 # Invert (lines NOT matching)
grep -w "error" logfile                 # Whole words only
grep -B 3 "error" logfile               # 3 lines before
grep -A 3 "error" logfile               # 3 lines after
grep -C 3 "error" logfile               # 3 lines before and after
```

### Regular Expressions

```bash
grep "^error" logfile                   # Lines starting with
grep "error$" logfile                   # Lines ending with
grep -E "error|warning" logfile         # OR (extended regex)
grep -E "[0-9]{3}\.[0-9]{3}" logfile    # Pattern matching
grep "^[^#]" file                       # Lines not starting with #
```

### Real Examples

```bash
# Find errors in logs
grep -i "error\|fail\|critical" /var/log/messages

# Count occurrences
grep -o "error" logfile | wc -l

# Find in code
grep -rn "TODO\|FIXME" --include="*.py" .

# Find processes
ps aux | grep nginx | grep -v grep

# Search compressed files
zgrep "error" /var/log/messages.gz
```

---

## touch Command

> **What it is:** `touch` creates empty files or updates timestamps.

### Basic Usage

```bash
touch newfile.txt                       # Create empty file
touch file1.txt file2.txt               # Create multiple
touch file{1..10}.txt                   # Create with expansion
```

### Modify Timestamps

```bash
touch existingfile.txt                  # Update to now
touch -a file.txt                       # Update access time only
touch -m file.txt                       # Update modification time only
touch -t 202401151200.00 file.txt       # Set specific time
touch -r reference.txt target.txt       # Copy timestamp from file
touch -d "2 days ago" file.txt          # Relative time
```

### Real Examples

```bash
# Create placeholder files
touch /tmp/maintenance_mode
touch /var/run/app.pid

# Create test files
touch test{1..100}.txt

# Trigger rebuilds
touch Makefile
```

---

## Quick Reference

```bash
# find
find /path -name "*.txt"                # By name
find /path -type f -size +100M          # By size
find /path -mtime -7                    # By time
find /path -exec command {} \;          # Execute action

# grep
grep "pattern" file                     # Search
grep -r "pattern" dir/                  # Recursive
grep -i "pattern" file                  # Case insensitive
grep -v "pattern" file                  # Invert match

# touch
touch file                              # Create/update timestamp
touch -t YYYYMMDDhhmm.ss file          # Set specific time
```

---

*Part of the [Linux Documentation](README.md)*

