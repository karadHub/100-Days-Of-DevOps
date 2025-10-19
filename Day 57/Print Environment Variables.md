# Day 57 — Print Environment Variables

**Scenario**: The Nautilus DevOps team is setting up prerequisites for an application that sends greetings to different users. A sample pod deployment needs to be tested with environment variables and a custom command.

---

## Task Requirements

Create a pod named `print-envars-greeting` with the following specifications:

- **Container name**: `print-env-container`
- **Image**: `bash`
- **Environment variables**:
  - `GREETING` = `Welcome to`
  - `COMPANY` = `DevOps`
  - `GROUP` = `Ltd`
- **Command**: `["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']`
- **Restart policy**: `Never` (to avoid crash loop backoff)

---

## Solution

### Option A — Declarative (YAML Manifest)

Create a file named `pod-greeting.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: print-envars-greeting
spec:
  containers:
    - name: print-env-container
      image: bash
      env:
        - name: GREETING
          value: "Welcome to"
        - name: COMPANY
          value: "DevOps"
        - name: GROUP
          value: "Ltd"
      command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
  restartPolicy: Never
```

**Apply the manifest**:

```bash
kubectl apply -f pod-greeting.yaml
```

### Option B — Imperative (kubectl run with overrides)

For a quick one-liner approach (useful for testing):

```bash
kubectl run print-envars-greeting \
  --image=bash \
  --restart=Never \
  --env="GREETING=Welcome to" \
  --env="COMPANY=DevOps" \
  --env="GROUP=Ltd" \
  --command -- /bin/sh -c 'echo "$(GREETING) $(COMPANY) $(GROUP)"'
```

---

## Verification

### 1. Check Pod Status

```bash
kubectl get pod print-envars-greeting
```

Expected output:

```
NAME                    READY   STATUS      RESTARTS   AGE
print-envars-greeting   0/1     Completed   0          10s
```

> **Note**: The status `Completed` is normal because `restartPolicy: Never` and the command exits immediately after printing.

### 2. View Logs

```bash
kubectl logs print-envars-greeting
```

**Expected output**:

```
Welcome to DevOps Ltd
```

Alternatively, use `-f` (follow) flag if the pod is still running:

```bash
kubectl logs -f print-envars-greeting
```

### 3. Describe Pod (Optional)

To see detailed pod configuration including environment variables:

```bash
kubectl describe pod print-envars-greeting
```

Look for the `Environment:` section:

```
Environment:
  GREETING:  Welcome to
  COMPANY:   DevOps
  GROUP:     Ltd
```

---

## Understanding the Configuration

### Environment Variables in Kubernetes

Environment variables are set using the `env` field in the container spec:

```yaml
env:
  - name: GREETING
    value: "Welcome to"
```

They can be referenced in the command using `$(VAR_NAME)` syntax.

### Command vs Args

- **`command`**: Overrides the container's ENTRYPOINT
- **`args`**: Provides arguments to the ENTRYPOINT

In this case, we use `command` to run a shell command that echoes the concatenated environment variables.

### Restart Policy

- **`Never`**: Pod won't restart after completion (suitable for one-time tasks)
- **`OnFailure`**: Restart only if container exits with error
- **`Always`**: Always restart (default for Deployments)

For Jobs and one-time tasks, `Never` or `OnFailure` prevents unnecessary crash loops.

---

## Troubleshooting

### Pod stuck in CrashLoopBackOff

If you see this status, it means:

- The pod is restarting repeatedly (default `restartPolicy: Always`)
- **Fix**: Ensure `restartPolicy: Never` is set

### No output in logs

- Check pod status: `kubectl get pod print-envars-greeting`
- If status is `ImagePullBackOff`, the `bash` image may not be available
- **Alternative images**: `busybox`, `alpine`, or `ubuntu`

### Environment variables not expanding

- Ensure you use **single quotes** in YAML for the command to prevent shell expansion:
  ```yaml
  command: ["/bin/sh", "-c", 'echo "$(GREETING) $(COMPANY) $(GROUP)"']
  ```
- Double quotes would try to expand variables at manifest-parsing time (which would fail)

---

## Cleanup

```bash
kubectl delete pod print-envars-greeting
```

Or if you used the YAML file:

```bash
kubectl delete -f pod-greeting.yaml
```

---

## Key Takeaways

✅ Environment variables in Kubernetes are defined using `env` field
✅ Variables can be referenced in commands with `$(VAR_NAME)` syntax
✅ `restartPolicy: Never` prevents pods from restarting after completion
✅ Completed pods remain visible until explicitly deleted (useful for debugging)
✅ For production workloads, consider using ConfigMaps or Secrets instead of hardcoded env values
