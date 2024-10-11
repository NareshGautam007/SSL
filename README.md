# SSL Configuration with Nginx for React.js and Node.js Applications

This project demonstrates how to configure **SSL (Secure Socket Layer)** for both a **React.js frontend** and a **Node.js backend** application using **Nginx** and **Let's Encrypt** for free SSL certificates.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Server Setup](#server-setup)
- [Nginx Configuration for React.js Frontend](#nginx-configuration-for-reactjs-frontend)
- [Nginx Configuration for Node.js Backend](#nginx-configuration-for-nodejs-backend)
- [SSL Certificate Setup with Let's Encrypt](#ssl-certificate-setup-with-lets-encrypt)
- [Verify SSL Installation](#verify-ssl-installation)
- [Automating SSL Renewal](#automating-ssl-renewal)

---

## Prerequisites

- A server running **Ubuntu** or similar Linux distribution
- **Nginx** installed on the server
- **Certbot** and **Nginx plugin** for Certbot installed
- A registered domain name pointing to your server's IP address
- **Node.js** backend application running on a port (e.g., 3000)
- **React.js** application built and deployed on the server

---

## Server Setup

### 1. Install Nginx

If Nginx is not installed yet, install it using the following command:

```bash
sudo apt update
sudo apt install nginx
```

### 2. Install Certbot and Nginx Plugin

Certbot is used to obtain SSL certificates from Let's Encrypt:

```bash
sudo apt install certbot python3-certbot-nginx
```

---

## Nginx Configuration for React.js Frontend

1. **Create Nginx Configuration for React App**

   ```bash
   sudo nano /etc/nginx/sites-available/react-app
   ```

   Add the following configuration:

   ```nginx
   server {
       listen 80;
       server_name yourdomain.com www.yourdomain.com;

       root /var/www/react-app/build;
       index index.html index.htm;

       location / {
           try_files $uri /index.html;
       }

       # For Certbot validation
       location ~ /.well-known/acme-challenge {
           allow all;
       }
   }
   ```

   - Replace `yourdomain.com` with your actual domain.
   - Ensure that the `root` points to the correct location of your React build files.

2. **Enable the Site**

   ```bash
   sudo ln -s /etc/nginx/sites-available/react-app /etc/nginx/sites-enabled/
   ```

3. **Test and Reload Nginx**

   Test and reload Nginx to apply the configuration:

   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

---

## Nginx Configuration for Node.js Backend

1. **Create Nginx Configuration for the Backend**

   ```bash
   sudo nano /etc/nginx/sites-available/node-backend
   ```

   Add the following configuration:

   ```nginx
   server {
       listen 80;
       server_name api.yourdomain.com;

       location / {
           proxy_pass http://localhost:3000;  # Port where Node.js app runs
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }

       # For Certbot validation
       location ~ /.well-known/acme-challenge {
           allow all;
       }
   }
   ```

   - Replace `api.yourdomain.com` with your actual backend domain.
   - Replace `localhost:3000` with the port where your Node.js app runs.

2. **Enable the Site**

   ```bash
   sudo ln -s /etc/nginx/sites-available/node-backend /etc/nginx/sites-enabled/
   ```

3. **Test and Reload Nginx**

   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

---

## SSL Certificate Setup with Let’s Encrypt

1. **Obtain SSL Certificate for React App**

   Run Certbot to automatically obtain and configure SSL for your React app:

   ```bash
   sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
   ```

2. **Obtain SSL Certificate for Node.js Backend**

   Run Certbot for your backend domain:

   ```bash
   sudo certbot --nginx -d api.yourdomain.com
   ```

Certbot will automatically configure SSL and redirect HTTP traffic to HTTPS.

---

## Verify SSL Installation

After SSL is set up, visit the following URLs to verify the SSL certificate installation:

- **React App (Frontend):** https://yourdomain.com
- **Node.js Backend:** https://api.yourdomain.com

---

## Automating SSL Renewal

Certbot will automatically renew your SSL certificates. To ensure the renewal process works, you can run the following command:

```bash
sudo certbot renew --dry-run
```

---

## Conclusion

This guide shows how to configure SSL for a **React.js frontend** and **Node.js backend** using **Nginx** and **Let’s Encrypt**. The configuration ensures that your applications are secure and can serve requests over HTTPS, with automatic SSL certificate renewal.

