# meshcentral Installation Guide

meshcentral is a free and open-source remote management. MeshCentral provides full computer management over web

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
  - RAM: 2GB minimum
  - Storage: 10GB for data
  - Network: HTTP/WebSocket
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 443 (default meshcentral port)
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

# Install meshcentral
sudo dnf install -y meshcentral

# Enable and start service
sudo systemctl enable --now meshcentral

# Configure firewall
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload

# Verify installation
meshcentral --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install meshcentral
sudo apt install -y meshcentral

# Enable and start service
sudo systemctl enable --now meshcentral

# Configure firewall
sudo ufw allow 443

# Verify installation
meshcentral --version
```

### Arch Linux

```bash
# Install meshcentral
sudo pacman -S meshcentral

# Enable and start service
sudo systemctl enable --now meshcentral

# Verify installation
meshcentral --version
```

### Alpine Linux

```bash
# Install meshcentral
apk add --no-cache meshcentral

# Enable and start service
rc-update add meshcentral default
rc-service meshcentral start

# Verify installation
meshcentral --version
```

### openSUSE/SLES

```bash
# Install meshcentral
sudo zypper install -y meshcentral

# Enable and start service
sudo systemctl enable --now meshcentral

# Configure firewall
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload

# Verify installation
meshcentral --version
```

### macOS

```bash
# Using Homebrew
brew install meshcentral

# Start service
brew services start meshcentral

# Verify installation
meshcentral --version
```

### FreeBSD

```bash
# Using pkg
pkg install meshcentral

# Enable in rc.conf
echo 'meshcentral_enable="YES"' >> /etc/rc.conf

# Start service
service meshcentral start

# Verify installation
meshcentral --version
```

### Windows

```bash
# Using Chocolatey
choco install meshcentral

# Or using Scoop
scoop install meshcentral

# Verify installation
meshcentral --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/meshcentral

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
meshcentral --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable meshcentral

# Start service
sudo systemctl start meshcentral

# Stop service
sudo systemctl stop meshcentral

# Restart service
sudo systemctl restart meshcentral

# Check status
sudo systemctl status meshcentral

# View logs
sudo journalctl -u meshcentral -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add meshcentral default

# Start service
rc-service meshcentral start

# Stop service
rc-service meshcentral stop

# Restart service
rc-service meshcentral restart

# Check status
rc-service meshcentral status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'meshcentral_enable="YES"' >> /etc/rc.conf

# Start service
service meshcentral start

# Stop service
service meshcentral stop

# Restart service
service meshcentral restart

# Check status
service meshcentral status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start meshcentral
brew services stop meshcentral
brew services restart meshcentral

# Check status
brew services list | grep meshcentral
```

### Windows Service Manager

```powershell
# Start service
net start meshcentral

# Stop service
net stop meshcentral

# Using PowerShell
Start-Service meshcentral
Stop-Service meshcentral
Restart-Service meshcentral

# Check status
Get-Service meshcentral
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream meshcentral_backend {
    server 127.0.0.1:443;
}

server {
    listen 80;
    server_name meshcentral.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name meshcentral.example.com;

    ssl_certificate /etc/ssl/certs/meshcentral.example.com.crt;
    ssl_certificate_key /etc/ssl/private/meshcentral.example.com.key;

    location / {
        proxy_pass http://meshcentral_backend;
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
    ServerName meshcentral.example.com
    Redirect permanent / https://meshcentral.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName meshcentral.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/meshcentral.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/meshcentral.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:443/
    ProxyPassReverse / http://127.0.0.1:443/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend meshcentral_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/meshcentral.pem
    redirect scheme https if !{ ssl_fc }
    default_backend meshcentral_backend

backend meshcentral_backend
    balance roundrobin
    server meshcentral1 127.0.0.1:443 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R meshcentral:meshcentral /etc/meshcentral
sudo chmod 750 /etc/meshcentral

# Configure firewall
sudo firewall-cmd --permanent --add-port=443/tcp
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
sudo systemctl status meshcentral

# View logs
sudo journalctl -u meshcentral -f

# Monitor resource usage
top -p $(pgrep meshcentral)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/meshcentral"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/meshcentral-backup-$DATE.tar.gz" /etc/meshcentral /var/lib/meshcentral

echo "Backup completed: $BACKUP_DIR/meshcentral-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop meshcentral

# Restore from backup
tar -xzf /backup/meshcentral/meshcentral-backup-*.tar.gz -C /

# Start service
sudo systemctl start meshcentral
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u meshcentral -n 100
sudo tail -f /var/log/meshcentral/meshcentral.log

# Check configuration
meshcentral --version

# Check permissions
ls -la /etc/meshcentral
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 443

# Test connectivity
telnet localhost 443

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep meshcentral)

# Check disk I/O
iotop -p $(pgrep meshcentral)

# Check connections
ss -an | grep 443
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  meshcentral:
    image: meshcentral:latest
    ports:
      - "443:443"
    volumes:
      - ./config:/etc/meshcentral
      - ./data:/var/lib/meshcentral
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update meshcentral

# Debian/Ubuntu
sudo apt update && sudo apt upgrade meshcentral

# Arch Linux
sudo pacman -Syu meshcentral

# Alpine Linux
apk update && apk upgrade meshcentral

# openSUSE
sudo zypper update meshcentral

# FreeBSD
pkg update && pkg upgrade meshcentral

# Always backup before updates
tar -czf /backup/meshcentral-pre-update-$(date +%Y%m%d).tar.gz /etc/meshcentral

# Restart after updates
sudo systemctl restart meshcentral
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/meshcentral

# Clean old logs
find /var/log/meshcentral -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/meshcentral
```

## Additional Resources

- Official Documentation: https://docs.meshcentral.org/
- GitHub Repository: https://github.com/meshcentral/meshcentral
- Community Forum: https://forum.meshcentral.org/
- Best Practices Guide: https://docs.meshcentral.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
