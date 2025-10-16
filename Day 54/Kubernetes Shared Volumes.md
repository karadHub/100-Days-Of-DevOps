# Day 54: Kubernetes Shared Volumes

## 🛠️ Task

We are working on an application that will be deployed on multiple containers within a pod on Kubernetes cluster. There is a requirement to share a volume among the containers to save some temporary data. The Nautilus DevOps team is developing a similar template to replicate the scenario.

### Requirements:

- Create a pod named `volume-share-nautilus`
- **Container 1**:
  - Name: `volume-container-nautilus-1`
  - Image: `fedora:latest`
  - Run a sleep command to keep it running
  - Mount `volume-share` at `/tmp/beta`
- **Container 2**:
  - Name: `volume-container-nautilus-2`
  - Image: `fedora:latest`
  - Run a sleep command to keep it running
  - Mount `volume-share` at `/tmp/apps`
- **Volume**: `volume-share` of type `emptyDir`
- **Test**: Create `beta.txt` in container 1's mount path and verify it's accessible in container 2

---

## ✅ Solution

### 📄 `volume-share-nautilus.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-share-nautilus
spec:
  containers:
    - name: volume-container-nautilus-1
      image: fedora:latest
      command: ["sleep", "3600"]
      volumeMounts:
        - name: volume-share
          mountPath: /tmp/beta
    - name: volume-container-nautilus-2
      image: fedora:latest
      command: ["sleep", "3600"]
      volumeMounts:
        - name: volume-share
          mountPath: /tmp/apps
  volumes:
    - name: volume-share
      emptyDir: {}
```

---

## 🚀 Deployment & Testing Steps

### 1) Apply the Pod

```bash
kubectl apply -f volume-share-nautilus.yaml
```

### 2) Verify Pod is Running

```bash
kubectl get pod volume-share-nautilus
kubectl describe pod volume-share-nautilus
```

### 3) Exec into the First Container

```bash
kubectl exec -it volume-share-nautilus -c volume-container-nautilus-1 -- /bin/bash
```

### 4) Create the Test File

Inside the first container:

```bash
echo "Shared volume test from container 1" > /tmp/beta/beta.txt
cat /tmp/beta/beta.txt
exit
```

### 5) Verify from the Second Container

```bash
kubectl exec -it volume-share-nautilus -c volume-container-nautilus-2 -- /bin/bash
```

Inside the second container:

```bash
cat /tmp/apps/beta.txt
# You should see: "Shared volume test from container 1"
ls -la /tmp/apps/
exit
```

### 6) Quick One-Liner Verification

```bash
# Create file in container 1
kubectl exec volume-share-nautilus -c volume-container-nautilus-1 -- sh -c 'echo "Test" > /tmp/beta/beta.txt'

# Read from container 2
kubectl exec volume-share-nautilus -c volume-container-nautilus-2 -- cat /tmp/apps/beta.txt
```

---

## 🧠 Understanding emptyDir Volumes

| Aspect               | Description                                                         |
| -------------------- | ------------------------------------------------------------------- |
| **Type**             | Temporary volume that exists as long as the pod exists              |
| **Lifecycle**        | Created when pod is assigned to a node, deleted when pod is removed |
| **Storage Location** | Node's local storage (disk or RAM if `medium: Memory`)              |
| **Use Cases**        | Sharing data between containers, temporary cache, scratch space     |
| **Data Persistence** | NOT persistent - data lost when pod terminates                      |

---

## 🔍 Key Concepts

### Why emptyDir?

- Perfect for inter-container communication within a pod
- No external storage dependency
- Fast and simple setup
- Automatically cleaned up

### Volume Mount Paths

- Container 1 sees the volume at `/tmp/beta`
- Container 2 sees the same volume at `/tmp/apps`
- Both paths point to the SAME underlying storage
- Files created in one container are immediately visible in the other

### Alternative: emptyDir with Memory

For faster I/O (uses tmpfs):

```yaml
volumes:
  - name: volume-share
    emptyDir:
      medium: Memory
      sizeLimit: 128Mi
```

---

## 🔧 Additional Commands

```bash
# Check volume mounts in pod spec
kubectl get pod volume-share-nautilus -o jsonpath='{.spec.containers[*].volumeMounts}'

# View all volumes defined in the pod
kubectl get pod volume-share-nautilus -o jsonpath='{.spec.volumes}'

# Check disk usage in container 1
kubectl exec volume-share-nautilus -c volume-container-nautilus-1 -- df -h /tmp/beta

# Delete the pod
kubectl delete pod volume-share-nautilus
```
