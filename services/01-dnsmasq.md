# dnsmasq - DNS, DHCP, and TFTP Server

> **What it is:** dnsmasq is a lightweight DNS, DHCP, and TFTP server commonly used for local network services, development environments, and as a caching DNS forwarder. It's particularly popular in home networks, labs, and for PXE booting.

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         dnsmasq Functions                                    │
│                                                                              │
│   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐    │
│   │    DNS      │   │    DHCP     │   │    TFTP     │   │   DNS       │    │
│   │   Server    │   │   Server    │   │   Server    │   │   Cache     │    │
│   └──────┬──────┘   └──────┬──────┘   └──────┬──────┘   └──────┬──────┘    │
│          │                 │                 │                 │            │
│          └────────────┬────┴────────┬────────┴─────────────────┘            │
│                       │             │                                        │
│                       ▼             ▼                                        │
│              ┌─────────────────────────────┐                                │
│              │         dnsmasq             │                                │
│              │   /etc/dnsmasq.conf         │                                │
│              │   /etc/dnsmasq.d/*.conf     │                                │
│              └─────────────────────────────┘                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Installation

```bash
# RHEL/CentOS/Fedora
sudo dnf install dnsmasq

# Ubuntu/Debian
sudo apt install dnsmasq

# Start and enable
sudo systemctl enable --now dnsmasq

# Check status
sudo systemctl status dnsmasq
```

## Configuration Files

| File | Purpose |
|------|---------|
| `/etc/dnsmasq.conf` | Main configuration file |
| `/etc/dnsmasq.d/*.conf` | Drop-in configuration files |
| `/etc/hosts` | Static DNS entries (read automatically) |
| `/var/lib/dnsmasq/dnsmasq.leases` | DHCP lease database |

## Basic Configuration

```bash
# /etc/dnsmasq.conf

# Listen on specific interface
interface=eth0

# Listen on specific address
listen-address=127.0.0.1,192.168.1.1

# Port (default 53)
port=53

# Don't read /etc/resolv.conf (use your own upstream)
no-resolv

# Upstream DNS servers
server=8.8.8.8
server=8.8.4.4
server=1.1.1.1

# Domain for DHCP hosts
domain=lab.local

# Expand simple names with domain
expand-hosts

# Local domain - never forward queries for this domain
local=/lab.local/

# DNS cache size (default 150)
cache-size=1000

# Log queries (useful for debugging)
log-queries
log-facility=/var/log/dnsmasq.log
```

## Static DNS Entries

### Method 1: Using /etc/hosts (Simplest)

```bash
# /etc/hosts - dnsmasq reads this automatically
192.168.1.10    server1.lab.local server1
192.168.1.11    server2.lab.local server2
192.168.1.12    server3.lab.local server3
```

### Method 2: Using address directive

```bash
# /etc/dnsmasq.d/static-hosts.conf

# Single host
address=/server1.lab.local/192.168.1.10
address=/server2.lab.local/192.168.1.11

# Wildcard - all *.apps.lab.local resolve to same IP
address=/.apps.lab.local/192.168.1.100

# Block a domain (return NXDOMAIN)
address=/ads.example.com/
```

### Method 3: Using host-record

```bash
# More explicit format with IPv4 and IPv6 support
host-record=server1.lab.local,192.168.1.10
host-record=server2.lab.local,192.168.1.11,fd00::11
```

## DHCP Configuration

```bash
# /etc/dnsmasq.d/dhcp.conf

# Enable DHCP on interface
interface=eth0

# DHCP range: start, end, lease time
dhcp-range=192.168.1.100,192.168.1.200,24h

# Default gateway
dhcp-option=option:router,192.168.1.1

# DNS server to give to clients
dhcp-option=option:dns-server,192.168.1.1

# Domain name
dhcp-option=option:domain-name,lab.local

# NTP server
dhcp-option=option:ntp-server,192.168.1.1

# Set the DHCP server to authoritative mode
dhcp-authoritative
```

## Static DHCP Reservations

```bash
# /etc/dnsmasq.d/dhcp-static.conf

# Format: dhcp-host=MAC,hostname,IP,lease-time

# Assign fixed IP to MAC address
dhcp-host=00:11:22:33:44:55,server1,192.168.1.10,infinite

# Ignore a specific MAC (don't give DHCP)
dhcp-host=00:11:22:33:44:66,ignore

# Multiple entries for a lab
dhcp-host=aa:bb:cc:dd:ee:f1,web-server,192.168.1.20,24h
dhcp-host=aa:bb:cc:dd:ee:f2,db-server,192.168.1.21,24h
dhcp-host=aa:bb:cc:dd:ee:f3,app-server,192.168.1.22,24h
```

## TFTP for PXE Boot

```bash
# /etc/dnsmasq.d/tftp.conf

# Enable TFTP
enable-tftp

# TFTP root directory
tftp-root=/var/lib/tftpboot

# PXE boot file (BIOS)
dhcp-boot=pxelinux.0

# For UEFI clients
dhcp-match=set:efi-x86_64,option:client-arch,7
dhcp-boot=tag:efi-x86_64,grubx64.efi
```

## Complete Lab Example

```bash
# /etc/dnsmasq.d/lab.conf
# Complete lab environment with DNS + DHCP

# Listen on lab network interface
interface=ens192
bind-interfaces

# Don't use system resolv.conf
no-resolv

# Upstream DNS
server=8.8.8.8
server=1.1.1.1

# Local domain
domain=lab.local
local=/lab.local/
expand-hosts

# DHCP range
dhcp-range=10.0.0.100,10.0.0.200,255.255.255.0,12h

# Gateway and DNS for DHCP clients
dhcp-option=option:router,10.0.0.1
dhcp-option=option:dns-server,10.0.0.1

# Static reservations for lab servers
dhcp-host=00:50:56:aa:bb:01,hub-cluster,10.0.0.10,infinite
dhcp-host=00:50:56:aa:bb:02,sno1,10.0.0.11,infinite
dhcp-host=00:50:56:aa:bb:03,sno2,10.0.0.12,infinite

# Logging
log-queries
log-dhcp
log-facility=/var/log/dnsmasq-lab.log

# Cache size
cache-size=1000
```

## Commands

```bash
# Test configuration syntax
dnsmasq --test

# Run in foreground with debug output
dnsmasq --no-daemon --log-queries

# Query dnsmasq directly
dig @127.0.0.1 server1.lab.local
nslookup server1.lab.local 127.0.0.1

# Check DHCP leases
cat /var/lib/dnsmasq/dnsmasq.leases

# Flush DNS cache (send SIGHUP)
sudo systemctl reload dnsmasq
# or
sudo kill -HUP $(pidof dnsmasq)
```

## Troubleshooting

```bash
# Check if dnsmasq is running
systemctl status dnsmasq

# View logs
journalctl -u dnsmasq -f

# Check config syntax
dnsmasq --test

# Check listening ports
ss -tuln | grep :53     # DNS
ss -tuln | grep :67     # DHCP

# Test DNS resolution
dig @localhost example.com

# Firewall rules (if needed)
sudo firewall-cmd --add-service=dns --permanent
sudo firewall-cmd --add-service=dhcp --permanent
sudo firewall-cmd --reload
```

## Common Issues

| Problem | Solution |
|---------|----------|
| Port 53 already in use | Check for systemd-resolved: `systemctl stop systemd-resolved` |
| DHCP not working | Ensure interface is correct and firewall allows port 67/68 |
| DNS not resolving | Check upstream servers and `/etc/resolv.conf` |
| Clients not getting correct options | Verify `dhcp-option` syntax |

---

*Part of the [Services Documentation](README.md)*

