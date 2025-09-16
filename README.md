# Sympa Installation Guide

Sympa is a free and open-source Mailing List. Mailing list management software

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
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 80/443 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80/443 (default sympa port)
- **Dependencies**:
  - perl, mysql, apache
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

# Install sympa
sudo dnf install -y sympa perl, mysql, apache

# Enable and start service
sudo systemctl enable --now sympa

# Configure firewall
sudo firewall-cmd --permanent --add-service=sympa
sudo firewall-cmd --reload

# Verify installation
sympa --version || systemctl status sympa
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install sympa
sudo apt install -y sympa perl, mysql, apache

# Enable and start service
sudo systemctl enable --now sympa

# Configure firewall
sudo ufw allow 80/443

# Verify installation
sympa --version || systemctl status sympa
```

### Arch Linux

```bash
# Install sympa
sudo pacman -S sympa

# Enable and start service
sudo systemctl enable --now sympa

# Verify installation
sympa --version || systemctl status sympa
```

### Alpine Linux

```bash
# Install sympa
apk add --no-cache sympa

# Enable and start service
rc-update add sympa default
rc-service sympa start

# Verify installation
sympa --version || rc-service sympa status
```

### openSUSE/SLES

```bash
# Install sympa
sudo zypper install -y sympa perl, mysql, apache

# Enable and start service
sudo systemctl enable --now sympa

# Configure firewall
sudo firewall-cmd --permanent --add-service=sympa
sudo firewall-cmd --reload

# Verify installation
sympa --version || systemctl status sympa
```

### macOS

```bash
# Using Homebrew
brew install sympa

# Start service
brew services start sympa

# Verify installation
sympa --version
```

### FreeBSD

```bash
# Using pkg
pkg install sympa

# Enable in rc.conf
echo 'sympa_enable="YES"' >> /etc/rc.conf

# Start service
service sympa start

# Verify installation
sympa --version || service sympa status
```

### Windows

```powershell
# Using Chocolatey
choco install sympa

# Or using Scoop
scoop install sympa

# Verify installation
sympa --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/sympa

# Set up basic configuration
sudo tee /etc/sympa/sympa.conf << 'EOF'
# Sympa Configuration
bulk_max_count 100
EOF

# Test configuration
sudo sympa -t || sudo sympa configtest

# Reload service
sudo systemctl reload sympa
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R sympa:sympa /etc/sympa
sudo chmod 750 /etc/sympa

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable sympa

# Start service
sudo systemctl start sympa

# Stop service
sudo systemctl stop sympa

# Restart service
sudo systemctl restart sympa

# Reload configuration
sudo systemctl reload sympa

# Check status
sudo systemctl status sympa

# View logs
sudo journalctl -u sympa -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add sympa default

# Start service
rc-service sympa start

# Stop service
rc-service sympa stop

# Restart service
rc-service sympa restart

# Check status
rc-service sympa status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'sympa_enable="YES"' >> /etc/rc.conf

# Start service
service sympa start

# Stop service
service sympa stop

# Restart service
service sympa restart

# Check status
service sympa status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start sympa
brew services stop sympa
brew services restart sympa

# Check status
brew services list | grep sympa
```

### Windows Service Manager

```powershell
# Start service
net start sympa

# Stop service
net stop sympa

# Using PowerShell
Start-Service sympa
Stop-Service sympa
Restart-Service sympa

# Check status
Get-Service sympa
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/sympa/sympa.conf << 'EOF'
bulk_max_count 100
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart sympa
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream sympa_backend {
    server 127.0.0.1:80/443;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name sympa.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name sympa.example.com;

    ssl_certificate /etc/ssl/certs/sympa.example.com.crt;
    ssl_certificate_key /etc/ssl/private/sympa.example.com.key;

    location / {
        proxy_pass http://sympa_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName sympa.example.com
    Redirect permanent / https://sympa.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName sympa.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/sympa.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/sympa.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/443/
    ProxyPassReverse / http://127.0.0.1:80/443/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:80/443/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend sympa_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/sympa.pem
    redirect scheme https if !{ ssl_fc }
    default_backend sympa_backend

backend sympa_backend
    balance roundrobin
    option httpchk GET /health
    server sympa1 127.0.0.1:80/443 check
    server sympa2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R sympa:sympa /etc/sympa
sudo chmod 750 /etc/sympa

# Configure firewall
sudo firewall-cmd --permanent --add-service=sympa
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/sympa.conf << 'EOF'
[sympa]
enabled = true
port = 80/443
filter = sympa
logpath = /var/log/sympa/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/sympa.key \
    -out /etc/ssl/certs/sympa.crt

# Configure SSL in sympa
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE sympa_db;
CREATE USER sympa_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE sympa_db TO sympa_user;
EOF

# Configure sympa to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE sympa_db;
CREATE USER 'sympa_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON sympa_db.* TO 'sympa_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Sympa specific tuning
bulk_max_count 100
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
sympa soft nofile 65535
sympa hard nofile 65535
sympa soft nproc 32768
sympa hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'sympa'
    static_configs:
      - targets: ['localhost:80/443']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet sympa; then
    echo "Sympa is running"
    exit 0
else
    echo "Sympa is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/sympa << 'EOF'
/var/log/sympa/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 sympa sympa
    postrotate
        systemctl reload sympa > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Sympa backup script
BACKUP_DIR="/backup/sympa"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop sympa

# Backup configuration
tar -czf "$BACKUP_DIR/sympa-config-$DATE.tar.gz" /etc/sympa

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/sympa-data-$DATE.tar.gz" /var/lib/sympa

# Start service
systemctl start sympa

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop sympa

# Restore configuration
sudo tar -xzf /backup/sympa/sympa-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/sympa/sympa-data-*.tar.gz -C /

# Set permissions
sudo chown -R sympa:sympa /etc/sympa
sudo chown -R sympa:sympa /var/lib/sympa

# Start service
sudo systemctl start sympa
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u sympa -n 100
sudo tail -f /var/log/sympa/*.log

# Check configuration
sudo sympa -t || sudo sympa configtest

# Check permissions
ls -la /etc/sympa
ls -la /var/lib/sympa
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80/443
sudo netstat -tlnp | grep 80/443

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 80/443
nc -zv localhost 80/443
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep sympa)
htop -p $(pgrep sympa)

# Check connections
ss -ant | grep :80/443 | wc -l

# Monitor I/O
iotop -p $(pgrep sympa)
```

### Debug Mode

```bash
# Run in debug mode
sudo sympa -d
# or
sudo sympa debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  sympa:
    image: sympa:latest
    container_name: sympa
    ports:
      - "80/443:80/443"
    volumes:
      - ./config:/etc/sympa
      - ./data:/var/lib/sympa
    environment:
      - sympa_CONFIG=/etc/sympa/sympa.conf
    restart: unless-stopped
    networks:
      - sympa_net

networks:
  sympa_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sympa
spec:
  replicas: 3
  selector:
    matchLabels:
      app: sympa
  template:
    metadata:
      labels:
        app: sympa
    spec:
      containers:
      - name: sympa
        image: sympa:latest
        ports:
        - containerPort: 80/443
        volumeMounts:
        - name: config
          mountPath: /etc/sympa
      volumes:
      - name: config
        configMap:
          name: sympa-config
---
apiVersion: v1
kind: Service
metadata:
  name: sympa
spec:
  selector:
    app: sympa
  ports:
  - port: 80/443
    targetPort: 80/443
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Sympa
  hosts: all
  become: yes
  tasks:
    - name: Install sympa
      package:
        name: sympa
        state: present
    
    - name: Configure sympa
      template:
        src: sympa.conf.j2
        dest: /etc/sympa/sympa.conf
        owner: sympa
        group: sympa
        mode: '0640'
      notify: restart sympa
    
    - name: Start and enable sympa
      systemd:
        name: sympa
        state: started
        enabled: yes
  
  handlers:
    - name: restart sympa
      systemd:
        name: sympa
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update sympa

# Debian/Ubuntu
sudo apt update && sudo apt upgrade sympa

# Arch Linux
sudo pacman -Syu sympa

# Alpine Linux
apk update && apk upgrade sympa

# openSUSE
sudo zypper update sympa

# FreeBSD
pkg update && pkg upgrade sympa

# Always backup before updates
tar -czf /backup/sympa-pre-update-$(date +%Y%m%d).tar.gz /etc/sympa

# Restart after updates
sudo systemctl restart sympa
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/sympa -name "*.log" -mtime +30 -delete

# Verify integrity
sudo sympa --verify || sudo sympa check

# Update databases (if applicable)
sudo sympa-update-db

# Optimize performance
sudo sympa-optimize

# Check for security updates
sudo sympa --security-check
```

## Additional Resources

- Official Documentation: https://docs.sympa.org/
- GitHub Repository: https://github.com/sympa/sympa
- Community Forum: https://forum.sympa.org/
- Wiki: https://wiki.sympa.org/
- Comparison vs Mailman, Listmonk, phpList, Listserv: https://docs.sympa.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
