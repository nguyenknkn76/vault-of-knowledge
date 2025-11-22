---
tags:
  - aws
  - "#devops"
  - aws/ec2
  - aws/vpc
---
![[assets/Setup Privates and Public EC2 Instances (Basic)/file-20251121142825095.png]]

## 0. Resources 

| No. | Resource         | Name                  | IP            | Note                      |
| --- | ---------------- | --------------------- | ------------- | ------------------------- |
| 01  | VPC              | dop10-vpc             | 10.11.0.0/16  | only VPC + default config |
| 02  | Subnet           | dop10-public-sn       | 10.11.0.2/24  | zone a                    |
|     |                  | dop10-private-sn      | 10.11.11.0/24 | zone b                    |
|     |                  | dop10-databse-sn      | 10.11.20.0/24 | zone c                    |
| 03  | Route Table      | dop10-public-rtb      |               | associations + routes     |
|     |                  | dop10-private-rtb     |               | subnet associations       |
| 04  | Internet Gateway | dop10-igw             |               |                           |
| 05  | EC2              | dop10-public-ec2      |               | `t3.micro`                |
|     |                  | dop10-private-ec2     |               |                           |
|     |                  | dop10-db-ec2          |               |                           |
| 06  | Security Group   | dop10-public-sg       |               |                           |
|     |                  | dop10-private-sg      |               |                           |
| 07  | Key Pair         | dop10-public-ec2-kp   |               |                           |
|     |                  | dop10-private-ec2-kp  |               |                           |
|     |                  | dop10-database-ec2-kp |               |                           |
| 08  | NAT Gateway      | dop10-nat-private     |               | private access internet   |
| 09  | Elastic IP       |                       |               | if NOT assign public ip   |
| 10  | Open VPN         | dop10-vpn-openvpn-ec2 |               |                           |

## 1. Create VPC & Subnet

### VPC

- only VPC
- default config

### Subnet

### Route Table

1. Create Route Table
2. Connect with Subnet  
	-  `Route Table` > `Subnet associations` > Edit > Choose available subnets
3. Determine What Route Table is public or private?
	- `Route Table` > `Routes` > Edit
		- Destination: `0.0.0.0/0`
		- Target: `Internet Gateway` 
4. Determine Private Instances can access the internet?
	- `Private Route Table` > Routes > Edit
		- Destination `0.0.0.0/0` 
		- Target: `NAT Gateway`

> [!note]
> - NAT Gateway + private route table
> - Internet Gateway + public route table
### Internet Gateway

1. Create Internet Gateway
2. Action > Attach to VPC > `dop10-vpc`
### NAT Gateway

1. Create NAT Gateway
2. Config
	- Subnet: ...
	- Connectivity type: public
	- Elastic IP: `Allocate Elastic IP`

### Elastic IPs

1. Create Elastic IP (fixed public IP to config domain)
2. Action > Associate Elastic IP > Choose EC2 instance

> [!note]
> - Subnet in the same VPC can communicate together.

## 2. Create EC2 Instances
### Create public EC2 instances 

- OS + images + architecture
- Type: `t3.micro`
- Key pair (using to access to EC2)
	- Create key pair > config: `RSA` + `.pem` > Download token
- Choose VPC + Subnet
- Auto assign public IP
	- `enable`
	- `disable` -> use `Elastic IPs` > Create Elastic IPs
- Security group: 
	- Create `Security Group`
		- Inbound
			- Type: All traffic ( just use to test `ping` public IP from internet)
			- Source: Any where or My IP
		- Outbound
	- Config storage
- Double check current config before create > Launch instance

### Create private EC2  instances

- OS + images + architecture
- Type: `t3.micro`
- Key pair (using to access to EC2)
	- Create key pair > config: `RSA` + `.pem` > Download token
- Choose VPC + Subnet
- Auto assign public IP: disable 
- Security group: 
	- Create `Security Group`
		- Inbound
			- Type: Any where or My IP or ... 
			- EXPLAIN: don't have public IP + in private Subnet -> can NOT access this EC2 instances from internet 
		- Outbound
	- Config storage
- Double check current config before create > Launch instance

> [! tip] Best practices
> - Using diff security group for public and private EC2 instances
> - Using diff key pair for each EC2 instances

> [!note]
> - Key pair -> 2 keys -> public key in EC2 instances + private key in Local Machine -> using it to authenticate and access to EC2 instances
> - Can use `console-to-code` create EC2 through CLI
> - `0.0.0.0/0`: is mean everywhere in the internet 

> [!warning] 
> - Inbound of Security Group in real-world project: secure type (**custom**) + source (aka **IP white list**)
> - Get public IP (to add to Security Group) by using `whatismyipaddress`

## 3. Define of done 

- Internet can access EC2 public
- EC2 public can access EC2 private, database
- EC2 private, database can access internet, but internet can NOT access at all

### Internet access EC2 public

```bash
# ur device
ping <dop10-public-ec2 public-IP>

# access to EC2 instances ( Connect > SSH client )
# sample
chmod 400 dop10-public-ec2-kp.pem
ssh -i dop10-public-ec2-kp.pem ubuntu@3.96.172.223
# result (cli): ubuntu@10.11.0.23 
# -> private ip and ec2 instances in same vpc can access each other throught this private ip

# public ec2 instance
ping 8.8.8.8
```
 
### Public EC2 instance access private EC2 instance

```bash
# public ec2 instance
ping <dop10-private-ec2 private-IP>
```

### SSH to EC2 Private 

```bash 
# ur local device
# copy key pair to public instances
scp -i dop10-ec2-private-kp.pem ubuntu@3.35.50.243:/home/ubuntu/key-pair
# sus method: cat keypair content and copy manualy.

ls -la 
chmod 400 dop10-private-ec2-kp.pem 
ssh -i dop10-private-ec2-kp.pem ubuntu@10.11.11.11
# OS aws linux: user = ec2-user
```

### EC2 Private access internet

```bash
# note: have to config nat gateway
ping google.com
```


