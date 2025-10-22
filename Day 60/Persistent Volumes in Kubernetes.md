# Day 60 — Persistent Volumes in Kubernetes

Goal: Provision a PersistentVolume and PersistentVolumeClaim, run an NGINX pod that mounts the claim at the web root, and expose it via a NodePort Service at 30008.

---

## What you’ll create

- PersistentVolume `pv-devops` (hostPath /mnt/dba, 4Gi, ReadWriteOnce, storageClass `manual`)
- PersistentVolumeClaim `pvc-devops` (1Gi, ReadWriteOnce, storageClass `manual`)
- Pod `pod-devops` with container `container-devops` using `nginx:latest`, mounting the PVC at `/usr/share/nginx/html`
- Service `web-devops` of type NodePort, nodePort `30008`

---

## Manifests

You can keep these as separate files or combine them into one. Below are split files for clarity.

### 1) PersistentVolume — pv-devops (hostPath)

File: `pv-devops.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
	name: pv-devops
spec:
	storageClassName: manual
	capacity:
		storage: 4Gi
	accessModes:
		- ReadWriteOnce
	hostPath:
		path: /mnt/dba
```

### 2) PersistentVolumeClaim — pvc-devops

File: `pvc-devops.yaml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
	name: pvc-devops
spec:
	storageClassName: manual
	accessModes:
		- ReadWriteOnce
	resources:
		requests:
			storage: 1Gi
```

### 3) Pod — pod-devops (nginx:latest) mounting the PVC at web root

File: `pod-devops.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: pod-devops
	labels:
		app: pod-devops
spec:
	containers:
		- name: container-devops
			image: nginx:latest
			volumeMounts:
				- mountPath: /usr/share/nginx/html
					name: devops-storage
	volumes:
		- name: devops-storage
			persistentVolumeClaim:
				claimName: pvc-devops
```

### 4) Service — web-devops (NodePort 30008)

File: `web-devops.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
	name: web-devops
spec:
	type: NodePort
	selector:
		app: pod-devops
	ports:
		- port: 80
			targetPort: 80
			nodePort: 30008
```

---

## Apply and verify

```bash
kubectl apply -f pv-devops.yaml
kubectl apply -f pvc-devops.yaml
kubectl apply -f pod-devops.yaml
kubectl apply -f web-devops.yaml

# Wait for the pod to be ready
kubectl get pod pod-devops -w

# Check PV/PVC binding
kubectl get pv pv-devops
kubectl get pvc pvc-devops

# Check service
kubectl get svc web-devops -o wide
```

Optionally, create a test index.html inside the mounted path and then browse:

```bash
kubectl exec pod-devops -- /bin/sh -c 'echo "<h1>Hello from DevOps</h1>" > /usr/share/nginx/html/index.html'
```

Open in browser:

```
http://<NodeIP>:30008
```

You should see the NGINX welcome page or your custom index.html if you created one.

---

## Notes & Tips

- hostPath PVs are node-specific and intended for local clusters or development. For production, use a proper StorageClass (e.g., CSI drivers, cloud volumes).
- The PV’s `storageClassName` and the PVC’s must match (`manual`).
- Access mode `ReadWriteOnce` allows the volume to be mounted by a single node at a time.
- Ensure NodePort 30008 is within your cluster’s NodePort range (default 30000–32767).

---

## Cleanup

```bash
kubectl delete -f web-devops.yaml
kubectl delete -f pod-devops.yaml
kubectl delete -f pvc-devops.yaml
kubectl delete -f pv-devops.yaml
```
