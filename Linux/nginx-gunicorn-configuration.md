# Nginx and Gunicorn Configuration
> A guide to deploying Python web applications using Gunicorn as the application server and Nginx as the reverse proxy.

**Category:** Linux  
**Last Updated:** 2026-08-18  
**Author:** Brady Genik

---

## Terminology

| Term | Full Name | What It Means |
|------|-----------|---------------|
| **Nginx** | Nginx | A high-performance web server and reverse proxy |
| **Gunicorn** | Green Unicorn | A Python WSGI/ASGI HTTP server for running Python web apps |
| **WSGI** | Web Server Gateway Interface | A standard interface between Python web apps and servers |
| **ASGI** | Asynchronous Server Gateway Interface | An async version of WSGI — used by FastAPI |
| **Reverse Proxy** | Reverse Proxy | A server that sits in front of your app and forwards requests to it |
| **Upstream** | Upstream Server | The backend server Nginx forwards requests to (Gunicorn) |
| **Worker** | Gunicorn Worker | A separate process handling incoming requests |
| **Socket** | Unix Socket | A file-based communication channel between Nginx and Gunicorn |
| **SSL/TLS** | SSL/TLS | Encryption for HTTPS connections |
| **Let's Encrypt** | Let's Encrypt | A free SSL certificate authority |
| **Certbot** | Certbot | A tool for automatically obtaining Let's Encrypt certificates |
| **Virtual Host** | Server Block | Nginx configuration for a specific domain |
| **Server Block** | Server Block | Nginx's term for a virtual host configuration |
| **Location Block** | Location Block | A section in Nginx config defining behavior for specific URL paths |
| **Proxy Pass** | proxy_pass | Nginx directive forwarding requests to another server |
| **FastAPI** | FastAPI | A modern Python web framework using ASGI |
| **Uvicorn** | Uvicorn | An ASGI server — used as Gunicorn's worker class for FastAPI |

---

## Overview

**The typical production deployment stack:**

```
Internet → Nginx (port 80/443) → Gunicorn (port 8000 or Unix socket) → Python App (Flask/FastAPI/Django)
```

**Why this architecture:**
- **Nginx** handles static files, SSL termination, load balancing, and protects Gunicorn from direct internet exposure
- **Gunicorn** manages multiple Python worker processes to handle concurrent requests
- **Python app** contains your actual application logic

---

## Prerequisites
- Ubuntu/Debian server
- Python 3 installed
- Your Python application (Flask, FastAPI, or Django)
- Virtual environment with your app and Gunicorn installed
- Domain name pointing to your server (for SSL)

---

## 1. Installing Nginx

```bash
# Update package list
sudo apt update

# Install Nginx
sudo apt install nginx -y

# Start and enable Nginx
sudo systemctl enable --now nginx

# Verify Nginx is running
sudo systemctl status nginx

# Test configuration syntax
sudo nginx -t

# Reload Nginx (applies config changes without downtime)
sudo systemctl reload nginx

# Restart Nginx (full restart — brief downtime)
sudo systemctl restart nginx
```

---

## 2. Installing Gunicorn

```bash
# Create a virtual environment for your app (recommended)
python3 -m venv /opt/myapp/venv

# Activate the virtual environment
source /opt/myapp/venv/bin/activate

# Install Gunicorn
pip install gunicorn

# For FastAPI/ASGI apps — also install uvicorn
pip install uvicorn

# Deactivate virtual environment
deactivate

# Test Gunicorn manually (WSGI app — Flask/Django)
/opt/myapp/venv/bin/gunicorn --workers 4 --bind 0.0.0.0:8000 wsgi:app

# Test Gunicorn manually (ASGI app — FastAPI)
/opt/myapp/venv/bin/gunicorn --workers 4 --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000 server:app
```

---

## 3. Gunicorn Configuration

### Command Line Options

```bash
# Basic syntax
gunicorn [options] module:callable

# Where:
# module   = Python file name without .py (e.g. "server" for server.py)
# callable = The app object inside the module (e.g. "app")

# Flask example
gunicorn --workers 4 --bind 0.0.0.0:8000 app:app

# FastAPI example (requires uvicorn worker)
gunicorn --workers 4 --worker-class uvicorn.workers.UvicornWorker --bind 0.0.0.0:8000 server:app

# Bind to Unix socket (more efficient than TCP for local Nginx)
gunicorn --workers 4 --bind unix:/run/myapp/myapp.sock server:app
```

### Gunicorn Config File

```python
# /opt/myapp/gunicorn.conf.py

# Number of worker processes
# Recommended: (2 x CPU cores) + 1
workers = 4

# Worker class
# sync         = standard synchronous (Flask, Django)
# uvicorn.workers.UvicornWorker = async (FastAPI)
worker_class = "uvicorn.workers.UvicornWorker"

# Bind address
# Use Unix socket for production (Nginx communicates via socket)
bind = "unix:/run/argus/argus.sock"
# Or TCP for debugging
# bind = "0.0.0.0:8000"

# Request timeout in seconds
timeout = 120

# Maximum number of simultaneous connections per worker
worker_connections = 1000

# Restart workers after this many requests (prevents memory leaks)
max_requests = 1000
max_requests_jitter = 50

# Logging
accesslog = "/var/log/myapp/access.log"
errorlog = "/var/log/myapp/error.log"
loglevel = "info"

# PID file
pidfile = "/run/myapp/myapp.pid"

# Run as this user/group
user = "myapp"
group = "myapp"
```

```bash
# Run Gunicorn with config file
gunicorn --config /opt/myapp/gunicorn.conf.py server:app
```

### How Many Workers?

```
Recommended formula: (2 × number of CPU cores) + 1

1 CPU core  → 3 workers
2 CPU cores → 5 workers
4 CPU cores → 9 workers

Check CPU count:
nproc
```

---

## 4. Nginx Configuration — Basic Setup

Nginx configuration files live in `/etc/nginx/`. The main config is `/etc/nginx/nginx.conf` and site-specific configs go in `/etc/nginx/sites-available/`.

### File Structure

```
/etc/nginx/
├── nginx.conf                  ← Main configuration
├── sites-available/            ← Available site configs
│   └── myapp                   ← Your site config (disabled by default)
├── sites-enabled/              ← Symlinks to enabled sites
│   └── myapp → ../sites-available/myapp
├── conf.d/                     ← Additional configs (auto-loaded)
└── snippets/                   ← Reusable config snippets
```

### Basic HTTP Server Block

```nginx
# /etc/nginx/sites-available/myapp

server {
    listen 80;
    server_name example.com www.example.com;

    # Logs
    access_log /var/log/nginx/myapp-access.log;
    error_log /var/log/nginx/myapp-error.log;

    # Pass all requests to Gunicorn via Unix socket
    location / {
        proxy_pass http://unix:/run/myapp/myapp.sock;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 120s;
        proxy_connect_timeout 120s;
    }

    # Serve static files directly through Nginx (much faster)
    location /static/ {
        alias /opt/myapp/static/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # Serve media/uploads directly
    location /media/ {
        alias /opt/myapp/media/;
        expires 7d;
    }
}
```

### Connecting via TCP Instead of Socket

```nginx
# If Gunicorn is bound to TCP port 8000
location / {
    proxy_pass http://127.0.0.1:8000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

### Enabling the Site

```bash
# Create symlink to enable the site
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/

# Test Nginx configuration
sudo nginx -t

# Reload Nginx to apply changes
sudo systemctl reload nginx
```

---

## 5. SSL/HTTPS with Let's Encrypt

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx -y

# Obtain certificate and auto-configure Nginx
sudo certbot --nginx -d example.com -d www.example.com

# Certbot automatically:
# 1. Obtains SSL certificate from Let's Encrypt
# 2. Configures Nginx for HTTPS
# 3. Sets up HTTP → HTTPS redirect
# 4. Configures auto-renewal

# Test certificate renewal
sudo certbot renew --dry-run

# View certificates
sudo certbot certificates
```

### Manual HTTPS Server Block

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    # Redirect all HTTP to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    # SSL certificates (Certbot manages these)
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # SSL settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options DENY;
    add_header X-XSS-Protection "1; mode=block";

    access_log /var/log/nginx/myapp-access.log;
    error_log /var/log/nginx/myapp-error.log;

    location / {
        proxy_pass http://unix:/run/myapp/myapp.sock;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 120s;
    }

    location /static/ {
        alias /opt/myapp/static/;
        expires 30d;
    }
}
```

---

## 6. Systemd Service for Gunicorn

```bash
# Create socket directory
sudo mkdir -p /run/myapp
sudo chown myapp:myapp /run/myapp

# Create log directory
sudo mkdir -p /var/log/myapp
sudo chown myapp:myapp /var/log/myapp
```

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Python Application
After=network.target

[Service]
Type=notify
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
EnvironmentFile=/opt/myapp/.env
ExecStart=/opt/myapp/venv/bin/gunicorn \
    --config /opt/myapp/gunicorn.conf.py \
    server:app
ExecReload=/bin/kill -s HUP $MAINPID
Restart=on-failure
RestartSec=5s
KillMode=mixed
TimeoutStopSec=5
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

```bash
# Load and start the service
sudo systemctl daemon-reload
sudo systemctl enable --now myapp.service
sudo systemctl status myapp.service
```

---

## 7. Nginx Performance Tuning

```nginx
# /etc/nginx/nginx.conf — key settings to tune

http {
    # Number of worker processes (set to number of CPU cores)
    # worker_processes auto;  ← Set in the main nginx.conf context

    # Keepalive connections to upstream (Gunicorn)
    upstream myapp {
        server unix:/run/myapp/myapp.sock fail_timeout=0;
        keepalive 32;
    }

    # Gzip compression
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;
    gzip_min_length 1000;

    # Client body and timeout settings
    client_max_body_size 10M;      # Max upload size
    client_body_timeout 60s;
    client_header_timeout 60s;

    # Proxy timeouts
    proxy_read_timeout 120s;
    proxy_send_timeout 120s;
    proxy_connect_timeout 120s;

    # Buffer sizes
    proxy_buffer_size 4k;
    proxy_buffers 8 16k;
}
```

---

## 8. Troubleshooting

### Nginx Issues

```bash
# Test configuration syntax
sudo nginx -t

# View Nginx error log
sudo tail -f /var/log/nginx/error.log

# View Nginx access log
sudo tail -f /var/log/nginx/access.log

# Check Nginx status
sudo systemctl status nginx

# Check which process is using port 80
ss -tuln | grep :80
sudo lsof -i :80

# Check Nginx is listening
curl -I http://localhost
```

### Gunicorn Issues

```bash
# View service logs
journalctl -u myapp.service -f

# Test Gunicorn manually
sudo -u myapp /opt/myapp/venv/bin/gunicorn \
    --bind 0.0.0.0:8000 \
    --workers 2 \
    server:app

# Check if socket file exists
ls -la /run/myapp/myapp.sock

# Check socket permissions (Nginx needs read access)
ls -la /run/myapp/

# Check application can import correctly
sudo -u myapp /opt/myapp/venv/bin/python3 -c "from server import app; print('OK')"
```

### Common Issues & Fixes

| Problem | Cause | Fix |
|---------|-------|-----|
| 502 Bad Gateway | Gunicorn not running or socket missing | Check Gunicorn service status |
| 502 Bad Gateway | Socket permissions wrong | Nginx user needs access to socket directory |
| 504 Gateway Timeout | Gunicorn taking too long | Increase proxy_read_timeout in Nginx |
| 413 Request Too Large | File too big | Increase client_max_body_size in Nginx |
| Permission denied on socket | Nginx can't read socket | Add Nginx user to app group or fix permissions |
| Static files 404 | Wrong alias path | Verify alias path matches actual static file location |
| SSL certificate errors | Certbot not configured | Run certbot --nginx for the domain |

### Socket Permission Fix

```bash
# Add nginx user to your app group
sudo usermod -aG myapp www-data

# Or set socket permissions in gunicorn config
umask = 0o007  # Add to gunicorn.conf.py
```

---

## Quick Reference

```bash
# Nginx
sudo nginx -t                           # Test config
sudo systemctl reload nginx             # Reload config
sudo systemctl restart nginx            # Full restart
tail -f /var/log/nginx/error.log        # Watch error log

# Gunicorn (via systemd)
sudo systemctl status myapp             # Check status
sudo systemctl restart myapp            # Restart
journalctl -u myapp -f                  # Watch logs

# SSL
sudo certbot --nginx -d domain.com      # Get SSL cert
sudo certbot renew --dry-run            # Test renewal

# Site enable/disable
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/   # Enable
sudo rm /etc/nginx/sites-enabled/myapp                                   # Disable
sudo systemctl reload nginx
```

---

## Notes
- Always test Nginx config with `nginx -t` before reloading — a syntax error will prevent Nginx from reloading
- Unix sockets are faster than TCP for local Nginx-to-Gunicorn communication — use them in production
- Number of Gunicorn workers = (2 × CPU cores) + 1 is the standard recommendation
- Never expose Gunicorn directly to the internet — always put Nginx in front
- Let's Encrypt certificates expire every 90 days — Certbot sets up automatic renewal via a systemd timer
- Static files should always be served by Nginx, not Gunicorn — Nginx is much more efficient for this

---

## Related Documents
- [Systemd File Configuration](systemd-file-configuration.md)
- [Basic Linux CLI](basic-linux-cli.md)
- [Linux Security Basics](linux-security-basics.md)
