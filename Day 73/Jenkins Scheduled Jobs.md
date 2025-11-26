# Day 73: Jenkins Scheduled Jobs

## Task Requirements

The DevOps team of xFusionCorp Industries is working on to setup centralised logging management system to maintain and analyse server logs easily. Since it will take some time to implement, they wanted to gather some server logs on a regular basis. At least one of the app servers is having issues with the Apache server. The team needs Apache logs so that they can identify and troubleshoot the issues easily if they arise. So they decided to create a Jenkins job to collect logs from the server.

### Objectives

1. Access Jenkins UI and login (username: `admin`, password: `Adm!n321`)
2. Create a Jenkins job named `copy-logs`
3. Configure it to periodically build every 2 minutes
4. Copy Apache logs (both `access_log` and `error_log`) from App Server 2 to Storage Server
5. Logs should be copied from default Apache logs location to `/usr/src/security` on Storage Server

---

## Solution

### Step 1: Access Jenkins UI

1. Click on the Jenkins button in the top bar
2. Log in with credentials:
   - **Username**: `admin`
   - **Password**: `Adm!n321`

### Step 2: Create the Jenkins Job

1. From the Jenkins dashboard, click **New Item**
2. Enter the job name: `copy-logs`
3. Select **Freestyle project**
4. Click **OK**

### Step 3: Configure Periodic Build Trigger

In the job configuration page:

1. Scroll to the **Build Triggers** section
2. Check the box **Build periodically**
3. Enter the cron expression in the **Schedule** field:

   ```
   */2 * * * *
   ```

   **Explanation**: This cron expression means:

   - `*/2` = Every 2 minutes
   - `*` = Every hour
   - `*` = Every day of month
   - `*` = Every month
   - `*` = Every day of week

### Step 4: Configure Build Steps to Copy Logs

1. Scroll to the **Build** section
2. Click **Add build step** → **Execute shell**
3. Enter the following script:

#### Option A: Using sshpass (Direct SSH Copy)

```bash
# Copy Apache logs from App Server 2 to Storage Server
sshpass -p 'Bl@kW' scp -o StrictHostKeyChecking=no \
  /var/log/httpd/access_log \
  /var/log/httpd/error_log \
  natasha@ststor01:/usr/src/security/
```

#### Option B: Using SSH with All Log Files

```bash
# Copy all Apache logs from App Server 2 to Storage Server
sshpass -p 'Bl@kW' scp -o StrictHostKeyChecking=no \
  /var/log/httpd/* \
  natasha@ststor01:/usr/src/security/
```

#### Option C: Using SSH Agent (More Secure)

If SSH keys are configured:

```bash
# Copy Apache logs using SSH key authentication
scp -o StrictHostKeyChecking=no \
  /var/log/httpd/access_log \
  /var/log/httpd/error_log \
  natasha@ststor01:/usr/src/security/
```

### Step 5: Add Logging and Error Handling (Recommended)

For better visibility and troubleshooting, use an enhanced script:

```bash
#!/bin/bash

# Configuration
APP_SERVER="stapp02"  # App Server 2
STORAGE_SERVER="ststor01"
STORAGE_USER="natasha"
STORAGE_PASSWORD="Bl@kW"
LOG_SOURCE="/var/log/httpd"
LOG_DEST="/usr/src/security"

# Logging
echo "=========================================="
echo "Jenkins Log Copy Job"
echo "=========================================="
echo "Timestamp: $(date '+%Y-%m-%d %H:%M:%S')"
echo "Build Number: $BUILD_NUMBER"
echo "Source: ${LOG_SOURCE} on ${APP_SERVER}"
echo "Destination: ${LOG_DEST} on ${STORAGE_SERVER}"
echo "=========================================="

# Check if source logs exist
if [ ! -d "$LOG_SOURCE" ]; then
    echo "ERROR: Apache log directory not found at $LOG_SOURCE"
    exit 1
fi

# Count log files
LOG_COUNT=$(ls -1 ${LOG_SOURCE}/*.log 2>/dev/null | wc -l)
echo "Found $LOG_COUNT log files to copy"

# Copy logs with error handling
echo "Copying Apache logs..."
if sshpass -p "${STORAGE_PASSWORD}" scp -o StrictHostKeyChecking=no \
   ${LOG_SOURCE}/access_log \
   ${LOG_SOURCE}/error_log \
   ${STORAGE_USER}@${STORAGE_SERVER}:${LOG_DEST}/; then
    echo "✓ Logs copied successfully"
    echo "Files copied:"
    echo "  - access_log"
    echo "  - error_log"
    exit 0
else
    echo "✗ Failed to copy logs"
    exit 1
fi
```

### Step 6: Save and Verify the Job

1. Click **Save** at the bottom of the configuration page
2. You will be redirected to the job's main page
3. Wait for the first scheduled build (within 2 minutes)
4. Alternatively, click **Build Now** to test immediately

### Step 7: Verify Build Execution

1. Check the **Build History** panel on the left
2. Click on the latest build number
3. Click **Console Output** to verify the logs were copied successfully
4. Look for success messages in the output

---

## Understanding Jenkins Cron Syntax

Jenkins uses a cron-like syntax with **5 fields**:

```
MINUTE HOUR DOM MONTH DOW
```

| Field  | Values | Description                          |
| ------ | ------ | ------------------------------------ |
| MINUTE | 0-59   | Minute of the hour                   |
| HOUR   | 0-23   | Hour of the day (24-hour format)     |
| DOM    | 1-31   | Day of the month                     |
| MONTH  | 1-12   | Month of the year                    |
| DOW    | 0-7    | Day of the week (0 and 7 are Sunday) |

### Common Cron Expression Examples

| Expression        | Description                                     |
| ----------------- | ----------------------------------------------- |
| `*/2 * * * *`     | Every 2 minutes                                 |
| `*/5 * * * *`     | Every 5 minutes                                 |
| `0 * * * *`       | Every hour (at minute 0)                        |
| `0 0 * * *`       | Daily at midnight                               |
| `0 2 * * *`       | Daily at 2:00 AM                                |
| `0 0 * * 0`       | Weekly on Sunday at midnight                    |
| `0 0 1 * *`       | Monthly on the 1st at midnight                  |
| `H * * * *`       | Every hour at a random minute (load balancing)  |
| `H/15 * * * *`    | Every 15 minutes (with hash for load balancing) |
| `H(0-29) * * * *` | Once every hour, between minute 0-29            |

### Jenkins Special Symbols

- `*` : Any value
- `*/n` : Every n units (e.g., `*/5` means every 5 minutes)
- `H` : Hash symbol for even distribution (prevents all jobs from starting at once)
- `n-m` : Range (e.g., `0-15` means minutes 0 through 15)
- `n,m,o` : List (e.g., `0,15,30,45` means at minutes 0, 15, 30, and 45)

---

## Example Console Output

```
Started by timer
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/copy-logs
[copy-logs] $ /bin/sh -xe /tmp/jenkins1234567890.sh
+ echo ==========================================
==========================================
+ echo Jenkins Log Copy Job
Jenkins Log Copy Job
+ echo ==========================================
==========================================
++ date '+%Y-%m-%d %H:%M:%S'
+ echo 'Timestamp: 2025-11-27 10:30:00'
Timestamp: 2025-11-27 10:30:00
+ echo 'Build Number: 5'
Build Number: 5
+ echo 'Source: /var/log/httpd on stapp02'
Source: /var/log/httpd on stapp02
+ echo 'Destination: /usr/src/security on ststor01'
Destination: /usr/src/security on ststor01
+ echo ==========================================
==========================================
+ echo 'Copying Apache logs...'
Copying Apache logs...
+ sshpass -p Bl@kW scp -o StrictHostKeyChecking=no /var/log/httpd/access_log /var/log/httpd/error_log natasha@ststor01:/usr/src/security/
+ echo '✓ Logs copied successfully'
✓ Logs copied successfully
+ echo 'Files copied:'
Files copied:
+ echo '  - access_log'
  - access_log
+ echo '  - error_log'
  - error_log
Finished: SUCCESS
```

---

## Advanced Configuration

### Adding Timestamps to Copied Files

To avoid overwriting logs with each copy, add timestamps:

```bash
#!/bin/bash

TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
STORAGE_USER="natasha"
STORAGE_SERVER="ststor01"
STORAGE_PASS="Bl@kW"
DEST_DIR="/usr/src/security"

echo "Copying logs with timestamp: $TIMESTAMP"

# Copy access log with timestamp
sshpass -p "${STORAGE_PASS}" scp -o StrictHostKeyChecking=no \
  /var/log/httpd/access_log \
  ${STORAGE_USER}@${STORAGE_SERVER}:${DEST_DIR}/access_log_${TIMESTAMP}

# Copy error log with timestamp
sshpass -p "${STORAGE_PASS}" scp -o StrictHostKeyChecking=no \
  /var/log/httpd/error_log \
  ${STORAGE_USER}@${STORAGE_SERVER}:${DEST_DIR}/error_log_${TIMESTAMP}

echo "Logs copied with timestamp: $TIMESTAMP"
```

### Compressing Logs Before Transfer

To save bandwidth and storage:

```bash
#!/bin/bash

TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
ARCHIVE_NAME="apache_logs_${TIMESTAMP}.tar.gz"

echo "Creating compressed archive: $ARCHIVE_NAME"

# Create compressed archive
tar -czf /tmp/${ARCHIVE_NAME} -C /var/log/httpd access_log error_log

# Copy compressed archive
sshpass -p "Bl@kW" scp -o StrictHostKeyChecking=no \
  /tmp/${ARCHIVE_NAME} \
  natasha@ststor01:/usr/src/security/

# Clean up local archive
rm -f /tmp/${ARCHIVE_NAME}

echo "Compressed logs copied successfully"
```

### Log Rotation Management

Keep only the last N log copies on storage server:

```bash
#!/bin/bash

STORAGE_USER="natasha"
STORAGE_SERVER="ststor01"
STORAGE_PASS="Bl@kW"
DEST_DIR="/usr/src/security"
KEEP_LOGS=10  # Keep only last 10 copies

# Copy current logs
TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
sshpass -p "${STORAGE_PASS}" scp -o StrictHostKeyChecking=no \
  /var/log/httpd/access_log \
  ${STORAGE_USER}@${STORAGE_SERVER}:${DEST_DIR}/access_log_${TIMESTAMP}

sshpass -p "${STORAGE_PASS}" scp -o StrictHostKeyChecking=no \
  /var/log/httpd/error_log \
  ${STORAGE_USER}@${STORAGE_SERVER}:${DEST_DIR}/error_log_${TIMESTAMP}

# Remove old logs (keep only last N)
sshpass -p "${STORAGE_PASS}" ssh -o StrictHostKeyChecking=no \
  ${STORAGE_USER}@${STORAGE_SERVER} \
  "cd ${DEST_DIR} && ls -t access_log_* | tail -n +$((KEEP_LOGS+1)) | xargs -r rm && \
   cd ${DEST_DIR} && ls -t error_log_* | tail -n +$((KEEP_LOGS+1)) | xargs -r rm"

echo "Log rotation completed - keeping last $KEEP_LOGS copies"
```

---

## Troubleshooting

### Issue 1: Job Not Running on Schedule

**Symptoms**: Build history shows no automatic builds

**Solution**:

- Verify the cron expression is correct
- Check Jenkins system time: `Manage Jenkins` → `System Information`
- Ensure "Build periodically" is checked and saved
- Check Jenkins system logs for scheduler errors
- Wait the full interval (2 minutes) for first execution

### Issue 2: SSH Connection Fails

**Symptoms**: Error message about connection refused or authentication failure

**Solution**:

```bash
# Test SSH connection manually
sshpass -p 'Bl@kW' ssh -o StrictHostKeyChecking=no natasha@ststor01 'echo "Connection successful"'

# Check if sshpass is installed
which sshpass || sudo yum install -y sshpass

# Verify network connectivity
ping -c 3 ststor01
```

### Issue 3: Permission Denied on Source Files

**Symptoms**: Cannot read Apache log files

**Solution**:

```bash
# Check log file permissions
ls -l /var/log/httpd/

# Add jenkins user to apache group
sudo usermod -a -G apache jenkins

# Or grant read permissions
sudo chmod 644 /var/log/httpd/access_log /var/log/httpd/error_log
```

### Issue 4: Permission Denied on Destination Directory

**Symptoms**: Cannot write to destination directory on storage server

**Solution**:

```bash
# On storage server, check directory permissions
ssh natasha@ststor01 'ls -ld /usr/src/security'

# Create directory if it doesn't exist
ssh natasha@ststor01 'sudo mkdir -p /usr/src/security && sudo chown natasha:natasha /usr/src/security'

# Set proper permissions
ssh natasha@ststor01 'sudo chmod 755 /usr/src/security'
```

### Issue 5: Logs Not Found

**Symptoms**: No such file or directory error

**Solution**:

```bash
# Find actual Apache log location
find /var/log -name "access_log" -o -name "error_log" 2>/dev/null

# Check Apache configuration
grep "CustomLog\|ErrorLog" /etc/httpd/conf/httpd.conf

# Common locations:
# - /var/log/httpd/ (RHEL/CentOS)
# - /var/log/apache2/ (Debian/Ubuntu)
# - /var/log/apache/ (some systems)
```

---

## Best Practices

### 1. Use Hash for Load Distribution

Instead of exact times, use hash for better distribution:

```
H/2 * * * *  # Every 2 minutes, but at different times for different jobs
```

### 2. Add Email Notifications

Configure post-build actions to send emails on failure:

1. Go to **Post-build Actions**
2. Add **Email Notification**
3. Enter recipient emails
4. Check "Send e-mail for every unstable build"

### 3. Archive Build Artifacts

Save a copy of logs in Jenkins for quick access:

1. Add post-build action: **Archive the artifacts**
2. Set files to archive: `*.log`
3. Access archived files from build page

### 4. Use Jenkins Credentials Store

Instead of hardcoding passwords:

1. Go to **Manage Jenkins** → **Manage Credentials**
2. Add SSH credentials
3. Use credentials binding in job:
   ```groovy
   withCredentials([sshUserPrivateKey(...)]) {
       // SSH commands here
   }
   ```

### 5. Monitor Disk Space

Ensure destination server has enough space:

```bash
# Add disk space check before copying
DEST_SPACE=$(sshpass -p "Bl@kW" ssh -o StrictHostKeyChecking=no \
  natasha@ststor01 "df -h /usr/src/security | tail -1 | awk '{print \$5}' | sed 's/%//'")

if [ "$DEST_SPACE" -gt 90 ]; then
    echo "WARNING: Destination disk usage is ${DEST_SPACE}%"
    echo "Cleaning up old logs..."
    # Add cleanup logic here
fi
```

---

## Pipeline Equivalent

For reference, here's the Pipeline syntax version:

```groovy
pipeline {
    agent any

    triggers {
        cron('*/2 * * * *')  // Every 2 minutes
    }

    environment {
        STORAGE_USER = 'natasha'
        STORAGE_SERVER = 'ststor01'
        STORAGE_PASS = 'Bl@kW'
        LOG_SOURCE = '/var/log/httpd'
        LOG_DEST = '/usr/src/security'
    }

    stages {
        stage('Copy Apache Logs') {
            steps {
                script {
                    echo "Copying Apache logs from App Server 2 to Storage Server"
                    sh """
                        sshpass -p '${STORAGE_PASS}' scp -o StrictHostKeyChecking=no \
                            ${LOG_SOURCE}/access_log \
                            ${LOG_SOURCE}/error_log \
                            ${STORAGE_USER}@${STORAGE_SERVER}:${LOG_DEST}/
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✓ Logs copied successfully at ${new Date()}"
        }
        failure {
            echo "✗ Failed to copy logs"
        }
    }
}
```

---

## Monitoring and Maintenance

### Check Job Execution History

```bash
# View recent builds
# From Jenkins CLI
java -jar jenkins-cli.jar -s http://localhost:8080/ list-jobs

# Check specific job builds
java -jar jenkins-cli.jar -s http://localhost:8080/ get-job copy-logs
```

### Set Up Build Notifications

1. Install **Slack Notification Plugin** or **Email Extension Plugin**
2. Configure in job's Post-build Actions
3. Get notified on failures for immediate action

### Monitor Storage Server Disk Usage

Create a separate monitoring job:

```bash
#!/bin/bash
THRESHOLD=80
USAGE=$(ssh natasha@ststor01 "df -h /usr/src/security | tail -1 | awk '{print \$5}' | sed 's/%//'")

if [ "$USAGE" -gt "$THRESHOLD" ]; then
    echo "ALERT: Disk usage at ${USAGE}% exceeds threshold of ${THRESHOLD}%"
    # Send alert
    exit 1
fi
```

---

## Key Takeaways

1. ✅ Jenkins cron syntax enables precise scheduling of automated tasks
2. ✅ `*/2 * * * *` runs a job every 2 minutes
3. ✅ `sshpass` allows password-based SSH authentication in scripts
4. ✅ Regular log collection helps with troubleshooting and analysis
5. ✅ Error handling and logging improve job reliability
6. ✅ Scheduled jobs are ideal for recurring maintenance tasks
7. ✅ Hash symbol (H) in cron helps distribute load evenly
8. ✅ Monitoring scheduled jobs ensures continuous operation

---

## Validation Checklist

- ✅ Job `copy-logs` created successfully
- ✅ Build trigger configured to run every 2 minutes
- ✅ SSH connection to storage server works
- ✅ Apache logs (access_log and error_log) are copied
- ✅ Destination directory `/usr/src/security` receives files
- ✅ Job executes automatically on schedule
- ✅ Console output shows successful execution
- ✅ Multiple builds complete successfully over time

---

## Additional Resources

- [Jenkins Cron Syntax Documentation](https://www.jenkins.io/doc/book/pipeline/syntax/#cron-syntax)
- [Jenkins Build Triggers](https://www.jenkins.io/doc/book/pipeline/syntax/#triggers)
- [Crontab Guru - Cron Expression Editor](https://crontab.guru/)
- [SSH Copy (SCP) Command Guide](https://www.ssh.com/academy/ssh/scp)
- [Apache HTTP Server Log Files](https://httpd.apache.org/docs/2.4/logs.html)

---

**Task Completed Successfully! ✓**

The Jenkins scheduled job is now configured to automatically collect Apache logs every 2 minutes from App Server 2 and copy them to the Storage Server. This provides the DevOps team with regular log backups for troubleshooting and analysis while they work on implementing the centralized logging management system.
