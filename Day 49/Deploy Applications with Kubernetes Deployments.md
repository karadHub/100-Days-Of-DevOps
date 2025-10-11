# Day 49: Deploy Applications with Kubernetes Deployments

## 🛠️ Task

Create a deployment named `nginx` to deploy the application nginx using the image `nginx:latest` (ensure to specify the tag).

---

## ✅ Solution

### 🚀 Method 1: Using kubectl command line

```bash
kubectl create deployment nginx --image=nginx:latest
```

### 📝 Method 2: Using YAML manifest

#### 1. Create the deployment YAML file

```bash
vi nginx-deployment.yaml
```

#### 2. Deployment Configuration (`nginx-deployment.yaml`)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```

#### 3. Apply the deployment

```bash
kubectl apply -f nginx-deployment.yaml
```

---

## ✅ Verification Commands

```bash
# Check deployment status
kubectl get deployments

# Check pods created by deployment
kubectl get pods

# Get detailed deployment information
kubectl describe deployment nginx

# Check deployment rollout status
kubectl rollout status deployment/nginx
```

**Expected Output:**

```
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           30s
```

---

## 🔍 Key Differences: Pod vs Deployment

| Aspect           | Pod                | Deployment                    |
| ---------------- | ------------------ | ----------------------------- |
| **Purpose**      | Single instance    | Manages multiple pod replicas |
| **Self-healing** | No                 | Yes (recreates failed pods)   |
| **Scaling**      | Manual             | Automatic/declarative         |
| **Updates**      | Manual replacement | Rolling updates               |
| **Use Case**     | Testing/debugging  | Production applications       |

---

## 🎯 Kubernetes Deployment Concepts

- **Deployment**: Manages a set of identical pods
- **ReplicaSet**: Ensures desired number of pod replicas
- **Rolling Updates**: Gradual replacement of old pods with new ones
- **Selector**: Labels used to identify which pods belong to the deployment

---

## 🔧 Additional Deployment Commands

```bash
# Scale the deployment
kubectl scale deployment nginx --replicas=3

# Update the image
kubectl set image deployment/nginx nginx=nginx:1.21

# View deployment history
kubectl rollout history deployment/nginx

# Rollback to previous version
kubectl rollout undo deployment/nginx

# Delete the deployment
kubectl delete deployment nginx
```

---

## 📊 YAML Breakdown

| Field                 | Purpose                              |
| --------------------- | ------------------------------------ |
| `apiVersion: apps/v1` | API version for Deployment resource  |
| `kind: Deployment`    | Resource type                        |
| `metadata.name`       | Name of the deployment               |
| `spec.replicas`       | Number of pod instances              |
| `spec.selector`       | How deployment finds its pods        |
| `spec.template`       | Pod template used to create new pods |
