# WordPress + Elementor Feasibility Playbook

## Update Package Lists

```bash
sudo apt update
```

## Upgrade Installed Packages

```bash
sudo apt upgrade
```

## Install Apache

```bash
sudo apt install apache2
```

## Verify Apache Status

```bash
systemctl status apache2
```

## Install PHP and WordPress Dependencies

```bash
sudo apt install php libapache2-mod-php php-mysql php-curl php-gd php-intl php-mbstring php-xml php-zip
```

## Verify PHP Version

```bash
php -v
```

## Install MariaDB

```bash
sudo apt install mariadb-server
```

## Verify MariaDB Status

```bash
systemctl status mariadb
```

## Change to Temporary Directory

```bash
cd /tmp
```

## Download WordPress

```bash
wget https://wordpress.org/latest.tar.gz
```

## Extract WordPress Archive

```bash
tar -xzf latest.tar.gz
```

## Verify Extracted Files

```bash
ls -la /tmp/wordpress
```

## Create Website Directory

```bash
sudo mkdir -p /var/www/carmitcohen
```

## Copy WordPress Files

```bash
sudo cp -a /tmp/wordpress/. /var/www/carmitcohen/
```

## Verify Website Directory Contents

```bash
ls -la /var/www/carmitcohen
```

## Enter MariaDB

```bash
sudo mariadb
```

## Create Database

```sql
CREATE DATABASE wordpress_db;
```

## Create Database User

```sql
CREATE USER 'wordpress_user'@'localhost' IDENTIFIED BY 'YOUR_STRONG_PASSWORD';
```

## Grant Permissions

```sql
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wordpress_user'@'localhost';
```

## Apply Changes

```sql
FLUSH PRIVILEGES;
```

## Exit MariaDB

```sql
EXIT;
```

## Change to Website Directory

```bash
cd /var/www/carmitcohen
```

## Create wp-config.php

```bash
cp wp-config-sample.php wp-config.php
```

## Edit wp-config.php

```bash
vim wp-config.php
```

## Create Apache Virtual Host

```bash
sudo vim /etc/apache2/sites-available/carmitc.conf
```

## Apache Virtual Host Configuration

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

## Enable Website

```bash
sudo a2ensite carmitc.conf
```

## Enable mod_rewrite

```bash
sudo a2enmod rewrite
```

## Verify Apache Configuration

```bash
sudo apache2ctl configtest
```

## Reload Apache

```bash
sudo systemctl reload apache2
```

## Test Virtual Host Locally

```bash
curl -I -H "Host: carmitc.kfearoshri.com" http://127.0.0.1
```

## Check Website Error Log

```bash
sudo tail -n 50 /var/log/apache2/carmitc_error.log
```

## Check Global Apache Log

```bash
sudo tail -n 80 /var/log/apache2/error.log
```

## Create Temporary PHP Test File

```bash
echo "<?php phpinfo();" | sudo tee /var/www/carmitcohen/info.php
```

## Verify PHP Execution

```bash
curl -I -H "Host: carmitc.kfearoshri.com" http://127.0.0.1/info.php
```

## Verify wp-config.php Syntax

```bash
php -l /var/www/carmitcohen/wp-config.php
```

## Verify Database Connectivity

```bash
mariadb -u wordpress_user -p wordpress_db
```

## Verify WordPress Tables

```sql
SHOW TABLES;
```

## Exit MariaDB

```sql
EXIT;
```

## Inspect Important wp-config.php Settings

```bash
grep -n "DB_NAME\|DB_USER\|DB_HOST\|DB_CHARSET\|DB_COLLATE\|AUTH_KEY\|SECURE_AUTH_KEY\|LOGGED_IN_KEY\|NONCE_KEY" /var/www/carmitcohen/wp-config.php
```

## Generate WordPress Salts

```text
https://api.wordpress.org/secret-key/1.1/salt/
```

## Verify WordPress Response

```bash
curl -I -H "Host: carmitc.kfearoshri.com" http://127.0.0.1
```

## Verify DNS

```bash
nslookup carmitc.kfearoshri.com
```

## Verify External HTTP Access

```bash
curl -I http://carmitc.kfearoshri.com
```

# Current Status

- Debian 12 ARM64
- Apache installed and running
- PHP 8.2 installed
- MariaDB installed and running
- WordPress installed manually
- Apache Virtual Host configured
- WordPress accessible locally
- WordPress redirects to `install.php`
- DNS configuration pending verification
- HTTPS not configured yet
- Cloudflare returns HTTP 521 because nothing is listening on port 443
- WordPress installation wizard not yet completed
- Elementor not yet installed