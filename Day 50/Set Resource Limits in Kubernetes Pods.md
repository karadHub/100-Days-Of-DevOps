# Day 50: Set Resource Limits in Kubernetes Pods

## 🛠️ Task

The Nautilus DevOps team has noticed performance issues in some Kubernetes-hosted applications due to resource constraints. To address this, they plan to set limits on resource utilization.

Create a pod named `httpd-pod` with a container named `httpd-container` using image `httpd:latest` and set the following resource limits:

- Requests: Memory: 15Mi, CPU: 100m
- Limits: Memory: 20Mi, CPU: 100m

---

## ✅ Solution (Pod Manifest)

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: httpd-pod
spec:
	containers:
	- name: httpd-container
		image: httpd:latest
		resources:
			requests:
				memory: "15Mi"
				cpu: "100m"
			limits:
				memory: "20Mi"
				cpu: "100m"
```

---

## 🔍 Breakdown

- Pod name: `httpd-pod`
- Container name: `httpd-container`
- Image: `httpd:latest`
- Resource Requests: minimum guaranteed (15Mi memory, 100m CPU)
- Resource Limits: maximum allowed (20Mi memory, 100m CPU)

---

## 🚀 Apply and Verify

```bash
kubectl apply -f httpd-pod.yaml

# Verify pod and resources
kubectl get pod httpd-pod -o wide
kubectl describe pod httpd-pod | findstr -i -e "Limits" -e "Requests" -e "Image"
```

---

## 📘 Notes

- CPU units: `100m` equals 0.1 CPU (100 millicores)
- Memory units: `Mi` stands for mebibytes (base 1024)
- Requests are used by the scheduler; limits are enforced by the runtime
