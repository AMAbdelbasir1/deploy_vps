# Deployment Guide for VPS on DigitalOcean

This guide provides a clear and structured approach to deploying your application on a DigitalOcean VPS. Follow these steps sequentially to set up your server, install necessary tools, configure your application, and secure your environment.

---

## Step 1: Connect to Your Server

### Objective:
Establish a secure shell (SSH) connection to your VPS.

### Command:
```bash
ssh root@<your-server-ip>
```

---

## Step 2: Install Node.js and NPM

### Objective:
Install Node.js and NPM to manage and run your application.

### Commands:
1. Update the package list:
   ```bash
   sudo apt update
   ```

2. Install Node.js and NPM:
   ```bash
   sudo apt install -y nodejs npm
   ```

---

## Step 3: Install and Use NVM (Node Version Manager)

### Objective:
Use NVM to manage multiple versions of Node.js.

### Commands:
1. Download and install NVM:
   ```bash
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.4/install.sh | bash
   ```

2. Set up the environment variables:
   ```bash
   export NVM_DIR="$HOME/.nvm"
   [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
   [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
   ```

---

## Step 4: Install PM2

### Objective:
Install PM2 to manage Node.js processes.

### Commands:
1. Install PM2 globally:
   ```bash
   npm install -g pm2
   ```

2. Start the application:
   ```bash
   pm2 start app.js
   ```

3. Manage the application:
   - Restart:
     ```bash
     pm2 restart app.js
     ```
   - View logs:
     ```bash
     pm2 logs
     ```
   - Check status:
     ```bash
     pm2 status
     ```

4. Configure PM2 to start on system boot:
   ```bash
   pm2 startup ubuntu
   ```

---

## Step 5: Check Running Application

### Objective:
Verify if the application is running on the desired port.

### Command:
```bash
netstat -tuln | grep 3000
```

---

## Step 6: Install Git

### Objective:
Install Git for version control and repository management.

### Commands:
1. Update the package list:
   ```bash
   sudo apt update
   ```

2. Install Git:
   ```bash
   sudo apt install -y git
   ```

---

## Step 7: Configure Nginx

### Install Nginx:

#### Objective:
Set up Nginx as a reverse proxy server and install SSL management tools.

#### Commands:
1. Update the package list:
   ```bash
   sudo apt update
   ```

2. Install Nginx:
   ```bash
   sudo apt install -y nginx
   ```

3. Install Certbot for SSL:
   ```bash
   sudo apt install -y certbot python3-certbot-nginx
   ```

### Configure Nginx for Your Application:

#### Objective:
Set up Nginx to reverse proxy traffic to your application.

#### Commands:
1. Open the Nginx configuration file:
   ```bash
   sudo nano /etc/nginx/sites-available/your-domain.conf
   ```

2. Add the following configuration:
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

3. Enable the configuration and reload Nginx:
   ```bash
   sudo ln -s /etc/nginx/sites-available/your-domain.conf /etc/nginx/sites-enabled/
   sudo nginx -t
   sudo systemctl reload nginx
   ```

### Add SSL Using Certbot:

#### Objective:
Secure your application with an SSL certificate.

#### Commands:
1. Obtain an SSL certificate:
   ```bash
   sudo certbot --nginx -d your-domain.com -d www.your-domain.com
   ```

2. Test SSL renewal:
   ```bash
   sudo certbot renew --dry-run
   ```

### Optional: Self-Signed SSL Configuration

#### Objective:
Use a self-signed SSL certificate for local development or testing.

#### Commands:
1. Create a directory for SSL certificates:
   ```bash
   sudo mkdir -p /etc/nginx/ssl
   ```

2. Generate a self-signed SSL certificate:
   ```bash
   sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
       -keyout /etc/nginx/ssl/selfsigned.key \
       -out /etc/nginx/ssl/selfsigned.crt
   ```

3. Generate Diffie-Hellman parameters:
   ```bash
   sudo openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
   ```

4. Update the Nginx configuration:
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

5. Add the certificate to the system's trusted store:
   ```bash
   sudo cp /etc/nginx/ssl/selfsigned.crt /usr/local/share/ca-certificates/selfsigned.crt
   sudo update-ca-certificates
   ```

---

## Step 8: Secure Your VPS

### Configure UFW Firewall:

#### Objective:
Protect your server by allowing only essential traffic.

#### Commands:
1. Allow SSH connections:
   ```bash
   sudo ufw allow ssh
   ```

2. Allow traffic on port 3000:
   ```bash
   sudo ufw allow 3000/tcp
   ```

3. Allow full traffic for Nginx:
   ```bash
   sudo ufw allow 'Nginx Full'
   ```

4. Enable the firewall:
   ```bash
   sudo ufw enable
   ```

5. Reload the firewall to apply changes:
   ```bash
   sudo ufw reload
   ```

6. Check the firewall status:
   ```bash
   sudo ufw status
   ```

---

## Step 9: Install and Configure MySQL

### Install MySQL Server:

#### Objective:
Set up a MySQL database server.

#### Commands:
1. Install MySQL:
   ```bash
   sudo apt install mysql-server
   ```

2. Start the MySQL service:
   ```bash
   sudo systemctl start mysql
   ```

3. Restart the MySQL service:
   ```bash
   sudo systemctl restart mysql
   ```

### Secure MySQL Installation:

#### Objective:
Remove default vulnerabilities and set a root password.

#### Command:
```bash
sudo mysql_secure_installation
```

### Create a MySQL User and Database:

#### Objective:
Set up a dedicated user and database for your application.

#### Commands:
1. Log in to MySQL:
   ```sql
   sudo mysql -u root -p
   ```

2. Create a new user:
   ```sql
   CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'app_password';
   ```

3. Grant privileges:
   ```sql
   GRANT ALL PRIVILEGES ON your_database.* TO 'app_user'@'localhost';
   ```

4. Apply the changes:
   ```sql
   FLUSH PRIVILEGES;
   ```

---

## Conclusion

You have successfully set up your VPS on DigitalOcean with Node.js, Nginx, PM2, and MySQL. This deployment guide ensures your application is secured, running efficiently, and ready to handle traffic. Use this document as a reference for managing and maintaining your server.

