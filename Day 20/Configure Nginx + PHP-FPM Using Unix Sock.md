# Day 20: Configure Nginx + PHP-FPM Using Unix Socket

## 🎯 TASK

The Nautilus application team needs a PHP app deployed on `stapp03` with these requirements:

a. Install `nginx` on `stapp03`, configure it to listen on port **8095**, and set the document root to `/var/www/html`.
b. Install `php-fpm` version **8.1** on `stapp03` and configure it to use the unix socket `/var/run/php-fpm/default.sock` (create parent directories if missing).
c. Configure `php-fpm` and `nginx` to work together via the unix socket.
d. Verify from the jump host using: `curl http://stapp03:8095/index.php`.

> Note: `index.php` and `info.php` are already present under `/var/www/html`; do not modify them.

---

## 🛠 Solution (step-by-step)

Perform the steps below on `stapp03` (app server 3). Use `sudo` where necessary.

### 1) Install nginx

```bash
# RHEL/CentOS/Alma/Rocky
sudo yum install -y nginx
# or on newer systems
sudo dnf install -y nginx
```

Enable and start nginx:

```bash
sudo systemctl enable --now nginx
sudo systemctl status nginx --no-pager
```

### 2) Install PHP 8.1 and php-fpm

We want PHP 8.1. On RHEL-family you might need EPEL and Remi repo or the vendor repositories. Example using Remi (adjust if your environment already has PHP 8.1 packages):

```bash
# install prerequisites
sudo yum install -y https://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum install -y yum-utils
sudo yum-config-manager --enable remi-php81
sudo yum install -y php php-fpm php-mysqlnd php-cli php-xml php-gd php-mbstring
```

Verify php-fpm installed and version:

```bash
php -v
php-fpm -v
```

### 3) Configure php-fpm to use the unix socket `/var/run/php-fpm/default.sock`

Create the parent directory and set permissions:

```bash
sudo mkdir -p /var/run/php-fpm
sudo chown -R apache:apache /var/run/php-fpm || true
```

Edit the php-fpm pool configuration. On RHEL systems the pool file is typically at `/etc/php-fpm.d/www.conf`. Update these directives inside the file (or create a dedicated pool file):

Find and set:

```ini
[www]
user = apache
group = apache
listen = /var/run/php-fpm/default.sock
listen.owner = nginx
listen.group = nginx
listen.mode = 0660
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 2
pm.max_spare_servers = 10
```

Notes:

- Use the correct user/group for your distribution: some systems use `www-data` (Debian/Ubuntu). On RHEL-family `apache` (httpd) is common; `nginx` user is typically `nginx`. Adjust owners accordingly.

Restart php-fpm:

```bash
sudo systemctl enable --now php-fpm
sudo systemctl restart php-fpm
sudo systemctl status php-fpm --no-pager
```

### 4) Configure nginx to pass PHP requests to the unix socket

Create or edit an nginx server block (for example `/etc/nginx/conf.d/stapp03.conf`):

```nginx
server {
    listen 8095;
    server_name stapp03;
    root /var/www/html;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/var/run/php-fpm/default.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    # optional: increase client body size for uploads
    client_max_body_size 20M;
}
```

Test nginx configuration and reload:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

If SELinux is enforcing, allow nginx to connect to the php-fpm socket and read files:

```bash
# Allow httpd (nginx) to connect to network and unix sockets managed by php-fpm
sudo setsebool -P httpd_can_network_connect on
sudo chcon -R -t httpd_sys_content_t /var/www/html
sudo chcon -t httpd_var_run_t /var/run/php-fpm || true
```

### 5) Verify from the jump host

From the jump host run:

```bash
curl -I http://stapp03:8095/index.php
curl http://stapp03:8095/index.php
```

Expected: HTTP/200 and the PHP-generated page content. `info.php` should also work similarly.

### Troubleshooting

- Check php-fpm logs: `/var/log/php-fpm/www-error.log` (path may vary) or system journal: `sudo journalctl -u php-fpm -n 200`
- Check nginx logs: `/var/log/nginx/error.log` and `/var/log/nginx/access.log`
- Ensure socket exists and permissions allow nginx to read/write:

```bash
ls -l /var/run/php-fpm/default.sock
```

- If socket owner/group mismatch, either change `listen.owner`/`listen.group` in pool config or adjust nginx user in `/etc/nginx/nginx.conf`.

---

## ✅ Acceptance Criteria

- `nginx` installed and listening on port **8095** on `stapp03`.
- `php-fpm` 8.1 installed and using unix socket `/var/run/php-fpm/default.sock`.
- nginx passes PHP requests to php-fpm via the socket and serves `/var/www/html/index.php` correctly.
- `curl http://stapp03:8095/index.php` from the jump host returns the PHP output.

---

## 📚 References

- Nginx FastCGI docs: https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html
- PHP-FPM documentation: https://www.php.net/manual/en/install.fpm.php
