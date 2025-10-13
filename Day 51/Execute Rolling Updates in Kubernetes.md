# Day 51: Execute Rolling Updates in Kubernetes

## 🛠️ Task

An application currently running on the Kubernetes cluster employs the nginx web server. The Nautilus application development team has introduced some recent changes that need deployment. They've crafted an image `nginx:1.17` with the latest updates. Execute a rolling update for this application, integrating the `nginx:1.17` image. The deployment is named `nginx-deployment`. Ensure all pods are operational post-update.

---

## ✅ Solution Steps

### 1) Update the deployment image

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.17
```

### 2) Monitor rollout status

```bash
kubectl rollout status deployment/nginx-deployment
```

### 3) Verify pods are running

- If your deployment uses a label like `app=nginx-app`:

```bash
kubectl get pods -l app=nginx-app -o wide
```

- Otherwise, list pods for the deployment's ReplicaSet:

```bash
kubectl get rs -l app=nginx-app
kubectl get pods -l app=nginx-app
```

---

## 🔁 Helpful Rollout Commands

```bash
# See the current image used by containers in the deployment
kubectl get deployment nginx-deployment -o jsonpath='{.spec.template.spec.containers[*].image}'

# View rollout history
kubectl rollout history deployment/nginx-deployment

# Undo to previous revision (if needed)
kubectl rollout undo deployment/nginx-deployment
```

---

## 🧠 Notes

- Rolling updates are the default strategy for Deployments and ensure zero-downtime updates.
- Ensure the container name `nginx` in `kubectl set image` matches the name in your deployment spec.
- If your labels differ (e.g., `app=nginx`), adjust the `-l` selector accordingly when verifying pods.
