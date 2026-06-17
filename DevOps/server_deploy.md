# WordPress + Elementor Feasibility Playbook

This document captures a repeatable server setup flow for the Carmit Cohen WordPress environment.

Use it as the primary playbook for a fresh server build.

For HTTPS behind Cloudflare using a DNS challenge, see [https_cloudflare_certbot.md](C:/Users/Kfear/WebstormProjects/carmitcohen/DevOps/https_cloudflare_certbot.md:1).

That document is a reusable infrastructure reference.

This document remains the project-specific source of truth for the Carmit Cohen deployment flow.

## 1. System Preparation

### 1.1 Update Package Lists

```bash
sudo apt update
```

### 1.2 Upgrade Installed Packages

```bash
sudo apt upgrade
```

### 1.3 Install Apache

```bash
sudo apt install apache2
```

### 1.4 Verify Apache Status

```bash
systemctl status apache2
```

### 1.5 Install PHP and WordPress Dependencies

```bash
sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-intl php-mbstring php-xml php-zip
```

### 1.6 Verify PHP Version

```bash
php -v
```

### 1.7 Install MariaDB

```bash
sudo apt install mariadb-server
```

### 1.8 Verify MariaDB Status

```bash
systemctl status mariadb
```

## 2. WordPress Base Install

### 2.1 Change to Temporary Directory

```bash
cd /tmp
```

### 2.2 Download WordPress

```bash
wget https://wordpress.org/latest.tar.gz
```

### 2.3 Extract WordPress Archive

```bash
tar -xzf latest.tar.gz
```

### 2.4 Verify Extracted Files

```bash
ls -la /tmp/wordpress
```

### 2.5 Create Website Directory

```bash
sudo mkdir -p /var/www/carmitcohen
```

### 2.6 Copy WordPress Files

```bash
sudo cp -a /tmp/wordpress/. /var/www/carmitcohen/
```

### 2.7 Verify Website Directory Contents

```bash
ls -la /var/www/carmitcohen
```

## 3. Database Setup

### 3.1 Enter MariaDB

```bash
sudo mariadb
```

### 3.2 Create Database

```sql
CREATE DATABASE wordpress_db;
```

### 3.3 Create Database User

```sql
CREATE USER 'wordpress_user'@'localhost' IDENTIFIED BY 'YOUR_STRONG_PASSWORD';
```

### 3.4 Grant Permissions

```sql
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wordpress_user'@'localhost';
```

### 3.5 Apply Changes

```sql
FLUSH PRIVILEGES;
```

### 3.6 Exit MariaDB

```sql
EXIT;
```

## 4. WordPress Configuration

### 4.1 Change to Website Directory

```bash
cd /var/www/carmitcohen
```

### 4.2 Create `wp-config.php`

```bash
cp wp-config-sample.php wp-config.php
```

### 4.3 Edit `wp-config.php`

```bash
vim wp-config.php
```

Set at minimum:

```php
define( 'DB_NAME', 'wordpress_db' );
define( 'DB_USER', 'wordpress_user' );
define( 'DB_PASSWORD', 'YOUR_STRONG_PASSWORD' );
define( 'DB_HOST', 'localhost' );
```

### 4.4 Generate and Insert WordPress Salts

Open the following URL in a browser:

```text
https://api.wordpress.org/secret-key/1.1/salt/
```

Copy the generated block and replace the default authentication key and salt block in `wp-config.php`.

This is required for a production deployment and should also be treated as required for a development server when the goal is to mirror production behavior.

### 4.5 Inspect Important `wp-config.php` Settings

```bash
grep -n "DB_NAME\|DB_USER\|DB_HOST\|DB_CHARSET\|DB_COLLATE\|AUTH_KEY\|SECURE_AUTH_KEY\|LOGGED_IN_KEY\|NONCE_KEY" /var/www/carmitcohen/wp-config.php
```

### 4.6 Verify `wp-config.php` Syntax

```bash
php -l /var/www/carmitcohen/wp-config.php
```

## 5. Apache Virtual Host

### 5.1 Create Apache Virtual Host

```bash
sudo vim /etc/apache2/sites-available/carmitc.conf
```

### 5.2 Apache Virtual Host Configuration

```apache
<VirtualHost *:80>
    ServerName carmitc.kfearoshri.com
    DocumentRoot /var/www/carmitcohen

    <Directory /var/www/carmitcohen>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/carmitc_error.log
    CustomLog ${APACHE_LOG_DIR}/carmitc_access.log combined
</VirtualHost>
```

### 5.3 Enable Website

```bash
sudo a2ensite carmitc.conf
```

### 5.4 Enable `mod_rewrite`

```bash
sudo a2enmod rewrite
```

### 5.5 Verify Apache Configuration

```bash
sudo apache2ctl configtest
```

### 5.6 Reload Apache

```bash
sudo systemctl reload apache2
```

## 6. Local Verification

### 6.1 Test Virtual Host Locally

```bash
curl -I -H "Host: carmitc.kfearoshri.com" http://127.0.0.1
```

### 6.2 Check Website Error Log

```bash
sudo tail -n 50 /var/log/apache2/carmitc_error.log
```

### 6.3 Check Global Apache Log

```bash
sudo tail -n 80 /var/log/apache2/error.log
```

### 6.4 Create a Temporary PHP Test File

```bash
echo "<?php phpinfo();" | sudo tee /var/www/carmitcohen/info.php
```

### 6.5 Verify PHP Execution

```bash
curl -I -H "Host: carmitc.kfearoshri.com" http://127.0.0.1/info.php
```

### 6.6 Verify Database Connectivity

```bash
mariadb -u wordpress_user -p wordpress_db
```

### 6.7 Verify WordPress Tables

After WordPress is installed and the database is populated:

```sql
SHOW TABLES;
```

### 6.8 Exit MariaDB

```sql
EXIT;
```

### 6.9 Verify WordPress Response

```bash
curl -I -H "Host: carmitc.kfearoshri.com" http://127.0.0.1
```

## 7. External DNS, Cloudflare, and HTTPS

### 7.1 General Approach

This environment is operated so that development behaves as close to production as possible.

That includes:

- placing the development hostname behind Cloudflare
- issuing an origin certificate with Certbot using the Cloudflare DNS challenge
- configuring Apache to redirect HTTP to HTTPS
- verifying both the Apache origin behavior and the external Cloudflare behavior

### 7.2 Verify DNS

```bash
nslookup carmitc.kfearoshri.com
```

Confirm that the hostname resolves as expected for the environment.

### 7.3 Cloudflare Requirements

At minimum, confirm the following in Cloudflare:

- The DNS record points to the correct server public IP.
- The hostname is proxied according to the intended environment design.
- `SSL/TLS` mode is set to `Full`.

### 7.4 Install Certbot and the Cloudflare Plugin

Use one installation method only on the server.

Recommended practical path on this server type:

```bash
sudo apt update
sudo apt install -y certbot python3-certbot-dns-cloudflare
```

Verify:

```bash
sudo certbot plugins
```

Expected output should include:

```text
dns-cloudflare
```

For the broader reusable flow, see [https_cloudflare_certbot.md](C:/Users/Kfear/WebstormProjects/carmitcohen/DevOps/https_cloudflare_certbot.md:1).

Use the reference document for shared infrastructure knowledge, but use this playbook for the exact order and project-specific values required on the Carmit Cohen server.

### 7.5 Create the Cloudflare Credentials File

```bash
sudo vim /etc/letsencrypt/cloudflare.ini
```

Contents:

```ini
dns_cloudflare_api_token=YOUR_API_TOKEN
```

Restrict permissions:

```bash
sudo chmod 600 /etc/letsencrypt/cloudflare.ini
```

### 7.6 Issue the Certificate

```bash
sudo certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /etc/letsencrypt/cloudflare.ini \
  --cert-name carmitc.kfearoshri.com \
  -d carmitc.kfearoshri.com
```

### 7.7 Update Apache Virtual Host for HTTPS

Edit the existing site file:

```bash
sudo vim /etc/apache2/sites-available/carmitc.conf
```

Use this structure:

```apache
<VirtualHost *:80>
    ServerName carmitc.kfearoshri.com
    DocumentRoot /var/www/carmitcohen

    RewriteEngine On
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [R=301,L]
</VirtualHost>

<VirtualHost *:443>
    ServerName carmitc.kfearoshri.com
    DocumentRoot /var/www/carmitcohen

    <Directory /var/www/carmitcohen>
        AllowOverride All
        Require all granted
    </Directory>

    SSLEngine on
    SSLCertificateFile /etc/letsencrypt/live/carmitc.kfearoshri.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/carmitc.kfearoshri.com/privkey.pem

    ErrorLog ${APACHE_LOG_DIR}/carmitc_ssl_error.log
    CustomLog ${APACHE_LOG_DIR}/carmitc_ssl_access.log combined
</VirtualHost>
```

### 7.8 Enable Apache Modules Required for HTTPS

```bash
sudo a2enmod ssl
sudo a2enmod rewrite
```

### 7.9 Validate and Reload Apache

```bash
sudo apache2ctl configtest
sudo systemctl reload apache2
```

### 7.10 Verify Apache Virtual Host Registration

```bash
sudo apache2ctl -S
```

### 7.11 Verify Origin HTTP Redirect

This verifies the Apache origin directly:

```bash
curl -I -H "Host: carmitc.kfearoshri.com" http://127.0.0.1
```

Expected behavior: HTTP returns a redirect to `https://carmitc.kfearoshri.com/...`.

### 7.12 Verify Origin HTTPS Response

This verifies the Apache origin directly on `127.0.0.1` while forcing the target hostname:

```bash
curl -Ik --resolve carmitc.kfearoshri.com:443:127.0.0.1 https://carmitc.kfearoshri.com
```

### 7.13 Verify External HTTPS Through Cloudflare

This verifies the public behavior through Cloudflare:

```bash
curl -I https://carmitc.kfearoshri.com
```

### 7.14 Optional Log Inspection

```bash
sudo tail -n 50 /var/log/apache2/carmitc_ssl_error.log
sudo tail -n 80 /var/log/apache2/error.log
```

## 9. WordPress GUI Setup

### 9.1 Open the Website in a Browser

Browse to the site and complete the standard WordPress installation wizard.

### 9.2 Complete Initial Setup

Record at minimum:

- Site language
- Site title
- Admin username
- Admin email

### 9.3 Confirm Admin Access

Verify that:

- The site loads successfully
- Login works
- `/wp-admin` is accessible

### 9.4 Verify Database Population

After the GUI wizard completes, reconnect and confirm the WordPress tables exist:

```bash
mariadb -u wordpress_user -p wordpress_db
```

```sql
SHOW TABLES;
```

## 10. Current Status

- Debian 12 ARM64
- Apache installed and running
- PHP installed
- MariaDB installed and running
- WordPress installed manually
- Apache Virtual Host configured
- Local HTTP behavior verified with `curl` using the target `Host` header
- Development environment is expected to behave as close to production as practical
- Cloudflare and HTTPS should be treated as first-class parts of the deployment flow
- WordPress GUI installation must be completed and verified on each fresh environment
