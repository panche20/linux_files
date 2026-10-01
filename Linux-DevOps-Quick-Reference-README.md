# Linux & DevOps Quick Reference

> A practical Linux reference for DevOps, SRE, Kubernetes, Docker, and production troubleshooting.

This README reorganizes the material from the original **Linux commands for quick reference** notes into a GitHub-friendly reference.

---

## Table of Contents

- [1. Linux Filesystem and Kernel Interfaces](#1-linux-filesystem-and-kernel-interfaces)
- [2. `/etc` and Linux Configuration](#2-etc-and-linux-configuration)
- [3. Users, UID/GID and Authentication](#3-users-uidgid-and-authentication)
- [4. systemd and Services](#4-systemd-and-services)
- [5. Processes, PID and PPID](#5-processes-pid-and-ppid)
- [6. Signals and Process Termination](#6-signals-and-process-termination)
- [7. Linux Permissions and Ownership](#7-linux-permissions-and-ownership)
- [8. Linux Networking and Network Namespaces](#8-linux-networking-and-network-namespaces)
- [9. cgroups and Kubernetes Resource Limits](#9-cgroups-and-kubernetes-resource-limits)
- [10. Memory, Swap and OOM Killer](#10-memory-swap-and-oom-killer)
- [11. CPU Limits and Throttling](#11-cpu-limits-and-throttling)
- [12. System Performance Troubleshooting](#12-system-performance-troubleshooting)
- [13. Ports, Sockets and `lsof` / `ss`](#13-ports-sockets-and-lsof--ss)
- [14. Linux Logging and Journald](#14-linux-logging-and-journald)
- [15. Package Management with APT](#15-package-management-with-apt)
- [16. Linux DNS Resolution](#16-linux-dns-resolution)
- [17. Storage and Filesystems](#17-storage-and-filesystems)
- [18. File/Text Processing](#18-filetext-processing)
- [19. Finding Files](#19-finding-files)
- [20. SSH, SCP and Sudo](#20-ssh-scp-and-sudo)
- [21. Kubernetes Connections](#21-kubernetes-connections)
- [22. Linux Quick Revision Cheat Sheet](#22-linux-quick-revision-cheat-sheet)
- [23. DevOps Interview Questions](#23-devops-interview-questions)

---

# 1. Linux Filesystem and Kernel Interfaces

Linux contains several special filesystem locations that are interfaces to the kernel rather than ordinary directories.

| Path | Purpose |
|---|---|
| `/dev` | Device interfaces |
| `/proc` | Live process/kernel information |
| `/sys` | Kernel/device information and controls |
| `/dev/sda` | Disk device |
| `/dev/tty` | Terminal device |
| `/dev/null` | Discards data written to it |

### `/proc`

`/proc` provides a live view into kernel and process information.

Examples:

```bash
cat /proc/self/cgroup
cat /proc/self/status
cat /proc/1/status
cat /proc/meminfo
cat /proc/1/oom_score
```

The important mental model is:

```text
Application
    |
    v
Linux process
    |
    v
Linux kernel
    |
    +-- /proc
    +-- /sys
    +-- /dev
```

---

# 2. `/etc` and Linux Configuration

`/etc` contains important system configuration.

Common files:

```bash
/etc/passwd
/etc/shadow
/etc/hosts
/etc/resolv.conf
/etc/nsswitch.conf
/etc/services
/etc/ssh/sshd_config
```

The notes describe `/etc` as the "brain" of Linux because many system-wide configuration files live there.

---

# 3. Users, UID/GID and Authentication

## `/etc/passwd`

```bash
cat /etc/passwd
```

Example:

```text
root:x:0:0:root:/root:/bin/bash
```

Fields:

```text
username : password-field : UID : GID : description : home : shell
```

For the example:

- `root` → username
- `x` → password information is stored separately
- `0` → UID
- `0` → GID
- `root` → description
- `/root` → home directory
- `/bin/bash` → login shell

A UID of `0` represents the superuser account.

## `/etc/shadow`

```bash
cat /etc/shadow
```

The notes show password information in `/etc/shadow` as a hash rather than the original plaintext password.

---

# 4. systemd and Services

`systemd` is the service manager used by modern Linux systems.

At boot:

```text
Linux Kernel
     |
     v
  systemd
     |
     +-- ssh
     +-- networking
     +-- cron
     +-- docker
     +-- containerd
     +-- kubelet
     +-- nginx
```

Check running services:

```bash
sudo systemctl list-units --type=service | grep running
```

Useful commands:

```bash
systemctl status nginx
sudo systemctl restart nginx
sudo systemctl stop nginx
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl disable nginx
```

For Kubernetes nodes, systemd commonly manages:

```text
kubelet.service
containerd.service
docker.service
```

---

# 5. Processes, PID and PPID

A **process** is a running program in memory.

Examples:

- nginx
- ssh server
- Docker
- kubelet
- your shell
- `ps` itself

Even Bash is a process.

List processes:

```bash
ps -ef
```

Inspect PID 1:

```bash
ps -p 1 -o pid,ppid,cmd
```

### PID

PID = Process ID.

Every running process receives a PID.

### PPID

PPID = Parent Process ID.

Linux processes form a process tree.

Typical server:

```text
systemd (PID 1)
├── sshd
│   └── shell
│       └── ps
├── docker
├── containerd
└── nginx
```

Kubernetes node:

```text
systemd
└── kubelet
    └── container runtime
        └── pod container
```

---

# 6. Signals and Process Termination

Linux controls processes using signals.

| Signal | Meaning |
|---|---|
| `SIGTERM` | Request graceful shutdown |
| `SIGKILL` | Force immediate termination |
| `SIGHUP` | Reload/re-read configuration |
| `SIGSTOP` | Pause process |

Examples:

```bash
kill PID
kill -9 PID
```

The notes connect this directly to containers.

### Docker stop

Conceptually:

```text
docker stop
    |
    +-- SIGTERM
    |
    +-- wait
    |
    +-- SIGKILL if necessary
```

### Kubernetes Pod deletion

Conceptually:

```text
kubectl delete pod
        |
      kubelet
        |
  container runtime
        |
      SIGTERM
        |
      wait
        |
      SIGKILL
```

This is an important production concept: **container termination ultimately depends on Linux process behavior.**

---

# 7. Linux Permissions and Ownership

Every file has an owner and a group.

Important commands:

```bash
chmod
chown
chgrp
```

Useful mental model:

```text
Owner  -> house owner
Group  -> family members
Others -> strangers
```

- `chown` → change ownership
- `chgrp` → change group
- `chmod` → change permissions

Example:

```bash
chgrp appgroup /data/secret.txt
```

## Why this matters in Docker/Kubernetes

A container may run as:

```dockerfile
USER 1001
```

while files inside the image are owned by `root`.

The application may then fail to:

- write logs
- create temporary files
- update cache
- write application data

A common image-level fix is:

```dockerfile
RUN chown -R 1001:1001 /app
```

### Kubernetes `securityContext`

The notes connect Linux ownership to:

```yaml
securityContext:
  runAsUser: 1000
  fsGroup: 2000
  readOnlyRootFilesystem: true
```

The key production lesson is:

> A container running as a non-root UID can fail with `Permission denied` when the UID/GID does not match the ownership or permissions of a mounted volume.

### Avoid `chmod 777`

For example:

```bash
chmod -R 777 /var/www/html
```

gives broad permissions and can allow unintended users/processes to modify website files.

---

# 8. Linux Networking and Network Namespaces

Check interfaces:

```bash
ip addr
```

### Loopback

```text
lo -> 127.0.0.1
```

Traffic to localhost stays within the machine.

### Container networking

A container can have:

- its own network interface
- its own IP
- its own routing table

Linux achieves this using **network namespaces**.

The notes describe a container as a normal Linux process placed into isolated namespaces.

Core Linux isolation mechanisms discussed:

```text
PID namespace
Mount namespace
Network namespace
User namespace
cgroups
```

The important idea:

```text
Container
   |
   +-- isolated process view
   +-- isolated filesystem view
   +-- isolated network view
   |
   +-- shares the host Linux kernel
```

There is no hypervisor involved in ordinary container isolation.

---

# 9. cgroups and Kubernetes Resource Limits

**cgroups (Control Groups)** provide resource control for processes.

Without resource limits, one badly behaved application could consume the machine's resources:

```text
Application
    |
    +-- consumes RAM
    +-- consumes CPU
    |
    v
Other applications affected
    |
    v
System instability
```

cgroups allow Linux to apply limits such as:

- memory
- CPU
- disk I/O
- process count

A container is essentially a group of processes associated with resource controls.

## Inspect cgroup membership

```bash
cat /proc/self/cgroup
```

Example from the notes:

```text
0::/user.slice/user-1000.slice/session-1.scope
```

For cgroup v2, the notes explain:

```text
0
::
/user.slice/user-1000.slice/session-1.scope
```

The path identifies the process's location in the systemd/cgroup hierarchy.

## Inspect cgroup controls

```bash
cd /sys/fs/cgroup/user.slice/user-1000.slice/session-1.scope/
ls
```

Current memory usage:

```bash
echo "$(( $(cat memory.current) / 1024 / 1024 )) MB"
```

## Temporary resource limits

The notes demonstrate `systemd-run`:

```bash
systemd-run --user --scope \
  -p MemoryMax=100M \
  -p CPUQuota=20% \
  ./heavy_script.sh
```

Monitor cgroups:

```bash
systemd-cgtop --user
```

## Persistent limits

Example user-slice configuration:

```bash
sudo systemctl set-property user-1000.slice \
  MemoryMax=2G \
  CPUQuota=50%
```

The notes describe systemd creating a drop-in configuration under:

```text
/etc/systemd/system.control/user-1000.slice.d/
```

For a specific service:

```bash
systemctl edit my_app.service
```

Example:

```ini
[Service]
MemoryMax=500M
CPUQuota=10%
TasksMax=50
```

Then:

```bash
systemctl daemon-reload
```

---

# 10. Memory, Swap and OOM Killer

Check memory:

```bash
free -h
```

Kernel memory information:

```bash
cat /proc/meminfo | head
```

Important `free -h` fields:

| Field | Meaning |
|---|---|
| Total | RAM available to the system |
| Used | Memory actively used |
| Free | Completely unused memory |
| Buff/cache | Memory used for caching |
| Available | Memory that can be made available to applications |

### `Available` vs `Free`

Linux intentionally uses unused RAM for caching.

The important operational metric is often **Available**, not simply Free.

Conceptually:

```text
RAM
 |
 +-- Applications
 +-- Kernel
 +-- Buff/cache
 |
 +-- Available
```

Cache can be reclaimed when an application needs memory.

## Swap

Swap is disk space used as additional backing for memory pages.

The notes describe:

```text
Swap = disk used as backup RAM
```

Swap is much slower than physical RAM.

Check:

```bash
free -h
```

---

## OOM Killer

OOM = Out Of Memory.

When memory is exhausted, the Linux kernel can select processes to kill.

Check OOM score:

```bash
cat /proc/1/oom_score
```

The notes explain the basic model:

```text
Application consumes memory
        |
        v
cgroup memory limit exceeded
        |
        v
Linux OOM handling
        |
        v
Process killed
        |
        v
Container exits
        |
        v
Kubelet notices
        |
        v
Pod/container restarted
```

Kubernetes may report:

```text
Reason: OOMKilled
```

The important production distinction is:

> `OOMKilled` is fundamentally a Linux memory-management event that Kubernetes reports and reacts to.

Typical troubleshooting directions from the notes:

- increase the memory limit
- fix the application memory leak
- tune JVM heap
- adjust cache size

---

# 11. CPU Limits and Throttling

Memory and CPU limits behave differently.

### Memory

Exceeding a memory limit can result in a process being killed.

### CPU

Exceeding a CPU limit generally results in **CPU throttling**, not process termination.

Example:

```yaml
resources:
  limits:
    cpu: "500m"
```

The notes use:

```text
1 CPU = 100%
500m = 50%
```

Conceptually:

```text
CPU limit reached
       |
       v
Kernel scheduler throttles process
       |
       v
Application remains alive
       |
       v
Application becomes slow
       |
       v
Latency increases
```

This explains a common production symptom:

> Pods are running, but application response time becomes very high.

---

# 12. System Performance Troubleshooting

## `top`

When a production server becomes slow, the notes recommend starting with:

```bash
top
```

Important fields:

### Tasks

```text
total
running
sleeping
stopped
zombie
```

### CPU

Important fields:

- `us` → user-space CPU
- `sy` → kernel CPU
- `id` → idle CPU
- `wa` → I/O wait
- `st` → CPU stolen by the hypervisor

The notes emphasize that high `wa` can indicate disk I/O bottlenecks.

Useful keys inside `top`:

| Key | Action |
|---|---|
| `P` | Sort by CPU |
| `M` | Sort by memory |
| `1` | Show individual CPU cores |
| `k` | Kill a process |
| `q` | Quit |

## `vmstat`

```bash
vmstat 1 5
```

The notes recommend `vmstat` for deeper analysis of:

- CPU
- memory
- swap
- I/O

## `sar`

Examples:

```bash
sar -r 1 3
sar -n DEV 1 3
```

- `sar -r` → memory statistics
- `sar -n DEV` → network statistics

## Sort processes by memory

```bash
ps aux --sort=-%mem
```

### Troubleshooting mental model

| Symptom | Linux observation | Possible Kubernetes effect |
|---|---|---|
| High memory | Process memory is high | `OOMKilled` |
| High CPU | Process consumes CPU | Latency |
| High disk wait | `wa%` is high | Pod/application timeouts |
| Too many processes | PID/task limit | Container failure |

---

# 13. Ports, Sockets and `lsof` / `ss`

Linux treats network sockets as file descriptors.

Check which process owns a network socket:

```bash
lsof -i
```

Specific port:

```bash
lsof -i :8080
```

Example:

```bash
nc -l -p 8080
```

Then:

```bash
lsof -i :8080
```

This identifies:

- process
- PID
- port

## `ss`

Modern alternative:

```bash
ss -tulnp
```

Useful interpretation:

```text
*:8080 (LISTEN)
```

The notes also connect service names to:

```bash
cat /etc/services
```

Common ports:

| Port | Service |
|---:|---|
| `22` | SSH |
| `80` | HTTP |
| `443` | HTTPS |
| `8080` | HTTP alternate |

### Classic production problem

If Nginx fails to restart:

```bash
systemctl restart nginx
```

one possible cause is that another process is already using port 80.

Investigate:

```bash
sudo ss -lntp
sudo lsof -i :80
```

---

# 14. Linux Logging and Journald

Linux records important events such as:

- login attempts
- SSH failures
- disk errors
- service crashes
- network problems
- kernel events
- container-related events

Traditional log files are commonly under:

```text
/var/log
```

Important files mentioned in the notes:

```text
/var/log/syslog
/var/log/auth.log
/var/log/kern.log
/var/log/dmesg
/var/log/apt/
```

## Read logs safely

```bash
less /var/log/auth.log
```

Inside `less`:

| Key | Action |
|---|---|
| `Shift + G` | Go to bottom |
| `/word` | Search |
| `q` | Quit |

## Follow a log

```bash
tail -f /var/log/syslog
```

Example:

```bash
tail -f /var/log/dpkg.log
```

Then in another terminal:

```bash
sudo apt install -y tree
```

The installation activity can be observed in the log.

## Kernel messages

```bash
dmesg | tail
```

The notes associate `dmesg` with:

- memory problems
- disk failures
- device errors
- OOM Killer activity

A useful production command:

```bash
sudo dmesg | grep -i "oom\|kill\|fail"
```

Human-readable timestamps:

```bash
sudo dmesg -T
```

---

## systemd-journald

Modern Linux systems use journald as a primary logging mechanism.

Conceptual flow:

```text
Application
    |
    v
systemd-journald
    |
    v
rsyslog (optional)
    |
    v
/var/log files
```

Boot logs:

```bash
journalctl -b | head
```

Live journal:

```bash
sudo journalctl -f
```

Service-specific logs:

```bash
sudo journalctl -u ssh
sudo journalctl -u nginx
sudo journalctl -u kubelet
sudo journalctl -u containerd
```

Last 20 SSH messages:

```bash
sudo journalctl -u ssh -n 20
```

Recent Nginx logs:

```bash
journalctl -u nginx --since "1 hour ago"
```

### Kubernetes connection

When a Kubernetes node is `NotReady`, the notes recommend checking:

```bash
journalctl -u kubelet
journalctl -u containerd
dmesg
```

---

## Containers and logs

Traditional servers often write logs to files.

Containers commonly write logs to:

```text
stdout
stderr
```

Docker/Kubernetes capture these streams.

Therefore:

```bash
kubectl logs <pod>
```

reads the container's captured standard output rather than simply reading:

```text
/var/log/syslog
```

Mental model:

```text
Traditional Linux

Application
    |
    v
Log files
    |
    v
/var/log


Containers

Application
    |
    +-- stdout
    +-- stderr
          |
          v
Container runtime
          |
          v
kubectl logs
```

---

# 15. Package Management with APT

Ubuntu uses APT to manage packages.

Repository configuration can be inspected with:

```bash
cat /etc/apt/sources.list
```

The notes emphasize that package installation uses configured Ubuntu repositories rather than searching the internet manually.

## `apt update`

```bash
sudo apt update
```

This refreshes package metadata.

It does **not** itself upgrade installed packages.

Mental model:

```text
apt update
    =
refresh the software catalog
```

## `apt upgrade`

```bash
sudo apt upgrade
```

This upgrades installed packages.

## Installing a package

```bash
sudo apt install nginx
```

The notes describe the installation flow as:

```text
Resolve dependencies
       |
Download packages
       |
Verify package signatures
       |
Extract files
       |
Register service with systemd
       |
Start service
```

---

# 16. Linux DNS Resolution

Important files:

```bash
/etc/nsswitch.conf
/etc/hosts
/etc/resolv.conf
```

The notes describe the resolution path as:

```text
Application
    |
    v
OS Resolver / glibc
    |
    v
/etc/nsswitch.conf
    |
    v
/etc/hosts
    |
    v
/etc/resolv.conf
    |
    v
systemd-resolved
    |
    v
AWS DNS / upstream DNS
    |
    v
Public DNS
    |
    v
IP address
    |
    v
Application connects
```

## `/etc/hosts`

```bash
cat /etc/hosts
```

Example:

```text
127.0.0.1 localhost
```

You can add a local hostname mapping:

```text
1.2.3.4 mytestsite
```

Then:

```bash
ping mytestsite
```

The local mapping can be used before querying DNS.

## `/etc/resolv.conf`

```bash
cat /etc/resolv.conf
```

The notes show an example:

```text
nameserver 127.0.0.53
```

This can indicate a local resolver such as `systemd-resolved`.

Check resolver status:

```bash
sudo resolvectl status
```

Restart the resolver service if required:

```bash
sudo systemctl restart systemd-resolved
```

## `/etc/nsswitch.conf`

```bash
cat /etc/nsswitch.conf
```

Important line:

```text
hosts: files dns
```

This indicates the lookup order described in the notes:

```text
1. /etc/hosts
2. DNS
```

### DNS troubleshooting

If:

```bash
ping 8.8.8.8
```

works but:

```bash
ping google.com
```

fails, the notes identify DNS resolution as an area to investigate.

Useful commands:

```bash
cat /etc/resolv.conf
resolvectl status
systemctl status systemd-resolved
```

---

## Kubernetes DNS connection

Kubernetes follows the same fundamental idea.

The notes map:

| Linux | Kubernetes |
|---|---|
| `/etc/hosts` | Pod hosts file |
| system resolver | CoreDNS |
| DNS server | `kube-dns` service |
| hostname resolution | Service discovery |

Example:

```text
redis
```

can resolve through Kubernetes DNS to:

```text
redis.default.svc.cluster.local
```

Conceptually:

```text
Application
    |
    v
Linux resolver
    |
    v
CoreDNS
    |
    v
Kubernetes Service IP
```

---

# 17. Storage and Filesystems

Important storage concepts:

| Concept | Meaning |
|---|---|
| `mount` | Attach a filesystem/device to a directory |
| `overlayfs` | Layered filesystem used by containers |
| `tmpfs` | Memory-backed filesystem |
| volumes | Persistent storage mechanism |

The notes emphasize an important container concept:

> A container's writable layer is temporary, so application data should not be treated as persistent storage.

---

# 18. File/Text Processing

Useful commands from the notes:

```bash
cat file
less file
grep
sed
awk
sort
uniq
```

## `grep`

Example:

```bash
cat auth.log | grep -i -A2 -B2 cdrom
```

Where:

- `-A2` → two lines after the match
- `-B2` → two lines before the match

## `sed`

Example environment file:

```text
DB_HOST=10.0.0.5
DB_PORT=5432
DB_USER=appuser
DB_PASS=supersecret123
LOG_LEVEL=debug
MAX_CONNECTIONS=100
```

Find the password entry:

```bash
sed -n '/DB_PASS/p' app.env
```

Replace a value:

```bash
sed -i 's/LOG_LEVEL=debug/LOG_LEVEL=info/' app.env
```

Always be careful when using `sed -i` against configuration files.

## `awk`

Print selected fields:

```bash
cat dpkg.log | awk -F " " '{print $1 $2 $3}'
```

Print the last field:

```bash
cat dpkg.log | awk -F " " '{print $NF}'
```

Print the second-last field:

```bash
cat dpkg.log | awk -F " " '{print $(NF-1)}'
```

Count occurrences:

```bash
cat dpkg.log | awk -F " " '{print $NF}' | uniq -c
```

Sort by frequency:

```bash
cat dpkg.log | awk -F " " '{print $NF}' | uniq -c | sort -rn
```

Another form:

```bash
cat dpkg.log | awk -F " " '{print $NF}' | sort | uniq -c | sort -rn
```

---

# 19. Finding Files

Find directories:

```bash
sudo find / -type d -name "dist-upgrade"
```

Alternative using `locate`:

```bash
locate dist-upgrade
```

Recursive search:

```bash
ls -R /var | grep dist-upgrade
```

Find files larger than 50 MB:

```bash
sudo find / -type f -size +50M
```

---

# 20. SSH, SCP and Sudo

## SSH configuration

Inspect relevant SSH configuration:

```bash
sudo grep -i "PermitRootLogin\|PasswordAuth" /etc/ssh/sshd_config
```

Service logs:

```bash
journalctl -u ssh
```

## SCP

SCP is used for secure file copying.

General form:

```bash
scp -i path-to-your_private_key.pem \
    path-to-your-file \
    username@public-dns-or-ip:/remote/destination/path/
```

---

## Sudo access

Add a user to the sudo group:

```bash
sudo usermod -aG sudo devops
```

Another method is editing sudoers safely with:

```bash
sudo visudo
```

Then add:

```text
devops ALL=(ALL:ALL) ALL
```

---

# 21. Kubernetes Connections

One of the strongest themes in the notes is the connection between Linux internals and Kubernetes.

## Container mental model

```text
Kubernetes Pod
      |
      v
Container
      |
      v
Linux process
      |
      +-- namespaces
      +-- cgroups
      +-- filesystem
      +-- network
      |
      v
Linux kernel
```

### Resource limits

When Kubernetes specifies:

```yaml
resources:
  requests:
    memory: "256Mi"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

the notes connect this to:

```text
Kubernetes
    |
    v
kubelet
    |
    v
container runtime
    |
    v
Linux cgroups
    |
    v
Linux kernel
```

The kernel ultimately enforces the resource boundaries.

### Important mappings

| Linux concept | Kubernetes behavior |
|---|---|
| Process | Container workload |
| Process group | Pod/container workload |
| PID namespace | Process isolation |
| Network namespace | Container networking |
| Mount namespace | Filesystem isolation |
| cgroups | CPU/memory/process limits |
| OOM Killer | `OOMKilled` |
| SIGTERM | Graceful container termination |
| SIGKILL | Forced termination |
| stdout/stderr | `kubectl logs` source |

---

# 22. Linux Quick Revision Cheat Sheet

## Navigation & Filesystem

| Command | Purpose |
|---|---|
| `pwd` | Show current directory |
| `ls` | List files |
| `ls -l` | Detailed listing |
| `cd` | Change directory |
| `cd ..` | Go up one level |
| `ls /` | View root filesystem |
| `touch file` | Create empty file |
| `cat file` | Print file |
| `less file` | Read large file |
| `nano file` | Edit file |

### Interview question

**What is the difference between `/` and `/root`?**

```text
/      = filesystem root
/root  = root user's home directory
```

---

## Users and Permissions

| Command | Purpose |
|---|---|
| `whoami` | Current user |
| `adduser` / `useradd` | Create user |
| `su - user` | Switch user |
| `chmod` | Change permissions |
| `chown` | Change owner |
| `chgrp` | Change group |

### Interview question

**Why can a container fail with `Permission denied`?**

A common cause is a UID/GID or file-ownership mismatch with a mounted volume.

---

## Processes

| Command | Purpose |
|---|---|
| `ps -ef` | List processes |
| `top` | Live CPU/memory usage |
| `kill PID` | Send SIGTERM |
| `kill -9 PID` | Send SIGKILL |

---

## Monitoring

| Command | Purpose |
|---|---|
| `top` | CPU/process overview |
| `free -h` | Memory |
| `df -h` | Filesystem usage |
| `dmesg` | Kernel messages |
| `uptime` | System load |
| `vmstat` | CPU/memory/swap/I/O |
| `sar` | Historical/system statistics |

### Interview trap

If Linux shows high memory utilization:

> Check **Available** memory and cache behavior instead of assuming the machine is out of RAM.

---

## Networking

| Command | Purpose |
|---|---|
| `ip addr` | IP/interface information |
| `ss -lntp` | Listening TCP ports |
| `lsof -i` | Processes using network sockets |
| `ping` | Basic connectivity |
| `curl` | HTTP/service testing |
| `hostname` | Machine hostname |
| `mtr` | Path and latency troubleshooting |
| `tcpdump` | Packet capture |
| `nmap` | Port/service discovery |

Example:

```bash
curl -I http://localhost
```

---

## Logs

| Command | Purpose |
|---|---|
| `journalctl` | Systemd journal |
| `journalctl -u ssh` | SSH service logs |
| `journalctl -f` | Follow journal |
| `tail -f file` | Follow a log file |
| `/var/log/auth.log` | Authentication/security logs |
| `dmesg` | Kernel messages |

---

## Storage

| Command / Concept | Purpose |
|---|---|
| `df -h` | Filesystem usage |
| `du -sh` | Directory size |
| `mount` | Mounted filesystems |
| `tmpfs` | Memory filesystem |
| `overlayfs` | Layered container filesystem |

---

# 23. DevOps Interview Questions

## Linux

### 1. What is the difference between `/` and `/root`?

```text
/     = filesystem root
/root = root user's home directory
```

### 2. What happens when a process receives SIGTERM?

It is asked to terminate gracefully.

### 3. What is the difference between SIGTERM and SIGKILL?

```text
SIGTERM -> graceful termination request
SIGKILL -> immediate forced termination
```

### 4. Why can a container get `Permission denied`?

A common reason is UID/GID or filesystem ownership mismatch.

### 5. Why is `chmod 777` dangerous?

It gives very broad permissions and can allow unintended users/processes to modify files.

### 6. What is a Linux process?

A running program in memory.

### 7. What is PID 1?

On a normal Linux server it is typically the init/service-manager process. Inside a container, PID 1 is normally the container's main process.

### 8. What is a cgroup?

A Linux mechanism for grouping processes and controlling resources such as CPU, memory, I/O and process count.

### 9. What causes `OOMKilled` in Kubernetes?

The notes connect it to a process exceeding its available memory/cgroup boundary and the Linux OOM mechanism terminating the workload.

### 10. Does a CPU limit normally kill a container?

No. CPU limits result in throttling; the process remains alive but may become slower.

### 11. Why can a server show high RAM usage while still being healthy?

Linux uses spare RAM for filesystem/cache purposes. Available memory is therefore more useful than looking only at the "used" number.

### 12. How do you find which process is using port 8080?

```bash
lsof -i :8080
```

or:

```bash
ss -lntp
```

### 13. Nginx cannot start because port 80 is already in use. What do you do?

```bash
sudo ss -lntp
sudo lsof -i :80
```

Identify the process and determine whether it should be stopped/reconfigured.

### 14. How do you troubleshoot a service?

Start with:

```bash
systemctl status <service>
journalctl -u <service>
```

### 15. How do you troubleshoot a Kubernetes node that is `NotReady`?

The notes recommend checking:

```bash
journalctl -u kubelet
journalctl -u containerd
dmesg
```

### 16. How do you troubleshoot DNS?

Check:

```bash
cat /etc/hosts
cat /etc/resolv.conf
cat /etc/nsswitch.conf
resolvectl status
systemctl status systemd-resolved
```

### 17. A pod can reach a Kubernetes Service but cannot reach an external endpoint. What areas do you investigate?

The notes identify:

- DNS
- routing
- network policy

---

# Production Troubleshooting: First Commands

## Server is slow

```bash
top
free -h
vmstat 1 5
```

Then investigate CPU, memory, swap and I/O.

## Disk looks suspicious

```bash
df -h
du -sh */
dmesg | grep -i "fail"
```

## Service is failing

```bash
systemctl status <service>
journalctl -u <service>
```

## Port is already in use

```bash
ss -lntp
lsof -i :<port>
```

## DNS is failing

```bash
cat /etc/hosts
cat /etc/resolv.conf
cat /etc/nsswitch.conf
resolvectl status
```

## Kubernetes pod keeps restarting

Check:

```bash
kubectl logs <pod>
kubectl describe pod <pod>
```

Then correlate with Linux/kernel behavior:

```bash
dmesg
journalctl -u kubelet
journalctl -u containerd
```

If the reason is:

```text
OOMKilled
```

investigate memory usage and limits rather than increasing CPU.

---

# Useful Advanced Commands

## Network path

```bash
mtr google.com
```

Report mode:

```bash
mtr --report -c 10 10.0.0.5
```

## Packet capture

Example for PostgreSQL traffic:

```bash
tcpdump -i eth0 port 5432 -c 20
```

## Port scanning / service discovery

```bash
nmap localhost
```

## Check kernel OOM/failure messages

```bash
sudo dmesg | grep -i "oom\|kill\|fail"
```

## Check directory sizes

```bash
du -sh */
```

---

# Core Mental Models

## Linux → Docker → Kubernetes

```text
Linux Kernel
     |
     +-----------------------+
     |                       |
 namespaces               cgroups
     |                       |
     v                       v
process isolation       resource limits
     |                       |
     +-----------+-----------+
                 |
                 v
             Container
                 |
                 v
                Pod
                 |
                 v
            Kubernetes
```

## Memory failure

```text
Application
    |
    v
Memory usage increases
    |
    v
cgroup memory boundary
    |
    v
Linux OOM handling
    |
    v
Process killed
    |
    v
Container exits
    |
    v
Kubelet observes failure
    |
    v
Container/Pod restarted
```

## CPU saturation

```text
Application
    |
    v
CPU demand increases
    |
    v
CPU limit reached
    |
    v
cgroup CPU throttling
    |
    v
Application remains alive
    |
    v
Latency increases
```

## DNS

```text
Application
    |
    v
glibc / OS resolver
    |
    v
nsswitch.conf
    |
    +---- /etc/hosts
    |
    +---- DNS configuration
              |
              v
        systemd-resolved
              |
              v
          upstream DNS
              |
              v
          IP address
```

---

# Final DevOps Mental Model

The most important theme in these notes is the relationship between **Linux kernel primitives and higher-level DevOps platforms**.

```text
                    Kubernetes
                        |
                        v
                   Container Runtime
                        |
              +---------+---------+
              |                   |
          Namespaces            cgroups
              |                   |
              v                   v
        Isolation            Resource Control
              |                   |
              +---------+---------+
                        |
                        v
                   Linux Kernel
                        |
        +---------------+---------------+
        |               |               |
      Process         Network          Memory
      Signals        Namespace         / CPU
        |               |               |
        +---------------+---------------+
                        |
                        v
                  Physical / VM
                     Resources
```

Understanding these Linux primitives makes Docker and Kubernetes behavior much easier to troubleshoot.

---

## Quick Reference: Most Important Commands

```bash
# Identity
whoami

# Filesystem
pwd
ls -l
cd
cat
less

# Users
cat /etc/passwd
cat /etc/shadow

# Processes
ps -ef
top
ps -p 1 -o pid,ppid,cmd

# Process control
kill PID
kill -9 PID

# Memory
free -h
cat /proc/meminfo
cat /proc/1/oom_score

# cgroups
cat /proc/self/cgroup
systemd-cgtop --user

# Networking
ip addr
ss -lntp
lsof -i
lsof -i :8080
ping
curl -I http://localhost

# DNS
cat /etc/hosts
cat /etc/resolv.conf
cat /etc/nsswitch.conf
resolvectl status

# Logs
journalctl
journalctl -u nginx
journalctl -u kubelet
journalctl -f
dmesg
tail -f /var/log/syslog

# Packages
sudo apt update
sudo apt upgrade
sudo apt install <package>

# Storage
df -h
du -sh */

# Search
find
locate

# Text processing
grep
sed
awk
sort
uniq

# Network troubleshooting
mtr
tcpdump
nmap

# SSH / file transfer
scp

# Sudo
sudo visudo
usermod -aG sudo <user>
```

---

> **Study approach:** Don't memorize these commands in isolation. Learn the Linux subsystem behind each command — processes, namespaces, cgroups, networking, filesystems, systemd, and the kernel. That mental model is what makes Docker and Kubernetes troubleshooting much easier.
