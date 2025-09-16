# cockpit Installation Guide

cockpit is a free and open-source web-based server management interface. Developed by Red Hat, Cockpit provides a modern web interface for Linux server administration, serving as an alternative to Webmin or proprietary control panels

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
  - Storage: 100MB for installation
  - Network: HTTPS for web interface
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9090 (default cockpit port)
  - WebSocket support required
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

# Install cockpit
sudo dnf install -y cockpit

# Enable and start service
sudo systemctl enable --now cockpit

# Configure firewall
sudo firewall-cmd --permanent --add-port=9090/tcp
sudo firewall-cmd --reload

# Verify installation
cockpit-bridge --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install cockpit
sudo apt install -y cockpit

# Enable and start service
sudo systemctl enable --now cockpit

# Configure firewall
sudo ufw allow 9090

# Verify installation
cockpit-bridge --version
```

### Arch Linux

```bash
# Install cockpit
sudo pacman -S cockpit

# Enable and start service
sudo systemctl enable --now cockpit

# Verify installation
cockpit-bridge --version
```

### Alpine Linux

```bash
# Install cockpit
apk add --no-cache cockpit

# Enable and start service
rc-update add cockpit default
rc-service cockpit start

# Verify installation
cockpit-bridge --version
```

### openSUSE/SLES

```bash
# Install cockpit
sudo zypper install -y cockpit

# Enable and start service
sudo systemctl enable --now cockpit

# Configure firewall
sudo firewall-cmd --permanent --add-port=9090/tcp
sudo firewall-cmd --reload

# Verify installation
cockpit-bridge --version
```

### macOS

```bash
# Using Homebrew
brew install cockpit

# Start service
brew services start cockpit

# Verify installation
cockpit-bridge --version
```

### FreeBSD

```bash
# Using pkg
pkg install cockpit

# Enable in rc.conf
echo 'cockpit_enable="YES"' >> /etc/rc.conf

# Start service
service cockpit start

# Verify installation
cockpit-bridge --version
```

### Windows

```bash
# Using Chocolatey
choco install cockpit

# Or using Scoop
scoop install cockpit

# Verify installation
cockpit-bridge --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/cockpit

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
cockpit-bridge --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable cockpit

# Start service
sudo systemctl start cockpit

# Stop service
sudo systemctl stop cockpit

# Restart service
sudo systemctl restart cockpit

# Check status
sudo systemctl status cockpit

# View logs
sudo journalctl -u cockpit -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add cockpit default

# Start service
rc-service cockpit start

# Stop service
rc-service cockpit stop

# Restart service
rc-service cockpit restart

# Check status
rc-service cockpit status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'cockpit_enable="YES"' >> /etc/rc.conf

# Start service
service cockpit start

# Stop service
service cockpit stop

# Restart service
service cockpit restart

# Check status
service cockpit status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start cockpit
brew services stop cockpit
brew services restart cockpit

# Check status
brew services list | grep cockpit
```

### Windows Service Manager

```powershell
# Start service
net start cockpit

# Stop service
net stop cockpit

# Using PowerShell
Start-Service cockpit
Stop-Service cockpit
Restart-Service cockpit

# Check status
Get-Service cockpit
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream cockpit_backend {
    server 127.0.0.1:9090;
}

server {
    listen 80;
    server_name cockpit.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name cockpit.example.com;

    ssl_certificate /etc/ssl/certs/cockpit.example.com.crt;
    ssl_certificate_key /etc/ssl/private/cockpit.example.com.key;

    location / {
        proxy_pass http://cockpit_backend;
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
    ServerName cockpit.example.com
    Redirect permanent / https://cockpit.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName cockpit.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/cockpit.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/cockpit.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9090/
    ProxyPassReverse / http://127.0.0.1:9090/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend cockpit_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/cockpit.pem
    redirect scheme https if !{ ssl_fc }
    default_backend cockpit_backend

backend cockpit_backend
    balance roundrobin
    server cockpit1 127.0.0.1:9090 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R cockpit:cockpit /etc/cockpit
sudo chmod 750 /etc/cockpit

# Configure firewall
sudo firewall-cmd --permanent --add-port=9090/tcp
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
sudo systemctl status cockpit

# View logs
sudo journalctl -u cockpit -f

# Monitor resource usage
top -p $(pgrep cockpit)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/cockpit"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/cockpit-backup-$DATE.tar.gz" /etc/cockpit /var/lib/cockpit

echo "Backup completed: $BACKUP_DIR/cockpit-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop cockpit

# Restore from backup
tar -xzf /backup/cockpit/cockpit-backup-*.tar.gz -C /

# Start service
sudo systemctl start cockpit
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u cockpit -n 100
sudo tail -f /var/log/cockpit/cockpit.log

# Check configuration
cockpit-bridge --version

# Check permissions
ls -la /etc/cockpit
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9090

# Test connectivity
telnet localhost 9090

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep cockpit)

# Check disk I/O
iotop -p $(pgrep cockpit)

# Check connections
ss -an | grep 9090
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  cockpit:
    image: cockpit:latest
    ports:
      - "9090:9090"
    volumes:
      - ./config:/etc/cockpit
      - ./data:/var/lib/cockpit
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update cockpit

# Debian/Ubuntu
sudo apt update && sudo apt upgrade cockpit

# Arch Linux
sudo pacman -Syu cockpit

# Alpine Linux
apk update && apk upgrade cockpit

# openSUSE
sudo zypper update cockpit

# FreeBSD
pkg update && pkg upgrade cockpit

# Always backup before updates
tar -czf /backup/cockpit-pre-update-$(date +%Y%m%d).tar.gz /etc/cockpit

# Restart after updates
sudo systemctl restart cockpit
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/cockpit

# Clean old logs
find /var/log/cockpit -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/cockpit
```

## Additional Resources

- Official Documentation: https://docs.cockpit.org/
- GitHub Repository: https://github.com/cockpit/cockpit
- Community Forum: https://forum.cockpit.org/
- Best Practices Guide: https://docs.cockpit.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
