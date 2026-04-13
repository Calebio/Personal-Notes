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

# Kubernetes Labels — KCNA Key Points

## What are Labels

- **Labels** are key-value pairs attached to Kubernetes objects
- Used to **organize and group resources**
- Labels do **not provide uniqueness**

---

## Objects That Use Labels

Labels can be attached to:

- Pods  
- ReplicaSets  
- Deployments  
- Nodes  
- Namespaces  
- Persistent Volumes  

---

## Purpose of Labels

Labels are used to:

- Group objects logically  
- Select resources  
- Filter objects  
- Connect controllers to Pods  

Controllers use labels instead of names or IDs.

---

## Example Labels

```yaml
labels:
  app: frontend
  env: dev
```

# Kubernetes Label Selectors — KCNA Key Points

## What are Label Selectors

- **Label Selectors** are used to **filter and select Kubernetes objects**
- Used by:
  - Controllers (Deployments, ReplicaSets, etc.)
  - Services

---

## Types of Label Selectors

### 1. Equality-Based Selectors

Used to match labels using:

- `=`
- `==`
- `!=`

### Examples

Select objects with:

```bash
env=dev
```

# Kubernetes Deployments — KCNA Notes

## What is a Deployment

- A **Deployment** manages Pods and ReplicaSets
- Provides **declarative updates**
- Ensures **desired state = actual state**
- Supports **rolling updates** and **rollbacks**
- Handles **scaling automatically via ReplicaSets**
- Managed by **Deployment Controller**

---

## Why Use Deployments

Deployments provide:

- Rolling updates  
- Rollbacks  
- Self-healing  
- Scaling  
- Version control (Revisions)

---

## Deployment Architecture

- Deployment manages ReplicaSets  
- ReplicaSets manage Pods  
- Pods run containers  

---

## Update Strategies

### RollingUpdate (Default)
- Gradually replaces old Pods
- No downtime

### Recreate
- Deletes all old Pods
- Creates new Pods
- Causes downtime

---

## Example Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - name: nginx
        image: nginx:1.20.2
        ports:
        - containerPort: 80
```
```bash
$ kubectl apply -f nginx-deploy.yaml --record
$ kubectl get deployments $ kubectl get deploy -o wide
$ kubectl scale deploy nginx-deployment --replicas=4
$ kubectl get deploy nginx-deployment -o yaml
$ kubectl get deploy nginx-deployment -o json
$ kubectl describe deploy nginx-deployment $ kubectl rollout status deploy nginx-deployment
$ kubectl rollout history deploy nginx-deployment
$ kubectl rollout history deploy nginx-deployment --revision=1 $ kubectl set image deploy nginx-deployment nginx=nginx:1.21.5 --record
$ kubectl rollout history deploy nginx-deployment --revision=2
$ kubectl rollout undo deploy nginx-deployment --to-revision=1 $ kubectl get all -l app=nginx -o wide
$ kubectl delete deploy nginx-deployment 
```

# Kubernetes DaemonSets — KCNA Notes

## What is a DaemonSet

- A **DaemonSet** ensures **one Pod runs on every Node**
- Used to run **node-level services**
- Automatically runs Pods on:
  - All nodes
  - Selected nodes

---

## When to Use DaemonSets

DaemonSets are commonly used for:

- Monitoring agents
- Logging agents
- Networking agents
- Storage daemons
- Security agents

### Common Examples

- kube-proxy
- Fluentd (logging)
- Calico / Cilium (networking)

---

## How DaemonSets Work

- One Pod per Node
- New node added → Pod automatically created
- Node removed → Pod deleted
- Delete DaemonSet → All Pods deleted

---

## DaemonSet vs Deployment vs ReplicaSet

| Feature | DaemonSet | Deployment | ReplicaSet |
|---------|-----------|------------|------------|
| One Pod per Node | ✅ | ❌ | ❌ |
| Scaling | Automatic per node | Manual / Auto | Manual |
| Rolling Updates | ✅ | ✅ | ❌ |
| Self Healing | ✅ | ✅ | ✅ |

---

## Example DaemonSet YAML

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-agent
  namespace: default
  labels:
    k8s-app: fluentd-agent
spec:
  selector:
    matchLabels:
      k8s-app: fluentd-agent
  template:
    metadata:
      labels:
        k8s-app: fluentd-agent
    spec:
      containers:
      - name: fluentd
        image: quay.io/fluentd_elasticsearch/fluentd:v4.5.2
```

# Kubernetes Services — KCNA Notes

## What is a Service

A **Service** is a Kubernetes object used to **expose applications running in Pods** and enable **communication between applications**.

Pods are **not directly accessible** because:

- Pods are ephemeral (they can be recreated)
- Pod IP addresses change
- Containers do not expose ports automatically

A **Service solves this problem** by providing:

- Stable IP address
- Stable DNS name
- Load balancing across Pods

---

## Why Services are Needed

Without Services:

- Pods cannot be easily discovered
- Pod IPs change frequently
- No built-in load balancing

With Services:

- Applications can communicate reliably
- Traffic is load balanced across Pods
- Applications become discoverable

---

## How Services Work

Services use:

- kube-proxy
- iptables / routing rules
- Cluster DNS
- Load balancing

These components together expose Pods to:

- Other Pods
- Cluster network
- External clients (optional)

---

## Service Benefits

- Stable IP address
- DNS-based service discovery
- Load balancing
- Decouples Pods from clients
- Works with multiple replicas

---

## Example Use Case

Multiple Pods running the same app:
- Pod 1
- Pod 2
- Pod 3


## Key Concept

Pods change  
Services stay stable

---

## KCNA Key Points

- Services expose Pods
- Provide stable IP and DNS
- Enable service discovery
- Load balance traffic across Pods
- Work with multi-replica applications
- Recommended way to expose applications

