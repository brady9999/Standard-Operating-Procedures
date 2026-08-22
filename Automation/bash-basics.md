# Bash Basics
> A reference guide to Bash scripting for Linux system administration and automation.

**Category:** Automation  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **Bash** | Bourne Again Shell | The default shell on most Linux distributions |
| **Shell** | Shell | A program that interprets and executes commands |
| **Script** | Shell Script | A text file containing a sequence of shell commands |
| **Shebang** | Shebang | The first line `#!/bin/bash` telling the OS which interpreter to use |
| **Variable** | Variable | A named value storage location |
| **Environment Variable** | Environment Variable | A variable available to all processes in the session |
| **Exit Code** | Exit Code | A number returned by a command indicating success (0) or failure (non-zero) |
| **Pipe** | Pipe | Sends stdout of one command as stdin to another using `\|` |
| **Redirect** | Redirect | Sends output to a file using `>` or `>>` |
| **stdin** | Standard Input | Default input — usually keyboard |
| **stdout** | Standard Output | Default output — usually terminal |
| **stderr** | Standard Error | Error output stream |
| **Subshell** | Subshell | A child shell process created by `$()` or backticks |
| **Cron** | Cron | A time-based job scheduler in Linux |
| **Crontab** | Cron Table | A file defining scheduled cron jobs |
| **Heredoc** | Here Document | A multi-line string passed as stdin using `<<EOF` |

---

## Overview
Bash is the primary scripting language for Linux automation. While Python is more powerful for complex tasks, Bash is ideal for:
- System administration tasks
- File and directory operations
- Chaining Linux commands together
- Scheduled maintenance jobs
- Deployment and setup scripts

---

## 1. Script Structure

```bash
#!/bin/bash
# This is a comment

# Script: backup.sh
# Author: Brady Genik
# Description: Backs up important directories

# Exit on error (recommended for all scripts)
set -e

# Exit on undefined variable
set -u

# Show commands as they execute (for debugging)
set -x

echo "Script started"
```

```bash
# Make a script executable
chmod +x script.sh

# Run a script
./script.sh
bash script.sh

# Run with debugging
bash -x script.sh
```

---

## 2. Variables

```bash
# Assign variables (no spaces around =)
name="Brady"
age=19
is_admin=true

# Access variables
echo $name
echo "My name is $name"
echo "My name is ${name}Genik"   # Curly braces for clarity

# Command substitution — store command output in variable
current_date=$(date +%Y-%m-%d)
hostname=$(hostname)
disk_usage=$(df -h / | tail -1 | awk '{print $5}')

echo "Today is $current_date"
echo "Hostname: $hostname"
echo "Disk usage: $disk_usage"

# Read-only variables
readonly MAX_RETRIES=3

# Unset a variable
unset name

# Environment variables
echo $HOME         # Home directory
echo $USER         # Current username
echo $PATH         # Command search path
echo $PWD          # Current directory
echo $SHELL        # Current shell
echo $HOSTNAME     # Machine hostname

# Export variable (make available to child processes)
export MY_VAR="value"
```

---

## 3. Strings

```bash
# String assignment
greeting="Hello World"

# String length
echo ${#greeting}          # 11

# Substring
echo ${greeting:0:5}       # Hello (start=0, length=5)
echo ${greeting:6}         # World (start=6 to end)

# String replacement
echo ${greeting/World/Brady}    # Hello Brady (first match)
echo ${greeting//l/L}           # HeLLo WorLd (all matches)

# Remove prefix/suffix
filename="backup-2026-08-18.tar.gz"
echo ${filename#backup-}         # 2026-08-18.tar.gz (remove prefix)
echo ${filename%.tar.gz}         # backup-2026-08-18 (remove suffix)

# Uppercase/lowercase (Bash 4+)
echo ${greeting^^}    # HELLO WORLD
echo ${greeting,,}    # hello world

# Default values
echo ${unset_var:-"default"}     # Use default if unset
echo ${unset_var:="default"}     # Set and use default if unset

# Check if variable is set
echo ${name:?"Error: name is not set"}   # Error if unset

# Multiline string (heredoc)
cat <<EOF
Line 1
Line 2
Line 3
EOF

# Pass heredoc to a command
mysql -u root <<EOF
USE mydb;
SELECT * FROM users;
EOF
```

---

## 4. Arrays

```bash
# Create an array
fruits=("Apple" "Banana" "Cherry")
servers=("web01" "web02" "db01")

# Access elements
echo ${fruits[0]}          # Apple (zero-indexed)
echo ${fruits[-1]}         # Cherry (last element)
echo ${fruits[@]}          # All elements
echo ${#fruits[@]}         # Count: 3

# Add element
fruits+=("Date")

# Loop through array
for fruit in "${fruits[@]}"; do
    echo "Fruit: $fruit"
done

# Loop with index
for i in "${!fruits[@]}"; do
    echo "Index $i: ${fruits[$i]}"
done

# Associative array (dictionary)
declare -A person
person[name]="Brady"
person[age]="19"
person[city]="Winnipeg"

echo ${person[name]}       # Brady
echo ${!person[@]}         # Keys: name age city
echo ${person[@]}          # Values: Brady 19 Winnipeg
```

---

## 5. Operators and Conditionals

```bash
# Arithmetic
echo $((5 + 3))     # 8
echo $((10 - 4))    # 6
echo $((3 * 4))     # 12
echo $((10 / 3))    # 3 (integer division)
echo $((10 % 3))    # 1 (modulus)

count=5
((count++))         # Increment
((count += 3))      # Add 3

# String comparison
[ "$a" = "$b" ]     # Equal
[ "$a" != "$b" ]    # Not equal
[ -z "$a" ]         # Empty string
[ -n "$a" ]         # Non-empty string

# Numeric comparison
[ $a -eq $b ]       # Equal
[ $a -ne $b ]       # Not equal
[ $a -gt $b ]       # Greater than
[ $a -lt $b ]       # Less than
[ $a -ge $b ]       # Greater than or equal
[ $a -le $b ]       # Less than or equal

# File tests
[ -f "$file" ]      # Is a regular file
[ -d "$dir" ]       # Is a directory
[ -e "$path" ]      # Exists (file or dir)
[ -r "$file" ]      # Is readable
[ -w "$file" ]      # Is writable
[ -x "$file" ]      # Is executable
[ -s "$file" ]      # File is not empty

# Logical operators
[ $a -gt 0 ] && [ $a -lt 10 ]    # AND
[ $a -lt 0 ] || [ $a -gt 100 ]   # OR
[ ! -f "$file" ]                   # NOT

# Modern test syntax (preferred)
[[ $a == $b ]]     # Equal
[[ $a != $b ]]     # Not equal
[[ $a =~ ^[0-9]+$ ]]  # Regex match
[[ -f $file && -r $file ]]  # AND inside [[
```

---

## 6. Control Flow

```bash
# If/elif/else
age=19
if [ $age -ge 18 ]; then
    echo "Adult"
elif [ $age -ge 13 ]; then
    echo "Teenager"
else
    echo "Child"
fi

# Modern syntax
if [[ $age -ge 18 ]]; then
    echo "Adult"
fi

# One-liner
[ $age -ge 18 ] && echo "Adult" || echo "Not adult"

# Case statement
day="Monday"
case $day in
    Monday)
        echo "Start of week"
        ;;
    Friday)
        echo "End of week"
        ;;
    Saturday|Sunday)
        echo "Weekend"
        ;;
    *)
        echo "Midweek"
        ;;
esac

# For loop
for i in 1 2 3 4 5; do
    echo "Count: $i"
done

# For loop with range
for i in {1..10}; do
    echo "Count: $i"
done

# For loop with step
for i in {0..20..5}; do
    echo "Count: $i"    # 0 5 10 15 20
done

# For loop over files
for file in /var/log/*.log; do
    echo "Processing: $file"
done

# For loop over command output
for user in $(cut -d: -f1 /etc/passwd); do
    echo "User: $user"
done

# While loop
count=0
while [ $count -lt 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Until loop (runs while condition is false)
until [ $count -ge 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Break and continue
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        break       # Exit loop
    fi
    if [ $i -eq 3 ]; then
        continue    # Skip to next iteration
    fi
    echo $i
done
```

---

## 7. Functions

```bash
# Basic function
greet() {
    echo "Hello World"
}
greet    # Call the function

# Function with parameters ($1, $2, etc.)
get_user_info() {
    local username=$1      # local = only available inside function
    local domain=${2:-"company.com"}   # Default value
    echo "User: $username@$domain"
}
get_user_info "jsmith"
get_user_info "brady" "argusops.ca"

# Function with return value
# Bash functions return exit codes (0=success, non-zero=failure)
# Use echo to return actual values
get_disk_usage() {
    local drive=${1:-"/"}
    df -h $drive | tail -1 | awk '{print $5}'
}
usage=$(get_disk_usage "/")
echo "Root disk usage: $usage"

# Function with error handling
backup_file() {
    local source=$1
    local dest=$2

    if [ ! -f "$source" ]; then
        echo "Error: Source file $source does not exist" >&2
        return 1
    fi

    cp "$source" "$dest" || {
        echo "Error: Copy failed" >&2
        return 1
    }

    echo "Backed up $source to $dest"
    return 0
}

backup_file "/etc/nginx/nginx.conf" "/backup/nginx.conf"
if [ $? -eq 0 ]; then
    echo "Backup successful"
else
    echo "Backup failed"
fi
```

---

## 8. Error Handling

```bash
# Check exit code of last command
ls /nonexistent 2>/dev/null
if [ $? -ne 0 ]; then
    echo "Directory not found"
fi

# Short form using && and ||
ls /etc/nginx && echo "nginx config found" || echo "nginx not installed"

# set -e — exit script on any error
set -e
cp /source /dest    # Script exits here if this fails
echo "This won't run if copy failed"

# Trap — run code on exit or error
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/tempfile
}
trap cleanup EXIT    # Run cleanup when script exits
trap cleanup ERR     # Run cleanup on error

# Error with custom message
die() {
    echo "Error: $1" >&2
    exit 1
}

[ -f "$config_file" ] || die "Config file not found: $config_file"

# Redirect stderr to file
command 2>/tmp/errors.log

# Redirect both stdout and stderr
command &>/tmp/all-output.log

# Suppress all output
command &>/dev/null
```

---

## 9. Practical Scripts

### System Health Check
```bash
#!/bin/bash
set -e

echo "=== System Health Check ==="
echo "Date: $(date)"
echo "Hostname: $(hostname)"
echo ""

echo "--- CPU Usage ---"
top -bn1 | grep "Cpu(s)" | awk '{print "CPU Usage: " $2 "%"}'

echo ""
echo "--- Memory Usage ---"
free -h | awk '/Mem:/ {print "Total: "$2 "  Used: "$3 "  Free: "$4}'

echo ""
echo "--- Disk Usage ---"
df -h | grep -E '^/dev/' | awk '{print $1 ": " $5 " used (" $4 " free)"}'

echo ""
echo "--- Failed Services ---"
systemctl list-units --state=failed --no-legend 2>/dev/null || echo "None"

echo ""
echo "--- Recent Errors (last 20) ---"
journalctl -p err --since "1 hour ago" --no-pager -n 20 2>/dev/null || true
```

### Automated Log Rotation
```bash
#!/bin/bash

LOG_DIR="/var/log/myapp"
BACKUP_DIR="/var/log/myapp/archive"
MAX_AGE_DAYS=30

mkdir -p "$BACKUP_DIR"

# Compress logs older than 7 days
find "$LOG_DIR" -maxdepth 1 -name "*.log" -mtime +7 | while read logfile; do
    echo "Compressing: $logfile"
    gzip "$logfile"
done

# Move compressed logs to archive
find "$LOG_DIR" -maxdepth 1 -name "*.log.gz" | while read archive; do
    mv "$archive" "$BACKUP_DIR/"
done

# Delete archived logs older than MAX_AGE_DAYS
find "$BACKUP_DIR" -name "*.log.gz" -mtime +$MAX_AGE_DAYS -exec rm {} \;

echo "Log rotation complete: $(date)"
```

### Deploy Script
```bash
#!/bin/bash
set -e

APP_DIR="/opt/myapp"
BACKUP_DIR="/opt/backups/myapp"
SERVICE_NAME="myapp"
REPO_URL="git@github.com:company/myapp.git"

echo "Starting deployment: $(date)"

# Backup current version
echo "Backing up current version..."
mkdir -p "$BACKUP_DIR"
cp -r "$APP_DIR" "$BACKUP_DIR/$(date +%Y%m%d-%H%M%S)"

# Pull latest code
echo "Pulling latest code..."
cd "$APP_DIR"
git pull origin main

# Install dependencies
echo "Installing dependencies..."
source venv/bin/activate
pip install -r requirements.txt

# Run database migrations (if applicable)
echo "Running migrations..."
python manage.py migrate 2>/dev/null || true

# Restart service
echo "Restarting service..."
sudo systemctl restart "$SERVICE_NAME"

# Verify service is running
sleep 3
if systemctl is-active --quiet "$SERVICE_NAME"; then
    echo "Deployment successful: $SERVICE_NAME is running"
else
    echo "ERROR: $SERVICE_NAME failed to start" >&2
    exit 1
fi
```

---

## 10. Scheduling with Cron

```bash
# Edit crontab
crontab -e

# View current crontab
crontab -l

# Remove crontab
crontab -r

# Cron syntax:
# ┌─────────── minute (0-59)
# │ ┌───────── hour (0-23)
# │ │ ┌─────── day of month (1-31)
# │ │ │ ┌───── month (1-12)
# │ │ │ │ ┌─── day of week (0-7, 0 and 7 = Sunday)
# │ │ │ │ │
# * * * * * command

# Examples:
# Run every minute
* * * * * /opt/scripts/monitor.sh

# Run every hour
0 * * * * /opt/scripts/hourly-check.sh

# Run daily at 2 AM
0 2 * * * /opt/scripts/backup.sh

# Run every Monday at 6 AM
0 6 * * 1 /opt/scripts/weekly-report.sh

# Run first day of every month
0 0 1 * * /opt/scripts/monthly-cleanup.sh

# Run every 15 minutes
*/15 * * * * /opt/scripts/check-service.sh

# Redirect cron output to log
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
```

---

## Quick Reference

```bash
# Script header
#!/bin/bash
set -e          # Exit on error
set -u          # Exit on undefined variable

# Variables
name="value"
result=$(command)
echo "Value: $name"

# Conditionals
if [ condition ]; then ... fi
[ $a -eq $b ]   # numeric: -eq -ne -gt -lt -ge -le
[ "$a" = "$b" ] # string: = != -z -n
[ -f $file ]    # file: -f -d -e -r -w -x

# Loops
for i in {1..10}; do ... done
for item in "${array[@]}"; do ... done
while [ condition ]; do ... done

# Functions
func_name() { local var=$1; echo $var; }
result=$(func_name "arg")

# Error handling
command || { echo "Failed"; exit 1; }
trap cleanup EXIT

# Redirect
command > file.txt      # stdout to file
command >> file.txt     # append stdout
command 2> err.txt      # stderr to file
command &> all.txt      # all output to file
command 2>/dev/null     # suppress errors
```

---

## Notes
- Always use `set -e` at the top of scripts — it prevents errors from silently propagating
- Use `local` for variables inside functions — prevents polluting the global scope
- Quote your variables: `"$var"` not `$var` — prevents word splitting on spaces
- Use `[[ ]]` instead of `[ ]` in modern scripts — it's more powerful and forgiving
- Redirect stderr to `/dev/null` when you expect failures — `command 2>/dev/null`
- Test scripts with `bash -n script.sh` (syntax check) and `bash -x script.sh` (debug trace)
- Store cron output in a log file — silent failures are very hard to debug

---

## Related Documents
- [PowerShell Basics](powershell-basics.md)
- [Python Basics](python-basics.md)
- [Automation Principles](automation-principles.md)
- [Basic Linux CLI](../Linux/basic-linux-cli.md)
- [Systemd File Configuration](../Linux/systemd-file-configuration.md)
