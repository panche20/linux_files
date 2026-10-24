# Top 50 Linux Commands for DevOps Engineers

A comprehensive reference guide covering the essential Linux commands every DevOps engineer needs for system administration, performance monitoring, troubleshooting, networking, log analysis, and security.

> **Source**: Based on the breakdown from *"I Broke Down Top 50 Linux Command a DevOps Engineer Needs — So You Don't Have To"* by *Tech Tutorials with Piyush*.

---

## 📋 Table of Contents

- [1. System Health & Resource Monitoring](#1-system-health--resource-monitoring)
- [2. Process Management & Control](#2-process-management--control)
- [3. Log Analysis & Text Processing](#3-log-analysis--text-processing)
- [4. File System, Search & Storage](#4-file-system-search--storage)
- [5. Networking & Network Diagnostics](#5-networking--network-diagnostics)
- [6. Users, Permissions & Security](#6-users-permissions--security)
- [7. System Utilities & Binary Diagnostics](#7-system-utilities--binary-diagnostics)
- [🛠️ Essential DevOps Troubleshooting Workflows](#️-essential-devops-troubleshooting-workflows)

---

## 1. System Health & Resource Monitoring

Commands used to evaluate system performance, CPU/memory usage, load averages, and disk metrics.

| Command | Purpose | Common Flags / Usage Examples |
| :--- | :--- | :--- |
| **`ssh`** | Securely log into a remote Linux server via encrypted shell | `ssh user@host` |
| **`uptime`** | Displays how long the system has been running, active users, and load averages (1, 5, 15 min) | `uptime` |
| **`free`** | Displays total, used, and available RAM and swap memory | `free -h` *(human-readable)*, `free -m` *(megabytes)* |
| **`df`** | Shows disk space utilization per mounted file system | `df -h` *(human-readable)*, `df -T` *(file system type)*, `df -i` *(inode utilization)* |
| **`top`** | Real-time interactive process viewer for CPU and memory tracking | `top` *(Press `P` to sort by CPU, `M` for Memory, `q` to quit)* |
| **`htop`** | User-friendly, color-coded interactive process manager | `htop` |
| **`vmstat`** | Reports virtual memory, process, CPU, and I/O statistics | `vmstat 1 5` *(Interval: 1 sec, Count: 5 times)* |
| **`iostat`** | Displays CPU and I/O performance per device/disk | `iostat -x -h 1 3` *(Extended stats, human-readable)* |
| **`sar`** | Collects and reports historical system performance data (CPU, memory, network) | `sar -u 1 3` *(CPU)*, `sar -r 1 3` *(RAM)*, `sar -n DEV 1 3` *(Network)* |

---

## 2. Process Management & Control

Commands used to view, control, prioritize, and terminate running processes and system services.

| Command | Purpose | Common Flags / Usage Examples |
| :--- | :--- | :--- |
| **`ps`** | Reports a snapshot of active processes | `ps -ef`, `ps aux`, `ps aux --sort=-%mem` |
| **`kill`** | Sends a termination signal to a process using its Process ID (PID) | `kill <PID>`, `kill -9 <PID>` *(forceful kill)* |
| **`pkill`** | Kills processes based on name rather than PID | `pkill nginx`, `pkill -f app_service` |
| **`nice` / `renice`** | Sets or alters the scheduling priority of a process (-20 highest, 19 lowest) | `nice -n 10 <command>`, `renice -n -5 -p <PID>` |
| **`systemctl`** | Controls systemd services and system state | `systemctl status <service>`, `systemctl restart <service>` |

---

## 3. Log Analysis & Text Processing

Essential utilities for inspecting logs, searching for patterns, manipulating strings, and extracting fields.

| Command | Purpose | Common Flags / Usage Examples |
| :--- | :--- | :--- |
| **`tail`** | Outputs the last part of files; streams live log updates | `tail -n 50 app.log`, `tail -f /var/log/syslog` *(live stream)* |
| **`head`** | Outputs the beginning portion of files or piped output | `head -n 20 app.log` |
| **`cat`** | Concatenates and prints file contents to standard output | `cat app.log` |
| **`less`** | Interactive terminal pager for reading large text files | `less /var/log/nginx/access.log` |
| **`grep`** | Searches text for specified patterns | `grep -i "error" app.log`, `grep -c "FAIL" log.txt`, `grep -A 2 -B 2 "EXCEPTION" log.txt` |
| **`sed`** | Stream editor for filtering and transforming text | `sed -i 's/debug/info/g' config.env` *(in-place replace)* |
| **`awk`** | Pattern scanning and text-processing language for field extraction | `awk -F' ' '{print $1, $2, $3}' app.log`, `awk '{print $NF}'` *(last field)* |
| **`uniq`** | Filters out or reports repeated lines from sorted text | `uniq -c` *(count occurrences)* |
| **`sort`** | Sorts lines of text files | `sort -n -r` *(numeric reverse sort)* |
| **`dmesg`** | Prints ring buffer messages from kernel and system drivers | `dmesg -T` *(human-readable timestamps)*, `dmesg \| grep -i error` |
| **`journalctl`** | Queries and displays systemd journal logs | `journalctl -u <service_name> -f`, `journalctl --since "1 hour ago"` |

---

## 4. File System, Search & Storage

Utilities for navigating directories, searching for files, managing storage, archiving, and managing disk partitions.

| Command | Purpose | Common Flags / Usage Examples |
| :--- | :--- | :--- |
| **`ls`** | Lists directory contents | `ls -lrt` *(long format, sorted by time reverse)* |
| **`cd`** | Changes the current working directory | `cd /var/log`, `cd ..` |
| **`find`** | Searches directory trees for files matching criteria | `find /var -type f -name "*.log"`, `find . -size +50M`, `find . -mtime +60 -delete` |
| **`du`** | Estimates file and directory space usage | `du -sh *` *(human-readable summary per directory)* |
| **`rm`** | Removes files or directories *(Use with extreme caution)* | `rm file.txt`, `rm -rf directory/` *(recursive & forced)* |
| **`touch`** | Creates an empty file or updates file timestamps | `touch app.env` |
| **`tar`** | Creates, extracts, and manages compressed file archives | `tar -czf archive.tar.gz /path`, `tar -xzf archive.tar.gz` |
| **`lsblk`** | Lists information about all available block devices and partitions | `lsblk` |
| **`fdisk`** | Partition table manipulator for Linux disks | `fdisk -l` *(list partition tables)* |

---

## 5. Networking & Network Diagnostics

Commands used to test connectivity, monitor active sockets/ports, query DNS, analyze traffic, and audit network routes.

| Command | Purpose | Common Flags / Usage Examples |
| :--- | :--- | :--- |
| **`curl`** | Transfers data to/from servers via supported protocols | `curl -I http://localhost` *(fetch headers)* |
| **`wget`** | Non-interactive network downloader | `wget https://example.com/file.tar.gz` |
| **`ss`** | Dumps socket statistics; modern replacement for `netstat` | `ss -tulpn` *(TCP, UDP, Listening, Process, Numeric)* |
| **`netstat`** | Legacy network statistics viewer | `netstat -tulpn` |
| **`nslookup`** | Queries DNS servers for domain IP resolution | `nslookup google.com` |
| **`ip`** | Shows and manages network interfaces, IP addresses, and routing | `ip addr show`, `ip route` |
| **`mtr`** | Network diagnostic tool combining `traceroute` and `ping` | `mtr google.com` |
| **`tcpdump`** | Command-line packet analyzer for network traffic inspection | `tcpdump -i eth0 port 80 -c 20` |
| **`nmap`** | Network exploration tool and port scanner | `nmap localhost`, `nmap -p 22,80,443 192.168.1.1` |

---

## 6. Users, Permissions & Security

Commands for identity management, user permissions, ownership modifications, and host-level firewall management.

| Command | Purpose | Common Flags / Usage Examples |
| :--- | :--- | :--- |
| **`id`** | Displays user and group IDs for a specified user | `id`, `id devuser` |
| **`whoami`** | Prints the effective username of the current session | `whoami` |
| **`usermod`** | Modifies user account properties (e.g., group memberships) | `usermod -aG docker devuser` |
| **`groupmod`** | Modifies group definitions | `groupmod -n new_name old_name` |
| **`chmod`** | Changes file and directory access permissions | `chmod 700 file.sh`, `chmod +x script.py` |
| **`chown`** | Changes file user owner and group owner | `chown user:group /opt/app` |
| **`iptables`** | Administration tool for IPv4 packet filtering and NAT firewalls | `iptables -L -v -n` |

---

## 7. System Utilities & Binary Diagnostics

Tools for automation, process inspection, system call tracing, library dependency checks, and root path manipulation.

| Command | Purpose | Common Flags / Usage Examples |
| :--- | :--- | :--- |
| **`man`** | Displays online reference manual pages | `man free`, `man top` |
| **`xargs`** | Builds and executes commands from standard input | `find . -name "*.tmp" \| xargs rm -f` |
| **`lsof`** | Lists open files and the processes that opened them | `lsof -i :80` *(open port 80)*, `lsof -p <PID>` |
| **`strace`** | Traces system calls and signals during execution | `strace -p <PID> -e trace=open,read,write` |
| **`crontab`** | Maintains crontab files for task scheduling | `crontab -e` *(edit)*, `crontab -l` *(list)* |
| **`which`** | Returns the executable path of a command | `which nginx`, `which python3` |
| **`ldd`** | Prints shared library dependencies of an executable | `ldd /usr/bin/nginx` |
| **`chroot`** | Runs a command or interactive shell with a modified root directory | `chroot /mnt/recovery` |

---

## 🛠️ Essential DevOps Troubleshooting Workflows

### 1. Investigating High CPU or Memory Usage
```bash
# 1. Identify top resource-consuming processes
top -b -n 1 | head -n 20

# 2. Sort active processes by memory usage
ps aux --sort=-%mem | head -n 10

# 3. Check memory and swap usage
free -h
```

### 2. Diagnosing Full Disk or Out-of-Inode Errors
```bash
# 1. Check disk space by file system
df -h

# 2. Check inode utilization
df -i

# 3. Find top 10 largest directories/files
du -sh /* 2>/dev/null | sort -rh | head -n 10
```

### 3. Extracting & Counting Unique Error Logs
```bash
# Extract error status, count unique occurrences, and sort in descending order
awk -F' ' '{print $NF}' /var/log/my-app/app.log | sort | uniq -c | sort -nr
```

### 4. Checking Listening Network Ports
```bash
# Show all active TCP/UDP listening ports with PIDs and binary names
ss -tulpn
```

---

*Generated for Gemini Notebook — TOP 50 Linux Commands*
