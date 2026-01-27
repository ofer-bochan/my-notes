# /proc Filesystem, Kernel Modules & Hardware Inspection

> **What it is:** The `/proc` filesystem is a virtual filesystem providing an interface to kernel data structures. Kernel modules are code that can be loaded into the kernel on demand. This guide also covers hardware inspection tools like `lspci` and `dmidecode`.

---

## lsmod - List Loaded Modules

> **What it is:** `lsmod` shows currently loaded kernel modules and their dependencies.

```bash
lsmod

# Output:
# Module              Size  Used by
# vfat               24576  1
# fat                86016  1 vfat
# e1000             151552  0
# bridge            311296  1 br_netfilter
# stp                16384  1 bridge
# llc                16384  2 bridge,stp
```

### Understanding lsmod Output

| Column | Description |
|--------|-------------|
| **Module** | Name of the loaded kernel module |
| **Size** | Memory used by the module (in bytes) |
| **Used by** | Number of instances + modules that depend on it |

```bash
# Example breakdown:
# bridge            311296  1 br_netfilter
# ^^^^^^            ^^^^^^  ^ ^^^^^^^^^^^^
# Module name       Size    | Modules that depend on it
#                          Number of uses

# "0" in Used by = can be safely unloaded
# Dependencies listed = cannot unload until dependencies removed
```

### Module Commands

```bash
# Module info
modinfo e1000
modinfo -p e1000            # Parameters only
modinfo -d e1000            # Description only

# Load module
modprobe e1000
modprobe e1000 debug=1      # With parameter

# Unload module
modprobe -r e1000
rmmod e1000

# Check if loaded
lsmod | grep e1000

# Load at boot
echo "e1000" >> /etc/modules-load.d/e1000.conf

# Blacklist module
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf

# Module parameters at boot
echo "options e1000 debug=1" >> /etc/modprobe.d/e1000.conf

# View module parameters
cat /sys/module/e1000/parameters/debug
```

---

## The /proc Filesystem

> **What it is:** `/proc` is a virtual filesystem—files don't exist on disk, they're generated on-the-fly by the kernel.

### System Information

```bash
# CPU
cat /proc/cpuinfo
grep "model name" /proc/cpuinfo
grep -c processor /proc/cpuinfo         # Core count

# Memory
cat /proc/meminfo
grep MemTotal /proc/meminfo
grep MemAvailable /proc/meminfo

# Kernel version
cat /proc/version

# Uptime (seconds: total, idle)
cat /proc/uptime

# Load average
cat /proc/loadavg
# 0.15 0.10 0.05 1/234 5678
# 1min 5min 15min running/total last_pid

# Boot parameters
cat /proc/cmdline

# Supported filesystems
cat /proc/filesystems

# Partitions
cat /proc/partitions

# Mounted filesystems
cat /proc/mounts
```

### Kernel Parameters (/proc/sys)

```bash
# Directory structure
ls /proc/sys/
# kernel/  net/  vm/  fs/  dev/

# Read parameter
cat /proc/sys/net/ipv4/ip_forward
cat /proc/sys/kernel/hostname
cat /proc/sys/vm/swappiness
cat /proc/sys/fs/file-max

# Write parameter (temporary)
echo 1 > /proc/sys/net/ipv4/ip_forward
echo 10 > /proc/sys/vm/swappiness

# Or use sysctl (preferred)
sysctl net.ipv4.ip_forward
sysctl -w net.ipv4.ip_forward=1
```

### Common Parameters to Check

```bash
cat /proc/sys/kernel/pid_max           # Max PIDs
cat /proc/sys/kernel/threads-max       # Max threads
cat /proc/sys/fs/file-max              # Max open files
cat /proc/sys/fs/file-nr               # Current: allocated, free, max
cat /proc/sys/net/core/somaxconn       # Max socket backlog
cat /proc/sys/vm/dirty_ratio           # Dirty page %
cat /proc/sys/vm/overcommit_memory     # Memory overcommit (0,1,2)
cat /proc/sys/vm/swappiness            # Swap tendency (0-100)
```

### Process Information (/proc/PID)

```bash
# Each process has /proc/<PID>/
ls /proc/1234/

# Command line
cat /proc/1234/cmdline | tr '\0' ' '

# Process name
cat /proc/1234/comm

# Status (human readable)
cat /proc/1234/status

# Environment variables
cat /proc/1234/environ | tr '\0' '\n'

# Resource limits
cat /proc/1234/limits

# Memory mappings
cat /proc/1234/maps

# Open file descriptors
ls -la /proc/1234/fd/

# I/O statistics
cat /proc/1234/io

# Cgroup membership
cat /proc/1234/cgroup

# Current working directory
ls -la /proc/1234/cwd

# Executable path
ls -la /proc/1234/exe
```

### Network Info (/proc/net)

```bash
cat /proc/net/dev            # Interface stats
cat /proc/net/tcp            # TCP connections
cat /proc/net/udp            # UDP connections
cat /proc/net/arp            # ARP table
cat /proc/net/route          # Routing table
```

### Hardware Info

```bash
cat /proc/cpuinfo            # CPU details
cat /proc/meminfo            # Memory stats
cat /proc/scsi/scsi          # SCSI devices
cat /proc/diskstats          # Disk I/O stats
cat /proc/interrupts         # IRQ usage
cat /proc/ioports            # I/O ports
cat /proc/dma                # DMA channels
```

### Modules in /proc

```bash
cat /proc/modules
# Format: name size load_count dependencies state offset
# e1000 151552 0 - Live 0xffffffffc0a00000
```

---

## lspci - List PCI Devices

> **What it is:** `lspci` shows all PCI devices (network cards, graphics, storage controllers, USB controllers) connected to the system. Essential for hardware troubleshooting and driver identification.

```bash
# Basic listing
lspci

# Output example:
# 00:00.0 Host bridge: Intel Corporation 8th Gen Core Processor Host Bridge
# 00:02.0 VGA compatible controller: Intel Corporation UHD Graphics 620
# 00:1f.6 Ethernet controller: Intel Corporation Ethernet Connection I219-LM
# 02:00.0 Network controller: Intel Corporation Wireless 8265

# Verbose output (more details)
lspci -v
lspci -vv                      # Even more verbose
lspci -vvv                     # Maximum verbosity

# Show kernel drivers in use
lspci -k

# Output with -k:
# 00:1f.6 Ethernet controller: Intel Corporation Ethernet Connection
#     Subsystem: Lenovo ThinkPad X1 Carbon
#     Kernel driver in use: e1000e
#     Kernel modules: e1000e

# Numeric IDs (vendor:device)
lspci -nn

# Output:
# 00:1f.6 Ethernet controller [0200]: Intel Corporation [8086:15d7]
#                                                       ^^^^^^^^^^^
#                                                       vendor:device ID

# Tree view (show hierarchy)
lspci -t
lspci -tv                      # Tree with verbose

# Filter by device class
lspci | grep -i ethernet       # Network adapters
lspci | grep -i vga            # Graphics cards
lspci | grep -i audio          # Sound cards
lspci | grep -i usb            # USB controllers
lspci | grep -i nvme           # NVMe controllers
lspci | grep -i raid           # RAID controllers

# Show specific device details
lspci -s 00:1f.6 -v            # By slot
lspci -d 8086:15d7 -v          # By vendor:device ID

# Update PCI database
update-pciids
```

### Common Use Cases

```bash
# Find network card and its driver
lspci -k | grep -A3 -i ethernet

# Check if GPU is detected
lspci | grep -i nvidia
lspci | grep -i amd
lspci | grep -i vga

# Find NVMe/storage controllers
lspci | grep -i nvme
lspci | grep -i sata

# Check SR-IOV capable devices (for virtualization)
lspci -vvv | grep -i "Single Root I/O"

# Find device for driver binding
lspci -nn | grep -i network
# Use the [vendor:device] to find/load drivers
```

### PCI Slot Notation

```bash
# Format: [domain:]bus:device.function
# Example: 0000:02:00.0
#          ^^^^ ^^ ^^ ^
#          |    |  |  +-- Function (0-7)
#          |    |  +---- Device (0-31)
#          |    +------- Bus (0-255)
#          +----------- Domain (usually 0000)

# Most systems show: bus:device.function (02:00.0)
```

---

## dmidecode - DMI/SMBIOS Information

> **What it is:** `dmidecode` reads hardware information from the system's DMI (Desktop Management Interface) / SMBIOS table. Shows detailed info about BIOS, motherboard, CPU, memory, chassis, and more—data that's burned into the firmware.

```bash
# Full dump (requires root)
sudo dmidecode

# Show specific type
sudo dmidecode -t <type>

# Common types:
sudo dmidecode -t bios         # BIOS information
sudo dmidecode -t system       # System (manufacturer, product, serial)
sudo dmidecode -t baseboard    # Motherboard info
sudo dmidecode -t chassis      # Chassis type
sudo dmidecode -t processor    # CPU details
sudo dmidecode -t memory       # Memory devices
sudo dmidecode -t cache        # CPU cache
sudo dmidecode -t connector    # Port connectors
sudo dmidecode -t slot         # System slots (PCI, etc.)
```

### DMI Type Numbers

| Type | Keyword | Description |
|------|---------|-------------|
| 0 | bios | BIOS Information |
| 1 | system | System Information |
| 2 | baseboard | Motherboard |
| 3 | chassis | Chassis/Enclosure |
| 4 | processor | CPU |
| 7 | cache | CPU Cache |
| 8 | connector | Port Connectors |
| 9 | slot | System Slots |
| 16 | physical-memory-array | Memory Array |
| 17 | memory | Memory Device (DIMMs) |

### Practical Examples

```bash
# Get system serial number
sudo dmidecode -s system-serial-number

# Get manufacturer and model
sudo dmidecode -s system-manufacturer
sudo dmidecode -s system-product-name

# Get BIOS version
sudo dmidecode -s bios-version
sudo dmidecode -s bios-release-date

# Get CPU info
sudo dmidecode -t processor | grep -E "Version|Core Count|Thread Count"

# Get memory details (slots, size, speed)
sudo dmidecode -t memory | grep -E "Size|Speed|Manufacturer|Part Number"

# Count memory slots (total vs populated)
sudo dmidecode -t memory | grep "Size:" | wc -l          # Total slots
sudo dmidecode -t memory | grep "Size:" | grep -v "No Module" | wc -l  # Populated

# Get max supported memory
sudo dmidecode -t 16 | grep "Maximum Capacity"

# Check if server or desktop
sudo dmidecode -t chassis | grep Type

# Get asset tag (if set)
sudo dmidecode -s chassis-asset-tag
sudo dmidecode -s baseboard-asset-tag
```

### String Keywords (-s option)

```bash
# Common string keywords for quick lookups
sudo dmidecode -s bios-vendor
sudo dmidecode -s bios-version
sudo dmidecode -s bios-release-date
sudo dmidecode -s system-manufacturer
sudo dmidecode -s system-product-name
sudo dmidecode -s system-version
sudo dmidecode -s system-serial-number
sudo dmidecode -s system-uuid
sudo dmidecode -s baseboard-manufacturer
sudo dmidecode -s baseboard-product-name
sudo dmidecode -s baseboard-serial-number
sudo dmidecode -s chassis-manufacturer
sudo dmidecode -s chassis-type
sudo dmidecode -s chassis-serial-number
sudo dmidecode -s processor-manufacturer
sudo dmidecode -s processor-version
```

### Quick System Info Script

```bash
#!/bin/bash
# Quick hardware summary using dmidecode
echo "=== System ==="
echo "Manufacturer: $(sudo dmidecode -s system-manufacturer)"
echo "Model: $(sudo dmidecode -s system-product-name)"
echo "Serial: $(sudo dmidecode -s system-serial-number)"
echo ""
echo "=== BIOS ==="
echo "Vendor: $(sudo dmidecode -s bios-vendor)"
echo "Version: $(sudo dmidecode -s bios-version)"
echo "Date: $(sudo dmidecode -s bios-release-date)"
echo ""
echo "=== CPU ==="
sudo dmidecode -t processor | grep -E "^\s*(Version|Core Count|Thread Count):" | head -3
echo ""
echo "=== Memory ==="
echo "Max Capacity: $(sudo dmidecode -t 16 | grep 'Maximum Capacity' | awk '{print $3, $4}')"
echo "Installed DIMMs:"
sudo dmidecode -t memory | grep -E "^\s*Size:" | grep -v "No Module"
```

---

## Quick Reference

### /proc Paths

| Path | Description |
|------|-------------|
| `/proc/cpuinfo` | CPU details |
| `/proc/meminfo` | Memory stats |
| `/proc/version` | Kernel version |
| `/proc/cmdline` | Boot params |
| `/proc/uptime` | System uptime |
| `/proc/loadavg` | Load averages |
| `/proc/mounts` | Mounted FS |
| `/proc/modules` | Loaded modules |
| `/proc/sys/*` | Kernel params |
| `/proc/<PID>/` | Process info |
| `/proc/net/*` | Network stats |

### Hardware Commands

| Command | Description |
|---------|-------------|
| `lsmod` | List loaded kernel modules |
| `lspci` | List PCI devices |
| `lspci -k` | Show kernel drivers for PCI devices |
| `lspci -nn` | Show with vendor:device IDs |
| `dmidecode -t system` | System manufacturer/model/serial |
| `dmidecode -t memory` | Memory DIMMs info |
| `dmidecode -t processor` | CPU details from SMBIOS |
| `dmidecode -t bios` | BIOS version and date |

```bash
# Quick checks
lsmod | grep module                    # Is module loaded?
lspci -k | grep -A3 -i ethernet        # Network card + driver
sudo dmidecode -s system-serial-number # System serial
cat /proc/sys/net/ipv4/ip_forward      # IP forwarding?
cat /proc/meminfo | head -3            # Memory overview
cat /proc/loadavg                      # System load
```

---

*Part of the [Linux Documentation](README.md)*

