# Day 59 — Troubleshoot Deployment Issues in Kubernetes

The Nautilus DevOps team’s `redis-deployment` stopped working after some changes. Pods are not running. This guide shows a fast, structured way to diagnose and fix common image-related failures like `ErrImagePull` and `ImagePullBackOff`.

---

## Quick Diagnosis

1. Get pods and see current status

```bash
kubectl get pods -l app=redis -o wide
```

2. Describe the failing pod to see Events (root cause is usually here)

```bash
kubectl describe pod <POD_NAME>
```

Look for messages like:

- `ErrImagePull` / `ImagePullBackOff`
- `Failed to pull image` with an invalid tag or repository

3. Inspect the Deployment spec

```bash
kubectl get deploy redis-deployment -o yaml | more
```

Check `.spec.template.spec.containers[0].image` for a bad image or tag.

---

## Common Fix (Wrong Image/Tag)

If the image/tag was mistyped, update it to a known-good version and watch the rollout.

Option A — One-liner update:

```bash
kubectl set image deploy/redis-deployment \
	redis=redis:6.2 \
	--record

kubectl rollout status deploy/redis-deployment
```

Option B — Edit the Deployment (interactive):

```bash
kubectl edit deploy redis-deployment
```

Then set the container image to a valid value, for example:

```yaml
spec:
	template:
		spec:
			containers:
				- name: redis
					image: redis:6.2
```

Verify new pods:

```bash
kubectl get pods -l app=redis
```

Expected: `STATUS` becomes `Running` and `READY` shows `1/1`.

---

## Validate and Inspect

- Check rollout history and current image:
  ```bash
  kubectl rollout history deploy/redis-deployment
  kubectl get deploy redis-deployment -o=jsonpath='{.spec.template.spec.containers[0].image}\n'
  ```
- Inspect pod details and recent events again if needed:
  ```bash
  kubectl describe deploy redis-deployment
  kubectl describe pod <NEW_POD_NAME>
  ```

---

## Example Session (what you might see)

```
NAME                                READY   STATUS              RESTARTS   AGE
redis-deployment-5bcd4c7d64-zc4pt   0/1     ErrImagePull        0          14s
redis-deployment-6fd9d5fcb-dglc5    0/1     ContainerCreating   0          100s

# Fix by correcting the image/tag
kubectl set image deploy/redis-deployment redis=redis:6.2 --record
deployment.apps/redis-deployment image updated

kubectl get pods
NAME                                READY   STATUS              RESTARTS   AGE
redis-deployment-5bcd4c7d64-zc4pt   0/1     ImagePullBackOff    0          3m
redis-deployment-7c8d4f6ddf-rfzpv   0/1     ContainerCreating   0          4s

# After the new pod pulls successfully
kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
redis-deployment-7c8d4f6ddf-rfzpv   1/1     Running   0          13s
```

---

## Other Causes to Consider

- Registry auth issues (private images): create and reference an `imagePullSecrets` in the pod spec.
- Network/DNS issues pulling images: check node-level connectivity and CoreDNS.
- Pod spec typos (container name mismatch when using `kubectl set image`). Ensure the container name in the command matches the Deployment spec.

---

## Helpful Commands Cheat Sheet

```bash
# Pods and their images
kubectl get pods -o wide

# Describe resources to see Events
kubectl describe pod <POD_NAME>
kubectl describe deploy redis-deployment

# Patch image quickly
kubectl set image deploy/redis-deployment redis=redis:6.2 --record

# Watch rollout
kubectl rollout status deploy/redis-deployment
kubectl rollout history deploy/redis-deployment

# If needed, undo to a previous revision
kubectl rollout undo deploy/redis-deployment
```

---

With the image corrected and rollout completed, the `redis-deployment` should return to a healthy, Running state.
