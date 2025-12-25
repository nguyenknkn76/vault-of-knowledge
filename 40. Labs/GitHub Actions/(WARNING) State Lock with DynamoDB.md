---
tags:
  - devops
  - terraform
  - git
  - cicd
  - aws
  - aws/dynamodb
  - cicd/git
---
> [!bug] State Lock is still fail -> HAVE TO CHECK ONE MORE TIME

## Step 1: Create DynamoDB Table in AWS 

AWS CLI
```bash
aws dynamodb create-table \
  --table-name dop10-tfd-dynamodb-an2-lenv-01 \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 \
  --region ap-northeast-2
```

   * Replace my-terraform-state-lock with the name you want for your lock table.
   * Replace <YOUR_AWS_REGION> with your AWS region (e.g., us-east-1).

## Step 2: Config in Terraform Code

`/.infra/backend.tf`

```tf
terraform {
	backend "s3" {
		bucket = "dop10-tfd-s3-an2-lenv-01"
		key = "dop10/dev-tfstate"
		region = "ap-northeast-2"
		encrypt = true
		dynamodb_table = "dop10-tfd-dynamodb-an2-lenv-01" 
	}
	
	required_providers {
		aws = {
			source = "hashicorp/aws"
		}
	}
}
```

> [!note] 
> Using exactly name of table that be created in Step 1. 
> - (e.g. `dop10-tfd-dynamodb-an2-lenv-01`)


