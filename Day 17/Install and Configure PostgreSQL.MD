# Day 17: Install and Configure PostgreSQL

## 🎯 TASK

The Nautilus application development team has shared that they are planning to deploy one newly developed application on Nautilus infra in Stratos DC. The application uses PostgreSQL database, so as a pre-requisite we need to set up PostgreSQL database server as per requirements shared below:

PostgreSQL database server is already installed on the Nautilus database server.

**Requirements:**

- a. Create a database user `kodekloud_aim` and set its password to `dCV3szSGNA`
- b. Create a database `kodekloud_db6` and grant full permissions to user `kodekloud_aim` on this database

> **Note:** Please do not try to restart PostgreSQL server service.

---

## 🛠️ SOLUTION

To fulfill the PostgreSQL setup requirements for the Nautilus application deployment:

### Step-by-Step PostgreSQL Setup

#### 1. **Log in to PostgreSQL**

```bash
sudo -u postgres psql
```

#### 2. **Create the Database User**

```sql
CREATE USER kodekloud_aim WITH PASSWORD 'dCV3szSGNA';
```

#### 3. **Create the Database**

```sql
CREATE DATABASE kodekloud_db6;
```

#### 4. **Grant Full Permissions to the User**

```sql
GRANT ALL PRIVILEGES ON DATABASE kodekloud_db6 TO kodekloud_aim;
```

#### 5. **Exit PostgreSQL**

```sql
\q
```

---

## ✅ Verification

### Verify User and Database Creation

```bash
# List all users
sudo -u postgres psql -c "\du"

# List all databases
sudo -u postgres psql -c "\l"
```

### Test User Access

```bash
# Test connection with the new user
psql -U kodekloud_aim -d kodekloud_db6 -h localhost -W
```

---

## 📚 Key Learning Points

### PostgreSQL User Management

- **CREATE USER**: Creates a new database user/role
- **WITH PASSWORD**: Sets the password for the user
- **GRANT ALL PRIVILEGES**: Gives comprehensive permissions

### Database Permissions

- `ALL PRIVILEGES` includes:
  - CREATE (create tables)
  - SELECT (read data)
  - INSERT (add data)
  - UPDATE (modify data)
  - DELETE (remove data)
  - And more administrative privileges

### Best Practices

- Always use strong passwords in production
- Consider using environment variables for sensitive data
- Regularly review and audit database permissions
- Use least privilege principle in production environments

---

## 🔧 Additional Commands

### Check Current Connections

```sql
SELECT * FROM pg_stat_activity WHERE datname = 'kodekloud_db6';
```

### View User Privileges

```sql
SELECT grantee, privilege_type
FROM information_schema.role_table_grants
WHERE table_schema = 'public';
```

### Change User Password (if needed)

```sql
ALTER USER kodekloud_aim WITH PASSWORD 'new_password';
```
