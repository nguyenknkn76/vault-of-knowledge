---
tags:
  - aws/ec2
  - aws
  - aws/ecr
  - "#devops"
  - k8s
---
#aws/ecr
## 1. The Cargo Preparation

### 1.1. Initial React App

```bash
npm create vite@latest my-react-app
cd my-react-app
```

```sh

```

### 1.2. Build Dockerfile

```Dockerfile
# build stage
FROM node:21-alpine as build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . . 
RUN npm run build

# run stage
FROM nginx:1.25-alpine
RUN rm -rf /usr/share/nginx/html/*
COPY --from=build /app/build /usr/share/nginx/html  
EXPOSE 80

# ENTRYPOINT ["npm run dev"]
CMD ["nginx", "-g", "daemon off;"]
```

## 2. The Cloud Depot and Registry

### 2.1. Configure AWS CLI and Create Repository

```bash
aws configure

# config
export AWS_REGION="ap-northeast-2"
export REPO_NAME="k8s-repo"

printenv "AWS_REGION"

aws ecr create-repository \
    --repository-name $REPO_NAME \
    --region $AWS_REGION \
    --image-scanning-configuration scanOnPush=true \
    --query 'repository.repositoryUri'
```

### 2.2. Build, Tag and Push Image to ECR

#### Build Local Image and Authenticate

```bash
# Build image (using Dockerfile in the k8s-react-app directory)
docker build -t $REPO_NAME:latest .

# Get Account ID and determine ECR URI
export ACCOUNT_ID=$(aws sts get-caller-identity --query "Account" --output text)
export ECR_URI="$ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME"

# Authentication and login command
aws ecr get-login-password --region $AWS_REGION | \
    docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
```

#### Tag and Push Image

```bash
export IMAGE_TAG="v1.0.0"

docker tag $REPO_NAME:latest $ECR_URI:$IMAGE_TAG
docker push $ECR_URI:$IMAGE_TAG

echo "Image successfully pushed to: $ECR_URI:$IMAGE_TAG"
```

## 3. Manage K8s Cluster Initialization on EC2 (using Kubeadm)

| **Node Role**          | **Minimum Configuration (RAM/vCPU)** | **Required Open Ports (TCP Inbound)**                                                      | **Purpose**                            |
| ---------------------- | ------------------------------------ | ------------------------------------------------------------------------------------------ | -------------------------------------- |
| Master (Control Plane) | 2GB RAM / 2 vCPU (t3.small)          | 22, 6443, 2379-2380 (etcd), 10250 (Kubelet), 10259 (Scheduler), 10257 (Controller Manager) | API communication and Cluster Control. |
| Worker (Data Plane)    | 1GB RAM / 1 vCPU (t3.micro)          | 22, 10250 (Kubelet), **30000-32767 (NodePort)**                                            | Running Pods and Exposing Services.    |

> [!note]
> - Ensure that the Security Group (SG) applied to these EC2 instances has the necessary ports opened. Specifically, the port range **30000-32767** must be open for NodePort Services. This is crucial; otherwise, the NLB will not be able to send traffic to the Worker Nodes. 
> - Record the Private IP address of the Master Node (e.g., `10.0.1.10`).

### 3.1. Disable Swap and Configure Kernel

```bash
# Temporarily disable swap
sudo swapoff -a
# Permanently disable swap in /etc/fstab
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Load necessary kernel modules
sudo modprobe overlay
sudo modprobe br_netfilter

# Sysctl Configuration: Enable IP Forwarding and Bridge Filtering
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system # Apply immediately
```

### 3.2. Install Container Runtime (Containerd)

```bash
# Install Containerd
sudo apt update
sudo apt install -y containerd

# Configure and Enable systemd cgroup driver
sudo containerd config default | sudo tee /etc/containerd/config.toml
# Edit: Change SystemdCgroup = false to true
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd
```

### 3.3. Install Kubelet, Kubeadm, Kubectl

```bash
# Install required tools
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl

# Add K8s GPG Key and Repository [9, 10]
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install K8s components
sudo apt update

sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

sudo apt install -y kubelet kubeadm kubectl

# curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubelet"
# chmod +x kubelet && sudo mv kubelet /usr/bin/

kubelet --version
kubectl version
kubeadm version

sudo apt-mark hold kubelet kubeadm kubectl
```

### 3.4. Initial the Control Plane (on Master Node)

#### Cluster Initialization
```bash
MASTER_IP="10.0.1.10" # Replace with the actual Private IP

sudo kubeadm init \
    --apiserver-advertise-address=$MASTER_IP \
    --pod-network-cidr=192.168.0.0/16 \
    --upload-certs
```

#### Set up Kubectl and Save the Join Command
```bash
# Set up kubectl (On Master Node only)
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# CHECK (Pods will be Pending without CNI)
kubectl get nodes

# SAVE THE KUBEADM JOIN COMMAND PRINTED BY THE INIT COMMAND (Example below)
# kubeadm join 10.0.1.10:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

###  3.5. Install the Container Network Interface (CNI)

**ONLY ON THE MASTER NODE**
```bash
# Install Calico (Manifest version v3.28.3)
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.3/manifests/calico.yaml

# Monitor status (wait for CoreDNS and Calico Pods to transition to Running)
kubectl get pods -n kube-system
```


#### 3.6. Join Worker Nodes

**ON ALL WORKER NODES**
```bash
# Use the join command saved in step 3.4.2
sudo kubeadm join 10.0.1.10:6443 --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash>

# Verify cluster status
kubectl get nodes
# All Nodes should transition to the "Ready" status
```


