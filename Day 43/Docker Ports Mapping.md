# Day 43: Docker Ports Mapping

## Task

a. Pull `nginx:alpine-perl` docker image on Application Server 1.
b. Create a container named `games` using the image you pulled.
c. Map host port `8089` to container port `80`. Please keep the container in running state.

---

## Solution

### Step 1: Pull the nginx:alpine-perl Docker image

```bash
docker pull nginx:alpine-perl
```

This command downloads the Nginx image based on Alpine Linux with Perl support. Alpine-based images are significantly smaller than regular images, which makes them ideal for container deployments where size matters.

### Step 2: Create and run the container with port mapping

```bash
docker run -d --name games -p 8089:80 nginx:alpine-perl
```

**Command options explained:**

- `-d`: Detached mode (runs container in the background)
- `--name games`: Assigns the name "games" to the container
- `-p 8089:80`: Maps host port 8089 to container port 80
- `nginx:alpine-perl`: The image to use for the container

### Step 3: Verify the container is running

```bash
docker ps
```

This command should show your container running with the name "games" and port mapping visible in the output:

```
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS                  NAMES
a1b2c3d4e5f6   nginx:alpine-perl "/docker-entrypoint.…"   1 minute ago   Up 1 minute   0.0.0.0:8089->80/tcp   games
```

### Step 4: Test the web server (optional)

You can verify the server is accessible via port 8089:

```bash
curl http://localhost:8089
```

You should see the default Nginx welcome page HTML.

---

## Understanding Port Mapping

Docker port mapping allows applications inside containers to be accessible via the host's IP address and specified ports. The syntax is:

```
-p [HOST_PORT]:[CONTAINER_PORT]
```

In this task:

- **Host port**: 8089 (externally accessible)
- **Container port**: 80 (internal Nginx port)

This means when you access `http://[server-ip]:8089`, the request is forwarded to port 80 inside the container where Nginx is listening.

---

## Additional Commands

### Stopping the container:

```bash
docker stop games
```

### Starting the container:

```bash
docker start games
```

### Removing the container:

```bash
docker stop games
docker rm games
```

### Viewing container logs:

```bash
docker logs games
```

### Accessing the container shell:

```bash
docker exec -it games /bin/sh
```

---

## Key Takeaways

1. Port mapping (`-p`) is essential for making containerized web services accessible from outside.
2. Alpine-based images provide the same functionality in a much smaller package.
3. Container naming makes management easier than working with automatically generated IDs.
4. Always verify container status after creation to ensure proper setup.
