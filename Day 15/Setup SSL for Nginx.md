# Day 15 — Setup SSL for Nginx

Below is a compact Task / Solution table followed by the full, copy-pasteable solution steps.

| Task                                                                                                                                                                                                                                                                                                               | Solution                                                                                                                                                                                                                                                                                                                       |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Prepare App Server 2 to host the Nautilus application: 1) install and configure Nginx, 2) move existing self-signed cert & key from `/tmp` into a secure location and deploy in Nginx, 3) create `index.html` containing `Welcome!` under the Nginx document root, 4) verify access from the jump host via `curl`. | See the detailed steps below — install nginx, move cert/key to `/etc/ssl`, create `/etc/nginx/conf.d/nautilus_ssl.conf` to enable HTTPS, create `/usr/share/nginx/html/index.html` with `Welcome!`, validate with `nginx -t` and `curl -Ik https://<app-server-ip>/`. Troubleshooting and firewall/SELinux notes are included. |

---

## Full solution (copy-paste commands)

Prerequisites: you have sudo/root access on App Server 2 and the files `/tmp/nautilus.crt` and `/tmp/nautilus.key` exist.

1. Install Nginx

Debian/Ubuntu:

```bash
sudo apt update
sudo apt install -y nginx
```

RHEL/CentOS/Fedora:

```bash
sudo yum install -y nginx
```

Enable and start:

```bash
sudo systemctl enable --now nginx
```

2. Move cert and key to secure locations

```bash
sudo mkdir -p /etc/ssl/certs /etc/ssl/private
sudo mv /tmp/nautilus.crt /etc/ssl/certs/nautilus.crt
sudo mv /tmp/nautilus.key /etc/ssl/private/nautilus.key
sudo chown root:root /etc/ssl/private/nautilus.key /etc/ssl/certs/nautilus.crt
sudo chmod 600 /etc/ssl/private/nautilus.key
sudo chmod 644 /etc/ssl/certs/nautilus.crt
```

3. Add Nginx server block for HTTPS

```bash
sudo tee /etc/nginx/conf.d/nautilus_ssl.conf > /dev/null <<'NGINX'
server {
	listen 443 ssl;
	listen [::]:443 ssl;
	server_name _;

	ssl_certificate /etc/ssl/certs/nautilus.crt;
	ssl_certificate_key /etc/ssl/private/nautilus.key;

	root /usr/share/nginx/html;
	index index.html;

	location / {
		try_files $uri $uri/ =404;
	}
}

server {
	listen 80;
	listen [::]:80;
	server_name _;
	return 301 https://$host$request_uri;
}
NGINX
```

4. Deploy welcome page

```bash
echo 'Welcome!' | sudo tee /usr/share/nginx/html/index.html
sudo chown root:root /usr/share/nginx/html/index.html
sudo chmod 644 /usr/share/nginx/html/index.html
```

5. Validate and reload Nginx

```bash
sudo nginx -t
sudo systemctl restart nginx
```

6. Firewall / SELinux notes

firewalld (RHEL/CentOS):

```bash
sudo firewall-cmd --add-service=https --permanent
sudo firewall-cmd --reload
```

ufw (Ubuntu):

```bash
sudo ufw allow 'Nginx Full'
sudo ufw reload
```

If SELinux is enforcing:

```bash
sudo restorecon -Rv /etc/ssl
```

7. Verify from the jump host

Header-only (shows TLS handshake):

```bash
curl -Ik https://<app-server-ip>/
```

Fetch content (ignore self-signed trust):

```bash
curl -k https://<app-server-ip>/
```

Expected: `curl -Ik` returns `HTTP/1.1 200 OK` or `HTTP/2 200` and `curl -k` prints `Welcome!`.

Troubleshooting hints

- If you get SSL trust errors, that's expected for a self-signed cert; use `-k` for testing or install the cert on the client.
- If Nginx fails to start, run `sudo nginx -t` and check `/var/log/nginx/error.log`.
- If you see `403`, verify `index.html` exists and has correct permissions.
- If the site is unreachable, confirm firewall rules allow 443 and SELinux isn't denying access.

Requirements coverage

- Install and configure nginx: provided
- Move cert/key and secure permissions: provided
- Create index.html containing `Welcome!`: provided
- Verification instructions from jump host using `curl`: provided

---

End of Day 15.
