# Day 53: Resolve VolumeMounts Issue in Kubernetes

## 🧩 Task Overview: Restore Nginx + PHP-FPM Website on Kubernetes

This morning, the Kubernetes pod running our Nginx and PHP-FPM setup (`nginx-phpfpm`) experienced a failure, causing the hosted website to become inaccessible. Your objective is to investigate, resolve the issue, and restore functionality.

---

## 🔍 Investigation Details

- **Pod Name**: `nginx-phpfpm`
- **ConfigMap Name**: `nginx-config`

### Step 1: Diagnose the Issue

- Inspect the pod status and logs:
  ```bash
  kubectl describe pod nginx-phpfpm
  kubectl logs nginx-phpfpm -c nginx-container
  kubectl logs nginx-phpfpm -c php-fpm-container
  ```
- Review the Nginx configuration:
  ```bash
  kubectl get configmap nginx-config -o yaml
  ```

Look for:

- Incorrect `fastcgi_pass` target
- Mismatched document root paths between containers
- Missing or misconfigured volume mounts

---

## 🛠️ Resolution Steps

### Step 2: Fix Nginx Configuration

- Ensure both containers reference the same shared volume path:
  - Update `root` and `SCRIPT_FILENAME` in `nginx.conf` to:
    ```nginx
    root /usr/share/nginx/html;
    fastcgi_param SCRIPT_FILENAME /usr/share/nginx/html$fastcgi_script_name;
    ```
- Apply changes:
  ```bash
  kubectl edit configmap nginx-config
  kubectl delete pod nginx-phpfpm
  ```

### Step 3: Wait for Pod Recreation

```bash
kubectl get pods -w
# Wait for nginx-phpfpm to be in Running state
```

---

## 📁 Step 4: Deploy Website File

Once the pod is running correctly:

- Copy the PHP file from the jump host to the Nginx container:
  ```bash
  kubectl cp /home/thor/index.php nginx-phpfpm:/usr/share/nginx/html/index.php -c nginx-container
  ```

---

## ✅ Step 5: Validate

Use the **Website** button on the top bar to confirm the site is accessible and functioning as expected.

---

## 🔧 Additional Debugging Commands

```bash
# Check pod details
kubectl describe pod nginx-phpfpm

# Verify volume mounts
kubectl get pod nginx-phpfpm -o jsonpath='{.spec.containers[*].volumeMounts}'

# Test connectivity between containers
kubectl exec -it nginx-phpfpm -c nginx-container -- curl localhost:9000

# Check file permissions in shared volume
kubectl exec -it nginx-phpfpm -c nginx-container -- ls -la /usr/share/nginx/html/
```

---

## 🧠 Common Issues & Solutions

| Issue                     | Symptom                               | Solution                                             |
| ------------------------- | ------------------------------------- | ---------------------------------------------------- |
| Volume mount mismatch     | 502 Bad Gateway                       | Align document root paths between containers         |
| FastCGI connection failed | PHP files download instead of execute | Fix `fastcgi_pass` to point to `localhost:9000`      |
| File permissions          | 403 Forbidden                         | Ensure proper ownership/permissions on shared volume |
