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


# Access Kubernetes API Server Using Service Account Token

This sequence is used to **authenticate directly to the Kubernetes API server** using a **ServiceAccount token**, instead of using `kubectl`.

---

## Commands

```bash
export APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
echo "$APISERVER"

export TOKEN=$(kubectl create token default)
echo ${#TOKEN}

curl "$APISERVER" --header "Authorization: Bearer $TOKEN" --insecure
```


