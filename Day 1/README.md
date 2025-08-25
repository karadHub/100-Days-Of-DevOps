## Day 1: Create a User with a Non-Interactive Shell

| Task                                                                                                                                                                                                                                                                                          | Solution                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| To accommodate the backup agent tool's specifications, the system admin team at xFusionCorp Industries requires the creation of a user with a non-interactive shell.<br><br>**Here's your task:**<br>- Create a user named <code>mariyam</code> with a non-interactive shell on App Server 2. | **How to Complete the Task**<br>1. SSH into App Server 2 using the provided credentials (usually <code>steve@stapp02</code>).<br>2. Create the user with a non-interactive shell like <code>/sbin/nologin</code> or <code>/usr/sbin/nologin</code>.<br><br>**Here’s a typical command:**<br><br><pre>sudo useradd mariyam -s /sbin/nologin</pre><br>Alternatively, some users prefer:<br><br><pre>sudo adduser mariyam --shell /sbin/nologin</pre><br>Both achieve the same result: creating a user who cannot log in interactively. |

---

### More ways and variations (quick reference)

| Scenario                                                   | Command                                                                                                  |
| ---------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Create with non-interactive shell (RHEL/CentOS/Alma/Rocky) | <code>sudo useradd -M -s /sbin/nologin mariyam</code>                                                    |
| Create with non-interactive shell (Debian/Ubuntu)          | <code>sudo adduser --disabled-password --gecos "" --shell /usr/sbin/nologin mariyam</code>               |
| Convert an existing user to non-interactive                | <code>sudo usermod -s /sbin/nologin mariyam</code> (use <code>/usr/sbin/nologin</code> on Debian/Ubuntu) |
| Alternative non-interactive shell                          | <code>sudo chsh -s /sbin/nologin mariyam</code> or <code>sudo usermod -s /bin/false mariyam</code>       |
| Also lock password (defense in depth)                      | <code>sudo passwd -l mariyam</code>                                                                      |

### Automation options

- Ansible

  ```yaml
  - hosts: app_server_2
  	become: true
  	tasks:
  		- name: Ensure service user exists with non-interactive shell
  			user:
  				name: mariyam
  				shell: /sbin/nologin
  				create_home: no
  				state: present
  ```

- cloud-init

  ```yaml
  #cloud-config
  users:
  	- name: mariyam
  		shell: /sbin/nologin
  		lock_passwd: true
  		create_groups: false
  ```

### Verify

```bash
getent passwd mariyam  # should show nologin or /bin/false as the shell
grep -E '^(mariyam:)' /etc/passwd
su - mariyam           # should be denied (message like: This account is currently not available.)
```

### Notes and distro differences

- Path to nologin varies:
  - RHEL/CentOS/Alma/Rocky: /sbin/nologin
  - Debian/Ubuntu: /usr/sbin/nologin
  - Minimal containers: /bin/false is commonly available
- If unsure, check with: `command -v nologin`.
- Non-interactive users are ideal for running services or tools; they should not be used for human SSH access.
