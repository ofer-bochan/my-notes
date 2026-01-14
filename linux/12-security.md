# Security and SELinux

> **What it is:** Linux security involves multiple layers - file permissions (who can access files), SELinux (mandatory access control that restricts what processes can do), and firewall rules (network access control). This guide covers practical security for RHEL/CentOS systems.

---

## File Permissions

> **What it is:** Every file and directory has permissions that control who can read, write, or execute it. Permissions are set for three categories: owner (user), group, and others (everyone else).

### Understanding Permissions

```
Permission String: -rwxr-xr--

Position:  -    rwx    r-x    r--
           │     │      │      │
           │     │      │      └── Others (everyone else)
           │     │      └── Group (members of file's group)
           │     └── Owner (file's owner)
           └── Type (- = file, d = directory, l = link)

Permission values:
  r (read)    = 4
  w (write)   = 2
  x (execute) = 1
  - (none)    = 0
```

### Common Permission Numbers

| Number | Permissions | Meaning |
|--------|-------------|---------|
| **755** | rwxr-xr-x | Owner: full, Others: read+execute |
| **644** | rw-r--r-- | Owner: read+write, Others: read |
| **700** | rwx------ | Owner only |
| **600** | rw------- | Owner read+write only |
| **777** | rwxrwxrwx | Everyone: full (dangerous!) |

### chmod - Change Permissions

```bash
# Numeric method
chmod 755 script.sh           # rwxr-xr-x
chmod 644 config.txt          # rw-r--r--
chmod 600 private.key         # rw------- (secure!)

# Symbolic method
chmod u+x script.sh           # Add execute for user
chmod g-w file.txt            # Remove write for group
chmod o=r file.txt            # Set others to read only
chmod a+r file.txt            # Add read for all

# Recursive
chmod -R 755 /var/www/html/
```

### chown - Change Ownership

```bash
# Change owner
chown alice file.txt

# Change owner and group
chown alice:developers file.txt

# Change group only
chown :developers file.txt
chgrp developers file.txt

# Recursive
chown -R nginx:nginx /var/www/html/
```

### Real Example - Web Server Files

```bash
# Problem: Web server can't read your files
# Solution: Set correct ownership and permissions

# 1. Change ownership to web server user
chown -R nginx:nginx /var/www/html/

# 2. Set directory permissions (755 = read+execute for all)
find /var/www/html -type d -exec chmod 755 {} \;

# 3. Set file permissions (644 = read for all)
find /var/www/html -type f -exec chmod 644 {} \;

# 4. Set executable scripts
chmod 755 /var/www/html/cgi-bin/*.sh
```

---

## SELinux

> **What it is:** Security-Enhanced Linux (SELinux) is a mandatory access control (MAC) system built into the Linux kernel. It restricts what processes can do, even if running as root. Every file, process, and port has an SELinux context (label) that determines what it can access.

### SELinux Modes

| Mode | Description |
|------|-------------|
| **Enforcing** | SELinux enforces policies, denies violations |
| **Permissive** | SELinux logs violations but doesn't enforce |
| **Disabled** | SELinux is completely off |

```bash
# Check current mode
getenforce
# Enforcing

# Get detailed status
sestatus
# SELinux status:                 enabled
# SELinuxfs mount:                /sys/fs/selinux
# Current mode:                   enforcing
# Policy version:                 33
```

### Changing SELinux Mode

```bash
# Temporary change (until reboot)
setenforce 0                  # Permissive
setenforce 1                  # Enforcing

# Permanent change
vi /etc/selinux/config
# SELINUX=enforcing           # or permissive, disabled
# Requires reboot
```

### SELinux Contexts

Every file, process, and port has a context with four parts:

```
user:role:type:level

Example: system_u:object_r:httpd_sys_content_t:s0

user   = system_u (SELinux user)
role   = object_r (for files)
type   = httpd_sys_content_t (THE IMPORTANT PART)
level  = s0 (MLS level, usually s0)
```

### Viewing Contexts

```bash
# File context
ls -Z /var/www/html/
# -rw-r--r--. root root unconfined_u:object_r:httpd_sys_content_t:s0 index.html

# Process context
ps -eZ | grep httpd
# system_u:system_r:httpd_t:s0 1234 ? 00:00:00 httpd

# Port context
semanage port -l | grep http
# http_port_t    tcp    80, 443, 488, 8008, 8009, 8443
```

### Common SELinux Types

| Type | For | Example |
|------|-----|---------|
| `httpd_sys_content_t` | Web content (read) | /var/www/html |
| `httpd_sys_rw_content_t` | Web content (read/write) | Upload directories |
| `var_log_t` | Log files | /var/log/* |
| `etc_t` | Config files | /etc/* |
| `container_file_t` | Container files | Container volumes |

### Changing File Context

```bash
# Temporary change (doesn't survive restorecon)
chcon -t httpd_sys_content_t /var/www/html/newfile.html

# Permanent change (add to policy)
semanage fcontext -a -t httpd_sys_content_t "/srv/web(/.*)?"
restorecon -Rv /srv/web

# Restore default context
restorecon -v /var/www/html/file.html
restorecon -Rv /var/www/html/        # Recursive
```

### Real Example - Apache Can't Read Files

```bash
# Problem: Apache returns 403 Forbidden for new files

# 1. Check file context
ls -Z /var/www/html/problem.html
# -rw-r--r--. root root unconfined_u:object_r:default_t:s0 problem.html
#                                             ^^^^^^^^^ Wrong type!

# 2. Fix context
restorecon -v /var/www/html/problem.html
# Relabeled /var/www/html/problem.html from ... to ...httpd_sys_content_t:s0

# 3. Verify
ls -Z /var/www/html/problem.html
# -rw-r--r--. root root unconfined_u:object_r:httpd_sys_content_t:s0 problem.html
#                                             ^^^^^^^^^^^^^^^^^^^ Correct!
```

### SELinux Booleans

> **What it is:** Booleans are on/off switches that enable or disable SELinux features without modifying the policy.

```bash
# List all booleans
getsebool -a

# List booleans with description
semanage boolean -l

# Check specific boolean
getsebool httpd_can_network_connect
# httpd_can_network_connect --> off

# Enable boolean (temporary)
setsebool httpd_can_network_connect on

# Enable boolean (permanent)
setsebool -P httpd_can_network_connect on
```

### Common Booleans

| Boolean | Purpose |
|---------|---------|
| `httpd_can_network_connect` | Apache can make outbound connections |
| `httpd_can_network_connect_db` | Apache can connect to databases |
| `httpd_use_nfs` | Apache can use NFS mounts |
| `container_manage_cgroup` | Containers can manage cgroups |

### Real Example - Apache Can't Connect to Backend

```bash
# Problem: Apache proxy returns 503 when connecting to backend

# 1. Check audit log
ausearch -m avc -ts recent
# type=AVC denied { name_connect } for ... httpd_t ... port 3000

# 2. Apache needs permission to connect out
setsebool -P httpd_can_network_connect on

# 3. Or if connecting to specific port, add it to allowed ports
semanage port -a -t http_port_t -p tcp 3000
```

### Troubleshooting SELinux

```bash
# View recent denials
ausearch -m avc -ts recent

# Get suggested fix
ausearch -m avc -ts recent | audit2why

# Watch for denials in real-time
tail -f /var/log/audit/audit.log | grep denied

# Analyze and get suggestions
sealert -a /var/log/audit/audit.log

# Generate policy module from denials
ausearch -m avc -ts recent | audit2allow -M mypolicy
semodule -i mypolicy.pp
```

---

## ACLs (Access Control Lists)

> **What it is:** ACLs provide more fine-grained permissions than traditional owner/group/others. You can give specific users or groups specific permissions.

```bash
# View ACL
getfacl file.txt

# Add user ACL
setfacl -m u:alice:rwx file.txt

# Add group ACL
setfacl -m g:developers:rx file.txt

# Remove ACL
setfacl -x u:alice file.txt

# Remove all ACLs
setfacl -b file.txt

# Default ACL (for directories - new files inherit)
setfacl -d -m g:developers:rwx /shared/
```

---

## Quick Security Checklist

```bash
# 1. File permissions
find /etc -perm /o+w            # World-writable in /etc (bad!)
chmod 600 ~/.ssh/id_rsa         # Secure SSH key

# 2. SELinux
getenforce                      # Should be Enforcing
restorecon -Rv /var/www/html/   # Fix web content labels

# 3. SSH
grep "PermitRootLogin" /etc/ssh/sshd_config  # Should be no

# 4. Running services
systemctl list-units --type=service --state=running
# Disable unnecessary services

# 5. Open ports
ss -tuln                        # Check what's listening
```

