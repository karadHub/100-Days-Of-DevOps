# Day 67 — Deploy Guest Book App on Kubernetes

Goal: Deploy a multi-tier Guestbook application on Kubernetes with Redis backend (master-slave replication) and a PHP frontend, demonstrating service discovery, scaling, and resource management.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                 GUESTBOOK APPLICATION                │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────────────────────────────────────────┐  │
│  │           FRONTEND TIER                      │  │
│  │  ┌────────┐  ┌────────┐  ┌────────┐          │  │
│  │  │Frontend│  │Frontend│  │Frontend│          │  │
│  │  │  Pod   │  │  Pod   │  │  Pod   │          │  │
│  │  └────────┘  └────────┘  └────────┘          │  │
│  │       └──────────┬───────────┘                │  │
│  │            frontend-service                   │  │
│  │            (NodePort: 30009)                  │  │
│  └──────────────────────────────────────────────┘  │
│                     │                               │
│                     ▼                               │
│  ┌──────────────────────────────────────────────┐  │
│  │           BACKEND TIER                       │  │
│  │                                              │  │
│  │  ┌────────────────┐    ┌──────────────────┐ │  │
│  │  │ Redis Master   │───▶│  Redis Slave     │ │  │
│  │  │   (Write)      │    │   (Read)         │ │  │
│  │  │   1 replica    │    │   2 replicas     │ │  │
│  │  └────────────────┘    └──────────────────┘ │  │
│  │  redis-master-service  redis-slave-service  │  │
│  └──────────────────────────────────────────────┘  │
│                                                      │
└─────────────────────────────────────────────────────┘
```

---

## Requirements Summary

### Backend Tier

**Redis Master**:

- Deployment: `redis-master` (1 replica)
- Container: `master-redis-xfusion`, image: `redis`
- Resources: CPU 100m, Memory 100Mi
- Port: 6379
- Service: `redis-master` (ClusterIP)

**Redis Slave**:

- Deployment: `redis-slave` (2 replicas)
- Container: `slave-redis-xfusion`, image: `gcr.io/google_samples/gb-redisslave:v3`
- Resources: CPU 100m, Memory 100Mi
- Environment: `GET_HOSTS_FROM=dns`
- Port: 6379
- Service: `redis-slave` (ClusterIP)

### Frontend Tier

**Frontend**:

- Deployment: `frontend` (3 replicas)
- Container: `php-redis-xfusion`, image: `gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff`
- Resources: CPU 100m, Memory 100Mi
- Environment: `GET_HOSTS_FROM=dns`
- Port: 80
- Service: `frontend` (NodePort: 30009)

---

## Complete Manifests

Save as `guestbook-app.yaml`:

```yaml
# =======================
# BACKEND TIER
# =======================

# Redis Master Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-master
  labels:
    app: guestbook
    tier: backend
    role: master
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis-master
  template:
    metadata:
      labels:
        app: redis-master
        tier: backend
        role: master
    spec:
      containers:
        - name: master-redis-xfusion
          image: redis
          resources:
            requests:
              cpu: "100m"
              memory: "100Mi"
          ports:
            - containerPort: 6379
              name: redis
---
# Redis Master Service
apiVersion: v1
kind: Service
metadata:
  name: redis-master
  labels:
    app: guestbook
    tier: backend
    role: master
spec:
  type: ClusterIP
  ports:
    - port: 6379
      targetPort: 6379
      protocol: TCP
  selector:
    app: redis-master
---
# Redis Slave Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
  labels:
    app: guestbook
    tier: backend
    role: slave
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis-slave
  template:
    metadata:
      labels:
        app: redis-slave
        tier: backend
        role: slave
    spec:
      containers:
        - name: slave-redis-xfusion
          image: gcr.io/google_samples/gb-redisslave:v3
          resources:
            requests:
              cpu: "100m"
              memory: "100Mi"
          env:
            - name: GET_HOSTS_FROM
              value: dns
          ports:
            - containerPort: 6379
              name: redis
---
# Redis Slave Service
apiVersion: v1
kind: Service
metadata:
  name: redis-slave
  labels:
    app: guestbook
    tier: backend
    role: slave
spec:
  type: ClusterIP
  ports:
    - port: 6379
      targetPort: 6379
      protocol: TCP
  selector:
    app: redis-slave
---
# =======================
# FRONTEND TIER
# =======================

# Frontend Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: guestbook
      tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
        - name: php-redis-xfusion
          image: gcr.io/google-samples/gb-frontend@sha256:a908df8486ff66f2c4daa0d3d8a2fa09846a1fc8efd65649c0109695c7c5cbff
          resources:
            requests:
              cpu: "100m"
              memory: "100Mi"
          env:
            - name: GET_HOSTS_FROM
              value: dns
          ports:
            - containerPort: 80
              name: http
---
# Frontend Service (NodePort)
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30009
      protocol: TCP
  selector:
    app: guestbook
    tier: frontend
```

---

## Deployment Steps

### Apply All Resources

```bash
kubectl apply -f guestbook-app.yaml
```

### Watch Resources Being Created

```bash
# Watch all pods
kubectl get pods -w

# Or watch specific tiers
kubectl get pods -l tier=backend -w
kubectl get pods -l tier=frontend -w
```

---

## Verification

### 1. Check All Deployments

```bash
kubectl get deployments
```

Expected output:

```
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
redis-master   1/1     1            1           2m
redis-slave    2/2     2            2           2m
frontend       3/3     3            3           2m
```

### 2. Check All Pods

```bash
kubectl get pods -o wide
```

Expected:

- 1 redis-master pod (Running)
- 2 redis-slave pods (Running)
- 3 frontend pods (Running)

**Check by tier**:

```bash
kubectl get pods -l tier=backend
kubectl get pods -l tier=frontend
```

### 3. Check Services

```bash
kubectl get services
```

Expected output:

```
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
redis-master   ClusterIP   10.96.x.x       <none>        6379/TCP       2m
redis-slave    ClusterIP   10.96.x.x       <none>        6379/TCP       2m
frontend       NodePort    10.96.x.x       <none>        80:30009/TCP   2m
```

### 4. Verify Service Endpoints

```bash
kubectl get endpoints redis-master
kubectl get endpoints redis-slave
kubectl get endpoints frontend
```

Each should show the corresponding pod IPs.

### 5. Check Resource Requests

```bash
kubectl describe deployment redis-master | grep -A 5 "Requests:"
kubectl describe deployment redis-slave | grep -A 5 "Requests:"
kubectl describe deployment frontend | grep -A 5 "Requests:"
```

Verify each shows:

```
Requests:
  cpu:     100m
  memory:  100Mi
```

### 6. Check Environment Variables

```bash
# Redis slave
kubectl exec -it $(kubectl get pod -l app=redis-slave -o jsonpath='{.items[0].metadata.name}') -- env | grep GET_HOSTS_FROM

# Frontend
kubectl exec -it $(kubectl get pod -l tier=frontend -o jsonpath='{.items[0].metadata.name}') -- env | grep GET_HOSTS_FROM
```

Both should output:

```
GET_HOSTS_FROM=dns
```

---

## Testing the Application

### 1. Access via NodePort

Get a node IP:

```bash
kubectl get nodes -o wide
```

**Access in browser**:

```
http://<NODE_IP>:30009
```

You should see the Guestbook application interface.

**Or use curl**:

```bash
curl http://<NODE_IP>:30009
```

### 2. Test Guestbook Functionality

1. Open the app in a browser at `http://<NODE_IP>:30009`
2. Enter a message in the text box
3. Click "Submit"
4. Verify the message appears in the guestbook
5. Refresh the page to confirm data persistence

### 3. Test Service Discovery

**Verify frontend can reach Redis**:

```bash
# Get a frontend pod
FRONTEND_POD=$(kubectl get pod -l tier=frontend -o jsonpath='{.items[0].metadata.name}')

# Check if it can resolve redis-master
kubectl exec $FRONTEND_POD -- nslookup redis-master

# Check if it can resolve redis-slave
kubectl exec $FRONTEND_POD -- nslookup redis-slave
```

### 4. Test Redis Master-Slave Replication

**Write to master**:

```bash
MASTER_POD=$(kubectl get pod -l app=redis-master -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $MASTER_POD -- redis-cli SET testkey "Hello from master"
```

**Read from slave**:

```bash
SLAVE_POD=$(kubectl get pod -l app=redis-slave -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $SLAVE_POD -- redis-cli GET testkey
```

Should return: `"Hello from master"`

---

## Scaling the Application

### Scale Frontend

```bash
# Scale up
kubectl scale deployment frontend --replicas=5

# Verify
kubectl get deployment frontend
kubectl get pods -l tier=frontend
```

### Scale Redis Slave

```bash
# Scale up for more read capacity
kubectl scale deployment redis-slave --replicas=4

# Verify
kubectl get deployment redis-slave
kubectl get pods -l app=redis-slave
```

**Note**: Redis master should remain at 1 replica (single write node).

---

## Monitoring and Logs

### View Logs

```bash
# Frontend logs
kubectl logs -l tier=frontend --tail=50

# Redis master logs
kubectl logs -l app=redis-master --tail=50

# Redis slave logs
kubectl logs -l app=redis-slave --tail=50
```

### Follow Real-time Logs

```bash
kubectl logs -f -l tier=frontend
```

### Describe Resources

```bash
kubectl describe deployment frontend
kubectl describe svc frontend
kubectl describe pod <pod-name>
```

---

## Understanding the Components

### DNS-based Service Discovery

The environment variable `GET_HOSTS_FROM=dns` tells the application to use Kubernetes DNS for service discovery:

- Frontend looks up `redis-master` for writes
- Frontend looks up `redis-slave` for reads
- Kubernetes DNS automatically resolves these names to the correct service IPs

### Master-Slave Architecture

**Writes**:

- All write operations (new guestbook entries) go to `redis-master`
- Only one master ensures data consistency

**Reads**:

- Read operations are load-balanced across `redis-slave` pods
- Multiple slaves improve read throughput and availability

### Resource Requests

Each container requests:

- CPU: 100m (0.1 cores) - Ensures minimum CPU allocation
- Memory: 100Mi - Ensures minimum memory allocation

Benefits:

- Kubernetes can make better scheduling decisions
- Prevents resource starvation
- Enables horizontal pod autoscaling (if limits are also set)

---

## Production Improvements

### 1. Add Resource Limits

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "100Mi"
  limits:
    cpu: "200m"
    memory: "200Mi"
```

### 2. Add Health Probes

**Frontend**:

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

**Redis**:

```yaml
livenessProbe:
  tcpSocket:
    port: 6379
  initialDelaySeconds: 30
  periodSeconds: 10
readinessProbe:
  exec:
    command:
      - redis-cli
      - ping
  initialDelaySeconds: 5
  periodSeconds: 5
```

### 3. Use Persistent Storage for Redis

```yaml
volumeMounts:
  - name: redis-data
    mountPath: /data
volumes:
  - name: redis-data
    persistentVolumeClaim:
      claimName: redis-pvc
```

### 4. Add Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: frontend
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

### 5. Use Ingress Instead of NodePort

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: guestbook-ingress
spec:
  rules:
    - host: guestbook.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

### 6. Add Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-policy
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              tier: frontend
      ports:
        - protocol: TCP
          port: 6379
```

---

## Troubleshooting

### Frontend Can't Connect to Redis

**Check 1: Service DNS resolution**

```bash
kubectl exec -it $(kubectl get pod -l tier=frontend -o jsonpath='{.items[0].metadata.name}') -- nslookup redis-master
```

**Check 2: Redis pods are running**

```bash
kubectl get pods -l tier=backend
```

**Check 3: Environment variable set correctly**

```bash
kubectl exec -it $(kubectl get pod -l tier=frontend -o jsonpath='{.items[0].metadata.name}') -- env | grep GET_HOSTS_FROM
```

### Pods Stuck in Pending

```bash
kubectl describe pod <pod-name>
```

Common causes:

- Insufficient cluster resources (CPU/memory)
- Node taints preventing scheduling
- Image pull issues

### Application Not Loading

**Check frontend pod logs**:

```bash
kubectl logs -l tier=frontend
```

**Check service endpoints**:

```bash
kubectl get endpoints frontend
```

**Verify NodePort is accessible**:

```bash
# Test from within cluster
kubectl run test --rm -it --image=curlimages/curl -- curl http://frontend:80

# Test externally
curl http://<NODE_IP>:30009
```

---

## Cleanup

```bash
kubectl delete -f guestbook-app.yaml
```

Or delete individual resources:

```bash
kubectl delete deployment redis-master redis-slave frontend
kubectl delete service redis-master redis-slave frontend
```

---

## Key Takeaways

✅ **Multi-tier architecture** separates concerns (frontend, backend)
✅ **Master-slave pattern** improves read scalability and availability
✅ **DNS-based service discovery** enables loose coupling between tiers
✅ **Resource requests** ensure minimum resource allocation
✅ **Multiple replicas** provide high availability and load distribution
✅ **ClusterIP services** for internal communication (Redis)
✅ **NodePort service** for external access (Frontend)
✅ **Labels and selectors** organize and route traffic to pods
✅ **Scaling** is as simple as changing replica count
✅ **Kubernetes handles** load balancing, service discovery, and pod lifecycle automatically

This guestbook application demonstrates fundamental Kubernetes patterns used in production systems!
