# Day 41: Write a Docker File

## Task

Create a Dockerfile on App Server 2 at `/opt/docker/Dockerfile` (capital D) that builds an image meeting these requirements:

a. Use `ubuntu:24.04` as the base image.
b. Install `apache2` and configure it to work on port `5001` (do not change other Apache settings like DocumentRoot).

---

## Solution (commands to run on App Server 2)

1. Create the target directory and the Dockerfile with the exact filename `/opt/docker/Dockerfile`:

```bash
sudo mkdir -p /opt/docker
sudo tee /opt/docker/Dockerfile > /dev/null <<'EOF'
FROM ubuntu:24.04
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y apache2 && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Configure Apache to listen on 5001 (keep other settings intact)
RUN sed -i 's/Listen 80/Listen 5001/' /etc/apache2/ports.conf && \
    sed -i 's/<VirtualHost \*:80>/<VirtualHost *:5001>/' /etc/apache2/sites-available/000-default.conf

EXPOSE 5001

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
EOF
```

Notes:

- The `tee` + heredoc approach writes the file atomically with proper permissions when run with `sudo`.
- We cleaned apt lists to reduce image size.

2. Build the Docker image (example tag `news:apache5001`):

```bash
sudo docker build -t news:apache5001 /opt/docker
```

3. Run the container exposing port 5001 (map host port as needed):

```bash
sudo docker run -d --name news_apache -p 5001:5001 news:apache5001
```

4. Verify Apache inside container is listening on 5001 from host:

```bash
# list containers
sudo docker ps

# check host port mapping
sudo ss -tuln | grep 5001
# or curl localhost:5001 on the host
curl -I http://localhost:5001
```

5. (Optional) Inspect built image and confirm Apache config changes:

```bash
sudo docker run --rm news:apache5001 cat /etc/apache2/ports.conf
sudo docker run --rm news:apache5001 cat /etc/apache2/sites-available/000-default.conf | sed -n '1,80p'
```

---

## Dockerfile (for reference)

FROM ubuntu:24.04
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
 apt-get install -y apache2 && \
 apt-get clean && rm -rf /var/lib/apt/lists/\*

RUN sed -i 's/Listen 80/Listen 5001/' /etc/apache2/ports.conf && \
 sed -i 's/<VirtualHost \*:80>/<VirtualHost \*:5001>/' /etc/apache2/sites-available/000-default.conf

EXPOSE 5001

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]

---

## Notes & Best Practices

- Prefer using a reproducible Dockerfile rather than committing container state.
- Keep image size small by cleaning apt caches and minimizing layers.
- For production, consider adding healthchecks and a non-root user.

Finally, update the main `README.md` to include Day 41 in the Containers & Containerization section.
