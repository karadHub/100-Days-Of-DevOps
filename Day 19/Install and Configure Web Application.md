# Day 19: Install and Configure Web Application

## 🎯 TASK

xFusionCorp Industries is planning to host two static websites on their infra in Stratos Datacenter. The development of these websites is still in-progress, but we want to get the servers ready. Please perform the following steps to accomplish the task:

- a. Install `httpd` package and dependencies on app server 1.(depending on given app server)
- b. Apache should serve on port **5001**.
- c. There are two website backups on the jump host: `/home/thor/ecommerce` and `/home/thor/games`. Set them up on Apache so ecommerce is available at `http://localhost:5001/ecommerce/` and games at `http://localhost:5001/games/` on the app server.
  d. Verify using `curl` on the app server:
  - `curl http://localhost:5001/ecommerce/`
  - `curl http://localhost:5001/games/`

---

## 🛠️ Solution (step-by-step)

Follow these steps on the target app server (App Server 1).

### 1) Install Apache (httpd)

On RHEL/CentOS/Alma/Rocky systems:

```bash
sudo yum install -y httpd
# or if using dnf:
sudo dnf install -y httpd
```

Verify installation:

```bash
httpd -v
```

### 2) Configure Apache to listen on port 5001

Open `/etc/httpd/conf/httpd.conf` and update the Listen directive:

```conf
# change or add
Listen 5001
```

Also ensure the default `VirtualHost` (or create one) listens on 5001. Example minimal virtual host in `/etc/httpd/conf.d/000-default.conf`:

```apache
<VirtualHost *:5001>
    ServerName localhost
    DocumentRoot "/var/www/html"
    <Directory "/var/www/html">
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
</VirtualHost>
```

Restart and enable httpd:

```bash
sudo systemctl enable --now httpd
sudo systemctl restart httpd
sudo systemctl status httpd --no-pager
```

If firewalld is active, allow the port:

```bash
sudo firewall-cmd --permanent --add-port=5001/tcp
sudo firewall-cmd --reload
```

### 3) Deploy the two static sites from the jump host backups

On the jump host, copy the directories to the app server. From the jump host run:

```bash
scp -r /home/thor/ecommerce/ app_server_1:/var/www/html/ecommerce/
scp -r /home/thor/games/ app_server_1:/var/www/html/games/
```

Alternatively, from the app server you can pull files via `scp` or `rsync`:

```bash
scp -r jump_user@jump_host:/home/thor/ecommerce /var/www/html/ecommerce
scp -r jump_user@jump_host:/home/thor/games /var/www/html/games
```

Ensure ownership and permissions are correct (on RHEL-family Apache user `apache`):

```bash
sudo chown -R apache:apache /var/www/html/ecommerce
sudo chown -R apache:apache /var/www/html/games
sudo find /var/www/html -type d -exec chmod 755 {} +
sudo find /var/www/html -type f -exec chmod 644 {} +
```

If you prefer explicit URL mappings, add aliases inside the VirtualHost block:

```apache
Alias /ecommerce/ "/var/www/html/ecommerce/"
Alias /games/ "/var/www/html/games/"

<Directory "/var/www/html/ecommerce">
    Require all granted
</Directory>
<Directory "/var/www/html/games">
    Require all granted
</Directory>
```

After any config change restart httpd.

### 4) Verify locally on the app server

On App Server 1 run:

```bash
curl -I http://localhost:5001/ecommerce/
curl -I http://localhost:5001/games/
```

Expected: HTTP/1.1 200 OK (or 301/302 if index redirects). You can fetch full content without `-I` if needed.

If curl fails, check logs and permissions:

```bash
sudo tail -n 100 /var/log/httpd/error_log
sudo tail -n 100 /var/log/httpd/access_log
sudo ls -la /var/www/html/ecommerce
```

### Troubleshooting notes

- If SELinux is enforcing, allow httpd to read files in user home mounts or set proper contexts:

```bash
# if files are on a mounted path with wrong context:
sudo chcon -R -t httpd_sys_content_t /var/www/html/ecommerce
sudo chcon -R -t httpd_sys_content_t /var/www/html/games
```

- Ensure no other service binds to port 5001: `sudo ss -tlnp | grep 5001`
- Ensure DocumentRoot content includes an index file (index.html/index.php).

---

## ✅ Acceptance criteria (how this is graded)

- `httpd` installed and service running on App Server 1
- Apache listening on port **5001**
- Directories `/var/www/html/ecommerce` and `/var/www/html/games` exist and served
- `curl http://localhost:5001/ecommerce/` returns site content on App Server 1
- `curl http://localhost:5001/games/` returns site content on App Server 1

---

## 📚 References

- Apache HTTP Server documentation: https://httpd.apache.org/
