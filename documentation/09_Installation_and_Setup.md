# 09 Installation & Setup

## System Requirements

### Minimum Hardware Requirements
- **CPU**: 2-core processor (Intel/AMD)
- **RAM**: 4GB minimum (8GB recommended for production)
- **Storage**: 50GB+ for documents and database

### Software Requirements
- **Operating System**: Windows 10+, Ubuntu 18.04+, CentOS 7+, macOS 10.12+
- **Web Server**: Apache 2.4+ with mod_rewrite
- **Database**: MySQL 5.7+ or MariaDB 10.1+
- **PHP**: 7.2+ (7.4+ recommended)
- **PHP Extensions Required**:
  - `mysqli` (MySQL database connection)
  - `gd` (Image processing for documents)
  - `curl` (API calls to Gemini and Cloud Storage)
  - `json` (JSON parsing)
  - `openssl` (HTTPS/TLS encryption)
  - `fileinfo` (File type detection)
  - `mbstring` (Multi-byte string handling)

### Internet Connectivity
- Outbound HTTPS for external APIs (Google Gemini, Google Cloud Storage)
- SMTP server access for email notifications
- Optional: CDN for static assets

---

## Local Development Setup (XAMPP)

### Step 1: Install XAMPP

1. Download XAMPP from: https://www.apachefriends.org/
2. Choose version for your OS (Windows, Linux, macOS)
3. Run installer and follow prompts
4. Install to default directory: `C:\xampp` (Windows)
5. Complete installation

### Step 2: Start Services

1. Open XAMPP Control Panel
2. Start "Apache" service
3. Start "MySQL" service
4. Verify both show green status

### Step 3: Clone/Setup Project

```bash
# Navigate to web root
cd C:\xampp\htdocs

# Clone repository (if using Git)
git clone https://github.com/yourorg/DulagPolytechnicPortal_github.git

# Or copy project files to:
C:\xampp\htdocs\DulagPolytechnicPortal_github\

# Verify directory structure:
C:\xampp\htdocs\DulagPolytechnicPortal_github\
├── application/
├── system/
├── assets/
├── uploads/
├── .htaccess
├── index.php
└── composer.json
```

### Step 4: Create Database

```sql
-- Connect to MySQL via phpMyAdmin or command line
-- URL: http://localhost/phpmyadmin

-- Create database
CREATE DATABASE polytechnicportal CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Create user (optional, for security)
CREATE USER 'enrollment_user'@'localhost' IDENTIFIED BY 'secure_password_here';
GRANT ALL PRIVILEGES ON polytechnicportal.* TO 'enrollment_user'@'localhost';
FLUSH PRIVILEGES;

-- Import schema (from documentation or backup)
-- Use phpMyAdmin to import polytechnicportal.sql
```

### Step 5: Configure Application

#### database.php Configuration

```php
// application/config/database.php

$db['default'] = array(
	'hostname' => 'localhost',      // MySQL server
	'username' => 'root',           // Change to 'enrollment_user' for production
	'password' => '',               // Add password for production
	'database' => 'polytechnicportal',
	'dbdriver' => 'mysqli',
	'char_set' => 'utf8mb4',
	'dbcollat' => 'utf8mb4_unicode_ci',
	'db_debug' => TRUE,             // FALSE in production
);
```

#### config.php Configuration

```php
// application/config/config.php

$config['base_url'] = 'http://localhost/DulagPolytechnicPortal_github/';
$config['index_page'] = '';           // Remove index.php from URLs
$config['uri_protocol'] = 'REQUEST_URI';
$config['encryption_key'] = 'generate_random_string_here'; // 32-character random string
$config['sess_cookie_name'] = 'polytechnic_session';
$config['sess_expiration'] = 1800;    // 30 minutes
$config['sess_match_ip'] = FALSE;
$config['sess_match_user_agent'] = TRUE;
$config['sess_cookie_secure'] = FALSE; // TRUE in production with HTTPS
$config['sess_cookie_httponly'] = TRUE;
$config['sess_cookie_samesite'] = 'Strict';
```

#### .htaccess Configuration

```apache
# C:\xampp\htdocs\DulagPolytechnicPortal_github\.htaccess

<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /DulagPolytechnicPortal_github/
    
    # Prevent direct access to system folder
    RewriteRule ^system/(.*) - [F,L]
    
    # Remove index.php from URL
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^(.*)$ index.php?/$1 [L]
</IfModule>
```

### Step 6: Set Permissions

```bash
# Windows (XAMPP): Usually automatic
# Linux/Mac:

chmod 755 /var/www/html/DulagPolytechnicPortal_github/
chmod 755 /var/www/html/DulagPolytechnicPortal_github/application/
chmod 777 /var/www/html/DulagPolytechnicPortal_github/application/logs/
chmod 777 /var/www/html/DulagPolytechnicPortal_github/application/cache/
chmod 777 /var/www/html/DulagPolytechnicPortal_github/uploads/
```

### Step 7: Install Composer Dependencies

```bash
# Navigate to project root
cd C:\xampp\htdocs\DulagPolytechnicPortal_github\

# Install dependencies
composer install

# This installs:
# - Google Cloud Storage client library
# - PHPMailer for email
# - Other third-party libraries
```

### Step 8: Access Application

```
URL: http://localhost/DulagPolytechnicPortal_github/

Public Pages:
- http://localhost/DulagPolytechnicPortal_github/welcome/login
- http://localhost/DulagPolytechnicPortal_github/welcome/register

Default Admin Credentials (after first setup):
- Email: admin@polytechnic.edu
- Password: TempPassword123! (Change immediately)

Demo Student Account:
- Email: student@polytechnic.edu
- Password: Student123! (Change on first login)
```

---

## Environment Variables Configuration

### Create .env File

```bash
# C:\xampp\htdocs\DulagPolytechnicPortal_github\.env

# Database
DB_HOST=localhost
DB_USER=root
DB_PASS=
DB_NAME=polytechnicportal

# Email/SMTP
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-specific-password
SENDER_EMAIL=no-reply@polytechnic.edu
SENDER_NAME=Dulag Polytechnic

# Google APIs
GOOGLE_PROJECT_ID=your-gcp-project-id
GOOGLE_KEY_FILE=/path/to/service-account-key.json
GOOGLE_BUCKET_NAME=your-bucket-name

# Gemini API
GEMINI_API_KEY=your-gemini-api-key

# Cloudflare Turnstile
TURNSTILE_SITE_KEY=your-site-key
TURNSTILE_SECRET_KEY=your-secret-key

# Application
APP_ENV=development
APP_DEBUG=true
APP_URL=http://localhost/DulagPolytechnicPortal_github/
SESSION_TIMEOUT=1800

# Security
ENCRYPTION_KEY=your-32-character-encryption-key
BCRYPT_COST=12
```

### Load Environment Variables

```php
// application/config/config.php or index.php

if (file_exists(__DIR__ . '/.env')) {
    $env_vars = parse_ini_file(__DIR__ . '/.env');
    foreach ($env_vars as $key => $value) {
        putenv("$key=$value");
    }
}

// Or use library like vlucas/phpdotenv
require_once APPPATH . 'vendor/autoload.php';
$dotenv = new \Dotenv\Dotenv(APPPATH);
$dotenv->load();
```

---

## API Configuration

### Google Cloud Storage Setup

```bash
# 1. Create GCP project
# 2. Enable Cloud Storage API
# 3. Create service account with Storage Admin role
# 4. Download JSON key file
# 5. Add to project: application/config/google-cloud-storage.json

# Install Google Cloud Storage library:
composer require google/cloud-storage
```

### Google Gemini API Setup

```bash
# 1. Go to Google AI Studio: https://makersuite.google.com/app/apikey
# 2. Create new API key
# 3. Add to .env: GEMINI_API_KEY=your-key
# 4. Test: curl https://generativelanguage.googleapis.com/v1beta/models
```

### Cloudflare Turnstile Setup

```bash
# 1. Create Cloudflare account
# 2. Go to https://dash.cloudflare.com/turnstile
# 3. Create site
# 4. Add to application config:
#    - Site Key (public)
#    - Secret Key (private)
# 5. Add keys to .env
```

### SMTP Configuration

```php
// For Gmail:
$config['protocol'] = 'smtp';
$config['smtp_host'] = 'smtp.gmail.com';
$config['smtp_port'] = 587;
$config['smtp_user'] = 'your-email@gmail.com';
$config['smtp_pass'] = 'your-app-password'; // Generate in Gmail settings
$config['smtp_crypto'] = 'tls';
$config['charset'] = 'utf-8';
$config['wordwrap'] = TRUE;

// For institutional email:
$config['smtp_host'] = 'mail.yourinstitution.edu';
$config['smtp_port'] = 587; // or 25, 465
$config['smtp_user'] = 'your-email@yourinstitution.edu';
$config['smtp_pass'] = 'your-password';
$config['smtp_crypto'] = 'tls'; // or 'ssl'
```

---

## Database Migration/Seeding

### Create Initial Schema

```sql
-- Save as: database/schema.sql
-- Import via: mysql -u root polytechnicportal < schema.sql

USE polytechnicportal;

-- Users table
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role ENUM('STUDENT', 'ADMIN', 'SUPERADMIN') NOT NULL DEFAULT 'STUDENT',
    gender ENUM('M', 'F', 'Other'),
    batch_year INT,
    is_validated TINYINT NOT NULL DEFAULT 0,
    last_known_ip VARCHAR(45),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX (email),
    INDEX (role),
    INDEX (is_validated)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- (Additional tables... see Database Design document)
```

### Create Demo/Test Data

```bash
# Create seeder script: database/seeders/DemoDataSeeder.php

php -r "
include 'database/seeders/DemoDataSeeder.php';
\$seeder = new DemoDataSeeder();
\$seeder->seed_users();
\$seeder->seed_courses();
\$seeder->seed_batches();
"
```

---

## File Uploads & Storage

### Local Storage (Development)

```bash
# Create upload directories
mkdir -p uploads/temp
mkdir -p uploads/documents
mkdir -p uploads/receipts

# Set permissions
chmod 777 uploads/
chmod 777 uploads/temp/
chmod 777 uploads/documents/
```

### Cloud Storage (Production)

```php
// Upload to Google Cloud Storage instead of local

$this->load->library('CloudStorageManager');

// Upload file
$cloud_path = $this->cloudstorage->upload_document(
    $local_file_path,
    'birth-certificates',
    $user_id
);

// Delete file
$this->cloudstorage->delete_document($cloud_path);

// Get file URL
$file_url = $this->cloudstorage->get_document_url($cloud_path);
```

---

## Security Hardening

### PHP Configuration (php.ini)

```ini
; Disable dangerous functions
disable_functions = exec,shell_exec,passthru,system,proc_open,popen,curl_exec,curl_multi_exec,parse_ini_file,show_source

; Limit file uploads
upload_max_filesize = 10M
post_max_size = 10M
max_file_uploads = 5

; Security headers
expose_php = Off

; Session security
session.use_strict_mode = 1
session.cookie_httponly = 1
session.cookie_secure = 1  ; for HTTPS
session.cookie_samesite = Strict

; Prevent XXE attacks
libxml_disable_entity_loader = 1
```

### Apache Configuration (.htaccess)

```apache
# Prevent directory listing
Options -Indexes

# Disable script execution in upload directories
<Directory "/var/www/uploads">
    php_flag engine off
    AddHandler cgi-script .php .phtml .php3 .php4 .php5 .phtml .phps
    <FilesMatch "\.ph(p[3-6]|tml)$">
        Deny from all
    </FilesMatch>
</Directory>

# Add security headers
<IfModule mod_headers.c>
    Header set X-Content-Type-Options "nosniff"
    Header set X-Frame-Options "SAMEORIGIN"
    Header set X-XSS-Protection "1; mode=block"
    Header set Referrer-Policy "strict-origin-when-cross-origin"
    Header set Content-Security-Policy "default-src 'self'"
</IfModule>

# HTTPS redirect (production)
# RewriteCond %{HTTPS} off
# RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

---

## Create Initial Admin Account

### Via Web Interface (First Time)

```
1. Navigate to http://localhost/DulagPolytechnicPortal_github/
2. Click "Create Admin Account" (only available if no admins exist)
3. Enter:
   - Email: admin@polytechnic.edu
   - Password: (must be strong, 8+ chars, mixed case, numbers, special chars)
   - Confirm Password: (same)
   - Name: System Administrator
4. Click "Create Admin"
5. Use email/password to login to admin portal
```

### Via Database (Manual)

```php
// Hash a password for manual insertion
$password = password_hash('Admin@123456', PASSWORD_BCRYPT, ['cost' => 12]);

// Insert via phpMyAdmin or command line:
INSERT INTO users (name, email, password, role, is_validated, created_at)
VALUES ('System Administrator', 'admin@polytechnic.edu', '$2y$12$...', 'SUPERADMIN', 1, NOW());
```

---

## Verification & Testing

### Test Local Installation

```bash
# 1. Check database connection
# Navigate to http://localhost/DulagPolytechnicPortal_github/welcome/login
# If page loads without error, database connection OK

# 2. Check file permissions
# Try uploading a document as student
# Should work without permission errors

# 3. Check email configuration
# Request password reset
# Should receive email within 1 minute

# 4. Test API connections
# Create a test student and identity verification
# Should see results from Gemini API
```

### Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| "Database Connection Error" | Wrong credentials in database.php | Verify hostname, username, password, database name |
| ".htaccess not working" | mod_rewrite disabled | Enable mod_rewrite in Apache config |
| "Permission Denied" file uploads | Wrong directory permissions | chmod 777 uploads/ |
| "SMTP Connection Failed" | Wrong SMTP settings | Verify SMTP server, port, credentials |
| "Google API Error" | Invalid API key or credentials | Verify API keys in .env file |
| "Blank page" | PHP error | Enable PHP error logging, check error.log |
| "Session not persisting" | Session directory not writable | chmod 777 application/cache/ |

---

## Create Demo Accounts

### Demo Student Account

```sql
-- Email: demo-student@polytechnic.edu
-- Password: DemoStudent@123

INSERT INTO users (name, email, password, role, batch_year, is_validated, created_at)
VALUES (
    'Demo Student',
    'demo-student@polytechnic.edu',
    '$2y$12$...',  -- Hash of DemoStudent@123
    'STUDENT',
    2024,
    1,  -- Already validated for testing
    NOW()
);
```

### Demo Admin Account

```sql
-- Email: demo-admin@polytechnic.edu
-- Password: DemoAdmin@123

INSERT INTO users (name, email, password, role, is_validated, created_at)
VALUES (
    'Demo Administrator',
    'demo-admin@polytechnic.edu',
    '$2y$12$...',  -- Hash of DemoAdmin@123
    'ADMIN',
    1,
    NOW()
);
```

---

## Production Deployment Checklist

- [ ] Server provisioned with recommended specifications
- [ ] SSL certificate installed (HTTPS/TLS)
- [ ] Database backup system configured
- [ ] Email/SMTP fully configured and tested
- [ ] External APIs (Gemini, Cloud Storage) configured
- [ ] File upload directory secured (outside web root if possible)
- [ ] PHP security hardened (php.ini)
- [ ] Apache security configured (.htaccess, headers)
- [ ] Database user created with minimal privileges
- [ ] Error logging configured (separate from web root)
- [ ] Session storage on persistent backend
- [ ] Automated backups scheduled
- [ ] Monitoring and alerting configured
- [ ] Load balancer configured (if multiple servers)
- [ ] CDN configured for static assets
- [ ] All API keys and secrets stored securely
- [ ] .gitignore prevents credential exposure
- [ ] Performance optimization complete
- [ ] Staging environment tested before production
- [ ] Rollback plan documented

---

This installation guide provides step-by-step instructions for setting up the system in both development and production environments. Always follow security best practices and test thoroughly before deploying to production.
