# Systemd Service File Configuration
> A guide to creating, managing, and troubleshooting systemd service files on Linux.

**Category:** Linux  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **systemd** | systemd | The init system and service manager used by most modern Linux distributions |
| **Unit** | Systemd Unit | A configuration file defining a resource managed by systemd |
| **Service** | Service Unit | A unit that manages a daemon or process |
| **Target** | Target Unit | A group of units — similar to runlevels |
| **Daemon** | Daemon | A background process that runs continuously |
| **PID** | Process ID | A unique number identifying a running process |
| **Socket** | Socket Unit | A unit that manages IPC or network sockets |
| **Timer** | Timer Unit | A unit that triggers another unit on a schedule |
| **Journal** | Systemd Journal | The logging system used by systemd |
| **journalctl** | Journal Control | The command for viewing systemd logs |
| **systemctl** | System Control | The command for managing systemd units |
| **ExecStart** | ExecStart | The command that starts the service |
| **WantedBy** | WantedBy | The target that pulls this service in during startup |
| **After** | After | Specifies that this unit starts after another unit |
| **Restart** | Restart policy | When systemd should automatically restart the service |

---

## Overview
Systemd is the init system used by Ubuntu, Debian, CentOS, Fedora, and most modern Linux distributions. It manages services, mounts, timers, and more. Service files tell systemd how to start, stop, and manage your applications.

**Service file locations:**
```
/etc/systemd/system/          ← Your custom service files go here (highest priority)
/lib/systemd/system/          ← Package-installed service files
/run/systemd/system/          ← Runtime generated service files
```

**Always put your custom service files in `/etc/systemd/system/`**

---

## 1. Basic Service File Structure

A systemd service file has three main sections:

```ini
[Unit]
# Metadata and dependencies

[Service]
# How to run the service

[Install]
# How to enable the service at boot
```

### Minimal Service File Example

```ini
[Unit]
Description=My Application
After=network.target

[Service]
ExecStart=/usr/bin/python3 /opt/myapp/app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

---

## 2. [Unit] Section Options

```ini
[Unit]
# Human-readable description shown in systemctl status
Description=Argus IMS - Data Center Infrastructure Management

# Documentation URL (optional)
Documentation=https://argusops.ca/docs

# Start this service AFTER these units are started
After=network.target postgresql.service

# Require these units to be running — if they fail, this fails too
Requires=postgresql.service

# Like Requires but less strict — start after if available
Wants=network-online.target

# Fail if this condition is not met
ConditionPathExists=/opt/Argus/UMS/server.py

# Only start if this file exists
ConditionFileNotEmpty=/opt/Argus/UMS/.env
```

### Common After= Values

| Value | Meaning |
|-------|---------|
| `network.target` | Basic network is up |
| `network-online.target` | Network is fully online with connectivity |
| `postgresql.service` | PostgreSQL database is running |
| `multi-user.target` | System has reached multi-user mode |
| `syslog.target` | Syslog is available |

---

## 3. [Service] Section Options

```ini
[Service]
# The type of service
# simple    = ExecStart is the main process (default)
# forking   = Process forks and parent exits (traditional daemons)
# oneshot   = Process runs once and exits
# notify    = Like simple but sends systemd a notification when ready
# idle      = Like simple but waits until other jobs finish
Type=simple

# Command to start the service
ExecStart=/usr/bin/gunicorn -w 4 -k uvicorn.workers.UvicornWorker server:app

# Command to run before ExecStart (optional)
ExecStartPre=/bin/mkdir -p /var/run/myapp

# Command to run after ExecStart (optional)
ExecStartPost=/bin/echo "Service started"

# Command to stop the service (optional — systemd sends SIGTERM by default)
ExecStop=/bin/kill -s QUIT $MAINPID

# Command to reload the service (optional)
ExecReload=/bin/kill -s HUP $MAINPID

# Working directory for the service
WorkingDirectory=/opt/Argus/UMS

# Run as this user
User=argus

# Run as this group
Group=argus

# Environment variables
Environment="PORT=8000"
Environment="DEBUG=false"

# Load environment variables from a file
EnvironmentFile=/opt/Argus/UMS/.env

# When to restart the service
# no        = never restart
# always    = always restart
# on-failure = restart only if exit code is non-zero
# on-abnormal = restart on signal, timeout, or watchdog
Restart=on-failure

# How long to wait before restarting
RestartSec=5s

# How long to wait for the service to start before considering it failed
TimeoutStartSec=30

# How long to wait for the service to stop before killing it
TimeoutStopSec=30

# Output logging
StandardOutput=journal
StandardError=journal

# Security options (optional but recommended)
NoNewPrivileges=true
ProtectSystem=strict
PrivateTmp=true
```

---

## 4. [Install] Section Options

```ini
[Install]
# Which target pulls this service in when enabled
# multi-user.target = standard multi-user system (no GUI)
# graphical.target  = system with GUI
WantedBy=multi-user.target

# Alias name for this service (optional)
Alias=myapp.service
```

---

## 5. Real-World Examples

### Argus IMS (FastAPI + Gunicorn)

```ini
[Unit]
Description=Argus IMS - Data Center Infrastructure Management Platform
After=network.target postgresql.service
Wants=postgresql.service

[Service]
Type=simple
User=argus
Group=argus
WorkingDirectory=/opt/Argus/UMS
EnvironmentFile=/opt/Argus/UMS/.env
ExecStart=/opt/Argus/venv/bin/gunicorn \
    --workers 4 \
    --worker-class uvicorn.workers.UvicornWorker \
    --bind 0.0.0.0:8000 \
    --timeout 120 \
    server:app
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```

### Morpheus SNMP Agent

```ini
[Unit]
Description=Morpheus - Argus SNMP Polling Agent
After=network.target argus.service
Wants=argus.service

[Service]
Type=simple
User=argus
Group=argus
WorkingDirectory=/opt/Argus/Agents/Morpheus
EnvironmentFile=/opt/Argus/Agents/Morpheus/.env
ExecStart=/opt/Argus/venv/bin/python3 Morpheus.py
Restart=on-failure
RestartSec=10s
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

### Simple Python Script

```ini
[Unit]
Description=My Python Script
After=network.target

[Service]
Type=simple
User=brady
WorkingDirectory=/home/brady/scripts
ExecStart=/usr/bin/python3 /home/brady/scripts/monitor.py
Restart=always
RestartSec=30s

[Install]
WantedBy=multi-user.target
```

### Node.js Application

```ini
[Unit]
Description=My Node.js App
After=network.target

[Service]
Type=simple
User=nodeuser
WorkingDirectory=/opt/myapp
Environment=NODE_ENV=production
Environment=PORT=3000
ExecStart=/usr/bin/node /opt/myapp/server.js
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

### Systemd Timer (Scheduled Task)

```ini
# Create: /etc/systemd/system/backup.service
[Unit]
Description=Daily Backup Job

[Service]
Type=oneshot
ExecStart=/opt/scripts/backup.sh
User=backup
```

```ini
# Create: /etc/systemd/system/backup.timer
[Unit]
Description=Run backup daily at 2AM

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

---

## 6. Managing Services with systemctl

```bash
# Reload systemd after creating or editing a service file
sudo systemctl daemon-reload

# Enable service to start at boot
sudo systemctl enable servicename

# Disable service from starting at boot
sudo systemctl disable servicename

# Start a service
sudo systemctl start servicename

# Stop a service
sudo systemctl stop servicename

# Restart a service
sudo systemctl restart servicename

# Reload service configuration without stopping
sudo systemctl reload servicename

# Enable and start in one command
sudo systemctl enable --now servicename

# Check service status
sudo systemctl status servicename

# View all running services
sudo systemctl list-units --type=service --state=running

# View all enabled services
sudo systemctl list-unit-files --type=service --state=enabled

# View all failed services
sudo systemctl list-units --type=service --state=failed

# Check if a service is enabled
sudo systemctl is-enabled servicename

# Check if a service is active
sudo systemctl is-active servicename

# View service dependencies
sudo systemctl list-dependencies servicename
```

---

## 7. Viewing Logs with journalctl

```bash
# View logs for a specific service
journalctl -u servicename

# Follow logs in real time
journalctl -u servicename -f

# View logs since last boot
journalctl -u servicename -b

# View last 50 lines
journalctl -u servicename -n 50

# View logs for a time range
journalctl -u servicename --since "2026-08-18 10:00" --until "2026-08-18 11:00"

# View only errors
journalctl -u servicename -p err

# View all logs since today
journalctl --since today

# View kernel messages
journalctl -k

# Clear old logs
sudo journalctl --vacuum-time=7d      # Keep only last 7 days
sudo journalctl --vacuum-size=500M    # Keep only 500MB of logs
```

---

## 8. Creating and Installing a Service File

### Step by Step

```bash
# 1. Create the service file
sudo nano /etc/systemd/system/myapp.service

# 2. Write the service file content (see examples above)

# 3. Set correct permissions
sudo chmod 644 /etc/systemd/system/myapp.service

# 4. Reload systemd to recognize the new file
sudo systemctl daemon-reload

# 5. Enable the service to start at boot
sudo systemctl enable myapp.service

# 6. Start the service
sudo systemctl start myapp.service

# 7. Verify it's running
sudo systemctl status myapp.service

# 8. View logs to confirm no errors
journalctl -u myapp.service -f
```

---

## 9. Troubleshooting Service Issues

```bash
# Service fails to start
sudo systemctl status servicename
# Read the error in the output

# Get more detail
journalctl -u servicename -n 100

# Check for syntax errors in service file
systemd-analyze verify /etc/systemd/system/myapp.service

# Check if ExecStart path is correct
which gunicorn
which python3
ls -la /opt/myapp/server.py

# Check if the user exists
id username

# Check if environment file exists
ls -la /opt/myapp/.env

# Test the ExecStart command manually as the service user
sudo -u username /usr/bin/python3 /opt/myapp/app.py

# Check for port conflicts
ss -tuln | grep :8000

# Check file permissions
ls -la /opt/myapp/
```

### Common Service Failures

| Error | Cause | Fix |
|-------|-------|-----|
| `No such file or directory` | Wrong path in ExecStart | Verify the full path is correct |
| `Permission denied` | Wrong user or file permissions | Check User= setting and file ownership |
| `Failed to load environment file` | .env file missing or wrong path | Verify EnvironmentFile= path |
| `Address already in use` | Port already occupied | Check `ss -tuln`, stop conflicting service |
| `Exit code 1` | Application error | Check application logs in journalctl |
| `Start request repeated too quickly` | Crash loop | Fix underlying app error before re-enabling |

---

## Quick Reference

```bash
# Create service
sudo nano /etc/systemd/system/myapp.service

# Reload after changes
sudo systemctl daemon-reload

# Enable and start
sudo systemctl enable --now myapp.service

# Check status
sudo systemctl status myapp.service

# View logs
journalctl -u myapp.service -f

# Restart
sudo systemctl restart myapp.service

# Stop and disable
sudo systemctl disable --now myapp.service

# Check syntax
systemd-analyze verify /etc/systemd/system/myapp.service
```

---

## Notes
- Always run `systemctl daemon-reload` after creating or editing a service file
- Use `EnvironmentFile=` instead of hardcoding secrets in the service file
- Run services as a dedicated non-root user — never as root if avoidable
- Use `Restart=on-failure` for production services — they automatically recover from crashes
- `journalctl -u servicename -f` is your best friend for debugging service issues
- The `WorkingDirectory=` setting is important — relative paths in ExecStart are relative to this
- Use the full absolute path in ExecStart — do not rely on PATH

---

## Related Documents
- [Nginx Gunicorn Configuration](nginx-gunicorn-configuration.md)
- [Basic Linux CLI](basic-linux-cli.md)
- [Linux Security Basics](linux-security-basics.md)
