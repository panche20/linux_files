# Complete Linux Course for DevOps — Command Reference Guide

A comprehensive command reference and cheat sheet derived from the **Complete Linux Course for DevOps** masterclass. This guide covers fundamental CLI utilities, file operations, permissions, user/group management, package installation, process monitoring, systemd unit files, networking, Caddy reverse proxy setup, PM2 process management, and CI/CD pipelines.

---

## Table of Contents
1. [System Identification & Navigation](#1-system-identification--navigation)
2. [File System Operations & Manipulation](#2-file-system-operations--manipulation)
3. [Users, Groups & Security Permissions](#3-users-groups--security-permissions)
4. [Package Management (APT, NVM, PNPM)](#4-package-management-apt-nvm-pnpm)
5. [Process Management & Job Control](#5-process-management--job-control)
6. [Systemd Service Management & Logs](#6-systemd-service-management--logs)
7. [Networking, Web Servers & Reverse Proxies](#7-networking-web-servers--reverse-proxies)
8. [DevOps Utilities (PM2 & Git)](#8-devops-utilities-pm2--git)
9. [Key File Paths Reference](#9-key-file-paths-reference)

---

## 1. System Identification & Navigation

| Command | Syntax / Options | Description & DevOps Context |
| :--- | :--- | :--- |
| `whoami` | `whoami` | Prints the effective username of the currently logged-in user in the active shell. |
| `pwd` | `pwd` | Displays the absolute path of the Present Working Directory. |
| `uptime` | `uptime` | Shows server running time, active user count, and system load averages over 1, 5, and 15 minutes. |
| `hostnamectl` | `hostnamectl` | Displays OS distribution, kernel version, architecture, and machine hardware details. |
| `uname` | `uname -r`<br>`uname -a` | `-r` prints kernel release version; `-a` displays all system metadata. |
| `lsb_release` | `lsb_release -a` | Prints Linux Standard Base (LSB) distribution release details (e.g., Ubuntu 24.04 LTS). |
| `printenv` | `printenv`<br>`echo $VAR` | Lists all active environment variables or echoes specific variables like `echo $PWD` or `echo $SHELL`. |
| `exit` | `exit` | Closes the current terminal shell session or SSH login connection. |

---

## 2. File System Operations & Manipulation

| Command | Syntax / Options | Description & DevOps Context |
| :--- | :--- | :--- |
| `ls` | `ls`<br>`ls -a`<br>`ls -la`<br>`ls -lh` | `-a` lists all files including hidden (`.` dotfiles).<br>`-l` enables detailed long-listing mode.<br>`-h` converts file sizes to human-readable units (KB, MB, GB). |
| `cd` | `cd <dir>`<br>`cd ..`<br>`cd ~`<br>`cd -` | `..` moves up one directory level.<br>`~` jumps directly to user's home directory.<br>`-` instantly toggles back to the previous directory path. |
| `mkdir` | `mkdir <dir>`<br>`mkdir -p path/to/dir` | `-p` recursively creates parent directories if they do not already exist. |
| `touch` | `touch <filename>` | Creates new empty file or updates access/modification timestamps of existing files. |
| `cat` | `cat <filename>` | Prints entire file contents to standard output (`stdout`). |
| `echo` | `echo "text" > file`<br>`echo "text" >> file` | `>` overwrites target file with output.<br>`>>` appends new line output to existing file content without overwriting. |
| `cp` | `cp <src> <dest>`<br>`cp -r <dir_src> <dir_dest>` | Copies files or directories; `-r` copies directories recursively. Wildcards supported (`cp *.txt backup/`). |
| `mv` | `mv <src> <dest>` | Moves files/directories to new location or renames them in place. |
| `rm` | `rm <file>`<br>`rm -r <dir>`<br>`rm -rf <dir>` | `-r` deletes directory tree recursively.<br>`-f` forces deletion without confirmation prompts. |
| `less` | `less <filename>` | Opens file in an interactive pagination viewer (press `q` to exit, scroll line-by-line). |
| `tree` | `tree -L <depth>` | Displays visual directory tree hierarchy up to the specified level depth. |
| `which` | `which <executable>` | Finds and prints the absolute path of executable binaries in system `$PATH`. |

---

## 3. Users, Groups & Security Permissions

| Command | Syntax / Options | Description & DevOps Context |
| :--- | :--- | :--- |
| `who` | `who` | Shows list of all currently logged-in users and their login terminals. |
| `last` | `last` | Displays historical record of user logins, logouts, system reboots, and session durations. |
| `sudo` | `sudo <command>`<br>`sudo !!` | Executes command with root privileges. `sudo !!` re-runs the previous command prefixed with `sudo`. |
| `useradd` | `sudo useradd <username>` | Low-level utility to add a user account. |
| `adduser` | `sudo adduser <username>` | Interactive high-level script to create user, home directory, password, and shell profile. |
| `groupadd` | `sudo groupadd <groupname>` | Creates a new user group on the system. |
| `usermod` | `sudo usermod -aG <group> <user>` | `-aG` appends user to a supplementary group without removing existing group memberships. |
| `chmod` | `chmod 755 <file>`<br>`chmod +x <file>`<br>`chmod u-w <file>` | Modifies file access permissions using octal notation (`755`, `700`, `644`) or symbolic notation (`u`, `g`, `o`, `+`, `-`, `r`, `w`, `x`). |
| `chown` | `sudo chown <user>:<group> <path>`<br>`sudo chown -R <user>:<group> <path>` | Changes owner and group assignment; `-R` applies recursively across all nested files and subdirectories. |
| `grep` | `grep "pattern" <file>`<br>`grep -i "pattern" <file>` | Searches for matching text strings in files or standard output; `-i` ignores case sensitivity. |
| `ssh-keygen` | `ssh-keygen -t ed25519 -C "email"` | Generates public/private SSH key pairs for secure remote server access and Git deployments. |
| `vim` | `vim <file>` | Terminal text editor (`i` for insert mode, `Esc` to return to normal mode, `:wq` to write and quit, `:q!` to quit without saving). |

---

## 4. Package Management (APT, NVM, PNPM)

| Command | Syntax / Options | Description & DevOps Context |
| :--- | :--- | :--- |
| `apt update` | `sudo apt update` | Refreshes local repository package index catalogs against remote mirrors. |
| `apt upgrade` | `sudo apt upgrade` | Upgrades all installed system packages to their latest available versions. |
| `apt list` | `apt list --upgradable` | Lists all packages that have newer versions available in repository catalogs. |
| `apt install` | `sudo apt install <package>` | Installs specified package along with all necessary dependencies. |
| `apt remove` | `sudo apt remove <package>` | Uninstalls package binary while keeping configuration files intact. |
| `apt autoremove` | `sudo apt autoremove` | Cleans up orphaned dependencies no longer required by any installed software. |
| `nvm` | `nvm install --lts`<br>`nvm use <version>`<br>`nvm ls-remote` | Node Version Manager tool to manage multiple active Node.js runtimes. |
| `pnpm` | `pnpm install`<br>`pnpm build`<br>`pnpm start` | Fast, disk-space efficient Node.js package manager. |

---

## 5. Process Management & Job Control

| Command | Syntax / Options | Description & DevOps Context |
| :--- | :--- | :--- |
| `ps` | `ps`<br>`ps aux`<br>`ps aux \| grep <name>` | `aux` lists all processes across all users with CPU/memory metrics and control TTYs. |
| `top` | `top` | Real-time interactive process viewer displaying system resource usage and active PID list. |
| `htop` | `htop` | Enhanced interactive process manager (`F4` to filter, `F6` to sort by memory/CPU, `F9` to kill). |
| `&` | `<command> &` | Appending `&` runs command directly in the background, freeing up the active terminal. |
| `Ctrl + Z` | `Ctrl + Z` | Suspends (pauses) the currently running foreground process. |
| `bg` | `bg` | Resumes a suspended foreground process in the background. |
| `jobs` | `jobs` | Displays list of background jobs managed within the current terminal session. |
| `kill` | `kill <PID>`<br>`kill -15 <PID>`<br>`kill -9 <PID>` | Sends process termination signals (`-15` SIGTERM for graceful exit; `-9` SIGKILL for forced immediate termination). |
| `pkill` | `pkill -u <user> <process_name>` | Terminates active processes matching specified process name or owner. |
| `nohup` | `nohup <command> &` | Runs command immune to hangups (`SIGHUP`), continuing execution after SSH logout. Output logs to `nohup.out`. |

---

## 6. Systemd Service Management & Logs

| Command | Syntax / Options | Description & DevOps Context |
| :--- | :--- | :--- |
| `systemctl daemon-reload` | `sudo systemctl daemon-reload` | Reloads systemd manager configuration after creating or modifying `.service` unit files. |
| `systemctl start` | `sudo systemctl start <service>` | Starts an inactive systemd service immediately. |
| `systemctl stop` | `sudo systemctl stop <service>` | Stops an active systemd service. |
| `systemctl restart` | `sudo systemctl restart <service>` | Restarts a running systemd service. |
| `systemctl status` | `systemctl status <service>` | Displays operational status, PID, uptime, memory, and recent stdout/stderr logs. |
| `systemctl enable` | `sudo systemctl enable <service>` | Configures service to start automatically upon system boot. |
| `systemctl enable --now` | `sudo systemctl enable --now <service>` | Enables service for auto-boot AND starts it immediately. |
| `systemctl disable` | `sudo systemctl disable <service>` | Disables service from launching automatically during system boot. |
| `journalctl` | `journalctl`<br>`journalctl -u <service> -n 50`<br>`journalctl -u <service> -f` | Queries systemd journal logs. `-u` filters by service unit, `-n` specifies line count, `-f` streams logs in real-time. |
| `watch` | `watch -n 2 <command>` | Re-executes command periodically at specified intervals (e.g. `watch -n 2 free -h`). |
| `free` | `free -h` | Displays total, used, and available RAM and swap memory space. |

### Sample Systemd Service Unit File (`/etc/systemd/system/backend.service`)
```ini
[Unit]
Description=ChaiLearn DevOps API Service
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=yntp
Group=devops
WorkingDirectory=/opt/chailearn/backend
Environment=NODE_ENV=production
EnvironmentFile=/opt/chailearn/backend/.env.production
ExecStart=/usr/bin/node src/index.js
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

---

## 7. Networking, Web Servers & Reverse Proxies

| Command | Syntax / Options | Description & DevOps Context |
| :--- | :--- | :--- |
| `curl` | `curl <url>`<br>`curl -I <url>`<br>`curl wttr.in/Mumbai` | Command-line HTTP transfer tool; `-I` fetches HTTP response headers only. |
| `dig` | `dig <domain>` | Performs DNS lookup queries to verify domain IP resolution (`A` records). |
| `caddy` | `caddy validate --config /etc/caddy/Caddyfile`<br>`sudo systemctl reload caddy` | Modern reverse proxy web server featuring automatic HTTPS/TLS certificate provision. |

### Sample Caddyfile Configuration (`/etc/caddy/Caddyfile`)
```caddy
linux.yntp.me {
    handle /api/* {
        reverse_proxy localhost:3000
    }
    handle {
        reverse_proxy localhost:3001
    }
}
```

---

## 8. DevOps Utilities (PM2 & Git)

| Command | Syntax / Options | Description & DevOps Context |
| :--- | :--- | :--- |
| `pm2 start` | `pm2 start ecosystem.config.js` | Starts Node.js applications defined inside PM2 process configuration files. |
| `pm2 list` | `pm2 list` | Displays real-time status table of all applications managed by PM2. |
| `pm2 logs` | `pm2 logs <app_name>`<br>`pm2 logs --lines 100` | Streams stdout and stderr logs for PM2-managed processes. |
| `pm2 restart` | `pm2 restart <app_name>` | Restarts target PM2 process without modifying configurations. |
| `pm2 save` | `pm2 save` | Freezes current process list state to restore automatically across server reboots. |
| `pm2 startup` | `pm2 startup` | Generates systemd startup integration script to auto-revive PM2 on reboot. |
| `git clone` | `git clone <repo_url>` | Downloads remote repository clone to local filesystem. |
| `git remote` | `git remote -v`<br>`git remote set-url origin <url>` | Views or modifies remote Git repository URLs. |
| `git commit` | `git commit --amend` | Modifies the most recent commit message or includes staged changes into previous commit. |

### Sample PM2 Ecosystem Config (`ecosystem.config.js`)
```javascript
module.exports = {
  apps: [
    {
      name: "chailearn-backend",
      script: "src/index.js",
      cwd: "/opt/chailearn/backend",
      interpreter: "node",
      env: {
        NODE_ENV: "production",
        PORT: 3000
      }
    },
    {
      name: "chailearn-frontend",
      script: ".output/server/index.mjs",
      cwd: "/opt/chailearn/frontend",
      interpreter: "node",
      env: {
        NODE_ENV: "production",
        PORT: 3001
      }
    }
  ]
};
```

---

## 9. Key File Paths Reference

* `/root` — Root superuser home directory.
* `/home/<username>` — Standard user home directories.
* `/etc` — System-wide configuration files (e.g. `/etc/passwd`, `/etc/group`, `/etc/caddy/Caddyfile`, `/etc/systemd/system/`).
* `/var/log` — System and application log directories.
* `/tmp` — Temporary storage files (auto-cleared on reboot).
* `/opt` — Optional third-party application software installs (standard location for custom application code like `/opt/chailearn`).
* `/bin` & `/usr/bin` — Executable binaries for standard user commands.
* `/sbin` & `/usr/sbin` — Executable binaries reserved for system administration.
* `~/.ssh/authorized_keys` — Contains public SSH key fingerprints authorized to log in as the target user.
