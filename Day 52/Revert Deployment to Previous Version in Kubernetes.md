# Day 52: Revert Deployment to Previous Version in Kubernetes

## 🛠️ Task

Earlier today, the Nautilus DevOps team deployed a new release for an application. However, a customer reported a bug in this release. Revert the deployment named `nginx-deployment` to the previous revision and ensure all pods are healthy after the rollback.

---

## ✅ Solution

### 1) Check rollout history (optional but recommended)

```bash
kubectl rollout history deployment/nginx-deployment
```

### 2) Roll back to the previous revision

```bash
kubectl rollout undo deployment/nginx-deployment
```

If you need to roll back to a specific revision:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=<REVISION_NUMBER>
```

### 3) Monitor the rollout

```bash
kubectl rollout status deployment/nginx-deployment
```

### 4) Verify pods are running and image reverted

```bash
# Check the current image used by the deployment
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'

# List pods and their status
kubectl get pods -l app=nginx-app -o wide

# Describe deployment for detailed status and events
kubectl describe deployment nginx-deployment
```

Expected output after the undo command (example):

```
deployment.apps/nginx-deployment rolled back
```

---

## 🧠 Notes

- Ensure the container name you updated previously (e.g., `nginx`) matches the container in your deployment spec when inspecting images.
- If the deployment has only one revision recorded, `undo` may have no effect. In that case, update the image explicitly to the desired version.
- Use `kubectl rollout history` to see available revisions and confirm the target version before undoing.

---

## 🔧 Useful Commands Cheat Sheet

```bash
# Show current image
kubectl get deploy nginx-deployment -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'

# View rollout history
kubectl rollout history deploy/nginx-deployment

# Roll back to previous
kubectl rollout undo deploy/nginx-deployment

# Roll back to a specific revision
kubectl rollout undo deploy/nginx-deployment --to-revision=2

# Watch rollout status
kubectl rollout status deploy/nginx-deployment
```
