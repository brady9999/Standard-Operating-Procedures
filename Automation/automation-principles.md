# Automation Principles
> A guide to the core principles, patterns, and best practices for IT automation.

**Category:** Automation  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **Automation** | IT Automation | Using scripts or tools to perform tasks without manual intervention |
| **Idempotency** | Idempotency | Running the same automation multiple times produces the same result |
| **IaC** | Infrastructure as Code | Managing infrastructure through code rather than manual configuration |
| **CI/CD** | Continuous Integration/Continuous Deployment | Automating the build, test, and deployment pipeline |
| **Orchestration** | Orchestration | Coordinating multiple automation tasks in sequence or parallel |
| **Configuration Management** | Configuration Management | Maintaining consistent system configurations across an environment |
| **Ansible** | Ansible | An agentless configuration management and automation tool |
| **Terraform** | Terraform | An IaC tool for provisioning infrastructure |
| **Git** | Git | Version control system — essential for managing automation code |
| **YAML** | YAML Ain't Markup Language | A human-readable data format used in many automation tools |
| **API** | Application Programming Interface | A way for automation scripts to interact with systems and services |
| **Webhook** | Webhook | An HTTP callback triggered automatically by an event |
| **Cron** | Cron | Linux job scheduler for time-based automation |
| **Task Scheduler** | Task Scheduler | Windows equivalent of cron |
| **Runbook** | Runbook | A documented procedure for performing a task — the basis for automation |
| **Dry Run** | Dry Run | Running automation in preview mode without making actual changes |
| **Secret** | Secret | Sensitive values like passwords, API keys — must be handled carefully |
| **Environment Variable** | Environment Variable | A way to pass configuration to scripts without hardcoding values |

---

## Overview
Automation is the foundation of modern IT operations. The goal is to eliminate repetitive manual tasks, reduce human error, and allow IT teams to scale without proportionally increasing headcount.

**The automation mindset:**
- If you do it twice, script it
- If you do it ten times, automate it
- If it runs automatically, monitor it
- If it's automated, document it

---

## 1. Core Principles

### Idempotency
An idempotent operation produces the same result no matter how many times you run it.

```bash
# NOT idempotent — adds a line every time it runs
echo "export MY_VAR=value" >> ~/.bashrc

# Idempotent — only adds if not already present
grep -qxF 'export MY_VAR=value' ~/.bashrc || echo 'export MY_VAR=value' >> ~/.bashrc
```

```python
# NOT idempotent — creates a duplicate user every run
def create_user(username):
    subprocess.run(["useradd", username])

# Idempotent — checks first
def create_user(username):
    result = subprocess.run(["id", username], capture_output=True)
    if result.returncode != 0:  # User doesn't exist
        subprocess.run(["useradd", username], check=True)
        print(f"Created user: {username}")
    else:
        print(f"User already exists: {username}")
```

```powershell
# NOT idempotent
New-LocalUser -Name "ITUser" -Password (ConvertTo-SecureString "Pass" -AsPlainText -Force)

# Idempotent
if (-not (Get-LocalUser -Name "ITUser" -ErrorAction SilentlyContinue)) {
    New-LocalUser -Name "ITUser" -Password (ConvertTo-SecureString "Pass" -AsPlainText -Force)
    Write-Host "Created user ITUser"
} else {
    Write-Host "User ITUser already exists"
}
```

### Least Privilege
Automation scripts should run with the minimum permissions needed — not as root or Domain Admin.

```bash
# Bad — running backup script as root unnecessarily
sudo ./backup.sh

# Better — create a dedicated service account with only what it needs
sudo useradd -r -s /bin/false backup-agent
sudo chown backup-agent:backup-agent /opt/backup/
sudo -u backup-agent ./backup.sh
```

### Fail Loudly
Scripts should fail immediately when something goes wrong — not silently continue.

```bash
# Bad — continues even if a step fails
cp /source/file /dest/
process_file /dest/file
upload_file /dest/file

# Good — exits immediately on failure
set -e
cp /source/file /dest/
process_file /dest/file
upload_file /dest/file
```

```python
# Bad — swallows errors
try:
    result = risky_operation()
except:
    pass  # Never do this

# Good — handles specific errors, re-raises others
try:
    result = risky_operation()
except PermissionError as e:
    logging.error(f"Permission denied: {e}")
    raise SystemExit(1)
except FileNotFoundError as e:
    logging.error(f"File not found: {e}")
    raise SystemExit(1)
```

---

## 2. Where to Start — Automating IT Tasks

### The Automation Ladder
```
Level 1 — Document it
  Write a runbook: step-by-step instructions for the task

Level 2 — Script it
  Convert the runbook into a script
  The script still requires a human to run it

Level 3 — Schedule it
  Add the script to cron/Task Scheduler
  Runs automatically at specified times

Level 4 — Event-driven
  Script triggers based on events (webhook, file watcher, API call)
  No human involvement at all

Level 5 — Self-healing
  System detects and fixes its own problems
  Alerts only when it can't fix itself
```

### High-Value Automation Targets
```
High frequency, low complexity → Automate first:
✅ User onboarding/offboarding
✅ Password resets
✅ Log rotation and cleanup
✅ Disk space monitoring and alerting
✅ Service monitoring and restart
✅ Patch deployment
✅ Backup jobs
✅ Report generation
✅ Certificate renewal (Let's Encrypt)

High complexity, low frequency → Document first, automate later:
⚠️ Disaster recovery procedures
⚠️ Major version upgrades
⚠️ Network changes
```

---

## 3. Script Design Patterns

### Configuration Over Hardcoding
Never hardcode values that might change. Use configuration files or environment variables.

```python
# Bad — hardcoded values
def send_alert(message):
    requests.post(
        "https://hooks.slack.com/services/ABC123",
        json={"text": message}
    )

# Good — configurable
import os

SLACK_WEBHOOK = os.environ.get("SLACK_WEBHOOK")
ALERT_EMAIL = os.environ.get("ALERT_EMAIL")

def send_alert(message, method="slack"):
    if method == "slack" and SLACK_WEBHOOK:
        requests.post(SLACK_WEBHOOK, json={"text": message})
    elif method == "email" and ALERT_EMAIL:
        send_email(ALERT_EMAIL, "Alert", message)
```

### Logging
Every automation script should log what it does — you need to know what happened when something goes wrong.

```python
import logging
import sys
from datetime import datetime

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(name)s — %(message)s",
    handlers=[
        logging.FileHandler(f"/var/log/myapp/script-{datetime.now():%Y%m%d}.log"),
        logging.StreamHandler(sys.stdout)
    ]
)

logger = logging.getLogger(__name__)

def backup_database():
    logger.info("Starting database backup")
    try:
        # Do the backup
        logger.info("Backup completed successfully")
    except Exception as e:
        logger.error(f"Backup failed: {e}", exc_info=True)
        raise
```

```bash
#!/bin/bash
LOG_FILE="/var/log/backup-$(date +%Y%m%d).log"

log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') $1" | tee -a "$LOG_FILE"
}

log "INFO — Backup started"
tar -czf /backup/data.tar.gz /data >> "$LOG_FILE" 2>&1
log "INFO — Backup completed"
```

### Dry Run Mode
Always build a preview/dry-run mode so you can test what the script will do before it does it.

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--dry-run", action="store_true", help="Preview changes without applying")
args = parser.parse_args()

def delete_old_files(path, dry_run=False):
    files = find_old_files(path)
    for f in files:
        if dry_run:
            print(f"Would delete: {f}")
        else:
            os.remove(f)
            print(f"Deleted: {f}")

delete_old_files("/tmp/logs", dry_run=args.dry_run)
```

```powershell
# PowerShell built-in -WhatIf support
function Remove-OldLogs {
    [CmdletBinding(SupportsShouldProcess=$true)]
    param([string]$Path)
    $files = Get-ChildItem $Path | Where-Object {$_.LastWriteTime -lt (Get-Date).AddDays(-30)}
    foreach ($file in $files) {
        if ($PSCmdlet.ShouldProcess($file.FullName, "Delete")) {
            Remove-Item $file.FullName
        }
    }
}
Remove-OldLogs -Path "C:\Logs" -WhatIf    # Preview
Remove-OldLogs -Path "C:\Logs"            # Execute
```

---

## 4. Secret Management

Never store passwords, API keys, or tokens in scripts or source code.

### Environment Variables (Simple)
```bash
# Store in environment (not in script)
export DB_PASSWORD="secretpassword"
export API_KEY="abc123"

# Access in script
python3 script.py

# script.py
import os
db_password = os.environ.get("DB_PASSWORD")
if not db_password:
    raise ValueError("DB_PASSWORD environment variable not set")
```

### .env Files (Development)
```bash
# .env file — never commit to Git
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
DB_PASSWORD=secretpassword
API_KEY=abc123

# Load in Python with python-dotenv
from dotenv import load_dotenv
load_dotenv()
import os
db_password = os.getenv("DB_PASSWORD")
```

```bash
# .gitignore — always exclude .env files
echo ".env" >> .gitignore
echo "*.key" >> .gitignore
echo "secrets.txt" >> .gitignore
```

### Systemd Environment File (Production)
```ini
# /etc/systemd/system/myapp.service
[Service]
EnvironmentFile=/opt/myapp/.env
ExecStart=/opt/myapp/venv/bin/python3 app.py
```

```bash
# Set restrictive permissions on .env
chmod 600 /opt/myapp/.env
chown myapp:myapp /opt/myapp/.env
```

---

## 5. Version Control for Automation

All scripts must be in version control — treat automation code like production code.

```bash
# Initialize a Git repo for your scripts
mkdir ~/scripts
cd ~/scripts
git init
git config user.name "Brady Genik"
git config user.email "brady@company.com"

# Create a .gitignore
cat > .gitignore << EOF
.env
*.key
*.pem
__pycache__/
*.pyc
.DS_Store
EOF

# Add and commit scripts
git add backup.sh
git commit -m "Add backup script with log rotation"

# Push to remote
git remote add origin git@github.com:brady/scripts.git
git push -u origin main
```

### Commit Message Best Practices
```
# Format: type: short description
# Types: feat, fix, docs, style, refactor, chore

feat: add disk space monitoring script
fix: correct log rotation path on Ubuntu
docs: update README with setup instructions
chore: update backup retention to 30 days
refactor: extract email sending to shared function
```

---

## 6. Testing Automation Scripts

```bash
# Bash — test in a safe environment
# 1. Use a VM or container for destructive tests
# 2. Use set -n for syntax checking
bash -n script.sh

# 3. Use echo/print instead of real commands first
# Replace: rm -rf /tmp/old/
# With:    echo "Would delete: /tmp/old/"

# 4. Use --dry-run flags where available

# 5. ShellCheck for static analysis
# Install: apt install shellcheck
shellcheck script.sh
```

```python
# Python — use unittest or pytest
import unittest
from unittest.mock import patch, MagicMock

class TestBackupScript(unittest.TestCase):

    @patch("subprocess.run")
    def test_backup_succeeds(self, mock_run):
        mock_run.return_value = MagicMock(returncode=0)
        result = backup_database()
        self.assertTrue(result)

    @patch("subprocess.run")
    def test_backup_fails_gracefully(self, mock_run):
        mock_run.side_effect = subprocess.CalledProcessError(1, "pg_dump")
        with self.assertRaises(BackupError):
            backup_database()

# Run tests
# pytest tests/
# python -m unittest discover
```

---

## 7. Common IT Automation Patterns

### Retry Pattern
```python
import time

def retry(func, max_attempts=3, delay=5):
    """Retry a function on failure"""
    for attempt in range(1, max_attempts + 1):
        try:
            return func()
        except Exception as e:
            if attempt == max_attempts:
                raise
            print(f"Attempt {attempt} failed: {e}. Retrying in {delay}s...")
            time.sleep(delay)

# Usage
result = retry(lambda: requests.get("https://api.example.com/data", timeout=10))
```

### Notification Pattern
```python
import smtplib
import requests
import os

def notify(message, subject="Automation Alert"):
    """Send notification via email or Slack"""
    slack_webhook = os.environ.get("SLACK_WEBHOOK")
    email = os.environ.get("ALERT_EMAIL")

    if slack_webhook:
        requests.post(slack_webhook, json={"text": f"*{subject}*\n{message}"})

    if email:
        # Simple email via SMTP
        with smtplib.SMTP("smtp.company.com", 587) as server:
            server.sendmail("alerts@company.com", email,
                          f"Subject: {subject}\n\n{message}")
```

### Parallel Execution Pattern
```python
from concurrent.futures import ThreadPoolExecutor, as_completed

servers = ["server1", "server2", "server3", "server4", "server5"]

def check_server(server):
    """Check a single server"""
    import subprocess
    result = subprocess.run(["ping", "-c", "1", "-W", "2", server],
                          capture_output=True)
    return server, result.returncode == 0

# Check all servers in parallel
with ThreadPoolExecutor(max_workers=10) as executor:
    futures = {executor.submit(check_server, s): s for s in servers}
    for future in as_completed(futures):
        server, is_up = future.result()
        status = "UP" if is_up else "DOWN"
        print(f"{server}: {status}")
```

---

## 8. Documentation Standards

Every automation script should have:

```bash
#!/bin/bash
# =============================================================================
# Script: backup-vms.sh
# Author: Brady Genik
# Created: 2026-08-18
# Modified: 2026-08-18
#
# Description:
#   Backs up all Proxmox VMs to local storage and syncs to Backblaze B2.
#   Runs nightly at 2 AM via cron.
#
# Usage:
#   ./backup-vms.sh [--dry-run] [--vm-id VMID]
#
# Dependencies:
#   - vzdump (included with Proxmox)
#   - rclone (configured for B2)
#   - notify-send or curl for Slack notifications
#
# Environment Variables:
#   BACKUP_STORAGE  — Proxmox storage name (default: local)
#   B2_REMOTE       — rclone remote name (default: b2:mybackups)
#   SLACK_WEBHOOK   — Slack webhook URL for notifications
#
# Cron:
#   0 2 * * * /opt/scripts/backup-vms.sh >> /var/log/vm-backup.log 2>&1
#
# Exit Codes:
#   0 — Success
#   1 — Backup failed
#   2 — Sync to B2 failed
# =============================================================================
```

---

## 9. Automation Checklist

Before deploying any automation script to production:

```
Code Quality:
✅ Script is in version control
✅ .env / secrets excluded from Git
✅ Script passes syntax check (shellcheck, pylint)
✅ Code reviewed by at least one other person (if team environment)

Functionality:
✅ Script is idempotent — safe to run multiple times
✅ Dry run mode tested successfully
✅ Error handling tested — what happens when something fails?
✅ Script tested in staging/dev environment first

Operations:
✅ Logging configured — output goes to a log file
✅ Log rotation configured for log files
✅ Alerts configured for failures
✅ Monitoring in place for scheduled jobs

Documentation:
✅ Script has a header comment explaining what it does
✅ Usage instructions documented
✅ Dependencies listed
✅ Runbook updated to reference the automation
✅ Recovery procedure documented (what to do if automation fails)
```

---

## Quick Reference

```
Automation Decision:
→ Do it once     = Do it manually
→ Do it twice    = Write a script
→ Do it 10x      = Schedule it
→ Runs itself    = Monitor it

Key Principles:
→ Idempotent     = Same result every run
→ Fail loudly    = Error immediately, don't swallow failures
→ Least privilege = Minimum permissions needed
→ Log everything = You'll need to debug it someday
→ No hardcoding  = Use env vars or config files for values that change
→ Version control = All scripts in Git

Secret handling:
→ Never in scripts
→ Never in Git
→ Use environment variables or .env files
→ Restrict .env file permissions (chmod 600)

Testing:
→ Syntax check first
→ Dry run mode
→ Test in dev/staging
→ Test failure scenarios
```

---

## Notes
- Automation that isn't monitored is a liability — always add alerting for failures
- The best automation is automation you don't have to think about
- Document the manual process first — automation without a runbook is unmaintainable
- Start simple — a working Bash one-liner beats a complex Python framework that isn't finished
- Security applies to automation too — every script is an attack surface if credentials are mishandled
- Regularly review and retire old automation — dead scripts cause confusion and maintenance burden

---

## Related Documents
- [PowerShell Basics](powershell-basics.md)
- [Bash Basics](bash-basics.md)
- [Python Basics](python-basics.md)
- [Linux Security Basics](../Linux/linux-security-basics.md)
- [Systemd File Configuration](../Linux/systemd-file-configuration.md)
