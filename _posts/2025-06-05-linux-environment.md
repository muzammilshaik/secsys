---
author: muju
title: "Linux Environment Variables"
description: "Comprehensive guide on Linux environment variables, from system-wide and user-specific configurations to practical security tips and hacking tricks for power users."
categories: ["Linux"]
tags: ["Linux", "Environment Variables", "SysAdmin", "Security"]
image:
  path: /assets/linux/env/env.webp
  lqip: data:image/webp;base64,UklGRuQAAABXRUJQVlA4INgAAADQBACdASoUABQAPpFAmEmlo6IhKAqosBIJagCxHzQ1WQAuwABTkPblb25mAffhAAD+9C9xYX09zTI5Lw1Y3FNtils+VI+1oFMpjia9jOqP/s6//+1dNA19fbL2ff4vSCckNizDcHtHbwtyCysiw/b+u4SbvEwK12PrL6f4HT0tW/wuKGx1RvwAXCbUDkp2nenHwRzZd7e8VlUTq26QZGSBHS7wvjR6HUJUYSRKqPKcL1nf42DxqDVrdtX8IwrkvYJ+UVzl3+T13HBVLu++5SU8TFf+VuBgAAA=
published: true
hidden: false
toc: true
# https://medium.verylazytech.com/15-linux-environment-variables-hackers-use-you-should-too-f4b9397098dd#bypass
---

# Introduction to Linux Environment Variables

In a **Linux operating system**, **environment variables** are dynamic values that define the behavior of system processes and applications. These variables store configuration data, such as the system path, user preferences, and settings, making them essential for efficient system operations and automation.

## Environment Variables

### System-Wide Variables

These are available for all users and are set by the system administrator. They are defined in files such as:

- `/etc/environment`
- `/etc/profile`
- `/etc/bash.bashrc`

### User-Specific Variables

These are defined per user and stored in:

- `~/.bashrc`
- `~/.profile`
- `~/.bash_profile`

### Shell Variables

Shell variables exist only within the running shell session. They can be created and modified within the terminal.

## Commonly Used Linux Variables

### File & Directory
**PATH** The `PATH` variable defines directories where the system searches for executable files.

```bash
# PATH - Executable lookup directories
echo $PATH
export PATH=/usr/local/bin:$PATH

# HOME - User's home directory
echo $HOME

# PWD - Present working directory
echo $PWD
```

### User Information

```bash
# USER - Current username
echo $USER

# HOSTNAME - System hostname
echo $HOSTNAME
```

### Shell Settings
```bash
# SHELL - Default shell
echo $SHELL

# PS1 - Bash prompt string
echo $PS1
export PS1="[\u@\h \W]\$ "
```

### Editors and Language
```bash
# EDITOR - Default text editor
export EDITOR=nano

# LANG - System language
export LANG=en_US.UTF-8
```

### GUI and Display
```bash
# DISPLAY - X Window System display
echo $DISPLAY
export DISPLAY=:0.0
```

### History
```bash
# HISTFILESIZE - Max history file size
echo $HISTFILESIZE
export HISTFILESIZE=5000

# HISTSIZE - Commands per session
echo $HISTSIZE
export HISTSIZE=1000
```

### Mail
```bash
# MAIL - Mail spool location
echo $MAIL
```

### Documentation
```bash
# MANPATH - Manual search path
echo $MANPATH
export MANPATH=/usr/local/share/man:$MANPATH
```

### Terminal 
```bash
# TERM - Terminal type
echo $TERM
export TERM=xterm-256color
```

### Time
```bash
# TZ - Timezone
export TZ=Asia/Kolkata
```

### OS Info
```bash
# OSTYPE - OS type
echo $OSTYPE
```

## Viewing Environment Variables

### Using `printenv`
```bash
printenv
printenv PATH
```

### Using `env`
```bash
env
```

### Using set
```bash
set | less
```

## Setting Environment Variables

### Temporarily (Current Session Only)
```bash
export MY_VAR="Hello World"
echo $MY_VAR
```

### Permanently (Across Sessions)

Add the variable to `~/.bashrc` or `~/.profile`:
```bash
echo 'export MY_VAR="Hello World"' >> ~/.bashrc
source ~/.bashrc
```

### Unsetting Variables
```bash
unset MY_VAR
echo $MY_VAR  # No output expected
```

## Using Variables in Scripts

### Example Bash Script

```bash
#!/bin/bash
echo "Current user: $USER"
echo "Home directory: $HOME"
```

Save as `script.sh` and run:

```bash
chmod +x script.sh
./script.sh
```

## Security Tips for Env Variables

1. Avoid storing sensitive data (e.g., passwords) in environment variables.
2. Use readonly for critical variables:
    ```bash
    readonly SECURE_VAR="Secret"
    ```
3. Restrict permissions:
    ```bash
    chmod 600 ~/.bashrc
    ```
## Advanced and Offensive Usage

### History Deletion
```bash
export HISTFILESIZE=0
export HISTSIZE=0
```

### Proxy Redirection
```bash
export http_proxy="http://10.10.10.10:8080"
export https_proxy="http://10.10.10.10:8080"
```

### SSL Certificate Trust
```bash
export SSL_CERT_FILE=/path/to/ca-bundle.pem
export SSL_CERT_DIR=/path/to/ca-certificates
```

### Library Injection
```bash
export LD_PRELOAD=/tmp/malicious.so
export LD_LIBRARY_PATH=/tmp/mylib:$LD_LIBRARY_PATH
```

### PATH Hijacking
```bash
export PATH=/tmp/malicious:$PATH
```

### Auto-Logout Timeout
```bash
export TMOUT=1
```

### App Config Hijack
```bash
export XDG_CONFIG_HOME=/tmp/custom-config
```

### Field Separator Manipulation
```bash
export IFS=$'\n'
```

### PS1 Prompt Manipulation
```bash
export PS1='[\u@\h \W]# '
```

### HOME Override
```bash
export HOME=/tmp/fakehome
```

### MAIL Redirection
```bash
export MAIL=/tmp/mail
```

### Sudo Askpass Exploit
```bash
export SUDO_ASKPASS=/tmp/fake-pass-prompt
sudo -A whoami
```

### GDB Exploitation
```bash
export GDBINIT=/tmp/malicious-gdbinit
```

## Summary

Environment variables are a fundamental aspect of Linux systems. They control how the system and user sessions behave, help in scripting, system management, and even (mis)use in penetration testing. Managing them wisely leads to a more secure and efficient Linux environment.