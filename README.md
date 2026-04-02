# Kubernetes Health Probes Lab: Deep Dive

This lab provides hands-on experience with Kubernetes health checks. You will learn how to ensure application availability and reliability using **Liveness**, **Readiness**, and **Startup** probes.

## Prerequisites
- A running Kubernetes cluster (e.g., Minikube, Kind, or a managed service).
- `kubectl` CLI tool configured.

---

## 💓 Part 1: Liveness Probes
Liveness probes tell Kubernetes when to **restart** a container. They are used to detect when an application is in a "broken" state (e.g., deadlock) and cannot recover on its own.

### Task 1: HTTP GET Method
Uses an HTTP request to determine health.
- **Manifest**: `liveness-http.yaml`
- **Simulation**: Break Nginx by deleting its `index.html`.
  ```bash
  kubectl exec liveness-http -- rm /usr/share/nginx/html/index.html
  ```
- **Result**: After 3 failed attempts (30s), the pod will restart.

### Task 2: Exec Command Method
Runs a command inside the container. Success is a zero exit code.
- **Manifest**: `liveness-exec.yaml`
- **Formula**: `time-to-restart = initialDelaySeconds + (failureThreshold × periodSeconds)`
- **Observation**: The container deletes `/tmp/healthy` after 30s. The probe then fails, and Kubernetes restarts the pod.

---

## 🚦 Part 2: Readiness Probes
Readiness probes tell Kubernetes when a container is **ready to receive traffic**. If a readiness probe fails, the pod is removed from the Service's endpoints (no traffic is sent), but the container is **not** restarted.

### Task 3: Remove from Service
- **Manifest**: `ready-demo.yaml` (Deployment + Service)
- **Simulation**: Break one of the two pods.
  ```bash
  # Find a pod name
  kubectl get pods -l app=ready-demo
  # Break it
  kubectl exec <pod-name> -- rm /usr/share/nginx/html/index.html
  ```
- **Result**: The pod's status changes to `0/1 READY`. Run `kubectl get endpoints ready-demo` to see it removed from the load balancer.

---

## 🛡️ Part 3: Startup Probes
Startup probes protect **slow-starting applications**. They disable Liveness and Readiness checks until the container has successfully started.

### Task 4: Slow-Starting App Protection
- **Manifest**: `slow-app.yaml`
- **Configuration**: Allows up to 300 seconds (30 attempts × 10s) for the app to start.
- **Benefit**: Prevents the liveness probe from killing the container before it has finished booting up.

---

## 🔧 Part 4: The Production Pattern
A production-ready pod should use all three probes in combination to ensure maximum reliability.

### Task 5: All-in-One Manifest
- **Manifest**: `production-app.yaml`
- **Ordering**:
  1. **Startup Probe** runs first.
  2. Once Startup passes, **Liveness** and **Readiness** probes begin their checks.

---

## Troubleshooting & Verification

| Command | Purpose |
| :--- | :--- |
| `kubectl get pods -w` | Watch pod status and restart counts in real-time. |
| `kubectl describe pod <name>` | View probe settings and recent failure events. |
| `kubectl get endpoints <service>` | Verify which pods are currently receiving traffic. |
| `kubectl logs <pod-name>` | Check application logs for probe-related errors. |
