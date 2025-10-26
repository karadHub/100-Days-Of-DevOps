# Day 61 — Init Containers in Kubernetes

Goal: Use an init container to perform setup tasks before the main application container starts. This pattern is common for configuration, seeding data, or waiting for dependencies.

---

## Scenario

The xFusionCorp DevOps team needs to deploy applications with prerequisites that can't be baked into the main container image. They'll use an **init container** to write a configuration message to a shared volume, then the main container reads and displays it continuously.

---

## Task Requirements

Create a Deployment named `ic-deploy-xfusion` with:

- **Replicas**: 1
- **Labels**: `app: ic-xfusion` (both selector and template labels)
- **Init Container** (`ic-msg-xfusion`):
  - Image: `ubuntu:latest`
  - Command: `['/bin/bash', '-c', 'echo Init Done - Welcome to xFusionCorp Industries > /ic/beta']`
  - Volume mount: `ic-volume-xfusion` at `/ic`
- **Main Container** (`ic-main-xfusion`):
  - Image: `ubuntu:latest`
  - Command: `['/bin/bash', '-c', 'while true; do cat /ic/beta; sleep 5; done']`
  - Volume mount: `ic-volume-xfusion` at `/ic`
- **Volume**: `ic-volume-xfusion` (emptyDir)

---

## Solution

### Deployment YAML

File: `ic-deploy-xfusion.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ic-deploy-xfusion
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ic-xfusion
  template:
    metadata:
      labels:
        app: ic-xfusion
    spec:
      volumes:
        - name: ic-volume-xfusion
          emptyDir: {}
      initContainers:
        - name: ic-msg-xfusion
          image: ubuntu:latest
          command:
            [
              "/bin/bash",
              "-c",
              "echo Init Done - Welcome to xFusionCorp Industries > /ic/beta",
            ]
          volumeMounts:
            - name: ic-volume-xfusion
              mountPath: /ic
      containers:
        - name: ic-main-xfusion
          image: ubuntu:latest
          command:
            ["/bin/bash", "-c", "while true; do cat /ic/beta; sleep 5; done"]
          volumeMounts:
            - name: ic-volume-xfusion
              mountPath: /ic
```

---

## Apply and Verify

### 1. Apply the Deployment

```bash
kubectl apply -f ic-deploy-xfusion.yaml
```

### 2. Check Pod Status

```bash
kubectl get pods -l app=ic-xfusion
```

Expected output:

```
NAME                                  READY   STATUS    RESTARTS   AGE
ic-deploy-xfusion-xxxxxxxxxx-xxxxx   1/1     Running   0          30s
```

> If stuck in `Init:0/1` or `Init:Error`, the init container failed—check logs in step 3.

### 3. Inspect Init Container Logs

```bash
kubectl logs <pod-name> -c ic-msg-xfusion
```

Expected: (empty or a simple echo output; the message goes to the file, not stdout)

### 4. Verify File Creation

Check that the init container wrote the message to `/ic/beta`:

```bash
kubectl exec -it <pod-name> -c ic-main-xfusion -- cat /ic/beta
```

Expected output:

```
Init Done - Welcome to xFusionCorp Industries
```

### 5. Watch Live Output

Since the main container loops and prints the file every 5 seconds:

```bash
kubectl logs <pod-name> -c ic-main-xfusion -f
```

You'll see the message repeated every 5 seconds:

```
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
Init Done - Welcome to xFusionCorp Industries
...
```

---

## Understanding Init Containers

### What are Init Containers?

Init containers run **before** the main application containers start. They must complete successfully before the main container begins.

### Common Use Cases

- **Wait for dependencies**: Wait for a database or service to be ready
- **Pre-populate data**: Seed configuration files, clone Git repos, download assets
- **Setup tasks**: Change file permissions, run migrations, register with service discovery
- **Security**: Fetch secrets or certificates from external systems

### Key Characteristics

- Run sequentially (one after another if multiple init containers exist)
- Must complete successfully (exit 0) for the pod to proceed
- Share volumes with main containers
- Do **not** support readiness/liveness probes
- Restart if they fail (subject to pod's `restartPolicy`)

### Init Containers vs Sidecar Containers

| Feature    | Init Containers                | Sidecar Containers           |
| ---------- | ------------------------------ | ---------------------------- |
| Lifecycle  | Run once before main container | Run alongside main container |
| Purpose    | Setup/prerequisites            | Ongoing auxiliary tasks      |
| Completion | Must complete successfully     | Run continuously             |
| Example    | Database migration             | Log shipping, proxies        |

---

## Troubleshooting

### Pod Stuck in `Init:0/1` Status

- Check init container logs:
  ```bash
  kubectl logs <pod-name> -c ic-msg-xfusion
  kubectl describe pod <pod-name>
  ```
- Look for errors in the Events section
- Common issues: image pull errors, command failures, permission issues

### Init Container Completes but Main Container Fails

- Check main container logs:
  ```bash
  kubectl logs <pod-name> -c ic-main-xfusion
  ```
- Verify volume mounts are correct
- Ensure the file written by init container exists

### File Not Found in Main Container

- Verify both containers mount the **same volume** with the **same name**
- Check mount paths match what your commands expect
- Use `kubectl exec` to inspect the filesystem:
  ```bash
  kubectl exec -it <pod-name> -c ic-main-xfusion -- ls -la /ic
  ```

---

## Advanced Examples

### Multiple Init Containers (Sequential)

```yaml
initContainers:
  - name: wait-for-db
    image: busybox:latest
    command:
      ["sh", "-c", "until nslookup mydb; do echo waiting for db; sleep 2; done"]
  - name: clone-repo
    image: alpine/git:latest
    command: ["git", "clone", "https://github.com/example/repo.git", "/data"]
    volumeMounts:
      - name: app-data
        mountPath: /data
```

### Init Container with Environment Variables

```yaml
initContainers:
  - name: config-setup
    image: busybox:latest
    env:
      - name: CONFIG_URL
        value: "https://config.example.com/app.conf"
    command: ["sh", "-c", "wget -O /config/app.conf $CONFIG_URL"]
    volumeMounts:
      - name: config-volume
        mountPath: /config
```

---

## Cleanup

```bash
kubectl delete -f ic-deploy-xfusion.yaml
```

Or:

```bash
kubectl delete deploy ic-deploy-xfusion
```

---

## Key Takeaways

✅ Init containers run **before** main containers and must complete successfully
✅ Use them for setup tasks like config generation, waiting for dependencies, or data seeding
✅ Init containers share volumes with main containers via `volumeMounts`
✅ Multiple init containers run **sequentially**, not in parallel
✅ If an init container fails, Kubernetes restarts the entire pod (based on `restartPolicy`)
✅ Init containers are great for separating concerns: keep setup logic out of main app images
