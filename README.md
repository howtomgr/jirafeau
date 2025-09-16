# jirafeau Installation Guide

jirafeau is a free and open-source file upload service. Jirafeau allows simple file upload and sharing with expiration dates

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
  - RAM: 128MB minimum
  - Storage: 100MB for app
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default jirafeau port)
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

# Install jirafeau
sudo dnf install -y jirafeau

# Enable and start service
sudo systemctl enable --now jirafeau

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
jirafeau --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install jirafeau
sudo apt install -y jirafeau

# Enable and start service
sudo systemctl enable --now jirafeau

# Configure firewall
sudo ufw allow 80

# Verify installation
jirafeau --version
```

### Arch Linux

```bash
# Install jirafeau
sudo pacman -S jirafeau

# Enable and start service
sudo systemctl enable --now jirafeau

# Verify installation
jirafeau --version
```

### Alpine Linux

```bash
# Install jirafeau
apk add --no-cache jirafeau

# Enable and start service
rc-update add jirafeau default
rc-service jirafeau start

# Verify installation
jirafeau --version
```

### openSUSE/SLES

```bash
# Install jirafeau
sudo zypper install -y jirafeau

# Enable and start service
sudo systemctl enable --now jirafeau

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
jirafeau --version
```

### macOS

```bash
# Using Homebrew
brew install jirafeau

# Start service
brew services start jirafeau

# Verify installation
jirafeau --version
```

### FreeBSD

```bash
# Using pkg
pkg install jirafeau

# Enable in rc.conf
echo 'jirafeau_enable="YES"' >> /etc/rc.conf

# Start service
service jirafeau start

# Verify installation
jirafeau --version
```

### Windows

```bash
# Using Chocolatey
choco install jirafeau

# Or using Scoop
scoop install jirafeau

# Verify installation
jirafeau --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/jirafeau

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
jirafeau --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable jirafeau

# Start service
sudo systemctl start jirafeau

# Stop service
sudo systemctl stop jirafeau

# Restart service
sudo systemctl restart jirafeau

# Check status
sudo systemctl status jirafeau

# View logs
sudo journalctl -u jirafeau -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add jirafeau default

# Start service
rc-service jirafeau start

# Stop service
rc-service jirafeau stop

# Restart service
rc-service jirafeau restart

# Check status
rc-service jirafeau status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'jirafeau_enable="YES"' >> /etc/rc.conf

# Start service
service jirafeau start

# Stop service
service jirafeau stop

# Restart service
service jirafeau restart

# Check status
service jirafeau status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start jirafeau
brew services stop jirafeau
brew services restart jirafeau

# Check status
brew services list | grep jirafeau
```

### Windows Service Manager

```powershell
# Start service
net start jirafeau

# Stop service
net stop jirafeau

# Using PowerShell
Start-Service jirafeau
Stop-Service jirafeau
Restart-Service jirafeau

# Check status
Get-Service jirafeau
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream jirafeau_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name jirafeau.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name jirafeau.example.com;

    ssl_certificate /etc/ssl/certs/jirafeau.example.com.crt;
    ssl_certificate_key /etc/ssl/private/jirafeau.example.com.key;

    location / {
        proxy_pass http://jirafeau_backend;
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
    ServerName jirafeau.example.com
    Redirect permanent / https://jirafeau.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName jirafeau.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/jirafeau.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/jirafeau.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend jirafeau_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/jirafeau.pem
    redirect scheme https if !{ ssl_fc }
    default_backend jirafeau_backend

backend jirafeau_backend
    balance roundrobin
    server jirafeau1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R jirafeau:jirafeau /etc/jirafeau
sudo chmod 750 /etc/jirafeau

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
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
sudo systemctl status jirafeau

# View logs
sudo journalctl -u jirafeau -f

# Monitor resource usage
top -p $(pgrep jirafeau)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/jirafeau"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/jirafeau-backup-$DATE.tar.gz" /etc/jirafeau /var/lib/jirafeau

echo "Backup completed: $BACKUP_DIR/jirafeau-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop jirafeau

# Restore from backup
tar -xzf /backup/jirafeau/jirafeau-backup-*.tar.gz -C /

# Start service
sudo systemctl start jirafeau
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u jirafeau -n 100
sudo tail -f /var/log/jirafeau/jirafeau.log

# Check configuration
jirafeau --version

# Check permissions
ls -la /etc/jirafeau
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep jirafeau)

# Check disk I/O
iotop -p $(pgrep jirafeau)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  jirafeau:
    image: jirafeau:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/jirafeau
      - ./data:/var/lib/jirafeau
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update jirafeau

# Debian/Ubuntu
sudo apt update && sudo apt upgrade jirafeau

# Arch Linux
sudo pacman -Syu jirafeau

# Alpine Linux
apk update && apk upgrade jirafeau

# openSUSE
sudo zypper update jirafeau

# FreeBSD
pkg update && pkg upgrade jirafeau

# Always backup before updates
tar -czf /backup/jirafeau-pre-update-$(date +%Y%m%d).tar.gz /etc/jirafeau

# Restart after updates
sudo systemctl restart jirafeau
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/jirafeau

# Clean old logs
find /var/log/jirafeau -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/jirafeau
```

## Additional Resources

- Official Documentation: https://docs.jirafeau.org/
- GitHub Repository: https://github.com/jirafeau/jirafeau
- Community Forum: https://forum.jirafeau.org/
- Best Practices Guide: https://docs.jirafeau.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
