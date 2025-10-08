# Day 46: Deploy an App on Docker Containers

## 🛠️ Task

On App Server 3 in Stratos Datacenter create a docker compose file `/opt/itadmin/docker-compose.yml` (should be named exactly).
The compose should deploy two services (web and DB), and each service should deploy a container as per details below:

### Web service:

- Container name: `php_host`
- Image: `php` with any apache tag (e.g., `php:8.2-apache`)
- Map container port 80 to host port 8082
- Map `/var/www/html` volume (container) to `/var/www/html` (host)

### DB service:

- Container name: `mysql_host`
- Image: `mariadb` (preferably latest)
- Map container port 3306 to host port 3306
- Map `/var/lib/mysql` volume (container) to `/var/lib/mysql` (host)
- Set `MYSQL_DATABASE=database_host` and use any custom user (not root) with a complex password

---

## ✅ docker-compose.yml

```yaml
version: "3.8"

services:
  web:
    container_name: php_host
    image: php:8.2-apache
    ports:
      - "8082:80"
    volumes:
      - /var/www/html:/var/www/html

  db:
    container_name: mysql_host
    image: mariadb:latest
    ports:
      - "3306:3306"
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    environment:
      MYSQL_DATABASE: database_host
      MYSQL_USER: nautilus_user
      MYSQL_PASSWORD: S3cur3P@ssw0rd!
      MYSQL_ROOT_PASSWORD: throwawayrootpass
```

---

## 🔍 Key Notes

- **PHP Image**: `php:8.2-apache` is a stable tag with Apache support.
- **MariaDB Image**: `mariadb:latest` ensures you get the most recent version.
- **User Setup**: `nautilus_user` is a placeholder—feel free to change it. The password is strong and meets complexity requirements.
- **Root Password**: Required by MariaDB even if not used—set a throwaway value.

---

## 🚀 Deployment Steps

1. SSH into App Server 3.
2. Create the directory if it doesn’t exist:
   ```bash
   sudo mkdir -p /opt/itadmin
   ```
3. Paste the above YAML into `/opt/itadmin/docker-compose.yml`.
4. Run the stack:
   ```bash
   cd /opt/itadmin
   sudo docker-compose up -d
   ```
5. Test with:
   ```bash
   curl <server-ip>:8082/
   ```
