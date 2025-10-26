# Day 62 — Manage Secrets in Kubernetes

Goal: Store sensitive information (licenses, passwords, API keys) securely in Kubernetes using Secrets, and consume them in pods via volume mounts.

---

## Scenario

The Nautilus DevOps team needs to deploy license-based tools in their Kubernetes cluster. License keys and passwords must be stored securely using Kubernetes Secrets and mounted into pods for application use.

---

## Task Requirements

1. **Create a Secret** named `news` from the file `/opt/news.txt` (contains license/password)
2. **Create a Pod** named `secret-datacenter` with:
   - Container name: `secret-container-datacenter`
   - Image: `ubuntu:latest`
   - Command: keep container running with sleep
   - Mount the secret at `/opt/apps` inside the container
3. **Verify** the secret is accessible inside the container

---

## Solution

### Step 1: Create the Secret from File

Assuming you're on the jump host and `news.txt` exists at `/opt/news.txt`:

```bash
kubectl create secret generic news --from-file=/opt/news.txt
```

**What this does:**

- Creates a secret named `news`
- The key inside the secret will be `news.txt` (filename)
- The value will be the file contents

**Verify the secret:**

```bash
kubectl get secret news
kubectl describe secret news
```

---

### Step 2: Create the Pod YAML

File: `secret-datacenter.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-datacenter
spec:
  containers:
    - name: secret-container-datacenter
      image: ubuntu:latest
      command: ["/bin/bash", "-c", "sleep infinity"]
      volumeMounts:
        - name: secret-volume
          mountPath: /opt/apps
          readOnly: true
  volumes:
    - name: secret-volume
      secret:
        secretName: news
```

**Apply the pod:**

```bash
kubectl apply -f secret-datacenter.yaml
```

---

### Step 3: Verify the Secret Mount

**Wait for pod to be running:**

```bash
kubectl get pod secret-datacenter -w
```

**Exec into the container:**

```bash
kubectl exec -it secret-datacenter -- /bin/bash
```

**Inside the container, check the mounted secret:**

```bash
# List files in the mount path
ls -la /opt/apps

# Display the secret content
cat /opt/apps/news.txt
```

You should see the license/password content from the original `/opt/news.txt` file.

**Alternative verification (without exec):**

```bash
kubectl exec secret-datacenter -- cat /opt/apps/news.txt
```

---

## Understanding Kubernetes Secrets

### What are Secrets?

Secrets are Kubernetes objects used to store sensitive information like:

- Passwords
- API tokens
- SSH keys
- TLS certificates
- License keys

### Secret Types

| Type                                  | Use Case                    | Example                 |
| ------------------------------------- | --------------------------- | ----------------------- |
| `Opaque` (generic)                    | Generic key-value pairs     | API keys, passwords     |
| `kubernetes.io/service-account-token` | Service account tokens      | Auto-created for pods   |
| `kubernetes.io/dockerconfigjson`      | Docker registry credentials | Private image pulls     |
| `kubernetes.io/tls`                   | TLS certificates            | HTTPS/SSL certs         |
| `kubernetes.io/ssh-auth`              | SSH keys                    | Git authentication      |
| `kubernetes.io/basic-auth`            | Basic auth credentials      | Username/password pairs |

---

## Different Ways to Create Secrets

### 1. From File (as shown above)

```bash
kubectl create secret generic news --from-file=/opt/news.txt
```

### 2. From Literal Values

```bash
kubectl create secret generic db-creds \
  --from-literal=username=admin \
  --from-literal=password=super-secret-password
```

### 3. From YAML Manifest (Base64 encoded)

First, encode your values:

```bash
echo -n "my-license-key" | base64
# Output: bXktbGljZW5zZS1rZXk=
```

Then create the YAML:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: news
type: Opaque
data:
  license: bXktbGljZW5zZS1rZXk=
```

Apply:

```bash
kubectl apply -f secret.yaml
```

### 4. From Multiple Files

```bash
kubectl create secret generic app-secrets \
  --from-file=./config.json \
  --from-file=./api-key.txt \
  --from-file=./cert.pem
```

---

## Consuming Secrets in Pods

### Method 1: As Volume Mounts (Used in this task)

**Pros:**

- Files appear as regular files in the filesystem
- Multiple keys appear as separate files
- Supports read-only mounts
- Atomic updates when secret changes

**Example:**

```yaml
volumeMounts:
  - name: secret-volume
    mountPath: /opt/apps
    readOnly: true
volumes:
  - name: secret-volume
    secret:
      secretName: news
```

### Method 2: As Environment Variables

**Pros:**

- Simple access via env vars
- No volume management needed

**Cons:**

- Env vars are visible in process listings
- Less secure than volume mounts

**Example:**

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-creds
        key: password
  - name: API_TOKEN
    valueFrom:
      secretKeyRef:
        name: app-secrets
        key: api-key
```

### Method 3: Mount Specific Keys Only

```yaml
volumes:
  - name: secret-volume
    secret:
      secretName: news
      items:
        - key: news.txt
          path: license.key
          mode: 0400
```

---

## Best Practices

### Security

✅ Use RBAC to restrict access to secrets
✅ Enable encryption at rest for secrets in etcd
✅ Use `readOnly: true` for volume mounts when possible
✅ Avoid logging secret values
✅ Don't commit secrets to Git (use external secret managers)
✅ Prefer volume mounts over environment variables for sensitive data

### Management

✅ Use descriptive names for secrets
✅ Document what each secret contains
✅ Rotate secrets regularly
✅ Use namespaces to isolate secrets
✅ Consider external secret managers (Vault, AWS Secrets Manager, Azure Key Vault)

### Monitoring

✅ Audit secret access via Kubernetes audit logs
✅ Monitor for unauthorized secret access
✅ Set up alerts for secret modifications

---

## Advanced: External Secret Management

For production environments, consider using external secret managers:

### Sealed Secrets (Bitnami)

```bash
# Install sealed-secrets controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml

# Encrypt secret
kubeseal --format=yaml < secret.yaml > sealed-secret.yaml

# Commit sealed-secret.yaml to Git safely
kubectl apply -f sealed-secret.yaml
```

### HashiCorp Vault Integration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: vault-example
  annotations:
    vault.hashicorp.com/agent-inject: "true"
    vault.hashicorp.com/role: "myapp"
    vault.hashicorp.com/agent-inject-secret-database: "database/creds/db-app"
spec:
  containers:
    - name: app
      image: myapp:latest
```

### External Secrets Operator

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: app-secrets
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: app-secrets
  data:
    - secretKey: api-key
      remoteRef:
        key: prod/app/api-key
```

---

## Troubleshooting

### Secret not appearing in pod

**Check 1: Verify secret exists**

```bash
kubectl get secret news
kubectl describe secret news
```

**Check 2: Check pod events**

```bash
kubectl describe pod secret-datacenter
```

Look for errors like `Secret "news" not found`

**Check 3: Verify volume mount configuration**

```bash
kubectl get pod secret-datacenter -o yaml | grep -A 10 volumeMounts
```

### Permission denied when accessing mounted secret

**Check file permissions:**

```bash
kubectl exec secret-datacenter -- ls -la /opt/apps
```

**Set custom permissions (if needed):**

```yaml
volumes:
  - name: secret-volume
    secret:
      secretName: news
      defaultMode: 0400 # Read-only for owner
```

### Secret data appears as Base64 encoded

When using environment variables, Kubernetes automatically decodes Base64. For volume mounts, files are also decoded automatically.

If you see Base64 in the file, you might be viewing the secret definition, not the mounted file.

---

## Viewing Secret Contents

### View secret keys (not values)

```bash
kubectl get secret news -o yaml
```

### Decode secret value manually

```bash
kubectl get secret news -o jsonpath='{.data.news\.txt}' | base64 --decode
```

**On Windows PowerShell:**

```powershell
$secret = kubectl get secret news -o jsonpath='{.data.news\.txt}'
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($secret))
```

---

## Cleanup

```bash
kubectl delete pod secret-datacenter
kubectl delete secret news
```

---

## Key Takeaways

✅ Secrets store sensitive data in Base64-encoded format (not encrypted by default)
✅ Volume mounts are more secure than environment variables for secrets
✅ Secrets are namespace-scoped and require RBAC for access control
✅ Enable encryption at rest in production environments
✅ Use external secret managers (Vault, Sealed Secrets) for better security
✅ Mounted secret files are automatically decoded from Base64
✅ Secret volume mounts support atomic updates when secrets change
✅ Always use `readOnly: true` for secret volume mounts when possible
