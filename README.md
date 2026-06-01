
# Production-Grade LEMP Stack & WordPress Deployment

A comprehensive, terminal-only manual deployment of a highly secure WordPress architecture running on a Linux (Ubuntu) environment. No control panels (cPanel) were utilized.

## 🛠️ Technology Stack
* **OS:** Linux (Ubuntu 24.04 LTS)
* **Web Server:** Nginx 1.24.0 (Reverse Proxy & Static Asset Handler)
* **Database Engine:** MySQL Server (Isolated Multi-User Provisioning)
* **Runtime Backend:** PHP-FPM 8.3 (FastCGI Process Manager via Unix Sockets)
* **Security & Infrastructure:** Certbot (SSL/TLS Encryption Core) & Cloudflare Shield Simulation

---

## 📅 Daily Execution Playbook

### Day 1 & 2: Base Infrastructure & Nginx Architecture
* **Key Commands:**
  ```bash
  sudo apt update && sudo apt upgrade -y
  sudo apt install curl git nginx mysql-server php php-fpm php-mysql unzip -y
  sudo systemctl enable --now nginx
  ```
* **Directory Layout:** Websites are served out of `/var/www/html/`.
* **Lessons Learned:** Debugged a critical `502 Bad Gateway` error. Discovered Nginx's default config was pointing to a rogue proxy port (`8080`) and fixed the `location /` routing blocks to serve static index files properly.

### Day 3 & 4: Database Provisioning & Manual WordPress Installation
* **Key Commands:**
  ```bash
  sudo mysql_secure_installation
  sudo chown -R www-data:www-data /var/www/html/
  sudo find /var/www/html/ -type d -exec chmod 755 {} \;
  sudo find /var/www/html/ -type f -exec chmod 644 {} \;
  ```
* **Database Isolation Configuration:**
  * Database Name: `wordpress_db`
  * Database User: `wp_clerk` bound strictly to `localhost`
* **Lessons Learned:** Encountered a terminal prompt trap (`'>`) caused by a missing single quote prefix. Learned to break stuck SQL statements using `Ctrl + C` without crashing the core server interface.

### Day 5 & 6: Cryptographic Security & Custom Routing
* **Key Commands:**
  ```bash
  sudo apt install certbot python3-certbot-nginx -y
  sudo certbot renew --dry-run
  sudo nano /etc/hosts
  ```
* **Local Simulation Overrides:** Successfully manipulated `/etc/hosts` to point the custom domain name `niyhiweb.local` directly to `127.0.0.1` locally for automated server testing completely free of charge.
* **Network Blueprint:** Audited the background automation worker (`certbot.timer`) responsible for managing 90-day SSL cryptographic roll rotations.

---

## 📂 Audited Production Nginx Configuration
```nginx
server {
        listen 80 default_server;
        root /var/www/html;
        index index.php index.html index.htm;
        server_name niyhiweb.local;

        location / {
                try_files \$uri \(uri/ /index.php?\)args;
        }

        location ~ \.php\$ {
                include snippets/fastcgi-php.conf;
                fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        }
}
```
