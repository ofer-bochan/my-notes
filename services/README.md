# Common Services and Applications

> **What this section covers:** Configuration and management of common Linux services including DNS servers, web servers, and databases.

## Topics

| Topic | Description |
|-------|-------------|
| [dnsmasq](01-dnsmasq.md) | Lightweight DNS, DHCP, and TFTP server for local networks |
| [Apache HTTP Server](02-apache.md) | World's most widely used web server - virtual hosts, SSL, proxying |
| [SQL Basics](03-sql.md) | Structured Query Language for relational databases |
| [Ansible](04-ansible.md) | Agentless automation - playbooks, roles, inventory, modules |

## Quick Reference

### Which service for what?

| Need | Service | Port |
|------|---------|------|
| Local DNS resolution | dnsmasq | 53 |
| DHCP for local network | dnsmasq | 67/68 |
| Static website | Apache | 80/443 |
| Reverse proxy | Apache/Nginx | 80/443 |
| Store structured data | MySQL/PostgreSQL | 3306/5432 |

### Common Commands

```bash
# Check service status
systemctl status dnsmasq
systemctl status httpd
systemctl status mariadb

# View logs
journalctl -u dnsmasq -f
tail -f /var/log/httpd/error_log
journalctl -u mariadb -f

# Test configs
dnsmasq --test
apachectl configtest
```

---

*Part of the [Technical Documentation](../README.md)*

