# Day 18: Configure LAMP server

## 🎯 TASK

xFusionCorp Industries is planning to host a WordPress website on their infra in Stratos Datacenter. The shared directory `/vaw/www/html` is already available and mounted on each app host under `/var/www/html`.

Perform the following:

- a. Install Apache (httpd), PHP and required PHP extensions on all app hosts.
- b. Configure Apache to listen on port **3002** for the application.
- c. Install and configure MariaDB on the DB server.
- d. Create database `kodekloud_db3` and user `kodekloud_gem` with password `ksH85UJjhb` and grant full privileges on the DB.
- e. Verify the app can connect to the database as user `kodekloud_gem` via the LBR link.

> Note: This guide assumes you have sudo privileges on target hosts. Do not reboot or change infra mounts.

---

## 🔧 Step 1 — App Hosts Setup (stapp01, stapp02, ...)

1. Install Apache and PHP with required extensions (DNF/YUM depending on distro):

```bash
# On RHEL/CentOS/Alma/Rocky using yum or dnf
sudo dnf install -y httpd php php-opcache php-gd php-curl php-mbstring php-xml php-mysqlnd wget
# or
sudo yum install -y httpd php php-mysqlnd wget
```

2. Configure Apache to listen on port 3002.

Open `/etc/httpd/conf/httpd.conf` and change the Listen directive (or add one):

```conf
# Find or add this line
Listen 3002
```

If virtual hosts are used, ensure the <VirtualHost> section includes the correct port, e.g. `<VirtualHost *:3002>`.

3. Ensure the document root is the mounted path. Confirm `/var/www/html` points to the shared mount.

```bash
ls -ld /var/www/html
```

4. Start and enable Apache:

```bash
sudo systemctl enable --now httpd
sudo systemctl restart httpd
sudo systemctl status httpd --no-pager
```

5. (Optional) Allow port 3002 in firewall (if firewalld is running):

```bash
sudo firewall-cmd --permanent --add-port=3002/tcp
sudo firewall-cmd --reload
```

---

## 🗄️ Step 2 — DB Server Setup (stdb01)

1. Install MariaDB server:

```bash
sudo yum install -y mariadb-server mariadb
```

2. Start and enable MariaDB:

```bash
sudo systemctl enable --now mariadb
sudo systemctl status mariadb --no-pager
```

3. Secure (optional) — run the secure installation script interactively if available:

```bash
sudo mysql_secure_installation
```

4. Log in as the MariaDB root user (using the unix socket):

```bash
sudo -u root mysql
```

5. Create the database and user, then grant privileges (run inside the mysql prompt):

```sql
CREATE DATABASE kodekloud_db3;
CREATE USER 'kodekloud_gem'@'%' IDENTIFIED BY 'ksH85UJjhb';
GRANT ALL PRIVILEGES ON kodekloud_db3.* TO 'kodekloud_gem'@'%';
FLUSH PRIVILEGES;
EXIT;
```

Notes:

- Allowing `'%'` lets the user connect from any host. For tighter security, replace `%` with the app hosts' IPs or hostnames.
- If your MariaDB uses `mysql_native_password` or caching_sha2_password differences, adapt the CREATE USER syntax accordingly.

6. Verify database and user:

```bash
sudo mysql -e "SHOW DATABASES LIKE 'kodekloud_db3';"
sudo mysql -e "SELECT User, Host FROM mysql.user WHERE User='kodekloud_gem';"
```

---

## ✅ Verify Application Connectivity

1. Place a simple PHP test file (example: `/var/www/html/db-test.php`) on an app host (the shared mount ensures all app hosts will serve it):

```php
<?php
$mysqli = new mysqli('stdb01', 'kodekloud_gem', 'ksH85UJjhb', 'kodekloud_db3');
if ($mysqli->connect_error) {
    die('Connect Error ('.$mysqli->connect_errno.') '.$mysqli->connect_error);
}
echo 'App is able to connect to the database using user kodekloud_gem';
$mysqli->close();
?>
```

Replace `'stdb01'` with the DB server hostname or IP reachable from the app hosts.

2. From the LBR link (App button on the top bar), open the app and navigate to `/db-test.php`. You should see the verification message.

3. Troubleshooting tips:

- Check Apache error logs: `sudo tail -n 100 /var/log/httpd/error_log`
- Check SELinux denials: `sudo ausearch -m AVC -ts recent` or `sudo journalctl -t setroubleshoot`
- Test DB connectivity from app host: `mysql -h stdb01 -u kodekloud_gem -p kodekloud_db3`

---

## 🔐 Security & Best Practices (summary)

- Use strong passwords and store them securely (secrets manager / environment variables)
- Limit DB user host binding instead of using `%` where possible
- Enable and configure firewalls and SELinux policies appropriately
- Run `mysql_secure_installation` to harden the DB server

---

## 📚 References

- MariaDB documentation: https://mariadb.org/
- Apache HTTP Server docs: https://httpd.apache.org/
