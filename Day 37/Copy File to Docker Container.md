# Day 37: Copy File to Docker Container

## Task

The Nautilus DevOps team possesses confidential data on App Server 3 in the Stratos Datacenter. A container named `ubuntu_latest` is running on the same server.

Copy an encrypted file `/tmp/nautilus.txt.gpg` from the docker host to the `ubuntu_latest` container located at `/usr/src/`. Ensure the file is not modified during this operation.

## Solution

This task is part of a hands-on DevOps simulation from KodeKloud Engineer, designed to test your ability to work with Docker containers and securely transfer files between host and container environments.

---

### 🧠 **Scenario Overview**

- You're working as part of the **Nautilus DevOps team**, which manages infrastructure in the **Stratos Datacenter**.
- **App Server 3** hosts a Docker container named `ubuntu_latest`.
- There's a **confidential encrypted file** located on the **Docker host** at `/tmp/nautilus.txt.gpg`.
- Your goal is to **copy this file into the running container** at the path `/usr/src/` without altering its contents.

---

### 🛠️ **Key Requirements**

- **Preserve file integrity**: The `.gpg` extension indicates it's encrypted with GnuPG, so any modification could corrupt it.
- **Use Docker commands**: You'll likely use `docker cp` to transfer the file.
- **Target container**: `ubuntu_latest` must already be running on App Server 3.

---

### ✅ **Step-by-Step Solution**

#### Step 1: Connect to App Server 3

First, SSH into App Server 3 if you're not already connected:

```bash
ssh username@app-server-3
```

#### Step 2: Verify the Container is Running

Check that the `ubuntu_latest` container is running:

```bash
docker ps
```

Look for the container named `ubuntu_latest` in the output. You should see something like:

```
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS   NAMES
abc123def456   ubuntu    "/bin/bash"   ...   Up ...    ubuntu_latest
```

#### Step 3: Verify the Source File Exists

Confirm that the encrypted file exists on the host:

```bash
ls -l /tmp/nautilus.txt.gpg
```

#### Step 4: Copy the File from Host to Container

Use the `docker cp` command to copy the file from the host to the container:

```bash
docker cp /tmp/nautilus.txt.gpg ubuntu_latest:/usr/src/
```

This command:

- `docker cp`: Docker command to copy files between host and container
- `/tmp/nautilus.txt.gpg`: Source file path on the host
- `ubuntu_latest:/usr/src/`: Destination path inside the container

#### Step 5: Verify the File was Copied Successfully

Check that the file exists inside the container:

```bash
docker exec ubuntu_latest ls -l /usr/src/
```

You should see the `nautilus.txt.gpg` file listed in the `/usr/src/` directory.

#### Optional: Verify File Integrity

To ensure the file wasn't modified during the transfer, you can compare checksums:

On the host:

```bash
md5sum /tmp/nautilus.txt.gpg
```

Inside the container:

```bash
docker exec ubuntu_latest md5sum /usr/src/nautilus.txt.gpg
```

The checksums should match, confirming the file integrity is preserved.

---

### 🔒 **Why It Matters**

This exercise simulates a real-world scenario where secure data must be handled carefully across environments. It tests your understanding of:

- **Docker container management**: Working with running containers
- **Secure file handling**: Preserving encrypted file integrity
- **Command-line precision**: Using the correct Docker commands for file operations
- **DevOps best practices**: Safely transferring sensitive data between environments

---

### 💡 **Key Takeaways**

1. **`docker cp` is the standard way** to copy files between host and container
2. **File integrity is crucial** when dealing with encrypted or sensitive data
3. **Always verify operations** by checking the destination after copying
4. **Container operations** can be performed without entering the container using `docker exec`

This task demonstrates essential skills for managing containerized applications and handling sensitive data in production environments.
