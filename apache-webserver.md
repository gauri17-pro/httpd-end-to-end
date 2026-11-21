# Apache Web Server Full Course & Study Guide

**Source Video:** [Apache Web Server Tutorial for Beginners | Custom Domain, HTTPS, SSL, VirtualHost, Reverse Proxy](https://www.youtube.com/watch?v=NACRKwkXKO0)  
**Instructor:** M Prashant  
**Duration:** ~1 Hour 28 Minutes  

---

## 📋 Table of Contents
1. [Introduction to Apache Web Server](#1-introduction-to-apache-web-server)
2. [Lab Setup & Installation (AWS EC2)](#2-lab-setup--installation-aws-ec2)
3. [Directory Structure & Configuration Overview](#3-directory-structure--configuration-overview)
4. [Hosting a Static Website](#4-hosting-a-static-website)
5. [Logging & Custom Error Pages](#5-logging--custom-error-pages)
6. [Custom Domain & HTTPS/SSL Configuration](#6-custom-domain--httpsssl-configuration)
7. [Virtual Hosts (Hosting Multiple Websites)](#7-virtual-hosts-hosting-multiple-websites)
8. [Apache as a Reverse Proxy](#8-apache-as-a-reverse-proxy)
9. [Load Balancing with Apache](#9-load-balancing-with-apache)
10. [Further Topics to Explore](#10-further-topics-to-explore)

---

## 1. Introduction to Apache Web Server
*(Timestamp: 00:00 – 04:45)*

* **What is Apache HTTP Server?**  
  An open-source, cross-platform web server software developed and maintained by the Apache Software Foundation. In RHEL/CentOS/Amazon Linux, the service/package is referred to as `httpd`, while in Debian/Ubuntu, it is referred to as `apache2`.
* **How it Works:**  
  Listens for incoming HTTP/HTTPS requests from clients (browsers) on ports `80` (HTTP) and `443` (HTTPS), processes them, and serves static files (HTML, CSS, JS, images) or forwards requests to backend applications.
* **Apache vs. NGINX:**
  * **Apache:** Process/thread-driven architecture, highly flexible via `.htaccess` directory-level overrides, rich ecosystem of dynamic modules.
  * **NGINX:** Event-driven, asynchronous architecture, excels at handling high concurrency and static content with low memory overhead.

---

## 2. Lab Setup & Installation (AWS EC2)
*(Timestamp: 04:45 – 13:10)*

### Prerequisites
* Launch an **AWS EC2 Instance** (Amazon Linux 2023 / CentOS / RHEL or Ubuntu).
* Configure the **Security Group** inbound rules to allow:
  * **SSH (Port 22):** Your IP
  * **HTTP (Port 80):** `0.0.0.0/0`
  * **HTTPS (Port 443):** `0.0.0.0/0`

### Installation & Service Management (Amazon Linux / RHEL / CentOS)
```bash
# Update system packages
sudo yum update -y

# Install Apache (httpd)
sudo yum install httpd -y

# Start the Apache service
sudo systemctl start httpd

# Enable Apache to start automatically on boot
sudo systemctl enable httpd

# Check service status
sudo systemctl status httpd
```

> **Note for Ubuntu/Debian users:** Replace `yum` with `apt` and `httpd` with `apache2`.

---

## 3. Directory Structure & Configuration Overview
*(Timestamp: 15:42 – 30:05)*

### Key Directories & Files (RHEL / Amazon Linux)
* `/etc/httpd/` — Main configuration root directory.
* `/etc/httpd/conf/httpd.conf` — Primary Apache configuration file.
* `/etc/httpd/conf.d/` — Directory for modular/additional `.conf` files (Virtual Hosts, SSL, Proxies).
* `/var/www/html/` — Default `DocumentRoot` where website files are stored.
* `/var/log/httpd/` — Default directory for `access_log` and `error_log`.

### Core Directives in `httpd.conf`
| Directive | Description | Example |
| :--- | :--- | :--- |
| `ServerRoot` | Top-level directory containing server config/log files | `ServerRoot "/etc/httpd"` |
| `Listen` | IP address and port Apache listens on | `Listen 80` |
| `IncludeOptional` | Loads external configuration files | `IncludeOptional conf.d/*.conf` |
| `User` / `Group` | System user/group running the Apache worker processes | `User apache` / `Group apache` |
| `ServerAdmin` | Email address included in server-generated error pages | `ServerAdmin admin@example.com` |
| `ServerName` | Hostname and port the server uses to identify itself | `ServerName www.example.com:80` |
| `DocumentRoot` | Directory from which Apache serves files | `DocumentRoot "/var/www/html"` |
| `<Directory>` | Applies access control and options to a specific folder | `<Directory "/var/www/html">` |

### Syntax Check & Reloading
Always validate syntax before restarting or reloading Apache:
```bash
# Test configuration syntax
sudo apachectl configtest   # or: sudo httpd -t

# Gracefully reload configuration without dropping active connections
sudo systemctl reload httpd
```

---

## 4. Hosting a Static Website
*(Timestamp: 13:10 – 15:42 & 30:05 – 32:09)*

1. Navigate to the web root directory:
   ```bash
   cd /var/www/html
   ```
2. Create or upload your HTML/CSS/JS assets (e.g., `index.html`).
3. Ensure proper ownership and permissions so the `apache` user can read the files:
   ```bash
   sudo chown -R apache:apache /var/www/html
   sudo chmod -R 755 /var/www/html
   ```
4. Access `http://<EC2-PUBLIC-IP>` in your browser to verify the site is live.

---

## 5. Logging & Custom Error Pages
*(Timestamp: 25:21 – 27:39 & 46:55 – 50:34)*

### Logging
* **Access Log (`/var/log/httpd/access_log`):** Records every incoming HTTP request (client IP, timestamp, HTTP method, requested URI, status code, user-agent).
* **Error Log (`/var/log/httpd/error_log`):** Records diagnostic information and server errors.
* **Custom Log Formats:** Configured using `LogFormat` and `CustomLog` directives inside `httpd.conf` or individual VirtualHost blocks.

### Custom Error Pages
Instead of displaying default Apache error messages (e.g., `404 Not Found`), you can route users to custom HTML pages using the `ErrorDocument` directive:
```apache
ErrorDocument 404 /custom_404.html
ErrorDocument 500 /custom_500.html
```
*(Place `custom_404.html` inside your `DocumentRoot`.)*

---

## 6. Custom Domain & HTTPS/SSL Configuration
*(Timestamp: 32:09 – 46:55)*

### Step 1: Pointing a Custom Domain
1. Open your domain registrar's DNS settings (e.g., Route 53, GoDaddy, Namecheap).
2. Create an **A Record** pointing your domain (e.g., `example.com` and `www.example.com`) to your EC2 instance's **Public IPv4 address**.

### Step 2: Securing with Let's Encrypt (Certbot)
Use Certbot to obtain a free SSL/TLS certificate and automatically configure HTTPS:
```bash
# Install Certbot and the Apache plugin
sudo yum install certbot python3-certbot-apache -y

# Obtain and install SSL certificate
sudo certbot --apache -d example.com -d www.example.com
```
* Follow the interactive prompts to enter your email, agree to the Terms of Service, and enable automatic HTTP-to-HTTPS redirection.
* Certbot automatically creates an SSL VirtualHost file (e.g., `/etc/httpd/conf.d/le-ssl.conf`) and sets up a systemd timer/cron job for auto-renewal (`sudo certbot renew --dry-run`).

---

## 7. Virtual Hosts (Hosting Multiple Websites)
*(Timestamp: 50:34 – 01:06:38)*

**Name-Based Virtual Hosting** allows a single Apache server (with one IP address) to host multiple independent websites based on the requested domain name.

### Directory Setup
```bash
sudo mkdir -p /var/www/site1.com/html
sudo mkdir -p /var/www/site2.com/html
```

### Configuration Example (`/etc/httpd/conf.d/vhosts.conf`)
```apache
<VirtualHost *:80>
    ServerName site1.com
    ServerAlias www.site1.com
    DocumentRoot /var/www/site1.com/html

    ErrorLog /var/log/httpd/site1_error.log
    CustomLog /var/log/httpd/site1_access.log combined
</VirtualHost>

<VirtualHost *:80>
    ServerName site2.com
    ServerAlias www.site2.com
    DocumentRoot /var/www/site2.com/html

    ErrorLog /var/log/httpd/site2_error.log
    CustomLog /var/log/httpd/site2_access.log combined
</VirtualHost>
```
Run `sudo apachectl configtest` and `sudo systemctl reload httpd` to apply changes.

---

## 8. Apache as a Reverse Proxy
*(Timestamp: 01:06:38 – 01:15:34)*

A **Reverse Proxy** sits in front of backend application servers (such as Node.js, Python/Flask, React, or Java Tomcat running on local ports like `3000` or `8080`) and forwards client requests to them.

### Required Modules
Ensure `mod_proxy` and `mod_proxy_http` are enabled (enabled by default in standard RHEL/Amazon Linux builds; on Ubuntu, run `sudo a2enmod proxy proxy_http`).

### Reverse Proxy VirtualHost Example
```apache
<VirtualHost *:80>
    ServerName app.example.com

    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/

    ErrorLog /var/log/httpd/app_error.log
    CustomLog /var/log/httpd/app_access.log combined
</VirtualHost>
```
* `ProxyPreserveHost On`: Passes the original `Host` HTTP header from the client to the backend app.
* `ProxyPass`: Maps incoming requests to the backend server URL.
* `ProxyPassReverse`: Adjusts URLs in HTTP redirect headers sent back by the backend server.

---

## 9. Load Balancing with Apache
*(Timestamp: 01:15:34 – 01:24:19)*

Apache can distribute incoming traffic across multiple backend servers using `mod_proxy_balancer`.

### Load Balancer Configuration Example
```apache
<Proxy "balancer://mycluster">
    BalancerMember http://10.0.1.10:8080
    BalancerMember http://10.0.1.11:8080
    ProxySet lbmethod=byrequests
</Proxy>

<VirtualHost *:80>
    ServerName lb.example.com

    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/
</VirtualHost>
```
* **`lbmethod` Options:**
  * `byrequests`: Distributes traffic evenly based on request count (Weighted Round Robin).
  * `bytraffic`: Distributes based on byte count.
  * `bybusyness`: Routes requests to the server with the fewest active requests.

---

## 10. Further Topics to Explore
*(Timestamp: 01:24:19 – End)*

To continue mastering Apache Web Server, explore:
* **`.htaccess` & URL Rewriting (`mod_rewrite`):** Creating SEO-friendly URLs and directory-level overrides.
* **Basic Authentication (`.htpasswd`):** Password-protecting sensitive endpoints or admin directories.
* **Performance Tuning:** Understanding Multi-Processing Modules (**MPMs**) such as `prefork`, `worker`, and `event`, along with `KeepAlive` settings.
* **Security Hardening:** Hiding server version banners (`ServerTokens Prod`, `ServerSignature Off`), configuring security headers (`X-Frame-Options`, `Strict-Transport-Security`), and setting up ModSecurity (WAF).