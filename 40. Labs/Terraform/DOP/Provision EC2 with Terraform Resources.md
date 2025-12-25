---
tags:
  - devops
  - iac
  - terraform
  - aws/ec2
---
## Requirements 

![[assets/Provision EC2 with Terraform Resources/file-20251127160557015.png]]

## Guidance 

- Define region in `Terraform Provider`
- Define VPC
- Define Route Table
- For public Instance: 
	- Define Internet Gateway -> Route Table > routes
- For private Instance:
	- Define NAT Gateway -> Route Table > routes ( NAT Gateway in public subnet )
- Define Subnet
- Define EC2

