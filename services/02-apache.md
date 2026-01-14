# Apache HTTP Server

> **What it is:** Apache HTTP Server (httpd) is the world's most widely used web server software. It's modular, extensible, and supports virtual hosts, SSL/TLS, proxying, URL rewriting, and more.

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Apache Architecture                                  │
│                                                                              │
│   Client Request ──► Port 80/443                                            │
│                          │                                                   │
│                          ▼                                                   │
│   ┌─────────────────────────────────────────────────────────────────┐       │
│   │                    Apache httpd                                  │       │
│   │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │       │
│   │  │   MPM       │  │   Modules   │  │   Virtual   │             │       │
│   │  │ (prefork/   │  │  (mod_ssl,  │  │   Hosts     │             │       │
│   │  │  worker/    │  │  mod_proxy, │  │             │             │       │
│   │  │  event)     │  │  mod_rewrite│  │             │             │       │
│   │  └─────────────┘  └─────────────┘  └─────────────┘             │       │
│   └─────────────────────────────────────────────────────────────────┘       │
│                          │                                                   │
│                          ▼                                                   │
│              ┌──────────────────────┐                                       │
│              │   Document Root      │                                       │
│              │   /var/www/html      │                                       │
│              └──────────────────────┘                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Installation

```bash
# RHEL/CentOS/Fedora
sudo dnf install httpd mod_ssl
sudo systemctl enable --now httpd

# Ubuntu/Debian
sudo apt install apache2
sudo systemctl enable --now apache2

# Check status
systemctl status httpd   # RHEL
systemctl status apache2 # Ubuntu
```

## Configuration Files (RHEL/CentOS)

```
/etc/httpd/
├── conf/
│   └── httpd.conf           # Main configuration
├── conf.d/
│   ├── ssl.conf             # SSL/TLS settings
│   ├── welcome.conf         # Default welcome page
│   └── *.conf               # Additional configs
├── conf.modules.d/
│   └── *.conf               # Module loading
└── logs -> /var/log/httpd   # Symlink to logs
```

## Basic Configuration

```apache
# /etc/httpd/conf/httpd.conf

# Server root directory
ServerRoot "/etc/httpd"

# Listen on port 80
Listen 80

# Server admin email
ServerAdmin root@localhost

# Server name (set your domain)
ServerName www.example.com:80

# Document root
DocumentRoot "/var/www/html"

# Directory permissions
<Directory "/var/www/html">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>

# Default file to serve
DirectoryIndex index.html index.php

# Log files
ErrorLog "logs/error_log"
CustomLog "logs/access_log" combined
```

## Virtual Hosts

### Name-based Virtual Hosts

```apache
# /etc/httpd/conf.d/vhosts.conf

# Site 1: example.com
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com/public_html
    
    ErrorLog /var/log/httpd/example.com-error.log
    CustomLog /var/log/httpd/example.com-access.log combined
    
    <Directory /var/www/example.com/public_html>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>

# Site 2: app.example.com
<VirtualHost *:80>
    ServerName app.example.com
    DocumentRoot /var/www/app.example.com/public_html
    
    ErrorLog /var/log/httpd/app-error.log
    CustomLog /var/log/httpd/app-access.log combined
    
    <Directory /var/www/app.example.com/public_html>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

## SSL/TLS Configuration

```apache
# /etc/httpd/conf.d/ssl-vhost.conf

<VirtualHost *:443>
    ServerName secure.example.com
    DocumentRoot /var/www/secure/public_html
    
    # Enable SSL
    SSLEngine on
    
    # Certificate files
    SSLCertificateFile /etc/pki/tls/certs/server.crt
    SSLCertificateKeyFile /etc/pki/tls/private/server.key
    SSLCertificateChainFile /etc/pki/tls/certs/chain.crt
    
    # Modern SSL settings
    SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
    SSLCipherSuite ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256
    SSLHonorCipherOrder off
    
    # HSTS header
    Header always set Strict-Transport-Security "max-age=31536000"
    
    <Directory /var/www/secure/public_html>
        Require all granted
    </Directory>
</VirtualHost>

# Redirect HTTP to HTTPS
<VirtualHost *:80>
    ServerName secure.example.com
    Redirect permanent / https://secure.example.com/
</VirtualHost>
```

## Common Modules

```bash
# List loaded modules
httpd -M

# Key modules
mod_ssl        # HTTPS support
mod_rewrite    # URL rewriting
mod_proxy      # Reverse proxy
mod_headers    # Custom headers
mod_security   # Web application firewall
mod_deflate    # Compression
```

## URL Rewriting

```apache
# Enable rewrite engine
RewriteEngine On

# Redirect www to non-www
RewriteCond %{HTTP_HOST} ^www\.(.*)$ [NC]
RewriteRule ^(.*)$ http://%1/$1 [R=301,L]

# Redirect HTTP to HTTPS
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}/$1 [R=301,L]

# Pretty URLs (remove .php extension)
RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME}\.php -f
RewriteRule ^(.*)$ $1.php [L]

# Block specific user agents
RewriteCond %{HTTP_USER_AGENT} (bot|spider) [NC]
RewriteRule .* - [F,L]
```

## Reverse Proxy

```apache
# /etc/httpd/conf.d/proxy.conf

# Proxy to backend application
<VirtualHost *:80>
    ServerName app.example.com
    
    ProxyPreserveHost On
    ProxyPass / http://localhost:3000/
    ProxyPassReverse / http://localhost:3000/
    
    # WebSocket support
    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) ws://localhost:3000/$1 [P,L]
</VirtualHost>

# Load balancer
<Proxy "balancer://mycluster">
    BalancerMember http://backend1:8080
    BalancerMember http://backend2:8080
    BalancerMember http://backend3:8080
    ProxySet lbmethod=byrequests
</Proxy>

<VirtualHost *:80>
    ServerName lb.example.com
    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/
</VirtualHost>
```

## Security Settings

```apache
# /etc/httpd/conf.d/security.conf

# Hide Apache version
ServerTokens Prod
ServerSignature Off

# Disable directory listing
Options -Indexes

# Prevent clickjacking
Header always set X-Frame-Options "SAMEORIGIN"

# XSS Protection
Header always set X-XSS-Protection "1; mode=block"

# Prevent MIME type sniffing
Header always set X-Content-Type-Options "nosniff"

# Restrict HTTP methods
<LimitExcept GET POST HEAD>
    Require all denied
</LimitExcept>

# Block access to sensitive files
<FilesMatch "^\.ht">
    Require all denied
</FilesMatch>
```

## Commands

```bash
# Test configuration syntax
apachectl configtest
httpd -t

# Check virtual host configuration
httpd -S

# List loaded modules
httpd -M

# Start/Stop/Restart
systemctl start httpd
systemctl stop httpd
systemctl restart httpd
systemctl reload httpd  # Graceful reload

# View logs
tail -f /var/log/httpd/access_log
tail -f /var/log/httpd/error_log

# Check what's listening on port 80
ss -tuln | grep :80
```

## Troubleshooting

```bash
# Check config
apachectl configtest

# Check error log
tail -100 /var/log/httpd/error_log

# Check SELinux context
ls -Z /var/www/html/

# Fix SELinux for custom directory
semanage fcontext -a -t httpd_sys_content_t "/custom/path(/.*)?"
restorecon -Rv /custom/path

# Allow Apache to connect to network (for proxy)
setsebool -P httpd_can_network_connect 1

# Firewall
firewall-cmd --add-service=http --permanent
firewall-cmd --add-service=https --permanent
firewall-cmd --reload
```

## Common Issues

| Problem | Solution |
|---------|----------|
| 403 Forbidden | Check directory permissions and SELinux context |
| 500 Internal Server Error | Check error_log and file permissions |
| Proxy not working | Enable `mod_proxy` and set `httpd_can_network_connect` |
| SSL certificate errors | Verify cert paths and permissions |
| Virtual host not matching | Check `ServerName` and DNS resolution |

---

*Part of the [Services Documentation](README.md)*

