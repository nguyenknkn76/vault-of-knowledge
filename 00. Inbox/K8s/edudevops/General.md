## K8s
- What is K8s? What is K8s used for?

![[file-20251102224402080.png]]
- When should K8s be used? big project, microservices, multi env, scaling, self healing.

## K8s Architecture

![[file-20251102224951645.png]]

- Control Plane (Master Node)
	- cloud control manager
	- kube api server
	- etcd: save status and config of cluster
	- schduler: decide Node for Pod running base on resources, limitation and policy
	- controller manager: monitor status of cluster
- Worker Node
	- kubelet: receive request from `kube api server` to deploy `pod` on `node`
	- kube proxy: network for communication between pods and external 
	- pod:  atomic unit in k8s cluster, can have many container

> [!note] on premise, don't have cloud control manager

Example: 
![[file-20251102225357251.png]]

![[file-20251102225520403.png]]

