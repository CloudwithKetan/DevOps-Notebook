# 🐧 Linux & Shell Scripting — Complete Notes



![Linux Filesystem](images/linux-filesystem.svg)

> Linux is the backbone of DevOps. Almost every server, container, and cloud VM runs Linux. Being comfortable on the command line is not optional — it's the foundation everything else is built on.

---

## 1. Linux File System Structure

Linux organises everything as files — including hardware devices and processes. The structure starts from `/` (root) and branches down:

```
/
├── bin/      → Essential binaries (ls, cp, mv, bash)
├── sbin/     → System binaries (for root/admin use)
├── etc/      → Configuration files (nginx.conf, /etc/hosts)
├── home/     → User home directories (/home/omkar, /home/rohit)
├── root/     → Home directory for the root user
├── var/      → Variable data: logs (/var/log), databases, mail
├── tmp/      → Temporary files (cleared on reboot)
├── usr/      → User programs and libraries
├── opt/      → Optional/third-party software
├── proc/     → Virtual filesystem — running processes info
├── dev/      → Device files (disks, terminals)
├── mnt/      → Temporary mount points
└── lib/      → Shared libraries used by binaries
```

**Practical tip:** When a service breaks, check `/var/log/` first — that's where log files live (`/var/log/nginx/error.log`, `/var/log/syslog`, etc.)

---

## 2. Navigation & File Operations

```bash
# Where am I?
pwd                         # Print working directory

# Move around
cd /var/log                 # Go to absolute path
cd ..                       # Go up one level
cd ~                        # Go to your home directory
cd -                        # Go to previous directory (toggle)

# List files
ls                          # Basic list
ls -l                       # Long format (permissions, owner, size, date)
ls -la                      # Include hidden files (starting with .)
ls -lh                      # Human-readable sizes (KB, MB, GB)
ls -lt                      # Sort by modification time (newest first)

# Create
touch file.txt              # Create empty file (or update timestamp)
mkdir mydir                 # Create directory
mkdir -p /a/b/c             # Create nested directories (-p = parents)

# Copy, Move, Delete
cp file.txt backup.txt      # Copy file
cp -r mydir/ backup/        # Copy directory recursively
mv file.txt /tmp/           # Move file (also used to rename)
rm file.txt                 # Delete file (NO recycle bin!)
rm -rf mydir/               # Force delete directory (dangerous — double check)

# View file contents
cat file.txt                # Print entire file
less file.txt               # Page through file (q to quit)
head -n 20 file.txt         # First 20 lines
tail -n 20 file.txt         # Last 20 lines
tail -f /var/log/app.log    # Follow a log file in real time
```

---

## 3. Permissions — Understanding rwx

Every file and directory has permissions for three groups: **owner**, **group**, and **others**.

```
-rwxr-xr--  1  omkar  devops  4096  Apr 25 10:00  script.sh
 ↑↑↑↑↑↑↑↑↑     ↑↑↑↑↑  ↑↑↑↑↑↑
 │└──┤└──┤└─┤   owner  group
 │   │   │  └── Others: r-- (read only)
 │   │   └───── Group: r-x (read + execute)
 │   └───────── Owner: rwx (read + write + execute)
 └───────────── File type: - = file, d = directory, l = symlink
```

| Symbol | Octal | Meaning |
|--------|-------|---------|
| r | 4 | Read |
| w | 2 | Write |
| x | 1 | Execute |
| rw- | 6 | Read + Write |
| rwx | 7 | Read + Write + Execute |
| r-x | 5 | Read + Execute |

```bash
# Change permissions
chmod 755 script.sh        # Owner: rwx, Group: r-x, Others: r-x
chmod +x script.sh         # Add execute permission for everyone
chmod -w file.txt          # Remove write permission
chmod 600 private.key      # Owner: rw, Group: none, Others: none (SSH keys)

# Change owner
chown omkar file.txt       # Change owner to omkar
chown omkar:devops file.txt # Change owner and group
chown -R omkar mydir/      # Recursive — change entire directory
```

**Common permission numbers to remember:**
- `777` — everyone can do everything (dangerous, avoid)
- `755` — typical for scripts and directories
- `644` — typical for config files (owner writes, others read)
- `600` — private files (SSH keys, credentials)

---

## 4. Searching & Filtering

```bash
# Find files
find /home -name "*.log"            # Find by name
find /var/log -size +100M           # Files larger than 100MB
find . -type f -newer config.txt    # Files modified after config.txt
find /tmp -mtime +7 -delete         # Delete files older than 7 days

# Search inside files
grep "error" app.log                # Lines containing "error"
grep -i "error" app.log             # Case insensitive
grep -r "TODO" ./src/               # Recursive search in directory
grep -n "error" app.log             # Show line numbers
grep -v "debug" app.log             # Lines NOT containing "debug"
grep -c "error" app.log             # Count matching lines

# Combining tools with pipe |
cat access.log | grep "404" | wc -l  # Count 404 errors
ps aux | grep nginx                   # Find nginx processes
```

---

## 5. Process Management

```bash
# View running processes
ps aux                          # All processes, detailed
ps aux | grep nginx             # Filter for nginx
top                             # Interactive process viewer (q to quit)
htop                            # Better version of top (may need install)

# Kill processes
kill <PID>                      # Graceful stop (SIGTERM)
kill -9 <PID>                   # Force kill (SIGKILL) — use as last resort
killall nginx                   # Kill all processes named nginx

# Background jobs
./script.sh &                   # Run in background
jobs                            # List background jobs
fg %1                           # Bring job 1 to foreground
nohup ./script.sh &             # Run even after logout (logs to nohup.out)
```

---

## 6. Networking Commands

```bash
# Check connectivity
ping google.com                 # Test reachability
ping -c 4 google.com            # Send exactly 4 packets

# DNS lookup
nslookup google.com             # Query DNS
dig google.com                  # Detailed DNS info
dig +short google.com           # Just the IP

# Network interfaces
ip addr show                    # Show IPs on all interfaces
ip route show                   # Show routing table
ifconfig                        # Older alternative

# Open ports
netstat -tulnp                  # All listening ports with process names
ss -tulnp                       # Faster modern alternative

# Test connections
curl -I https://google.com      # HTTP headers only
curl -o /dev/null -s -w "%{http_code}" https://api.com  # Just HTTP status code
wget https://example.com/file.zip  # Download file

# Firewall (UFW - Ubuntu)
ufw status
ufw allow 80/tcp
ufw deny 22
ufw enable
```

---

## 7. Disk & System Info

```bash
# Disk usage
df -h                           # Disk space by filesystem (human readable)
du -sh /var/log/                # Size of a specific directory
du -sh /* | sort -rh | head -10  # Top 10 largest directories

# Memory
free -h                         # RAM usage in human readable format

# System info
uname -a                        # Kernel version and system info
uptime                          # How long system has been running
hostname                        # System hostname
whoami                          # Current user
id                              # UID, GID, groups
```

---

## 8. Bash Scripting

### Script structure

```bash
#!/bin/bash
# Above line = shebang — tells OS to run this with bash
# Script: deploy.sh
# Purpose: Deploy application

# Exit immediately if any command fails
set -e

# Variables
APP_NAME="myapp"
VERSION="1.2.0"
DEPLOY_DIR="/var/www/$APP_NAME"

echo "Deploying $APP_NAME version $VERSION..."
```

### Variables

```bash
# Assign (no spaces around =)
name="omkar"
count=10
today=$(date +%Y-%m-%d)    # Command substitution

# Use variable
echo "Hello, $name"
echo "Today is $today"
echo "Files: $(ls | wc -l)"

# Special variables
echo $0    # Script name
echo $1    # First argument passed to script
echo $2    # Second argument
echo $@    # All arguments
echo $#    # Number of arguments
echo $?    # Exit code of last command (0 = success)
echo $$    # Current process ID (PID)
```

### Conditionals

```bash
# if / elif / else
if [ "$1" == "start" ]; then
    echo "Starting service..."
    systemctl start nginx
elif [ "$1" == "stop" ]; then
    echo "Stopping service..."
    systemctl stop nginx
else
    echo "Usage: $0 [start|stop]"
    exit 1
fi

# File/directory checks
if [ -f "/etc/nginx/nginx.conf" ]; then
    echo "Config file exists"
fi

if [ -d "/var/www/html" ]; then
    echo "Web root directory exists"
fi

if [ ! -d "$DEPLOY_DIR" ]; then
    mkdir -p "$DEPLOY_DIR"
    echo "Created deploy directory"
fi

# Number comparisons
if [ $count -gt 10 ]; then echo "More than 10"; fi
if [ $count -eq 0 ]; then echo "Zero"; fi
# -eq  equal | -ne  not equal | -gt  greater | -lt  less
```

### Loops

```bash
# for loop — iterate over list
for server in web1 web2 web3; do
    echo "Deploying to $server..."
    ssh $server "cd /app && git pull && systemctl restart app"
done

# for loop — iterate over files
for file in /var/log/*.log; do
    echo "Processing: $file"
    gzip "$file"
done

# while loop
counter=1
while [ $counter -le 5 ]; do
    echo "Attempt $counter"
    counter=$((counter + 1))
done

# Loop with C-style syntax
for ((i=1; i<=10; i++)); do
    echo "Number: $i"
done
```

### Functions

```bash
# Define a function
check_service() {
    local service_name=$1   # Local variable — only inside function
    
    if systemctl is-active --quiet "$service_name"; then
        echo "✅ $service_name is running"
        return 0
    else
        echo "❌ $service_name is NOT running"
        return 1
    fi
}

# Call the function
check_service nginx
check_service mysql

# Function with return value check
if check_service nginx; then
    echo "Proceeding with deployment"
else
    echo "Nginx not running — aborting"
    exit 1
fi
```

### Practical Script Example — Health Check

```bash
#!/bin/bash
# health-check.sh — check if services are running and disk/memory are OK

set -euo pipefail  # Exit on error, undefined vars, pipe failures

ALERT_EMAIL="admin@company.com"
DISK_THRESHOLD=85   # Alert if disk > 85%
MEM_THRESHOLD=90    # Alert if memory > 90%

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1"
}

check_service() {
    local svc=$1
    if systemctl is-active --quiet "$svc"; then
        log "✅ $svc: OK"
    else
        log "❌ $svc: FAILED — attempting restart"
        systemctl restart "$svc"
        sleep 3
        if systemctl is-active --quiet "$svc"; then
            log "✅ $svc: Restarted successfully"
        else
            log "🚨 $svc: Could not restart — sending alert"
            echo "$svc failed on $(hostname)" | mail -s "Service Alert" "$ALERT_EMAIL"
        fi
    fi
}

check_disk() {
    local usage
    usage=$(df / | awk 'NR==2 {print $5}' | tr -d '%')
    if [ "$usage" -gt "$DISK_THRESHOLD" ]; then
        log "🚨 Disk usage is ${usage}% — threshold is ${DISK_THRESHOLD}%"
        echo "Disk at ${usage}% on $(hostname)" | mail -s "Disk Alert" "$ALERT_EMAIL"
    else
        log "✅ Disk usage: ${usage}%"
    fi
}

log "=== Health Check Started ==="
check_service nginx
check_service mysql
check_disk
log "=== Health Check Complete ==="
```

---

## 9. Cron Jobs — Scheduled Tasks

Cron runs scripts automatically on a schedule.

```bash
# Edit your crontab
crontab -e

# Format: minute hour day month weekday command
# ┌───── minute (0–59)
# │ ┌─── hour (0–23)
# │ │ ┌─ day of month (1–31)
# │ │ │ ┌ month (1–12)
# │ │ │ │ ┌ weekday (0=Sun, 6=Sat)
# │ │ │ │ │
# * * * * *  command

# Examples:
0 * * * *   /scripts/hourly-backup.sh          # Every hour at :00
0 2 * * *   /scripts/daily-backup.sh           # Every day at 2 AM
0 2 * * 0   /scripts/weekly-report.sh          # Every Sunday at 2 AM
0 9 1 * *   /scripts/monthly-invoice.sh        # 1st of each month at 9 AM
*/5 * * * * /scripts/health-check.sh           # Every 5 minutes

# Always use full paths in cron — cron has a minimal environment
# Redirect output to avoid silent failures:
0 2 * * * /scripts/backup.sh >> /var/log/backup.log 2>&1
```

---

## 10. Environment Variables

```bash
# View all environment variables
env
printenv

# View one variable
echo $HOME
echo $PATH
echo $USER

# Set temporarily (only for current session)
export MY_VAR="hello"

# Set permanently — add to ~/.bashrc or ~/.bash_profile
echo 'export MY_APP_KEY="abc123"' >> ~/.bashrc
source ~/.bashrc    # Reload without restarting terminal

# System-wide — add to /etc/environment
# Never put secrets in environment files committed to Git
```

---

*These Linux skills form the foundation — every other DevOps tool (Docker, K8s, CI/CD) relies on comfort with the command line.*
