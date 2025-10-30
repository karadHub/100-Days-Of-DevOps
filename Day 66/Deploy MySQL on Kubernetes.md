# Day 66 — Deploy MySQL on Kubernetes

Goal: Deploy a production-ready MySQL database on Kubernetes with persistent storage, proper secret management, and external access via NodePort.

---

## Requirements Summary

1. **PersistentVolume**: `mysql-pv` with 250Mi capacity
2. **PersistentVolumeClaim**: `mysql-pv-claim` requesting 250Mi
3. **Deployment**: `mysql-deployment` mounting PVC at `/var/lib/mysql`
4. **Service**: NodePort `mysql` on port 30007
5. **Secrets**: Three secrets for root password, user credentials, and database name
6. **Environment Variables**: Configured from secret references

---

## Complete Manifests

You can create a single file `mysql-deployment.yaml` or split into separate files.

### 1. PersistentVolume

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data/mysql
  persistentVolumeReclaimPolicy: Retain
```

**Notes**:

- `hostPath` is used for development/testing (single-node clusters)
- For production, use cloud provider volumes (AWS EBS, GCP PD, Azure Disk) or network storage (NFS, Ceph)
- `Retain` policy keeps data even after PVC deletion

---

### 2. PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi
```

**Notes**:

- This will bind to the `mysql-pv` PersistentVolume
- If using a StorageClass, you can add `storageClassName: <class-name>` for dynamic provisioning

---

### 3. Secrets

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-root-pass
type: Opaque
stringData:
  password: YUIidhb667
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-user-pass
type: Opaque
stringData:
  username: kodekloud_roy
  password: Rc5C9EyvbU
---
apiVersion: v1
kind: Secret
metadata:
  name: mysql-db-url
type: Opaque
stringData:
  database: kodekloud_db8
```

**Notes**:

- Using `stringData` instead of `data` (no need to base64 encode manually)
- For production, use external secret managers (Vault, AWS Secrets Manager, etc.)
- Never commit secrets to Git repositories

**Alternative: Create secrets via kubectl**:

```bash
kubectl create secret generic mysql-root-pass --from-literal=password=YUIidhb667
kubectl create secret generic mysql-user-pass --from-literal=username=kodekloud_roy --from-literal=password=Rc5C9EyvbU
kubectl create secret generic mysql-db-url --from-literal=database=kodekloud_db8
```

---

### 4. Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          ports:
            - containerPort: 3306
              name: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-root-pass
                  key: password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-db-url
                  key: database
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: username
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-user-pass
                  key: password
          volumeMounts:
            - name: mysql-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-storage
          persistentVolumeClaim:
            claimName: mysql-pv-claim
```

**Notes**:

- MySQL container will automatically create the database and user on first startup
- Data persists in `/var/lib/mysql` which is backed by the PVC
- Using `mysql:8.0` (you can use `mysql:5.7` or `mysql:latest` if preferred)

---

### 5. Service (NodePort)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  selector:
    app: mysql
  ports:
    - name: mysql
      port: 3306
      targetPort: 3306
      nodePort: 30007
      protocol: TCP
```

**Notes**:

- NodePort `30007` makes MySQL accessible from outside the cluster
- For production, consider using ClusterIP and accessing via an internal service or Ingress
- Security: Ensure firewall rules restrict access to trusted IPs only

---

## Deployment Steps

### Option A: Apply All at Once

Create `mysql-deployment.yaml` with all manifests separated by `---`:

```bash
kubectl apply -f mysql-deployment.yaml
```

### Option B: Apply in Order

```bash
# 1. Create secrets first
kubectl apply -f mysql-secrets.yaml

# 2. Create PV and PVC
kubectl apply -f mysql-pv.yaml
kubectl apply -f mysql-pvc.yaml

# Wait for PVC to bind
kubectl get pvc mysql-pv-claim -w

# 3. Create deployment
kubectl apply -f mysql-deployment.yaml

# 4. Create service
kubectl apply -f mysql-service.yaml
```

---

## Verification

### 1. Check PersistentVolume and Claim

```bash
kubectl get pv mysql-pv
kubectl get pvc mysql-pv-claim
```

Expected output:

```
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   AGE
mysql-pv   250Mi      RWO            Retain           Bound    default/mysql-pv-claim                  1m

NAME              STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pv-claim    Bound    mysql-pv   250Mi      RWO                           1m
```

The `STATUS` should be `Bound`.

---

### 2. Check Secrets

```bash
kubectl get secrets | grep mysql
```

Expected:

```
mysql-db-url       Opaque   1      2m
mysql-root-pass    Opaque   1      2m
mysql-user-pass    Opaque   2      2m
```

**Verify secret contents** (base64 decoded):

```bash
kubectl get secret mysql-root-pass -o jsonpath='{.data.password}' | base64 --decode
kubectl get secret mysql-user-pass -o jsonpath='{.data.username}' | base64 --decode
kubectl get secret mysql-db-url -o jsonpath='{.data.database}' | base64 --decode
```

---

### 3. Check Deployment and Pods

```bash
kubectl get deployment mysql-deployment
kubectl get pods -l app=mysql
```

Expected:

```
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
mysql-deployment    1/1     1            1           3m

NAME                                READY   STATUS    RESTARTS   AGE
mysql-deployment-xxxxxxxxxx-xxxxx   1/1     Running   0          3m
```

**Check pod logs**:

```bash
POD=$(kubectl get pods -l app=mysql -o jsonpath='{.items[0].metadata.name}')
kubectl logs $POD
```

Look for messages like:

```
[Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.xx'
[Server] MySQL Community Server - GPL
```

---

### 4. Check Service

```bash
kubectl get svc mysql
```

Expected:

```
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
mysql   NodePort   10.96.x.x       <none>        3306:30007/TCP   3m
```

**Describe service**:

```bash
kubectl describe svc mysql
```

Verify:

- `Type: NodePort`
- `NodePort: 30007`
- `Endpoints` shows the pod IP

---

### 5. Verify Environment Variables

```bash
kubectl exec $POD -- env | grep MYSQL
```

Expected output:

```
MYSQL_ROOT_PASSWORD=YUIidhb667
MYSQL_DATABASE=kodekloud_db8
MYSQL_USER=kodekloud_roy
MYSQL_PASSWORD=Rc5C9EyvbU
```

---

## Testing MySQL Connection

### From Within the Cluster

```bash
# Create a temporary pod with mysql client
kubectl run mysql-client --rm -it --image=mysql:8.0 -- /bin/bash

# Inside the pod, connect to MySQL
mysql -h mysql -u kodekloud_roy -pRc5C9EyvbU kodekloud_db8
```

**Test commands**:

```sql
SHOW DATABASES;
USE kodekloud_db8;
CREATE TABLE test (id INT, name VARCHAR(50));
INSERT INTO test VALUES (1, 'Hello Kubernetes');
SELECT * FROM test;
```

---

### From Outside the Cluster (NodePort)

Get a node IP:

```bash
kubectl get nodes -o wide
```

**Connect using mysql client** (from your machine):

```bash
mysql -h <NODE_IP> -P 30007 -u kodekloud_roy -pRc5C9EyvbU kodekloud_db8
```

**Or using a Docker container**:

```bash
docker run -it --rm mysql:8.0 mysql -h <NODE_IP> -P 30007 -u kodekloud_roy -pRc5C9EyvbU kodekloud_db8
```

---

## Production Improvements

### 1. Add Resource Limits

```yaml
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1000m"
```

### 2. Add Health Probes

```yaml
livenessProbe:
  exec:
    command:
      - mysqladmin
      - ping
      - -h
      - localhost
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5

readinessProbe:
  exec:
    command:
      - mysqladmin
      - ping
      - -h
      - localhost
  initialDelaySeconds: 10
  periodSeconds: 5
  timeoutSeconds: 1
```

### 3. Use ConfigMap for MySQL Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  my.cnf: |
    [mysqld]
    max_connections=200
    innodb_buffer_pool_size=256M
```

Mount it:

```yaml
volumeMounts:
  - name: mysql-config
    mountPath: /etc/mysql/conf.d
volumes:
  - name: mysql-config
    configMap:
      name: mysql-config
```

### 4. Use StatefulSet for High Availability

For production, consider using a StatefulSet instead of Deployment for better identity and ordered deployment:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 1
  # ... rest of spec
```

### 5. Enable Automated Backups

Use CronJobs to periodically back up MySQL data:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: mysql-backup
spec:
  schedule: "0 2 * * *" # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: mysql-backup
              image: mysql:8.0
              command:
                - /bin/sh
                - -c
                - |
                  mysqldump -h mysql -u root -p$MYSQL_ROOT_PASSWORD --all-databases > /backup/mysql-$(date +%Y%m%d).sql
              env:
                - name: MYSQL_ROOT_PASSWORD
                  valueFrom:
                    secretKeyRef:
                      name: mysql-root-pass
                      key: password
              volumeMounts:
                - name: backup
                  mountPath: /backup
          volumes:
            - name: backup
              persistentVolumeClaim:
                claimName: mysql-backup-pvc
          restartPolicy: OnFailure
```

---

## Troubleshooting

### Pod Not Starting

```bash
kubectl describe pod $POD
kubectl logs $POD
```

Common issues:

- PVC not bound → Check PV configuration
- Secret not found → Verify secret names
- Permission errors → Check volume mount permissions

### Connection Refused

- Verify pod is running: `kubectl get pods -l app=mysql`
- Check service endpoints: `kubectl get endpoints mysql`
- Test internal connectivity: `kubectl run test --rm -it --image=mysql:8.0 -- mysql -h mysql -u kodekloud_roy -pRc5C9EyvbU`
- For NodePort access: ensure firewall allows port 30007

### Data Not Persisting

- Check PVC is bound to PV
- Verify volume mount path: `kubectl exec $POD -- ls -la /var/lib/mysql`
- Check PV reclaim policy is set to `Retain`

### MySQL Initialization Fails

Check logs for errors:

```bash
kubectl logs $POD | grep -i error
```

Common causes:

- Invalid credentials
- Database already exists with different configuration
- Insufficient permissions on volume

---

## Security Best Practices

✅ **Never hardcode passwords** in Deployment YAML
✅ **Use Secrets** for all sensitive data
✅ **Rotate credentials** regularly
✅ **Use RBAC** to restrict access to secrets
✅ **Enable encryption at rest** for PersistentVolumes
✅ **Use ClusterIP** instead of NodePort for internal-only databases
✅ **Implement network policies** to restrict database access
✅ **Enable MySQL SSL/TLS** for encrypted connections
✅ **Regular backups** to external storage
✅ **Monitor and audit** database access logs

---

## Cleanup

```bash
# Delete all resources
kubectl delete deployment mysql-deployment
kubectl delete svc mysql
kubectl delete pvc mysql-pv-claim
kubectl delete pv mysql-pv
kubectl delete secret mysql-root-pass mysql-user-pass mysql-db-url

# Or if using single file
kubectl delete -f mysql-deployment.yaml
```

**Warning**: Deleting PVC will delete the PV if reclaim policy is `Delete`. Use `Retain` to keep data.

---

## Key Takeaways

✅ PersistentVolumes provide durable storage for stateful applications like databases
✅ Secrets should be used for sensitive data (passwords, connection strings)
✅ Environment variables can reference secret keys using `secretKeyRef`
✅ MySQL automatically creates users and databases on first startup based on env vars
✅ NodePort services expose applications outside the cluster (use with caution in production)
✅ Always add resource limits, health probes, and backups for production databases
✅ For production, consider StatefulSets, dedicated storage classes, and high availability setups
✅ Test both internal (ClusterIP) and external (NodePort) connectivity
✅ Monitor MySQL logs and performance metrics regularly
