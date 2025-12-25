---
tags:
  - cicd
  - git
  - devops
  - cicd/git
  - aws
---
### Step 1: Create Identity Provider (IdP) on AWS

1. Login into AWS Console > Access IAM Service > Identity providers > Add Provider > OpenID Connect
2. Enter Information: 
	- Provider URL: `https://token.actions.githubusercontent.com` 
	- Audience: `sts.amazonaws.com`
3. Get thumbprint (auto get AWS certificate) > Add provider 

### Step 2: Create IAM Role for GitHub Actions

1. IAM menu > Roles > Create role 
2. Select trusted entity:
	- Choose Web identity
	- Identity provider: `token.actions.githubusercontent.com`
	- Audience: `sts.amazonaws.com`
3. Add Permissions: `AdministratorAccess`, `AmazonS3FullAccess`, `AmazonEC2FullAccess`
4. Naming Role (e.g. `GitHubActionsDeployRole`) > CREATE

### Step 3: Config Trust Relationship

Choose `GitHubActionsDeployRole` > Trust relationships > Edit trust policy 

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

### Step 4: Create GitHub Actions Workflow

```yaml
# /.github/workflows/deploy.yml
name: AWS OIDC Connect Demo

on:
  push:
    branches:
      - main

permissions:
  id-token: write   # MUST HAVE to request OIDC token
  contents: read    # NEED to checkout code

env: 
	AWS_REGION: ap-northeast-2
	AWS_IAM_ID: ${{secrets.AWS_IAM_ID}}
	
jobs:
	aws-login:
	    runs-on: ubuntu-latest
	    steps:
		    - name: Checkout Code
		      uses: actions/checkout@v4
	        
			- name: Configure AWS Credentials
			  uses: aws-actions/configure-aws-credentials@v4
			  with:
				  role-to-assume: arn:aws:iam::${{env.AWS_IAM_ID}}:role/GithubActionsDeployRole
				  aws-region: ${{env.AWS_REGION}}
				  
			- name: Verify AWS Connection
			  run: |
				  aws sts get-caller-identity
				  aws s3 ls
```