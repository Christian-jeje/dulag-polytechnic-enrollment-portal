# 12 Troubleshooting

This guide documents common issues encountered during development and maintenance, their causes, and resolutions.

---

## Database Issues

### Problem: "Database Connection Error"

**Symptoms**:
- Application shows error: "Unable to connect to your database server"
- Login page fails to load
- White screen with no output

**Investigation Steps**:
1. Check database credentials in `application/config/database.php`
2. Verify MySQL is running: `systemctl status mysql` (Linux) or MySQL service in XAMPP
3. Test connection: `mysql -u root -p`
4. Check database exists: `SHOW DATABASES;`

**Root Causes & Solutions**:

| Cause | Solution |
|-------|----------|
| Wrong hostname | Use 'localhost' for local, server IP for remote |
| Wrong username/password | Verify credentials match actual database user |
| MySQL not running | Start MySQL service: `sudo systemctl start mysql` |
| Database doesn't exist | Create with: `CREATE DATABASE polytechnicportal;` |
| Network firewall blocking | Check port 3306 is accessible, add firewall rule |
| User lacks permissions | Grant privileges: `GRANT ALL ON polytechnicportal.* TO 'user'@'localhost';` |

---

### Problem: "Table Not Found" Errors

**Symptoms**:
- Error: "Table 'polytechnicportal.users' doesn't exist"
- Feature works in some operations but not others

**Investigation Steps**:
1. Connect to database: `mysql -u user -p polytechnicportal`
2. List tables: `SHOW TABLES;`
3. Check if expected tables exist

**Solutions**:
1. **Import Schema**: Restore database schema from backup
   ```bash
   mysql -u user -p polytechnicportal < schema.sql
   ```

2. **Check Capitalization**: Table names are case-sensitive on Linux
   ```sql
   -- Wrong
   SELECT * FROM Users;  -- If table is 'users'
   
   -- Correct
   SELECT * FROM users;
   ```

3. **Run Migrations**: If using migration system
   ```bash
   php index.php migrate
   ```

---

### Problem: "Incorrect Column Specification"

**Symptoms**:
- Error: "Incorrect column specification for column 'some_column'"
- Data insertion fails
- Application crashes during enrollment

**Investigation Steps**:
1. Check table structure: `DESCRIBE table_name;`
2. Verify column exists and has correct type
3. Check for typos in column names

**Solution**:
```sql
-- Add missing column
ALTER TABLE enrollments ADD COLUMN uli VARCHAR(50);

-- Fix column type
ALTER TABLE users MODIFY COLUMN batch_year INT;

-- Reset auto-increment
ALTER TABLE enrollments AUTO_INCREMENT = 1;
```

---

## File Upload Issues

### Problem: "File Upload Failed - Permission Denied"

**Symptoms**:
- Student cannot upload documents
- Error: "Permission denied"
- Uploads directory becomes full

**Investigation Steps**:
1. Check directory permissions: `ls -la uploads/`
2. Check disk space: `df -h`
3. Check file ownership: `sudo chown -R www-data:www-data uploads/`

**Solutions**:

| Issue | Solution |
|-------|----------|
| Wrong permissions | `chmod 777 uploads/` (development) or `chmod 755` (production) |
| Wrong owner | `sudo chown www-data:www-data uploads/` |
| Disk full | Delete old uploads: `rm -rf uploads/temp/*` |
| Temp directory full | Check `/tmp`: `df /tmp` |
| File too large | Check max upload size in php.ini: `upload_max_filesize` |

---

### Problem: "Invalid File Format"

**Symptoms**:
- Student tries to upload valid PDF but gets error
- File validates correctly on desktop but fails in system
- Only certain file types rejected

**Root Causes**:
1. MIME type detection failing
2. File corrupted during upload
3. File extension doesn't match actual content

**Solutions**:
```php
// In application/controllers/Student.php

// Check actual file content, not just extension
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime_type = finfo_file($finfo, $_FILES['file']['tmp_name']);
finfo_close($finfo);

// Valid MIME types
$allowed_mimes = ['application/pdf', 'image/jpeg', 'image/png'];

if (!in_array($mime_type, $allowed_mimes)) {
    $this->session->set_flashdata('error', 'Invalid file type');
    return false;
}
```

---

## Email/SMTP Issues

### Problem: "Failed to Send Email"

**Symptoms**:
- OTP emails not arriving
- Enrollment confirmation emails missing
- Error: "SMTP connection failed"

**Investigation Steps**:
1. Check SMTP configuration: `echo getenv('SMTP_HOST');`
2. Test SMTP connection: `telnet smtp.gmail.com 587`
3. Check application logs for errors
4. Verify email service (Gmail, Outlook, etc.) not blocking

**Common SMTP Solutions**:

| Provider | Host | Port | Crypto | Solution |
|----------|------|------|--------|----------|
| Gmail | smtp.gmail.com | 587 | TLS | Use app-specific password, enable "Less secure apps" |
| Outlook | smtp.office365.com | 587 | TLS | Use full email address, verify account active |
| Generic SMTP | your-mail-server | 25 or 587 | TLS/SSL | Check firewall, verify credentials |

**Testing Email**:
```php
// In controller or testing file
$this->load->library('email');
$this->email->from('test@polytechnic.edu');
$this->email->to('recipient@test.com');
$this->email->subject('Test');
$this->email->message('Test message');

if ($this->email->send()) {
    echo "Email sent successfully";
} else {
    echo $this->email->print_debugger();
}
```

---

### Problem: "Emails Going to Spam Folder"

**Symptoms**:
- Emails sent successfully but arrive in spam
- Recipients report emails missing

**Solutions**:

1. **Configure SPF Record** (DNS):
   ```
   v=spf1 include:sendgrid.net ~all
   ```

2. **Configure DKIM** (Domain Keys):
   - Verify email domain ownership
   - Add DKIM records to DNS

3. **Configure DMARC** (Email Authentication):
   ```
   v=DMARC1; p=quarantine; rua=mailto:admin@polytechnic.edu
   ```

4. **Use Proper Headers**:
   ```php
   $this->email->set_header('Reply-To', 'no-reply@polytechnic.edu');
   $this->email->set_header('X-Priority', '3');
   ```

---

## Authentication Issues

### Problem: "Session Not Persisting - Logged Out Unexpectedly"

**Symptoms**:
- Users logged out after short period
- Session expires too quickly
- "Please login again" message frequent

**Investigation Steps**:
1. Check session timeout: `grep sess_expiration application/config/config.php`
2. Check session directory: `ls -la /tmp/` (Linux) or `application/cache/`
3. Verify permissions on session directory

**Solutions**:

| Issue | Fix |
|-------|-----|
| Timeout too short | Increase `sess_expiration` in config.php (in seconds) |
| Session directory permissions | `chmod 777 /tmp` or `chmod 777 application/cache/` |
| Session directory filling up | Delete old sessions: `rm /tmp/sess_*` |
| Session storage on network | Use database for session storage for consistency |
| Cookie not accepting | Check `sess_cookie_secure` matches protocol (HTTPS) |

**Configure Database Sessions** (for persistence):
```php
// application/config/config.php
$config['sess_driver'] = 'database';
$config['sess_database_group'] = 'default';
$config['sess_table_name'] = 'ci_sessions';

// Run migration to create table
php index.php migrate
```

---

### Problem: "Cannot Reset Password - No Email Received"

**Symptoms**:
- Student requests password reset
- No email arrives
- Recovery link not working

**Investigation Steps**:
1. Check OTP table: `SELECT * FROM otp WHERE email='student@test.com';`
2. Verify OTP is not expired
3. Check email logs for delivery failure

**Solutions**:
```sql
-- Manually verify OTP in database
SELECT * FROM otp WHERE email = 'student@test.com' AND used = 0;

-- Check if expired
SELECT *, 
  CASE WHEN expires_at < NOW() THEN 'EXPIRED' ELSE 'VALID' END as status 
FROM otp WHERE email = 'student@test.com';
```

---

## API Integration Issues

### Problem: "Gemini API Analysis Failed"

**Symptoms**:
- Error: "Document analysis service temporarily unavailable"
- Birth certificate verification stuck on "pending"
- Fallback to manual review triggered often

**Investigation Steps**:
1. Verify API key: `echo getenv('GEMINI_API_KEY');`
2. Check API rate limits: Monitor API usage in Google Cloud Console
3. Test API directly: `curl https://generativelanguage.googleapis.com/v1beta/models?key=YOUR_KEY`
4. Check network connectivity

**Solutions**:

| Issue | Solution |
|-------|----------|
| Invalid API key | Regenerate key in Google Cloud Console |
| Rate limit exceeded | Wait 60 seconds, implement retry logic |
| Network timeout | Add retry logic with exponential backoff |
| API service down | Check Google Cloud status, use fallback |
| JSON parsing error | Verify response format, check for encoding issues |

**Add Retry Logic**:
```php
// application/libraries/GeminiAnalyzer.php
private function call_api_with_retry($request, $max_attempts = 3) {
    for ($attempt = 1; $attempt <= $max_attempts; $attempt++) {
        try {
            $response = $this->call_gemini_api($request);
            return $response;
        } catch (Exception $e) {
            if ($attempt < $max_attempts) {
                $wait_time = pow(2, $attempt - 1); // Exponential backoff
                sleep($wait_time);
                continue;
            } else {
                throw $e;
            }
        }
    }
}
```

---

### Problem: "Google Cloud Storage Upload Fails"

**Symptoms**:
- Error: "Unable to upload document to cloud storage"
- Documents stored locally but not backed up
- File retrieval returns 404

**Investigation Steps**:
1. Verify GCS credentials: `echo getenv('GOOGLE_KEY_FILE');`
2. Check bucket permissions: `gsutil ls gs://your-bucket/`
3. Test upload: `gsutil cp test.pdf gs://your-bucket/test/`

**Solutions**:

| Issue | Solution |
|-------|----------|
| Invalid credentials file | Download fresh key from GCP Service Accounts |
| Bucket doesn't exist | Create bucket: `gsutil mb gs://bucket-name/` |
| Insufficient permissions | Grant Storage Object Admin role to service account |
| Network timeout | Check firewall allows outbound HTTPS to googleapis.com |
| Quota exceeded | Check storage usage, delete old files if needed |

---

## Performance Issues

### Problem: "Page Loads Slowly - Takes 5+ Seconds"

**Symptoms**:
- Dashboard loads slowly
- Reports generation takes long time
- Enrollment list filtering slow

**Investigation & Diagnosis**:
```php
// Add timing debug (development only)
define('ELAPSED_TIME_START', microtime(true));

// In view
<?php 
$elapsed = microtime(true) - ELAPSED_TIME_START;
echo "<!-- Page generated in {$elapsed} seconds -->";
?>
```

**Solutions**:

| Cause | Solution |
|-------|----------|
| Missing database indices | Run: `ALTER TABLE enrollments ADD INDEX user_id_idx (user_id);` |
| N+1 query problem | Use joins instead of loops with queries |
| Large result sets | Implement pagination, limit 100 records per page |
| API calls in loop | Batch API calls or move to background jobs |
| Uncompressed assets | Enable gzip in Apache: `a2enmod deflate` |
| Query too complex | Break into multiple simpler queries |
| Slow CSS/JS parsing | Minify and compress assets |

**Identify Slow Queries**:
```sql
-- Enable query logging
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;

-- Check slow query log
tail -f /var/log/mysql/slow.log
```

---

### Problem: "Database Runs Out of Disk Space"

**Symptoms**:
- Error: "Disk full"
- New enrollments cannot be saved
- Database stops accepting connections

**Investigation**:
```bash
# Check disk usage
df -h

# Check database size
du -sh /var/lib/mysql/

# Find large tables
mysql -e "SELECT TABLE_NAME, ROUND(((data_length + index_length) / 1024 / 1024), 2) AS size 
FROM information_schema.TABLES WHERE TABLE_SCHEMA='polytechnicportal' 
ORDER BY size DESC;"
```

**Solutions**:

1. **Archive Old Data**:
   ```sql
   -- Move old enrollments to archive
   INSERT INTO enrollments_archive 
   SELECT * FROM enrollments WHERE created_at < DATE_SUB(NOW(), INTERVAL 1 YEAR);
   DELETE FROM enrollments WHERE created_at < DATE_SUB(NOW(), INTERVAL 1 YEAR);
   ```

2. **Clean Up Logs**:
   ```bash
   # Delete application logs older than 30 days
   find application/logs -name "*.php" -mtime +30 -delete
   ```

3. **Optimize Tables**:
   ```sql
   OPTIMIZE TABLE enrollments;
   OPTIMIZE TABLE users;
   ```

4. **Expand Storage**:
   - Add more disk space to server
   - Migrate to larger storage solution

---

## Security Issues

### Problem: "Possible SQL Injection Attack Detected"

**Symptoms**:
- Error logged: "SQL injection attempt"
- Unusual database queries in logs
- Suspicious user activity

**Investigation**:
```sql
-- Check activity log for suspicious patterns
SELECT * FROM activity_log 
WHERE resource = 'user' 
AND details LIKE '%union%' OR details LIKE '%drop%'
ORDER BY created_at DESC;
```

**Prevention** (already implemented):
- All queries use prepared statements
- Input validation on all endpoints
- CSRF tokens on all forms
- Output escaping on all user data

**Response**:
1. Review activity logs
2. Check for unauthorized data modifications
3. Notify affected users if data compromised
4. Reset affected user accounts
5. Enable additional logging

---

### Problem: "Account Hacked - Password Changed"

**Symptoms**:
- User reports not recognizing recent activities
- Unauthorized enrollment changes
- Suspicious IP addresses in logs

**Investigation**:
```sql
-- Check user activity
SELECT * FROM activity_log 
WHERE user_id = 123 
ORDER BY created_at DESC LIMIT 20;

-- Check login history
SELECT user_id, ip_address, created_at FROM activity_log 
WHERE action = 'LOGIN' AND user_id = 123 
ORDER BY created_at DESC;
```

**Resolution**:
1. **Force Password Reset**:
   ```php
   // Admin force user to change password
   UPDATE users SET password = null WHERE id = 123;
   // User must reset password on next login
   ```

2. **Review Recent Actions**:
   - Check what was modified
   - Revert unauthorized changes if necessary

3. **Enable MFA** (if available):
   - Add additional security layer

4. **Monitor Account**:
   - Watch for suspicious activity
   - Alert on unusual login patterns

---

## Backup & Recovery Issues

### Problem: "Database Backup Failed"

**Symptoms**:
- No backup file created
- Backup scheduled but not running
- Error in backup logs

**Investigation**:
```bash
# Check backup script
sudo cat /var/log/backup.log

# Test backup manually
mysqldump -u user -p polytechnicportal > test-backup.sql

# Check disk space for backup
df -h /backups/
```

**Solutions**:

| Issue | Fix |
|-------|-----|
| Disk full | Delete old backups or expand storage |
| Permissions | `chmod 777 /backups/` |
| User credentials wrong | Verify MySQL user in backup script |
| Cron not running | Check cron service: `sudo systemctl status cron` |
| Script path wrong | Use absolute path in cron job |

---

### Problem: "Cannot Restore from Backup"

**Symptoms**:
- Error during restore: "Table already exists"
- Backup file corrupted
- Restore takes too long

**Solutions**:

1. **Handle Duplicate Tables**:
   ```bash
   # Drop existing database before restore
   mysql -u root -p -e "DROP DATABASE IF EXISTS polytechnicportal;"
   mysql -u root -p -e "CREATE DATABASE polytechnicportal;"
   
   # Then restore
   mysql -u root -p polytechnicportal < backup-file.sql
   ```

2. **Verify Backup Integrity**:
   ```bash
   # Check backup file
   head -50 backup-file.sql
   tail -50 backup-file.sql
   
   # Restore to test database
   mysql -u root -p test_polytechnic < backup-file.sql
   ```

3. **Restore Specific Table**:
   ```bash
   # Extract single table from backup
   sed -n '/CREATE TABLE `users`/,/;$/p' backup-file.sql > users-table.sql
   mysql -u root -p polytechnicportal < users-table.sql
   ```

---

## Deployment Issues

### Problem: ".htaccess Rules Not Working"

**Symptoms**:
- URLs like `/index.php?/student/index` not working as `/student/index`
- 404 errors on clean URLs
- Pages accessible but styling/scripts missing

**Investigation**:
```bash
# Check if mod_rewrite is enabled
sudo apache2ctl -M | grep rewrite

# Check if AllowOverride is set
grep -r "AllowOverride" /etc/apache2/

# Check .htaccess syntax
apache2ctl configtest
```

**Solutions**:

1. **Enable mod_rewrite**:
   ```bash
   sudo a2enmod rewrite
   sudo systemctl reload apache2
   ```

2. **Fix AllowOverride**:
   ```apache
   # In /etc/apache2/sites-available/site.conf
   <Directory /var/www/polytechnic/public_html>
       AllowOverride All  # Must be 'All' not 'None'
   </Directory>
   ```

3. **Verify .htaccess**:
   ```apache
   <IfModule mod_rewrite.c>
       RewriteEngine On
       RewriteBase /DulagPolytechnicPortal_github/
       
       RewriteCond %{REQUEST_FILENAME} !-f
       RewriteCond %{REQUEST_FILENAME} !-d
       RewriteRule ^(.*)$ index.php?/$1 [L]
   </IfModule>
   ```

---

## Getting Help

### Debug Mode

Enable debug output for development:
```php
// application/config/config.php
define('ENVIRONMENT', 'development');

// Show detailed error messages
$config['log_threshold'] = 4; // Log everything
$config['log_path'] = 'application/logs/';

// Display errors to browser (development only)
ini_set('display_errors', 1);
error_reporting(E_ALL);
```

### Check Logs

```bash
# Application logs
tail -f application/logs/log-*.php

# Apache error log
tail -f /var/log/apache2/error.log

# Apache access log
tail -f /var/log/apache2/access.log

# PHP error log
tail -f /var/log/php-errors.log

# MySQL error log
tail -f /var/log/mysql/error.log

# System logs
sudo journalctl -u apache2 -f
```

### Common Solutions

1. **Clear Cache**:
   ```bash
   rm -rf application/cache/*
   ```

2. **Check Permissions**:
   ```bash
   sudo chown -R www-data:www-data /var/www/polytechnic/
   sudo chmod -R 755 /var/www/polytechnic/
   ```

3. **Restart Services**:
   ```bash
   sudo systemctl restart apache2
   sudo systemctl restart mysql
   sudo systemctl restart php*-fpm
   ```

4. **Test Connection**:
   ```php
   // Create test file: test-connection.php
   <?php
   $link = mysqli_connect('localhost', 'root', '', 'polytechnicportal');
   if ($link) {
       echo "Database connection successful!";
   } else {
       echo "Connection failed: " . mysqli_connect_error();
   }
   ?>
   ```

---

This troubleshooting guide provides solutions for the most common issues encountered during development and operation. For issues not covered here, refer to logs and contact system administrators.
