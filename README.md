### Guide: Securing Apache Server by Hiding Server Information and Enabling HSTS

This guide will walk you through the steps to hide the Apache server version, disable web server name headers, and enable HTTP Strict Transport Security (HSTS) for enhanced security. 

---

### **1. Install Required Modules**
First, install the required Apache modules for security and headers handling. These modules allow you to modify HTTP headers and enhance security configurations.

Run the following commands:
```bash
apt-get update
apt-get install libapache2-mod-security2
a2enmod headers security2
systemctl restart apache2
```

---

### **2. Update `security.conf`**
Edit the `security.conf` file to disable server version information and unwanted HTTP features:
```bash
nano /etc/apache2/conf-available/security.conf
```

Add or update the following lines:
```apache
# Minimize server information exposure
ServerTokens Prod
ServerSignature Off
TraceEnable Off

# Add security-related headers
Header set X-Content-Type-Options "nosniff"
Header set Content-Security-Policy "frame-ancestors 'self';"
```

Restart Apache to apply the changes:
```bash
systemctl restart apache2
```

---

### **3. Update `apache2.conf`**
Edit the `apache2.conf` file to enhance security further:
```bash
nano /etc/apache2/apache2.conf
```

Add the following configuration at the bottom of the file:
```apache
<IfModule security2_module>
    SecRuleEngine off
    ServerTokens Min
    SecServerSignature " "
</IfModule>


<IfModule mod_headers.c>
   Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
   Header set X-Powered-By "Unknown"
</IfModule>
```

Restart Apache to apply the new configurations:
```bash
systemctl restart apache2
```

---

### **4. Verify the Configuration**

#### **Check Headers**
Use the `curl` command to inspect the HTTP headers:
```bash
curl -I https://yourdomain.com
```

Ensure that:
- **Server:** header reveals minimal information (`Apache` or nothing).
- **X-Content-Type-Options:** and **Content-Security-Policy:** headers are present.
- **Strict-Transport-Security:** header is enabled.

#### **Test HSTS**
Visit your site in a browser and check the response headers using browser developer tools to confirm the **Strict-Transport-Security** header is applied.

---

### **Optional Enhancements**
1. **Disabling Unnecessary Modules**: Disable any Apache modules not in use to reduce the attack surface:
   ```bash
   a2dismod module_name
   systemctl restart apache2
   ```

2. **Enable HTTPS**: Ensure your site uses HTTPS by installing and configuring an SSL/TLS certificate (e.g., via Let's Encrypt):
   ```bash
   apt-get install certbot python3-certbot-apache
   certbot --apache
   ```

3. **Log Monitoring**: Monitor logs for suspicious activity:
   ```bash
   tail -f /var/log/apache2/access.log /var/log/apache2/error.log
   ```

---

This configuration enhances Apache's security posture by hiding sensitive server details and enforcing strict HTTPS policies.
