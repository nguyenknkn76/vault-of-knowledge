---
tags:
  - aws
  - aws/eks
  - devops
---
## Resources

| No. | Resource | Name | IP  | Note |
| --- | -------- | ---- | --- | ---- |
|     |          |      |     |      |
|     |          |      |     |      |

## Create + Config EKS Cluster

1. Create EKS cluster
	- Cluster IAM role -> Create IAM role
		- Trusted entity
		- Add permissions
		- Name, review, and create
	- Node IAM role
	- VPC + Subnet ()
2. Create node group (template to create worker) 
	- Config
		- Permission Policies
		- IAM role
		- Launch template (define EC2 instance template)
	- Set compute and scaling 
		- AMI: e.g. Amazon Linux
		- Capacity: On-Demand
		- Types + Disk size
		- Node group scaling config: desired (2) + min (1) + max (3)
		- Node group update config: max unavailable + strategy (default)
	- Specify networking (don't need)

> [!note] Create EKS cluster
> - Should use private subnet + at least 2 subnets
> - Create EKS node group -> auto add EC2 instances as EKS nodes 
> - Networking of `Node group` don't need b/c just need access to `control plane` and using control plane to manage node.

> 
> - `on demand`: always open -> need availability -> can using scale down to reduce costs
> - `spot`: run one time 

## Setup Helm on EKS

