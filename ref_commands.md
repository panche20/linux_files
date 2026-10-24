**1. System Identification & Basic Navigation**

whoami: Displays the active logged-in shell user
.
pwd: Prints the absolute path of the Present Working Directory
.
uptime: Displays continuous server uptime, active user count, and load averages
.
hostnamectl: Shows OS distribution, kernel version, architecture, and system metadata
.
uname -r / uname -a: Prints kernel release details or complete system information
.
lsb_release -a: Displays Linux Standard Base and Ubuntu release version
.
printenv / echo $VAR: Inspects environment variables (e.g., echo $PWD, echo $SHELL)
.
exit: Terminate and log out of the current SSH or terminal session
.
**2. File System Operations & Manipulation**

ls / ls -a / ls -la / ls -lh: Lists directory contents; flags reveal hidden files (-a), detailed permissions/owners (-l), and human-readable file sizes (-h)
.
cd: Changes directory (cd .. to move up, cd ~ for home directory, cd - to jump back to the previous location)
.
mkdir / mkdir -p: Creates directories; -p recursively builds nested parent directories
.
touch: Creates new empty files or updates access timestamps
.
cat: Displays file contents directly to standard output
.
echo with > & >>: Redirects string output to overwrite (>) or append (>>) to a file
.
cp / mv: Copies or moves/renames files and directories (supports wildcard matching like *.txt)
.
rm / rm -r / rm -rf: Removes files or directories recursively (-r) and forcefully (-f)
.
less: Interactive viewer to scroll through large log files page-by-page
.
man / --help: Accesses official command manuals or quick flag option reference lists
.
**3. Users, Groups & Security Permissions**

/etc/passwd & /etc/group: System configuration files listing all user accounts and group memberships
.
who / last: Shows currently logged-in users or historical login and reboot logs
.
sudo / sudo !!: Executes commands with elevated root privileges; sudo !! re-executes the previous command with sudo
.
useradd / adduser: Creates user accounts (low-level utility vs. interactive setup)
.
groupadd / usermod -aG: Creates user groups and appends users to supplementary groups
.
chmod: Modifies file permissions using numeric octal mode (e.g., 755, 700, 644) or symbolic notation (u+x, g-w)
.
chown / chown -R: Changes file or directory user and group ownership recursively
.
grep: Filters lines matching a pattern from files or command outputs
.
ssh-keygen: Generates SSH public and private key pairs
.
vim: Terminal text editor (i for insert mode, :wq to save and quit, :q! to exit without saving)
.
**4. Package Management (APT, NVM, PNPM)**

apt update / apt upgrade: Refreshes package indices and upgrades installed packages
.
apt list --upgradable: Identifies all packages with available upgrades
.
apt install / apt remove / apt autoremove: Installs packages, removes applications, or cleans orphaned dependencies
.
tree -L <depth>: Displays directory hierarchy in a visual tree format up to specified depth
.
which: Locates the absolute filesystem path of an executable binary
.
nvm: Node Version Manager (nvm install --lts, nvm use, nvm ls-remote)
.
pnpm / npm: Node.js package managers (pnpm install, pnpm build, pnpm start)
.
**5. Process Management & Job Control**

ps / ps aux: Captures a snapshot of active processes across all users
.
top / htop: Real-time process monitoring; htop offers an interactive color-coded interface (F4 filter, F6 sort, F9 kill)
.
& / Ctrl + Z / bg / jobs: Background execution (&), process suspension (Ctrl + Z), background resuming (bg), and job tracking (jobs)
.
kill / kill -9 / pkill: Sends signals to processes by PID or process name (-15 SIGTERM, -9 SIGKILL)
.
nohup: Runs commands immune to session hangups or SSH disconnects
.
**6. Systemd Service Management & Monitoring**

systemctl daemon-reload: Reloads systemd configuration after editing .service unit files
.
systemctl start / stop / restart / reload / status: Controls service lifecycle states and checks execution logs
.
systemctl enable / enable --now / disable: Configures whether services start automatically upon system boot
.
journalctl / journalctl -u <service> -n 50: Queries systemd system and service logs
.
watch -n <sec> <command>: Re-executes commands periodically to track real-time changes
.
free -h: Displays RAM and swap memory consumption in human-readable units
.
**7. Networking, Web Servers & Reverse Proxies**

curl: CLI HTTP client to fetch web content or test API endpoints
.
caddy: Modern web server and reverse proxy with automatic SSL/TLS certificate management (caddy validate, systemctl reload caddy)
.
dig: DNS lookup utility to inspect domain record resolution
.
**8. DevOps Process Utilities & Version Control**

pm2 start / pm2 list / pm2 logs: Production Node.js process manager for multi-app management and log streaming
.
git clone / git commit --amend / git remote -v: Essential Git operations for managing repository code on remote Linux servers
.
