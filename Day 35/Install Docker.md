# Day 35: Install Docker Packages and Start Docker Service

## 🎯 Task

The Nautilus DevOps team plans to containerize applications on **App Server 1 (stapp01)**. Install Docker, Docker Compose, and ensure the Docker service is running and enabled.

---

## 🛠️ Steps

1. **SSH into App Server 1**

```bash
ssh tony@stapp01
```

2. **Enable Docker repository (CentOS/RHEL)**

```bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

3. **Install Docker Engine and components**

```bash
sudo dnf install -y \
  docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

4. **Start and enable Docker**

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

5. **Verify installation**

```bash
docker --version
docker compose version
sudo systemctl status docker --no-pager
```

---

## ✅ Notes

- Requires sudo privileges for `dnf` and `systemctl`.
- Use `sudo usermod -aG docker <user>` if running Docker without sudo is desired.
- For offline nodes, mirror Docker RPMs from the official repo.
