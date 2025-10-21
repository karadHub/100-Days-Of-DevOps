# Day 58 — Deploy Grafana on Kubernetes Cluster

Goal: Deploy Grafana in Kubernetes using a Deployment and expose it via a NodePort Service on 32000 so the Grafana login page is reachable from your machine.

---

## Prerequisites

- A running Kubernetes cluster and kubectl configured
- NodePort range open (default: 30000–32767)

---

## Option A — Declarative (recommended)

Create a single file named `grafana.yaml` containing both the Deployment and the Service:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
	name: grafana-deployment-xfusion
	labels:
		app: grafana
spec:
	replicas: 1
	selector:
		matchLabels:
			app: grafana
	template:
		metadata:
			labels:
				app: grafana
		spec:
			containers:
				- name: grafana
					image: grafana/grafana:latest
					ports:
						- containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
	name: grafana-service
	labels:
		app: grafana
spec:
	type: NodePort
	selector:
		app: grafana
	ports:
		- name: http
			port: 3000
			targetPort: 3000
			nodePort: 32000
```

Apply and verify:

```bash
kubectl apply -f grafana.yaml
kubectl get deploy grafana-deployment-xfusion
kubectl get pods -l app=grafana -o wide
kubectl get svc grafana-service -o wide
```

Access in your browser:

```
http://<NodeIP>:32000
```

You should see the Grafana login page.

---

## Option B — Split files (Deployment and Service)

If you prefer separate files, use:

`grafana-deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
	name: grafana-deployment-xfusion
	labels:
		app: grafana
spec:
	replicas: 1
	selector:
		matchLabels:
			app: grafana
	template:
		metadata:
			labels:
				app: grafana
		spec:
			containers:
				- name: grafana
					image: grafana/grafana:latest
					ports:
						- containerPort: 3000
```

`grafana-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
	name: grafana-service
spec:
	type: NodePort
	selector:
		app: grafana
	ports:
		- port: 3000
			targetPort: 3000
			nodePort: 32000
```

Apply both:

```bash
kubectl apply -f grafana-deployment.yaml
kubectl apply -f grafana-service.yaml
```

---

## Verification and Tips

- Wait for the pod to be Ready:
  ```bash
  kubectl rollout status deploy/grafana-deployment-xfusion
  kubectl get pods -l app=grafana
  ```
- Find a node IP and open in browser: `http://<NodeIP>:32000`
- Default Grafana credentials (if prompted): username `admin`, password `admin` (you can change later; no internal configuration required for this task).

If you use minikube you can also run:

```bash
minikube service grafana-service --url
```

---

## Cleanup

```bash
kubectl delete -f grafana.yaml
# or, if applied separately
kubectl delete -f grafana-deployment.yaml -f grafana-service.yaml
```

---

## Notes

- NodePort 32000 must be available in your cluster’s NodePort range (default 30000–32767).
- For kind or Docker Desktop Kubernetes, NodePort may not be directly reachable from the host; consider `kubectl port-forward svc/grafana-service 3000:3000` and browse to http://localhost:3000.
