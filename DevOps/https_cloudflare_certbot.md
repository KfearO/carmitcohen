# HTTPS Behind Cloudflare with Certbot

This document describes the repeatable HTTPS setup for servers that are placed behind Cloudflare and use a DNS challenge for Let's Encrypt.

Use this flow when direct HTTP validation is not the right fit, or when you want development and production to follow the same operational pattern.

## 1. General Rules

### 1.1 Choose One Installation Method

Do not mix `snap` and `apt` installs of Certbot on the same server.

Recommended by Certbot: `snap`

Practical fallback: `apt`

Use the `apt` path if `snap` fails on the server. In practice, `snap` can be problematic on some `arm64` environments.

### 1.2 Why DNS Challenge

Advantages:

- No dependency on port `80`
- Works when the server is behind Cloudflare
- Does not require direct public validation to the origin server
- Supports repeatable issuance for development and production

## 2. Cloudflare Prerequisites

### 2.1 SSL/TLS Mode

In the Cloudflare dashboard:

```text
SSL/TLS -> Overview
```

Set:

```text
Current encryption mode: Full
```

### 2.2 API Token

In the Cloudflare dashboard:

```text
My Profile -> API Tokens
```

Create or edit a token with:

```text
Zone -> DNS -> Edit
Zone -> Zone -> Read
```

Scope the token only to the required zone whenever possible.

## 3. Install Certbot

### 3.1 Option A: Install with `snap`

Install `snapd` if needed:

```bash
sudo apt update
sudo apt install -y snapd
```

Install Certbot:

```bash
sudo snap install core
sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/local/bin/certbot
```

Install the Cloudflare plugin:

```bash
sudo snap set certbot trust-plugin-with-root=ok
sudo snap install certbot-dns-cloudflare
```

### 3.2 Option B: Install with `apt`

Use this when `snap` does not work, especially on `arm64`.

```bash
sudo apt update
sudo apt install -y certbot python3-certbot-dns-cloudflare
```

### 3.3 Verify Plugin Availability

```bash
sudo certbot plugins
```

Expected output should include:

```text
dns-cloudflare
```

## 4. Create Cloudflare Credentials File

### 4.1 Create the File

```bash
sudo vim /etc/letsencrypt/cloudflare.ini
```

Contents:

```ini
dns_cloudflare_api_token=YOUR_API_TOKEN
```

### 4.2 Restrict Permissions

```bash
sudo chmod 600 /etc/letsencrypt/cloudflare.ini
```

## 5. Issue a Certificate

### 5.1 Single-Domain Example

```bash
sudo certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /etc/letsencrypt/cloudflare.ini \
  --cert-name carmitc.kfearoshri.com \
  -d carmitc.kfearoshri.com
```

### 5.2 Multi-Domain Example

```bash
sudo certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /etc/letsencrypt/cloudflare.ini \
  --cert-name example.com \
  -d example.com \
  -d www.example.com
```

## 6. Apache Integration

### 6.1 Confirm SSL Module

```bash
sudo a2enmod ssl
```

### 6.2 Confirm Apache Configuration

```bash
sudo apache2ctl configtest
```

### 6.3 Reload Apache

```bash
sudo systemctl reload apache2
```

### 6.4 Inspect Apache Vhosts

```bash
sudo apache2ctl -S
```

## 7. Renewal and Inspection

### 7.1 List Certificates

```bash
sudo certbot certificates
```

### 7.2 Renew Certificates in Dry Run Mode

```bash
sudo certbot renew --dry-run
```

### 7.3 Check Certificate Expiration Date

```bash
openssl x509 -in /etc/letsencrypt/live/your-domain/fullchain.pem -noout -dates
```

### 7.4 View Logs

```bash
sudo less /var/log/letsencrypt/letsencrypt.log
```

```bash
sudo tail -f /var/log/letsencrypt/letsencrypt.log
```

## 8. Notes

- `Certificate Name` in Certbot is an internal label, not the authoritative list of covered hostnames.
- Actual domain coverage is determined by the `Domains:` section in `certbot certificates`.
- Keep this flow aligned between development and production whenever the goal is operational parity.
