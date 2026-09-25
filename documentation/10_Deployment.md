# 10 Deployment

## Deployment Architecture

```
┌────────────────────────────────────────────────────────────┐
│                    PRODUCTION ENVIRONMENT                  │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌──────────────────────────────────────────────────┐    │
│  │        LOAD BALANCER (HAProxy/Nginx)             │    │
│  │        - Route traffic across web servers        │    │
│  │        - SSL/TLS termination                     │    │
│  │        - Session persistence                     │    │
│  └──────────────┬───────────────────────────────────┘    │
│                 │                                         │
│  ┌──────────────┴─────────────────────────────────────┐  │
│  │  WEB SERVERS (Apache + PHP) - Multiple instances │  │
│  ├──────────────────────────────────────────────────┤  │
│  │ Server 1: Application code + PHP runtime         │  │
│  │ Server 2: Application code + PHP runtime         │  │
│  │ Server 3: Application code + PHP runtime         │  │
│  └──────────────┬──────────────────────────────────┬┘  │
│                 │                                  │     │
│  ┌──────────────┴──────────────────────────────────┴─┐  │
│  │       DATABASE LAYER (MySQL Replication)         │  │
│  ├────────────────────────────────────────────────────┤  │
│  │ Primary DB: Read/Write                           │  │
│  │ Replica DB: Read-only backup                     │  │
│  │ Backup DB: Archive and disaster recovery         │  │
│  └────────────────────────────────────────────────────┘  │
│                 │                                         │
│  ┌──────────────┴──────────────────────────────────────┐ │
│  │        EXTERNAL SERVICES                          │ │
│  ├───────────────────────────────────────────────────┤ │
│  │ • Google Cloud Storage (documents)                │ │
│  │ • Google Gemini API (document analysis)           │ │
│  │ • SMTP Server (email notifications)               │ │
│  │ • Cloudflare Turnstile (bot protection)          │ │
│  └───────────────────────────────────────────────────┘ │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

## Hosting Environment

### Recommended Hosting Providers

**Option 1: Traditional VPS/Dedicated Server**
- Provider: DigitalOcean, Linode, AWS EC2, Azure VM, Google Cloud Compute
- Configuration: Ubuntu 20.04 LTS
- Specs: 2-4 vCPU, 8-16GB RAM, 100GB+ SSD
- Cost: $20-100/month

**Option 2: Platform-as-a-Service (PaaS)**
- Provider: Heroku, Platform.sh, Fortrabbit
- Advantage: Auto-scaling, auto-backups, simplified deployment
- Cost: $50-500+/month

**Option 3: Container/Kubernetes**
- Provider: Docker + Kubernetes, AWS ECS, Google GKE
- Advantage: Horizontal scaling, containerization
- Complexity: Higher DevOps required

### Recommended for Institution: Option 1 (Traditional VPS)
- **Provider**: DigitalOcean or AWS (reliable, cost-effective)
- **OS**: Ubuntu 20.04 LTS
- **Server**: Droplet/Instance with 4GB RAM, 2vCPU, 100GB SSD
- **Estimated Cost**: $40-80/month

---

## Server Setup

### Initial Server Configuration

```bash
# 1. SSH into server
ssh root@your-server-ip

# 2. Update system packages
apt-get update
apt-get upgrade -y

# 3. Set timezone
timedatectl set-timezone Asia/Manila  # For Philippines

# 4. Create non-root user (for security)
adduser deployment
usermod -aG sudo deployment

# 5. Configure firewall
ufw enable
ufw allow 22/tcp    # SSH
ufw allow 80/tcp    # HTTP
ufw allow 443/tcp   # HTTPS
ufw allow 3306/tcp  # MySQL (internal only, restrict IP)

# 6. Set hostname
hostnamectl set-hostname polytechnic-enrollment-01
```

### Install Required Software

```bash
# Install Apache
apt-get install -y apache2
a2enmod rewrite
a2enmod ssl
a2enmod headers

# Install MySQL
apt-get install -y mysql-server-8.0
mysql_secure_installation  # Remove test databases, set root password

# Install PHP 7.4
apt-get install -y php7.4 php7.4-cli php7.4-fpm php7.4-mysql
apt-get install -y php7.4-curl php7.4-gd php7.4-mbstring php7.4-xml

# Install Git
apt-get install -y git

# Install Composer
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer

# Install Certbot for SSL
apt-get install -y certbot python3-certbot-apache
```

### Apache Configuration

```bash
# Create virtual host file
cat > /etc/apache2/sites-available/polytechnic.conf << 'EOF'
<VirtualHost *:80>
    ServerName polytechnic.edu
    ServerAlias www.polytechnic.edu
    DocumentRoot /var/www/polytechnic/public_html

    <Directory /var/www/polytechnic/public_html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
        
        <IfModule mod_rewrite.c>
            RewriteEngine On
            RewriteCond %{REQUEST_FILENAME} !-f
            RewriteCond %{REQUEST_FILENAME} !-d
            RewriteRule ^(.*)$ index.php?/$1 [L]
        </IfModule>
    </Directory>

    # Deny access to sensitive directories
    <Directory /var/www/polytechnic/public_html/system>
        Deny from all
    </Directory>
    <Directory /var/www/polytechnic/public_html/application>
        Deny from all
    </Directory>

    # Error and access logs
    ErrorLog ${APACHE_LOG_DIR}/polytechnic-error.log
    CustomLog ${APACHE_LOG_DIR}/polytechnic-access.log combined

    # Security headers
    <IfModule mod_headers.c>
        Header set X-Content-Type-Options "nosniff"
        Header set X-Frame-Options "SAMEORIGIN"
        Header set X-XSS-Protection "1; mode=block"
        Header set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
    </IfModule>
</VirtualHost>

# HTTP to HTTPS redirect
<VirtualHost *:80>
    ServerName polytechnic.edu
    ServerAlias www.polytechnic.edu
    Redirect permanent / https://polytechnic.edu/
</VirtualHost>

# HTTPS configuration (after SSL certificate installed)
<VirtualHost *:443>
    ServerName polytechnic.edu
    ServerAlias www.polytechnic.edu
    DocumentRoot /var/www/polytechnic/public_html

    SSLEngine On
    SSLCertificateFile /etc/letsencrypt/live/polytechnic.edu/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/polytechnic.edu/privkey.pem
    
    # ... (rest of configuration from above)
</VirtualHost>
EOF

# Enable site
a2ensite polytechnic.conf
a2dissite 000-default.conf

# Test configuration
apache2ctl configtest  # Should say "Syntax OK"

# Reload Apache
systemctl reload apache2
```

### SSL/HTTPS Setup

```bash
# Obtain free SSL certificate from Let's Encrypt
certbot --apache -d polytechnic.edu -d www.polytechnic.edu

# Auto-renewal (certbot handles this, verify with):
systemctl list-timers snapd.service
certbot renew --dry-run

# Force HTTPS redirect
echo "Redirect permanent / https://polytechnic.edu/" >> /var/www/polytechnic/public_html/.htaccess
```

### PHP Security Configuration

```bash
# Edit PHP configuration
sudo nano /etc/php/7.4/apache2/php.ini

# Key settings:
upload_max_filesize = 10M
post_max_size = 10M
memory_limit = 256M
max_execution_time = 300
disable_functions = exec,shell_exec,passthru,system,proc_open,popen
expose_php = Off
display_errors = Off              # Off in production
log_errors = On
error_log = /var/log/php-errors.log

# Restart PHP-FPM
systemctl restart php7.4-fpm
```

---

## Database Setup

### Create Production Database & User

```sql
-- Connect as root
mysql -u root -p

-- Create database
CREATE DATABASE polytechnicportal CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Create limited user for application
CREATE USER 'enrollment'@'localhost' IDENTIFIED BY 'very_secure_password_here';

-- Grant only necessary privileges
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, ALTER, INDEX, LOCK TABLES 
ON polytechnicportal.* TO 'enrollment'@'localhost';

-- Create backup user (read-only)
CREATE USER 'enrollment_backup'@'localhost' IDENTIFIED BY 'backup_password_here';
GRANT SELECT ON polytechnicportal.* TO 'enrollment_backup'@'localhost';

FLUSH PRIVILEGES;
EXIT;

-- Import schema and initial data
mysql -u enrollment -p polytechnicportal < schema.sql
mysql -u enrollment -p polytechnicportal < initial_data.sql
```

### MySQL Configuration for Production

```bash
# Edit MySQL configuration
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf

# Key settings:
[mysqld]
# Performance
max_connections = 500
innodb_buffer_pool_size = 4G  # 50-80% of available RAM
innodb_log_file_size = 512M

# Replication (for backup server)
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_do_db = polytechnicportal

# Security
skip-external-locking
skip-name-resolve

# Restart MySQL
sudo systemctl restart mysql
```

### Database Backup Strategy

```bash
# Create backup script
cat > /usr/local/bin/backup-polytechnic.sh << 'EOF'
#!/bin/bash

BACKUP_DIR="/backups/database"
DATE=$(date +"%Y-%m-%d-%H%M%S")
BACKUP_FILE="$BACKUP_DIR/polytechnicportal-$DATE.sql.gz"

# Create backup directory if not exists
mkdir -p $BACKUP_DIR

# Backup database
mysqldump -u enrollment_backup -p'backup_password' \
    --single-transaction \
    --quick \
    polytechnicportal | gzip > $BACKUP_FILE

# Keep only last 30 days of backups
find $BACKUP_DIR -name "polytechnicportal-*.sql.gz" -mtime +30 -delete

# Optional: Upload to cloud storage
# gsutil cp $BACKUP_FILE gs://your-bucket/backups/

echo "Backup completed: $BACKUP_FILE"
EOF

# Make executable and schedule with cron
chmod +x /usr/local/bin/backup-polytechnic.sh

# Add to crontab (daily at 2 AM)
# 0 2 * * * /usr/local/bin/backup-polytechnic.sh >> /var/log/backup.log 2>&1
```

---

## Application Deployment

### Clone and Setup Application

```bash
# Navigate to web directory
cd /var/www
sudo mkdir -p polytechnic/public_html

# Clone repository
cd polytechnic/public_html
sudo git clone https://github.com/yourorg/DulagPolytechnicPortal_github.git .

# Or download as ZIP and extract

# Set permissions
sudo chown -R www-data:www-data /var/www/polytechnic
sudo chmod -R 755 /var/www/polytechnic
sudo chmod -R 777 /var/www/polytechnic/application/logs
sudo chmod -R 777 /var/www/polytechnic/application/cache
sudo chmod -R 777 /var/www/polytechnic/uploads
```

### Configure Environment Variables

```bash
# Create .env file with production settings
cat > /var/www/polytechnic/public_html/.env << 'EOF'
APP_ENV=production
APP_DEBUG=false

# Database
DB_HOST=localhost
DB_USER=enrollment
DB_PASS=very_secure_password_here
DB_NAME=polytechnicportal

# Email
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=notifications@polytechnic.edu
SMTP_PASS=app-specific-password
SENDER_EMAIL=no-reply@polytechnic.edu

# Google APIs
GOOGLE_PROJECT_ID=polytechnic-project-12345
GOOGLE_KEY_FILE=/var/www/polytechnic/config/gcs-key.json
GOOGLE_BUCKET_NAME=polytechnic-documents

# Other configuration
SESSION_TIMEOUT=1800
ENCRYPTION_KEY=very-long-random-string-32-chars-min
EOF

# Secure .env file
sudo chmod 600 /var/www/polytechnic/public_html/.env
```

### Install Dependencies

```bash
# Navigate to project root
cd /var/www/polytechnic/public_html

# Install PHP dependencies
composer install --no-dev --optimize-autoloader

# This may take a few minutes
```

### Run Database Migrations (if applicable)

```bash
# If using migration system
php index.php migrate

# Or import SQL directly
mysql -u enrollment -p polytechnicportal < database/schema.sql
```

---

## Deployment Automation

### GitHub Actions (CI/CD)

```yaml
# .github/workflows/deploy.yml

name: Deploy to Production

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Deploy to Server
      env:
        PRIVATE_KEY: ${{ secrets.DEPLOY_PRIVATE_KEY }}
        DEPLOY_HOST: ${{ secrets.DEPLOY_HOST }}
        DEPLOY_USER: ${{ secrets.DEPLOY_USER }}
      run: |
        mkdir -p ~/.ssh
        echo "$PRIVATE_KEY" > ~/.ssh/deploy_key
        chmod 600 ~/.ssh/deploy_key
        ssh-keyscan -H $DEPLOY_HOST >> ~/.ssh/known_hosts
        ssh -i ~/.ssh/deploy_key $DEPLOY_USER@$DEPLOY_HOST << 'DEPLOY'
          cd /var/www/polytechnic/public_html
          git fetch origin
          git checkout main
          git pull origin main
          composer install --no-dev --optimize-autoloader
          php index.php migrate
          sudo systemctl reload apache2
        DEPLOY
```

---

## Monitoring & Maintenance

### System Monitoring

```bash
# Install monitoring tools
apt-get install -y htop iotop nethogs
apt-get install -y prometheus node-exporter grafana

# Monitor application logs
tail -f /var/log/apache2/polytechnic-access.log
tail -f /var/log/apache2/polytechnic-error.log
tail -f /var/log/php-errors.log
tail -f /var/log/mysql/error.log
```

### Error Logging

```bash
# View application errors
cat /var/www/polytechnic/public_html/application/logs/log-*.php

# Setup centralized logging (optional)
apt-get install -y rsyslog
systemctl enable rsyslog
```

### Performance Optimization

```bash
# Enable gzip compression
a2enmod deflate
a2enmod cache
a2enmod cache_disk

# Configure opcache for PHP
cat >> /etc/php/7.4/apache2/php.ini << 'EOF'
opcache.enable=1
opcache.memory_consumption=256
opcache.max_accelerated_files=10000
opcache.validate_timestamps=0
EOF

# Enable database query caching
# Already configured in CodeIgniter config
```

---

## Domain Configuration

### DNS Records

```
Type    Name                 Value                    TTL
A       polytechnic.edu      your-server-ip          3600
A       www.polytechnic.edu  your-server-ip          3600
MX      polytechnic.edu      mail.polytechnic.edu    3600
TXT     polytechnic.edu      v=spf1 include:...      3600
CNAME   mail                 your-mail-server        3600
```

### Email Configuration

```bash
# For outgoing mail, ensure SMTP is configured in .env
# For receiving mail, configure MX records (if needed)

# Test email delivery
echo "Test message" | mail -s "Test" admin@polytechnic.edu
```

---

## Security Hardening (Production)

### Fail2Ban (Brute Force Protection)

```bash
apt-get install -y fail2ban

cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5

[sshd]
enabled = true

[apache-auth]
enabled = true

[apache-badbots]
enabled = true
EOF

systemctl restart fail2ban
```

### UFW Firewall Rules

```bash
# Already configured in setup, but verify:
ufw status verbose

# Additional: Rate limit SSH
ufw limit 22/tcp

# Allow specific IPs to admin panel (optional)
ufw allow from 203.0.113.0/24 to any port 443
```

### File Integrity Monitoring

```bash
apt-get install -y tripwire

# Configure tripwire to monitor application files
tripwire --init
tripwire --check
```

---

## Disaster Recovery

### Backup Restoration

```bash
# Restore from backup
mysql -u enrollment -p polytechnicportal < /backups/database/polytechnicportal-2024-01-15-020000.sql.gz

# Restore files from cloud storage
gsutil -m cp -r gs://your-bucket/backups/files/* /var/www/polytechnic/
```

### Failover Procedure

```bash
# If primary server fails:
1. Promote replica database to primary
2. Update application to point to new database
3. Start application on secondary server
4. Update DNS to point to secondary server IP
5. Rebuild primary server
6. Restore database replication
```

---

## Deployment Checklist

- [ ] Server provisioned and configured
- [ ] SSL certificate installed and auto-renewal configured
- [ ] Database created and backed up
- [ ] Application code deployed
- [ ] Dependencies installed (Composer)
- [ ] Environment variables configured
- [ ] File permissions set correctly
- [ ] Apache modules enabled
- [ ] PHP extensions verified
- [ ] SMTP email configured and tested
- [ ] External APIs configured and tested
- [ ] Monitoring and logging configured
- [ ] Backup system tested and scheduled
- [ ] SSL/HTTPS working and forced
- [ ] Domain DNS records configured
- [ ] Firewall rules configured
- [ ] Security scanning completed
- [ ] Load testing performed
- [ ] Rollback plan documented
- [ ] Admin notified of deployment

---

## Rollback Procedure

```bash
# If deployment has issues:

# 1. Revert code
cd /var/www/polytechnic/public_html
git revert HEAD

# 2. Reload Apache
sudo systemctl reload apache2

# 3. Check logs for errors
tail -f /var/log/apache2/polytechnic-error.log

# 4. Restore database if needed
mysql -u enrollment -p polytechnicportal < /backups/latest-good-backup.sql

# 5. Verify application is working
curl https://polytechnic.edu/welcome/login
```

---

This deployment guide provides production-ready configuration and processes for deploying and maintaining the enrollment system in a live environment.
