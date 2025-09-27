# Pterodactyl File Locations & Editing Guide

This guide lists common files and directories you'll need to edit when customizing or administering your Pterodactyl setup.

---

## Panel (Web Interface)

**Location:** Usually installed at `/var/www/panel`

| File/Folder                      | Purpose                                  | Edit When...                |
|----------------------------------|------------------------------------------|-----------------------------|
| `/var/www/panel/.env`            | Environment variables (DB, mail, etc)    | Changing DB, mail, etc      |
| `/var/www/panel/resources/`      | Blade templates (UI HTML)                | Customizing UI              |
| `/var/www/panel/public/`         | Public assets (CSS, JS, etc)             | Editing static files        |
| `/var/www/panel/config/`         | Laravel app config files                 | Advanced config             |
| `/var/www/panel/app/`            | PHP source code                          | Extending/modifying backend |
| `/var/www/panel/storage/logs/`   | Panel logs                               | Debugging panel issues      |
| `/var/www/panel/database/`       | Migrations & seeds                       | DB structure                |

---

## Wings (Node Daemon)

**Location:** Binary at `/usr/local/bin/wings`  
**Config:** `/etc/pterodactyl/config.yml`

| File/Folder                      | Purpose                                  | Edit When...                |
|----------------------------------|------------------------------------------|-----------------------------|
| `/etc/pterodactyl/config.yml`    | Node configuration                       | Set up node                 |
| `/etc/pterodactyl/`              | Wings config directory                   | Storing configs/certs       |
| `/var/log/wings.log`             | Wings log file (may vary)                | Debugging node issues       |

---

## Webserver (Nginx/Apache for Panel)

| File/Folder                          | Purpose                | Edit When...           |
|--------------------------------------|------------------------|------------------------|
| `/etc/nginx/sites-available/`        | Nginx site configs     | Change domain/SSL      |
| `/etc/nginx/sites-enabled/`          | Nginx active configs   | Enable/disable sites   |
| `/etc/apache2/sites-available/`      | Apache site configs    | Change domain/SSL      |
| `/etc/apache2/sites-enabled/`        | Apache active configs  | Enable/disable sites   |

---

## SSL Certificates

| File/Folder                              | Purpose                | Edit/View When...      |
|------------------------------------------|------------------------|------------------------|
| `/etc/letsencrypt/live/<your-domain>/`   | Certbot SSL certs      | Renew/view certs       |

---

## Database

| Service               | Default Location           | Edit When...           |
|-----------------------|---------------------------|------------------------|
| MariaDB/MySQL         | `/etc/mysql/`             | DB config              |
| PostgreSQL            | `/etc/postgresql/`        | DB config              |

---

## Summary Table

| Component      | Main Directory         | Main Editable Files           |
|----------------|-----------------------|-------------------------------|
| Panel          | `/var/www/panel`      | `.env`, `resources/`, `config/`, `public/` |
| Wings          | `/etc/pterodactyl`    | `config.yml`                  |
| Webserver      | `/etc/nginx/` or `/etc/apache2/` | Site configs        |
| SSL Certs      | `/etc/letsencrypt/`   | cert files                    |
| Database       | `/etc/mysql/` or `/etc/postgresql/` | DB config         |

---

## How to Find/Edit Files

- **Panel config:**  
  `sudo nano /var/www/panel/.env`
- **Node config:**  
  `sudo nano /etc/pterodactyl/config.yml`
- **Nginx config:**  
  `sudo nano /etc/nginx/sites-available/panel`
- **Apache config:**  
  `sudo nano /etc/apache2/sites-available/panel.conf`
- **Wings logs:**  
  `sudo tail -f /var/log/wings.log` (or check journalctl)

---

**Tip:**  
Always back up configuration files before editing, especially `.env` and `config.yml`.  
For source code edits, consider using a version control system like Git.
