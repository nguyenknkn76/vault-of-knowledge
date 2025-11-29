- Podman
- Minikube
- Kind
## Minikube vs Kind

- Minikube: tool to run 1 cluster K8s with single node in VM or Container Runtime (like Docker)
- Kind (K8s in Docker/Podman): run K8s cluster by using containers via "node". Using to testing CI/CD quickly

### Example: Install and Init in Fedora 42

1. Install tools (Podman/Docker and Kubectl)
```bash
# install podman
sudo dnf install -y podman kubectl

# install minikube (install binary and mv to PATH)
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64  
sudo install minikube-linux-amd64 /usr/local/bin/minikube  
rm minikube-linux-amd64**
```

2. Init Cluster with Podman Driver
```bash
# init with podman
minikube start --driver=podman
# init with docker
minikube start --driver=docker
```

3. Check Cluster
```bash
kubectl cluster-info
kubectl get nodes
```

4. some minikube command 
```bash
# stop cluster
minikube stop
# delete cluster
minikube delete
```
