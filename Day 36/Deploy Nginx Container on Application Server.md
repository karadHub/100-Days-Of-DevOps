# Day 36: Deploy Nginx Container on Application Server

## Task

The Nautilus DevOps team is conducting application deployment tests on selected application servers. They require a nginx container deployment on Application Server 3. Complete the task with the following instructions:

On Application Server 3 create a container named `nginx_3` using the `nginx` image with the `alpine` tag. Ensure container is in a running state.

## Solution

### Step 1: Connect to the Application Server

First, connect to `Application Server 3` via SSH. Make sure you have the necessary credentials and access.

### Step 2: Pull the Nginx Alpine Image

It's a good practice to pull the image first, although `docker run` will do it automatically if it's not present locally.

```bash
docker pull nginx:alpine
```

### Step 3: Run the Nginx Container

Create a container named `nginx_3` from the `nginx:alpine` image. The `-d` flag runs it in detached mode.

```bash
docker run -d --name nginx_3 nginx:alpine
```

### Step 4: Verify the Container is Running

You can check the status of running containers to ensure `nginx_3` is up.

```bash
docker ps
```

You should see `nginx_3` in the list of running containers. You can also check the logs.

```bash
docker logs nginx_3
```
