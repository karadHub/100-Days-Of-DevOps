# Day 68 — Set Up Jenkins Server 🔧

[![Jenkins](https://img.shields.io/badge/-Jenkins-D24939?style=flat&logo=jenkins&logoColor=white)](https://www.jenkins.io/)
[![CI/CD](https://img.shields.io/badge/-CI%2FCD-4A90E2?style=flat&logo=circleci&logoColor=white)](https://en.wikipedia.org/wiki/CI/CD)
[![Linux](https://img.shields.io/badge/-Linux-FCC624?style=flat&logo=linux&logoColor=black)](https://www.linux.org/)

## 📋 Task Overview

The DevOps team at xFusionCorp Industries is initiating the setup of CI/CD pipelines and has decided to utilize Jenkins as their automation server. This task involves installing Jenkins on a dedicated server, configuring it, and creating an admin user.

### 🎯 Requirements

1. **Install Jenkins** on the `jenkins` server using the `yum` utility only
2. **Start Jenkins service** and ensure it's enabled on boot
3. **Create admin user** with the following credentials:
   - **Username**: `theadmin`
   - **Password**: `Adm!n321`
   - **Full Name**: `Ravi`
   - **Email**: `ravi@jenkins.stratos.xfusioncorp.com`

### 📝 Access Information

- **Jump Host**: SSH using root user (password: `S3curePass`)
- **Jenkins Server**: Accessible from jump host as `jenkins`
- **Jenkins UI**: Access via Jenkins button on top bar after installation

---

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                     Jenkins Architecture                     │
└─────────────────────────────────────────────────────────────┘

    ┌──────────────┐           ┌──────────────────────┐
    │  Jump Host   │    SSH    │   Jenkins Server     │
    │              │──────────▶│                      │
    │  root user   │           │  - Jenkins Master    │
    └──────────────┘           │  - Port 8080         │
                               │  - Java Runtime      │
                               │  - systemd service   │
                               └──────────────────────┘
                                          │
                                          │ HTTP
                                          ▼
                               ┌──────────────────────┐
                               │    Jenkins Web UI    │
                               │                      │
                               │  - Initial Setup     │
                               │  - Admin Creation    │
                               │  - Plugin Install    │
                               └──────────────────────┘

Components:
├── Jenkins Master: Main server coordinating all tasks
├── Java 11: Runtime environment for Jenkins
├── systemd: Service management for Jenkins daemon
└── Web UI: Browser-based administration interface
```

---

## 🚀 Complete Solution

### Step 1: Connect to the Jenkins Server

#### 1.1 SSH to Jump Host

```bash
# Connect to jump host
ssh root@<jump_host_ip>
# When prompted, enter password: S3curePass
```

#### 1.2 Connect to Jenkins Server

```bash
# From jump host, SSH to Jenkins server
ssh root@jenkins
```

**Expected Output:**

```
[root@jenkins ~]#
```

---

### Step 2: Prepare System for Jenkins Installation

#### 2.1 Update System Packages

```bash
# Update all packages to latest version
yum update -y
```

#### 2.2 Install Required Dependencies

```bash
# Install wget if not available
yum install wget -y

# Install curl for troubleshooting
yum install curl -y
```

---

### Step 3: Install Java (Jenkins Prerequisite)

Jenkins requires Java to run. We'll install OpenJDK 11.

#### 3.1 Install Java 11

```bash
# Install Java 11 OpenJDK
yum install java-11-openjdk java-11-openjdk-devel -y
```

#### 3.2 Verify Java Installation

```bash
# Check Java version
java -version
```

**Expected Output:**

```
openjdk version "11.0.x" 2023-xx-xx LTS
OpenJDK Runtime Environment 18.9 (build 11.0.x+x-LTS)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.x+x-LTS, mixed mode, sharing)
```

#### 3.3 Set JAVA_HOME (Optional but Recommended)

```bash
# Find Java installation path
java_path=$(alternatives --display java | grep 'link currently points to' | awk '{print $5}')
java_home=$(dirname $(dirname $java_path))

# Set JAVA_HOME environment variable
echo "export JAVA_HOME=$java_home" >> /etc/profile.d/java.sh
echo "export PATH=\$JAVA_HOME/bin:\$PATH" >> /etc/profile.d/java.sh

# Apply changes
source /etc/profile.d/java.sh

# Verify
echo $JAVA_HOME
```

---

### Step 4: Add Jenkins Repository

#### 4.1 Download Jenkins Repository Configuration

```bash
# Add Jenkins stable repository
wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
```

**Expected Output:**

```
--2025-11-04 10:30:00--  https://pkg.jenkins.io/redhat-stable/jenkins.repo
Resolving pkg.jenkins.io... 151.101.xxx.xxx
Connecting to pkg.jenkins.io... connected.
HTTP request sent, awaiting response... 200 OK
Length: xxx
Saving to: '/etc/yum.repos.d/jenkins.repo'

/etc/yum.repos.d/jenkins.repo    100%[====>]   xxx  --.-KB/s    in 0.001s

2025-11-04 10:30:00 (xxx KB/s) - '/etc/yum.repos.d/jenkins.repo' saved
```

#### 4.2 Import Jenkins GPG Key

```bash
# Import Jenkins repository GPG key for package verification
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
```

#### 4.3 Verify Repository Configuration

```bash
# Check if Jenkins repository is configured
cat /etc/yum.repos.d/jenkins.repo
```

**Expected Output:**

```
[jenkins]
name=Jenkins-stable
baseurl=https://pkg.jenkins.io/redhat-stable
gpgcheck=1
```

---

### Step 5: Install Jenkins

#### 5.1 Install Jenkins Package

```bash
# Install Jenkins using yum
yum install jenkins -y
```

**Expected Output:**

```
Loaded plugins: fastestmirror, ovl
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package jenkins.noarch 0:x.xxx.x-x.x will be installed
...
Installed:
  jenkins.noarch 0:x.xxx.x-x.x

Complete!
```

#### 5.2 Verify Jenkins Installation

```bash
# Check Jenkins version
rpm -qa | grep jenkins

# Check Jenkins files
rpm -ql jenkins | head -20
```

**Key Jenkins Files:**

- `/usr/bin/jenkins`: Jenkins startup script
- `/etc/sysconfig/jenkins`: Jenkins configuration file
- `/var/lib/jenkins`: Jenkins home directory
- `/var/log/jenkins`: Jenkins log directory

---

### Step 6: Configure Jenkins Service

#### 6.1 Check Jenkins Service Status

```bash
# Check if Jenkins service is installed
systemctl status jenkins
```

**Expected Output (Inactive):**

```
● jenkins.service - Jenkins Continuous Integration Server
   Loaded: loaded (/usr/lib/systemd/system/jenkins.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
```

#### 6.2 Enable Jenkins Service

```bash
# Enable Jenkins to start on boot
systemctl enable jenkins
```

**Expected Output:**

```
Created symlink from /etc/systemd/system/multi-user.target.wants/jenkins.service to /usr/lib/systemd/system/jenkins.service.
```

#### 6.3 Start Jenkins Service

```bash
# Start Jenkins service
systemctl start jenkins
```

#### 6.4 Verify Jenkins is Running

```bash
# Check service status
systemctl status jenkins
```

**Expected Output:**

```
● jenkins.service - Jenkins Continuous Integration Server
   Loaded: loaded (/usr/lib/systemd/system/jenkins.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2025-11-04 10:35:00 UTC; 10s ago
 Main PID: 12345 (java)
   CGroup: /docker/xxxxx/system.slice/jenkins.service
           └─12345 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war...

Nov 04 10:35:00 jenkins systemd[1]: Starting Jenkins Continuous Integration Server...
Nov 04 10:35:00 jenkins jenkins[12345]: Running from: /usr/share/java/jenkins.war
Nov 04 10:35:00 jenkins systemd[1]: Started Jenkins Continuous Integration Server.
```

#### 6.5 Check Jenkins Port

```bash
# Verify Jenkins is listening on port 8080
netstat -tulpn | grep 8080
# OR
ss -tulpn | grep 8080
```

**Expected Output:**

```
tcp6       0      0 :::8080                 :::*                    LISTEN      12345/java
```

---

### Step 7: Retrieve Initial Admin Password

#### 7.1 Get Initial Admin Password

```bash
# Display the initial admin password
cat /var/lib/jenkins/secrets/initialAdminPassword
```

**Expected Output:**

```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

**Important:** Copy this password - you'll need it for the initial Jenkins setup.

#### 7.2 Alternative: Check Jenkins Logs

```bash
# View Jenkins startup logs
tail -50 /var/log/jenkins/jenkins.log
```

Look for lines containing the initial admin password.

---

### Step 8: Access Jenkins Web UI

#### 8.1 Open Jenkins in Browser

1. **Click the "Jenkins" button** on the top bar of your lab environment

   - This will open Jenkins UI in a new tab
   - URL will be similar to: `http://jenkins:8080`

2. **Alternative: Direct Access**

   ```bash
   # If you need to access via port forwarding or direct URL
   # Get server IP
   hostname -I

   # Access via: http://<server_ip>:8080
   ```

#### 8.2 Unlock Jenkins Screen

You'll see the "Unlock Jenkins" screen.

1. **Enter the initial admin password** (retrieved in Step 7.1)
2. **Click "Continue"**

---

### Step 9: Initial Jenkins Setup

#### 9.1 Customize Jenkins - Plugin Installation

You'll be presented with two options:

1. **Install suggested plugins** (Recommended for beginners)
2. **Select plugins to install** (Advanced users)

**Choose:** "Install suggested plugins"

**Plugins Being Installed:**

- Git plugin
- Pipeline plugin
- Credentials plugin
- SSH Slaves plugin
- Matrix Authorization Strategy
- And many more...

**Wait** for all plugins to install (usually 2-5 minutes).

#### 9.2 Handling Plugin Installation Issues

If any plugins fail to install:

```bash
# Check Jenkins logs
tail -f /var/log/jenkins/jenkins.log

# Restart Jenkins if needed
systemctl restart jenkins
```

Then refresh the browser and retry.

---

### Step 10: Create Admin User

#### 10.1 Create First Admin User

After plugin installation, you'll see "Create First Admin User" screen.

**Fill in the following details:**

| Field            | Value                                  |
| ---------------- | -------------------------------------- |
| Username         | `theadmin`                             |
| Password         | `Adm!n321`                             |
| Confirm Password | `Adm!n321`                             |
| Full Name        | `Ravi`                                 |
| Email Address    | `ravi@jenkins.stratos.xfusioncorp.com` |

**Click:** "Save and Continue"

#### 10.2 Instance Configuration

You'll see "Instance Configuration" screen with Jenkins URL.

**Default URL:** `http://jenkins:8080/`

**Click:** "Save and Finish"

#### 10.3 Jenkins is Ready!

You'll see "Jenkins is ready!" screen.

**Click:** "Start using Jenkins"

---

### Step 11: Verify Jenkins Setup

#### 11.1 Login with Admin Credentials

1. **Enter credentials:**

   - **Username:** `theadmin`
   - **Password:** `Adm!n321`

2. **Click:** "Sign in"

#### 11.2 Verify Dashboard Access

You should now see the Jenkins Dashboard with:

- ✅ Welcome message
- ✅ Create new jobs option
- ✅ Manage Jenkins link
- ✅ Admin user logged in (top right corner)

#### 11.3 Check User Configuration

1. **Click on "theadmin"** (top right corner)
2. **Click "Configure"**
3. **Verify:**
   - Full Name: `Ravi`
   - Email: `ravi@jenkins.stratos.xfusioncorp.com`

---

## 🔍 Verification Commands

### Verify Jenkins Service

```bash
# Check service status
systemctl status jenkins

# Check if enabled on boot
systemctl is-enabled jenkins

# Check service uptime
systemctl show jenkins --property=ActiveEnterTimestamp
```

### Verify Jenkins Process

```bash
# Check Jenkins process
ps aux | grep jenkins

# Check Java process for Jenkins
pgrep -f jenkins

# Check Jenkins port
netstat -tulpn | grep 8080
```

### Verify Jenkins Installation

```bash
# Check Jenkins version
java -jar /usr/share/java/jenkins.war --version

# Check installed packages
rpm -qa | grep jenkins

# Check Jenkins home directory
ls -la /var/lib/jenkins/
```

### Verify Jenkins Configuration

```bash
# Check Jenkins configuration file
cat /etc/sysconfig/jenkins | grep -v '^#' | grep -v '^$'

# Check Jenkins user
id jenkins

# Check Jenkins home permissions
ls -ld /var/lib/jenkins/
```

### Verify Jenkins Logs

```bash
# Check recent logs
tail -50 /var/log/jenkins/jenkins.log

# Follow live logs
tail -f /var/log/jenkins/jenkins.log

# Check for errors
grep -i error /var/log/jenkins/jenkins.log

# Check systemd journal
journalctl -u jenkins -n 50
```

---

## 🧪 Testing Jenkins Setup

### Test 1: Create a Test Job

1. **Click "New Item"** on Jenkins Dashboard
2. **Enter job name:** `test-job`
3. **Select:** "Freestyle project"
4. **Click:** "OK"
5. **In Build section:**
   - Click "Add build step"
   - Select "Execute shell"
   - Enter command: `echo "Hello from Jenkins!"`
6. **Click:** "Save"
7. **Click:** "Build Now"
8. **Check Console Output** - should show "Hello from Jenkins!"

### Test 2: Verify System Information

1. **Go to:** "Manage Jenkins" → "System Information"
2. **Verify:**
   - Java version (should be Java 11)
   - Jenkins version
   - Operating system information
   - Environment variables

### Test 3: Check Plugin Installation

1. **Go to:** "Manage Jenkins" → "Manage Plugins"
2. **Click:** "Installed" tab
3. **Verify** key plugins are installed:
   - Git plugin
   - Pipeline
   - Credentials
   - SSH Build Agents

### Test 4: Verify User Management

1. **Go to:** "Manage Jenkins" → "Manage Users"
2. **Verify** "theadmin" user exists
3. **Click** on "theadmin" to view details
4. **Check** email and full name are correct

---

## 🛠️ Troubleshooting Guide

### Issue 1: Jenkins Service Fails to Start

**Symptom:**

```bash
systemctl start jenkins
# Job for jenkins.service failed...
```

**Solution:**

```bash
# Check detailed error
systemctl status jenkins -l
journalctl -xe -u jenkins

# Common fixes:

# 1. Check Java installation
java -version

# 2. Check port 8080 is not in use
netstat -tulpn | grep 8080
# If something else is using 8080, change Jenkins port:
sed -i 's/JENKINS_PORT="8080"/JENKINS_PORT="8081"/g' /etc/sysconfig/jenkins
systemctl restart jenkins

# 3. Check Jenkins home directory permissions
chown -R jenkins:jenkins /var/lib/jenkins
chmod -R 755 /var/lib/jenkins

# 4. Check disk space
df -h

# 5. Increase startup timeout
mkdir -p /etc/systemd/system/jenkins.service.d
cat > /etc/systemd/system/jenkins.service.d/override.conf << EOF
[Service]
TimeoutStartSec=300
EOF
systemctl daemon-reload
systemctl restart jenkins
```

### Issue 2: Timeout When Starting Jenkins

**Symptom:**

```
A start job is running for Jenkins... (timeout)
```

**Solution:**

```bash
# Method 1: Increase timeout
systemctl edit jenkins
# Add:
[Service]
TimeoutStartSec=300

# Save and exit, then:
systemctl daemon-reload
systemctl restart jenkins

# Method 2: Check what's blocking
tail -f /var/log/jenkins/jenkins.log

# Method 3: Disable IPv6 if causing issues
echo "JENKINS_JAVA_OPTIONS=\"-Djava.net.preferIPv4Stack=true\"" >> /etc/sysconfig/jenkins
systemctl restart jenkins
```

### Issue 3: Cannot Access Jenkins Web UI

**Symptom:**
Browser shows "Connection refused" or "Unable to connect"

**Solution:**

```bash
# 1. Verify Jenkins is running
systemctl status jenkins

# 2. Check Jenkins port
netstat -tulpn | grep 8080

# 3. Check firewall (if enabled)
firewall-cmd --list-ports
# Add port if needed:
firewall-cmd --permanent --add-port=8080/tcp
firewall-cmd --reload

# 4. Check SELinux (if enabled)
getenforce
# If Enforcing and blocking:
semanage port -a -t http_port_t -p tcp 8080
# Or temporarily:
setenforce 0

# 5. Test local connectivity
curl http://localhost:8080

# 6. Check Jenkins logs
tail -50 /var/log/jenkins/jenkins.log
```

### Issue 4: Initial Admin Password Not Found

**Symptom:**
Cannot find `/var/lib/jenkins/secrets/initialAdminPassword`

**Solution:**

```bash
# 1. Wait for Jenkins to fully start
systemctl status jenkins
# Ensure it shows "active (running)" for at least 30 seconds

# 2. Check if file exists
ls -la /var/lib/jenkins/secrets/

# 3. Check Jenkins logs for password
grep -i password /var/log/jenkins/jenkins.log

# 4. Restart Jenkins and wait
systemctl restart jenkins
sleep 60
cat /var/lib/jenkins/secrets/initialAdminPassword

# 5. If still not found, reset Jenkins
systemctl stop jenkins
rm -rf /var/lib/jenkins/secrets/
systemctl start jenkins
# Wait 60 seconds then check again
```

### Issue 5: Plugin Installation Fails

**Symptom:**
Plugins fail to install during setup wizard

**Solution:**

```bash
# 1. Check internet connectivity
curl -I https://updates.jenkins.io

# 2. Configure proxy if needed (in Jenkins UI)
# Manage Jenkins → Manage Plugins → Advanced → HTTP Proxy

# 3. Or skip plugins and install later
# Click "Skip and continue as admin"
# Then install plugins from Manage Jenkins → Manage Plugins

# 4. Manual plugin installation
cd /var/lib/jenkins/plugins
wget https://updates.jenkins.io/latest/[plugin-name].hpi
chown jenkins:jenkins *.hpi
systemctl restart jenkins
```

### Issue 6: Admin User Creation Fails

**Symptom:**
Cannot create admin user, form errors

**Solution:**

1. **Check password requirements:**

   - Ensure `Adm!n321` meets complexity requirements
   - If Jenkins enforces different rules, adjust in security settings

2. **Check email format:**

   - Ensure email is valid: `ravi@jenkins.stratos.xfusioncorp.com`

3. **Browser issues:**

   - Clear browser cache
   - Try different browser
   - Disable browser extensions

4. **Skip and create manually:**
   ```bash
   # Continue as admin, then:
   # Manage Jenkins → Manage Users → Create User
   ```

### Issue 7: Java Version Issues

**Symptom:**

```
Jenkins requires Java 11 or newer
```

**Solution:**

```bash
# 1. Check installed Java versions
yum list installed | grep java

# 2. Remove old Java if present
yum remove java-1.8.0-openjdk -y

# 3. Install Java 11
yum install java-11-openjdk java-11-openjdk-devel -y

# 4. Set Java 11 as default
alternatives --config java
# Select Java 11

# 5. Verify
java -version

# 6. Restart Jenkins
systemctl restart jenkins
```

### Issue 8: Permission Denied Errors

**Symptom:**

```
Permission denied errors in /var/lib/jenkins/
```

**Solution:**

```bash
# Fix Jenkins home permissions
chown -R jenkins:jenkins /var/lib/jenkins
chmod -R 755 /var/lib/jenkins

# Fix log directory
chown -R jenkins:jenkins /var/log/jenkins
chmod -R 755 /var/log/jenkins

# Fix cache directory
chown -R jenkins:jenkins /var/cache/jenkins

# Restart Jenkins
systemctl restart jenkins
```

---

## 🔒 Security Best Practices

### 1. Change Default Port (Optional)

```bash
# Edit Jenkins configuration
vi /etc/sysconfig/jenkins

# Change:
JENKINS_PORT="8080"
# To:
JENKINS_PORT="8090"

# Restart
systemctl restart jenkins
```

### 2. Configure Security Realm

1. **Navigate to:** "Manage Jenkins" → "Configure Global Security"
2. **Enable:** "Jenkins' own user database"
3. **Disable:** "Allow users to sign up"
4. **Enable:** "Logged-in users can do anything"
5. **Later configure:** Matrix-based security for fine-grained permissions

### 3. Enable CSRF Protection

1. **Navigate to:** "Manage Jenkins" → "Configure Global Security"
2. **Check:** "Prevent Cross Site Request Forgery exploits"
3. **Save**

### 4. Set Up HTTPS (Production)

```bash
# Generate keystore
keytool -genkey -keyalg RSA -alias jenkins -keystore /var/lib/jenkins/jenkins.jks -storepass changeit -keysize 2048

# Edit Jenkins configuration
vi /etc/sysconfig/jenkins

# Add:
JENKINS_HTTPS_PORT="8443"
JENKINS_HTTPS_KEYSTORE="/var/lib/jenkins/jenkins.jks"
JENKINS_HTTPS_KEYSTORE_PASSWORD="changeit"

# Restart
systemctl restart jenkins
```

### 5. Regular Updates

```bash
# Check for Jenkins updates regularly
yum check-update jenkins

# Update Jenkins
yum update jenkins -y
systemctl restart jenkins
```

### 6. Backup Strategy

```bash
# Create backup directory
mkdir -p /backup/jenkins

# Backup Jenkins home (excluding workspace)
tar -czf /backup/jenkins/jenkins-backup-$(date +%Y%m%d).tar.gz \
    --exclude='/var/lib/jenkins/workspace/*' \
    /var/lib/jenkins/

# Automate with cron
echo "0 2 * * * root tar -czf /backup/jenkins/jenkins-backup-\$(date +\%Y\%m\%d).tar.gz --exclude='/var/lib/jenkins/workspace/*' /var/lib/jenkins/" >> /etc/crontab
```

---

## 📊 Post-Installation Configuration

### Configure System Settings

```bash
# Set Jenkins URL
# Manage Jenkins → Configure System → Jenkins Location
# Jenkins URL: http://jenkins:8080/

# Set Admin Email
# System Admin e-mail address: ravi@jenkins.stratos.xfusioncorp.com
```

### Configure Build Tools

1. **JDK Configuration:**

   - Manage Jenkins → Global Tool Configuration → JDK
   - Add JDK → Name: `Java11`
   - JAVA_HOME: `/usr/lib/jvm/java-11-openjdk`

2. **Git Configuration:**
   - Manage Jenkins → Global Tool Configuration → Git
   - Name: `Default`
   - Path: `/usr/bin/git` (install git if needed: `yum install git -y`)

### Configure Build Executors

```bash
# Manage Jenkins → Configure System → # of executors
# Set based on CPU cores (typically 2x cores)
grep -c processor /proc/cpuinfo
# Set accordingly (e.g., 2 for single core)
```

---

## 🧹 Cleanup and Maintenance

### Stop Jenkins

```bash
# Stop service
systemctl stop jenkins

# Disable from boot
systemctl disable jenkins
```

### Remove Jenkins (If Needed)

```bash
# Stop service
systemctl stop jenkins
systemctl disable jenkins

# Remove package
yum remove jenkins -y

# Remove repository
rm -f /etc/yum.repos.d/jenkins.repo

# Remove home directory (CAUTION: Deletes all data)
rm -rf /var/lib/jenkins

# Remove logs
rm -rf /var/log/jenkins
```

### Clean Workspace

```bash
# Remove old builds (via UI)
# Job → Configure → Discard old builds
# Days to keep: 7
# Max # of builds: 10

# Or manually:
rm -rf /var/lib/jenkins/jobs/*/builds/*

# Clean workspace
rm -rf /var/lib/jenkins/workspace/*
```

---

## 📚 Key Takeaways

### ✅ What We Learned

1. **Jenkins Installation:**

   - Jenkins requires Java 11+ runtime
   - Installed via yum package manager
   - Uses systemd for service management

2. **Repository Management:**

   - Added official Jenkins repository
   - Imported GPG key for package verification
   - Configured yum for Jenkins updates

3. **Service Configuration:**

   - Enabled Jenkins service for auto-start
   - Configured systemd timeout settings
   - Managed service lifecycle (start/stop/restart)

4. **Initial Setup:**

   - Retrieved initial admin password
   - Installed recommended plugins
   - Created custom admin user
   - Configured Jenkins URL

5. **Security Basics:**
   - User authentication setup
   - Email configuration
   - CSRF protection awareness

### 🎓 Skills Developed

- ✅ Linux package management with yum
- ✅ Java environment setup
- ✅ systemd service management
- ✅ Repository configuration
- ✅ Web application deployment
- ✅ User management and authentication
- ✅ Basic CI/CD concepts
- ✅ Troubleshooting service startup issues

### 🚀 Next Steps

1. **Configure Jenkins Agents:**

   - Set up distributed builds
   - Configure SSH slaves
   - Set up cloud agents (Docker, Kubernetes)

2. **Create CI/CD Pipelines:**

   - Freestyle jobs
   - Pipeline as Code (Jenkinsfile)
   - Multibranch pipelines

3. **Integrate with SCM:**

   - Configure GitHub/GitLab webhooks
   - Set up automated builds on commit
   - Implement branch strategies

4. **Implement Security:**

   - Role-based access control (RBAC)
   - Credentials management
   - Secret handling with Vault

5. **Add Monitoring:**
   - Install monitoring plugins
   - Configure email notifications
   - Set up Slack integration

---

## 🔗 Additional Resources

- **Official Documentation:**

  - [Jenkins Documentation](https://www.jenkins.io/doc/)
  - [Jenkins Installation Guide](https://www.jenkins.io/doc/book/installing/)
  - [Jenkins Security](https://www.jenkins.io/doc/book/security/)

- **Tutorials:**

  - [Jenkins Pipeline Tutorial](https://www.jenkins.io/doc/book/pipeline/)
  - [Jenkins Best Practices](https://www.jenkins.io/doc/book/managing/)

- **Community:**
  - [Jenkins Community Forums](https://community.jenkins.io/)
  - [Jenkins IRC](https://www.jenkins.io/chat/)

---

## 📝 Summary

In this task, we successfully:

1. ✅ **Installed Jenkins** on a dedicated server using yum
2. ✅ **Configured Java 11** as the runtime environment
3. ✅ **Added Jenkins repository** and GPG keys
4. ✅ **Started Jenkins service** with systemd
5. ✅ **Completed initial setup** with plugin installation
6. ✅ **Created admin user** with specified credentials:
   - Username: `theadmin`
   - Password: `Adm!n321`
   - Full Name: `Ravi`
   - Email: `ravi@jenkins.stratos.xfusioncorp.com`
7. ✅ **Verified installation** through UI and CLI
8. ✅ **Implemented basic security** configurations

The Jenkins server is now ready for creating and managing CI/CD pipelines for xFusionCorp Industries! 🎉

---

**Difficulty Level:** 🟡 Intermediate
**Estimated Time:** 30-45 minutes
**Prerequisites:** Linux basics, systemd knowledge, basic networking

---
