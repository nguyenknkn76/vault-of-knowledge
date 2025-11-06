Setup K8s Cluster on AWS EC2
## Setup AWS

* Create VPC 
* Create Route Table 
* Create Subnet 
* Create Internet Gateway
* Create EC2 Instance
* Create NAT Gateway -> ur instances can be called from internet
* Create inbound rules
## Setup K8s Cluster
### Some ways to create K8s Cluster
- Using kubeadm
* Using kops
### Using kubeadm

- General: 	Setup system, user, docker, kubectl, kubeadm
- In master
	- run `kubectl init` -> master nodes
- In worker
	- run `kubectl ... join ...` -> join cluster




