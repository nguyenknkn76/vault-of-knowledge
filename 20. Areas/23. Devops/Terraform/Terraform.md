## Infrastructure as Code ( IAC )

- What?
	- Process of provision: `Code` > `Version Control` > `Code Review` > `Integrate` > `Deploy`
- Why?
	- Can use with **multiple Cloud Provider**. (ex: using GCP to analysis data, using Azure to manage enterprise office, using to deploy high traffic application, etc)
- When?

- State in Terraform
![[assets/Terraform/file-20251127153454199.png]]
- Key feature: 
	- IAC Terraform Graph
	- Resource Provisioning
	- State Management ( manage AWS resource )
	- Plan and Apply
	- State Locking ( ensures that only one process is existed at one time )
	- Module & Reusable

- Some important concepts 
	- Terraform Provider
	- Terraform Actions
	- Terraform Module
	- Terraform States File `.tfstate`
	- Terraform Workflow

> [! note] Remote State Storage 
> - NOT save state file in personal device -> **save in storage centralize** (e.g AWS S3)

![[assets/Terraform/file-20251127160312316.png]]


## Terraform Module 

## AWS Naming Convention

`<company/project>-<team>-<app_name>-<service_name>-<region>-<env>-<suffix>`

> [!warning] 
> - NOT force stop `terraform apply` or  `terraform destroy` process

> [!tip] How to manage Terraform State File?
> - Using S3 Bucket
> - Remote State Locking 

> [!note]
> - After update backend provider.tf file have to run `terraform init` again.

