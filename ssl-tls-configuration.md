# Apache SSL/TLS Configuration

This addendum provides a manual SSL/TLS VirtualHost configuration for Apache. The paths below are examples; replace them with the actual certificate and private-key paths for your server and domain.

```apache
<VirtualHost *:443>
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/html

    SSLEngine on
    SSLCertificateFile /path/to/cert
    SSLCertificateKeyFile /path/to/Key

    ErrorLog /var/log/httpd/ssl_error.log
    CustomLog /var/log/httpd/ssl_access.log combined
</VirtualHost>
```

Ensure Apache's SSL module is installed and enabled, and that the server listens on port `443`. On RHEL/Amazon Linux, the SSL module is commonly supplied by `mod_ssl`; install it if needed with `sudo yum install mod_ssl -y`. On Ubuntu/Debian, enable the module with `sudo a2enmod ssl`.

Keep the private key readable only by the required privileged services and administrators. Before applying the configuration, check that the certificate matches the domain and private key, then validate and reload Apache:

```bash
sudo apachectl configtest
sudo systemctl reload httpd   # Ubuntu/Debian: sudo systemctl reload apache2
```

Allow inbound HTTPS traffic on port `443` in the host firewall and cloud security group. For Let's Encrypt certificates managed by Certbot, prefer the certificate paths and VirtualHost configuration Certbot installs; it also handles renewal.
