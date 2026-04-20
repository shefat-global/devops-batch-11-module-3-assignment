# devops-batch-11-module-3-assignment
devops batch 11 module 3 assignment

# Project Setup: Next.js with Nginx, SSL, Node.js, and PM2

## 1. Install Nginx

```bash
sudo apt update
sudo apt install nginx
```

Start and enable Nginx to run on boot:

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

## 2. Configure Nginx for HTTPS and Reverse Proxy

Edit the Nginx site configuration:

```bash
sudo nano /etc/nginx/sites-available/secure-app
```

Add the following configuration for HTTPS and reverse proxy:

```nginx
server {
    listen 80;
    server_name 13.201.6.22;  # Use your IP or domain

    # Redirect all HTTP requests to HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name 13.201.6.22;  # Use your IP or domain

    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

    # SSL configurations for better security
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'HIGH:!aNULL:!MD5';
    ssl_prefer_server_ciphers on;

    # Path to the frontend files (static files)
    root /var/www/secure-app;
    index index.html index.htm;

    # Serve static files directly if they exist
    location / {
        try_files $uri $uri/ @proxy;
    }

    # Reverse Proxy to Node.js backend if static files don't exist
    location @proxy {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable the site configuration:

```bash
sudo ln -s /etc/nginx/sites-available/secure-app /etc/nginx/sites-enabled/
```

Test the configuration:

```bash
sudo nginx -t
```

Reload Nginx to apply the changes:

```bash
sudo systemctl reload nginx
```

## 3. Install Node.js, NPM, and PM2

Install Node.js and NPM:

```bash
sudo apt update
sudo apt install -y nodejs npm
```

Install PM2 globally to manage the app process:

```bash
sudo npm install pm2@latest -g
```

## 4. Set Up Your Project

Navigate to your project folder:

```bash
cd /var/www/secure-app
```

Clone the repository:

```bash
sudo git clone https://github.com/mdarifahammedreza/next-japan.git .
```

Install project dependencies:

```bash
sudo npm install
```

Build the Next.js app:

```bash
sudo npm run build
```

Start the app using PM2:

```bash
pm2 start npm --name "next-japan" -- run start
```

Save the PM2 process list:

```bash
pm2 save
```

Set up PM2 to restart on reboot:

```bash
pm2 startup
```

## 5. Generate SSL Certificate Using OpenSSL

Create the SSL directory:

```bash
sudo mkdir -p /etc/nginx/ssl
```

Generate a self-signed SSL certificate:

```bash
sudo openssl req -x509 -newkey rsa:4096 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt -days 365 -nodes
```

- Common Name: Use `13.201.6.22` or your domain name.

## 6. System Updates and Cleanup

Update your system:

```bash
sudo apt update
sudo apt upgrade
```

Perform a full upgrade:

```bash
sudo apt full-upgrade
```

Remove unused packages:

```bash
sudo apt autoremove
sudo apt clean
```

## 7. Permissions

Ensure proper permissions for the `/var/www/secure-app` directory:

```bash
sudo chown -R www-data:www-data /var/www/secure-app
sudo chmod -R 755 /var/www/secure-app
```

