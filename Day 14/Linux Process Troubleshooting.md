## Day 14: Linux Process Troubleshooting

| Task                                                                                                                                                                                                                                                                | Solution                         |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------- |
| The production support team at xFusionCorp Industries reported Apache (httpd) service unavailability on one of the app servers in the Stratos DC. Identify the faulty app host and fix Apache so it is up and listening on port <code>6400</code> on all app hosts. | **How to Complete the Task**<br> |

---

### Step-by-step Troubleshooting

1. **Determine the OS and environment**

   ```bash
   cat /etc/os-release
   ```

2. **Check service status**

   ```bash
   systemctl status httpd
   ```

3. **Confirm Apache is configured to listen on port 6400**

   ```bash
   grep -n "Listen .*6400" /etc/httpd/conf/httpd.conf || grep -n 6400 /etc/httpd/conf.d/*.conf
   ```

4. **Restart the service and re-check status**

   ```bash
   sudo systemctl restart httpd
   sudo systemctl status httpd
   ```

5. **Verify TCP listener**

   ```bash
   ss -tuln | grep 6400 || netstat -luntp | grep 6400
   ```

6. **Check SELinux enforcement (if blocking port)**

   ```bash
   getenforce
   sudo setenforce 0   # temporary (not recommended for long term)
   ```

7. **Inspect logs for errors**

   ```bash
   journalctl -u httpd.service -xe
   journalctl -u httpd.service --since "2 minutes ago"
   journalctl -u httpd.service --since "10 minutes ago"
   ```

8. **Check if another process is using the port**
   ```bash
   netstat -luntp | grep 6400
   ```
   - Note the PID (e.g., `754`) and stop it:
     ```bash
     sudo kill <pid>
     sudo systemctl restart httpd
     ```

---

### Quick Command Summary (per host)

```bash
# identify OS
cat /etc/os-release

# check httpd
systemctl status httpd

# check Apache config for 6400
grep -n 6400 /etc/httpd/conf/httpd.conf || grep -n 6400 /etc/httpd/conf.d/*.conf

# restart and re-check
sudo systemctl restart httpd
sudo systemctl status httpd

# verify socket (listener)
ss -tuln | grep 6400 || netstat -luntp | grep 6400

# view journal logs
journalctl -u httpd.service -xe
journalctl -u httpd.service --since "10 minutes ago"

# if a process is using the port
# sudo kill <PID>

# if SELinux is blocking, temporarily set permissive
getenforce && sudo setenforce 0
```

---

### Notes, Tips & Explanations

- **Install Apache if missing**

  - RHEL/CentOS/Alma/Rocky:
    ```bash
    sudo yum install httpd -y
    ```
  - Debian/Ubuntu:
    ```bash
    sudo apt install apache2 -y
    ```
    (service name will be `apache2` instead of `httpd`)

- **Configure Apache to listen on port 6400**
  Add/edit directive:

  ```apache
  Listen 6400
  ```

  in `/etc/httpd/conf/httpd.conf` or a file in `/etc/httpd/conf.d/`.
  Restart Apache afterward.

- **SELinux adjustments for custom port**

  ```bash
  sudo semanage port -a -t http_port_t -p tcp 6400
  ```

  If `semanage` is missing, install:

  ```bash
  # RHEL/CentOS
  sudo yum install policycoreutils-python-utils -y

  # Debian/Ubuntu
  sudo apt install policycoreutils-python-utils -y
  ```

- **General troubleshooting flow**
  - Identify the faulty host.
  - Restart service & validate config.
  - Check SELinux.
  - Ensure no other process is blocking port.

---

### Verification (Success Criteria)

- `systemctl status httpd` → shows `Active: active (running)`.
- `ss -tuln | grep 6400` → shows `LISTEN` on `0.0.0.0:6400` (or `127.0.0.1:6400`) bound by `httpd`.
- `journalctl -u httpd.service --since "5 minutes ago"` → no fatal errors.

---

### Common Edge Cases

- **Port already in use** → Kill conflicting process or reassign ports.
- **Apache misconfiguration** → Run `apachectl configtest` or `httpd -t` to find syntax errors.
- **SELinux blocking port** → Use `semanage` to allow the port under `http_port_t`.
