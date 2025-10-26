# Day 64 — Fix Python App Deployed on Kubernetes Cluster

Goal: Troubleshoot and fix a misconfigured Python Flask application deployment on Kubernetes. The app is deployed but not accessible via NodePort due to port mismatch issues.

---

## Problem Statement

A Python Flask application (`python-deployment-devops`) using the `poroko/flask-demo-app` image has been deployed to Kubernetes, but it's not accessible via the expected NodePort `32345`. The deployment and service exist, but something is misconfigured.

**Expected Behavior**: App should be accessible at `http://<NodeIP>:32345`
**Actual Behavior**: Connection fails or returns errors

---

## Diagnostic Process

### Step 1: Gather Information

First, check what resources exist:

```bash
# List all resources
kubectl get all -l app=python_app

# Check deployment
kubectl get deployment python-deployment-devops

# Check service
kubectl get svc python-service-devops

# Check pods
kubectl get pods -l app=python_app
```

---

### Step 2: Inspect the Deployment

```bash
kubectl get deployment python-deployment-devops -o yaml
```

**Key things to verify:**

- Image name and tag
- Container port configuration
- Labels and selectors

**Expected Configuration:**

```yaml
spec:
  template:
    spec:
      containers:
        - name: python-container
          image: poroko/flask-demo-app
          ports:
            - containerPort: 5000 # Flask default port
```

✅ **Finding**: Deployment correctly exposes port `5000` (Flask's default port)

---

### Step 3: Inspect the Service

```bash
kubectl get svc python-service-devops -o yaml
```

**Check the service configuration:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-service-devops
spec:
  type: NodePort
  selector:
    app: python_app
  ports:
    - port: 8080
      targetPort: 8080 # ❌ PROBLEM: Should be 5000!
      nodePort: 32345
```

❌ **Root Cause Found**: The `targetPort` is set to `8080`, but the Flask app listens on port `5000`!

---

### Step 4: Verify Pod Logs

```bash
# Get pod name
POD_NAME=$(kubectl get pods -l app=python_app -o jsonpath='{.items[0].metadata.name}')

# Check logs
kubectl logs $POD_NAME
```

**Expected log output:**

```
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

This confirms the Flask app is listening on port `5000`, not `8080`.

---

### Step 5: Test Service Endpoint

```bash
kubectl get endpoints python-service-devops
```

Check if the endpoint shows the correct pod IP and port:

```
NAME                      ENDPOINTS         AGE
python-service-devops     10.244.x.x:5000   5m
```

If the port is wrong here, it confirms the service misconfiguration.

---

## Solution

### Option A: Patch the Existing Service

The quickest fix is to patch the service to correct the `targetPort`:

```bash
kubectl patch svc python-service-devops \
  -p '{"spec":{"ports":[{"port":8080,"targetPort":5000,"nodePort":32345,"protocol":"TCP"}]}}'
```

**Verify the patch:**

```bash
kubectl get svc python-service-devops -o yaml | grep -A 5 ports
```

Expected output:

```yaml
ports:
  - port: 8080
    targetPort: 5000 # ✅ Fixed!
    nodePort: 32345
    protocol: TCP
```

---

### Option B: Edit the Service Interactively

```bash
kubectl edit svc python-service-devops
```

Change:

```yaml
targetPort: 8080
```

To:

```yaml
targetPort: 5000
```

Save and exit.

---

### Option C: Replace with Correct YAML

Create a file `python-service-fixed.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-service-devops
spec:
  type: NodePort
  selector:
    app: python_app
  ports:
    - name: http
      port: 8080
      targetPort: 5000
      nodePort: 32345
      protocol: TCP
```

Apply it:

```bash
kubectl apply -f python-service-fixed.yaml
```

---

## Verification

### 1. Check Service Configuration

```bash
kubectl describe svc python-service-devops
```

Verify:

- `TargetPort: 5000/TCP`
- `NodePort: 32345/TCP`
- `Endpoints` shows pod IP with port 5000

---

### 2. Check Endpoints

```bash
kubectl get endpoints python-service-devops
```

Expected:

```
NAME                      ENDPOINTS           AGE
python-service-devops     10.244.x.x:5000     10m
```

---

### 3. Test Internal Connectivity

From within the cluster:

```bash
kubectl run test-pod --rm -it --image=curlimages/curl -- sh

# Inside the pod
curl http://python-service-devops:8080
```

Expected response:

```
Hello World Pyvo 1!
```

---

### 4. Test NodePort Access

Get a node IP:

```bash
kubectl get nodes -o wide
```

Test from your machine:

```bash
curl http://<NODE_IP>:32345
```

**Expected output:**

```
Hello World Pyvo 1!
```

Or access in a browser:

```
http://<NODE_IP>:32345
```

---

### 5. Check Pod Logs

Verify the Flask app is receiving requests:

```bash
kubectl logs -f $POD_NAME
```

You should see access logs like:

```
10.244.x.x - - [26/Oct/2025 10:30:45] "GET / HTTP/1.1" 200 -
```

---

## Understanding the Issue

### Port Configuration in Kubernetes Services

```
External Client → NodePort (32345) → Service Port (8080) → TargetPort (5000) → Container Port (5000)
                                                              ↑
                                                        This must match!
```

**Key points:**

- **`containerPort`** (in Deployment): The port the application listens on inside the container
- **`targetPort`** (in Service): Must match the `containerPort`
- **`port`** (in Service): The port the service listens on (cluster-internal)
- **`nodePort`** (in Service): The port exposed on each node (external access)

**The problem**: `targetPort: 8080` didn't match `containerPort: 5000`

---

## Common Troubleshooting Commands

### Quick Diagnosis Checklist

```bash
# 1. Check deployment port
kubectl get deployment python-deployment-devops -o jsonpath='{.spec.template.spec.containers[0].ports[0].containerPort}'

# 2. Check service targetPort
kubectl get svc python-service-devops -o jsonpath='{.spec.ports[0].targetPort}'

# 3. Compare them - they should match!

# 4. Check endpoints
kubectl get endpoints python-service-devops

# 5. Check pod logs
kubectl logs -l app=python_app

# 6. Test connectivity
kubectl run test --rm -it --image=curlimages/curl -- curl http://python-service-devops:8080
```

---

## Production Best Practices

### 1. Use Consistent Port Naming

```yaml
# Deployment
ports:
  - name: http
    containerPort: 5000

# Service
ports:
  - name: http
    port: 8080
    targetPort: http  # ✅ Reference by name!
    nodePort: 32345
```

### 2. Add Readiness Probes

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 5000
  initialDelaySeconds: 5
  periodSeconds: 5
```

### 3. Add Liveness Probes

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 5000
  initialDelaySeconds: 15
  periodSeconds: 20
```

### 4. Use ConfigMaps for App Configuration

```yaml
env:
  - name: FLASK_PORT
    valueFrom:
      configMapKeyRef:
        name: flask-config
        key: port
```

### 5. Use Ingress for Production

Instead of NodePort, use an Ingress controller:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: flask-ingress
spec:
  rules:
    - host: flask.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: python-service-devops
                port:
                  number: 8080
```

---

## Troubleshooting Other Common Issues

### Issue: Pods Not Starting

```bash
kubectl describe pod $POD_NAME
```

Look for:

- `ImagePullBackOff`: Wrong image name/tag or registry auth issues
- `CrashLoopBackOff`: Application crashes on startup
- `Pending`: Insufficient resources or scheduling issues

### Issue: Service Not Routing Traffic

```bash
# Check if labels match
kubectl get pods --show-labels
kubectl get svc python-service-devops -o yaml | grep -A 3 selector

# Check endpoints
kubectl get endpoints python-service-devops
```

If endpoints are empty, the selector doesn't match any pods.

### Issue: NodePort Not Accessible

- Verify firewall rules allow traffic on port 32345
- Check node external IPs: `kubectl get nodes -o wide`
- For cloud providers, check security groups/NSGs
- For Minikube: `minikube service python-service-devops --url`
- For kind: Port forwarding may be needed

---

## Complete Working Configuration

### Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-deployment-devops
spec:
  replicas: 1
  selector:
    matchLabels:
      app: python_app
  template:
    metadata:
      labels:
        app: python_app
    spec:
      containers:
        - name: python-container
          image: poroko/flask-demo-app
          ports:
            - name: http
              containerPort: 5000
          readinessProbe:
            httpGet:
              path: /
              port: 5000
            initialDelaySeconds: 5
            periodSeconds: 5
```

### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: python-service-devops
spec:
  type: NodePort
  selector:
    app: python_app
  ports:
    - name: http
      port: 8080
      targetPort: 5000
      nodePort: 32345
      protocol: TCP
```

---

## Cleanup

```bash
kubectl delete deployment python-deployment-devops
kubectl delete svc python-service-devops
```

---

## Key Takeaways

✅ **Always match** `targetPort` in Service with `containerPort` in Deployment
✅ Flask apps default to port `5000` unless configured otherwise
✅ Use `kubectl describe` and `kubectl logs` for initial diagnosis
✅ Check endpoints to verify service-to-pod routing
✅ Use port names (not numbers) for better maintainability
✅ Add health probes for production deployments
✅ Test connectivity both internally (ClusterIP) and externally (NodePort)
✅ Document port configurations in your deployment specs
✅ Consider using Ingress instead of NodePort for production workloads
