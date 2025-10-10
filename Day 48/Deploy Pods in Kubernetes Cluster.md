# Day 48: Deploy Pods in Kubernetes Cluster

## 🛠️ Task

The Nautilus DevOps team is diving into Kubernetes for application management. One team member has a task to create a pod according to the details below:

- Create a pod named `pod-httpd` using the `httpd` image with the `latest` tag. Ensure to specify the tag as `httpd:latest`.
- Set the `app` label to `httpd_app`, and name the container as `httpd-container`.

---

## ✅ Solution

### 📝 1. Create the Pod YAML file

```bash
vi pod.yaml
```

### 📄 2. Pod Configuration (`pod.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-httpd
  labels:
    app: httpd_app
spec:
  containers:
    - name: httpd-container
      image: httpd:latest
```

### 🚀 3. Deploy the Pod

```bash
kubectl apply -f pod.yaml
```

### ✅ 4. Verify the Deployment

```bash
kubectl get pod
```

**Expected Output:**

```
NAME        READY   STATUS    RESTARTS   AGE
pod-httpd   1/1     Running   0          8s
```

---

## 🔍 Command Breakdown

| Command                          | Purpose                                  |
| -------------------------------- | ---------------------------------------- |
| `kubectl apply -f pod.yaml`      | Creates the pod from the YAML file       |
| `kubectl get pod`                | Lists all pods and their status          |
| `kubectl describe pod pod-httpd` | Shows detailed information about the pod |

---

## 🎯 Key Kubernetes Concepts

- **Pod**: The smallest deployable unit in Kubernetes
- **Labels**: Key-value pairs for organizing and selecting resources
- **Container**: The runtime instance of an image
- **Image Tag**: Specifies the version of the container image to use

---

## 🔧 Additional Commands

```bash
# Get detailed pod information
kubectl describe pod pod-httpd

# View pod logs
kubectl logs pod-httpd

# Delete the pod
kubectl delete pod pod-httpd
```
