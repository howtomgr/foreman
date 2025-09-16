# foreman Installation Guide

foreman is a free and open-source lifecycle management. Foreman provides complete lifecycle management for servers

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
  - Storage: 20GB for data
  - Network: HTTP/HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 443 (default foreman port)
  - Smart proxy on 8443
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

# Install foreman
sudo dnf install -y foreman

# Enable and start service
sudo systemctl enable --now foreman

# Configure firewall
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload

# Verify installation
foreman --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install foreman
sudo apt install -y foreman

# Enable and start service
sudo systemctl enable --now foreman

# Configure firewall
sudo ufw allow 443

# Verify installation
foreman --version
```

### Arch Linux

```bash
# Install foreman
sudo pacman -S foreman

# Enable and start service
sudo systemctl enable --now foreman

# Verify installation
foreman --version
```

### Alpine Linux

```bash
# Install foreman
apk add --no-cache foreman

# Enable and start service
rc-update add foreman default
rc-service foreman start

# Verify installation
foreman --version
```

### openSUSE/SLES

```bash
# Install foreman
sudo zypper install -y foreman

# Enable and start service
sudo systemctl enable --now foreman

# Configure firewall
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload

# Verify installation
foreman --version
```

### macOS

```bash
# Using Homebrew
brew install foreman

# Start service
brew services start foreman

# Verify installation
foreman --version
```

### FreeBSD

```bash
# Using pkg
pkg install foreman

# Enable in rc.conf
echo 'foreman_enable="YES"' >> /etc/rc.conf

# Start service
service foreman start

# Verify installation
foreman --version
```

### Windows

```bash
# Using Chocolatey
choco install foreman

# Or using Scoop
scoop install foreman

# Verify installation
foreman --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/foreman

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
foreman --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable foreman

# Start service
sudo systemctl start foreman

# Stop service
sudo systemctl stop foreman

# Restart service
sudo systemctl restart foreman

# Check status
sudo systemctl status foreman

# View logs
sudo journalctl -u foreman -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add foreman default

# Start service
rc-service foreman start

# Stop service
rc-service foreman stop

# Restart service
rc-service foreman restart

# Check status
rc-service foreman status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'foreman_enable="YES"' >> /etc/rc.conf

# Start service
service foreman start

# Stop service
service foreman stop

# Restart service
service foreman restart

# Check status
service foreman status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start foreman
brew services stop foreman
brew services restart foreman

# Check status
brew services list | grep foreman
```

### Windows Service Manager

```powershell
# Start service
net start foreman

# Stop service
net stop foreman

# Using PowerShell
Start-Service foreman
Stop-Service foreman
Restart-Service foreman

# Check status
Get-Service foreman
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream foreman_backend {
    server 127.0.0.1:443;
}

server {
    listen 80;
    server_name foreman.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name foreman.example.com;

    ssl_certificate /etc/ssl/certs/foreman.example.com.crt;
    ssl_certificate_key /etc/ssl/private/foreman.example.com.key;

    location / {
        proxy_pass http://foreman_backend;
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
    ServerName foreman.example.com
    Redirect permanent / https://foreman.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName foreman.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/foreman.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/foreman.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:443/
    ProxyPassReverse / http://127.0.0.1:443/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend foreman_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/foreman.pem
    redirect scheme https if !{ ssl_fc }
    default_backend foreman_backend

backend foreman_backend
    balance roundrobin
    server foreman1 127.0.0.1:443 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R foreman:foreman /etc/foreman
sudo chmod 750 /etc/foreman

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
sudo systemctl status foreman

# View logs
sudo journalctl -u foreman -f

# Monitor resource usage
top -p $(pgrep foreman)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/foreman"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/foreman-backup-$DATE.tar.gz" /etc/foreman /var/lib/foreman

echo "Backup completed: $BACKUP_DIR/foreman-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop foreman

# Restore from backup
tar -xzf /backup/foreman/foreman-backup-*.tar.gz -C /

# Start service
sudo systemctl start foreman
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u foreman -n 100
sudo tail -f /var/log/foreman/foreman.log

# Check configuration
foreman --version

# Check permissions
ls -la /etc/foreman
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
top -p $(pgrep foreman)

# Check disk I/O
iotop -p $(pgrep foreman)

# Check connections
ss -an | grep 443
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  foreman:
    image: foreman:latest
    ports:
      - "443:443"
    volumes:
      - ./config:/etc/foreman
      - ./data:/var/lib/foreman
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update foreman

# Debian/Ubuntu
sudo apt update && sudo apt upgrade foreman

# Arch Linux
sudo pacman -Syu foreman

# Alpine Linux
apk update && apk upgrade foreman

# openSUSE
sudo zypper update foreman

# FreeBSD
pkg update && pkg upgrade foreman

# Always backup before updates
tar -czf /backup/foreman-pre-update-$(date +%Y%m%d).tar.gz /etc/foreman

# Restart after updates
sudo systemctl restart foreman
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/foreman

# Clean old logs
find /var/log/foreman -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/foreman
```

## Additional Resources

- Official Documentation: https://docs.foreman.org/
- GitHub Repository: https://github.com/foreman/foreman
- Community Forum: https://forum.foreman.org/
- Best Practices Guide: https://docs.foreman.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
