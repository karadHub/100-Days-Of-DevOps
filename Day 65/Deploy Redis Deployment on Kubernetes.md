# Day 65 — Deploy Redis Deployment on Kubernetes

Goal: Deploy a Redis instance for testing using a ConfigMap for configuration, a Deployment using `redis:alpine`, and appropriate volume mounts and resource requests.

---

## Requirements

- Create a ConfigMap `my-redis-config` containing a key `redis-config` with content `maxmemory 2mb`.
- Create a Deployment `redis-deployment`:
  - Image: `redis:alpine`
  - Container name: `redis-container`
  - Replicas: 1
  - Container requests: cpu: "1"
  - Expose port 6379
  - Mount volumes:
    - `data` (emptyDir) → `/redis-master-data`
    - `redis-config` (configMap) → `/redis-master` (use `subPath: redis-config` to mount single file)
- Ensure deployment is running.

---

## Manifests

Save the following as `redis-deployment.yaml` (it contains both the ConfigMap and the Deployment):

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-redis-config
data:
  redis-config: |
    maxmemory 2mb
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis-container
          image: redis:alpine
          ports:
            - containerPort: 6379
          resources:
            requests:
              cpu: "1"
          volumeMounts:
            - name: data
              mountPath: /redis-master-data
            - name: redis-config
              mountPath: /redis-master
              subPath: redis-config
      volumes:
        - name: data
          emptyDir: {}
        - name: redis-config
          configMap:
            name: my-redis-config
```

Notes:

- We mount the ConfigMap key `redis-config` at the path `/redis-master` (as a file) using `subPath`. If you want Redis to load configuration from a specific file, you can start Redis with that config path (see Optional section below).

---

## Apply and Verify

```bash
kubectl apply -f redis-deployment.yaml

# Check pods
kubectl get pods -l app=redis

# Describe pod for events and volume mounts
POD=$(kubectl get pods -l app=redis -o jsonpath='{.items[0].metadata.name}')
kubectl describe pod $POD

# Check container logs
kubectl logs $POD -c redis-container
```

You should see the Pod move to `Running` status. The container exposes port 6379. The config file from the ConfigMap will be present inside the pod at `/redis-master` (as a file named `redis-config`).

To inspect the config inside the pod:

```bash
kubectl exec -it $POD -- cat /redis-master
```

Expected output:

```
maxmemory 2mb
```

---

## Optional: Start Redis with Custom Config

By default the `redis:alpine` image may run `redis-server` with its default configuration. If you want Redis to load the mounted config file, modify the Deployment container command to explicitly start Redis with it, for example:

```yaml
command: ["redis-server", "/redis-master"]
```

Replace the container spec in the Deployment with the above `command` if needed.

---

## Troubleshooting

- Pod remains in `Pending`: check node resources and scheduling
- `ImagePullBackOff`: image name/tag typo or registry issues
- Config file not present:
  - Verify ConfigMap exists: `kubectl get configmap my-redis-config -o yaml`
  - Verify volume mount: `kubectl describe pod $POD`
- Redis not using config: ensure Redis is started with the config file path if needed

---

## Cleanup

```bash
kubectl delete -f redis-deployment.yaml
```

---

## Key Takeaways

- ConfigMaps are useful for injecting configuration files into pods
- `emptyDir` is ephemeral and useful for temporary storage during pod lifetime
- `subPath` mounts a specific key/file from a volume at a chosen mount path
- Ensure container requests/limits reflect node capacity

```

```
