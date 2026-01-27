# Linux Networking

> **What it is:** Linux networking involves configuring network interfaces, IP addresses, routes, DNS, and firewall rules. Modern RHEL/CentOS/Fedora systems use NetworkManager for network configuration, with tools like `nmcli`, `ip`, and `firewall-cmd`.

---

## nmcli - NetworkManager CLI

> **What it is:** `nmcli` is the command-line tool for NetworkManager. It manages network connections, devices, and settings without needing a GUI.

### Connection Management

```bash
# List all connections
nmcli connection show
nmcli con show                  # Short form

# Show active connections only
nmcli con show --active

# Show connection details
nmcli con show "Wired connection 1"
nmcli con show eth0

# Add new connection (static IP)
nmcli con add type ethernet \
    con-name "my-static" \
    ifname eth0 \
    ipv4.addresses 192.168.1.100/24 \
    ipv4.gateway 192.168.1.1 \
    ipv4.dns "8.8.8.8,8.8.4.4" \
    ipv4.method manual

# Add new connection (DHCP)
nmcli con add type ethernet \
    con-name "my-dhcp" \
    ifname eth0 \
    ipv4.method auto

# Modify existing connection
nmcli con mod "Wired connection 1" ipv4.addresses 192.168.1.50/24
nmcli con mod "Wired connection 1" ipv4.gateway 192.168.1.1
nmcli con mod "Wired connection 1" ipv4.dns "8.8.8.8"
nmcli con mod "Wired connection 1" ipv4.method manual
nmcli con mod "Wired connection 1" connection.autoconnect yes

# Add secondary IP address
nmcli con mod "eth0" +ipv4.addresses 192.168.1.101/24

# Remove IP address
nmcli con mod "eth0" -ipv4.addresses 192.168.1.101/24

# Activate/deactivate connection
nmcli con up "my-static"
nmcli con down "my-static"

# Reload connection (after manual file edit)
nmcli con reload
nmcli con up "my-static"

# Delete connection
nmcli con delete "my-static"
```

### Device Management

```bash
# List devices
nmcli device status
nmcli dev status                # Short form

# Show device details
nmcli dev show eth0

# Connect/disconnect device
nmcli dev connect eth0
nmcli dev disconnect eth0

# Set device to be managed/unmanaged by NetworkManager
nmcli dev set eth0 managed yes
nmcli dev set eth0 managed no

# WiFi operations
nmcli dev wifi list             # Scan for networks
nmcli dev wifi connect "SSID" password "password"
nmcli radio wifi on             # Enable WiFi
nmcli radio wifi off            # Disable WiFi
```

### Useful nmcli Options

```bash
# Terse output (for scripting)
nmcli -t con show

# Fields selection
nmcli -f NAME,UUID,TYPE,DEVICE con show

# Watch for changes
nmcli monitor

# General status
nmcli general status
nmcli general hostname
nmcli general hostname newhostname    # Set hostname
```

### Connection Files Location

```bash
# Connection profiles stored in:
/etc/NetworkManager/system-connections/

# Example: View connection file
cat /etc/NetworkManager/system-connections/my-static.nmconnection

# After manual edit, reload:
nmcli con reload
```

---

## ip Command (iproute2)

> **What it is:** The `ip` command is the modern replacement for older tools like `ifconfig`, `route`, and `arp`. It's part of the iproute2 package and provides comprehensive network configuration.

### ip addr - Address Management

```bash
# Show all addresses
ip addr
ip a                            # Short form

# Show specific interface
ip addr show eth0
ip a s eth0                     # Short form

# Show only IPv4
ip -4 addr

# Show only IPv6
ip -6 addr

# Add IP address
ip addr add 192.168.1.100/24 dev eth0

# Add secondary IP
ip addr add 192.168.1.101/24 dev eth0 label eth0:1

# Delete IP address
ip addr del 192.168.1.100/24 dev eth0

# Flush all addresses from interface
ip addr flush dev eth0
```

### Understanding ip addr Output

```bash
# Example output:
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:11:22:33:44:55 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.100/24 brd 192.168.1.255 scope global dynamic eth0
       valid_lft 86400sec preferred_lft 86400sec
    inet6 fe80::211:22ff:fe33:4455/64 scope link
       valid_lft forever preferred_lft forever
```

| Field | Description |
|-------|-------------|
| `2:` | Interface index number |
| `eth0:` | Interface name |
| `<BROADCAST,MULTICAST,UP,LOWER_UP>` | Interface flags (see below) |
| `mtu 1500` | Maximum Transmission Unit (bytes) |
| `qdisc fq_codel` | Queuing discipline (packet scheduler) |
| `state UP` | Operational state (UP, DOWN, UNKNOWN) |
| `group default` | Interface group |
| `qlen 1000` | Transmit queue length |
| `link/ether` | MAC address type |
| `00:11:22:33:44:55` | MAC address |
| `brd ff:ff:ff:ff:ff:ff` | Broadcast MAC address |
| `inet` | IPv4 address |
| `192.168.1.100/24` | IP address with CIDR prefix |
| `brd 192.168.1.255` | Broadcast address |
| `scope global` | Address scope (global, link, host) |
| `dynamic` | Address obtained via DHCP |
| `valid_lft` | Valid lifetime (DHCP lease) |
| `preferred_lft` | Preferred lifetime |
| `inet6` | IPv6 address |
| `scope link` | Link-local address |

**Interface Flags:**

| Flag | Description |
|------|-------------|
| `UP` | Interface is administratively up |
| `LOWER_UP` | Physical link is up (cable connected) |
| `BROADCAST` | Supports broadcast |
| `MULTICAST` | Supports multicast |
| `LOOPBACK` | Loopback interface |
| `PROMISC` | Promiscuous mode enabled |
| `NO-CARRIER` | No physical link detected |
| `DORMANT` | Driver waiting for external event |

**Address Scopes:**

| Scope | Description |
|-------|-------------|
| `global` | Valid everywhere, routable |
| `link` | Valid only on this link (e.g., fe80::) |
| `host` | Valid only on this host (e.g., 127.0.0.1) |

---

## ifconfig (Legacy)

> **What it is:** `ifconfig` is the legacy tool for network configuration. Replaced by `ip` command but still available on most systems. Use `ip` for new scripts.

```bash
# Show all interfaces
ifconfig
ifconfig -a                     # Include down interfaces

# Show specific interface
ifconfig eth0

# Example output:
# eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
#         inet 192.168.1.100  netmask 255.255.255.0  broadcast 192.168.1.255
#         inet6 fe80::211:22ff:fe33:4455  prefixlen 64  scopeid 0x20<link>
#         ether 00:11:22:33:44:55  txqueuelen 1000  (Ethernet)
#         RX packets 123456  bytes 123456789 (117.7 MiB)
#         RX errors 0  dropped 0  overruns 0  frame 0
#         TX packets 98765  bytes 12345678 (11.7 MiB)
#         TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### Understanding ifconfig Output

| Field | Description |
|-------|-------------|
| `flags=4163` | Numeric flags value |
| `<UP,BROADCAST,RUNNING,MULTICAST>` | Human-readable flags |
| `mtu 1500` | Maximum Transmission Unit |
| `inet` | IPv4 address |
| `netmask` | Subnet mask |
| `broadcast` | Broadcast address |
| `inet6` | IPv6 address |
| `prefixlen 64` | IPv6 prefix length |
| `scopeid 0x20<link>` | IPv6 scope |
| `ether` | MAC address |
| `txqueuelen` | Transmit queue length |
| `RX packets` | Received packet count |
| `RX bytes` | Received bytes |
| `RX errors` | Receive errors |
| `RX dropped` | Dropped incoming packets |
| `RX overruns` | Buffer overruns (packets lost) |
| `RX frame` | Frame errors |
| `TX packets` | Transmitted packet count |
| `TX bytes` | Transmitted bytes |
| `TX errors` | Transmit errors |
| `TX dropped` | Dropped outgoing packets |
| `TX overruns` | Transmit buffer overruns |
| `TX carrier` | Carrier errors (link issues) |
| `collisions` | Packet collisions (half-duplex) |

**ifconfig Flags:**

| Flag | Description |
|------|-------------|
| `UP` | Interface is up |
| `BROADCAST` | Supports broadcast |
| `RUNNING` | Interface is operational |
| `MULTICAST` | Supports multicast |
| `LOOPBACK` | Loopback interface |
| `PROMISC` | Promiscuous mode |
| `ALLMULTI` | Receive all multicast |

### ifconfig Commands (Legacy)

```bash
# Bring interface up/down
ifconfig eth0 up
ifconfig eth0 down

# Set IP address
ifconfig eth0 192.168.1.100 netmask 255.255.255.0

# Set MTU
ifconfig eth0 mtu 9000

# Enable promiscuous mode
ifconfig eth0 promisc

# ifconfig to ip command mapping:
# ifconfig           -> ip addr
# ifconfig eth0 up   -> ip link set eth0 up
# ifconfig eth0 down -> ip link set eth0 down
# route              -> ip route
# arp                -> ip neigh
```

---

### ip link - Interface Management

```bash
# Show all interfaces
ip link
ip l                            # Short form

# Show specific interface
ip link show eth0

# Bring interface up/down
ip link set eth0 up
ip link set eth0 down

# Set MTU
ip link set eth0 mtu 9000

# Set MAC address
ip link set eth0 address 00:11:22:33:44:55

# Enable/disable promiscuous mode
ip link set eth0 promisc on
ip link set eth0 promisc off

# Show interface statistics
ip -s link show eth0
```

### ip route - Routing Table

```bash
# Show routing table
ip route
ip r                            # Short form

# Show route to specific destination
ip route get 8.8.8.8

# Add route
ip route add 10.0.0.0/8 via 192.168.1.1
ip route add 10.0.0.0/8 via 192.168.1.1 dev eth0

# Add default gateway
ip route add default via 192.168.1.1

# Delete route
ip route del 10.0.0.0/8

# Replace route (add or modify)
ip route replace 10.0.0.0/8 via 192.168.1.254

# Add route with metric
ip route add 10.0.0.0/8 via 192.168.1.1 metric 100
```

### ip neigh - ARP/Neighbor Table

```bash
# Show ARP table
ip neigh
ip n                            # Short form

# Show for specific interface
ip neigh show dev eth0

# Add static ARP entry
ip neigh add 192.168.1.50 lladdr 00:11:22:33:44:55 dev eth0

# Delete ARP entry
ip neigh del 192.168.1.50 dev eth0

# Flush ARP cache
ip neigh flush dev eth0
```

### Comparing Old vs New Commands

| Old Command | New Command |
|-------------|-------------|
| `ifconfig` | `ip addr`, `ip link` |
| `ifconfig eth0 up` | `ip link set eth0 up` |
| `route` | `ip route` |
| `route add` | `ip route add` |
| `arp` | `ip neigh` |
| `netstat` | `ss` |

---

## ethtool - NIC Configuration

> **What it is:** `ethtool` queries and controls network driver and hardware settings—speed, duplex, wake-on-LAN, ring buffers, offload features, and driver information.

### Basic Information

```bash
# Show interface settings
ethtool eth0

# Example output:
# Settings for eth0:
#     Supported ports: [ TP ]
#     Supported link modes:   10baseT/Half 10baseT/Full
#                             100baseT/Half 100baseT/Full
#                             1000baseT/Full
#     Supported pause frame use: No
#     Supports auto-negotiation: Yes
#     Supported FEC modes: Not reported
#     Advertised link modes:  10baseT/Half 10baseT/Full
#                             100baseT/Half 100baseT/Full
#                             1000baseT/Full
#     Advertised pause frame use: No
#     Advertised auto-negotiation: Yes
#     Advertised FEC modes: Not reported
#     Speed: 1000Mb/s
#     Duplex: Full
#     Port: Twisted Pair
#     PHYAD: 1
#     Transceiver: internal
#     Auto-negotiation: on
#     MDI-X: on (auto)
#     Supports Wake-on: pumbg
#     Wake-on: g
#     Current message level: 0x00000007 (7)
#                            drv probe link
#     Link detected: yes

# Show driver information
ethtool -i eth0

# Output:
# driver: e1000e
# version: 3.8.7-k
# firmware-version: 0.5-4
# bus-info: 0000:00:1f.6
# supports-statistics: yes
# supports-test: yes
# supports-eeprom-access: yes
# supports-register-dump: yes
# supports-priv-flags: yes
```

### Understanding ethtool Output

| Field | Description |
|-------|-------------|
| `Supported ports` | Physical port types (TP=Twisted Pair, FIBRE, etc.) |
| `Supported link modes` | Speeds/duplex the NIC hardware supports |
| `Advertised link modes` | Speeds/duplex being advertised to link partner |
| `Speed` | Current negotiated speed |
| `Duplex` | Full (simultaneous TX/RX) or Half (one direction at a time) |
| `Port` | Physical port type in use |
| `PHYAD` | PHY address (physical layer chip) |
| `Transceiver` | Internal or external transceiver |
| `Auto-negotiation` | Whether auto-negotiation is enabled |
| `MDI-X` | Cable crossover mode (auto/manual) |
| `Supports Wake-on` | WoL capabilities (see below) |
| `Wake-on` | Currently enabled WoL mode |
| `Link detected` | Physical link status (cable connected) |

**Wake-on-LAN Flags:**

| Flag | Description |
|------|-------------|
| `p` | Wake on PHY activity |
| `u` | Wake on unicast messages |
| `m` | Wake on multicast messages |
| `b` | Wake on broadcast messages |
| `g` | Wake on magic packet |
| `s` | Enable SecureOn password |
| `d` | Disable (wake on nothing) |

### ethtool Driver Info Fields (-i)

| Field | Description |
|-------|-------------|
| `driver` | Kernel driver name |
| `version` | Driver version |
| `firmware-version` | NIC firmware version |
| `bus-info` | PCI bus location (domain:bus:slot.func) |
| `supports-statistics` | Driver provides detailed stats |
| `supports-test` | Self-test available |
| `supports-eeprom-access` | Can read/write NIC EEPROM |
| `supports-register-dump` | Can dump NIC registers |
| `supports-priv-flags` | Has private flags |

### ethtool Statistics (-S)

```bash
# Show NIC statistics
ethtool -S eth0

# Key statistics to watch:
# rx_packets          - Received packets
# tx_packets          - Transmitted packets
# rx_bytes            - Received bytes
# tx_bytes            - Transmitted bytes
# rx_errors           - Receive errors (bad packets)
# tx_errors           - Transmit errors
# rx_dropped          - Dropped RX (no buffer space)
# tx_dropped          - Dropped TX
# rx_missed_errors    - Packets missed (ring buffer full)
# rx_crc_errors       - CRC errors (cable/hardware issue)
# rx_frame_errors     - Frame alignment errors
# rx_fifo_errors      - FIFO overrun (NIC overwhelmed)
# tx_fifo_errors      - TX FIFO errors
# collisions          - Collisions (half-duplex only)
# tx_carrier_errors   - Carrier errors (link problems)
```

### Speed and Duplex

```bash
# Set speed and duplex manually
ethtool -s eth0 speed 1000 duplex full autoneg off

# Enable auto-negotiation
ethtool -s eth0 autoneg on

# Advertise specific speeds
ethtool -s eth0 advertise 0x020  # 1000baseT Full
```

### Statistics and Diagnostics

```bash
# Show NIC statistics
ethtool -S eth0

# Show all statistics (long output)
ethtool -S eth0 | head -50

# Show ring buffer sizes
ethtool -g eth0

# Set ring buffer sizes
ethtool -G eth0 rx 4096 tx 4096

# Show offload settings
ethtool -k eth0

# Enable/disable offload features
ethtool -K eth0 tso on          # TCP Segmentation Offload
ethtool -K eth0 gso on          # Generic Segmentation Offload
ethtool -K eth0 gro on          # Generic Receive Offload
ethtool -K eth0 tx-checksum-ipv4 on

# Run self-test (if supported)
ethtool -t eth0

# Blink LED (identify NIC)
ethtool -p eth0 5               # Blink for 5 seconds
```

### Wake-on-LAN

```bash
# Show WoL settings
ethtool eth0 | grep Wake

# Enable Wake-on-LAN
ethtool -s eth0 wol g           # g = magic packet

# Disable Wake-on-LAN
ethtool -s eth0 wol d

# WoL options:
# p - Wake on PHY activity
# u - Wake on unicast
# m - Wake on multicast
# b - Wake on broadcast
# g - Wake on magic packet
# d - Disable
```

### Persistent ethtool Settings

```bash
# For RHEL/CentOS - add to connection profile
nmcli con mod "eth0" ethtool.feature-tso on

# Or create udev rule
# /etc/udev/rules.d/70-ethtool.rules
ACTION=="add", SUBSYSTEM=="net", NAME=="eth0", RUN+="/sbin/ethtool -K eth0 tso on"

# Or NetworkManager dispatcher script
# /etc/NetworkManager/dispatcher.d/99-ethtool
#!/bin/bash
if [ "$1" = "eth0" ] && [ "$2" = "up" ]; then
    ethtool -K eth0 tso on
fi
```

---

## tcpdump - Packet Capture

> **What it is:** `tcpdump` captures network packets for analysis. Essential for troubleshooting network issues, verifying traffic flow, and debugging applications.

### Basic Capture

```bash
# Capture on interface (requires root)
tcpdump -i eth0

# Capture all interfaces
tcpdump -i any

# Capture with verbose output
tcpdump -v -i eth0
tcpdump -vv -i eth0             # More verbose
tcpdump -vvv -i eth0            # Maximum verbosity

# Don't resolve hostnames (faster)
tcpdump -n -i eth0

# Don't resolve hostnames or ports
tcpdump -nn -i eth0

# Limit capture count
tcpdump -c 100 -i eth0          # Stop after 100 packets
```

### Filtering

```bash
# Filter by host
tcpdump -i eth0 host 192.168.1.100
tcpdump -i eth0 src host 192.168.1.100
tcpdump -i eth0 dst host 192.168.1.100

# Filter by network
tcpdump -i eth0 net 192.168.1.0/24

# Filter by port
tcpdump -i eth0 port 80
tcpdump -i eth0 port 80 or port 443
tcpdump -i eth0 src port 22
tcpdump -i eth0 dst port 443

# Filter by protocol
tcpdump -i eth0 tcp
tcpdump -i eth0 udp
tcpdump -i eth0 icmp
tcpdump -i eth0 arp

# Combined filters
tcpdump -i eth0 'host 192.168.1.100 and port 80'
tcpdump -i eth0 'src 192.168.1.100 and dst port 443'
tcpdump -i eth0 'tcp and port 22 and host 10.0.0.1'

# Exclude traffic
tcpdump -i eth0 'not port 22'
tcpdump -i eth0 'host 192.168.1.100 and not port 22'

# Filter by TCP flags
tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'    # SYN packets
tcpdump -i eth0 'tcp[tcpflags] & tcp-rst != 0'    # RST packets
tcpdump -i eth0 'tcp[tcpflags] == tcp-syn'        # SYN only (no ACK)
```

### Output Options

```bash
# Write to file (pcap format)
tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap

# Read with filters
tcpdump -r capture.pcap 'port 80'

# Show packet contents (ASCII)
tcpdump -A -i eth0

# Show packet contents (hex and ASCII)
tcpdump -X -i eth0

# Show packet contents (hex only)
tcpdump -xx -i eth0

# Timestamp formats
tcpdump -tttt -i eth0           # Human readable
tcpdump -ttttt -i eth0          # Delta from previous

# Rotate capture files
tcpdump -i eth0 -w capture-%H%M.pcap -G 3600 -W 24
# -G 3600 = rotate every hour
# -W 24 = keep 24 files

# Limit file size
tcpdump -i eth0 -w capture.pcap -C 100
# -C 100 = rotate at 100MB
```

### Common Scenarios

```bash
# Capture HTTP traffic
tcpdump -i eth0 -nn -A 'port 80 and (tcp[32:4] = 0x47455420 or tcp[32:4] = 0x504f5354)'

# Capture DNS queries
tcpdump -i eth0 -nn port 53

# Capture DHCP traffic
tcpdump -i eth0 -nn port 67 or port 68

# Capture SSH connections (just headers, not content)
tcpdump -i eth0 -nn 'port 22 and tcp[tcpflags] & tcp-syn != 0'

# Capture ICMP (ping)
tcpdump -i eth0 icmp

# Capture traffic between two hosts
tcpdump -i eth0 'host 192.168.1.10 and host 192.168.1.20'

# Capture large packets (possible fragmentation)
tcpdump -i eth0 'ip[2:2] > 1500'

# Watch for ARP traffic
tcpdump -i eth0 arp
```

---

## ss - Socket Statistics

> **What it is:** `ss` (socket statistics) is the modern replacement for `netstat`. It's faster and provides more detailed information about network connections.

### Basic Usage

```bash
# Show all connections
ss

# Show listening sockets
ss -l

# Show TCP connections
ss -t

# Show UDP connections
ss -u

# Show listening TCP ports
ss -tl

# Show listening UDP ports
ss -ul

# Show all listening ports (TCP + UDP)
ss -tul

# Include process info (requires root)
ss -tlp
ss -tulpn                       # Most common: TCP/UDP, listening, process, numeric
```

### Output Columns

```bash
# ss -tulpn output:
# State    Recv-Q  Send-Q  Local Address:Port  Peer Address:Port  Process
# LISTEN   0       128     0.0.0.0:22          0.0.0.0:*          users:(("sshd",pid=1234))

# States:
# LISTEN      - Waiting for connections
# ESTABLISHED - Active connection
# TIME-WAIT   - Waiting after close
# CLOSE-WAIT  - Remote closed, waiting for local close
```

### Filtering

```bash
# Filter by state
ss -t state established
ss -t state listening
ss -t state time-wait

# Filter by port
ss -tl 'sport = :22'            # Source port 22
ss -t 'dport = :443'            # Destination port 443
ss -t '( sport = :22 or dport = :22 )'

# Filter by address
ss -t 'src 192.168.1.100'
ss -t 'dst 10.0.0.0/8'

# Combined filters
ss -t 'dport = :443 and src 192.168.1.0/24'

# Show connections to specific port
ss -t dst :443
```

### Statistics

```bash
# Summary statistics
ss -s

# Output:
# Total: 234
# TCP:   45 (estab 12, closed 5, orphaned 0, timewait 3)
# UDP:   8

# Show timer information
ss -to

# Show memory usage
ss -tm

# Show detailed info
ss -ti                          # TCP internal info
```

### Comparison: ss vs netstat

| netstat | ss |
|---------|-----|
| `netstat -tulpn` | `ss -tulpn` |
| `netstat -an` | `ss -an` |
| `netstat -r` | `ip route` |
| `netstat -i` | `ip -s link` |
| `netstat -g` | `ip maddr` |

---

## firewall-cmd - Firewalld

> **What it is:** `firewall-cmd` is the CLI for firewalld, the default firewall on RHEL/CentOS/Fedora. It uses zones to define trust levels for network connections.

### Zone Management

```bash
# List all zones
firewall-cmd --get-zones

# Get default zone
firewall-cmd --get-default-zone

# Set default zone
firewall-cmd --set-default-zone=public

# Get active zones
firewall-cmd --get-active-zones

# List zone configuration
firewall-cmd --zone=public --list-all
firewall-cmd --list-all                  # Default zone
firewall-cmd --list-all-zones            # All zones

# Assign interface to zone
firewall-cmd --zone=trusted --change-interface=eth0 --permanent
```

### Service Management

```bash
# List available services
firewall-cmd --get-services

# List enabled services
firewall-cmd --list-services

# Add service (runtime only)
firewall-cmd --add-service=http
firewall-cmd --add-service=https

# Add service (permanent)
firewall-cmd --permanent --add-service=http
firewall-cmd --reload                    # Apply permanent changes

# Remove service
firewall-cmd --remove-service=http
firewall-cmd --permanent --remove-service=http
```

### Port Management

```bash
# List open ports
firewall-cmd --list-ports

# Add port (runtime)
firewall-cmd --add-port=8080/tcp

# Add port (permanent)
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload

# Add port range
firewall-cmd --permanent --add-port=5000-5100/tcp

# Remove port
firewall-cmd --permanent --remove-port=8080/tcp
```

### Rich Rules

```bash
# Add rich rule
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" accept'

# Allow specific source to specific port
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.0.5" port port="3306" protocol="tcp" accept'

# Reject traffic from source
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.100" reject'

# Rate limiting
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" service name="ssh" accept limit value="3/m"'

# List rich rules
firewall-cmd --list-rich-rules

# Remove rich rule
firewall-cmd --permanent --remove-rich-rule='rule family="ipv4" source address="192.168.1.0/24" accept'
```

### Masquerading and Port Forwarding

```bash
# Enable masquerading (NAT)
firewall-cmd --permanent --add-masquerade

# Port forwarding
firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080
firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toaddr=192.168.1.100
firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080:toaddr=192.168.1.100

# List forward ports
firewall-cmd --list-forward-ports
```

### Emergency and Troubleshooting

```bash
# Panic mode (block all traffic)
firewall-cmd --panic-on
firewall-cmd --panic-off
firewall-cmd --query-panic

# Reload firewall
firewall-cmd --reload

# Complete reload (lose runtime config)
firewall-cmd --complete-reload

# Check if running
firewall-cmd --state
systemctl status firewalld
```

---

## DNS Tools

> **What it is:** Tools for querying DNS servers, troubleshooting name resolution, and diagnosing DNS issues.

### dig - DNS Lookup

```bash
# Basic query
dig google.com

# Query specific record type
dig google.com A                # IPv4 address
dig google.com AAAA             # IPv6 address
dig google.com MX               # Mail servers
dig google.com NS               # Name servers
dig google.com TXT              # TXT records
dig google.com SOA              # Start of Authority
dig google.com ANY              # All records

# Short output
dig +short google.com

# Query specific DNS server
dig @8.8.8.8 google.com
dig @1.1.1.1 google.com

# Reverse lookup
dig -x 8.8.8.8

# Trace delegation
dig +trace google.com

# No recursion (query authoritative only)
dig +norecurse google.com

# Show query time
dig +stats google.com

# TCP query (instead of UDP)
dig +tcp google.com
```

### nslookup

```bash
# Basic query
nslookup google.com

# Query specific server
nslookup google.com 8.8.8.8

# Query record type
nslookup -type=mx google.com
nslookup -type=ns google.com

# Reverse lookup
nslookup 8.8.8.8
```

### host

```bash
# Basic query
host google.com

# Verbose output
host -v google.com

# Query specific type
host -t mx google.com
host -t ns google.com

# Query specific server
host google.com 8.8.8.8

# Reverse lookup
host 8.8.8.8
```

### getent (NSS lookup)

```bash
# Query using system resolver (respects /etc/nsswitch.conf)
getent hosts google.com
getent ahosts google.com        # All addresses

# This uses NSS, so it checks:
# - /etc/hosts
# - DNS
# - Other configured sources
```

### /etc/resolv.conf

```bash
# DNS configuration file
cat /etc/resolv.conf

# Example:
# nameserver 8.8.8.8
# nameserver 8.8.4.4
# search example.com

# On NetworkManager systems, managed by NM
# To override, use nmcli:
nmcli con mod "eth0" ipv4.dns "8.8.8.8 8.8.4.4"
nmcli con mod "eth0" ipv4.ignore-auto-dns yes
```

---

## iperf / iperf3 - Network Bandwidth Testing

> **What it is:** `iperf` and `iperf3` measure network bandwidth between two hosts. One host runs as server, the other as client. Essential for testing network throughput, latency, and jitter.

### Basic Usage

```bash
# Install
dnf install iperf3

# Server side (listen for connections)
iperf3 -s
iperf3 -s -p 5201              # Specify port (default: 5201)
iperf3 -s -D                   # Run as daemon

# Client side (connect to server)
iperf3 -c server_ip
iperf3 -c 192.168.1.100

# Output example:
# [ ID] Interval       Transfer     Bitrate
# [  5] 0.00-10.00 sec  1.10 GBytes  941 Mbits/sec  sender
# [  5] 0.00-10.00 sec  1.10 GBytes  940 Mbits/sec  receiver
```

### Test Options

```bash
# Test duration
iperf3 -c server_ip -t 30      # 30 seconds (default: 10)

# Parallel streams
iperf3 -c server_ip -P 4       # 4 parallel streams

# Reverse mode (server sends, client receives)
iperf3 -c server_ip -R

# Bidirectional test
iperf3 -c server_ip --bidir

# UDP test (default is TCP)
iperf3 -c server_ip -u
iperf3 -c server_ip -u -b 100M  # UDP with 100 Mbps target

# Set bandwidth target
iperf3 -c server_ip -b 500M    # Target 500 Mbps

# Report interval
iperf3 -c server_ip -i 2       # Report every 2 seconds

# Number of bytes to transmit
iperf3 -c server_ip -n 1G      # Transfer 1GB then stop

# Set buffer/window size
iperf3 -c server_ip -w 256K

# JSON output
iperf3 -c server_ip -J
iperf3 -c server_ip -J > results.json
```

### Common Scenarios

```bash
# Basic throughput test
iperf3 -c server -t 30

# Test maximum TCP throughput
iperf3 -c server -P 10 -t 60

# UDP latency/jitter test
iperf3 -c server -u -b 10M -t 30
# Look for: Jitter and Lost/Total Datagrams

# Test with specific MTU
iperf3 -c server -M 1400       # Set MSS (MTU - 40)

# One-way throughput from server to client
iperf3 -c server -R

# Bind to specific interface
iperf3 -c server -B 192.168.1.10

# Test through firewall (use allowed port)
iperf3 -s -p 443               # Server on port 443
iperf3 -c server -p 443        # Client connects to 443
```

### Reading iperf3 Output

```bash
# TCP output columns:
# Interval    - Time period for this row
# Transfer    - Data transferred in interval
# Bitrate     - Throughput (this is what you care about)
# Retr        - TCP retransmits (high = congestion/packet loss)
# Cwnd        - TCP congestion window

# UDP output columns:
# Interval    - Time period
# Transfer    - Data sent
# Bitrate     - Throughput
# Jitter      - Variation in latency (lower = better)
# Lost/Total  - Packet loss (0% = good)
```

---

## TRex - Traffic Generator

> **What it is:** TRex is a high-performance, open-source traffic generator developed by Cisco. It can generate stateless or stateful traffic at line rate using DPDK. Used for network performance testing, NFV/SDN testing, and benchmarking.

### Installation

```bash
# Download TRex
cd /opt
wget --no-check-certificate https://trex-tgn.cisco.com/trex/release/latest
tar -xzvf latest
cd v3.xx                        # Version directory

# Check system requirements
./dpdk_setup_ports.py -s        # Show NIC info

# Bind NICs to DPDK
./dpdk_setup_ports.py -i        # Interactive setup
# Or manual:
./dpdk_setup_ports.py -b igb_uio eth1 eth2
```

### Configuration

```yaml
# /etc/trex_cfg.yaml
- port_limit: 2
  version: 2
  interfaces: ["eth1", "eth2"]  # Or PCI addresses: ["0000:03:00.0", "0000:03:00.1"]
  port_info:
    - ip: 192.168.1.1
      default_gw: 192.168.1.254
    - ip: 192.168.2.1
      default_gw: 192.168.2.254
  platform:
    master_thread_id: 0
    latency_thread_id: 1
    dual_if:
      - socket: 0
        threads: [2,3,4,5]
```

### Starting TRex

```bash
# Stateless mode (most common)
./t-rex-64 -i                   # Interactive mode
./t-rex-64 -i -c 4              # With 4 cores

# Stateful mode
./t-rex-64 -f cap2/dns.yaml     # Run traffic profile

# With specific config
./t-rex-64 -i --cfg /etc/trex_cfg.yaml

# Background/daemon mode
./t-rex-64 -i -d

# Check TRex is running
./trex-console                  # Connect to running TRex
```

### TRex Console Commands

```bash
# Connect to TRex
./trex-console

# Basic commands
trex> portattr -a               # Show all port attributes
trex> stats -p 0                # Show port 0 stats
trex> stats -g                  # Global stats
trex> clear                     # Clear stats

# Start/stop traffic
trex> start -f stl/bench.py     # Start traffic profile
trex> start -f stl/bench.py -m 10gbps  # At 10Gbps
trex> start -f stl/bench.py -m 1mpps   # At 1M packets/sec
trex> stop -a                   # Stop all ports
trex> pause -a                  # Pause traffic
trex> resume -a                 # Resume traffic

# Port operations
trex> port -a                   # Show all ports
trex> portattr -p 0 --prom on   # Promiscuous mode
trex> service -p 0              # Enable service mode (ARP, ping)
trex> l3 -p 0 --src 192.168.1.1 --dst 192.168.1.2  # Set L3 mode

# Capture
trex> capture monitor start --rx 0  # Monitor RX on port 0
trex> capture record start --rx 0 -o /tmp/cap.pcap  # Record to pcap
```

### Traffic Profiles (Python)

```python
# Simple UDP traffic profile: stl/my_traffic.py
from trex_stl_lib.api import *

class STLS1(object):
    def create_stream(self):
        # Create packet
        pkt = Ether()/IP(src="16.0.0.1",dst="48.0.0.1")/UDP(dport=12,sport=1025)/('x'*60)
        
        # Create stream
        return STLStream(
            packet = STLPktBuilder(pkt = pkt),
            mode = STLTXCont(pps = 1000)  # 1000 packets per second
        )

    def get_streams(self, direction=0, **kwargs):
        return [self.create_stream()]

# Register
def register():
    return STLS1()
```

```python
# Burst traffic profile
from trex_stl_lib.api import *

class STLBurst(object):
    def create_stream(self):
        pkt = Ether()/IP()/UDP()
        
        return STLStream(
            packet = STLPktBuilder(pkt = pkt),
            mode = STLTXSingleBurst(total_pkts=1000, pps=10000)
        )

    def get_streams(self, direction=0, **kwargs):
        return [self.create_stream()]

def register():
    return STLBurst()
```

### TRex Python API (Automation)

```python
#!/usr/bin/env python
from trex_stl_lib.api import *

# Connect to TRex
c = STLClient(server='127.0.0.1')
c.connect()

# Prepare ports
c.reset(ports=[0, 1])

# Load traffic profile
c.add_streams(streams=STLStream(
    packet=STLPktBuilder(
        pkt=Ether()/IP(dst="192.168.1.100")/UDP()
    ),
    mode=STLTXCont(pps=100000)  # 100K pps
), ports=[0])

# Start traffic
c.start(ports=[0], mult="1gbps", duration=60)

# Wait and get stats
c.wait_on_traffic(ports=[0])
stats = c.get_stats()
print(f"TX packets: {stats[0]['opackets']}")
print(f"RX packets: {stats[1]['ipackets']}")

# Disconnect
c.disconnect()
```

### Common Traffic Profiles

```bash
# Built-in profiles in stl/ directory:
stl/bench.py              # Benchmark traffic
stl/udp_1pkt_simple.py    # Simple UDP
stl/imix.py               # Internet mix (IMIX)
stl/syn_attack.py         # SYN flood (testing only!)
stl/pcap.py               # Replay PCAP file

# Run with rate multiplier
trex> start -f stl/bench.py -m 100%    # 100% line rate
trex> start -f stl/bench.py -m 10gbps  # 10 Gbps
trex> start -f stl/bench.py -m 5mpps   # 5M packets/sec
```

### Performance Tuning

```bash
# CPU pinning
./t-rex-64 -i -c 8              # Use 8 cores

# NUMA awareness
./t-rex-64 -i --cfg /etc/trex_cfg.yaml  # Configure in YAML

# Huge pages (required for DPDK)
echo 1024 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
mkdir -p /mnt/huge
mount -t hugetlbfs nodev /mnt/huge

# Check hugepages
cat /proc/meminfo | grep Huge

# Verify DPDK binding
./dpdk_setup_ports.py -s
```

### Useful TRex Commands Summary

| Command | Description |
|---------|-------------|
| `./t-rex-64 -i` | Start TRex interactive |
| `./trex-console` | Connect to TRex |
| `portattr -a` | Show port attributes |
| `stats -g` | Global statistics |
| `start -f <file> -m <rate>` | Start traffic |
| `stop -a` | Stop all traffic |
| `service -p 0` | Enable service mode |
| `capture` | Packet capture |

---

## nmap - Network Scanner

> **What it is:** `nmap` (Network Mapper) is a powerful network discovery and security auditing tool. It scans hosts for open ports, services, OS detection, and vulnerabilities.

### Basic Scanning

```bash
# Install
dnf install nmap

# Scan single host
nmap 192.168.1.100
nmap hostname.example.com

# Scan multiple hosts
nmap 192.168.1.100 192.168.1.101
nmap 192.168.1.100-110          # Range
nmap 192.168.1.0/24             # Subnet

# Scan from file
nmap -iL hosts.txt
```

### Scan Types

```bash
# TCP SYN scan (default, requires root)
nmap -sS 192.168.1.100

# TCP connect scan (no root needed)
nmap -sT 192.168.1.100

# UDP scan
nmap -sU 192.168.1.100

# TCP + UDP scan
nmap -sS -sU 192.168.1.100

# Ping scan (host discovery only)
nmap -sn 192.168.1.0/24
nmap -sP 192.168.1.0/24         # Older syntax

# Skip ping (scan even if host doesn't respond to ping)
nmap -Pn 192.168.1.100

# ACK scan (detect firewall rules)
nmap -sA 192.168.1.100

# FIN scan (stealthy)
nmap -sF 192.168.1.100
```

### Port Specification

```bash
# Scan specific ports
nmap -p 22 192.168.1.100
nmap -p 22,80,443 192.168.1.100
nmap -p 1-1000 192.168.1.100

# Scan all 65535 ports
nmap -p- 192.168.1.100

# Scan common ports (fast)
nmap -F 192.168.1.100           # Top 100 ports
nmap --top-ports 1000 192.168.1.100

# Scan specific port ranges
nmap -p 22,80,443,8000-9000 192.168.1.100
```

### Service and Version Detection

```bash
# Service version detection
nmap -sV 192.168.1.100

# OS detection
nmap -O 192.168.1.100

# Aggressive scan (OS, version, scripts, traceroute)
nmap -A 192.168.1.100

# Increase version detection intensity
nmap -sV --version-intensity 5 192.168.1.100
```

### Output Options

```bash
# Normal output to file
nmap -oN scan.txt 192.168.1.100

# XML output
nmap -oX scan.xml 192.168.1.100

# Grepable output
nmap -oG scan.grep 192.168.1.100

# All formats
nmap -oA scan 192.168.1.100     # Creates scan.nmap, scan.xml, scan.gnmap

# Verbose output
nmap -v 192.168.1.100
nmap -vv 192.168.1.100          # More verbose
```

### Timing and Performance

```bash
# Timing templates (0=paranoid, 5=insane)
nmap -T0 192.168.1.100          # Slowest, stealthiest
nmap -T3 192.168.1.100          # Default
nmap -T4 192.168.1.100          # Aggressive (recommended for local network)
nmap -T5 192.168.1.100          # Fastest

# Rate limiting
nmap --max-rate 100 192.168.1.0/24    # Max 100 packets/sec
nmap --min-rate 50 192.168.1.0/24     # Min 50 packets/sec
```

### NSE Scripts

```bash
# List available scripts
ls /usr/share/nmap/scripts/
nmap --script-help default

# Run default scripts
nmap -sC 192.168.1.100

# Run specific script
nmap --script http-headers 192.168.1.100
nmap --script ssl-cert -p 443 192.168.1.100

# Run script category
nmap --script vuln 192.168.1.100       # Vulnerability scripts
nmap --script safe 192.168.1.100       # Safe scripts
nmap --script discovery 192.168.1.100  # Discovery scripts

# Run multiple scripts
nmap --script "http-* and not http-brute" 192.168.1.100
```

### Common Use Cases

```bash
# Quick network inventory
nmap -sn 192.168.1.0/24

# Find all web servers
nmap -p 80,443,8080,8443 192.168.1.0/24

# Full scan single host
nmap -A -T4 192.168.1.100

# Check if specific port is open
nmap -p 22 192.168.1.100

# Scan for SSH servers
nmap -p 22 --open 192.168.1.0/24

# Detect service versions on common ports
nmap -sV -p 22,80,443,3306,5432 192.168.1.100

# Scan behind firewall (no ping)
nmap -Pn -sS -p 22,80,443 192.168.1.100
```

---

## nstat / netstat - Network Statistics

> **What it is:** `nstat` shows kernel SNMP counters and network statistics. `netstat` (legacy) shows connections, routing, interface stats. Modern replacement is `ss` for connections and `nstat` for counters.

### nstat - Kernel Network Statistics

```bash
# Show all statistics
nstat

# Show non-zero statistics only
nstat -z

# Show absolute values (not rates)
nstat -a

# Continuous monitoring (every 2 seconds)
nstat -n 2

# Reset statistics after reading
nstat -r

# JSON output
nstat -j

# Show specific pattern
nstat | grep -i tcp
nstat | grep -i retrans
nstat | grep -i error
```

### Understanding nstat Output

```bash
# Key TCP statistics:
# TcpActiveOpens    - Connections initiated (client)
# TcpPassiveOpens   - Connections received (server)
# TcpInSegs         - Segments received
# TcpOutSegs        - Segments sent
# TcpRetransSegs    - Retransmitted segments (watch this!)
# TcpInErrs         - Segments received in error

# Key IP statistics:
# IpInReceives      - Packets received
# IpInDelivers      - Packets delivered to upper layer
# IpOutRequests     - Packets sent
# IpInDiscards      - Input packets discarded
# IpOutDiscards     - Output packets discarded

# Key UDP statistics:
# UdpInDatagrams    - Datagrams received
# UdpOutDatagrams   - Datagrams sent
# UdpInErrors       - Receive errors
# UdpNoPorts        - No application on port
```

### netstat (Legacy)

```bash
# Show all connections
netstat -a

# Show TCP connections
netstat -t
netstat -at                     # All TCP

# Show UDP connections
netstat -u
netstat -au                     # All UDP

# Show listening ports
netstat -l
netstat -tl                     # TCP listening
netstat -ul                     # UDP listening
netstat -tuln                   # TCP+UDP listening, numeric

# Show with process info
netstat -tulpn                  # Most common usage

# Show routing table
netstat -r
netstat -rn                     # Numeric

# Show interface statistics
netstat -i
netstat -ie                     # Extended (like ifconfig)

# Show network statistics
netstat -s                      # Protocol statistics
netstat -st                     # TCP statistics
netstat -su                     # UDP statistics

# Continuous display
netstat -c                      # Update every second
```

---

## Additional Network Tools

### ping / ping6 - Connectivity Test

```bash
# Basic ping
ping 8.8.8.8
ping -c 4 8.8.8.8              # 4 packets only

# IPv6
ping6 ::1
ping -6 google.com

# Set interval
ping -i 0.2 8.8.8.8            # 200ms interval

# Set packet size
ping -s 1472 8.8.8.8           # Test MTU (1472 + 28 = 1500)

# Flood ping (requires root, use carefully)
ping -f 192.168.1.1            # Flood
ping -f -c 1000 192.168.1.1    # 1000 packets flood

# Set TTL
ping -t 10 8.8.8.8

# Don't fragment (MTU discovery)
ping -M do -s 1472 8.8.8.8

# Record route
ping -R 8.8.8.8

# Quiet output (summary only)
ping -q -c 10 8.8.8.8
```

### traceroute / tracepath - Path Discovery

```bash
# Trace route to host
traceroute google.com
traceroute -n google.com       # Numeric (no DNS)

# Use ICMP instead of UDP
traceroute -I google.com

# Use TCP (port 80)
traceroute -T -p 80 google.com

# Set max hops
traceroute -m 20 google.com

# tracepath (no root needed)
tracepath google.com
tracepath -n google.com        # Numeric

# mtr (combines ping + traceroute)
mtr google.com
mtr -n google.com              # Numeric
mtr -r -c 10 google.com        # Report mode, 10 cycles
```

### curl - HTTP/API Testing

```bash
# Basic GET request
curl http://example.com
curl -s http://example.com     # Silent (no progress)

# Show response headers
curl -I http://example.com     # HEAD request
curl -i http://example.com     # Include headers in output

# Follow redirects
curl -L http://example.com

# POST request
curl -X POST -d "data=value" http://example.com/api
curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' http://api.example.com

# Download file
curl -O http://example.com/file.tar.gz
curl -o myfile.tar.gz http://example.com/file.tar.gz

# With authentication
curl -u user:password http://example.com
curl -H "Authorization: Bearer TOKEN" http://api.example.com

# Verbose output
curl -v http://example.com

# Test connection time
curl -o /dev/null -s -w "Connect: %{time_connect}s\nTTFB: %{time_starttransfer}s\nTotal: %{time_total}s\n" http://example.com

# Test HTTPS certificate
curl -vI https://example.com 2>&1 | grep -A6 "Server certificate"

# Ignore SSL errors
curl -k https://self-signed.example.com
```

### wget - File Download

```bash
# Download file
wget http://example.com/file.tar.gz

# Download to specific name
wget -O myfile.tar.gz http://example.com/file.tar.gz

# Continue interrupted download
wget -c http://example.com/largefile.iso

# Download in background
wget -b http://example.com/file.tar.gz

# Mirror website
wget -m http://example.com

# Limit download speed
wget --limit-rate=1M http://example.com/file.tar.gz

# Recursive download
wget -r -l 2 http://example.com   # 2 levels deep
```

### nc (netcat) - Network Swiss Army Knife

```bash
# Test if port is open
nc -zv 192.168.1.100 22
nc -zv 192.168.1.100 20-30     # Port range

# Simple server (listen on port)
nc -l 1234

# Connect to server
nc 192.168.1.100 1234

# Transfer file
# Server: nc -l 1234 > received_file
# Client: nc server_ip 1234 < file_to_send

# Port scanning
nc -zv 192.168.1.100 1-1000

# UDP mode
nc -u 192.168.1.100 53

# Chat between two hosts
# Host A: nc -l 1234
# Host B: nc host_a 1234

# HTTP request
echo -e "GET / HTTP/1.1\r\nHost: example.com\r\n\r\n" | nc example.com 80
```

### arping - ARP Ping

```bash
# ARP ping (layer 2)
arping 192.168.1.1
arping -c 4 192.168.1.1        # 4 packets

# Specify interface
arping -I eth0 192.168.1.1

# Detect duplicate IP
arping -D 192.168.1.100

# Find MAC address
arping -c 1 192.168.1.1 | grep "reply from"
```

### arp - ARP Table

```bash
# Show ARP table
arp -a
arp -n                         # Numeric

# Add static entry
arp -s 192.168.1.100 00:11:22:33:44:55

# Delete entry
arp -d 192.168.1.100

# Modern alternative
ip neigh
ip neigh show
```

### lsof - Network Connections

```bash
# Show all network connections
lsof -i

# Show TCP connections
lsof -i TCP

# Show UDP connections
lsof -i UDP

# Show connections to specific port
lsof -i :80
lsof -i :22

# Show connections by process
lsof -i -P -n | grep ssh

# Show listening ports
lsof -i -P -n | grep LISTEN

# Show established connections
lsof -i -P -n | grep ESTABLISHED

# Show connections for specific PID
lsof -i -a -p 1234
```

### iftop / nethogs - Live Bandwidth Monitoring

```bash
# Install
dnf install iftop nethogs

# iftop - bandwidth by connection
iftop
iftop -i eth0                  # Specific interface
iftop -n                       # No hostname resolution
iftop -P                       # Show ports

# nethogs - bandwidth by process
nethogs
nethogs eth0                   # Specific interface
```

### bmon / vnstat - Bandwidth Monitoring

```bash
# Install
dnf install bmon vnstat

# bmon - real-time bandwidth monitor
bmon
bmon -p eth0                   # Specific interface

# vnstat - traffic accounting
vnstat                         # Summary
vnstat -l                      # Live monitor
vnstat -h                      # Hourly stats
vnstat -d                      # Daily stats
vnstat -m                      # Monthly stats
vnstat -t                      # Top days
```

### ip statistics

```bash
# Interface statistics
ip -s link
ip -s link show eth0

# Detailed statistics
ip -s -s link show eth0        # More detail

# Address statistics
ip -s addr

# Route cache statistics
ip route show cache
```

---

## Quick Reference

### Common Commands

| Task | Command |
|------|---------|
| Show IP addresses | `ip addr` or `ip a` |
| Show interfaces | `ip link` |
| Show routes | `ip route` |
| Show ARP table | `ip neigh` |
| Show connections | `nmcli con show` |
| Show devices | `nmcli dev status` |
| Listening ports | `ss -tulpn` |
| Capture packets | `tcpdump -i eth0` |
| NIC info | `ethtool eth0` |
| Firewall rules | `firewall-cmd --list-all` |
| DNS lookup | `dig example.com` |
| Bandwidth test | `iperf3 -c server` |
| Port scan | `nmap -p 22,80,443 host` |
| Network stats | `nstat` |
| Test connectivity | `ping -c 4 host` |
| Trace route | `traceroute host` |
| Test port open | `nc -zv host port` |

### Network Configuration Files

| File | Purpose |
|------|---------|
| `/etc/NetworkManager/system-connections/` | NM connection profiles |
| `/etc/resolv.conf` | DNS resolver config |
| `/etc/hosts` | Static host mappings |
| `/etc/hostname` | System hostname |
| `/etc/sysconfig/network-scripts/` | Legacy network scripts (RHEL 7) |

### Quick Troubleshooting

```bash
# Check connectivity
ping -c 4 8.8.8.8               # Internet reachability
ping -c 4 gateway_ip            # Gateway reachability
traceroute 8.8.8.8              # Path to destination

# Check DNS
dig +short google.com           # DNS resolution
cat /etc/resolv.conf            # DNS servers

# Check listening services
ss -tulpn                       # What's listening
ss -tulpn | grep :80            # Specific port
lsof -i :80                     # Process using port

# Check firewall
firewall-cmd --list-all         # Firewall rules

# Check interface
ip addr show eth0               # IP configuration
ethtool eth0                    # Link status
ip -s link show eth0            # Interface statistics

# Port/service scanning
nc -zv host 22                  # Test if port open
nmap -p 22,80,443 host          # Scan multiple ports
nmap -sn 192.168.1.0/24         # Discover hosts on network

# Bandwidth testing
iperf3 -s                       # Start server
iperf3 -c server_ip             # Test from client

# Packet capture for specific issue
tcpdump -i eth0 -nn port 80     # HTTP traffic
tcpdump -i eth0 -nn host 10.0.0.1  # Specific host

# Network statistics
nstat                           # Kernel network stats
nstat | grep -i retrans         # Check for retransmissions
```

---

*Part of the [Linux Documentation](README.md)*

