# Day 56 — Deploy Nginx Web Server on Kubernetes Cluster

Goal: Deploy an Nginx web server on Kubernetes with 3 replicas and expose it via a NodePort service on port 30011. Provide both imperative (kubectl) and declarative (YAML) solutions, plus verification and cleanup steps.

---

## Prerequisites

- A running Kubernetes cluster and kubectl configured to talk to it
- NodePort range open (default: 30000–32767)
- Optional: curl installed locally to test via NodeIP:NodePort

---

## Option A — Imperative (kubectl commands)

1. Create a Deployment with 3 replicas

```bash
kubectl create deployment nginx-deploy \
  --image=nginx:stable \
  --dry-run=client -o yaml > deploy.yaml

# Update the container name and replicas in the generated YAML
# (container name should be nginx-container and replicas: 3)
sed -i.bak "s/name: nginx/name: nginx-container/" deploy.yaml
sed -i.bak "s/replicas: 1/replicas: 3/" deploy.yaml

kubectl apply -f deploy.yaml
```

Windows (PowerShell) alternative to edit in-place:

- Open `deploy.yaml` in an editor and adjust:
  - metadata.name: nginx-deploy
  - spec.replicas: 3
  - spec.template.spec.containers[0].name: nginx-container

2. Expose the Deployment via NodePort 30011

```bash
# Create a Service (NodePort)
kubectl expose deployment nginx-deploy \
  --type=NodePort \
  --name=nginx-svc \
  --port=80 \
  --target-port=80

# Patch the service to use a fixed nodePort 30011
kubectl patch svc nginx-svc \
  -p '{"spec":{"ports":[{"port":80,"targetPort":80,"protocol":"TCP","nodePort":30011}],"type":"NodePort"}}'
```

3. Verify

```bash
kubectl get deploy/nginx-deploy
kubectl get pods -l app=nginx-deploy -o wide
kubectl get svc/nginx-svc -o wide
```

4. Test from your machine

- Find a worker node IP
  ```bash
  kubectl get nodes -o wide
  ```
- Then curl the NodeIP:30011
  ```bash
  curl http://<NODE_IP>:30011/
  ```
  You should see the Nginx default welcome page HTML.

---

## Option B — Declarative (YAML manifests)

Create a single file (e.g., `nginx-k8s.yaml`) with both Deployment and Service:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx-container
          image: nginx:stable
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 30011
```

Apply and verify:

```bash
kubectl apply -f nginx-k8s.yaml
kubectl get deploy nginx-deploy
kubectl get pods -l app=nginx -o wide
kubectl get svc nginx-svc -o wide
```

Test via NodeIP:30011 as described above. If you use Minikube, you can also do:

```bash
minikube service nginx-svc --url
```

---

## Notes and Troubleshooting

- NodePort range must allow 30011 (default is 30000–32767). If your cluster disallows fixed NodePorts, omit nodePort to let Kubernetes assign one automatically.
- If using kind or Docker Desktop Kubernetes, ensure NodePort traffic is reachable from your host. For kind on Windows/Mac, consider `kubectl port-forward svc/nginx-svc 8080:80` and then browse http://localhost:8080.
- To change image version later:
  ```bash
  kubectl set image deploy/nginx-deploy nginx-container=nginx:1.27
  kubectl rollout status deploy/nginx-deploy
  ```

---

## Cleanup

```bash
kubectl delete svc nginx-svc
kubectl delete deploy nginx-deploy
# Or if you used the single YAML file
kubectl delete -f nginx-k8s.yaml
```
