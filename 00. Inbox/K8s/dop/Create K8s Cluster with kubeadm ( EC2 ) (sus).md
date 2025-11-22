---
tags:
  - k8s
  - devops
  - aws
  - aws/ec2
---
## 0. Resource 

| No. | Resource         | Name                            | IP  | Note        |
| --- | ---------------- | ------------------------------- | --- | ----------- |
| 01  | EC2              | dop10-k8s-cluster-ec2-master    |     | `t3.medium` |
|     |                  | dop10-k8s-cluster-ec2-worker-01 |     | `t3.small`  |
|     |                  | dop10-k8s-cluster-ec2-worker-02 |     | `t3.small`  |
| 02  | VPC              | dop10-k8s-cluster-vpc           |     |             |
| 03  | Route Table      | dop10-k8s-cluster-rtb-public    |     |             |
|     |                  | dop10-k8s-cluster-rtb-private   |     |             |
| 04  | Subnet           | dop10-k8s-cluster-sn-public     |     |             |
|     |                  | dop10-k8s-cluster-sn-private    |     |             |
| 05  | Internet Gateway | dop10-k8s-cluster-igw           |     |             |
| 06  | Key Pair         | dop10-k8s-cluster-kp-master     |     |             |
|     |                  | dop10-k8s-cluster-kp-worker     |     |             |
| 07  | Security Group   | dop10-k8s-cluster-sg-master     |     |             |
|     |                  | dop10-k8s-cluster-sg-worker     |     |             |
| 08  | NAT Gateway      | dop10-k8s-cluster-nat-private   |     |             |
| 09  | ECR              | shopsquare/frontend             |     |             |

## 1. Setup Environment

1. Create VPC
2. Create Subnet
3. Create Route Table + association Subnet
4. Create Internet Gateway + config Route Table
5. Create EC2 Instances + Security Group + Key Pair
6. Create NAT Gateway (for worker) + config Route Table
7. Create ECR
## 2. Setup Master Node

```bash 
# master
ssh -i dop10-k8s-cluster-kp-master.pem ubuntu@3.96.192.253
ping <worker-private-ip>

# create cluster by use kubeadm
# install docker
sudo apt install ...

# install kubeadm
sudo apt install ...

kubectl version
kubeadm version

# init cluster
sudo kubeadm init ...

kubectl get nodes
```

> [!note] Setup Security Group for Worker
> - E.g. (Inbound Rules of Master Node) Source -> choose Security Group
## 3. Setup Worker Nodes

```bash
# worker
# join cluster
sudo kubeadm join ...

kubectl get nodes
kubectl get nodes -o wide
```

## 4. 

```bash
aws configure
aws s3 ls

# master
# install aws cli
...
...

# authenticate
aws configure
# access key id:  
# secret access key id:

kubectl create secret docker-registry ...
kubectl apply -f deployment.yml

# troubleshot command
kubectl get pods
kubectl get all
kubectl describe ...
kubectl logs -f ...

# build image + push image to registry
docker build -t quickbite/frontend .
dokcer tag ...
docker push ...
```
## 

Deployment K8s File Structure:
- Deployement
	- deployment.yaml
	- service.yaml
	- config.yaml
	- secret.yaml

```yaml
# deployment.yaml
# using imagePullSecrets: ... to fetch authentication info from secret.yaml

# service.yaml
# using type: nodePort to expose application to the internet 

# secret.yaml
```


> [!note]
> Only working on **master** node

## 5. Apply K8s Deployment File (.yaml)

### Method 1: In control plane 

```bash
# method 1: in control plane
scp ... # cp file deployment.yaml from ur device to control plane
kubectl apply -f deployment.yaml

kubectl get pods
kubectl get all
kubectl describe ...
kubectl logs ...

scp ... # service.yaml
kubectl apply -f service.yaml
kubectl get all

# kubectl create secret docker-registry ...
# aws ecr get-login-password --region ...
```

```bash
ls -la 
cd .kube
cat config
scp ... # scp file config
```
### Method 2: In personal device

- Copy config file from control plane to ur device
```bash 
ls -la 
cd .kube
cat config
scp ... # scp file config
```

- Apply kubeconfig by pasting > have to change IP in kubeconfig to public IP

> [!note]
> Have to change IP in kubeconfig to public IP of control plane

> [!tip]
> Using **Ranger**, **Lens**, or **K9s** UI to manage K8s cluster 