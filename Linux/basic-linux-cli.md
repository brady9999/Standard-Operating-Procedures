# Basic Linux CLI
> A reference guide to essential Linux command line interface commands for system administration.

**Category:** Linux  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **CLI** | Command Line Interface | Text-based interface for interacting with the OS |
| **Shell** | Shell | The program that interprets and executes commands — bash is most common |
| **Bash** | Bourne Again Shell | The default shell on most Linux distributions |
| **Terminal** | Terminal Emulator | The application providing access to the shell |
| **Root** | Root User | The superuser with unrestricted access to the system |
| **sudo** | Superuser Do | Run a command with root privileges |
| **PATH** | PATH Variable | A list of directories the shell searches for commands |
| **stdin** | Standard Input | Default input stream — usually the keyboard |
| **stdout** | Standard Output | Default output stream — usually the terminal |
| **stderr** | Standard Error | Error output stream |
| **Pipe** | Pipe | Sends the output of one command as input to another using `\|` |
| **Redirect** | Redirect | Sends output to a file using `>` or `>>` |
| **Flag** | Command Flag | An option passed to a command e.g. `ls -la` |
| **Argument** | Argument | A value passed to a command e.g. `cd /home` |
| **Wildcard** | Wildcard | Characters that match multiple files e.g. `*` matches everything |
| **Shebang** | Shebang | The first line of a script `#!/bin/bash` telling the OS which interpreter to use |

---

## Overview
The Linux CLI is the primary tool for system administration. Most server management, automation, and troubleshooting is done via the command line. Mastering these commands is essential for working with any Linux system.

**Getting help for any command:**
```bash
man command        # Full manual page
command --help     # Quick help summary
info command       # Detailed info page
whatis command     # One-line description
```

---

## 1. Navigation and File System

```bash
# Print current directory
pwd

# List files and directories
ls              # Basic list
ls -l           # Long format (permissions, size, date)
ls -la          # Include hidden files (starting with .)
ls -lh          # Human-readable file sizes
ls -lt          # Sort by modification time

# Change directory
cd /path/to/dir     # Go to absolute path
cd ..               # Go up one directory
cd ~                # Go to home directory
cd -                # Go to previous directory

# Create directories
mkdir dirname           # Create a directory
mkdir -p dir/sub/dir    # Create nested directories

# Remove files and directories
rm filename             # Remove a file
rm -r dirname           # Remove directory and contents recursively
rm -rf dirname          # Force remove without prompts (use carefully)
rmdir dirname           # Remove empty directory only

# Copy files
cp source dest          # Copy file
cp -r source dest       # Copy directory recursively
cp -p source dest       # Preserve permissions and timestamps

# Move and rename
mv source dest          # Move or rename file/directory

# Find files
find /path -name "filename"         # Find by name
find /path -name "*.log"            # Find by pattern
find /path -type f -size +100M      # Find files larger than 100MB
find /path -mtime -7                # Find files modified in last 7 days
find /path -user username           # Find files owned by user

# Locate (uses a database — faster than find)
locate filename         # Search the locate database
updatedb                # Update the locate database
```

---

## 2. Viewing and Editing Files

```bash
# View file contents
cat filename            # Print entire file
cat -n filename         # Print with line numbers
less filename           # Scroll through file (q to quit)
more filename           # Similar to less
head filename           # First 10 lines
head -n 20 filename     # First 20 lines
tail filename           # Last 10 lines
tail -n 20 filename     # Last 20 lines
tail -f filename        # Follow file in real time (great for logs)

# Search within files
grep "pattern" filename         # Search for pattern in file
grep -i "pattern" filename      # Case-insensitive search
grep -r "pattern" /path         # Recursive search in directory
grep -n "pattern" filename      # Show line numbers
grep -v "pattern" filename      # Show lines NOT matching
grep -l "pattern" /path/*.log   # List files containing pattern

# Word count
wc filename         # Lines, words, characters
wc -l filename      # Count lines only

# Text editors
nano filename       # Simple beginner-friendly editor
vim filename        # Powerful editor (steep learning curve)

# Vim basics:
# i       = insert mode (type text)
# Esc     = return to command mode
# :w      = save
# :q      = quit
# :wq     = save and quit
# :q!     = quit without saving
# /text   = search for text
# dd      = delete current line
# yy      = copy current line
# p       = paste

# Create an empty file
touch filename

# Write to a file
echo "text" > filename      # Write (overwrites)
echo "text" >> filename     # Append to file

# Compare files
diff file1 file2
```

---

## 3. File Permissions

Linux uses a permission system with three categories and three permission types.

```
Permission format: -rwxrwxrwx
                   │└──┴──┴── other (everyone else)
                   │   └──┴── group
                   │      └── owner (user)
                   └── file type (- = file, d = directory, l = link)

r = read    (4)
w = write   (2)
x = execute (1)
```

```bash
# View permissions
ls -l filename
# Example: -rw-r--r-- 1 brady IT 4096 Aug 18 10:00 file.txt

# Change permissions (chmod)
chmod 755 filename      # rwxr-xr-x (owner=all, group=rx, other=rx)
chmod 644 filename      # rw-r--r-- (owner=rw, group=r, other=r)
chmod 600 filename      # rw------- (owner=rw only)
chmod 777 filename      # rwxrwxrwx (everyone everything — avoid)
chmod +x filename       # Add execute permission for all
chmod u+x filename      # Add execute for owner only
chmod -x filename       # Remove execute for all
chmod -R 755 /path      # Recursive permission change

# Change ownership (chown)
chown user filename             # Change owner
chown user:group filename       # Change owner and group
chown -R user:group /path       # Recursive ownership change

# Change group (chgrp)
chgrp groupname filename

# Common permission numbers
# 777 = rwxrwxrwx (dangerous — avoid)
# 755 = rwxr-xr-x (web server files, scripts)
# 644 = rw-r--r-- (regular files)
# 600 = rw------- (private keys, sensitive files)
# 400 = r-------- (read-only sensitive files)
```

---

## 4. User and Group Management

```bash
# View current user
whoami
id

# Switch user
su username             # Switch to user (needs their password)
su -                    # Switch to root
sudo command            # Run command as root
sudo -i                 # Open root shell

# User management
sudo useradd username                   # Create user
sudo useradd -m -s /bin/bash username   # Create user with home dir and bash shell
sudo passwd username                    # Set password
sudo usermod -aG groupname username     # Add user to group
sudo usermod -s /bin/bash username      # Change shell
sudo userdel username                   # Delete user
sudo userdel -r username                # Delete user and home directory

# Group management
sudo groupadd groupname             # Create group
sudo groupdel groupname             # Delete group
groups username                     # Show user's groups
cat /etc/group                      # View all groups

# View users
cat /etc/passwd                     # All users
who                                 # Currently logged in users
w                                   # Logged in users with activity
last                                # Login history
```

---

## 5. Process Management

```bash
# View running processes
ps aux                      # All processes with details
ps aux | grep processname   # Find specific process
top                         # Real-time process monitor (q to quit)
htop                        # Better process monitor (install separately)

# Process control
kill PID                    # Send SIGTERM (graceful stop) to process
kill -9 PID                 # Send SIGKILL (force kill) to process
killall processname         # Kill all processes with that name
pkill processname           # Kill process by name

# Run in background
command &                   # Run command in background
nohup command &             # Run command immune to hangup (survives logout)
jobs                        # List background jobs
fg %1                       # Bring job 1 to foreground
bg %1                       # Resume job 1 in background
Ctrl + Z                    # Suspend current foreground process
Ctrl + C                    # Kill current foreground process

# Process priority
nice -n 10 command          # Run with lower priority (nice value 10)
renice -n 5 -p PID          # Change priority of running process
```

---

## 6. Disk and Storage

```bash
# Disk usage
df -h                       # Disk space usage (human readable)
df -hT                      # Include filesystem type
du -sh /path                # Size of directory
du -sh *                    # Size of each item in current directory
du -sh /* | sort -rh        # Sorted by size

# Disk and partition info
lsblk                       # List block devices
fdisk -l                    # List partitions (requires root)
blkid                       # Show UUID and type of partitions

# Mount and unmount
mount /dev/sdb1 /mnt/disk   # Mount a partition
umount /mnt/disk            # Unmount
mount | grep sdb            # Show mounted devices

# Check filesystem
fsck /dev/sdb1              # Check and repair filesystem (unmounted)

# Create filesystem
mkfs.ext4 /dev/sdb1         # Format as ext4
mkfs.xfs /dev/sdb1          # Format as XFS

# /etc/fstab — persistent mounts
# UUID=xxxx /mnt/data ext4 defaults 0 2
# Edit with: sudo nano /etc/fstab
```

---

## 7. Networking

```bash
# IP address information
ip addr                     # View all interfaces and IPs
ip addr show eth0           # View specific interface
ifconfig                    # Legacy (may need net-tools package)

# Network connectivity
ping hostname               # Test connectivity
ping -c 4 8.8.8.8          # Ping 4 times then stop
traceroute hostname         # Trace route to destination
mtr hostname                # Combined ping and traceroute

# DNS
nslookup hostname           # DNS lookup
dig hostname                # Detailed DNS lookup
dig hostname MX             # Look up MX records
host hostname               # Simple DNS lookup

# Network connections
netstat -tuln               # View listening ports (legacy)
ss -tuln                    # View listening ports (modern)
ss -tulnp                   # Include process names

# Routing
ip route                    # View routing table
route                       # Legacy routing table
ip route add default via 192.168.1.1    # Add default route

# Firewall (UFW — Ubuntu)
sudo ufw status             # View firewall status
sudo ufw enable             # Enable firewall
sudo ufw allow 22           # Allow SSH
sudo ufw allow 80/tcp       # Allow HTTP
sudo ufw deny 23            # Block Telnet
sudo ufw delete allow 80    # Remove rule

# Firewall (firewalld — CentOS/RHEL)
sudo firewall-cmd --list-all
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload

# Download files
curl -O https://example.com/file.tar.gz     # Download file
wget https://example.com/file.tar.gz        # Download file
curl -I https://example.com                 # Get HTTP headers only
```

---

## 8. Package Management

```bash
# Ubuntu/Debian (apt)
sudo apt update                     # Update package list
sudo apt upgrade                    # Upgrade all packages
sudo apt install packagename        # Install package
sudo apt remove packagename         # Remove package
sudo apt purge packagename          # Remove package and config files
sudo apt autoremove                 # Remove unused packages
apt search packagename              # Search for package
apt show packagename                # Show package info

# CentOS/RHEL/Rocky (dnf/yum)
sudo dnf update                     # Update all packages
sudo dnf install packagename        # Install package
sudo dnf remove packagename         # Remove package
dnf search packagename              # Search
dnf info packagename                # Package info

# Check installed packages
dpkg -l | grep packagename          # Ubuntu/Debian
rpm -qa | grep packagename          # CentOS/RHEL
```

---

## 9. Archiving and Compression

```bash
# tar archives
tar -cvf archive.tar /path          # Create archive
tar -xvf archive.tar                # Extract archive
tar -czvf archive.tar.gz /path      # Create compressed archive (gzip)
tar -xzvf archive.tar.gz            # Extract gzip compressed archive
tar -cjvf archive.tar.bz2 /path     # Create compressed archive (bzip2)
tar -xjvf archive.tar.bz2           # Extract bzip2 archive
tar -tvf archive.tar                # List contents without extracting

# zip/unzip
zip -r archive.zip /path            # Create zip archive
unzip archive.zip                   # Extract zip
unzip -l archive.zip                # List contents

# gzip
gzip filename                       # Compress file (replaces original)
gunzip filename.gz                  # Decompress

# Flags reminder for tar:
# c = create
# x = extract
# v = verbose (show progress)
# f = file (specify archive name)
# z = gzip compression
# j = bzip2 compression
```

---

## 10. System Information

```bash
# System info
uname -a                    # Kernel version and system info
hostnamectl                 # Hostname and OS info
lsb_release -a              # Distribution info
cat /etc/os-release         # OS release info

# Hardware info
lscpu                       # CPU information
free -h                     # RAM usage
lsmem                       # Memory information
lspci                       # PCI devices (GPUs, NICs, etc.)
lsusb                       # USB devices
lshw                        # Complete hardware list (requires lshw)
dmidecode                   # Hardware info from BIOS

# System performance
uptime                      # How long system has been running
top                         # Real-time CPU and memory
vmstat 1 5                  # System stats every 1 second, 5 times
iostat                      # Disk I/O statistics

# System logs
journalctl                  # View systemd journal logs
journalctl -f               # Follow logs in real time
journalctl -u servicename   # Logs for specific service
journalctl --since today    # Today's logs
journalctl -p err           # Error level and above only
dmesg                       # Kernel ring buffer (hardware events)
dmesg -T                    # With human-readable timestamps
cat /var/log/syslog         # System log (Ubuntu)
cat /var/log/messages       # System log (CentOS)
```

---

## 11. Useful Shortcuts and Tips

```bash
# Keyboard shortcuts
Ctrl + C        # Kill current process
Ctrl + Z        # Suspend current process
Ctrl + D        # Exit/logout
Ctrl + L        # Clear screen (same as: clear)
Ctrl + A        # Move cursor to beginning of line
Ctrl + E        # Move cursor to end of line
Ctrl + R        # Search command history
Tab             # Autocomplete command or filename
↑ / ↓           # Scroll through command history

# Command history
history             # Show command history
history | grep ssh  # Search history
!!                  # Repeat last command
!n                  # Repeat command number n from history
!string             # Repeat last command starting with string

# Aliases
alias ll='ls -la'                   # Create shortcut
alias update='sudo apt update && sudo apt upgrade'
unalias ll                          # Remove alias
# Add to ~/.bashrc for permanent aliases

# Pipe and redirect
command | grep pattern              # Pipe output to grep
command > file.txt                  # Redirect output to file (overwrites)
command >> file.txt                 # Append output to file
command 2> error.txt                # Redirect errors to file
command &> all-output.txt           # Redirect all output to file
command1 && command2                # Run command2 only if command1 succeeds
command1 || command2                # Run command2 only if command1 fails

# Environment variables
echo $HOME                          # Print variable
echo $PATH                          # Print PATH
export MYVAR="value"                # Set variable for current session
echo 'export MYVAR="value"' >> ~/.bashrc    # Make permanent

# Run multiple commands
command1 ; command2                 # Run both regardless of outcome
command1 && command2                # Run command2 only if command1 succeeds
```

---

## Quick Reference Card

```bash
# Navigation
pwd / ls -la / cd / mkdir / rm -r / cp -r / mv / find

# Files
cat / less / head / tail -f / grep -r / nano / vim / touch / echo

# Permissions
chmod 755 / chown user:group / ls -l

# Users
whoami / sudo / useradd / passwd / usermod -aG / userdel

# Processes
ps aux / top / kill / kill -9 / jobs / nohup

# Disk
df -h / du -sh / lsblk / mount / umount

# Network
ip addr / ping / ss -tuln / curl / wget / dig

# Packages (Ubuntu)
apt update / apt install / apt remove / apt upgrade

# Archives
tar -czvf / tar -xzvf / zip -r / unzip

# System
uname -a / free -h / uptime / journalctl -f / dmesg
```

---

## Notes
- Always use `sudo` for administrative tasks — never log in as root for daily work
- `rm -rf` is irreversible — there is no recycle bin in Linux
- Use `tab` completion constantly — it prevents typos and shows available options
- `Ctrl + C` kills the current process — use this when a command hangs
- `tail -f /var/log/syslog` is invaluable for watching what's happening on the system in real time
- Use `man command` whenever you're unsure about flags or usage

---

## Related Documents
- [Systemd File Configuration](systemd-file-configuration.md)
- [Nginx Gunicorn Configuration](nginx-gunicorn-configuration.md)
- [Linux Security Basics](linux-security-basics.md)
