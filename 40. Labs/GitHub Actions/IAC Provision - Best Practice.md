---
tags:
  - aws
  - terraform
  - git
  - cicd/git
---
## 1. Configure AWS Role for OIDC

1. Create Identity Provider (IdP)
2. Create IAM Role for GitHub Actions `GitHubActionsDeployRole`
3. Config Trust Relationship

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::DATA_CUA_BAN:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                },
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:USERNAME/REPO-NAME:*"
                }
            }
        }
    ]
}
```

4. Config GitHub Actions Workflow

```yml

permission: 
	id-token: write
	...
	
env: 
	AWS_REGION: ap-northeast-2
	AWS_IAM_ID: ${{secrets.AWS_IAM_ID}}
	
jobs: 
	...
		runs-on: ...
		steps: 
			- ...
			- Configure AWS Credentials
			  uses: aws-actions/configure-aws-credentials@v4
			  with:
				  role-to-assume: arn:aws:iam::${{env.AWS_IAM_ID}}:role/GithubActionsDeployRole
				  aws-region: ${{env.AWS_REGION}}
			  
```

> Read more here: [[Access AWS with OIDC]]

> [!tip] BEST PRACTICE: Separation Role Strategy (Plan & Apply) 
> Create 2 Roles:
> - Plan Role for `Plan Job`
> - Apply Role for `Apply Job`

## 2. Configure S3 Bucket for State File

1. Create S3 Bucket `dop10-tfd-s3-an2-lenv-01` in AWS
2. Setup IAC  
`/.infra/backend.tf`

```tf
terraform {
	backend "s3" {
		bucket = "dop10-tfd-s3-an2-lenv-01"
		key = "dop10/dev-tfstate"
		region = "ap-northeast-2"
	}

  

	required_providers {
		aws = {
			source = "hashicorp/aws"
		}
	}
}
```

`/.infra/provider.tf`

```
provider "aws" {
	region = "ap-northeast-2"
}
```

## 3. GitHub Actions: Setup Manually Approve

Git > Settings > Environments > Deployment protection rules > Enable `required reviewers` 

`/.github/workflows/deploy.yml`

```yml
jobs: 
	terraform-plan:
		...
	terraform-apply:
		name: Terraform Apply
		needs: terraform-plan # wait terraform plan run successful
		runs-on: self_hosted_runner
		environment: dev # enable Rule Approval
```

## 4. GitHub Actions Workflows 

```yml
name: IAC Deploy Best Practice

on:
	push: 
		branches:
			- main

permissions:
	id-token: write
	contents: read

env:
	AWS_REGION: ap-northeast-2
	AWS_IAM_ID: 1234567890

jobs:
	# --- JOB 1: PLAN ---
	terraform-plan:
		name: Terraform Plan
		runs-on: self_hosted_runner
		outputs:
			exitcode: ${{ steps.plan.outputs.exitcode }}
		steps:
			- name: Checkout
			  uses: actions/checkout@v4
			  
			- name: Configure AWS Credentials
			  uses: aws-actions/configure-aws-credentials@v4
			  with:
				  role-to-assume: arn:aws:iam::${{env.AWS_IAM_ID}}:role/GitHubActionsDeployRole
				  aws-region: ${{env.AWS_REGION}}
				  
			- name: Setup Terraform 
			  uses: hashicorp/setup-terraform@v3
			  
			- name: Terraform Init 
			  run: terraform init 
			  
			- name: Terraform Plan 
			  id: plan 
			  # create tfplan binary file. WARN: this file include sensitive data
			  run: terraform plan -no-color -out=tfplan
			  
			  # upload tfplan artifact to repo -> use this artifact in next step
			- name: Upload TF Plan Artifact 
			  uses: actions/upload-artifact@v4 
			  with: 
				  name: tfplan-artifact 
				  path: tfplan 
				  retention-days: 1 # just save one day for security
				  
	# --- JOB 2: APPLY (wait for approval) ---
	terraform-apply:
		name: Terraform Apply
		needs: terraform-plan # wait terraform plan run successful
		runs-on: self_hosted_runner
		environment: dev # enable Rule Approval
		
		steps: 
			- name: Checkout 
			  uses: actions/checkout@v4
			
			- name: Configure AWS Credentials
			  uses: aws-actions/configure-aws-credentials@v4
			  with:
				  role-to-assume: arn:aws:iam::${{env.AWS_IAM_ID}}:role/GitHubActionsDeployRole
				  aws-region: ${{env.AWS_REGION}}
				  
			- name: Setup Terraform 
			  uses: hashicorp/setup-terraform@v3
			  
			- name: Terrafrom Init
			  run: terraform init
			  
			# download tfplan artifact of previous job
			- name: Download TFPlan Artifact
			  uses: actions/download-artifact@v4
			  with: 
				  name: tfplan-artifact
				  path: .
				  
			- name: Terrafrom Apply
			  # using tfplan -> DON'T NEED auto-approve flag
			  run: terraform apply -no-color tfplan 
```

> [!note] Re run `terraform init` in Terraform Apply Job
> - `terraform init`: Run again in Job 2. b/c GitHub Runner in this job is different with Runner in previous job **Plan Stage**

> [!note] State Locking
> `tfplan` file **can be changed** in period of time when we wait approve from Approval.
> - **Solution:** Using `Dynamo DB` for **state locking**.
> -> Terraform will warn, If someone try to change infrastructure when this process doesn't finished.

> [!warning] Security `tfplan` file
> - `retention-days: 1`  Preventing `tfplan` be leaked and Attacker can decode tfplan content (in binary form).
> - GitHub Action Artifact Default is `private`, only user (with `read repo` permission) can download `tfplan` from GitHub Actions Artifact -> private repo is safer
> - `terraform plan` : Result of this script includes sensitive data.







