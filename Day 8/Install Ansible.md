## Day 8: Install Ansible (pip3, global)

| Task                                                                                                                                                                                                                                                    | Solution                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The Nautilus DevOps team will use the jump host as the Ansible controller. Install Ansible version <code>4.7.0</code> on the jump host using <code>pip3</code> only, and make sure the <code>ansible</code> binary is available globally for all users. | 🚀 Steps to install Ansible 4.7.0 globally via pip3<br><br>1. Install Python 3 and pip3 (RHEL/CentOS):<br><br><pre>sudo yum install -y python3</pre><br>Verify pip3 is available:<br><pre>pip3 --version</pre><br><br>2. Install Ansible 4.7.0 into a global prefix so binaries land in <code>/usr/local/bin</code> (accessible by all users):<br><br><pre>sudo pip3 install ansible==4.7.0 --prefix=/usr/local</pre><br><br>3. Confirm installation:<br><br><pre>ansible --version</pre><br>The output should include an Ansible core version (for 4.7.0 the bundled core is 2.11.12). Also verify the pip package:<br><br><pre>pip3 show ansible |

# Look for: Version: 4.7.0</pre><br><br>4. If <code>ansible</code> is not found, add <code>/usr/local/bin</code> to the system PATH (system-wide):<br><br><pre>sudo sh -c 'echo "export PATH=$PATH:/usr/local/bin" >> /etc/profile'

source /etc/profile</pre><br><br>🧪 Optional quick connectivity test (example inventory and ping):<br><br><pre># /etc/ansible/hosts
[nautilus]
172.16.238.10
172.16.238.11
172.16.238.12

ansible all -m ping</pre> |

---

### More ways and variations

- Install using a virtual environment (isolated):

<pre>python3 -m venv ~/ansible-venv
source ~/ansible-venv/bin/activate
pip install ansible==4.7.0</pre>

- On Debian/Ubuntu you can install dependencies then use pip3:

<pre>sudo apt-get update
sudo apt-get install -y python3 python3-pip
sudo pip3 install ansible==4.7.0 --prefix=/usr/local</pre>

- Use the OS package (not requested here) for quick installs: <code>yum install -y ansible</code> or <code>apt-get install -y ansible</code> (may not match 4.7.0).

### Verify

```bash
ansible --version
pip3 show ansible | grep Version
which ansible || echo "/usr/local/bin not in PATH"
```

### Notes

- Installing with <code>--prefix=/usr/local</code> places executable wrappers into <code>/usr/local/bin</code>. Ensure that path is in <code>/etc/profile</code> for all users.
- Installing as root via <code>sudo pip3 install</code> writes to the global site-packages; prefer a prefix or virtualenv to avoid conflicts.
- Ansible 4.7.0 bundles core 2.11.12; the output of <code>ansible --version</code> shows both.
