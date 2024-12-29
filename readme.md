# Deployment Guide for VPS on DigitalOcean

This guide provides step-by-step instructions to deploy your application on a DigitalOcean VPS. Follow these steps to set up your server, install necessary tools, configure your application, and secure your environment.

## Step 1: Connect to Your Server

### Command:

```bash
ssh root@<your-server-ip>
```

### What it does:

Establishes a secure shell (SSH) connection to your VPS.

## Step 2: Install Node.js and NPM

### Command:

```bash
sudo apt update
sudo apt install -y nodejs npm
```

### What it does:

Updates the package list and installs Node.js and NPM for running and managing Node.js applications.

## Step 3: Install and Use NVM (Node Version Manager)

### Commands:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.4/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

### What it does:

Installs NVM to manage different Node.js versions and sets up the environment variables for using NVM.

## Step 4: Install PM2

### Commands:

```bash
npm install -g pm2
pm2 start app.js
pm2 restart app.js
pm2 logs
pm2 status
pm2 list
pm2 startup ubuntu
```

### What it does:

- Installs PM2 globally to manage Node.js processes.
- Starts or restarts the application.
- Displays logs and the status of the application.
- Configures PM2 to start on system boot.

## Step 5: Check Running Application

### Command:

```bash
netstat -tuln | grep 3000
```

### What it does:

Checks if the application is running on port 3000.

## Step 6: Install Git

### Command:

```bash
sudo apt update
sudo apt install -y git
```

### What it does:

Installs Git for version control and cloning repositories.

## Step 7: Configure Nginx

### Install Nginx:

#### Commands:

```bash
sudo apt update
sudo apt install -y nginx
sudo apt install -y certbot python3-certbot-nginx
```

#### What it does:

Installs Nginx as a reverse proxy server and Certbot for SSL management.

### Configure Nginx for Your Application:

#### Command:

```bash
sudo nano /etc/nginx/sites-available/your-domain.conf
```

#### What it does:

Opens the Nginx configuration file for editing.

#### Example Configuration:

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

#### Enable Configuration and Reload Nginx:

```bash
sudo ln -s /etc/nginx/sites-available/your-domain.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

#### What it does:

Links the configuration file, tests for errors, and reloads Nginx.

### Add SSL Using Certbot:

#### Commands:

```bash
sudo certbot --nginx -d your-domain.com -d www.your-domain.com
sudo certbot renew --dry-run
```

#### What it does:

Generates and renews SSL certificates for your domain using Certbot.

### Optional: Self-Signed SSL Configuration

#### Local SSL Configuration:

##### Commands:

```bash
sudo mkdir -p /etc/nginx/ssl
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/ssl/selfsigned.key \
  -out /etc/nginx/ssl/selfsigned.crt
sudo openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
```

##### What it does:

Generates a self-signed SSL certificate and Diffie-Hellman parameters for local use.

#### Update SSL Configuration:

##### Command:

```nginx
server {
    listen 443 ssl;
    server_name your-domain.com;

    ssl_certificate /etc/nginx/ssl/selfsigned.crt;
    ssl_certificate_key /etc/nginx/ssl/selfsigned.key;
    ssl_dhparam /etc/nginx/ssl/dhparam.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

server {
    listen 80;
    server_name your-domain.com;

    return 301 https://$host$request_uri;
}
```

##### What it does:

Configures Nginx to use the self-signed certificate for HTTPS and redirects HTTP to HTTPS.

#### Update System Certificates:

```bash
sudo cp /etc/nginx/ssl/selfsigned.crt /usr/local/share/ca-certificates/selfsigned.crt
sudo update-ca-certificates
```

#### What it does:

Adds the self-signed certificate to the system's trusted certificate store.

## Step 8: Secure Your VPS

### Configure UFW Firewall:

#### Commands:

```bash
sudo ufw allow ssh
sudo ufw allow 3000/tcp
sudo ufw allow 'Nginx Full'
sudo ufw enable
sudo ufw reload
sudo ufw status
```

#### What it does:

Configures the UFW firewall to allow SSH, the application port, and Nginx traffic.

## Step 9: Install and Configure MySQL

### Install MySQL Server:

#### Commands:

```bash
sudo apt install mysql-server
sudo systemctl start mysql
sudo systemctl restart mysql
```

#### What it does:

Installs and starts the MySQL server.

### Secure MySQL Installation:

#### Command:

```bash
sudo mysql_secure_installation
```

#### What it does:

Secures the MySQL installation by setting a root password and removing insecure defaults.

### Create a MySQL User and Database:

#### Commands:

```sql
sudo mysql -u root -p
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'app_password';
GRANT ALL PRIVILEGES ON your_database.* TO 'app_user'@'localhost';
FLUSH PRIVILEGES;
```

#### What it does:

Creates a database user, grants privileges, and applies changes.

## Conclusion

You have now set up your VPS on DigitalOcean with Node.js, Nginx, PM2, and MySQL. Your application is secured with SSL and ready to handle traffic. Use this documentation as a reference to manage and maintain your deployment.
