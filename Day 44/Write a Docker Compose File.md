# Day 44: Write a Docker Compose File

## Task

a. On App Server 3 in Stratos DC, create a container named `httpd` using a docker compose file `/opt/docker/docker-compose.yml` (use the exact name for the file).
b. Use `httpd` (preferably latest tag) image for the container and make sure the container is named as `httpd`; you can use any name for the service.
c. Map port 80 of the container with port 8085 of the docker host.
d. Map container's `/usr/local/apache2/htdocs` volume with `/opt/data` volume of the docker host (which already exists). Do not modify any data within these locations.

---

## Solution

### Step 1: Create the Directory Structure (if needed)

```bash
# Make sure the directory exists
sudo mkdir -p /opt/docker
```

### Step 2: Create the Docker Compose File

```bash
# Create docker-compose.yml file
sudo vi /opt/docker/docker-compose.yml
```

Add the following content to the file:

```yaml
version: "3"
services:
  webserver:
    image: httpd:latest
    container_name: httpd
    ports:
      - "8085:80"
    volumes:
      - /opt/data:/usr/local/apache2/htdocs
```

### Step 3: Deploy the Container Using Docker Compose

```bash
# Navigate to the directory containing docker-compose.yml
cd /opt/docker

# Start the container in detached mode
sudo docker-compose up -d
```

### Step 4: Verify the Deployment

```bash
# Check if the container is running
sudo docker ps

# Expected output should show the 'httpd' container running

# Verify port mapping
sudo ss -tuln | grep 8085

# Test if the web server is serving content
curl http://localhost:8085
```

---

## Understanding the Docker Compose File

Let's break down the Docker Compose configuration:

```yaml
version: "3" # Docker Compose file format version
services: # Define services (containers)
  webserver: # Service name - can be anything descriptive
    image: httpd:latest # Docker image to use - Apache HTTP Server (latest)
    container_name: httpd # Explicitly naming the container
    ports: # Port mapping configuration
      - "8085:80" # Maps host port 8085 to container port 80
    volumes: # Volume mounting configuration
      - /opt/data:/usr/local/apache2/htdocs # Maps host directory to container directory
```

### Key Components:

1. **Service Name** (`webserver`): An arbitrary name for the service. You can choose any name that makes sense for your application.

2. **Image** (`httpd:latest`): Specifies the Docker image to use. In this case, we're using the official Apache HTTP Server image with the "latest" tag.

3. **Container Name** (`httpd`): Explicitly sets the container name. Without this, Docker Compose would generate a name automatically.

4. **Port Mapping** (`"8085:80"`): Maps port 80 inside the container (where Apache listens by default) to port 8085 on the host machine.

5. **Volume Mounting** (`/opt/data:/usr/local/apache2/htdocs`): Mounts the `/opt/data` directory from the host into the `/usr/local/apache2/htdocs` directory inside the container, which is Apache's default document root.

---

## Common Docker Compose Commands

```bash
# Start containers
docker-compose up -d

# Stop containers
docker-compose down

# View logs
docker-compose logs

# Restart containers
docker-compose restart

# Check status
docker-compose ps
```

---

## Troubleshooting

If you encounter any issues:

1. **Permissions Issues**:

   ```bash
   # Check permissions on /opt/data
   ls -la /opt/data

   # Fix permissions if needed
   sudo chmod -R 755 /opt/data
   ```

2. **Port Already in Use**:

   ```bash
   # Check if port is already in use
   sudo ss -tuln | grep 8085

   # Find and stop the process using the port
   sudo fuser -k 8085/tcp
   ```

3. **Docker Compose Version Issues**:

   ```bash
   # Check Docker Compose version
   docker-compose --version

   # If version is too old, you may need to update the version in the yaml file
   ```

---

## Verification

After deployment, the Apache HTTP Server should be running in a container named `httpd`, accessible via port 8085 on the host, and serving content from the `/opt/data` directory.

To verify everything is working correctly:

```bash
# Check container status
sudo docker ps | grep httpd

# Access the website
curl -I http://localhost:8085
```

You should see an HTTP 200 OK response if everything is set up correctly.
