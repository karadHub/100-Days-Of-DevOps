## Day 12: Linux Network Services

| Task                                                                                                                                                                                              | Solution (see details below)                                                                                                                                                                                                |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Our monitoring tool reports that Apache on port <code>6200</code> on <code>stapp01</code> is not reachable. Use tools like <code>telnet</code>, <code>ss</code>/<code>netstat</code> to diagnose. | Follow the detailed step-by-step solution in the sections below to check the service, fix Listen config duplicates, confirm sockets, and open the firewall if required. Do NOT modify the existing <code>index.html</code>. |

---

### Solution — detailed steps

1. SSH to the application host:

```bash
ssh tony@stapp01
```

2. Check the httpd service status:

```bash
sudo systemctl status httpd
```

3. Inspect Apache Listen directives (look for duplicates or wrong ports):

```bash
sudo grep -n 'Listen' /etc/httpd/conf/httpd.conf
grep -R 'Listen' /etc/httpd/conf.d/
```

If you identify a duplicate Listen line for port 6200, confirm it's redundant and then remove it safely (example only):

```bash
sudo sed -i '357d' /etc/httpd/conf/httpd.conf   # only if you've confirmed line 357 is the duplicate
```

4. Restart Apache and verify it's listening on port 6200:

```bash
sudo systemctl restart httpd
sudo systemctl status httpd
ss -tuln | grep 6200
# or, if netstat is available:
sudo netstat -tuln | grep 6200
```

If netstat is not installed and you are allowed to install packages:

```bash
sudo yum install -y net-tools
```

5. Check the host firewall (iptables):

```bash
sudo iptables -L -n --line-numbers
```

If INPUT doesn't allow port 6200, add a rule near the top:

```bash
sudo iptables -I INPUT 1 -p tcp --dport 6200 -j ACCEPT
sudo iptables -L -n --line-numbers
```

Persist rules per your environment, e.g. `service iptables save` or `iptables-save > /etc/sysconfig/iptables` if required.

6. Final checks on the app host:

```bash
ss -tuln | grep 6200
sudo systemctl status httpd
```

7. From the jump host verify connectivity and fetch the page:

```bash
telnet stapp01 6200
curl http://stapp01:6200
```

---

### Quick reference commands

| Scenario                    | Command                                                                                                            |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------ | -------------------------------------------- | ---------------- |
| Check Apache status         | <code>sudo systemctl status httpd</code>                                                                           |
| Show Listen directives      | <code>sudo grep -n 'Listen' /etc/httpd/conf/httpd.conf</code> and <code>grep -R 'Listen' /etc/httpd/conf.d/</code> |
| Check listening sockets     | <code>ss -tuln                                                                                                     | grep 6200</code> or <code>sudo netstat -tuln | grep 6200</code> |
| Allow port 6200 in iptables | <code>sudo iptables -I INPUT 1 -p tcp --dport 6200 -j ACCEPT</code>                                                |

### Notes and cautions

- Do NOT modify the site's `index.html` (task requirement).
- Only remove Listen config lines after confirming they are duplicates and not required by any virtual host.
- When changing firewall rules, follow the environment's policy for persisting and ordering rules.

```markdown
# Day 12 — Linux Network Services

## Problem

Our monitoring reported Apache on port 6200 is not reachable on stapp01. The cause may be: the httpd service down, Apache bound to the wrong port, or the host firewall blocking 6200. We must diagnose and fix the issue and confirm Apache is reachable from the jump host using curl without modifying the existing index.html.

## Solution (summary)

1. SSH to the application host as tony.
2. Verify the httpd service and Apache Listen directives.
3. Fix duplicate/misconfigured Listen entries if present.
4. Ensure Apache is listening on port 6200.
5. Open the firewall (iptables) for TCP port 6200 if needed.
6. Restart httpd and verify from the jump host with curl.

## Detailed steps and commands

- Log in:

  ssh tony@stapp01

- Check httpd service status:

  sudo systemctl status httpd

- Search for Listen directives (look for duplicates or wrong ports):

  sudo grep -n "Listen" /etc/httpd/conf/httpd.conf
  grep -R "Listen" /etc/httpd/conf.d/

  If you find a duplicate Listen 6200 line (or an extra incorrect Listen) remove the duplicate line using a safe editor or sed. Example (only if you confirmed line 357 is the duplicate):

  sudo sed -i '357d' /etc/httpd/conf/httpd.conf

- Restart Apache and confirm it's listening on 6200:

  sudo systemctl restart httpd
  sudo systemctl status httpd

  Check listening sockets:

  ss -tuln | grep 6200

  # or, if netstat is preferred and installed:

  sudo netstat -tuln | grep 6200

  If netstat is not available, install net-tools (if permitted by your environment):

  sudo yum install -y net-tools

- Verify firewall rules (iptables-based firewall):

  sudo iptables -L -n

  If INPUT does not allow port 6200, add an accept rule (place it near the top so it's evaluated early):

  sudo iptables -I INPUT 1 -p tcp --dport 6200 -j ACCEPT

  List rules again to confirm:

  sudo iptables -L -n --line-numbers

  Persisting rules: If the environment requires persistent iptables rules, use the host's standard tool to save them (for example, service iptables save or iptables-save > /etc/sysconfig/iptables) — follow the site's policy for persistence.

- Final verification on the app host:

  ss -tuln | grep 6200
  sudo systemctl status httpd

- From the jump host (replace with the actual jump host):

  # test TCP connectivity

  telnet stapp01 6200

  # or curl to fetch the index page

  curl http://stapp01:6200

## Notes and cautions

- Do NOT modify or edit the existing index.html content. Changing the page may cause task failure.
- Only remove duplicate Listen lines after confirming they are duplicates and not needed for any virtual host configuration.
- When changing firewall rules, follow environment policies for persistence and ordering.

## What I changed in the repo

- Added this Day 12 write-up: `Day 12/Linux-Network-Services.md`.

That's it — follow the steps above on the target host and verify from the jump host with `curl http://stapp01:6200`.
```
