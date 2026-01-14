# Shell Configuration and Aliases

> **What it is:** Bash uses several configuration files that run at different times. Understanding when each file runs helps you configure your environment properly.

## Configuration Files Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    Shell Configuration Files                                 │
│                                                                              │
│  LOGIN SHELL (ssh, console login, su -)                                     │
│  ┌─────────────────────────────────────────────────────┐                    │
│  │  /etc/profile          System-wide login settings    │                    │
│  │       ↓                                               │                    │
│  │  /etc/profile.d/*.sh   Additional system scripts     │                    │
│  │       ↓                                               │                    │
│  │  ~/.bash_profile       User login settings           │                    │
│  │       ↓  (or ~/.profile if bash_profile missing)     │                    │
│  │  ~/.bashrc             Usually sourced by above      │                    │
│  └─────────────────────────────────────────────────────┘                    │
│                                                                              │
│  NON-LOGIN SHELL (opening new terminal in GUI, scripts)                     │
│  ┌─────────────────────────────────────────────────────┐                    │
│  │  /etc/bash.bashrc      System-wide settings (Ubuntu) │                    │
│  │  /etc/bashrc           System-wide settings (RHEL)   │                    │
│  │       ↓                                               │                    │
│  │  ~/.bashrc             User settings                  │                    │
│  └─────────────────────────────────────────────────────┘                    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## When Each File Runs

| File | Login Shell | Non-Login Shell | Use For |
|------|-------------|-----------------|---------|
| `/etc/profile` | ✅ Yes | ❌ No | System-wide env vars, PATH |
| `~/.bash_profile` | ✅ Yes | ❌ No | User login settings |
| `~/.bashrc` | Via .bash_profile | ✅ Yes | Aliases, functions, prompt |
| `/etc/bashrc` | Via .bashrc | ✅ Yes | System-wide bash settings |

## Login vs Non-Login Shell

```bash
# Login shell - runs .bash_profile first:
ssh user@server                  # SSH connection
su - username                    # Switch user with dash
login                            # Console login

# Non-login shell - runs .bashrc only:
bash                             # Open new bash
gnome-terminal                   # GUI terminal
su username                      # Switch user without dash
./script.sh                      # Running scripts
```

## ~/.bash_profile

```bash
# ~/.bash_profile - Runs once at login

# Set environment variables
export EDITOR=vim
export PATH="$HOME/bin:$HOME/.local/bin:$PATH"

# Source .bashrc for aliases and functions
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
```

## ~/.bashrc

```bash
# ~/.bashrc - Runs for every new shell

# If not running interactively, don't do anything
[[ $- != *i* ]] && return

# History settings
HISTSIZE=10000
HISTFILESIZE=20000
HISTCONTROL=ignoreboth:erasedups

# Shell options
shopt -s histappend
shopt -s checkwinsize
shopt -s cdspell

# Prompt
PS1='\[\e[32m\]\u@\h\[\e[0m\]:\[\e[34m\]\w\[\e[0m\]\$ '

# Aliases
alias ll='ls -la'
alias reload='source ~/.bashrc'
```

---

## Aliases

### Creating Aliases

```bash
# Temporary alias (current session only)
alias ll='ls -la'

# List all aliases
alias

# Remove alias
unalias ll
```

### Common Aliases (add to ~/.bashrc)

```bash
# Navigation
alias ..='cd ..'
alias ...='cd ../..'

# List files
alias ls='ls --color=auto'
alias ll='ls -alF'
alias la='ls -A'
alias lt='ls -ltr'

# Safety
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# System
alias df='df -h'
alias free='free -h'
alias ports='ss -tulanp'

# Git
alias gs='git status'
alias ga='git add'
alias gc='git commit -m'
alias gp='git push'
alias gl='git pull'

# Kubernetes
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'

# Systemctl
alias ss='sudo systemctl status'
alias sr='sudo systemctl restart'
```

### Functions

```bash
# Create directory and cd into it
mkcd() { mkdir -p "$1" && cd "$1"; }

# Extract any archive
extract() {
    case "$1" in
        *.tar.gz)  tar xzf "$1" ;;
        *.tar.bz2) tar xjf "$1" ;;
        *.zip)     unzip "$1" ;;
        *.gz)      gunzip "$1" ;;
    esac
}

# Quick backup
backup() {
    cp "$1" "$1.bak.$(date +%Y%m%d)"
}
```

### Apply Changes

```bash
source ~/.bashrc
# or
. ~/.bashrc
```

---

*Part of the [Linux Documentation](README.md)*

