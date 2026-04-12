## Kubernenetes


### Docker:
A tool used to build and run containers. </br>
**Docker does the following:** </br>
- Runs containers
- You manage them manually
- Limited orchestration
- Runs everything with a single host focus.

### Minikube: 
A tool that lets you run Kubernetes locally on your computer. </br>
**Minikube does the following:** </br>
- Manages containers automatically
- Handles scaling
- Handles networking
- Handles failover
- Handles deployments
- Runs everything with Multi-node capability.


### Access Kubernetes API Server Using Service Account Token

This sequence is used to **authenticate directly to the Kubernetes API server** using a **ServiceAccount token**, instead of using `kubectl`.

---

## Commands

```bash
1. Get Kubernetes API Server Address
export APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')

2. Verify URL
echo "$APISERVER"

3. Generate Service Account Token
export TOKEN=$(kubectl create token default)

4. Verify Token Exists
echo ${#TOKEN}

5. Call Kubernetes API Directly
curl "$APISERVER" --header "Authorization: Bearer $TOKEN" --insecure
```


# Kubernetes Namespaces — KCNA Key Points

## What is a Namespace

- A **Namespace** is a virtual cluster inside Kubernetes
- Used to **separate resources** within the same cluster
- Resource names must be **unique within a namespace**, but can repeat across namespaces
- Used for **multi-tenancy** (multiple teams using same cluster)

---

## Default Namespaces

Kubernetes creates four namespaces by default:

### default
- Used for user workloads
- Resources are created here if no namespace is specified

### kube-system
- Contains Kubernetes system components
- Control plane and cluster services run here

### kube-public
- Public namespace
- Readable by anyone
- Used for non-sensitive cluster information

### kube-node-lease
- Stores node heartbeat information
- Used for node availability monitoring

---

## List Namespaces

```bash
kubectl get namespaces
```

# Kubernetes Pods — KCNA Key Points

## What is a Pod

- A **Pod** is the **smallest deployable unit** in Kubernetes  
- Represents a **single instance of an application**  
- A Pod contains **one or more containers**

---

## Pod Characteristics

Containers inside a Pod:

- Run on the **same node**
- Share the **same IP address**
- Share **network namespace**
- Share **storage volumes**
- Share **dependencies**

---

## Pods Are Ephemeral

- Pods are **temporary (ephemeral)**
- Pods **do not self-heal**
- Pods are usually managed by **controllers**

---

## Controllers That Manage Pods

Common controllers:

- Deployment  
- ReplicaSet  
- DaemonSet  
- Job  

Controllers provide:

- Self-healing  
- Scaling  
- Replication  
- Fault tolerance  

---

## Pod YAML Structure

Basic Pod definition fields:

- **apiVersion** → API version (v1 for Pods)
- **kind** → Object type (Pod)
- **metadata** → Name, labels, annotations
- **spec** → Pod configuration

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.22.1
```



