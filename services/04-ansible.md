# Ansible Basics

> **What it is:** Ansible is an agentless IT automation tool that uses SSH to configure systems, deploy software, and orchestrate tasks. It uses YAML for configuration (playbooks) and is known for being simple to learn and use.

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         Ansible Architecture                                 │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────┐       │
│   │                     CONTROL NODE                                 │       │
│   │  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────┐    │       │
│   │  │ Inventory │  │ Playbooks │  │   Roles   │  │  Modules  │    │       │
│   │  │ (hosts)   │  │  (.yml)   │  │           │  │           │    │       │
│   │  └───────────┘  └───────────┘  └───────────┘  └───────────┘    │       │
│   └────────────────────────┬────────────────────────────────────────┘       │
│                            │ SSH (no agent needed)                           │
│              ┌─────────────┼─────────────┐                                  │
│              ▼             ▼             ▼                                  │
│   ┌──────────────┐ ┌──────────────┐ ┌──────────────┐                       │
│   │ MANAGED NODE │ │ MANAGED NODE │ │ MANAGED NODE │                       │
│   │  (web1)      │ │  (web2)      │ │  (db1)       │                       │
│   └──────────────┘ └──────────────┘ └──────────────┘                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## Installation

```bash
# RHEL/CentOS/Fedora
sudo dnf install ansible-core

# Ubuntu/Debian
sudo apt install ansible

# Verify
ansible --version
```

## Inventory

```ini
# /etc/ansible/hosts or inventory file

[webservers]
web1.example.com
web2.example.com

[dbservers]
db1.example.com

[production:children]
webservers
dbservers

[webservers:vars]
ansible_user=deploy
http_port=80
```

## Ad-Hoc Commands

```bash
# Ping all hosts
ansible all -m ping

# Run command
ansible all -a "uptime"
ansible all -m shell -a "df -h | grep sda"

# Install package (with become/sudo)
ansible webservers -m dnf -a "name=nginx state=present" -b

# Copy file
ansible webservers -m copy -a "src=file.txt dest=/tmp/"

# Start service
ansible webservers -m service -a "name=nginx state=started" -b

# Gather facts
ansible web1 -m setup
```

## Playbooks

### Basic Structure

```yaml
# site.yml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  vars:
    http_port: 80
  
  tasks:
    - name: Install nginx
      dnf:
        name: nginx
        state: present
    
    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
    
    - name: Deploy config
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx
  
  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

### Running Playbooks

```bash
ansible-playbook site.yml
ansible-playbook site.yml -i inventory.yml
ansible-playbook site.yml --check --diff    # Dry run
ansible-playbook site.yml -e "http_port=8080"
ansible-playbook site.yml --limit web1
ansible-playbook site.yml -vvv              # Verbose
```

## Common Modules

### Package Management

```yaml
# DNF (RHEL/Fedora)
- name: Install packages
  dnf:
    name:
      - nginx
      - vim
    state: present

# APT (Ubuntu/Debian)
- name: Install packages
  apt:
    name: nginx
    state: present
    update_cache: yes
```

### File Management

```yaml
- name: Copy file
  copy:
    src: config.txt
    dest: /etc/myapp/config.txt
    owner: root
    mode: '0644'

- name: Create from template
  template:
    src: app.conf.j2
    dest: /etc/app/app.conf

- name: Create directory
  file:
    path: /opt/myapp
    state: directory
    mode: '0755'
```

### Service Management

```yaml
- name: Start and enable service
  service:
    name: nginx
    state: started
    enabled: yes
```

### User Management

```yaml
- name: Create user
  user:
    name: deploy
    shell: /bin/bash
    groups: wheel

- name: Add SSH key
  authorized_key:
    user: deploy
    key: "{{ lookup('file', 'deploy.pub') }}"
```

## Conditionals and Loops

```yaml
# Conditional
- name: Install on RHEL only
  dnf:
    name: nginx
  when: ansible_os_family == "RedHat"

# Loop
- name: Install packages
  dnf:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - vim
    - git

# Loop with dict
- name: Create users
  user:
    name: "{{ item.name }}"
    uid: "{{ item.uid }}"
  loop:
    - { name: 'alice', uid: 1001 }
    - { name: 'bob', uid: 1002 }
```

## Templates (Jinja2)

```jinja2
# nginx.conf.j2
server {
    listen {{ http_port }};
    server_name {{ server_name }};
    
{% if ssl_enabled %}
    ssl_certificate {{ ssl_cert }};
{% endif %}
}
```

## Roles

```
roles/nginx/
├── defaults/main.yml    # Default variables
├── handlers/main.yml    # Handlers
├── tasks/main.yml       # Tasks
├── templates/           # Jinja2 templates
└── vars/main.yml        # Variables
```

```yaml
# Using roles
- hosts: webservers
  roles:
    - nginx
    - { role: app, http_port: 8080 }
```

## Vault (Secrets)

```bash
ansible-vault create secrets.yml
ansible-vault edit secrets.yml
ansible-playbook site.yml --ask-vault-pass
```

## Quick Reference

```bash
# Ad-hoc
ansible all -m ping
ansible all -a "uptime"
ansible webservers -m dnf -a "name=nginx" -b

# Playbooks
ansible-playbook site.yml
ansible-playbook site.yml --check
ansible-playbook site.yml -e "var=value"

# Inventory
ansible-inventory --list
ansible-inventory --graph

# Vault
ansible-vault create secrets.yml
ansible-vault edit secrets.yml

# Galaxy
ansible-galaxy init roles/myrole
ansible-galaxy install geerlingguy.nginx
```

---

*Part of the [Services Documentation](README.md)*

