# Day 70 — Configure Jenkins User Access 👥

[![Jenkins](https://img.shields.io/badge/-Jenkins-D24939?style=flat&logo=jenkins&logoColor=white)](https://www.jenkins.io/)
[![Security](https://img.shields.io/badge/-Security-000000?style=flat&logo=security&logoColor=white)](https://www.jenkins.io/doc/book/security/)
[![Authorization](https://img.shields.io/badge/-Authorization-4A90E2?style=flat)](https://www.jenkins.io/doc/book/security/authorization/)

## 📋 Task Overview

The Nautilus team is integrating Jenkins into their CI/CD pipelines. After setting up a new Jenkins server, they need to configure user access for the development team. This task focuses on user management, authentication, and authorization using Project-based Matrix Authorization Strategy.

### 🎯 Requirements

1. **Create Jenkins user** named `john` with credentials:

   - Username: `john`
   - Password: `ksH85UJjhb`
   - Full Name: `John`

2. **Enable Project-based Matrix Authorization Strategy** for fine-grained access control

3. **Grant john user permissions:**

   - Overall: Read permission
   - Job: Read permission on existing jobs

4. **Remove Anonymous user permissions** while maintaining admin privileges

5. **Configure job-level authorization** for the john user

### 📝 Access Information

- **Jenkins URL**: Accessible via Jenkins button on top bar
- **Admin Username**: `admin`
- **Admin Password**: `Adm!n321`
- **New User**: `john` / `ksH85UJjhb`

---

## 🏗️ Jenkins Authorization Architecture

```
┌─────────────────────────────────────────────────────────────┐
│            Jenkins Authorization Architecture               │
└─────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                  Authentication Layer                        │
│                                                               │
│  Jenkins' own user database                                 │
│  ├── admin (Administer permissions)                          │
│  ├── john (Read permissions)                                 │
│  └── <other users>                                           │
│                                                               │
│  LDAP/Active Directory (optional)                            │
│  PAM-based authentication (optional)                         │
└──────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────┐
│               Authorization Layer (Matrix)                   │
│                                                               │
│  Project-based Matrix Authorization Strategy                │
│  ├── Global Level (Overall permissions)                      │
│  │   ├── Admin          (administer)     ✓                  │
│  │   ├── John           (read)           ✓                  │
│  │   └── Anonymous      (all)            ✗                  │
│  │                                                           │
│  └── Job Level (Job-specific permissions)                   │
│      ├── Admin          (all permissions) ✓                 │
│      ├── John           (read only)       ✓                 │
│      └── Anonymous      (no permissions)  ✗                 │
└──────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────────────┐
│                 Resource Access Control                      │
│                                                               │
│  Global Resources:      Dashboard, System Config             │
│  Job Resources:         Job config, Build history            │
│  Plugin Resources:      Plugin management                    │
│  Credentials Resources: Secrets, tokens                      │
│                                                               │
│  Each user gets specific access based on matrix rules        │
└──────────────────────────────────────────────────────────────┘

Permission Types:
┌────────────────────┬─────────────────────────────────────┐
│ Overall Permissions│ Agent Permissions                   │
├────────────────────┼─────────────────────────────────────┤
│ • Administer       │ • Build                             │
│ • Configure        │ • Connect                           │
│ • Create           │ • Create                            │
│ • Delete           │ • Delete                            │
│ • Extend Vote      │ • Disconnect                        │
│ • Move             │                                     │
│ • Read             │                                     │
│ • RunScript        │                                     │
├────────────────────┼─────────────────────────────────────┤
│ Job Permissions    │ Run Permissions                     │
├────────────────────┼─────────────────────────────────────┤
│ • Build            │ • Delete                            │
│ • Cancel           │ • Update                            │
│ • Configure        │                                     │
│ • Delete           │                                     │
│ • Discover         │                                     │
│ • Move             │                                     │
│ • Read             │                                     │
│ • Workspace        │                                     │
└────────────────────┴─────────────────────────────────────┘
```

---

## 🚀 Complete Solution

### Step 1: Login to Jenkins

#### 1.1 Access Jenkins UI

1. **Click the "Jenkins" button** on the top bar of your lab environment
2. Jenkins will open in a new tab

**Expected URL:**

```
http://jenkins:8080
```

#### 1.2 Login with Admin Credentials

1. **Enter credentials:**

   - **Username**: `admin`
   - **Password**: `Adm!n321`

2. **Click "Sign in"**

**Expected Result:**

- Jenkins Dashboard appears
- "Welcome to Jenkins!" message visible
- Left sidebar with navigation menu displayed

---

### Step 2: Create User 'john'

#### 2.1 Navigate to Manage Users

1. **Click "Manage Jenkins"** in the left sidebar

2. **Look for "Manage Users"** or "Manage User and Group"

   - In newer Jenkins versions: **Security** → **Manage Users**
   - In older versions: **Manage Jenkins** → **Manage Users**

3. **Click on "Manage Users"**

**Expected Screen:**

```
┌────────────────────────────────┐
│    Jenkins User Manager        │
├────────────────────────────────┤
│ List of Users:                 │
│ • admin                        │
│                                │
│ [Create User] button (top)     │
└────────────────────────────────┘
```

#### 2.2 Create New User

1. **Click "Create User"** button

2. **Fill in the user form:**

| Field            | Value                 |
| ---------------- | --------------------- |
| Username         | `john`                |
| Password         | `ksH85UJjhb`          |
| Confirm Password | `ksH85UJjhb`          |
| Full name        | `John`                |
| Email Address    | (optional, can leave) |

**Form Layout:**

```
┌─────────────────────────────────────────┐
│     Create User Form                    │
├─────────────────────────────────────────┤
│                                         │
│ Username:           [________john__]   │
│ Password:           [____ksH85UJjhb_]  │
│ Confirm Password:   [____ksH85UJjhb_]  │
│ Full name:          [________John__]   │
│ Email Address:      [________________] │
│                                         │
│                  [Create User] button    │
└─────────────────────────────────────────┘
```

3. **Click "Create User"**

**Expected Result:**

```
User 'john' created successfully
Users list now shows:
• admin
• john
```

---

### Step 3: Install Matrix Authorization Plugin (if needed)

#### 3.1 Check Current Authorization Strategy

1. **Navigate to:** Manage Jenkins → Configure Global Security

2. **Look at "Authorization" section**

3. **If "Project-based Matrix Authorization Strategy" is NOT available:**

#### 3.2 Install Plugin

1. **Go to:** Manage Jenkins → Manage Plugins

2. **Click on "Available" tab**

3. **Search for:** `Matrix Authorization Strategy`

4. **Find and check the plugin:**
   - Full name: "Matrix Authorization Strategy Plugin"
   - Published by: Jenkins Project

**Plugin Details:**

```
Name: Matrix Authorization Strategy
Short Name: matrix-auth
Description: Provides fine-grained permissions configuration
Features:
  - Global matrix authorization
  - Project-based matrix authorization
  - Fine-grained permission control
  - User and group management
  - Permission inheritance
```

5. **Select the plugin checkbox**

6. **Click:** "Download now and install after restart"

7. **Check the box:** "Restart Jenkins when installation is complete and no jobs are running"

#### 3.3 Wait for Restart

- **Monitor the installation progress** screen
- Jenkins will restart automatically
- **Wait 30-60 seconds** for Jenkins to come back online
- **Refresh browser** when it reappears
- **Login again** with admin credentials

---

### Step 4: Configure Global Security with Matrix Authorization

#### 4.1 Navigate to Configure Global Security

1. **After plugin installation (if done), click:** Manage Jenkins

2. **Select:** Configure Global Security

**Expected Screen:**

```
┌─────────────────────────────────────┐
│   Configure Global Security         │
├─────────────────────────────────────┤
│ Security Realm:                     │
│  ○ Jenkins' own user database       │
│  (others)                           │
│                                     │
│ Authorization:                      │
│  ○ Anyone can do anything           │
│  ○ Legacy mode                      │
│  ○ Logged-in users can do anything  │
│  ◉ Project-based Matrix Auth... (✓) │
│                                     │
│ [Apply] [Save] buttons              │
└─────────────────────────────────────┘
```

#### 4.2 Select Authorization Strategy

1. **Under "Authorization" section**, select:
   ```
   ◉ Project-based Matrix Authorization Strategy
   ```

**Important Note:**

- If this option doesn't appear, install the Matrix Authorization Strategy plugin (Step 3)

#### 4.3 Configure Global Permissions Matrix

After selecting Matrix Authorization Strategy, a **permissions matrix** will appear below.

**Current Matrix (before changes):**

```
┌──────────────┬──────┬────────┬───────────┬─────┐
│ User/Group   │ Read │ Config │ Discover  │ ... │
├──────────────┼──────┼────────┼───────────┼─────┤
│ admin        │  ✓   │   ✓    │    ✓      │ ... │
│ Anonymous    │  ✓   │   ✓    │    ✓      │ ... │
└──────────────┴──────┴────────┴───────────┴─────┘
```

#### 4.4 Add john User to Matrix

1. **Scroll down in the matrix area**

2. **Look for "Add user or group" section** (usually at bottom)

3. **Enter username:** `john`

4. **Click "Add" or similar button**

**Result: john user added to matrix:**

```
┌──────────────┬──────┬────────┬───────────┬─────┐
│ User/Group   │ Read │ Config │ Discover  │ ... │
├──────────────┼──────┼────────┼───────────┼─────┤
│ admin        │  ✓   │   ✓    │    ✓      │ ... │
│ john         │  ☐   │   ☐    │    ☐      │ ... │
│ Anonymous    │  ✓   │   ✓    │    ✓      │ ... │
└──────────────┴──────┴────────┴───────────┴─────┘
```

#### 4.5 Set john User Permissions

1. **In john's row, find "Overall" → "Read" column**

2. **Check ONLY this permission:**
   ```
   john:
   • Overall → Read: ✓ (CHECK THIS)
   • Everything else: ☐ (UNCHECK)
   ```

**Matrix after john permissions:**

```
┌──────────────┬──────┬────────┬───────────┬──────┐
│ User/Group   │ Read │ Config │ Discover  │ ...  │
├──────────────┼──────┼────────┼───────────┼──────┤
│ admin        │  ✓   │   ✓    │    ✓      │ ...  │
│ john         │  ✓   │   ☐    │    ☐      │ ... │
│ Anonymous    │  ✓   │   ✓    │    ✓      │ ...  │
└──────────────┴──────┴────────┴───────────┴──────┘
```

#### 4.6 Verify Admin Permissions

1. **Check admin user row:**

   ```
   admin should have ALL permissions checked (Administer, Read, Config, etc.)
   ```

2. **Confirm:** All checkboxes in admin row are checked ✓

#### 4.7 Remove Anonymous Permissions

1. **Locate the "Anonymous" row** in the matrix

2. **Uncheck ALL checkboxes** for Anonymous user
   ```
   Anonymous:
   • Overall → Administer: ☐ (UNCHECK)
   • Overall → Read: ☐ (UNCHECK)
   • Job → Build: ☐ (UNCHECK)
   • Job → Read: ☐ (UNCHECK)
   • (All others): ☐ (UNCHECK)
   ```

**Important:** Make sure NO checkboxes are checked for Anonymous

**Matrix after removing Anonymous permissions:**

```
┌──────────────┬──────┬────────┬───────────┬──────┐
│ User/Group   │ Read │ Config │ Discover  │ ...  │
├──────────────┼──────┼────────┼───────────┼──────┤
│ admin        │  ✓   │   ✓    │    ✓      │ ... │
│ john         │  ✓   │   ☐    │    ☐      │ ... │
│ Anonymous    │  ☐   │   ☐    │    ☐      │ ... │
└──────────────┴──────┴────────┴───────────┴──────┘
```

#### 4.8 Save Global Security Configuration

1. **Scroll to bottom of page**

2. **Click "Save" button**

**Expected Result:**

```
Configuration saved
(Page reloads or confirmation message appears)
```

---

### Step 5: Configure Job-Level Authorization

#### 5.1 Find and Open Existing Job

1. **Go to Jenkins Dashboard** (home page)

2. **Locate an existing job** (e.g., any job that was previously created)

   - You should see at least one job in the list
   - Click on it to open

3. **Click "Configure"** to open job configuration

**Expected Screen:**

```
┌─────────────────────────────┐
│ Job Configuration           │
├─────────────────────────────┤
│ General                     │
│ Source Code Management      │
│ Build Triggers              │
│ Build Steps                 │
│ Authorization ← Look for it │
│ ...                         │
│ Post-build Actions          │
│ [Save] [Cancel] buttons     │
└─────────────────────────────┘
```

#### 5.2 Enable Job-Level Authorization

1. **Scroll down to find "Authorization" section** (or "Security" section)

2. **Check the box:** "Enable project-based security"
   ```
   ☑ Enable project-based security
   ```

**Alternative names:**

- "Authorization"
- "Configure matrix-based security"
- "Project-based Matrix Authorization"

#### 5.3 Configure Job Permissions Matrix

After enabling, a **permissions matrix** appears for this job.

**Initial matrix:**

```
┌──────────────┬──────┬────────┬───────────┐
│ User/Group   │ Read │ Config │ Discover  │
├──────────────┼──────┼────────┼───────────┤
│ admin        │  ✓   │   ✓    │    ✓      │
└──────────────┴──────┴────────┴───────────┘
```

#### 5.4 Add john User to Job Matrix

1. **Look for "Add user or group" input** (usually at bottom of matrix)

2. **Type:** `john`

3. **Click "Add"** button

**Result: john added to job matrix:**

```
┌──────────────┬──────┬────────┬───────────┐
│ User/Group   │ Read │ Config │ Discover  │
├──────────────┼──────┼────────┼───────────┤
│ admin        │  ✓   │   ✓    │    ✓      │
│ john         │  ☐   │   ☐    │    ☐      │
└──────────────┴──────┴────────┴───────────┘
```

#### 5.5 Set john Job Permissions

1. **In john's row, find the "Read" permission**

2. **Check ONLY this permission:**
   ```
   john:
   • Read: ✓ (CHECK ONLY THIS)
   • Config: ☐ (UNCHECK)
   • Discover: ☐ (UNCHECK)
   • Build: ☐ (UNCHECK)
   • All others: ☐ (UNCHECK)
   ```

**Final job matrix:**

```
┌──────────────┬──────┬────────┬───────────┐
│ User/Group   │ Read │ Config │ Discover  │
├──────────────┼──────┼────────┼───────────┤
│ admin        │  ✓   │   ✓    │    ✓      │
│ john         │  ✓   │   ☐    │    ☐      │
└──────────────┴──────┴────────┴───────────┘
```

#### 5.6 Save Job Configuration

1. **Scroll to bottom of job configuration**

2. **Click "Save" button**

**Expected Result:**

```
Job configuration saved
(Redirected to job page)
```

---

### Step 6: Verify Configuration

#### 6.1 Test Admin User Permissions

1. **Stay logged in as admin**

2. **Verify you can:**

   - View all jobs ✓
   - Configure jobs ✓
   - View system settings ✓
   - Manage users ✓

3. **Expected result:** All actions work normally

#### 6.2 Test john User Permissions

1. **Open new browser tab (Incognito/Private mode)**

2. **Navigate to Jenkins:** `http://jenkins:8080`

3. **Login as john:**

   - Username: `john`
   - Password: `ksH85UJjhb`

4. **Verify john can:**
   - ✓ See the Dashboard (Overall Read)
   - ✓ See and view jobs (Job Read)
   - ✗ Cannot configure jobs (Job Config disabled)
   - ✗ Cannot edit system settings (Overall Config disabled)
   - ✗ Cannot delete jobs (Job Delete disabled)

**Expected Dashboard for john:**

```
Jenkins Dashboard (john's view)
├── Jobs visible (read-only)
│   ├── Existing job (can view)
│   │   ├── View job details ✓
│   │   ├── View build history ✓
│   │   ├── Cannot configure ✗
│   │   └── Cannot run ✗
│   └── Other jobs (if any)
│
├── Manage Jenkins (NOT visible - requires Overall Admin)
├── People (NOT visible)
└── Sidebar shows limited options
```

#### 6.3 Test Anonymous User Permissions

1. **Logout from john account** (or use another private tab)

2. **Access Jenkins without logging in:**

   - Navigate to `http://jenkins:8080`
   - Do NOT login

3. **Verify Anonymous user CANNOT:**
   - ✗ See Dashboard
   - ✗ See job list
   - ✗ See Manage Jenkins
   - ✗ Access system pages

**Expected Result:**

```
"Access Denied" message appears
OR
Jenkins redirects to login page
```

---

## 🔍 Verification Commands

### Verify Global Security Configuration

**Via Jenkins Web UI:**

```
1. Manage Jenkins → Configure Global Security

2. Verify settings:
   - Authorization: "Project-based Matrix Authorization Strategy"
   - Matrix shows:
     * admin: (all permissions)
     * john: (only Overall Read checked)
     * Anonymous: (all unchecked)

3. Status: ✓ Configuration correct
```

### Verify Job Configuration

**Via Jenkins Web UI:**

```
1. Job → Configure

2. Verify settings:
   - "Enable project-based security": ✓ Checked
   - Matrix shows:
     * admin: (all permissions)
     * john: (only Read checked)

3. Status: ✓ Configuration correct
```

### Verify User Exists

**Via Jenkins Web UI:**

```
1. Manage Jenkins → Manage Users

2. Users list shows:
   - admin
   - john

3. Status: ✓ User created
```

### Check Jenkins Log for Authorization Events

**Via SSH (on Jenkins server):**

```bash
# View recent logs
tail -100 /var/log/jenkins/jenkins.log | grep -i auth

# Look for:
# - User john login attempts
# - Authorization checks
# - Permission denials for Anonymous
```

---

## 🛠️ Troubleshooting Guide

### Issue 1: "Project-based Matrix Authorization Strategy" Option Not Available

**Symptom:**
Authorization section doesn't show matrix authorization option.

**Solution:**

```bash
# Method 1: Install Matrix Auth Plugin
1. Manage Jenkins → Manage Plugins → Available
2. Search: "Matrix Authorization Strategy"
3. Install: "Matrix Authorization Strategy Plugin"
4. Restart Jenkins when prompted

# Method 2: Check if Already Installed
1. Manage Jenkins → Manage Plugins → Installed
2. Search: "matrix"
3. If found but not showing, restart Jenkins:
   sudo systemctl restart jenkins

# Method 3: Manual Installation
# SSH to Jenkins server
sudo cd /var/lib/jenkins/plugins
sudo wget https://updates.jenkins.io/download/plugins/matrix-auth/latest/matrix-auth.hpi
sudo chown jenkins:jenkins matrix-auth.hpi
sudo systemctl restart jenkins
```

### Issue 2: Cannot Create User "john"

**Symptom:**
"Manage Users" option missing or user creation fails.

**Solution:**

```bash
# Method 1: Check Authentication Realm
1. Manage Jenkins → Configure Global Security
2. Security Realm must be: "Jenkins' own user database"
3. If different, change to Jenkins' own database

# Method 2: Check User Database Configuration
1. Manage Jenkins → Configure Global Security
2. Under Security Realm:
   ✓ "Jenkins' own user database" is selected
   ✓ "Allow users to sign up" is checked (or set manually)

# Method 3: Create User via Script Console
1. Manage Jenkins → Script Console
2. Paste:
   def user = jenkins.model.Jenkins.instance.getSecurityRealm()
   def passwd = user.createAccount("john", "ksH85UJjhb")
   jenkins.model.Jenkins.instance.save()
3. Run (press Ctrl+Enter)

# Method 4: Create User via CLI
# SSH to server
java -jar /usr/share/java/jenkins-cli.jar -s http://localhost:8080 \
  -auth admin:Adm!n321 create-job from-stdin << EOF
...
EOF
```

### Issue 3: john User Cannot Login

**Symptom:**
Login fails with john's credentials.

**Solution:**

```bash
# Method 1: Verify Password Entered Correctly
# Check: ksH85UJjhb (no typos)

# Method 2: Check User Exists
1. Manage Jenkins → Manage Users
2. Confirm john is in the list

# Method 3: Reset User Password
1. Manage Jenkins → Manage Users
2. Click on john
3. Click "Configure"
4. Change password to: ksH85UJjhb
5. Confirm password
6. Save

# Method 4: Check Authentication Realm
1. Manage Jenkins → Configure Global Security
2. Verify "Jenkins' own user database" is selected

# Method 5: Check Console Logs for Errors
sudo tail -50 /var/log/jenkins/jenkins.log | grep -i john
```

### Issue 4: john Can Still Access Everything (permissions not working)

**Symptom:**
john can configure jobs or access admin features.

**Solution:**

```bash
# Method 1: Verify Global Matrix Settings
1. Manage Jenkins → Configure Global Security
2. Check that john only has "Overall → Read":
   - Uncheck all other permissions
   - Ensure only Read is checked
3. Save

# Method 2: Verify Job-Level Matrix Settings
1. Job → Configure
2. Verify Authorization section:
   - john has ONLY "Read" checked
   - All other permissions unchecked
3. Save

# Method 3: Check for Anonymous Permissions
1. Manage Jenkins → Configure Global Security
2. Verify Anonymous user has NO permissions
3. If any are checked, uncheck all
4. Save

# Method 4: Verify Authorization Strategy Selected
1. Manage Jenkins → Configure Global Security
2. Confirm "Project-based Matrix Authorization Strategy" is selected
3. Not "Logged-in users can do anything"
4. Save

# Method 5: Clear Browser Cache and Retry
1. Close all Jenkins tabs
2. Clear browser cache (Ctrl+Shift+Delete)
3. Log out completely
4. Login again as john
5. Test permissions
```

### Issue 5: Anonymous User Still Has Access

**Symptom:**
Accessing Jenkins without login shows content.

**Solution:**

```bash
# Method 1: Remove All Anonymous Permissions
1. Manage Jenkins → Configure Global Security
2. Find "Anonymous" row in matrix
3. Uncheck ALL permissions:
   - Uncheck: Overall → Administer
   - Uncheck: Overall → Read
   - Uncheck: Overall → Configure
   - Uncheck: All Job permissions
   - Uncheck: All Agent permissions
   - Uncheck: All Run permissions
4. Save

# Method 2: Disable "Allow anonymous read access"
1. Manage Jenkins → Configure Global Security
2. Look for: "Allow anonymous read access" option
3. UNCHECK if checked
4. Save

# Method 3: Verify No Group Permissions
1. Check if groups (not just users) have permissions
2. Remove any group permissions
3. Save

# Method 4: Check Job-Level Settings
1. For each job → Configure
2. Remove any Anonymous permissions in matrix
3. Save for each job

# Method 5: Test in Incognito Window
1. Open new Incognito/Private tab
2. Access: http://jenkins:8080
3. Should see login page or access denied
4. Should NOT see dashboard or jobs
```

### Issue 6: Matrix Authorization Strategy Plugin Installation Fails

**Symptom:**
Plugin installation fails or times out.

**Solution:**

```bash
# Method 1: Check Internet Connectivity
curl -I https://updates.jenkins.io
# Should return 200 OK

# Method 2: Update Plugin List
1. Manage Jenkins → Manage Plugins → Advanced
2. Click "Check now"
3. Wait for update to complete
4. Try installing again

# Method 3: Manual Installation
# SSH to Jenkins server
wget https://updates.jenkins.io/download/plugins/matrix-auth/latest/matrix-auth.hpi
sudo cp matrix-auth.hpi /var/lib/jenkins/plugins/
sudo chown jenkins:jenkins /var/lib/jenkins/plugins/matrix-auth.hpi
sudo systemctl restart jenkins

# Method 4: Check Disk Space
df -h /var/lib/jenkins
# Ensure at least 500MB free

# Method 5: Check Jenkins Logs
sudo tail -50 /var/log/jenkins/jenkins.log | grep -i error
```

### Issue 7: Jenkins Restart Required But Jobs Are Running

**Symptom:**
Cannot restart Jenkins due to running jobs.

**Solution:**

```bash
# Method 1: Wait for Jobs to Complete
1. Dashboard → Build Queue
2. Wait for all jobs to finish
3. Then restart

# Method 2: Force Restart (via Jenkins UI)
1. Manage Jenkins → Prepare for Shutdown
2. Wait 60 seconds
3. Jenkins will restart when safe

# Method 3: Force Stop (via CLI)
# SSH to Jenkins server
sudo systemctl stop jenkins
# Wait 5 seconds
sudo systemctl start jenkins

# Method 4: Cancel Running Jobs
1. Job → Select build
2. Click "Stop" or "Abort"
3. Wait for cancellation
4. Then restart

# Method 5: Restart at Off-Peak Time
# Schedule restart when no builds running
# Check: Dashboard → Build Queue is empty
```

---

## 📚 Jenkins Authorization Strategies Explained

### 1. Anyone can do anything

```
├─ Anonymous user: Full access
├─ Authenticated user: Full access
└─ No access control
```

### 2. Legacy mode

```
├─ Similar to "Anyone can do anything"
├─ Deprecated in newer Jenkins versions
└─ For backward compatibility only
```

### 3. Logged-in users can do anything

```
├─ Anonymous user: No access
├─ Authenticated user: Full access
└─ Basic security level
```

### 4. Project-based Matrix Authorization Strategy ✓ (RECOMMENDED)

```
├─ Global level permissions matrix
├─ Job-level permissions matrix
├─ Group-based permissions
├─ User-specific permissions
├─ Fine-grained access control
└─ What we're using in this task
```

### 5. Role-based Strategy (Plugin required)

```
├─ Define roles (Admin, Developer, Viewer)
├─ Assign users to roles
├─ Assign permissions to roles
└─ Easier for large teams
```

---

## 🔒 Jenkins Permission Types

### Overall Permissions

```
• Administer       - Full Jenkins administration
• Configure        - Modify Jenkins configuration
• Create           - Create jobs/agents
• Delete           - Delete jobs/agents
• Discover         - List jobs/agents
• Extend Vote      - Extended voting capability
• Move             - Move/rename jobs
• Read             - View Jenkins and jobs
• RunScript        - Execute scripts
• Upload           - Upload files
```

### Job Permissions

```
• Build            - Trigger job builds
• Cancel           - Cancel running builds
• Configure        - Modify job configuration
• Delete           - Delete jobs
• Discover         - List jobs
• Move             - Move/rename jobs
• Read             - View job details
• Workspace        - Access job workspace
```

### Agent Permissions

```
• Build            - Build on agent
• Connect          - Connect agents
• Create           - Create agents
• Delete           - Delete agents
• Disconnect       - Disconnect agents
```

### Run Permissions

```
• Delete           - Delete builds
• Update           - Update build metadata
```

---

## 📊 Common Permission Configurations

### Configuration 1: Admin User (Full Access)

```yaml
Overall:
  - Administer: ✓
  - Configure: ✓
  - Create: ✓
  - Delete: ✓
  - Discover: ✓
  - Read: ✓

Job:
  - Build: ✓
  - Cancel: ✓
  - Configure: ✓
  - Delete: ✓
  - Read: ✓
  - Workspace: ✓
```

### Configuration 2: Developer (Read & Build)

```yaml
Overall:
  - Read: ✓
  - (all others): ☐

Job:
  - Build: ✓
  - Read: ✓
  - (all others): ☐
```

### Configuration 3: Read-Only User (View Only)

```yaml
Overall:
  - Read: ✓
  - (all others): ☐

Job:
  - Read: ✓
  - Discover: ✓
  - (all others): ☐
```

### Configuration 4: Anonymous (No Access)

```yaml
Overall:
  - (all): ☐

Job:
  - (all): ☐
```

---

## 📸 Expected UI Screens

### Matrix Authorization Strategy Screen

```
┌─────────────────────────────────────────────────────────┐
│ Authorization                                           │
│                                                         │
│ ◉ Project-based Matrix Authorization Strategy          │
│                                                         │
│ User/Group Access Control List:                        │
│                                                         │
│ ┌────────────┬────────┬────────┬────────┬────────────┐ │
│ │ User/Group │ Overall│ Job    │ Agent  │ Run        │ │
│ │            │ Perms  │ Perms  │ Perms  │ Perms      │ │
│ ├────────────┼────────┼────────┼────────┼────────────┤ │
│ │ admin      │  (✓✓✓) │  (✓✓) │  (✓✓) │ (✓✓)       │ │
│ │ john       │   (✓)  │  (✓)  │  ( )  │ ( )        │ │
│ │ Anonymous  │  ( )   │  ( )  │  ( )  │ ( )        │ │
│ └────────────┴────────┴────────┴────────┴────────────┘ │
│                                                         │
│ Add user or group: [  john   ] [Add]                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Job Configuration - Authorization Section

```
┌─────────────────────────────────────────────────────────┐
│ Authorization                                           │
│                                                         │
│ ☑ Enable project-based security                        │
│                                                         │
│ User/Group Access Control List:                        │
│                                                         │
│ ┌────────────┬─────┬────────┬─────────┐               │
│ │ User/Group │Read │ Config │ Build   │               │
│ ├────────────┼─────┼────────┼─────────┤               │
│ │ admin      │ ✓   │  ✓     │  ✓      │               │
│ │ john       │ ✓   │  ☐     │  ☐      │               │
│ └────────────┴─────┴────────┴─────────┘               │
│                                                         │
│ Add user or group: [ john ] [Add]                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 🧹 Cleanup (Optional)

### Delete User (If Needed)

```bash
# Via Jenkins Web UI:
1. Manage Jenkins → Manage Users
2. Click on user to delete
3. Click "Delete User"
4. Confirm deletion

# Via CLI:
java -jar jenkins-cli.jar -s http://localhost:8080 \
  -auth admin:Adm!n321 delete-user john
```

### Reset Authorization (Back to Default)

```bash
# Via Web UI:
1. Manage Jenkins → Configure Global Security
2. Change Authorization to: "Logged-in users can do anything"
3. Save

# Via Script Console:
1. Manage Jenkins → Script Console
2. Paste: Jenkins.instance.authorizationStrategy = \
     new hudson.security.AuthorizationStrategy.Unsecured()
3. Run
```

---

## 📚 Key Takeaways

### ✅ What We Learned

1. **User Management:**

   - Created new Jenkins user with credentials
   - Full name configuration
   - Password security best practices

2. **Authentication vs Authorization:**

   - Authentication: "Who are you?" (user login)
   - Authorization: "What can you do?" (permissions)

3. **Matrix Authorization Strategy:**

   - Configured global-level permissions
   - Set up job-level permissions
   - Understood permission hierarchy

4. **Permission Types:**

   - Overall permissions (global access)
   - Job permissions (job-specific access)
   - Principle of least privilege (john gets only Read)

5. **Security Best Practices:**
   - Remove anonymous access
   - Use strong passwords
   - Grant minimal necessary permissions
   - Maintain admin privileges

### 🎓 Skills Developed

- ✅ Jenkins user creation
- ✅ Matrix authorization configuration
- ✅ Permission matrix management
- ✅ Global vs job-level security
- ✅ Plugin installation and management
- ✅ Jenkins restart procedures
- ✅ Access control testing
- ✅ Security troubleshooting

### 🚀 Next Steps

1. **Create Additional Users:**

   - Developers with build permissions
   - QA team with read/discover permissions
   - DevOps with full permissions

2. **Implement Role-Based Strategy:**

   - Install Role-based Authorization plugin
   - Define custom roles
   - Assign users to roles

3. **Configure Advanced Security:**

   - LDAP/Active Directory integration
   - API token management
   - Credential access control
   - Audit logging

4. **Set Up Teams/Groups:**

   - Organize users by team
   - Assign team-based permissions
   - Simplify permission management

5. **Implement SSO:**
   - SAML authentication
   - OAuth integration
   - Enterprise directory sync

---

## 🔗 Additional Resources

- **Official Documentation:**

  - [Jenkins Security Overview](https://www.jenkins.io/doc/book/security/)
  - [Authorization](https://www.jenkins.io/doc/book/security/authorization/)
  - [Matrix Authorization Strategy](https://plugins.jenkins.io/matrix-auth/)

- **User Management:**

  - [Managing Users](https://www.jenkins.io/doc/book/managing/security/)
  - [Creating Users Programmatically](https://www.jenkins.io/doc/book/security/managing-users/)

- **Best Practices:**

  - [Jenkins Security Best Practices](https://www.jenkins.io/doc/book/security/security-best-practices/)
  - [Role-based Strategy](https://plugins.jenkins.io/role-strategy/)

- **Community:**
  - [Jenkins Community Forums](https://community.jenkins.io/)
  - [Jenkins Security Advisory](https://www.jenkins.io/security/)

---

## 📝 Summary

In this task, we successfully:

1. ✅ **Accessed Jenkins UI** with admin credentials
2. ✅ **Created new user** john with secure password:
   - Username: `john`
   - Password: `ksH85UJjhb`
   - Full Name: `John`
3. ✅ **Installed Matrix Authorization Strategy plugin** (if needed)
4. ✅ **Configured global-level permissions:**
   - Admin: Full access (Administer)
   - John: Read-only access (Overall Read)
   - Anonymous: No access (all permissions removed)
5. ✅ **Configured job-level permissions:**
   - John: Read-only on jobs
   - Other permissions (Config, Build, etc.) denied
6. ✅ **Tested access permissions:**
   - Admin: Full access verified
   - John: Read-only access verified
   - Anonymous: Access denied verified

The Jenkins server is now properly configured with fine-grained access control for the Nautilus development team! 🎉

---

**Difficulty Level:** 🟡 Intermediate
**Estimated Time:** 25-35 minutes
**Prerequisites:** Jenkins installation (Day 68), plugin installation (Day 69), basic security concepts

---
