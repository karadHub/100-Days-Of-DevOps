# Day 21: Set Up Git Repository on Storage Server

## 🎯 TASK

The Nautilus development team asked the DevOps team to create a Git repository on the Storage Server in Stratos DC.

Requirements:

- Install `git` using `yum` on the Storage Server.
- Create a **bare** repository at the exact path `/opt/blog.git`.

> A bare repository is used as a central remote repo and does not have a working tree.

---

## 🛠 Solution (step-by-step)

1. Install git using `yum`:

```bash
sudo yum install -y git
```

2. Create the bare repository at the required path `/opt/blog.git`:

```bash
sudo mkdir -p /opt # /opt doesn't exist
sudo git init --bare /opt/blog.git
```

3. Set permissions (optional):

If multiple users or a dedicated `git` user will access the repo, set ownership and restrictive permissions:

```bash
sudo chown -R git:git /opt/blog.git || true
sudo chmod -R 770 /opt/blog.git || true
```

4. Verify repository:

```bash
ls -ld /opt/blog.git
sudo git --git-dir=/opt/blog.git rev-parse --is-bare-repository
```

The second command should output `true`.

---

## ✅ Verification (from developer machine)

Developers can add this remote and push:

```bash
git remote add origin ssh://git@storage_server:/opt/blog.git
git push origin --all
```

Replace `git@storage_server` with the correct user and host.

---

## 📚 Notes & Best Practices

- Use SSH keys and a dedicated `git` user for access control.
- Consider initializing a `deploy` or `protected` branch policy with server-side hooks for CI/CD.
- Regularly back up the bare repository (file-system level snapshots or `git bundle`).
