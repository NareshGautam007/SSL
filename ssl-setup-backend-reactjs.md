### Prerequisites:
- **Nginx** is installed on your server.
- A domain name pointing to your server’s IP address.
- **Certbot** is installed on your server to automatically obtain and renew SSL certificates from Let’s Encrypt.
  
---

## Step 1: Install Certbot and Nginx Plugin

If you haven’t installed Certbot and its Nginx plugin, install them using the following commands:

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

## Step 2: Configure Nginx for React.js (Frontend)

1. **Create Nginx Configuration for the React.js Application:**
   Create a configuration file for your React.js application in the `/etc/nginx/sites-available/` directory:

   ```bash
   sudo nano /etc/nginx/sites-available/react-app
   ```

2. **Add the following configuration:**

   ```nginx
   server {
       listen 80;
       server_name yourdomain.com www.yourdomain.com;

       root /var/www/react-app/build;
       index index.html index.htm;

       location / {
           try_files $uri /index.html;
       }

       # Redirect all HTTP requests to HTTPS
       location ~ /.well-known/acme-challenge {
           allow all;
       }
   }
   ```

   - **Replace `yourdomain.com` with your actual domain name.**
   - **`/var/www/react-app/build` should be the path to your React.js app’s build folder.** You can adjust it based on where your app is deployed.

3. **Enable the Site:**

   Link the configuration file to the `sites-enabled` directory:

   ```bash
   sudo ln -s /etc/nginx/sites-available/react-app /etc/nginx/sites-enabled/
   ```

4. **Test and Reload Nginx:**

   Test if your Nginx configuration is valid:

   ```bash
   sudo nginx -t
   ```

   If the test is successful, reload Nginx:

   ```bash
   sudo systemctl reload nginx
   ```

---

## Step 3: Configure Nginx for Backend (Node.js)

1. **Create Nginx Configuration for the Backend:**

   Create a configuration file for your backend in the `/etc/nginx/sites-available/` directory:

   ```bash
   sudo nano /etc/nginx/sites-available/node-backend
   ```

2. **Add the following configuration:**

   ```nginx
   server {
       listen 80;
       server_name api.yourdomain.com;

       location / {
           proxy_pass http://localhost:3000; # Replace with the actual port your backend is running on
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }

       # Redirect all HTTP requests to HTTPS
       location ~ /.well-known/acme-challenge {
           allow all;
       }
   }
   ```

   - **Replace `api.yourdomain.com` with the domain you’ll use for your backend.**
   - **Replace `localhost:3000` with the actual port your backend is running on.**

3. **Enable the Site:**

   Link the configuration file to the `sites-enabled` directory:

   ```bash
   sudo ln -s /etc/nginx/sites-available/node-backend /etc/nginx/sites-enabled/
   ```

4. **Test and Reload Nginx:**

   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

---

## Step 4: Obtain SSL Certificates with Certbot

1. **Run Certbot for the React.js (Frontend) Application:**

   ```bash
   sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
   ```

   Follow the prompts to obtain and install the SSL certificate for your React.js app.

2. **Run Certbot for the Backend Application:**

   ```bash
   sudo certbot --nginx -d api.yourdomain.com
   ```

   Follow the prompts to obtain and install the SSL certificate for your backend.

---

## Step 5: Update Nginx Configuration for SSL

After obtaining the SSL certificates, Certbot will automatically configure Nginx to use them. You should see the following configuration in your Nginx config files:

### For React.js (Frontend):
```nginx
server {
    listen 443 ssl;
    server_name yourdomain.com www.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    root /var/www/react-app/build;
    index index.html index.htm;

    location / {
        try_files $uri /index.html;
    }

    location ~ /.well-known/acme-challenge {
        allow all;
    }
}

server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$host$request_uri;
}
```

### For Backend (Node.js):
```nginx
server {
    listen 443 ssl;
    server_name api.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/api.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.yourdomain.com/privkey.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location ~ /.well-known/acme-challenge {
        allow all;
    }
}

server {
    listen 80;
    server_name api.yourdomain.com;
    return 301 https://$host$request_uri;
}
```

---

## Step 6: Verify and Automate Renewal

1. **Verify SSL Installation:**

   After completing the setup, visit `https://yourdomain.com` and `https://api.yourdomain.com` in your browser to verify that the SSL certificates are working.

2. **Set Up Automatic Renewal:**

   Certbot will automatically set up a cron job to renew the certificates. You can test the renewal process with:

   ```bash
   sudo certbot renew --dry-run
   ```

---

## Summary:
1. **Frontend (React.js):** Create an Nginx configuration pointing to the React build folder and use Certbot to install SSL.
2. **Backend (Node.js):** Create an Nginx configuration for proxying requests to your backend and use Certbot to install SSL.
3. **Automatic SSL Renewal** is managed by Certbot, ensuring the certificates are always up to date.

This process ensures both your **React frontend** and **Node.js backend** are secured with SSL certificates, and the certificates are automatically renewed.
