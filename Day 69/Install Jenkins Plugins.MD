# Day 69 — Install Jenkins Plugins 🔌

[![Jenkins](https://img.shields.io/badge/-Jenkins-D24939?style=flat&logo=jenkins&logoColor=white)](https://www.jenkins.io/)
[![Git](https://img.shields.io/badge/-Git-F05032?style=flat&logo=git&logoColor=white)](https://git-scm.com/)
[![GitLab](https://img.shields.io/badge/-GitLab-FC6D26?style=flat&logo=gitlab&logoColor=white)](https://about.gitlab.com/)

## 📋 Task Overview

The Nautilus DevOps team has recently set up a Jenkins server for CI/CD workflows. Before creating jobs, they need to install essential plugins that will be used across most pipelines. This task focuses on installing the Git and GitLab plugins through the Jenkins web interface.

### 🎯 Requirements

1. **Access Jenkins UI** using the provided credentials
2. **Install plugins:**
   - Git plugin
   - GitLab plugin
3. **Restart Jenkins** after plugin installation (if required)
4. **Verify** plugin installation success

### 📝 Access Information

- **Jenkins URL**: Accessible via Jenkins button on top bar
- **Username**: `admin`
- **Password**: `Adm!n321`

---

## 🏗️ Plugin Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                  Jenkins Plugin System                       │
└─────────────────────────────────────────────────────────────┘

    ┌──────────────────────────────────────────┐
    │         Jenkins Core                      │
    │                                           │
    │  ┌────────────────────────────────────┐  │
    │  │      Plugin Manager                │  │
    │  │                                    │  │
    │  │  - Download plugins from repo     │  │
    │  │  - Install .hpi/.jpi files        │  │
    │  │  - Manage dependencies            │  │
    │  │  - Handle updates                 │  │
    │  │  - Restart coordination           │  │
    │  └────────────────────────────────────┘  │
    │                                           │
    │  ┌────────────────────────────────────┐  │
    │  │      Installed Plugins             │  │
    │  │                                    │  │
    │  │  ┌──────────┐   ┌──────────┐     │  │
    │  │  │   Git    │   │  GitLab  │     │  │
    │  │  │  Plugin  │   │  Plugin  │     │  │
    │  │  └──────────┘   └──────────┘     │  │
    │  │                                    │  │
    │  │  Features:                        │  │
    │  │  - SCM integration                │  │
    │  │  - Webhook support                │  │
    │  │  - Branch detection               │  │
    │  │  - Commit tracking                │  │
    │  └────────────────────────────────────┘  │
    └──────────────────────────────────────────┘
                      │
                      ▼
    ┌──────────────────────────────────────────┐
    │        Jenkins Update Center              │
    │                                           │
    │  - updates.jenkins.io                    │
    │  - Plugin repository                     │
    │  - Dependency resolution                 │
    │  - Security advisories                   │
    └──────────────────────────────────────────┘

Plugin Installation Flow:
1. User selects plugin from UI
2. Jenkins checks dependencies
3. Downloads .hpi file from update center
4. Installs plugin in JENKINS_HOME/plugins/
5. Loads plugin (requires restart for some)
6. Plugin becomes available in Jenkins
```

---

## 🚀 Complete Solution

### Step 1: Access Jenkins Web Interface

#### 1.1 Open Jenkins UI

1. **Click the "Jenkins" button** on the top bar of your lab environment
2. This will open Jenkins in a new browser tab

**Expected URL format:**

```
http://jenkins:8080
# or
http://<jenkins-server-ip>:8080
```

#### 1.2 Login to Jenkins

1. **Enter credentials:**

   - **Username**: `admin`
   - **Password**: `Adm!n321`

2. **Click "Sign in"**

**Expected Result:**

- You should see the Jenkins Dashboard
- Welcome message at the top
- Navigation menu on the left

---

### Step 2: Navigate to Plugin Manager

#### 2.1 Access Manage Jenkins

1. **From the Jenkins Dashboard**, click on **"Manage Jenkins"** in the left sidebar

   ```
   Dashboard
   ├── New Item
   ├── People
   ├── Build History
   ├── Manage Jenkins ← Click here
   └── My Views
   ```

#### 2.2 Open Plugin Manager

1. **On the Manage Jenkins page**, scroll down and click **"Manage Plugins"**

   Or use the newer interface:

   - Click **"Plugins"** (in newer Jenkins versions)

**Expected Screen:**

```
┌─────────────────────────────────────────┐
│         Jenkins Plugin Manager          │
├─────────────────────────────────────────┤
│  Tabs:                                  │
│  • Updates                              │
│  • Available  ← We'll use this         │
│  • Installed                            │
│  • Advanced                             │
└─────────────────────────────────────────┘
```

---

### Step 3: Install Git Plugin

#### 3.1 Search for Git Plugin

1. **Click on the "Available" tab** (or "Available plugins" in newer versions)

2. **In the search/filter box**, type: `Git`

3. **Locate the "Git plugin"** in the results
   - Look for the plugin named exactly "Git" or "Git plugin"
   - Published by: "Jenkins Project"

#### 3.2 Select Git Plugin

1. **Check the checkbox** next to the Git plugin

**Plugin Details:**

```
Name: Git
Description: This plugin integrates Git with Jenkins
Features:
  - Git SCM support
  - Multiple repository support
  - Branch and tag support
  - Submodule support
  - Git credential management
```

---

### Step 4: Install GitLab Plugin

#### 4.1 Search for GitLab Plugin

1. **In the same search/filter box**, clear previous text and type: `GitLab`

2. **Locate the "GitLab" plugin** in the results
   - Look for the plugin named "GitLab" or "GitLab Plugin"
   - Not "GitLab API Plugin" or other variants (unless that's the one you need)

#### 4.2 Select GitLab Plugin

1. **Check the checkbox** next to the GitLab plugin

**Plugin Details:**

```
Name: GitLab
Description: This plugin integrates GitLab with Jenkins
Features:
  - GitLab API integration
  - Webhook triggers
  - Merge request support
  - Build status notifications
  - GitLab authentication
```

**Current Selection:**

```
Selected Plugins:
☑ Git
☑ GitLab
```

---

### Step 5: Download and Install Plugins

#### 5.1 Install Selected Plugins

1. **Scroll down** to the bottom of the page

2. **Choose installation option:**

   **Option A: Install with Restart (Recommended)**

   - Click **"Download now and install after restart"**
   - Or click **"Install without restart"** (if available and sufficient)

   **Option B: Install and Restart Automatically**

   - Some versions have: **"Install"** button

3. **For this task, select:**
   ```
   ☑ Restart Jenkins when installation is complete and no jobs are running
   ```

**Installation Options Explained:**

```
┌─────────────────────────────────────────────────────────┐
│ Installation Options:                                   │
├─────────────────────────────────────────────────────────┤
│ 1. Download now and install after restart               │
│    → Downloads plugins, requires manual restart         │
│                                                          │
│ 2. Install without restart                              │
│    → Installs immediately (works for some plugins)      │
│                                                          │
│ 3. Checkbox: Restart Jenkins when installation is       │
│    complete and no jobs are running                     │
│    → Automatic restart after download ✓ (Use this)     │
└─────────────────────────────────────────────────────────┘
```

#### 5.2 Monitor Installation Progress

After clicking install, you'll see the **"Installing Plugins/Upgrades"** page.

**Expected Output:**

```
Installing Plugins/Upgrades
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Git
  Downloading...  [████████████████████] 100%
  Installing...   ✓ Success

GitLab
  Downloading...  [████████████████████] 100%
  Installing...   ✓ Success

Dependencies (auto-installed):
  git-client
    Downloading... [████████████████████] 100%
    Installing...  ✓ Success

  credentials
    Downloading... [████████████████████] 100%
    Installing...  ✓ Success

  [More dependencies as needed...]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Status: Installation complete
```

**Note:** Jenkins will automatically install any required dependencies.

---

### Step 6: Restart Jenkins

#### 6.1 Automatic Restart (If Checkbox Selected)

If you checked **"Restart Jenkins when installation is complete and no jobs are running"**:

1. **Wait for the message:**

   ```
   Installing Plugins/Upgrades

   Installation complete. Jenkins will restart automatically.
   ```

2. **Jenkins will restart automatically** after 5-10 seconds

#### 6.2 Manual Restart (Alternative Method)

If you need to restart manually:

**Method 1: Via Web UI**

```
Dashboard → Manage Jenkins → Prepare for Shutdown
Then: systemctl restart jenkins (on the server)
```

**Method 2: Via URL**

```
Navigate to: http://jenkins:8080/restart
Click: "Yes" to confirm restart
```

**Method 3: Via CLI (SSH to Jenkins server)**

```bash
# Restart Jenkins service
sudo systemctl restart jenkins

# Or using Jenkins CLI
java -jar jenkins-cli.jar -s http://localhost:8080/ restart
```

#### 6.3 Wait for Jenkins to Come Back Online

**Expected behavior:**

1. **Browser shows:** "Jenkins is restarting" or connection error
2. **Wait 30-60 seconds**
3. **Page auto-refreshes** or manually refresh
4. **Login page reappears**

**Console Output (if monitoring via terminal):**

```bash
# Check Jenkins status
systemctl status jenkins

# Output:
● jenkins.service - Jenkins Continuous Integration Server
   Active: active (running) since Mon 2025-11-04 11:00:00 UTC; 10s ago
```

---

### Step 7: Verify Plugin Installation

#### 7.1 Login Again After Restart

1. **Enter credentials** (if logged out):

   - Username: `admin`
   - Password: `Adm!n321`

2. **Click "Sign in"**

#### 7.2 Check Installed Plugins

1. **Navigate to:** Dashboard → Manage Jenkins → Manage Plugins

2. **Click on "Installed" tab**

3. **Search for installed plugins:**
   - Type "Git" in the filter box
   - Type "GitLab" in the filter box

**Expected Result:**

```
Installed Plugins:

☑ Git
  Version: x.x.x
  Enabled: ✓

☑ GitLab
  Version: x.x.x
  Enabled: ✓

☑ git-client (dependency)
  Version: x.x.x
  Enabled: ✓

☑ credentials (dependency)
  Version: x.x.x
  Enabled: ✓
```

#### 7.3 Verify Plugin Functionality

**Test Git Plugin:**

1. **Create a test job:**

   - Dashboard → New Item
   - Item name: `test-git-plugin`
   - Select: "Freestyle project"
   - Click: "OK"

2. **Configure Source Code Management:**
   - Scroll to "Source Code Management" section
   - **You should see "Git" option** ✓ (this confirms Git plugin is working)

**Test GitLab Plugin:**

1. **Check GitLab options in job configuration:**

   - In the same job or new job
   - Look for "Build Triggers" section
   - **You should see "Build when a change is pushed to GitLab"** ✓

2. **Check Manage Jenkins for GitLab:**
   - Manage Jenkins → Configure System
   - Scroll down to find **"GitLab"** section ✓

---

## 🔍 Verification Commands

### Via Jenkins Web UI

```
Step-by-step verification:

1. Dashboard → Manage Jenkins → Manage Plugins → Installed tab

2. Filter: "Git"
   Result: Git plugin should appear with green checkmark

3. Filter: "GitLab"
   Result: GitLab plugin should appear with green checkmark

4. Dashboard → New Item → Freestyle project
   In SCM section: Git option should be available

5. In Build Triggers: GitLab webhook option should be available
```

### Via Jenkins CLI (Advanced)

```bash
# SSH to Jenkins server
ssh root@jenkins

# List installed plugins
java -jar jenkins-cli.jar -s http://localhost:8080/ -auth admin:Adm!n321 list-plugins | grep -i git

# Expected output:
# git                    Git plugin               x.x.x
# gitlab-plugin          GitLab Plugin            x.x.x
# git-client             Git client plugin        x.x.x
```

### Via Jenkins Script Console (Advanced)

```groovy
// Dashboard → Manage Jenkins → Script Console

// List all installed plugins
Jenkins.instance.pluginManager.plugins.each { plugin ->
  if (plugin.shortName.contains('git')) {
    println("${plugin.shortName}: ${plugin.version}")
  }
}

// Check specific plugin
def gitPlugin = Jenkins.instance.pluginManager.getPlugin('git')
println "Git Plugin Installed: ${gitPlugin != null}"
println "Git Plugin Version: ${gitPlugin?.version}"

def gitlabPlugin = Jenkins.instance.pluginManager.getPlugin('gitlab-plugin')
println "GitLab Plugin Installed: ${gitlabPlugin != null}"
println "GitLab Plugin Version: ${gitlabPlugin?.version}"
```

### Via File System (SSH to Jenkins server)

```bash
# Check plugins directory
ls -la /var/lib/jenkins/plugins/ | grep git

# Expected output:
# drwxr-xr-x  3 jenkins jenkins  4096 Nov  4 11:00 git
# drwxr-xr-x  3 jenkins jenkins  4096 Nov  4 11:00 gitlab-plugin
# drwxr-xr-x  3 jenkins jenkins  4096 Nov  4 11:00 git-client

# Check plugin manifest
cat /var/lib/jenkins/plugins/git/META-INF/MANIFEST.MF

# Check plugin is enabled
ls -la /var/lib/jenkins/plugins/*.jpi | grep -E "(git|gitlab)"
```

---

## 🧪 Testing Plugin Installation

### Test 1: Create Job with Git SCM

```
1. Dashboard → New Item
2. Name: test-git-scm
3. Type: Freestyle project
4. Click: OK

5. Source Code Management section:
   ☑ Select "Git"

6. Repository URL: https://github.com/sample/repo.git
   (You should be able to enter this without errors)

7. If Git plugin is working:
   - Branches to build field appears
   - Additional Behaviours button appears
   - Repository browser option appears

8. Save (no need to run)
9. ✓ Test Passed: Git plugin is functional
```

### Test 2: Check GitLab Integration Options

```
1. Create or edit a job
2. Build Triggers section:
   ☑ Look for "Build when a change is pushed to GitLab"

3. If GitLab plugin is working:
   - GitLab webhook URL is generated
   - GitLab CI Service options appear
   - Secret token field appears

4. ✓ Test Passed: GitLab plugin is functional
```

### Test 3: Verify Plugin Dependencies

```
1. Manage Jenkins → Manage Plugins → Installed

2. Search for: git-client
   Status: Should be installed (dependency of Git)

3. Search for: credentials
   Status: Should be installed (common dependency)

4. Search for: workflow-scm-step
   Status: May be installed (for Pipeline support)

5. ✓ Test Passed: All dependencies resolved
```

---

## 🛠️ Troubleshooting Guide

### Issue 1: Cannot Find Plugins in Available Tab

**Symptom:**
Git or GitLab plugins don't appear in the Available plugins list.

**Solution:**

```bash
# Method 1: Update Plugin Site
1. Manage Jenkins → Manage Plugins → Advanced tab
2. Scroll to "Update Site"
3. URL: https://updates.jenkins.io/update-center.json
4. Click "Check now" button
5. Wait for update, then go back to Available tab

# Method 2: Clear Update Cache (SSH to Jenkins)
sudo systemctl stop jenkins
sudo rm -rf /var/lib/jenkins/updates
sudo systemctl start jenkins
# Wait 60 seconds, then refresh Jenkins UI
# Manage Jenkins → Manage Plugins → Click "Check now"

# Method 3: Check Internet Connectivity
curl -I https://updates.jenkins.io
# Should return 200 OK

# Method 4: Check Proxy Settings (if behind proxy)
# Manage Jenkins → Manage Plugins → Advanced
# HTTP Proxy Configuration:
#   Server: proxy.company.com
#   Port: 8080
```

### Issue 2: Plugin Installation Fails

**Symptom:**

```
Failed to download from https://updates.jenkins.io/download/plugins/git/...
```

**Solution:**

```bash
# Method 1: Retry Installation
1. Go back to Available plugins
2. Search and select plugin again
3. Try "Install without restart" first
4. If fails, try "Download now and install after restart"

# Method 2: Manual Plugin Installation
# Download plugin from Jenkins website
wget https://updates.jenkins.io/download/plugins/git/latest/git.hpi

# Upload via Jenkins UI
# Manage Jenkins → Manage Plugins → Advanced tab
# Upload Plugin section → Choose file → Upload

# Or copy to plugins directory
sudo cp git.hpi /var/lib/jenkins/plugins/
sudo chown jenkins:jenkins /var/lib/jenkins/plugins/git.hpi
sudo systemctl restart jenkins

# Method 3: Check Disk Space
df -h /var/lib/jenkins
# Ensure sufficient space (at least 500MB free)

# Method 4: Check Permissions
ls -la /var/lib/jenkins/plugins/
sudo chown -R jenkins:jenkins /var/lib/jenkins/plugins/
```

### Issue 3: Plugin Shows as Installed but Not Working

**Symptom:**
Plugin appears in Installed tab but features don't work.

**Solution:**

```bash
# Method 1: Enable the Plugin
1. Manage Jenkins → Manage Plugins → Installed
2. Find the plugin
3. Check if "Enabled" checkbox is checked
4. If unchecked, check it and click "Enable"

# Method 2: Check for Dependency Issues
1. Installed tab → Look for plugins with warning icons
2. Click on plugin name to see details
3. Install missing dependencies manually

# Method 3: Restart Jenkins
sudo systemctl restart jenkins

# Method 4: Reinstall Plugin
1. Uninstall the problematic plugin
2. Restart Jenkins
3. Install plugin again

# Method 5: Check Logs for Errors
sudo tail -f /var/log/jenkins/jenkins.log
# Look for plugin-related errors

# Method 6: Verify Plugin Extraction
ls -la /var/lib/jenkins/plugins/git/
# Should see extracted plugin contents, not just .jpi file
```

### Issue 4: Jenkins Doesn't Restart After Plugin Installation

**Symptom:**
Selected "Restart when complete" but Jenkins is stuck.

**Solution:**

```bash
# Method 1: Wait Longer
# Sometimes Jenkins takes 2-3 minutes to restart
# Be patient, especially if many plugins are being installed

# Method 2: Check Jenkins Process
ps aux | grep jenkins
# If process is running but UI is unresponsive:

sudo systemctl status jenkins
# If status shows "active", give it more time

# Method 3: Force Restart (SSH to server)
sudo systemctl restart jenkins

# Method 4: Check for Port Conflicts
sudo netstat -tulpn | grep 8080
# If another process is using 8080:
sudo kill -9 <PID>
sudo systemctl start jenkins

# Method 5: Check Logs
sudo journalctl -u jenkins -n 100 --no-pager
sudo tail -50 /var/log/jenkins/jenkins.log
# Look for startup errors
```

### Issue 5: Login Page Doesn't Reappear After Restart

**Symptom:**
After restart, browser shows connection refused or timeout.

**Solution:**

```bash
# Method 1: Check Jenkins Service Status
sudo systemctl status jenkins
# If inactive:
sudo systemctl start jenkins

# Method 2: Check Jenkins Port
sudo ss -tulpn | grep 8080
# If nothing is listening on 8080:
# Jenkins hasn't fully started yet, wait more

# Method 3: Monitor Jenkins Startup
sudo tail -f /var/log/jenkins/jenkins.log
# Wait for: "Jenkins is fully up and running"

# Method 4: Clear Browser Cache
# In browser: Ctrl+Shift+Delete
# Clear cache and cookies for Jenkins site
# Try accessing again

# Method 5: Access via Different URL
# If using http://jenkins:8080, try:
http://<ip-address>:8080
# or
http://localhost:8080 (if on same machine)

# Method 6: Check Firewall
sudo firewall-cmd --list-ports
# Ensure 8080/tcp is allowed:
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

### Issue 6: Dependency Conflicts

**Symptom:**

```
Dependency errors during plugin installation
Plugin X requires Y version, but Z is installed
```

**Solution:**

```bash
# Method 1: Update All Plugins First
1. Manage Jenkins → Manage Plugins → Updates tab
2. Check "Select All"
3. Click "Download now and install after restart"
4. Restart Jenkins
5. Then install Git and GitLab plugins

# Method 2: Install Dependencies Manually
1. Read the error message to identify missing dependencies
2. Go to Available tab
3. Search for each dependency
4. Install dependencies first
5. Then install main plugins

# Method 3: Use Plugin Manager Suggestions
# Modern Jenkins shows "Install suggested dependencies"
# Check this option during installation

# Method 4: Clean Install
1. Uninstall conflicting plugins
2. Restart Jenkins
3. Install plugins in correct order:
   - First: Core dependencies
   - Then: Main plugins
```

### Issue 7: Wrong Plugin Version Installed

**Symptom:**
Plugin installed but features are missing or different.

**Solution:**

```bash
# Method 1: Check Plugin Version Compatibility
# Visit: https://plugins.jenkins.io/git/
# Check "Compatibility" section
# Ensure your Jenkins version supports the plugin

# Method 2: Install Specific Plugin Version
1. Manage Jenkins → Manage Plugins → Advanced
2. Upload Plugin section
3. Download specific version from:
   https://updates.jenkins.io/download/plugins/git/
4. Upload the .hpi file

# Method 3: Upgrade Jenkins (if too old)
# Check Jenkins version: Dashboard (bottom right)
# If too old, consider upgrading Jenkins first:
sudo yum update jenkins -y
sudo systemctl restart jenkins
```

### Issue 8: Permission Errors During Installation

**Symptom:**

```
Failed to create plugin directory
Permission denied: /var/lib/jenkins/plugins/
```

**Solution:**

```bash
# Fix Jenkins home permissions
sudo chown -R jenkins:jenkins /var/lib/jenkins
sudo chmod -R 755 /var/lib/jenkins

# Fix plugins directory specifically
sudo chown -R jenkins:jenkins /var/lib/jenkins/plugins
sudo chmod -R 755 /var/lib/jenkins/plugins

# Restart Jenkins
sudo systemctl restart jenkins

# Verify permissions
ls -la /var/lib/jenkins/ | grep plugins
# Should show: drwxr-xr-x jenkins jenkins
```

---

## 📊 Plugin Information

### Git Plugin Details

```yaml
Plugin Name: Git
Short Name: git
Description: Integrates Git version control with Jenkins
Publisher: Jenkins Project
Category: Source Code Management

Key Features:
  - Git repository support
  - Multiple repository handling
  - Branch and tag selection
  - Submodule support
  - Shallow clone options
  - Advanced checkout behaviors
  - Git credential integration
  - Webhook triggers
  - Polling SCM capability

Use Cases:
  - Fetch source code from Git repositories
  - Trigger builds on Git commits
  - Support for GitHub, GitLab, Bitbucket
  - Pipeline Git steps
  - Multibranch pipeline jobs

Common Dependencies:
  - git-client
  - credentials
  - workflow-scm-step (for pipelines)
  - ssh-credentials (for SSH Git access)

Configuration:
  - Global: Manage Jenkins → Global Tool Configuration → Git
  - Job Level: Job → Configure → Source Code Management → Git
```

### GitLab Plugin Details

```yaml
Plugin Name: GitLab
Short Name: gitlab-plugin
Description: Integrates GitLab with Jenkins
Publisher: GitLab Community
Category: Source Code Management, Build Triggers

Key Features:
  - GitLab API integration
  - Webhook support for GitLab
  - Merge request builder
  - Build status notifications to GitLab
  - GitLab authentication
  - Commit status updates
  - Comment triggers
  - Pipeline status integration

Use Cases:
  - Trigger Jenkins jobs from GitLab push/MR events
  - Update GitLab with build status
  - Build merge requests automatically
  - Integrate with GitLab CI/CD
  - Use GitLab as authentication provider

Common Dependencies:
  - git
  - credentials
  - plain-credentials
  - git-client

Configuration:
  - Global: Manage Jenkins → Configure System → GitLab
  - Job Level: Job → Build Triggers → Build when change pushed to GitLab

API Token Setup: 1. GitLab → User Settings → Access Tokens
  2. Create token with api scope
  3. Add to Jenkins credentials
  4. Configure in Jenkins GitLab settings
```

---

## 🔒 Best Practices

### 1. Regular Plugin Updates

```bash
# Weekly routine:
1. Manage Jenkins → Manage Plugins → Updates tab
2. Review available updates
3. Read changelogs for breaking changes
4. Update non-critical plugins first
5. Test in non-production environment
6. Backup before major updates
```

### 2. Security Considerations

```bash
# Check for security advisories:
1. Manage Jenkins → Manage Plugins
2. Look for security warning icons
3. Visit: https://www.jenkins.io/security/advisories/
4. Update vulnerable plugins immediately

# Enable security scanning:
# Manage Jenkins → Configure System
# Consider installing:
  - OWASP Dependency-Check plugin
  - Warnings Next Generation
```

### 3. Plugin Management Strategy

```yaml
Essential Plugins to Install:
  Source Control:
    - Git
    - GitLab
    - GitHub (if using GitHub)

  Build Tools:
    - Maven Integration
    - Gradle
    - NodeJS

  Notifications:
    - Email Extension
    - Slack Notification

  Security:
    - Credentials Binding
    - Role-based Authorization Strategy

  Pipeline:
    - Pipeline
    - Pipeline: Stage View
    - Blue Ocean (modern UI)

  Utilities:
    - AnsiColor (colored console output)
    - Timestamper
    - Build Timeout

Installation Order: 1. Security plugins first
  2. Core functionality (Git, credentials)
  3. Build tools
  4. Notifications
  5. UI enhancements
```

### 4. Backup Strategy

```bash
# Before major plugin changes:

# Backup plugins directory
sudo tar -czf /backup/jenkins-plugins-$(date +%Y%m%d).tar.gz \
  /var/lib/jenkins/plugins/

# Backup Jenkins configuration
sudo tar -czf /backup/jenkins-config-$(date +%Y%m%d).tar.gz \
  --exclude='/var/lib/jenkins/workspace/*' \
  --exclude='/var/lib/jenkins/plugins/*' \
  /var/lib/jenkins/

# Restore if needed:
sudo systemctl stop jenkins
sudo tar -xzf /backup/jenkins-plugins-YYYYMMDD.tar.gz -C /
sudo chown -R jenkins:jenkins /var/lib/jenkins/plugins
sudo systemctl start jenkins
```

### 5. Plugin Compatibility Testing

```bash
# Before upgrading plugins in production:

1. Create test Jenkins instance
2. Mirror production plugin versions
3. Upgrade plugins one by one
4. Test critical jobs
5. Document any issues
6. Plan rollback if needed
```

---

## 📸 Documentation Tips

### Screenshots to Capture

```
1. Plugin Manager - Available Tab
   - Shows search results for Git and GitLab
   - Checkboxes selected

2. Plugin Installation Progress
   - Shows downloading/installing status
   - Success messages

3. Restart Screen
   - "Jenkins is restarting" message

4. Installed Plugins Tab
   - Git plugin with version number
   - GitLab plugin with version number

5. Job Configuration
   - Git option in SCM section
   - GitLab webhook in Build Triggers

6. Verification
   - Test job showing Git integration working
```

### Using Screen Recording (loom.com)

```
Recording Steps:
1. Sign up at loom.com
2. Install Loom browser extension or desktop app
3. Start recording
4. Navigate through entire process:
   - Login
   - Plugin Manager
   - Search plugins
   - Install plugins
   - Restart
   - Verify installation
5. Stop recording
6. Share link with team/reviewer

Benefits:
- Shows complete workflow
- Captures any errors in real-time
- Easier for review than multiple screenshots
- Can add voice narration
```

---

## 🧹 Cleanup (Optional)

### Uninstall Plugins (If Needed)

```bash
# Via Web UI:
1. Manage Jenkins → Manage Plugins → Installed
2. Find the plugin
3. Click "Uninstall" button (right side)
4. Confirm uninstall
5. Restart Jenkins

# Via SSH (Manual):
sudo systemctl stop jenkins
sudo rm -rf /var/lib/jenkins/plugins/git*
sudo rm -rf /var/lib/jenkins/plugins/gitlab*
sudo systemctl start jenkins
```

### Disable Plugins (Without Uninstalling)

```bash
# Via Web UI:
1. Manage Jenkins → Manage Plugins → Installed
2. Find the plugin
3. Uncheck "Enabled" checkbox
4. Click "Apply"

# Via SSH:
sudo systemctl stop jenkins
sudo touch /var/lib/jenkins/plugins/git.jpi.disabled
sudo systemctl start jenkins
# To re-enable, remove .disabled file
```

---

## 📚 Key Takeaways

### ✅ What We Learned

1. **Plugin Management:**

   - Navigated Jenkins Plugin Manager UI
   - Searched for and selected plugins
   - Understood installation options

2. **Installation Process:**

   - Downloaded plugins from Jenkins Update Center
   - Managed plugin dependencies automatically
   - Configured auto-restart after installation

3. **Jenkins Restart:**

   - Understood when restart is necessary
   - Used automatic restart feature
   - Monitored restart process

4. **Verification:**

   - Confirmed plugin installation via Installed tab
   - Tested plugin functionality in job configuration
   - Verified plugin features are available

5. **Troubleshooting:**
   - Identified common plugin installation issues
   - Applied fixes for various error scenarios
   - Used multiple verification methods

### 🎓 Skills Developed

- ✅ Jenkins UI navigation
- ✅ Plugin management best practices
- ✅ Dependency resolution understanding
- ✅ Service restart procedures
- ✅ Installation verification techniques
- ✅ Troubleshooting plugin issues
- ✅ Documentation and screenshot capture
- ✅ Git and GitLab integration basics

### 🚀 Next Steps

1. **Configure Git Plugin:**

   - Set up Git credentials
   - Configure global Git settings
   - Create jobs using Git SCM

2. **Configure GitLab Plugin:**

   - Add GitLab API token
   - Set up webhook URLs
   - Configure merge request builder

3. **Create CI/CD Pipelines:**

   - Build jobs that fetch from Git/GitLab
   - Set up automated triggers
   - Implement build notifications

4. **Explore More Plugins:**

   - GitHub plugin for GitHub integration
   - Docker plugin for containerized builds
   - Kubernetes plugin for cloud builds
   - Blue Ocean for modern UI

5. **Implement Best Practices:**
   - Regular plugin updates
   - Security scanning
   - Plugin compatibility testing
   - Backup procedures

---

## 🔗 Additional Resources

- **Official Documentation:**

  - [Jenkins Plugin Tutorial](https://www.jenkins.io/doc/book/managing/plugins/)
  - [Git Plugin Documentation](https://plugins.jenkins.io/git/)
  - [GitLab Plugin Documentation](https://plugins.jenkins.io/gitlab-plugin/)

- **Plugin Sites:**

  - [Jenkins Plugin Index](https://plugins.jenkins.io/)
  - [Jenkins Update Center](https://updates.jenkins.io/)

- **Tutorials:**

  - [Managing Jenkins Plugins](https://www.jenkins.io/doc/book/managing/)
  - [Git Integration Guide](https://www.jenkins.io/doc/book/pipeline/jenkinsfile/#using-a-git-repository)

- **Community:**
  - [Jenkins Community Forums](https://community.jenkins.io/)
  - [Jenkins IRC Chat](https://www.jenkins.io/chat/)

---

## 📝 Summary

In this task, we successfully:

1. ✅ **Accessed Jenkins UI** with admin credentials
2. ✅ **Navigated to Plugin Manager** via Manage Jenkins
3. ✅ **Selected plugins for installation:**
   - Git plugin (for Git repository integration)
   - GitLab plugin (for GitLab webhook and API integration)
4. ✅ **Installed plugins** with automatic dependency resolution
5. ✅ **Restarted Jenkins** using auto-restart option
6. ✅ **Verified installation** via Installed Plugins tab
7. ✅ **Tested functionality** by checking SCM and build trigger options
8. ✅ **Documented the process** for team review

The Jenkins server now has Git and GitLab plugins installed and ready for CI/CD pipeline creation! 🎉

---

**Difficulty Level:** 🟢 Beginner
**Estimated Time:** 10-15 minutes
**Prerequisites:** Jenkins server setup (Day 68), basic Jenkins UI familiarity

---
