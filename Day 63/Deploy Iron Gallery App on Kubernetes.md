# Day 63 — Deploy Iron Gallery App on Kubernetes

Goal: Deploy a multi-tier application (Iron Gallery frontend + MariaDB backend) on Kubernetes with proper namespace isolation, resource limits, volume mounts, and service exposure.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│         iron-namespace-xfusion                      │
│                                                     │
│  ┌──────────────────┐       ┌──────────────────┐  │
│  │ Iron Gallery     │       │ MariaDB          │  │
│  │ (Frontend)       │──────▶│ (Database)       │  │
│  │ NodePort: 32678  │       │ ClusterIP: 3306  │  │
│  └──────────────────┘       └──────────────────┘  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## Task Requirements

### Namespace

- Name: `iron-namespace-xfusion`

### Iron Gallery Deployment

- Name: `iron-gallery-deployment-xfusion`
- Labels: `run: iron-gallery`
- Replicas: 1
- Container: `iron-gallery-container-xfusion`
- Image: `kodekloud/irongallery:2.0`
- Resource limits: 100Mi memory, 50m CPU
- Volume mounts: `config` → `/usr/share/nginx/html/data`, `images` → `/usr/share/nginx/html/uploads`
- Volumes: Both emptyDir

### Iron DB Deployment

- Name: `iron-db-deployment-xfusion`
- Labels: `db: mariadb`
- Replicas: 1
- Container: `iron-db-container-xfusion`
- Image: `kodekloud/irondb:2.0`
- Environment: `MYSQL_DATABASE`, `MYSQL_ROOT_PASSWORD`, `MYSQL_PASSWORD`, `MYSQL_USER`
- Volume mount: `db` → `/var/lib/mysql`
- Volume: emptyDir

### Services

- **Iron DB Service**: `iron-db-service-xfusion` (ClusterIP, port 3306)
- **Iron Gallery Service**: `iron-gallery-service-xfusion` (NodePort 32678, port 80)

---

## Solution

### Complete YAML Manifests

You can create a single file `iron-gallery-app.yaml` or split into separate files.

#### 1. Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: iron-namespace-xfusion
```

---

#### 2. Iron Gallery Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-gallery-deployment-xfusion
  namespace: iron-namespace-xfusion
  labels:
    run: iron-gallery
spec:
  replicas: 1
  selector:
    matchLabels:
      run: iron-gallery
  template:
    metadata:
      labels:
        run: iron-gallery
    spec:
      containers:
        - name: iron-gallery-container-xfusion
          image: kodekloud/irongallery:2.0
          resources:
            limits:
              memory: "100Mi"
              cpu: "50m"
          volumeMounts:
            - name: config
              mountPath: /usr/share/nginx/html/data
            - name: images
              mountPath: /usr/share/nginx/html/uploads
      volumes:
        - name: config
          emptyDir: {}
        - name: images
          emptyDir: {}
```

---

#### 3. Iron DB Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iron-db-deployment-xfusion
  namespace: iron-namespace-xfusion
  labels:
    db: mariadb
spec:
  replicas: 1
  selector:
    matchLabels:
      db: mariadb
  template:
    metadata:
      labels:
        db: mariadb
    spec:
      containers:
        - name: iron-db-container-xfusion
          image: kodekloud/irondb:2.0
          env:
            - name: MYSQL_DATABASE
              value: database_blog
            - name: MYSQL_ROOT_PASSWORD
              value: "R00tP@ssw0rd!"
            - name: MYSQL_PASSWORD
              value: "S3cur3P@ssw0rd!"
            - name: MYSQL_USER
              value: "ironuser"
          volumeMounts:
            - name: db
              mountPath: /var/lib/mysql
      volumes:
        - name: db
          emptyDir: {}
```

> **Security Note**: For production, use Kubernetes Secrets instead of hardcoded passwords! See Day 62 for secret management.

---

#### 4. Iron DB Service (ClusterIP)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: iron-db-service-xfusion
  namespace: iron-namespace-xfusion
spec:
  selector:
    db: mariadb
  type: ClusterIP
  ports:
    - protocol: TCP
      port: 3306
      targetPort: 3306
```

---

#### 5. Iron Gallery Service (NodePort)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: iron-gallery-service-xfusion
  namespace: iron-namespace-xfusion
spec:
  selector:
    run: iron-gallery
  type: NodePort
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 32678
```

---

## Deployment Steps

### Option A: Single Combined File

Create `iron-gallery-app.yaml` with all manifests separated by `---`, then:

```bash
kubectl apply -f iron-gallery-app.yaml
```

### Option B: Separate Files

```bash
kubectl apply -f namespace.yaml
kubectl apply -f iron-db-deployment.yaml
kubectl apply -f iron-db-service.yaml
kubectl apply -f iron-gallery-deployment.yaml
kubectl apply -f iron-gallery-service.yaml
```

### Option C: Apply All Files in a Directory

```bash
kubectl apply -f ./iron-gallery/
```

---

## Verification Steps

### 1. Check Namespace

```bash
kubectl get namespace iron-namespace-xfusion
```

### 2. Check Deployments

```bash
kubectl get deployments -n iron-namespace-xfusion
```

Expected output:

```
NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
iron-db-deployment-xfusion         1/1     1            1           2m
iron-gallery-deployment-xfusion    1/1     1            1           2m
```

### 3. Check Pods

```bash
kubectl get pods -n iron-namespace-xfusion -o wide
```

Wait for both pods to show `Running` status.

### 4. Check Services

```bash
kubectl get services -n iron-namespace-xfusion
```

Expected output:

```
NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
iron-db-service-xfusion        ClusterIP   10.96.x.x       <none>        3306/TCP       2m
iron-gallery-service-xfusion   NodePort    10.96.x.x       <none>        80:32678/TCP   2m
```

### 5. Describe Resources for Details

```bash
kubectl describe deployment iron-gallery-deployment-xfusion -n iron-namespace-xfusion
kubectl describe deployment iron-db-deployment-xfusion -n iron-namespace-xfusion
```

### 6. Check Resource Limits

```bash
kubectl get pod -n iron-namespace-xfusion -o yaml | grep -A 5 resources
```

---

## Testing the Application

### 1. Access Iron Gallery via NodePort

Get a node IP:

```bash
kubectl get nodes -o wide
```

Access in browser:

```
http://<NODE_IP>:32678
```

### 2. Verify Database Connectivity (from within cluster)

```bash
# Get the gallery pod name
GALLERY_POD=$(kubectl get pod -n iron-namespace-xfusion -l run=iron-gallery -o jsonpath='{.items[0].metadata.name}')

# Exec into the gallery pod and test DB connection
kubectl exec -it $GALLERY_POD -n iron-namespace-xfusion -- /bin/bash

# Inside the pod, test connectivity to the database service
# (if mysql client is available)
mysql -h iron-db-service-xfusion -u ironuser -p database_blog
```

### 3. Check Logs

**Gallery logs:**

```bash
kubectl logs -n iron-namespace-xfusion -l run=iron-gallery
```

**Database logs:**

```bash
kubectl logs -n iron-namespace-xfusion -l db=mariadb
```

### 4. Verify Volume Mounts

```bash
kubectl exec -n iron-namespace-xfusion -l run=iron-gallery -- ls -la /usr/share/nginx/html/data
kubectl exec -n iron-namespace-xfusion -l run=iron-gallery -- ls -la /usr/share/nginx/html/uploads
kubectl exec -n iron-namespace-xfusion -l db=mariadb -- ls -la /var/lib/mysql
```

---

## Understanding the Components

### Resource Limits

```yaml
resources:
  limits:
    memory: "100Mi"
    cpu: "50m"
```

- **Memory**: 100 MiB maximum
- **CPU**: 50 millicores (0.05 cores, or 5% of a single CPU core)

Benefits:

- Prevents resource starvation
- Helps scheduler make better placement decisions
- Protects cluster from runaway processes

### EmptyDir Volumes

```yaml
volumes:
  - name: config
    emptyDir: {}
```

Characteristics:

- Created when pod starts
- Deleted when pod is removed
- Shared between containers in same pod
- Not suitable for persistent data (use PersistentVolumes for that)

Good for:

- Temporary scratch space
- Caching
- Sharing data between containers in a pod

### Service Types

**ClusterIP** (Database):

- Only accessible within the cluster
- Perfect for backend services that don't need external access
- Default service type

**NodePort** (Frontend):

- Accessible from outside the cluster via `<NodeIP>:<NodePort>`
- Port range: 30000-32767 (default)
- Good for development/testing
- For production, use LoadBalancer or Ingress

---

## Production Improvements

### 1. Use Secrets for Passwords

Create a secret:

```bash
kubectl create secret generic iron-db-secret \
  -n iron-namespace-xfusion \
  --from-literal=root-password='R00tP@ssw0rd!' \
  --from-literal=user-password='S3cur3P@ssw0rd!'
```

Reference in deployment:

```yaml
env:
  - name: MYSQL_ROOT_PASSWORD
    valueFrom:
      secretKeyRef:
        name: iron-db-secret
        key: root-password
  - name: MYSQL_PASSWORD
    valueFrom:
      secretKeyRef:
        name: iron-db-secret
        key: user-password
```

### 2. Use PersistentVolumes for Database

Replace emptyDir with PVC:

```yaml
volumes:
  - name: db
    persistentVolumeClaim:
      claimName: iron-db-pvc
```

### 3. Add Health Checks

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 30
  periodSeconds: 10
readinessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
```

### 4. Use Ingress Instead of NodePort

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: iron-gallery-ingress
  namespace: iron-namespace-xfusion
spec:
  rules:
    - host: gallery.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: iron-gallery-service-xfusion
                port:
                  number: 80
```

### 5. Add Resource Requests (Not Just Limits)

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "25m"
  limits:
    memory: "100Mi"
    cpu: "50m"
```

---

## Troubleshooting

### Pods Not Starting

```bash
kubectl get pods -n iron-namespace-xfusion
kubectl describe pod <pod-name> -n iron-namespace-xfusion
kubectl logs <pod-name> -n iron-namespace-xfusion
```

Common issues:

- Image pull errors (check image name/tag)
- Resource constraints (insufficient CPU/memory on nodes)
- Volume mount failures

### Gallery Can't Connect to Database

**Check 1: Service DNS resolution**

```bash
kubectl exec -n iron-namespace-xfusion -l run=iron-gallery -- nslookup iron-db-service-xfusion
```

**Check 2: Database pod is running**

```bash
kubectl get pods -n iron-namespace-xfusion -l db=mariadb
```

**Check 3: Service endpoints**

```bash
kubectl get endpoints iron-db-service-xfusion -n iron-namespace-xfusion
```

### NodePort Not Accessible

- Verify firewall rules allow traffic on port 32678
- Check if nodes have external IPs
- For cloud providers, ensure security groups/NSGs allow the port
- For Minikube, use `minikube service iron-gallery-service-xfusion -n iron-namespace-xfusion`

---

## Cleanup

Delete all resources in the namespace:

```bash
kubectl delete namespace iron-namespace-xfusion
```

Or delete individual resources:

```bash
kubectl delete -f iron-gallery-app.yaml
```

---

## Key Takeaways

✅ Namespaces provide logical isolation for related resources
✅ Resource limits prevent pods from consuming excessive cluster resources
✅ EmptyDir volumes are ephemeral and suitable for temporary data
✅ ClusterIP services are for internal-only communication
✅ NodePort services expose apps outside the cluster (development/testing)
✅ Environment variables can configure containerized applications
✅ For production, use Secrets for sensitive data and PVs for persistent storage
✅ Labels and selectors connect Deployments to Services
✅ Multi-tier apps (frontend + database) are common Kubernetes patterns
