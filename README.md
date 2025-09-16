# onedev Installation Guide

onedev is a free and open-source Git server with CI/CD. OneDev provides all-in-one DevOps platform with Git and CI/CD

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
  - CPU: 2+ cores
  - RAM: 4GB minimum
  - Storage: 20GB for repos
  - Network: HTTP/SSH/Git
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 6610 (default onedev port)
  - SSH on 6611
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

# Install onedev
sudo dnf install -y onedev

# Enable and start service
sudo systemctl enable --now onedev

# Configure firewall
sudo firewall-cmd --permanent --add-port=6610/tcp
sudo firewall-cmd --reload

# Verify installation
onedev --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install onedev
sudo apt install -y onedev

# Enable and start service
sudo systemctl enable --now onedev

# Configure firewall
sudo ufw allow 6610

# Verify installation
onedev --version
```

### Arch Linux

```bash
# Install onedev
sudo pacman -S onedev

# Enable and start service
sudo systemctl enable --now onedev

# Verify installation
onedev --version
```

### Alpine Linux

```bash
# Install onedev
apk add --no-cache onedev

# Enable and start service
rc-update add onedev default
rc-service onedev start

# Verify installation
onedev --version
```

### openSUSE/SLES

```bash
# Install onedev
sudo zypper install -y onedev

# Enable and start service
sudo systemctl enable --now onedev

# Configure firewall
sudo firewall-cmd --permanent --add-port=6610/tcp
sudo firewall-cmd --reload

# Verify installation
onedev --version
```

### macOS

```bash
# Using Homebrew
brew install onedev

# Start service
brew services start onedev

# Verify installation
onedev --version
```

### FreeBSD

```bash
# Using pkg
pkg install onedev

# Enable in rc.conf
echo 'onedev_enable="YES"' >> /etc/rc.conf

# Start service
service onedev start

# Verify installation
onedev --version
```

### Windows

```bash
# Using Chocolatey
choco install onedev

# Or using Scoop
scoop install onedev

# Verify installation
onedev --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/onedev

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
onedev --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable onedev

# Start service
sudo systemctl start onedev

# Stop service
sudo systemctl stop onedev

# Restart service
sudo systemctl restart onedev

# Check status
sudo systemctl status onedev

# View logs
sudo journalctl -u onedev -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add onedev default

# Start service
rc-service onedev start

# Stop service
rc-service onedev stop

# Restart service
rc-service onedev restart

# Check status
rc-service onedev status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'onedev_enable="YES"' >> /etc/rc.conf

# Start service
service onedev start

# Stop service
service onedev stop

# Restart service
service onedev restart

# Check status
service onedev status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start onedev
brew services stop onedev
brew services restart onedev

# Check status
brew services list | grep onedev
```

### Windows Service Manager

```powershell
# Start service
net start onedev

# Stop service
net stop onedev

# Using PowerShell
Start-Service onedev
Stop-Service onedev
Restart-Service onedev

# Check status
Get-Service onedev
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream onedev_backend {
    server 127.0.0.1:6610;
}

server {
    listen 80;
    server_name onedev.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name onedev.example.com;

    ssl_certificate /etc/ssl/certs/onedev.example.com.crt;
    ssl_certificate_key /etc/ssl/private/onedev.example.com.key;

    location / {
        proxy_pass http://onedev_backend;
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
    ServerName onedev.example.com
    Redirect permanent / https://onedev.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName onedev.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/onedev.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/onedev.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:6610/
    ProxyPassReverse / http://127.0.0.1:6610/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend onedev_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/onedev.pem
    redirect scheme https if !{ ssl_fc }
    default_backend onedev_backend

backend onedev_backend
    balance roundrobin
    server onedev1 127.0.0.1:6610 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R onedev:onedev /etc/onedev
sudo chmod 750 /etc/onedev

# Configure firewall
sudo firewall-cmd --permanent --add-port=6610/tcp
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
sudo systemctl status onedev

# View logs
sudo journalctl -u onedev -f

# Monitor resource usage
top -p $(pgrep onedev)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/onedev"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/onedev-backup-$DATE.tar.gz" /etc/onedev /var/lib/onedev

echo "Backup completed: $BACKUP_DIR/onedev-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop onedev

# Restore from backup
tar -xzf /backup/onedev/onedev-backup-*.tar.gz -C /

# Start service
sudo systemctl start onedev
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u onedev -n 100
sudo tail -f /var/log/onedev/onedev.log

# Check configuration
onedev --version

# Check permissions
ls -la /etc/onedev
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 6610

# Test connectivity
telnet localhost 6610

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep onedev)

# Check disk I/O
iotop -p $(pgrep onedev)

# Check connections
ss -an | grep 6610
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  onedev:
    image: onedev:latest
    ports:
      - "6610:6610"
    volumes:
      - ./config:/etc/onedev
      - ./data:/var/lib/onedev
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update onedev

# Debian/Ubuntu
sudo apt update && sudo apt upgrade onedev

# Arch Linux
sudo pacman -Syu onedev

# Alpine Linux
apk update && apk upgrade onedev

# openSUSE
sudo zypper update onedev

# FreeBSD
pkg update && pkg upgrade onedev

# Always backup before updates
tar -czf /backup/onedev-pre-update-$(date +%Y%m%d).tar.gz /etc/onedev

# Restart after updates
sudo systemctl restart onedev
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/onedev

# Clean old logs
find /var/log/onedev -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/onedev
```

## Additional Resources

- Official Documentation: https://docs.onedev.org/
- GitHub Repository: https://github.com/onedev/onedev
- Community Forum: https://forum.onedev.org/
- Best Practices Guide: https://docs.onedev.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
