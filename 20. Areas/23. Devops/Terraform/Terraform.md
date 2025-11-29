## Infrastructure as Code ( IAC )

- What?
- Why?
	- Can use with **multiple Cloud Provider**.
	- 
- When?
- Process of provision: `Code` > `Version Control` > `Code Review` > `Integrate` > `Deploy`

![[assets/Terraform/file-20251127153454199.png]]
- Key feature: 
	- Terraform Graph
	- Resource Provisioning
	- State Management ( manage AWS resource )
	- Plan and Apply
	- State Locking ( ensures that only one process is existed at a time )
	- Module & Reusable

- Terraform Provider
- Terraform Actions
![[assets/Terraform/file-20251127155425908.png]]
install terraform to using plan ( manage terraform through CLI)
```bash
...
```

- Terraform Module
- Terraform States Files

```tfstate
{
	...
}
```

>[!note] How to save and manage Stage File?
> e.g: AWS S3, Azure Storage, GCP Storage, Terraform Cloud, Google Drive, etc

- Terraform Workflow
![[assets/Terraform/file-20251127160312316.png]]