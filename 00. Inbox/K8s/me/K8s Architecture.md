## General 
- Control Plane (aka Master Node)
	- etcd: distributed key value storage, single source of truth, save status and config of cluster
	- Kube-API Server: 
		- resolve REST requests, authenticate, authorization, and save status to `etcd` 
		- Another components (Client, Scheduler, Controller) communicate via API Server
	- Kube-Scheduler: Manage Pod that NOT be assigned Node -> decide Node for Pod running base on resources, limitation and policy.
	- Kube-Controller Manager: Daemon - Core control loop. -> follows status of cluster and change it to match with desired state
- Worker Node
	- Kubelet: run in each node -> make sure container in Pod is running and healthy
	- Kube-Proxy: communicate with Pods & Services. like internal load balancer
	- Container Runtime: (like: Podman, Containerd)

- **Control loop**