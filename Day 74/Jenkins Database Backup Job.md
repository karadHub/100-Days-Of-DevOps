# Day 74: Jenkins Database Backup Job

## Task Requirements

There is a requirement to create a Jenkins job to automate the database backup for the Stratos Datacenter infrastructure.

### Objectives

1. Access Jenkins UI and login (username: `admin`, password: `Adm!n321`)
2. Create a Jenkins job named `database-backup`
3. Configure it to take a database dump of `kodekloud_db01` database from Database server
4. Database credentials: username `kodekloud_roy`, password `asdfgdsd`
5. Dump file format: `db_$(date +%F).sql` (where date +%F is current date in YYYY-MM-DD format)
6. Copy the dump to Backup Server at location `/home/clint/db_backups`
7. Schedule the job to run periodically at `*/10 * * * *` (every 10 minutes)

### Important Notes

- Install required plugins and restart Jenkins service if necessary
- Use "Restart Jenkins when installation is complete and no jobs are running"
- Refresh UI page if Jenkins gets stuck after service restart
- Take screenshots or record screen for review purposes

---

## Solution

### Step 1: Access Jenkins UI

1. Click on the Jenkins button in the top bar
2. Log in with credentials:
   - **Username**: `admin`
   - **Password**: `Adm!n321`

### Step 2: Install Required Plugins

1. Navigate to **Manage Jenkins** → **Manage Plugins**
2. Go to the **Available** tab
3. Search and install the following plugins:
   - **SSH Plugin** or **Publish Over SSH** (for remote command execution)
   - **SSH Agent Plugin** (for SSH operations)
4. Check "Restart Jenkins when installation is complete and no jobs are running"
5. Wait for Jenkins to restart
6. Refresh the browser if the UI becomes unresponsive

### Step 3: Configure SSH Connection (Optional)

If using SSH plugins, configure the database and backup servers:

1. Go to **Manage Jenkins** → **Configure System**
2. Scroll to **Publish over SSH** section
3. Add SSH Servers:

   **Database Server**:

   - **Name**: `database-server`
   - **Hostname**: (database server hostname/IP)
   - **Username**: (SSH user)

   **Backup Server**:

   - **Name**: `backup-server`
   - **Hostname**: `stbkp01`
   - **Username**: `clint`

4. Test connections and save

### Step 4: Create the Jenkins Job

1. From the Jenkins dashboard, click **New Item**
2. Enter the job name: `database-backup`
3. Select **Freestyle project**
4. Click **OK**

### Step 5: Configure Build Triggers

In the job configuration page:

1. Scroll to the **Build Triggers** section
2. Check the box **Build periodically**
3. Enter the cron expression in the **Schedule** field:

   ```
   */10 * * * *
   ```

   **Explanation**: This runs the job every 10 minutes

   - `*/10` = Every 10 minutes
   - `*` = Every hour
   - `*` = Every day of month
   - `*` = Every month
   - `*` = Every day of week

### Step 6: Configure Build Steps

1. Scroll to the **Build** section
2. Click **Add build step** → **Execute shell**
3. Enter the following shell script:

#### Complete Backup Script

```bash
#!/bin/bash

# Configuration
DB_HOST="database-server"  # Replace with actual hostname/IP
DB_NAME="kodekloud_db01"
DB_USER="kodekloud_roy"
DB_PASS="asdfgdsd"
BACKUP_SERVER="stbkp01"
BACKUP_USER="clint"
BACKUP_PASS="H@wk3y3"
BACKUP_DIR="/home/clint/db_backups"

# Generate backup filename with current date
BACKUP_FILE="db_$(date +%F).sql"

echo "=========================================="
echo "Jenkins Database Backup Job"
echo "=========================================="
echo "Timestamp: $(date '+%Y-%m-%d %H:%M:%S')"
echo "Build Number: $BUILD_NUMBER"
echo "Database: $DB_NAME"
echo "Backup File: $BACKUP_FILE"
echo "=========================================="

# Step 1: Create database dump
echo "Creating database dump..."
mysqldump -h $DB_HOST -u $DB_USER -p$DB_PASS $DB_NAME > $BACKUP_FILE

# Check if dump was successful
if [ $? -eq 0 ]; then
    echo "✓ Database dump created successfully"

    # Get file size
    FILE_SIZE=$(ls -lh $BACKUP_FILE | awk '{print $5}')
    echo "Backup file size: $FILE_SIZE"
else
    echo "✗ Failed to create database dump"
    exit 1
fi

# Step 2: Copy backup to Backup Server
echo "Copying backup to Backup Server..."
sshpass -p "$BACKUP_PASS" scp -p -o StrictHostKeyChecking=no \
    $BACKUP_FILE ${BACKUP_USER}@${BACKUP_SERVER}:${BACKUP_DIR}/

# Check if copy was successful
if [ $? -eq 0 ]; then
    echo "✓ Backup copied successfully to ${BACKUP_SERVER}:${BACKUP_DIR}/"
else
    echo "✗ Failed to copy backup to Backup Server"
    exit 1
fi

# Step 3: Verify backup on remote server
echo "Verifying backup on remote server..."
sshpass -p "$BACKUP_PASS" ssh -o StrictHostKeyChecking=no \
    ${BACKUP_USER}@${BACKUP_SERVER} \
    "ls -lh ${BACKUP_DIR}/${BACKUP_FILE}"

if [ $? -eq 0 ]; then
    echo "✓ Backup verified on remote server"
else
    echo "⚠ Warning: Could not verify backup on remote server"
fi

# Step 4: Cleanup local backup file
echo "Cleaning up local backup file..."
rm -f $BACKUP_FILE

echo "=========================================="
echo "✓ Database backup completed successfully!"
echo "=========================================="
```

#### Simplified Version (Minimal Script)

```bash
#!/bin/bash

# Create database dump
mysqldump -u kodekloud_roy -pasdfgdsd kodekloud_db01 > db_$(date +%F).sql

# Copy to backup server
sshpass -p 'H@wk3y3' scp -p -o StrictHostKeyChecking=no \
    db_$(date +%F).sql clint@stbkp01:/home/clint/db_backups

# Cleanup
rm -f db_$(date +%F).sql

echo "Backup completed successfully"
```

#### Remote Execution Version

If executing on the database server directly:

```bash
#!/bin/bash

# Execute mysqldump on database server and copy to backup server
ssh -o StrictHostKeyChecking=no user@database-server << 'EOF'
    mysqldump kodekloud_db01 -u kodekloud_roy -pasdfgdsd > db_$(date +%F).sql
    sshpass -p 'H@wk3y3' scp -p -o StrictHostKeyChecking=no \
        db_$(date +%F).sql clint@stbkp01:/home/clint/db_backups
    rm -f db_$(date +%F).sql
EOF

echo "Remote backup completed successfully"
```

### Step 7: Save and Test the Job

1. Click **Save** at the bottom of the configuration page
2. From the job page, click **Build Now** to test immediately
3. Monitor the **Console Output** to verify the backup process
4. Wait for scheduled execution to confirm automation

### Step 8: Verify Backup on Backup Server

Manually verify the backup was created:

```bash
# SSH to backup server
ssh clint@stbkp01

# Check backup directory
ls -lh /home/clint/db_backups/

# Verify recent backups
ls -lt /home/clint/db_backups/ | head -5
```

---

## Example Console Output

```
Started by timer
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/database-backup
[database-backup] $ /bin/sh -xe /tmp/jenkins1234567890.sh
+ echo ==========================================
==========================================
+ echo Jenkins Database Backup Job
Jenkins Database Backup Job
+ echo ==========================================
==========================================
++ date '+%Y-%m-%d %H:%M:%S'
+ echo 'Timestamp: 2025-12-09 10:30:00'
Timestamp: 2025-12-09 10:30:00
+ echo 'Build Number: 5'
Build Number: 5
+ echo 'Database: kodekloud_db01'
Database: kodekloud_db01
++ date +%F
+ echo 'Backup File: db_2025-12-09.sql'
Backup File: db_2025-12-09.sql
+ echo ==========================================
==========================================
+ echo 'Creating database dump...'
Creating database dump...
++ date +%F
+ mysqldump -u kodekloud_roy -pasdfgdsd kodekloud_db01
+ echo '✓ Database dump created successfully'
✓ Database dump created successfully
+ echo 'Copying backup to Backup Server...'
Copying backup to Backup Server...
++ date +%F
+ sshpass -p 'H@wk3y3' scp -p -o StrictHostKeyChecking=no db_2025-12-09.sql clint@stbkp01:/home/clint/db_backups/
+ echo '✓ Backup copied successfully to stbkp01:/home/clint/db_backups/'
✓ Backup copied successfully to stbkp01:/home/clint/db_backups/
++ date +%F
+ rm -f db_2025-12-09.sql
+ echo ==========================================
==========================================
+ echo '✓ Database backup completed successfully!'
✓ Database backup completed successfully!
+ echo ==========================================
==========================================
Finished: SUCCESS
```

---

## Understanding the Backup Process

### MySQL Dump Command Breakdown

```bash
mysqldump -u kodekloud_roy -pasdfgdsd kodekloud_db01 > db_$(date +%F).sql
```

| Component              | Description                       |
| ---------------------- | --------------------------------- |
| `mysqldump`            | MySQL backup utility              |
| `-u kodekloud_roy`     | Database username                 |
| `-pasdfgdsd`           | Password (no space after -p)      |
| `kodekloud_db01`       | Database name to backup           |
| `> db_$(date +%F).sql` | Redirect output to file with date |
| `$(date +%F)`          | Current date in YYYY-MM-DD format |

### Date Format Options

| Format              | Output              | Description            |
| ------------------- | ------------------- | ---------------------- |
| `%F`                | 2025-12-09          | Full date (YYYY-MM-DD) |
| `%Y`                | 2025                | Year (4 digits)        |
| `%m`                | 12                  | Month (01-12)          |
| `%d`                | 09                  | Day (01-31)            |
| `%Y%m%d`            | 20251209            | Compact date format    |
| `%Y-%m-%d_%H-%M-%S` | 2025-12-09_10-30-00 | Date with time         |

### SCP Command Breakdown

```bash
sshpass -p 'H@wk3y3' scp -p -o StrictHostKeyChecking=no \
    db_$(date +%F).sql clint@stbkp01:/home/clint/db_backups
```

| Component                     | Description                                        |
| ----------------------------- | -------------------------------------------------- |
| `sshpass -p 'H@wk3y3'`        | Provide SSH password non-interactively             |
| `scp`                         | Secure copy over SSH                               |
| `-p`                          | Preserve file attributes (timestamps, permissions) |
| `-o StrictHostKeyChecking=no` | Disable host key verification                      |
| `db_$(date +%F).sql`          | Source file to copy                                |
| `clint@stbkp01`               | Destination user and host                          |
| `:/home/clint/db_backups`     | Destination directory                              |

---

## Advanced Configuration

### Adding Backup Compression

Compress backups to save space:

```bash
#!/bin/bash

BACKUP_FILE="db_$(date +%F).sql"
COMPRESSED_FILE="db_$(date +%F).sql.gz"

# Create and compress backup
mysqldump -u kodekloud_roy -pasdfgdsd kodekloud_db01 | gzip > $COMPRESSED_FILE

# Copy compressed backup
sshpass -p 'H@wk3y3' scp -p -o StrictHostKeyChecking=no \
    $COMPRESSED_FILE clint@stbkp01:/home/clint/db_backups/

# Cleanup
rm -f $COMPRESSED_FILE

echo "Compressed backup created: $COMPRESSED_FILE"
```

### Adding Backup Rotation

Keep only the last N backups:

```bash
#!/bin/bash

BACKUP_FILE="db_$(date +%F).sql"
KEEP_BACKUPS=7  # Keep last 7 days

# Create backup
mysqldump -u kodekloud_roy -pasdfgdsd kodekloud_db01 > $BACKUP_FILE

# Copy to backup server
sshpass -p 'H@wk3y3' scp -p -o StrictHostKeyChecking=no \
    $BACKUP_FILE clint@stbkp01:/home/clint/db_backups/

# Cleanup local file
rm -f $BACKUP_FILE

# Remove old backups on remote server
sshpass -p 'H@wk3y3' ssh -o StrictHostKeyChecking=no clint@stbkp01 \
    "cd /home/clint/db_backups && ls -t db_*.sql | tail -n +$((KEEP_BACKUPS+1)) | xargs -r rm"

echo "Backup rotation completed - keeping last $KEEP_BACKUPS backups"
```

### Adding Multiple Database Backups

Backup multiple databases:

```bash
#!/bin/bash

DATABASES=("kodekloud_db01" "kodekloud_db02" "kodekloud_db03")
DB_USER="kodekloud_roy"
DB_PASS="asdfgdsd"
DATE=$(date +%F)

for DB in "${DATABASES[@]}"; do
    echo "Backing up database: $DB"
    BACKUP_FILE="db_${DB}_${DATE}.sql"

    mysqldump -u $DB_USER -p$DB_PASS $DB > $BACKUP_FILE

    sshpass -p 'H@wk3y3' scp -p -o StrictHostKeyChecking=no \
        $BACKUP_FILE clint@stbkp01:/home/clint/db_backups/

    rm -f $BACKUP_FILE
    echo "✓ $DB backed up successfully"
done
```

### Adding Email Notifications

Send email on backup completion or failure:

```bash
#!/bin/bash

BACKUP_FILE="db_$(date +%F).sql"
EMAIL="admin@example.com"

# Create backup
if mysqldump -u kodekloud_roy -pasdfgdsd kodekloud_db01 > $BACKUP_FILE; then
    # Copy to backup server
    if sshpass -p 'H@wk3y3' scp -p -o StrictHostKeyChecking=no \
        $BACKUP_FILE clint@stbkp01:/home/clint/db_backups/; then

        # Success notification
        echo "Database backup completed successfully" | \
            mail -s "Backup Success: $BACKUP_FILE" $EMAIL
    else
        # Copy failure notification
        echo "Failed to copy backup to remote server" | \
            mail -s "Backup Failed: Copy Error" $EMAIL
        exit 1
    fi
else
    # Dump failure notification
    echo "Failed to create database dump" | \
        mail -s "Backup Failed: Dump Error" $EMAIL
    exit 1
fi

rm -f $BACKUP_FILE
```

### Adding Backup Verification

Verify backup integrity:

```bash
#!/bin/bash

BACKUP_FILE="db_$(date +%F).sql"

# Create backup
mysqldump -u kodekloud_roy -pasdfgdsd kodekloud_db01 > $BACKUP_FILE

# Verify backup is not empty and contains SQL statements
if [ -s "$BACKUP_FILE" ] && grep -q "CREATE TABLE" "$BACKUP_FILE"; then
    echo "✓ Backup file validated successfully"

    # Calculate checksum
    CHECKSUM=$(md5sum $BACKUP_FILE | awk '{print $1}')
    echo "Backup checksum: $CHECKSUM"

    # Copy to backup server
    sshpass -p 'H@wk3y3' scp -p -o StrictHostKeyChecking=no \
        $BACKUP_FILE clint@stbkp01:/home/clint/db_backups/

    # Save checksum
    echo "$CHECKSUM  $BACKUP_FILE" | \
        sshpass -p 'H@wk3y3' ssh -o StrictHostKeyChecking=no clint@stbkp01 \
        "cat >> /home/clint/db_backups/checksums.txt"
else
    echo "✗ Backup validation failed - file is empty or corrupted"
    exit 1
fi

rm -f $BACKUP_FILE
```

---

## Troubleshooting

### Issue 1: mysqldump Command Not Found

**Symptoms**: `mysqldump: command not found`

**Solution**:

```bash
# Install MySQL client tools
sudo yum install -y mysql

# Or specify full path
/usr/bin/mysqldump -u kodekloud_roy -pasdfgdsd kodekloud_db01 > backup.sql

# Find mysqldump location
which mysqldump
find / -name mysqldump 2>/dev/null
```

### Issue 2: Access Denied for Database User

**Symptoms**: `ERROR 1045 (28000): Access denied for user 'kodekloud_roy'@'host'`

**Solution**:

```bash
# Test database connection
mysql -u kodekloud_roy -pasdfgdsd kodekloud_db01 -e "SELECT 1;"

# Check user privileges
mysql -u root -p -e "SHOW GRANTS FOR 'kodekloud_roy'@'%';"

# Grant necessary privileges
mysql -u root -p -e "GRANT SELECT, LOCK TABLES ON kodekloud_db01.* TO 'kodekloud_roy'@'%';"
mysql -u root -p -e "FLUSH PRIVILEGES;"
```

### Issue 3: sshpass Not Installed

**Symptoms**: `sshpass: command not found`

**Solution**:

```bash
# Install sshpass
sudo yum install -y sshpass

# Or use SSH keys instead (more secure)
ssh-keygen -t rsa -b 4096
ssh-copy-id clint@stbkp01
```

### Issue 4: Permission Denied on Backup Directory

**Symptoms**: `scp: /home/clint/db_backups/: Permission denied`

**Solution**:

```bash
# On backup server, create and set permissions
ssh clint@stbkp01
mkdir -p /home/clint/db_backups
chmod 755 /home/clint/db_backups

# Verify ownership
ls -ld /home/clint/db_backups
# Should show: drwxr-xr-x clint clint
```

### Issue 5: Backup File Too Large

**Symptoms**: Transfer takes too long or fails

**Solution**:

```bash
# Compress during backup
mysqldump -u kodekloud_roy -pasdfgdsd kodekloud_db01 | gzip > db_$(date +%F).sql.gz

# Or exclude large tables
mysqldump -u kodekloud_roy -pasdfgdsd kodekloud_db01 \
    --ignore-table=kodekloud_db01.large_table > backup.sql

# Or use --single-transaction for InnoDB
mysqldump -u kodekloud_roy -pasdfgdsd --single-transaction kodekloud_db01 > backup.sql
```

### Issue 6: Network Connection Issues

**Symptoms**: `ssh: connect to host stbkp01 port 22: Connection refused`

**Solution**:

```bash
# Test network connectivity
ping stbkp01

# Test SSH connectivity
ssh -v clint@stbkp01

# Check if SSH service is running on backup server
ssh clint@stbkp01 "systemctl status sshd"

# Check firewall rules
ssh clint@stbkp01 "sudo iptables -L -n | grep 22"
```

---

## Best Practices

### 1. Security Best Practices

**Use SSH Keys Instead of Passwords**:

```bash
# Generate SSH key on Jenkins server
ssh-keygen -t rsa -b 4096 -f ~/.ssh/jenkins_backup_key

# Copy to backup server
ssh-copy-id -i ~/.ssh/jenkins_backup_key.pub clint@stbkp01

# Use in script without password
scp -i ~/.ssh/jenkins_backup_key db_$(date +%F).sql clint@stbkp01:/home/clint/db_backups/
```

**Store Credentials Securely**:

```bash
# Use Jenkins credentials store
# Or use environment variables
# Or use external secret management (Vault, AWS Secrets Manager)
```

### 2. Monitoring and Alerting

```bash
# Add status logging
echo "Backup started at $(date)" >> /var/log/jenkins-backups.log

# Monitor backup size
BACKUP_SIZE=$(stat -f%z "$BACKUP_FILE" 2>/dev/null || stat -c%s "$BACKUP_FILE")
if [ $BACKUP_SIZE -lt 1000 ]; then
    echo "WARNING: Backup file is too small ($BACKUP_SIZE bytes)"
fi
```

### 3. Backup Testing

Regularly test backup restoration:

```bash
# Restore to test database
mysql -u root -p test_db < db_2025-12-09.sql

# Verify table count
mysql -u root -p -e "SELECT COUNT(*) FROM information_schema.tables WHERE table_schema = 'test_db';"
```

### 4. Documentation

Document the backup process:

- Backup schedule (every 10 minutes)
- Retention policy (how long backups are kept)
- Recovery procedures (how to restore)
- Contact information for issues

### 5. Disaster Recovery Planning

```bash
# Store backups in multiple locations
scp db_$(date +%F).sql user@offsite-backup:/backups/
aws s3 cp db_$(date +%F).sql s3://backup-bucket/database/
```

---

## Pipeline Equivalent

For reference, here's the Pipeline syntax version:

```groovy
pipeline {
    agent any

    triggers {
        cron('*/10 * * * *')  // Every 10 minutes
    }

    environment {
        DB_NAME = 'kodekloud_db01'
        DB_USER = 'kodekloud_roy'
        DB_PASS = 'asdfgdsd'
        BACKUP_SERVER = 'stbkp01'
        BACKUP_USER = 'clint'
        BACKUP_PASS = 'H@wk3y3'
        BACKUP_DIR = '/home/clint/db_backups'
    }

    stages {
        stage('Create Database Backup') {
            steps {
                script {
                    def backupFile = "db_\$(date +%F).sql"
                    echo "Creating backup: ${backupFile}"

                    sh """
                        mysqldump -u ${DB_USER} -p${DB_PASS} ${DB_NAME} > ${backupFile}

                        if [ \$? -eq 0 ]; then
                            echo "✓ Database dump created successfully"
                        else
                            echo "✗ Failed to create database dump"
                            exit 1
                        fi
                    """
                }
            }
        }

        stage('Copy to Backup Server') {
            steps {
                script {
                    def backupFile = "db_\$(date +%F).sql"

                    sh """
                        sshpass -p '${BACKUP_PASS}' scp -p -o StrictHostKeyChecking=no \
                            ${backupFile} ${BACKUP_USER}@${BACKUP_SERVER}:${BACKUP_DIR}/

                        if [ \$? -eq 0 ]; then
                            echo "✓ Backup copied to ${BACKUP_SERVER}"
                        else
                            echo "✗ Failed to copy backup"
                            exit 1
                        fi

                        rm -f ${backupFile}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✓ Database backup completed successfully at \${new Date()}"
        }
        failure {
            echo "✗ Database backup failed"
            // Add email notification here
        }
    }
}
```

---

## Key Takeaways

1. ✅ Jenkins automates database backup processes effectively
2. ✅ `mysqldump` creates consistent database backups
3. ✅ Cron scheduling ensures regular automated backups
4. ✅ `sshpass` and `scp` enable secure remote file transfer
5. ✅ Date formatting in filenames provides backup traceability
6. ✅ Error handling ensures backup reliability
7. ✅ Compression and rotation optimize storage usage
8. ✅ Regular testing validates backup integrity

---

## Validation Checklist

- ✅ Job `database-backup` created successfully
- ✅ Build trigger configured for `*/10 * * * *` (every 10 minutes)
- ✅ Database dump created with correct filename format `db_YYYY-MM-DD.sql`
- ✅ Backup copied to `/home/clint/db_backups` on Backup Server
- ✅ Job executes automatically on schedule
- ✅ Console output shows successful execution
- ✅ Backup files verified on backup server
- ✅ Multiple scheduled executions complete successfully

---

## Additional Resources

- [MySQL mysqldump Documentation](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html)
- [Jenkins Cron Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/#cron-syntax)
- [SSH and SCP Best Practices](https://www.ssh.com/academy/ssh/copy-id)
- [Database Backup Best Practices](https://dev.mysql.com/doc/refman/8.0/en/backup-and-recovery.html)
- [Jenkins Pipeline Examples](https://www.jenkins.io/doc/pipeline/examples/)

---

**Task Completed Successfully! ✓**

The Jenkins database backup job is now configured to automatically backup the `kodekloud_db01` database every 10 minutes, copy the backup to the Backup Server, and maintain a reliable automated backup system for the Stratos Datacenter infrastructure. This ensures data protection and quick recovery capabilities in case of data loss or system failures.
