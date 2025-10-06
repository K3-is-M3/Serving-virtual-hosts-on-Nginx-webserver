<h1>Hosting virtual hosts on nginx webserver</h1>


<h2>Description</h2>
This project provides a step-by-step process on how to set up nginx on your Linux server, configure your virtual hosts (domains/subdomains) correctly, and also procure SSL certs for them using the python-certbot package for nginx
<br />

<h2>Technologies Used</h2>

- <b>nginx</b>
- <b>python3-certbot-nginx</b>

<h2>Environments Used</h2>

- <b>Ubuntu Linux</b> 

---

<h2>Installation & Setup Walk-through:</h2>

- <b>First make sure your repositories are up-to-date, then install nginx and python3-certbot-nginx</b>

### 1. Install nginx and python3-certbot-nginx
```bash
sudo apt update && sudo apt install nginx python3-certbot-nginx
```
Ensure nginx service is up and running; enable it as well to start on boot:
```
sudo systemctl start nginx; sudo systemctl enable nginx
```
Navigate to the sites-available folder in the nginx directory, where you will create the configuration file for your virtual host:
```
cd /etc/nginx/sites-available
sudo nano domain.conf 
```
Populate the configuration file with an Nginx virtual host template.
This initial setup listens on port 80 and serves traffic over the HTTP protocol.

Before proceeding, ensure you’ve already created the appropriate A record for your domain or subdomain in your DNS or web hosting provider. This step is required for virtual hosting to function correctly in Nginx.
```
# ===========================
# Virtual Host (HTTP Only)
# ===========================

server {
    listen 80;
    listen [::]:80;

    server_name ##YOUR_DOMAIN_HERE;

    # If serving a frontend, use:
    # root /var/www//html;
    # index index.html;

    # Reverse proxy to backend (adjust port/IP as needed)
    location / {
        proxy_pass http://SERVER_IP:PORT/;   # or your backend IP: PORT
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Logging
    access_log /var/log/nginx/api.support.numeraliot.com_access.log;
    error_log /var/log/nginx/api.support.numeraliot.com_error.log;
}
```
Save changes and quit

Next, create a symbolic link for the configuration file you created inside the /etc/nginx/sites-enabled directory.
This ensures that any updates made to your configuration file in the /etc/nginx/sites-available directory are automatically reflected in the active configuration used by Nginx.
```
sudo ln -s /etc/nginx/sites-available/domain.conf /etc/nginx/sites-enabled
```
Verify your Nginx configuration for syntax errors.
If no issues are found, you’re good to go — you can safely restart Nginx.
If there are any errors, they’ll be displayed in the terminal so you can identify and fix them before reloading the service.
```
sudo nginx -t
sudo systemctl restart nginx
```
Your web server is now partially configured and listening on port 80 (HTTP).
Next, let’s secure it by enabling HTTPS so your site can serve encrypted traffic over port 443.

### 2. Run python3-certbot on your configured virtual host
```bash
sudo certbot --nginx -d domain.com -d www.domain.com
```
Below is what a successful SSL certificate request looks like in the terminal:
```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Requesting a certificate for domain.com and www.domain.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/domain.com/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/domain.com/privkey.pem
This certificate expires on 2026-01-04.
These files will be automatically renewed every 90 days.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

Deploying certificate
Successfully deployed certificate for domain.com to /etc/nginx/sites-enabled/domain.conf
Successfully deployed certificate for www.domain.com to /etc/nginx/sites-enabled/domain.conf
Congratulations! You have successfully enabled HTTPS on:
  - domain.com
  - www.domain.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you like to redirect all HTTP traffic to HTTPS?
1: No redirect - Make no further changes to the webserver configuration
2: Redirect - Make all requests redirect to secure HTTPS access
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter]: **2**

Redirecting all traffic on port 80 to HTTPS in /etc/nginx/sites-enabled/domain.conf

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! Your HTTPS setup is complete.

Your certificate and chain have been saved at:
   /etc/letsencrypt/live/domain.com/fullchain.pem
Your key file has been saved at:
   /etc/letsencrypt/live/domain.com/privkey.pem
Your web server has been configured to automatically redirect HTTP traffic to HTTPS.

Test your configuration at:
   https://www.ssllabs.com/ssltest/analyze.html?d=domain.com

```

### 3. Test renewal simulation
```
sudo certbot renew --dry-run
```
Expected output:
```
Congratulations, all simulated renewals succeeded.
```
🧾 Notes for your the user

You can safely replace domain.com with **any placeholder** or your project subdomain.

No real DNS or SSL validation happens in the demo — it’s purely a walkthrough simulation of a successful Certbot run.

### 4. etc/nginx/sites-available/domain.conf (after SSL setup)
```
# ===========================
# Virtual Host (HTTP + HTTPS)
# Auto-configured by Certbot
# ===========================

# --- HTTP configuration ---
server {
    listen 80;
    listen [::]:80;
    server_name ##YOUR_DOMAIN_HERE;

    # Redirect all HTTP requests to HTTPS
    return 301 https://$host$request_uri;
}

# --- HTTPS configuration ---
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name ##YOUR_DOMAIN_HERE;

    # SSL certificate files (managed by Certbot)
    ssl_certificate /etc/letsencrypt/live/##YOUR_DOMAIN_HERE/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/##YOUR_DOMAIN_HERE/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # If serving a frontend, uncomment and adjust:
    # root /var/www/##YOUR_DOMAIN_HERE/html;
    # index index.html;

    # Reverse proxy to backend (adjust port/IP as needed)
    location / {
        proxy_pass http://SERVER_IP:PORT/;  # backend IP and port
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Logging
    access_log /var/log/nginx/##YOUR_DOMAIN_HERE_access.log;
    error_log /var/log/nginx/##YOUR_DOMAIN_HERE_error.log;
}

```
You should now be able to securely access your newly configured domain or subdomain through your web browser!
