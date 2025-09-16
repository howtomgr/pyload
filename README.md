# pyload Installation Guide

pyload is a free and open-source download manager. pyLoad provides automated downloading from one-click hosting sites

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 1 core minimum
  - RAM: 512MB minimum
  - Storage: 1GB for downloads
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 8000 (default pyload port)
  - None
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install pyload
sudo dnf install -y pyload

# Enable and start service
sudo systemctl enable --now pyload

# Configure firewall
sudo firewall-cmd --permanent --add-port=8000/tcp
sudo firewall-cmd --reload

# Verify installation
pyload --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install pyload
sudo apt install -y pyload

# Enable and start service
sudo systemctl enable --now pyload

# Configure firewall
sudo ufw allow 8000

# Verify installation
pyload --version
```

### Arch Linux

```bash
# Install pyload
sudo pacman -S pyload

# Enable and start service
sudo systemctl enable --now pyload

# Verify installation
pyload --version
```

### Alpine Linux

```bash
# Install pyload
apk add --no-cache pyload

# Enable and start service
rc-update add pyload default
rc-service pyload start

# Verify installation
pyload --version
```

### openSUSE/SLES

```bash
# Install pyload
sudo zypper install -y pyload

# Enable and start service
sudo systemctl enable --now pyload

# Configure firewall
sudo firewall-cmd --permanent --add-port=8000/tcp
sudo firewall-cmd --reload

# Verify installation
pyload --version
```

### macOS

```bash
# Using Homebrew
brew install pyload

# Start service
brew services start pyload

# Verify installation
pyload --version
```

### FreeBSD

```bash
# Using pkg
pkg install pyload

# Enable in rc.conf
echo 'pyload_enable="YES"' >> /etc/rc.conf

# Start service
service pyload start

# Verify installation
pyload --version
```

### Windows

```bash
# Using Chocolatey
choco install pyload

# Or using Scoop
scoop install pyload

# Verify installation
pyload --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/pyload

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
pyload --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable pyload

# Start service
sudo systemctl start pyload

# Stop service
sudo systemctl stop pyload

# Restart service
sudo systemctl restart pyload

# Check status
sudo systemctl status pyload

# View logs
sudo journalctl -u pyload -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add pyload default

# Start service
rc-service pyload start

# Stop service
rc-service pyload stop

# Restart service
rc-service pyload restart

# Check status
rc-service pyload status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'pyload_enable="YES"' >> /etc/rc.conf

# Start service
service pyload start

# Stop service
service pyload stop

# Restart service
service pyload restart

# Check status
service pyload status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start pyload
brew services stop pyload
brew services restart pyload

# Check status
brew services list | grep pyload
```

### Windows Service Manager

```powershell
# Start service
net start pyload

# Stop service
net stop pyload

# Using PowerShell
Start-Service pyload
Stop-Service pyload
Restart-Service pyload

# Check status
Get-Service pyload
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream pyload_backend {
    server 127.0.0.1:8000;
}

server {
    listen 80;
    server_name pyload.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name pyload.example.com;

    ssl_certificate /etc/ssl/certs/pyload.example.com.crt;
    ssl_certificate_key /etc/ssl/private/pyload.example.com.key;

    location / {
        proxy_pass http://pyload_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName pyload.example.com
    Redirect permanent / https://pyload.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName pyload.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/pyload.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/pyload.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:8000/
    ProxyPassReverse / http://127.0.0.1:8000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend pyload_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/pyload.pem
    redirect scheme https if !{ ssl_fc }
    default_backend pyload_backend

backend pyload_backend
    balance roundrobin
    server pyload1 127.0.0.1:8000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R pyload:pyload /etc/pyload
sudo chmod 750 /etc/pyload

# Configure firewall
sudo firewall-cmd --permanent --add-port=8000/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status pyload

# View logs
sudo journalctl -u pyload -f

# Monitor resource usage
top -p $(pgrep pyload)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/pyload"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/pyload-backup-$DATE.tar.gz" /etc/pyload /var/lib/pyload

echo "Backup completed: $BACKUP_DIR/pyload-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop pyload

# Restore from backup
tar -xzf /backup/pyload/pyload-backup-*.tar.gz -C /

# Start service
sudo systemctl start pyload
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u pyload -n 100
sudo tail -f /var/log/pyload/pyload.log

# Check configuration
pyload --version

# Check permissions
ls -la /etc/pyload
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 8000

# Test connectivity
telnet localhost 8000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep pyload)

# Check disk I/O
iotop -p $(pgrep pyload)

# Check connections
ss -an | grep 8000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  pyload:
    image: pyload:latest
    ports:
      - "8000:8000"
    volumes:
      - ./config:/etc/pyload
      - ./data:/var/lib/pyload
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update pyload

# Debian/Ubuntu
sudo apt update && sudo apt upgrade pyload

# Arch Linux
sudo pacman -Syu pyload

# Alpine Linux
apk update && apk upgrade pyload

# openSUSE
sudo zypper update pyload

# FreeBSD
pkg update && pkg upgrade pyload

# Always backup before updates
tar -czf /backup/pyload-pre-update-$(date +%Y%m%d).tar.gz /etc/pyload

# Restart after updates
sudo systemctl restart pyload
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/pyload

# Clean old logs
find /var/log/pyload -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/pyload
```

## Additional Resources

- Official Documentation: https://docs.pyload.org/
- GitHub Repository: https://github.com/pyload/pyload
- Community Forum: https://forum.pyload.org/
- Best Practices Guide: https://docs.pyload.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
