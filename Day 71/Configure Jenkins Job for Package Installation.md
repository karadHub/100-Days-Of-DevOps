# Day 71: Configure Jenkins Job for Package Installation

## Task Requirements

Some new requirements have come up to install and configure some packages on the Nautilus infrastructure under Stratos Datacenter. The Nautilus DevOps team installed and configured a new Jenkins server so they wanted to create a Jenkins job to automate this task.

### Objectives

1. Access the Jenkins UI and log in with credentials (username: `admin`, password: `Adm!n321`)
2. Create a new Jenkins job named `install-packages`
3. Configure the job with:
   - A string parameter named `PACKAGE`
   - Automation to install the package specified in `$PACKAGE` parameter on the storage server

### Important Notes

- Install any required plugins and restart Jenkins service if necessary
- Use "Restart Jenkins when installation is complete and no jobs are running"
- Verify the Jenkins job runs successfully on repeated executions

---

## Solution

### Step 1: Access Jenkins UI

1. Click on the Jenkins button in the top bar
2. Log in with credentials:
   - **Username**: `admin`
   - **Password**: `Adm!n321`

### Step 2: Install Required Plugins (if needed)

1. Navigate to **Manage Jenkins** → **Manage Plugins**
2. Go to the **Available** tab
3. Install the following plugins if not already installed:
   - **SSH Plugin** or **Publish Over SSH** (for remote command execution)
   - **Parameterized Trigger Plugin** (for parameter support)
4. Check "Restart Jenkins when installation is complete and no jobs are running"
5. Wait for Jenkins to restart and refresh the UI page

### Step 3: Configure SSH Connection to Storage Server

Before creating the job, set up SSH credentials for the storage server:

1. Go to **Manage Jenkins** → **Manage Credentials**
2. Click on **(global)** domain
3. Click **Add Credentials**
4. Configure the credentials:
   - **Kind**: SSH Username with private key
   - **Scope**: Global
   - **ID**: `storage-server-ssh`
   - **Description**: Storage Server SSH Credentials
   - **Username**: (appropriate user with sudo privileges, typically `natasha` or similar)
   - **Private Key**: Enter directly or from file
   - Click **OK**

Alternatively, if using **Publish Over SSH** plugin:

1. Go to **Manage Jenkins** → **Configure System**
2. Scroll to **Publish over SSH** section
3. Add SSH Server:
   - **Name**: `storage-server`
   - **Hostname**: (storage server hostname/IP)
   - **Username**: (user with sudo privileges)
   - **Remote Directory**: `/tmp`
4. Click **Test Configuration** to verify
5. Save the configuration

### Step 4: Create the Jenkins Job

1. From the Jenkins dashboard, click **New Item**
2. Enter the job name: `install-packages`
3. Select **Freestyle project**
4. Click **OK**

### Step 5: Configure Job Parameters

In the job configuration page:

1. Check the box **This project is parameterized**
2. Click **Add Parameter** → **String Parameter**
3. Configure the parameter:
   - **Name**: `PACKAGE`
   - **Default Value**: (leave empty or set a default like `httpd`)
   - **Description**: `Name of the package to install on the storage server`

### Step 6: Configure Build Steps

#### Option A: Using SSH Plugin

1. Scroll to **Build** section
2. Click **Add build step** → **Execute shell script on remote host using ssh**
3. Configure:

   - **SSH site**: Select your configured storage server
   - **Command**:

   ```bash
   # Update package cache
   sudo yum update -y

   # Install the package
   sudo yum install -y $PACKAGE

   # Verify installation
   rpm -qa | grep $PACKAGE
   ```

#### Option B: Using Publish Over SSH Plugin

1. Scroll to **Build** section
2. Click **Add build step** → **Send files or execute commands over SSH**
3. Configure:
   - **Name**: Select `storage-server`
   - **Exec command**:
   ```bash
   sudo yum install -y $PACKAGE && echo "Package $PACKAGE installed successfully" || echo "Failed to install $PACKAGE"
   ```

#### Option C: Using SSH Agent Plugin (Most Flexible)

1. Click **Add build step** → **Execute shell**
2. Add the following script:

   ```bash
   ssh -o StrictHostKeyChecking=no user@storage-server << 'ENDSSH'
   # Update package cache
   sudo yum update -y

   # Install the package
   sudo yum install -y $PACKAGE

   # Verify installation
   if rpm -qa | grep -q $PACKAGE; then
       echo "Package $PACKAGE installed successfully"
       exit 0
   else
       echo "Failed to install package $PACKAGE"
       exit 1
   fi
   ENDSSH
   ```

### Step 7: Add Post-Build Actions (Optional but Recommended)

1. Scroll to **Post-build Actions**
2. Add **Editable Email Notification** or **Email Notification** to notify on success/failure
3. Configure email recipients and triggers as needed

### Step 8: Save and Test the Job

1. Click **Save** at the bottom of the configuration page
2. From the job page, click **Build with Parameters**
3. Enter a test package name (e.g., `tree`, `wget`, `vim`)
4. Click **Build**
5. Monitor the **Console Output** to verify the installation

### Step 9: Verify Repeated Executions

1. Run the job multiple times with different package names
2. Run the job with the same package name to ensure idempotency
3. Verify that:
   - New packages are installed successfully
   - Already installed packages don't cause failures
   - Console output is clear and informative

---

## Example Console Output

```
Started by user admin
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/install-packages
[install-packages] $ /bin/sh -xe /tmp/jenkins1234567890.sh
+ ssh -o StrictHostKeyChecking=no natasha@storage-server
Loaded plugins: fastestmirror, ovl
Loading mirror speeds from cached hostfile
 * base: mirror.example.com
 * extras: mirror.example.com
 * updates: mirror.example.com
Resolving Dependencies
--> Running transaction check
---> Package tree.x86_64 0:1.7.0-15.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

Package installed successfully
Finished: SUCCESS
```

---

## Troubleshooting

### Issue 1: SSH Connection Fails

**Solution**:

- Verify SSH credentials are correct
- Ensure storage server is accessible from Jenkins server
- Check SSH key permissions (should be 600)
- Test SSH connection manually: `ssh user@storage-server`

### Issue 2: Permission Denied During Package Installation

**Solution**:

- Ensure the user has sudo privileges
- Configure passwordless sudo for the user:
  ```bash
  echo "natasha ALL=(ALL) NOPASSWD: /usr/bin/yum" | sudo tee /etc/sudoers.d/natasha
  ```
- Or configure SSH to use root user (less secure)

### Issue 3: Package Already Installed Error

**Solution**:

- Modify the command to handle already-installed packages:
  ```bash
  sudo yum install -y $PACKAGE || echo "Package may already be installed"
  rpm -qa | grep $PACKAGE && echo "Package verified"
  ```

### Issue 4: Variable $PACKAGE Not Expanding

**Solution**:

- Ensure parameter name matches exactly: `PACKAGE`
- Use proper syntax: `$PACKAGE` or `${PACKAGE}`
- Check that "This project is parameterized" is enabled

---

## Best Practices

1. **Error Handling**: Add proper error checking in the script

   ```bash
   if sudo yum install -y $PACKAGE; then
       echo "✓ Package $PACKAGE installed successfully"
       exit 0
   else
       echo "✗ Failed to install package $PACKAGE"
       exit 1
   fi
   ```

2. **Logging**: Include timestamps and clear messages

   ```bash
   echo "[$(date)] Starting installation of package: $PACKAGE"
   ```

3. **Validation**: Verify package parameter is not empty

   ```bash
   if [ -z "$PACKAGE" ]; then
       echo "ERROR: PACKAGE parameter is empty"
       exit 1
   fi
   ```

4. **Idempotency**: Ensure the job can run multiple times safely

   ```bash
   if rpm -qa | grep -q "^$PACKAGE"; then
       echo "Package $PACKAGE is already installed"
   else
       sudo yum install -y $PACKAGE
   fi
   ```

5. **Security**: Use Jenkins credentials store instead of hardcoding passwords

---

## Alternative Approaches

### Using Ansible via Jenkins

1. Install Ansible on Jenkins server
2. Create an Ansible playbook:

   ```yaml
   ---
   - name: Install package on storage server
     hosts: storage
     become: yes
     tasks:
       - name: Install package
         yum:
           name: "{{ package_name }}"
           state: present
   ```

3. Jenkins job executes:
   ```bash
   ansible-playbook -i inventory install-package.yml -e "package_name=$PACKAGE"
   ```

### Using Pipeline Job

Create a Jenkinsfile:

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'PACKAGE', description: 'Package name to install')
    }
    stages {
        stage('Install Package') {
            steps {
                script {
                    sshagent(['storage-server-ssh']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no user@storage-server '
                                sudo yum install -y ${params.PACKAGE}
                                rpm -qa | grep ${params.PACKAGE}
                            '
                        """
                    }
                }
            }
        }
    }
    post {
        success {
            echo "Package ${params.PACKAGE} installed successfully"
        }
        failure {
            echo "Failed to install package ${params.PACKAGE}"
        }
    }
}
```

---

## Validation Steps

1. ✅ Jenkins job `install-packages` created
2. ✅ String parameter `PACKAGE` configured
3. ✅ Job can connect to storage server via SSH
4. ✅ Package installation executes successfully
5. ✅ Job completes with SUCCESS status
6. ✅ Repeated executions work reliably
7. ✅ Different package names can be installed
8. ✅ Console output is clear and informative

---

## Key Takeaways

1. Jenkins parameterized builds enable flexible job configurations
2. SSH plugins provide multiple options for remote command execution
3. Proper error handling ensures job reliability
4. Idempotent operations allow safe repeated executions
5. Security best practices include using credential stores and least-privilege access
6. Testing with various scenarios validates job robustness

---

## Additional Resources

- [Jenkins Parameterized Build Documentation](https://www.jenkins.io/doc/book/pipeline/syntax/#parameters)
- [Publish Over SSH Plugin](https://plugins.jenkins.io/publish-over-ssh/)
- [SSH Agent Plugin](https://plugins.jenkins.io/ssh-agent/)
- [YUM Package Manager Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-yum)

---

**Task Completed Successfully! ✓**

The Jenkins job is now configured to install packages on the storage server using parameterized builds, providing a reusable and automated solution for package management in the Nautilus infrastructure.
