# Day 39: Create a Docker Image From Container

## Task

A Nautilus developer made changes inside a running container named `ubuntu_latest` on Application Server 2. The DevOps team needs to create a new image from that container so the changes are preserved.

Requirements:

- Create a new Docker image named `news:devops` from the running container `ubuntu_latest` on Application Server 2.
- Ensure the image is created and verified.

---

## Solution

### Summary

On Application Server 2, commit the running container `ubuntu_latest` into a new image named `news:devops` using `docker commit`, then verify the image exists locally.

### Step-by-step

1. SSH into Application Server 2 (if not already):

```bash
ssh <user>@app-server-2
```

2. Confirm the container `ubuntu_latest` is running:

```bash
docker ps --filter "name=ubuntu_latest"
```

You should see the container in the output. If not running, start it or confirm the correct container name.

3. Commit the running container to a new image:

```bash
docker commit ubuntu_latest news:devops
```

This creates a new image `news:devops` from the current state of the container.

4. Verify the image was created:

```bash
docker images | grep news
```

You should see an entry similar to:

```
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
news         devops    abc123def456   1 second ago    120MB
```

5. (Optional) Inspect the image details:

```bash
docker inspect news:devops
```

6. (Optional) Tag & push to a registry if required for sharing:

```bash
docker tag news:devops myregistry.example.com/myrepo/news:devops
docker push myregistry.example.com/myrepo/news:devops
```

---

## Notes & Best Practices

- `docker commit` captures the container filesystem as an image; however, it's best to create reproducible Dockerfiles for long-term maintenance.
- Use descriptive tags and keep an audit trail (e.g., `news:devops-2025-10-01`) for traceability.
- If the container had running processes or unsaved changes, ensure they are in the desired state before committing.

---
