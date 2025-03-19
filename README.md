# Kubernetes Notes

## K8S Basics

Docker is a container creation platform, while Kubernetes is a container orchestration platform.

Kubernetes solves four key challenges that Docker alone struggles with:
1. **Single Host Limitation**
2. **Auto-Scaling**
3. **Auto-Healing**
4. **Enterprise Support**

---

## Kubernetes Architecture

### Worker Nodes / Data Plane
These nodes are responsible for running applications and handling workloads. The key components are:

1. **Container Runtime** – Like Docker Engine, it runs the actual containers. Kubernetes supports multiple runtimes like Docker, containerd, and CRI-O.
2. **PODs** – The smallest deployable unit in Kubernetes. Each POD can contain one or more containers that share the same storage and network.
3. **Kubelet** – A critical component that ensures the Pods are running and healthy. It continuously communicates with the Control Plane to report the node’s status.
4. **Kube-Proxy** – Unlike Docker’s bridge networking, Kube-Proxy manages network communication, assigns IP addresses, and ensures proper inter-Pod connectivity.

### Master Node / Control Plane
This is the brain of the Kubernetes cluster, responsible for managing the Worker Nodes. The key components are:

1. **API Server** – The heart of Kubernetes. It handles all requests from users and internal components, deciding where and when new Pods should be created.
2. **Scheduler** – Assigns new workloads (Pods) to Worker Nodes based on available resources and constraints.
3. **ETCD** – The database of Kubernetes. It stores cluster state and configurations in a key-value format, acting like a backup (similar to terraform.tfstate.backup).
4. **Controller Manager** – Ensures the cluster is always in the desired state. It monitors if Pods are running as expected and takes action when needed (e.g., auto-scaling).
5. **Cloud Controller Manager (CCM)** – Allows cloud providers to integrate their own services (e.g., AWS EKS, GCP GKE, Azure AKS) with Kubernetes.

---

## Common Kubernetes Commands

```bash
kubectl version  # Client and Server version related to Kubernetes
kubectl --help  # List of all commands
kubectl get nodes  # Outputs the Name, Status, Roles, Age, and Version of all nodes
kubectl get nodes -o wide  # Outputs additional details like internal port, external port, OS image, etc.
kubectl run nginx --image=nginx  # Create a pod with the name nginx using the nginx image from Docker Hub
kubectl get pods  # List all the pods available with details
```

---

## POD: Basic Configuration

A basic pod definition in declarative format (`pod-definition.yaml`) must include:

```yaml
apiVersion: v1  # Correct API version for Pod
kind: Pod  # Specifies the type of object
metadata:
  name: myapp-pod  # Name of the pod
  labels:
    app: myapp  # Labels for the pod
spec:
  containers:
    - name: nginx-container
      image: nginx
```

### Commands for Managing Pods

```bash
kubectl create -f pod-definition.yaml  # Create pod from the YAML definition
kubectl apply -f pod-definition.yaml  # Create/update pod with YAML definition
kubectl get pods  # List all pods
kubectl describe pod myapp-pod  # Detailed information about the pod
```

---

## Difference Between `kubectl create` and `kubectl apply`

| Command | Description |
|---------|-------------|
| `kubectl create -f pod.yaml` | Creates the resource defined in `pod.yaml`. Fails if the resource already exists. Best for one-time creation. |
| `kubectl apply -f pod.yaml` | Creates or updates the resource declaratively. If the resource exists, only changes are applied. Preferred for managing configurations over time. |

---

## Conclusion

This document serves as a basic guide to Kubernetes architecture, commands, and pod management. Keep updating it as you explore more Kubernetes features!
