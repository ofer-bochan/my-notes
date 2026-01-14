# Bash Scripting Essentials

> **What it is:** Bash (Bourne Again Shell) is the default command-line interpreter on most Linux systems. This guide covers essential commands and scripting techniques for automation and system administration.

---

## Viewing Files: tail, head, cat, less

### tail - View End of Files

> **What it does:** Displays the last part of files. Most commonly used with `-f` to follow log files in real-time.

```bash
# Last 10 lines (default)
tail /var/log/messages

# Last N lines
tail -20 /var/log/messages

# Follow file in real-time (MOST USEFUL!)
tail -f /var/log/messages

# Follow multiple files
tail -f /var/log/messages /var/log/secure

# Follow with retry (if file recreated)
tail -F /var/log/app.log
```

**Real Example - Monitor logs during deployment:**
```bash
# Terminal 1: Watch application log
tail -f /var/log/myapp.log

# Terminal 2: Watch system messages
tail -f /var/log/messages

# Terminal 3: Deploy
systemctl restart myapp
```

### head - View Beginning of Files

> **What it does:** Displays the first part of files.

```bash
# First 10 lines
head /etc/passwd

# First N lines
head -20 /etc/passwd

# First N bytes
head -c 100 file.txt
```

### cat - Display File Contents

> **What it does:** Concatenates and displays file contents.

```bash
# Display file
cat file.txt

# Display with line numbers
cat -n file.txt

# Display multiple files
cat file1.txt file2.txt

# Create file (Ctrl+D to end)
cat > newfile.txt
```

### less - Page Through Files

> **What it does:** View files one screen at a time, with search capability.

```bash
less /var/log/messages

# Navigation:
# Space     - next page
# b         - previous page
# /pattern  - search forward
# ?pattern  - search backward
# n         - next match
# N         - previous match
# g         - go to beginning
# G         - go to end
# q         - quit
```

---

## watch - Repeat Commands

> **What it does:** Runs a command repeatedly and displays output full-screen. Perfect for monitoring changing values.

```bash
# Basic usage (every 2 seconds)
watch df -h

# Custom interval
watch -n 5 df -h

# Highlight differences
watch -d df -h

# Exit when output changes
watch -g "ls /tmp/*.done"
```

**Real Example - Monitor Kubernetes:**
```bash
# Watch pods
watch kubectl get pods

# Watch with 1-second interval
watch -n 1 kubectl get pods

# Watch nodes
watch kubectl top nodes
```

**Real Example - Monitor System:**
```bash
# Disk space
watch df -h

# Memory
watch free -h

# Connections
watch "ss -tuln | grep LISTEN"

# File count
watch "ls /tmp | wc -l"
```

---

## Loops

### For Loop

> **What it does:** Repeats commands for each item in a list.

```bash
# Basic syntax
for item in list; do
    commands
done
```

**Real Example - Process multiple servers:**
```bash
for server in web1 web2 web3; do
    echo "Checking $server..."
    ping -c 1 $server && echo "UP" || echo "DOWN"
done
```

**Real Example - Rename files:**
```bash
for file in *.log; do
    mv "$file" "backup_$file"
done
```

**Real Example - Loop through numbers:**
```bash
for i in {1..10}; do
    echo "Processing $i"
done

# C-style
for ((i=0; i<5; i++)); do
    echo $i
done
```

### While Loop

> **What it does:** Repeats while condition is true.

```bash
# Basic syntax
while condition; do
    commands
done
```

**Real Example - Read file line by line:**
```bash
while read line; do
    echo "Processing: $line"
done < servers.txt
```

**Real Example - Wait for service:**
```bash
while ! nc -z localhost 5432; do
    echo "Waiting for database..."
    sleep 5
done
echo "Database ready!"
```

---

## AWK - Text Processing

> **What it does:** Processes text by columns. Perfect for parsing logs and extracting data.

```bash
# Basic syntax
awk 'pattern { action }' file

# Variables:
# $0 = entire line
# $1 = first column
# $2 = second column
# NF = number of fields
# NR = line number
```

**Real Example - Extract columns:**
```bash
# Get username and shell from passwd
awk -F: '{print $1, $7}' /etc/passwd

# Output:
# root /bin/bash
# alice /bin/bash
```

**Real Example - Analyze Apache logs:**
```bash
# Top 10 IPs
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head

# Output:
# 15234 192.168.1.100
#  8456 10.0.0.50
```

**Real Example - Sum values:**
```bash
# Total file sizes
ls -l | awk '{sum += $5} END {print "Total:", sum}'
```

**Real Example - Filter by condition:**
```bash
# Processes using >10% CPU
ps aux | awk '$3 > 10 {print $11, $3"%"}'

# Users with UID > 1000
awk -F: '$3 > 1000 {print $1}' /etc/passwd
```

---

## SED - Stream Editor

> **What it does:** Search and replace text in files. Can edit files in place.

```bash
# Basic syntax
sed 's/search/replace/' file       # First match
sed 's/search/replace/g' file      # All matches (global)
sed -i 's/search/replace/g' file   # Edit in place
```

**Real Example - Search and replace:**
```bash
# Replace hostname
sed -i 's/oldserver/newserver/g' config.yaml

# With backup
sed -i.bak 's/old/new/g' config.yaml
```

**Real Example - Delete lines:**
```bash
# Delete lines with DEBUG
sed -i '/DEBUG/d' app.log

# Delete empty lines
sed -i '/^$/d' file.txt

# Delete comments
sed -i '/^#/d' config.conf
```

**Real Example - Multiple operations:**
```bash
sed -i \
    -e 's/localhost/192.168.1.100/g' \
    -e 's/DEBUG/INFO/g' \
    -e '/TODO/d' \
    config.yaml
```

---

## Counting: wc, sort, uniq

### wc - Word Count

```bash
# Lines, words, bytes
wc file.txt
# 100 500 3500 file.txt

# Only lines
wc -l file.txt

# Only words
wc -w file.txt
```

### sort - Sort Lines

```bash
# Alphabetical
sort file.txt

# Numeric
sort -n numbers.txt

# Reverse
sort -r file.txt

# By column
sort -k2 file.txt

# Unique only
sort -u file.txt
```

### uniq - Unique Lines

```bash
# Remove duplicates (must be sorted first!)
sort file.txt | uniq

# Count occurrences
sort file.txt | uniq -c

# Show only duplicates
sort file.txt | uniq -d
```

**Real Example - Find most common:**
```bash
# Most common HTTP status codes
awk '{print $9}' access.log | sort | uniq -c | sort -rn | head

# 50000 200
# 10000 304
#  5000 404
```

---

## grep - Search Text

> **What it does:** Searches for patterns in files.

```bash
# Basic search
grep "error" /var/log/messages

# Case insensitive
grep -i "error" /var/log/messages

# Recursive (all files in directory)
grep -r "TODO" /path/

# Show line numbers
grep -n "error" file.txt

# Show context (lines before/after)
grep -B 3 -A 3 "error" file.txt

# Invert match (lines NOT matching)
grep -v "DEBUG" app.log

# Count matches
grep -c "error" file.txt

# Only filenames
grep -l "pattern" *.txt
```

**Real Example - Find config issues:**
```bash
# Find all TODO comments
grep -rn "TODO" /opt/myapp/

# Find hardcoded passwords (security audit)
grep -rn "password\s*=" /etc/
```

---

## File Operations

### Creating and Removing

```bash
# Create directory
mkdir mydir
mkdir -p path/to/deep/dir    # Create parents

# Remove file
rm file.txt
rm -f file.txt               # Force (no confirm)

# Remove directory
rmdir emptydir               # Only if empty
rm -r directory/             # Recursive
rm -rf directory/            # Force recursive (CAREFUL!)
```

### Copying and Moving

```bash
# Copy file
cp source.txt dest.txt

# Copy directory
cp -r sourcedir/ destdir/

# Preserve permissions/timestamps
cp -a source dest

# Move/rename
mv oldname.txt newname.txt
mv file.txt /path/to/dir/
```

---

## Permissions

### chmod - Change Permissions

```bash
# Numeric
chmod 755 script.sh    # rwxr-xr-x
chmod 644 file.txt     # rw-r--r--
chmod 600 secret.key   # rw-------

# Symbolic
chmod +x script.sh     # Add execute
chmod u+x script.sh    # Add execute for user
chmod g-w file.txt     # Remove write from group

# Recursive
chmod -R 755 /var/www/
```

### chown - Change Ownership

```bash
# Change owner
chown alice file.txt

# Change owner and group
chown alice:developers file.txt

# Recursive
chown -R nginx:nginx /var/www/
```

---

## Quick Reference

```bash
# === Viewing Files ===
tail -f /var/log/messages       # Follow log
head -20 file.txt               # First 20 lines
less file.txt                   # Page through
cat file.txt                    # Display all

# === Monitoring ===
watch df -h                     # Watch disk space
watch kubectl get pods          # Watch pods

# === Text Processing ===
awk '{print $1}' file           # First column
sed 's/old/new/g' file          # Replace
grep "pattern" file             # Search

# === Counting ===
wc -l file                      # Count lines
sort | uniq -c                  # Count unique

# === Loops ===
for f in *.txt; do echo $f; done
while read line; do echo $line; done < file
```

