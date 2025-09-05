## Day 13: IPtables Installation And Configuration

| Task                                                                                                                                                                                                                                                                                        | Solution                   |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| The security team requires iptables to be installed on all app hosts. Apache listens on port <code>5004</code> and that port is currently open to everyone. Allow access to port 5004 only from the LBR host (example IP <code>172.16.238.14</code>) and ensure rules persist after reboot. | **Steps to implement**<br> |

1. Install iptables services on each app host:<br>

```bash
sudo yum install -y iptables-services
```

This provides both iptables and the service for persistence on RHEL/CentOS systems.

2. Apply firewall rules (replace LBR_IP with your real LBR IP):

```bash
# Define variables
LBR_IP="172.16.238.14"
PORT="5004"

# Flush existing rules (optional, use with caution)
sudo iptables -F

# Best practice: allow loopback and established connections first
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow only LBR to access the app port
sudo iptables -A INPUT -p tcp --dport ${PORT} -s ${LBR_IP} -j ACCEPT

# Drop other attempts to reach the app port
sudo iptables -A INPUT -p tcp --dport ${PORT} -j DROP

# Confirm rules
sudo iptables -L -n --line-numbers
```

3. Persist rules across reboots:

```bash
sudo service iptables save
sudo systemctl enable iptables
sudo systemctl start iptables
```

4. Verification:

```bash
# From LBR or allowed host (should succeed):
curl http://<app-host-ip>:5004

# From other hosts (should be blocked):
curl --max-time 5 http://<app-host-ip>:5004   # expect timeout or connection refused

# Confirm rules on the app host:
sudo iptables -L -n --line-numbers
```

5. Optional: Script to apply the rule on multiple hosts (idempotent if you adjust to check existing rules). Example script `iptables_lbr_rule.sh` is safe to run as root:

```bash
#!/bin/bash
LBR_IP="172.16.238.14"
PORT="5004"

echo "Applying iptables rules to restrict port $PORT access..."

# Flush existing rules (optional; comment out if you prefer incremental changes)
iptables -F

iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport $PORT -s $LBR_IP -j ACCEPT
iptables -A INPUT -p tcp --dport $PORT -j DROP

service iptables save
systemctl enable iptables
systemctl start iptables

echo "Firewall rules applied and persisted."
```

Notes and cautions:

- Replace <code>172.16.238.14</code> with the actual LBR IP(s). If there are multiple LBRs add multiple ACCEPT rules before the DROP.
- Flushing rules (`iptables -F`) deletes all existing rules — use only when you understand the current firewall policy.
- If your environment uses firewalld, consider using firewalld or mapping rules there instead of iptables-services.
- If you need to make rules idempotent for automation (Ansible), prefer managing the `/etc/sysconfig/iptables` file or use the iptables module in Ansible.
