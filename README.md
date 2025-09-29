# Pterodactyl ARM Installation Guide

Step-by-step guide for installing **Pterodactyl Panel** and **Wings** on ARM devices (e.g., Raspberry Pi).

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1: Install a 64-Bit OS](#step-1-install-a-64-bit-os)
- [Step 2: Enable SSH (Optional)](#step-2-enable-ssh-optional)
- [Step 3: Update Your System](#step-3-update-your-system)
- [Step 4: Install Docker](#step-4-install-docker)
- [Step 5: Install Pterodactyl Panel](#step-5-install-pterodactyl-panel)
- [Step 6: Install Pterodactyl Wings](#step-6-install-pterodactyl-wings)
- [Step 7: Set Up SSL](#step-7-set-up-ssl)
- [Step 8: Deploy & Configure Your Node](#step-8-deploy--configure-your-node)
- [Step 9: Allocations & Networking](#step-9-allocations--networking)
- [Troubleshooting](#troubleshooting)
- [Credits](#credits)

---

## Prerequisites

- ARM device (e.g., Raspberry Pi 4/5)
- 64-bit OS (32-bit is NOT supported)
- At least 2GB RAM (Panel recommended: 4GB+)
- Internet connection
- Basic knowledge of Linux command line

---

## Step 1: Install a 64-Bit OS

1. Download [Raspberry Pi Imager](https://www.raspberrypi.org/software/).
2. Download the [Raspberry Pi OS 64-bit Image](https://downloads.raspberrypi.org/raspios_arm64/images/raspios_arm64-2020-05-28/2020-05-27-raspios-buster-arm64.zip).
3. Launch Raspberry Pi Imager.
4. Choose your OS and SD card, and click **Write**.

---

## Step 2: Enable SSH (Optional)

1. Click the Raspberry Pi logo (top-left).
2. Go to **Preferences > Raspberry Pi Configuration**.
3. In the **Interfaces** tab, enable **SSH**.
4. Click **OK** to save.
5. Make sure port 22 is open ([Guide](https://www.hellotech.com/guide/for/how-to-port-forward)).

---

## Step 3: Update Your System

```bash
sudo apt update
sudo apt full-upgrade
```

---

## Step 4: Install Docker

### Quick Install

```bash
curl -sSL https://get.docker.com/ | CHANNEL=stable bash
```

### If Quick Install Fails

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common

# Add Docker's GPG key
curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | sudo apt-key add -

# Add Docker repository
echo "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list

sudo apt update
sudo apt install -y --no-install-recommends docker-ce cgroupfs-mount

# Enable and start Docker
sudo systemctl enable docker
sudo systemctl start docker
```

---

## Step 5: Install Pterodactyl Panel

> **Tip:** If you want to install the Panel on a different machine (recommended), use a VPS or server running Ubuntu 22.04 or Debian 11+.

### 1. Install Required Packages

```bash
sudo apt -y install php8.3 php8.3-{common,cli,gd,mysql,mbstring,bcmath,xml,fpm,curl,zip} mariadb-server nginx tar unzip git redis-server

curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
```

**Note:** Replace `php8.3` with the latest available PHP version for your OS.

### 2. Create Database

```bash
sudo mysql -u root
```

In the MariaDB shell:

```sql
CREATE DATABASE panel;
CREATE USER 'pterodactyl'@'127.0.0.1' IDENTIFIED BY 'strongpassword';
GRANT ALL PRIVILEGES ON panel.* TO 'pterodactyl'@'127.0.0.1';
FLUSH PRIVILEGES;
EXIT;
```

### 3. Download Panel

```bash
cd /var/www/
sudo git clone https://github.com/pterodactyl/panel.git --branch=v1.11.4 --depth=1 panel
cd panel
```

> Check [Pterodactyl releases](https://github.com/pterodactyl/panel/releases) for the latest stable version.

### 4. Install Composer

```bash
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

### 5. Set Permissions

```bash
sudo chown -R www-data:www-data /var/www/panel
```

### 6. Install Dependencies

```bash
composer install --no-dev --optimize-autoloader
```

### 7. Configure Environment

```bash
cp .env.example .env
php artisan key:generate
```

Edit `.env` and set your database, cache, and mail settings.

### 8. Run Database Migrations

```bash
php artisan migrate --seed --force
```

### 9. Set Up Nginx

Replace the default Nginx config with [the official sample](https://pterodactyl.io/panel/1.0/getting_started.html#webserver-configuration) or:

```nginx
server {
    listen 80;
    server_name panel.example.com;

    root /var/www/panel/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

Reload Nginx:

```bash
sudo systemctl restart nginx
```

---

## Step 6: Install Pterodactyl Wings

Back on the node (ARM device):

```bash
sudo mkdir -p /etc/pterodactyl
curl -L -o /usr/local/bin/wings https://github.com/pterodactyl/wings/releases/latest/download/wings_linux_arm64
chmod u+x /usr/local/bin/wings
```

---

## Step 7: Set Up SSL

On your Panel server:

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d panel.example.com
```

On your Wings node (if using SSL for Wings):

```bash
sudo apt install -y certbot
sudo certbot certonly --standalone -d node.example.com
```

---

## Step 8: Deploy & Configure Your Node

1. Go to your Panel ([http://panel.example.com](http://panel.example.com))
2. Login as admin.
3. **Administration > Nodes > Create Node**
4. Fill in node details, get the configuration for your Wings node.
5. On your Wings node, create `/etc/pterodactyl/config.yml` and paste the config.
6. Start Wings:

```bash
sudo systemctl enable wings
sudo systemctl start wings
```

If `systemctl` fails:

```bash
sudo apt install screen
screen -S wings
wings
```
*(Close the terminal window to detach)*

---

## Step 9: Allocations & Networking

- **Don't use your public IP** for allocations, use your gateway IP:
  
  ```bash
  hostname -I | awk '{print $1}'
  ```

---

## Troubleshooting

- If you encounter issues, run Wings in debug mode and create a GitHub issue:

  ```bash
  wings --debug
  ```

- Check the [Pterodactyl documentation](https://pterodactyl.io/) for Panel or Wings errors.
- For PHP or Nginx errors, check logs in `/var/log/nginx/` and `/var/www/panel/storage/logs/`.

---

## Credits

- Guide by [Hecker-01](https://github.com/Hecker-01)
- Pterodactyl Project: [GitHub](https://github.com/pterodactyl/panel) | [Wings](https://github.com/pterodactyl/wings)

---

*If you find issues or need help, feel free to open an issue on this repository!*
