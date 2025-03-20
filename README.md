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

### Check Kubernetes Version
Check the client and server version of Kubernetes.
```bash
kubectl version
```

### Get Help
List all available Kubernetes commands.
```bash
kubectl --help
```

### List All Nodes
Outputs the Name, Status, Roles, Age, and Version of all nodes.
```bash
kubectl get nodes
```

### List Nodes with Additional Details
Displays extra information such as internal port, external port, OS image, kernel version, etc.
```bash
kubectl get nodes -o wide
```

### Create a Pod Imperatively
Creates a pod named `nginx` using the nginx image from Docker Hub.
```bash
kubectl run nginx --image=nginx
```

### List All Pods
Displays all available pods with details.
```bash
kubectl get pods
```

---

## Kubernetes Object Fields

### 1. `apiVersion`
Specifies the API version to be used for the Kubernetes object.
- **Pod:** `v1`
- **Service:** `v1`
- **ReplicaSet:** `apps/v1`
- **Deployment:** `apps/v1`

```yaml
apiVersion: v1  # Correct API version for Pod
```

### 2. `kind`
Defines the type of Kubernetes object to be created. It is case-sensitive.
- Example values: `Pod`, `Service`, `ReplicaSet`, `Deployment`

```yaml
kind: Pod  # Specifies the type of object
```

### 3. `metadata`
Provides metadata about the object, such as its name and labels.

```yaml
metadata:
  name: myapp-pod  # Name of the pod
  labels:
    app: myapp  # Labels for the pod
```

### 4. `spec`
Specifies the desired state of the object, such as container details.

```yaml
spec:
  containers:
    - name: nginx-container
      image: nginx
```

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
---

### Commands for Managing Pods

### Create a Pod from YAML
Creates a pod using the provided YAML configuration.
```bash
kubectl create -f pod-definition.yaml
```

### Apply Changes to a Pod
Creates or updates a pod with the provided YAML definition.
```bash
kubectl apply -f pod-definition.yaml
```

### List All Pods
Displays all available pods with details.
```bash
kubectl get pods
```

### Describe a Pod
Provides detailed information about a specific pod, including its name, namespace, container image, state, conditions, and events.
```bash
kubectl describe pod myapp-pod
```

---

## Difference Between `kubectl create` and `kubectl apply`

| Command | Description |
|---------|-------------|
| `kubectl create -f pod.yaml` | Creates the resource defined in `pod.yaml`. Fails if the resource already exists. Best for one-time creation. |
| `kubectl apply -f pod.yaml` | Creates or updates the resource declaratively. If the resource exists, only changes are applied. Preferred for managing configurations over time. |

---

## Deletion Process

### Delete a Pod Using a YAML Definition
```bash
kubectl delete -f pod.yaml
```

### Delete a Pod with Minimal Delay
```bash
kubectl delete pod foo --now
```

### Force Delete a Pod on a Dead Node
```bash
kubectl delete pod foo --force
```

### Delete All Pods
```bash
kubectl delete pods --all
```

---

## REPLICASET

An application is running on a pod, and if the application crashes and pods go down, users will lose access. To prevent this, we need **ReplicaSet**. ReplicaSet **creates, manages, and monitors pods**. If any pods go down, ReplicaSet brings them back, ensuring the desired and current state always match. Kube-Scheduler assigns which pod will be created in which node, but ReplicaSet can create pods dynamically in any cluster or node to manage load.

### ReplicaSet Definition (`replicaset-definition.yaml`)

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec: 
  template:
    metadata: 
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: front-end
```

### Commands

#### Create a ReplicaSet
```bash
kubectl create -f replicaset-definition.yaml
```

#### Check the ReplicaSet
```bash
kubectl get replicaset
```

#### Delete the ReplicaSet
```bash
kubectl delete replicaset myapp-replicaset
```

Now, inside `spec` of ReplicaSet, we see the entire pod definition. This is because we are creating a ReplicaSet for pods. `replicas` stands for `the desired number of pods`. It can be any number depending on requirements. `selector` and `matchLabels` help ReplicaSet `monitor and manage specific pods`.

### Scaling in ReplicaSet

#### Scenario: Increase Replicas from 3 to 6
We have two ways to do this:

1. **Using Scale Command** _(Not Recommended)_
   ```bash
   kubectl scale --replicas=6 -f replicaset-definition.yaml
   ```
   This increases the replicas to 6, but the main config file still shows 3. This is not the best approach.

2. **Updating the YAML File and Replacing ReplicaSet** _(Correct Approach)_
   - First, update `replicas` in `replicaset-definition.yaml` to `6`.
   - Then, run the following command:
   ```bash
   kubectl replace -f replicaset-definition.yaml
   ```

   ---

## Conclusion

This document serves as a basic guide to Kubernetes architecture, commands, and pod management. Keep updating it as you explore more Kubernetes features!
