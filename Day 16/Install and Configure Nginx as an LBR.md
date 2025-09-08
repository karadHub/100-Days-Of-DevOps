# Day 16 — Install and Configure Nginx as an LBR

This document shows how to configure the Nautilus HTTP Load Balancer (LBR) using Nginx. Follow the steps exactly and update only the main Nginx configuration file at `/etc/nginx/nginx.conf` as required by the task.

Checklist

- Install Nginx on the LBR server (`stlb01`)
- Configure HTTP load-balancing in `/etc/nginx/nginx.conf` using all App Servers as backends
- Do not change Apache ports on the app servers; ensure Apache is running on app servers
- Verify the site using the StaticApp button or a curl request to the LBR

Task / Solution (compact table)

| Task                                                                                                             | Solution                                                                                                                                                                                                                                      |
| ---------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Install nginx on LBR and configure HTTP load balancing using all App Servers; Edit only `/etc/nginx/nginx.conf`. | Install nginx via package manager, update `/etc/nginx/nginx.conf` to include an `upstream` block listing all app servers and a `server` block listening on port 80 that proxies to the upstream. Leave Apache ports on app servers untouched. |

---

Important constraint

- Edit only `/etc/nginx/nginx.conf` (the main Nginx config). Do not create separate files under `conf.d` or `sites-available` for this exercise.

Environment details (for reference)

- stapp01 — 172.16.238.10
- stapp02 — 172.16.238.11
- stapp03 — 172.16.238.12
- stlb01 — 172.16.238.14 (LBR)

Full solution (copy-paste safe)

1. Install Nginx on the LBR server

Debian/Ubuntu:

```bash
sudo apt update
sudo apt install -y nginx
```

RHEL/CentOS/Fedora:

```bash
sudo yum install -y nginx
```

Enable and start Nginx:

```bash
sudo systemctl enable --now nginx
```

2. Edit `/etc/nginx/nginx.conf` (only this file)

Make a backup first, then open the file for editing.

```bash
sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
sudo vi /etc/nginx/nginx.conf
```

Inside the `http { ... }` block, add an `upstream` block and a `server` block. If an `http` block doesn't exist, locate it and place the following inside it.

Example snippet to include inside `http { ... }`:

```nginx
upstream nautilus_backend {
	# round-robin (default)
	server 172.16.238.10;  # stapp01
	server 172.16.238.11;  # stapp02
	server 172.16.238.12;  # stapp03
}

server {
	listen 80;
	server_name stlb01.stratos.xfusioncorp.com;

	# Proxy all requests to the backend upstream
	location / {
		proxy_pass http://nautilus_backend;
		proxy_http_version 1.1;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header X-Forwarded-Proto $scheme;
		proxy_set_header Connection "";
		proxy_buffering off;
	}
}
```

Notes:

- Do not change Apache ports on the app servers. The upstream uses the default HTTP port (80) because the app servers' Apache is already configured to listen on that port.
- If app servers run Apache on a different port, list the port explicitly (for example `server 172.16.238.10:8080;`). The task note indicates you should not update the Apache port, so assume port 80 is correct.

3. Validate Nginx configuration and reload

```bash
sudo nginx -t
sudo systemctl reload nginx
```

4. Verify load balancing

From the LBR itself or from a jump host run:

```bash
curl -I http://stlb01.stratos.xfusioncorp.com
curl http://stlb01.stratos.xfusioncorp.com/  # repeat several times to observe round-robin behaviour
```

You can also use the StaticApp button in the lab UI if provided.

Troubleshooting

- If `nginx -t` fails: restore `/etc/nginx/nginx.conf.bak` and re-edit. Check logs at `/var/log/nginx/error.log`.
- If backend returns 502/504: ensure Apache is running on the app servers and listening on the expected port (`ss -ltnp | grep :80` or `systemctl status httpd`/`apache2`).
- Firewall: ensure LBR can reach backend servers on the Apache port and clients can reach LBR on port 80.

Requirements coverage

- Install nginx on LBR: provided
- Configure HTTP load balancing using all App Servers in `/etc/nginx/nginx.conf`: provided
- Do not change Apache ports on app servers and ensure Apache is up: instructions and checks provided
- Access site via StaticApp or curl: provided

End of Day 16.
