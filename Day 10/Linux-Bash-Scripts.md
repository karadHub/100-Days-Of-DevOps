## Day 10: Website Media Backup Script

| Task details                                                                                | Solution                                                                                                                               |
| ------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Create the bash script at `/scripts/media_backup.sh` on App Server 3                        | Create `/scripts/media_backup.sh` with the provided script content and make it executable: `chmod +x /scripts/media_backup.sh`.        |
| Archive `/var/www/html/media` into `xfusioncorp_media.zip`                                  | Use `zip -r /backup/xfusioncorp_media.zip /var/www/html/media` inside the script.                                                      |
| Save the archive locally to `/backup`                                                       | Ensure the script writes to `/backup` and creates it if missing: `mkdir -p /backup`.                                                   |
| Copy the archive to Nautilus Backup Server `stbkp01.stratos.xfusioncorp.com` into `/backup` | Use `scp /backup/xfusioncorp_media.zip clint@stbkp01.stratos.xfusioncorp.com:/backup/` (script calls `scp`).                           |
| Ensure the copy does not prompt for password                                                | Configure SSH key-based auth for the script user: `ssh-keygen -t rsa -b 2048` and `ssh-copy-id clint@stbkp01.stratos.xfusioncorp.com`. |
| Script must not include `sudo`                                                              | Do permission/setup changes outside the script as admin; the script contains no `sudo`.                                                |
| `zip` package must be installed manually before running the script                          | Install manually on App Server 3: `sudo yum install zip -y` (CentOS) or `sudo apt-get install zip -y` (Debian/Ubuntu).                 |
| Ensure `/backup` exists and is writable on both hosts                                       | On each host run: `mkdir -p /backup` and (admin) `chown <user>:<group> /backup && chmod 755 /backup`.                                  |
| Make script runnable by the intended user                                                   | Run and schedule the script as the same user that has the SSH key and write access to `/backup` (do not use `sudo`).                   |
| Test the script                                                                             | Run `/scripts/media_backup.sh` as the intended user and verify `/backup/xfusioncorp_media.zip` locally and on the remote server.       |

### 🔎 Context Breakdown

- Company/lab: xFusionCorp Industries (exercise)
- App Server: App Server 3 (CentOS) — script path required: <code>/scripts/media_backup.sh</code>
- Source to archive: <code>/var/www/html/media</code>
- Local temporary backup dir: <code>/backup</code> (cleaned weekly)
- Remote backup host: <code>stbkp01.stratos.xfusioncorp.com</code>
- Remote backup dir: <code>/backup</code>

### ✅ What you will do (high-level plan)

1. Manually ensure the <code>zip</code> package is installed on App Server 3.
2. Create the script at <code>/scripts/media_backup.sh</code> that creates <code>/backup/xfusioncorp_media.zip</code> from the source directory and copies it to the remote backup server using <code>scp</code>.
3. Make the script executable.
4. Configure SSH key-based authentication for the user that will run the script so <code>scp</code> does not prompt for a password.
5. Verify the backup exists locally and on the Nautilus Backup Server.

### Script contract (inputs / outputs / error modes)

- Inputs: none (script uses fixed paths).
- Outputs: <code>/backup/xfusioncorp_media.zip</code> on App Server 3 and the same file on <code>stbkp01.stratos.xfusioncorp.com:/backup</code>.
- Success criteria: zip created successfully (exit code 0) and <code>scp</code> transfer completes without prompting for a password.
- Error modes: source dir missing, <code>zip</code> not installed, remote host unreachable, ssh key not installed.

### Edge cases to consider

- Source directory is empty or missing.
- Local <code>/backup</code> does not exist or is not writable by the user running the script.
- Remote user account or directory does not exist on backup server.
- Network interruption during transfer.

---

### Step 1 — Install required package (manual)

Ensure <code>zip</code> is installed on App Server 3 before running the script (run as an administrator outside the script):

```bash
# CentOS / RHEL
sudo yum install zip -y
# Debian/Ubuntu
sudo apt-get install zip -y
```

> Note: This installation step is intentionally outside the script per requirements.

### Step 2 — Create the script

Create the file <code>/scripts/media_backup.sh</code> with the following content. The script does not use <code>sudo</code> and assumes it's run by a user with appropriate permissions to read the source and write to <code>/backup</code>.

```bash
#!/bin/bash

# media_backup.sh
# Archives /var/www/html/media to /backup/xfusioncorp_media.zip and copies it to remote backup server

set -euo pipefail

# Configuration
SOURCE_DIR="/var/www/html/media"
ARCHIVE_NAME="xfusioncorp_media.zip"
LOCAL_BACKUP_DIR="/backup"
REMOTE_USER="clint"
REMOTE_HOST="stbkp01.stratos.xfusioncorp.com"
REMOTE_BACKUP_DIR="/backup"

# Ensure local backup dir exists and is writable by the current user
if [ ! -d "$LOCAL_BACKUP_DIR" ]; then
	echo "Local backup directory $LOCAL_BACKUP_DIR does not exist. Creating..."
	mkdir -p "$LOCAL_BACKUP_DIR" || { echo "Failed to create $LOCAL_BACKUP_DIR" >&2; exit 2; }
fi

# Verify source directory exists
if [ ! -d "$SOURCE_DIR" ]; then
	echo "Source directory $SOURCE_DIR does not exist. Aborting." >&2
	exit 3
fi

ARCHIVE_PATH="$LOCAL_BACKUP_DIR/$ARCHIVE_NAME"

echo "Creating zip archive: $ARCHIVE_PATH from $SOURCE_DIR"
# -r recurse into directories; -q quiet (optional)
zip -r "$ARCHIVE_PATH" "$SOURCE_DIR"

echo "Copying archive to remote backup server: $REMOTE_USER@$REMOTE_HOST:$REMOTE_BACKUP_DIR"
# scp will be passwordless if SSH keys are configured for the running user
scp -q "$ARCHIVE_PATH" "$REMOTE_USER@$REMOTE_HOST:$REMOTE_BACKUP_DIR/"

echo "Backup completed: local=$ARCHIVE_PATH remote=$REMOTE_HOST:$REMOTE_BACKUP_DIR/"

exit 0
```

#### Make the script executable

```bash
chmod +x /scripts/media_backup.sh
```

### Step 3 — Configure passwordless SSH for the user who will run the script

Run these commands on App Server 3 as the user who will execute the script (for example, the webapp user). This will ensure <code>scp</code> does not ask for a password.

```bash
# Generate a key pair (press Enter to accept defaults)
ssh-keygen -t rsa -b 2048

# Copy the public key to the remote backup server
# Replace 'clint' with the actual remote account you configured in the script if different.
ssh-copy-id clint@stbkp01.stratos.xfusioncorp.com
```

When prompted for the remote user's password, enter that user's password on the Nautilus Backup Server. (Do NOT hardcode or store plaintext passwords in scripts or repository files.)

> If <code>ssh-copy-id</code> is not available, copy the contents of <code>~/.ssh/id_rsa.pub</code> on App Server 3 into <code>~/.ssh/authorized_keys</code> of the remote account.

### Step 4 — Ensure /backup exists on both hosts

On App Server 3 and on <code>stbkp01.stratos.xfusioncorp.com</code> ensure the directory exists and is writable by the chosen users.

```bash
mkdir -p /backup
# set owner or permissions as required (run as an admin outside the script)
# sudo chown <user>:<group> /backup
# chmod 755 /backup
```

### Step 5 — Test the script

Run the script as the intended user (not with sudo):

```bash
/scripts/media_backup.sh
```

Expected results:

- <code>/backup/xfusioncorp_media.zip</code> exists on App Server 3.
- The same file is present in <code>/backup</code> on <code>stbkp01.stratos.xfusioncorp.com</code>.
- No password prompt should appear during the copy.

---

### Notes & gotchas

- Do not use <code>sudo</code> inside the script per requirements. Permission adjustments must be handled outside the script by an administrator.
- The script uses a fixed archive name (<code>xfusioncorp_media.zip</code>) so each run will overwrite the previous backup in <code>/backup</code>. If you need versioned or timestamped backups, update <code>ARCHIVE_NAME</code> accordingly.
- Ensure firewall/network rules allow SSH from App Server 3 to the Nautilus Backup Server.
- For automation (cron) ensure the cron job runs as the same user that has the SSH key configured.

### Requirements coverage

- Archive /var/www/html/media into <code>xfusioncorp_media.zip</code> and save to <code>/backup</code>: Done (script).
- Copy the archive to Nautilus Backup Server <code>/backup</code>: Done (scp).
- Script must not ask for password while copying: Achieved via SSH key setup instructions.
- Do not use sudo in the script: Script contains no sudo.
- Zip package must be installed manually before running script: Covered in Step 1.

---

If you want, I can also:

- Add a timestamped variant of the script so backups are not overwritten.
- Add a small systemd timer or cron example (run-as-user) to automate nightly backups.
