---
tags:
  - devops
  - iac
  - terraform
  - aws/ec2
---
## Install Terraform 

```bash
sudo dnf install -y dnf-plugins-core curl
sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo

sudo dnf install terraform
terraform -v
# Output: Terraform vX.Y.Z on linux_amd64
```

> [!bug] Can't run `sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo`
> -> Create repo file manually

- Create config repo 

```bash
sudo nano /etc/yum.repos.d/hashicorp.repo
```

```repo
# in hashicorp.repo
[hashicorp]
name=HashiCorp Stable - $basearch
baseurl=https://rpm.releases.hashicorp.com/fedora/$releasever/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://rpm.releases.hashicorp.com/gpg
```

- Install Terraform

```bash
sudo dnf makecache
sudo dnf install terraform
terraform -v
```

## Provision EC2 Instances

```bash
export AWS_ACCESS_KEY_ID=
export AWS_SECRET_ACCESS_KEY=

# or run 
aws configure 
aws s3 ls
```

```bash
terraform fmt
terraform init
terraform validate
terraform plan
terraform apply
terraform destroy
```

> [!note] 
> - don't break process when running apply or destroy

## References

- [Terraform Quick Start](https://developer.hashicorp.com/terraform/tutorials)
