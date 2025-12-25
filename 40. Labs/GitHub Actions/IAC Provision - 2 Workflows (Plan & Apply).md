---
tags:
  - devops
  - cicd
  - cicd/git
  - git
  - terraform
---
## Sample Infrastructure Folders Structure 
```txt
my-infra-repo/
├──.github/
│   └── workflows/
│       ├── terraform-plan.yaml       # Workflow Run Plan
│       ├── terraform-apply.yaml      # Workflow Run Apply
│       └── drift-detection.yaml      # Workflow Drift Detection
├── bootstrap/                        # Setup (S3 State Bucket, DynamoDB)
│   ├── main.tf
│   └── README.md
├── modules/                          # Reusable Modules (Child Modules)
│   ├── networking/                   # Module VPC, Subnets, NAT, Routes
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── compute/                      # Module ECS/EKS Cluster, ASG
│   ├── database/                     # Module RDS, ElastiCache
│   └── security/                     # Module IAM Roles, Security Groups
├── environments/                     # Detail Config for each Environments (Root Modules)
│   ├── dev/
│   │   ├── backend.tf                # Config S3 remote state for Dev
│   │   ├── main.tf                   # Call modules from ../../modules
│   │   ├── providers.tf              # Provider AWS & Versions
│   │   ├── variables.tf              
│   │   └── terraform.tfvars          
│   ├── staging/
│   │   └──...
│   └── prod/
│       ├── backend.tf
│       ├── main.tf
│       └──...
└── README.md
```

## Workflow 1: Continuous Integration (On Pull Request)

```yaml
name: Terraform CI

on: 
	pull_request:
		branches: [main]
		paths: 
			- 'environments/prod/**' # only run when prod code is changed

permissions:
	id-token: write # MUST have for OIDC
	contents: read
	pull-requests: write # for comment in PR

env: 
	AWS_REGION: ap-northeast-2
	TF_VERSION: 1.6.0

jobs:
	plan: 
		name: Terraform Plan
		runs-on: ubuntu-latest
		defaults:
			run:
				working-directory: environments/prod
			steps:
				- name: Checkout Code
				  uses: actions/checkout@v4
				
				- name: Configure AWS Credentials (Plan Role)
				  uses: aws-actions/configure-aws-credentials@v4
				  with: 
					  role-to-assume: arn:aws:iam::1234567:role/TerraformPlanRole
					  aws-region: ${{env.AWS_REGION}}
					  
				- name: Setup Terraform  
					  uses: hashicorp/setup-terraform@v3  
					  with:  
						  terraform_version: ${{ env.TF_VERSION }}  
						  
				- name: Terraform Init  
				  run: terraform init -backend-config="bucket=my-state-bucket"
				    
				- name: Terraform Format & Validate  
				  run: |  
					  terraform fmt -check  
					  terraform validate
					  
				- name: Terraform Plan
				  id: plan
				  run: terraform plan -no-color -detailed-exitcode -out=tfplan
				  continue-on-error: true
				
				- name: Publish Plan to PR
				  uses: actions/github-script@v6
				  if: github.event_name == 'pull_request'
				  with:
					  script: |
					  # js script to read plan output + comment to PR
```

## Workflow 2: Continuous Deployment (On Push to Main)

```yaml
name: Terraform CD

on:
  push:
    branches: [ main ]
    paths:
      - 'environments/prod/**'

permissions:
  id-token: write
  contents: read

jobs:
	apply:
	    name: Terraform Apply
	    runs-on: ubuntu-latest
	    environment: production # Active Approval Rule HERE
	    defaults:
		    run:
		        working-directory: environments/prod
	
	    steps:
		    - name: Checkout Code
		      uses: actions/checkout@v4
		    - name: Configure AWS Credentials (Apply Role)
		      uses: aws-actions/configure-aws-credentials@v4
		      with:
		          role-to-assume: arn:aws:iam::123456789:role/TerraformApplyRole
		          aws-region: ap-northeast-2
			
			- name: Setup Terraform
			  uses: hashicorp/setup-terraform@v3
	
			- name: Terraform Init
			  run: terraform init -backend-config="bucket=my-state-bucket
			
			- name: Terraform Apply
			  run: terraform apply -auto-approve -lock-timeout=15m
```

