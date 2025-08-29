## Day 6: Create a Cron Job (Test cronie and cron entry)

| Task                                                                                                                                                                                                                                                                                                                                                                                                                                    | Solution                                                                                                                                          |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| The Nautilus system admins team has prepared scripts to automate several day-to-day tasks. They want them deployed on all app servers in Stratos DC on a schedule. Before that they need to test similar functionality with a sample cron job. Perform the steps below:<br>a. Install cronie package on all Nautilus app servers and start crond service.<br>b. Add a cron `*/5 * * * * echo hello > /tmp/cron_text` for the root user. | Below is a simple scripted solution you can run on a RHEL/CentOS style server. Adjust package manager commands for Debian/Ubuntu (apt) if needed. |

Create the helper script `setup_cron.sh` (example):

```bash
#!/bin/bash

# Install cronie (RHEL/CentOS/Amazon Linux)
sudo yum install -y cronie

# Start and enable crond
sudo systemctl start crond
sudo systemctl enable crond

# Show status
sudo systemctl status crond --no-pager

# Install the cron entry for root (preserve existing crontab)
(crontab -l 2>/dev/null; echo "*/5 * * * * echo hello > /tmp/cron_text") | crontab -

# Wait a short time and show the created file once the job has run
sleep 10

echo "--- /tmp/cron_text contents ---"
sudo cat /tmp/cron_text 2>/dev/null || echo "(file not present yet)"

echo "--- listing /tmp ---"
ls -ltr /tmp | head -n 20

```

Make the script executable and run it:

```bash
chmod +x setup_cron.sh
./setup_cron.sh
```

Notes:

- On Debian/Ubuntu use `sudo apt-get install -y cron` and `sudo systemctl start cron`.
- The cron line writes the string "hello" to `/tmp/cron_text` every 5 minutes. The script above waits 10 seconds and then attempts to show the file; you may need to wait up to 5 minutes for the first run.
- To inspect the root crontab manually: `sudo crontab -l`.
- To remove the test entry: `(crontab -l | grep -v "echo hello > /tmp/cron_text") | crontab -` as root.

Verification:

```bash
sudo systemctl status crond --no-pager
sudo crontab -l
tail -n +1 /tmp/cron_text
```

---

This completes the Day 6 sample cron job setup.
