# Salt (SaltStack) for Container Management

> **What it is:** Salt (SaltStack) is a configuration management and orchestration tool that can manage infrastructure at scale. It uses a master-minion architecture and can manage containers, VMs, cloud resources, and more.

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Salt Architecture                                    │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────┐       │
│   │                       SALT MASTER                                │       │
│   │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐    │       │
│   │  │ State     │  │ Pillar    │  │ Modules   │  │ Grains    │    │       │
│   │  │ Files     │  │ Data      │  │ (Docker,  │  │ (Facts)   │    │       │
│   │  │ (.sls)    │  │ (secrets) │  │ Podman)   │  │           │    │       │
│   │  └───────────┘  └───────────┘  └───────────┘  └───────────┘    │       │
│   └───────────────────────────┬─────────────────────────────────────┘       │
│                               │ ZeroMQ / SSH                                 │
│              ┌────────────────┼────────────────┐                            │
│              ▼                ▼                ▼                            │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                     │
│   │ SALT MINION  │  │ SALT MINION  │  │ SALT MINION  │                     │
│   │ (Container   │  │ (Container   │  │ (Container   │                     │
│   │  Host 1)     │  │  Host 2)     │  │  Host 3)     │                     │
│   └──────────────┘  └──────────────┘  └──────────────┘                     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Installation

```bash
# Install Salt Master (on control node)
# RHEL/CentOS
sudo dnf install salt-master salt-minion

# Ubuntu/Debian
sudo apt install salt-master salt-minion

# Start services
sudo systemctl enable --now salt-master
sudo systemctl enable --now salt-minion

# On minion, configure master
echo "master: salt-master.example.com" > /etc/salt/minion.d/master.conf
systemctl restart salt-minion

# Accept minion key on master
salt-key -L              # List keys
salt-key -A              # Accept all pending
salt-key -a minion-name  # Accept specific
```

## Basic Commands

```bash
# Test connectivity
salt '*' test.ping

# Run command on all minions
salt '*' cmd.run 'hostname'

# Target specific minions
salt 'web*' cmd.run 'uptime'           # Glob pattern
salt -G 'os:CentOS' cmd.run 'uptime'   # By grain
salt -L 'web1,web2,web3' test.ping     # List of minions

# Get system information (grains)
salt '*' grains.items
salt '*' grains.get os
```

## Docker Module

```bash
# Check Docker on minions
salt '*' docker.version

# List containers
salt '*' docker.ps_
salt '*' docker.ps_ all=True   # Include stopped

# List images
salt '*' docker.images

# Pull image
salt '*' docker.pull nginx:latest
salt '*' docker.pull quay.io/myorg/myapp:1.0

# Run container
salt '*' docker.run nginx:latest name=web-server ports="80:80" detach=True

# Stop and remove
salt '*' docker.stop web-server
salt '*' docker.rm web-server

# View logs
salt '*' docker.logs web-server tail=100
```

## Salt States for Containers

### Install Docker

```yaml
# /srv/salt/docker/init.sls
docker_repo:
  pkgrepo.managed:
    - name: docker-ce
    - humanname: Docker CE Repository
    - baseurl: https://download.docker.com/linux/centos/$releasever/$basearch/stable
    - gpgcheck: 1
    - gpgkey: https://download.docker.com/linux/centos/gpg

docker_packages:
  pkg.installed:
    - pkgs:
      - docker-ce
      - docker-ce-cli
      - containerd.io
    - require:
      - pkgrepo: docker_repo

docker_service:
  service.running:
    - name: docker
    - enable: True
    - require:
      - pkg: docker_packages
```

### Manage Nginx Container

```yaml
# /srv/salt/nginx/init.sls
include:
  - docker

nginx_image:
  docker_image.present:
    - name: nginx:1.25-alpine
    - require:
      - service: docker_service

nginx_container:
  docker_container.running:
    - name: nginx-web
    - image: nginx:1.25-alpine
    - ports:
      - 80:80
      - 443:443
    - binds:
      - /var/www/html:/usr/share/nginx/html:ro
    - restart_policy: always
    - require:
      - docker_image: nginx_image
```

### Manage Database Container

```yaml
# /srv/salt/mysql/init.sls
mysql_image:
  docker_image.present:
    - name: mysql:8.0

mysql_container:
  docker_container.running:
    - name: mysql-db
    - image: mysql:8.0
    - ports:
      - 3306:3306
    - environment:
      - MYSQL_ROOT_PASSWORD: {{ pillar['mysql']['root_password'] }}
      - MYSQL_DATABASE: {{ pillar['mysql']['database'] }}
    - volumes:
      - mysql_data:/var/lib/mysql
    - restart_policy: always
```

### Pillar for Secrets

```yaml
# /srv/pillar/mysql.sls
mysql:
  root_password: secure_root_password
  database: appdb
  user: appuser
  password: secure_user_password
```

### Top File

```yaml
# /srv/salt/top.sls
base:
  '*':
    - docker
  
  'web*':
    - nginx
  
  'db*':
    - mysql
```

## Orchestration

```yaml
# /srv/salt/orch/deploy.sls
# Run with: salt-run state.orchestrate orch.deploy

deploy_web:
  salt.state:
    - tgt: 'web*'
    - sls:
      - nginx

deploy_db:
  salt.state:
    - tgt: 'db*'
    - sls:
      - mysql
    - require:
      - salt: deploy_web

health_check:
  salt.function:
    - tgt: 'web*'
    - name: cmd.run
    - arg:
      - 'curl -f http://localhost/health || exit 1'
    - require:
      - salt: deploy_web
```

## Useful Commands

```bash
# Apply states
salt '*' state.apply docker
salt 'web*' state.apply nginx
salt '*' state.highstate

# Docker operations
salt '*' docker.pull quay.io/myorg/myapp:latest
salt '*' docker.restart mycontainer
salt '*' docker.logs mycontainer tail=50

# Bulk operations
salt '*' docker.prune containers=True images=True

# Execute in container
salt '*' docker.exec mycontainer cmd='cat /etc/os-release'

# Orchestration
salt-run state.orchestrate orch.deploy
```

## Salt vs Other Tools

| Feature | Salt | Ansible | Puppet |
|---------|------|---------|--------|
| Architecture | Master-Minion | Agentless (SSH) | Master-Agent |
| Language | YAML/Jinja | YAML/Jinja | Ruby DSL |
| Speed | Very Fast (ZeroMQ) | Slower (SSH) | Medium |
| Container Support | docker module | docker modules | docker module |
| Learning Curve | Medium | Easy | Steep |

---

*Part of the [Containers Documentation](README.md)*

