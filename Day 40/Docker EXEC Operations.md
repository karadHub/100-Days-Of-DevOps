# Day 40: Docker EXEC Operations

## Task

A Nautilus DevOps team member started configuring services inside a running container named `kkloud` on App Server 1 (Stratos Datacenter). You need to finish the remaining work:

a. Install `apache2` in the `kkloud` container using `apt`.
b. Configure Apache to listen on port `8085` (do not bind to a specific IP — it should listen on all interfaces).
c. Make sure Apache service is up and running inside the container and keep the container running.

---

## Solution

### Overview of steps

1. SSH into App Server 1
2. Use `docker exec` to run commands inside the `kkloud` container
3. Install `apache2` with `apt`
4. Update Apache configuration to listen on `8085`
5. Restart Apache and verify it's listening
6. Keep the container running

---

### Step-by-step commands

1. SSH to App Server 1 (example):

```bash
ssh tony@stapp01
```

2. Confirm the `kkloud` container is running:

```bash
docker ps --filter "name=kkloud"
```

3. Install Apache inside the container (run non-interactive commands with `docker exec`):

```bash
# update package lists
docker exec -it kkloud bash -c "apt-get update -y"

# install apache2
docker exec -it kkloud bash -c "DEBIAN_FRONTEND=noninteractive apt-get install -y apache2"
```

Notes:

- `-it` gives an interactive TTY; it's safe for these commands. If your environment prefers non-interactive runs, you can omit `-t`.
- `DEBIAN_FRONTEND=noninteractive` prevents apt from asking interactive questions.

4. Configure Apache to listen on port 8085. Modify `/etc/apache2/ports.conf` and virtual host configuration.

```bash
# update ports.conf
docker exec kkloud bash -c "sed -i 's/Listen 80/Listen 8085/' /etc/apache2/ports.conf"

# update default virtual host
docker exec kkloud bash -c "sed -i 's/<VirtualHost \*:80>/<VirtualHost *:8085>/g' /etc/apache2/sites-enabled/000-default.conf"
```

These commands replace the default `80` with `8085` while keeping Apache listening on all interfaces `*` (not bound to a specific IP).

5. Restart Apache in the container and verify the service is active and listening on 8085:

```bash
# restart apache
docker exec kkloud bash -c "service apache2 restart"

# check service status
docker exec kkloud bash -c "service apache2 status --no-pager"

# verify listening port (inside container)
docker exec kkloud bash -c "ss -tuln | grep 8085 || netstat -tuln | grep 8085"
```

Expected: an output line showing LISTEN on 0.0.0.0:8085 or :::8085 indicating Apache listens on all interfaces.

6. Ensure the container remains running:

- If the container was started with `--rm` or was intended to run interactive processes only, consider restarting it without `--rm` and with a long-running command (e.g., systemd or a foreground process). In most cases, `kkloud` will remain running if it was already created that way.

- Confirm from the host:

```bash
docker ps | grep kkloud
```

If `kkloud` is not running, start it properly (example):

```bash
# start container (example, adjust options as needed)
docker start kkloud
```

---

### Verification (Quick checklist)

- [ ] `apache2` package is installed inside `kkloud`
- [ ] `/etc/apache2/ports.conf` contains `Listen 8085`
- [ ] VirtualHost in `/etc/apache2/sites-enabled/000-default.conf` uses `*:8085`
- [ ] Apache process is running and listening on port `8085`
- [ ] Container `kkloud` remains in running state

---

### Troubleshooting tips

- If `apt-get` fails due to network/DNS issues, verify container network settings and resolve DNS (e.g., ensure `/etc/resolv.conf` is valid inside the container).
- If `service apache2 restart` fails, check logs:
  ```bash
  docker exec kkloud bash -c "journalctl -u apache2 --no-pager || tail -n 200 /var/log/apache2/error.log"
  ```
- If `ss` or `netstat` are not available, you can check Apache PID and process details:
  ```bash
  docker exec kkloud bash -c "ps aux | grep apache2"
  ```

---

This completes the required steps to install and configure Apache on port 8085 inside the `kkloud` container. Finally, update the main `README.md` to list Day 40.
