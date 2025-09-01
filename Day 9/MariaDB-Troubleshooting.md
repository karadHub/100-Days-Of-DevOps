## Day 9: Fix MariaDB /run PID directory (mariadb)

| Task                                                                                                                                                                                                              | Solution                                                                                                   |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| MariaDB service is down because it cannot create its PID/var-run directory under /run (tmpfs). Create a persistent tmpfiles rule, set ownership and SELinux context, and ensure mariadb starts and stays enabled. | See detailed steps below — includes root cause, permanent fix script, apply instructions and verification. |

---

### 1. Root Cause

- /run is a tmpfs cleaned on reboot.
- MariaDB expects /run/mariadb (or /var/run/mariadb) to exist and be owned by mysql:mysql.
- Missing directory or wrong SELinux context prevents PID file creation → mariadb fails to start.

---

### 2. Permanent Fix Script

Save this as `fix_mariadb.sh` and run it as a privileged user:

```bash
#!/bin/bash
# Permanent fix for MariaDB PID directory issues
# Works on RHEL/CentOS/Fedora-based systems

set -e

echo "=== Step 1: Create /run/mariadb directory ==="
sudo mkdir -p /run/mariadb
sudo chown mysql:mysql /run/mariadb
sudo chmod 0755 /run/mariadb

echo "=== Step 2: Fix SELinux Context (if enabled) ==="
if command -v getenforce &> /dev/null && [ "$(getenforce)" != "Disabled" ]; then
    if ! command -v semanage &> /dev/null; then
        echo "Installing semanage tool..."
        sudo dnf install -y policycoreutils-python-utils || sudo yum install -y policycoreutils-python
    fi
    sudo semanage fcontext -a -t mysqld_var_run_t "/run/mariadb(/.*)?"
    sudo restorecon -Rv /run/mariadb
fi

echo "=== Step 3: Create systemd-tmpfiles rule for persistence ==="
sudo bash -c 'cat > /etc/tmpfiles.d/mariadb.conf <<EOF
d /run/mariadb 0755 mysql mysql -
EOF'
sudo systemd-tmpfiles --create /etc/tmpfiles.d/mariadb.conf

echo "=== Step 4: Reload systemd and enable MariaDB ==="
sudo systemctl daemon-reexec
sudo systemctl enable --now mariadb

echo "=== Step 5: Verify service status ==="
sudo systemctl status mariadb --no-pager
```

---

### 3. How to Apply

```bash
# create and run the fix
nano fix_mariadb.sh       # paste script above
chmod +x fix_mariadb.sh
./fix_mariadb.sh
```

---

### 4. What This Script Does

1. Creates /run/mariadb with correct ownership and permissions.
2. Applies SELinux label mysqld_var_run_t if SELinux is enabled.
3. Adds a systemd-tmpfiles rule so the directory is recreated on every boot.
4. Enables and starts mariadb, and reexecs systemd to pick up changes.

---

### 5. Verify After Reboot

After the scheduled reboot, check:

```bash
sudo reboot                # (only if you need to test immediately)
# after reboot:
sudo systemctl status mariadb --no-pager
ls -ld /run/mariadb
```

Expected:

- MariaDB active and running.
- /run/mariadb exists and is owned by mysql:mysql.

---
