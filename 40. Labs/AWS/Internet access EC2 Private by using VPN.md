---
tags:
  - devops
  - aws
  - aws/ec2
  - vpn
  - "#openvpn"
---
## 0. Core concept

- Install VPN in public instances
- In local device: **Access VPN -> VPN return special IP (this special IP can access directly to private instances)**

## 1. Launch EC2 instances with VPN

- Create EC2 Instance
	- Image: AMI > Marketplace > OpenVPN ( VPN Provider ) > Sub OpenVPN (BYOL)
	- Type: `t2.small`
	-  Key pair: public
	- Networking:
		- VPC
		- Subnet: public subnet
		- Security group: using default config
		- Storage: 20 GB
- Assign Elastic IP for EC2 instances

## 2. Access to VPN EC2 instances 

- Access VPN instances
```bash
# ssh to VPN instances
ssh -i dop10-public-ec2-kp.pem openvpnas@15.156.56.185
```
- Using default config to install
- Access admin, client UI through URL ( be displayed in CLI )
- In Admin Interface
	- Create User Profile ( secret token ) -> auto download file.ovpn to ur device.
	- Create User: `User Permission` > config (admin + auto-login + default settings)
- In personal device: 
	- Download OpenVPN Application. 
	- In Application Interface 
		- Import `file.ovpn` to connect to VPN Server
		- Login
```bash
# ur device
ping <private-ec2-instances private-IP>
```

> [!note]
> - Can't `ping public ec2 instances` b/c VPN and public instances in the same subnet -> solution: config `user profile` > `access control`

## Use cases

e.g. dev need to access to database ( in private EC2 instances )
- method 1: ssh to public instances -> ssh to private instances.
- method 2: using VPN to access private instances directly from internet.