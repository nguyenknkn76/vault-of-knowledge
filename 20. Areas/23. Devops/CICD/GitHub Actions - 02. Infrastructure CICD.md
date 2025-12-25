---
tags:
  - aws
  - terraform
  - git
  - cicd/git
  - cicd
  - devops
  - devops/version-control
---
## 1. CI/CD Infrastructure ( Terraform )

## 2. State Management & Locking

### 2.1. Remote State with S3 and DynamoDB

- Storage (S3): File `terraform.tfstate` is saved in S3 (encryption + versioning)
- Locking (DynamoDB): through `LockID` 

### 2.2. Resolve `State Lock` Issues

When does it happen? `terraform apply` is cancelled suddenly

Solutions: 
1. Set Timeout: using `-lock-timeout` flag `terraform apply --lock-timeout=15m`
2. Force Unlock: (**DANGER ACTION**) `terraform force-unlock <LOCK_ID>`
3. Auto Clean: TTL (Time-To-Live) of Dynamo -> delete old key (**carefully**)
## 3. Security with OIDC and IAM Roles

### 3.1. OIDC

### 3.2. Setup Trust Policy & sub Claim Risk 

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::ACCOUNT_ID:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                },
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:MyOrg/my-backend-repo:ref:refs/heads/main"
                }
            }
        }
    ]
}
```

- aud (Audience): `sts.amazonaws.com`
- sub (Subject): (**most important**)
	- DANGER: sub = `repo:MyOrg/*` or `repo:MyOrg/my-repo:*`
	- **Strict**: `repo:MyOrg/my-repo:ref:refs/heads/main`

### 3.3. Separation of Roles Strategy: Plan & Apply

| Role Type  | Permissions/Policies                                            | Trust Policy                                        |
| ---------- | --------------------------------------------------------------- | --------------------------------------------------- |
| Plan Role  | **Read-Only:**  `ec2:Describe`, `s3:ListBucket`, `s3:GetObject` | assume by `pull_request` (by all branch)            |
| Apply Role | **Read-Write:** `s3:PutObject` ~ State                          | only be assumed by `merge` event (by `main` branch) |
|            |                                                                 |                                                     |
## 4. CI/CD Pipeline

### 4.1. Cost & Environments

- Public Repository
- Private Repository: Lock Environment with Approval feature (Free/Pro Account)
- Runner Cost

> [!tip] Workaround Solution
> Using 2 different workflows 
> - workflow 1: auto `plan`
> - workflow 2: manually trigger (workflow_dispatch) to run `apply` after review plan result

### 4.2. Passing Artifact Technique
 
Plan -> Export plan file -> Save as artifact -> Send it to Job Apply
 - Job Plan: `terraform plan -out=tfplan` -> `actions/upload-artifact`
 - Job Apply: `terraform apply tfplan` -> `actions/download-artifact`

### 4.3. Detail Config of Workflow (YAML)

[[../../../40. Labs/GitHub Actions/IAC Provision - 2 Workflows (Plan & Apply)|Pipeline For Enterprise]]

## 

### Drift Detection

Happen when: User changes resource directly in AWS Console with out using Terraform Code -> Drift: Different between real and Terraform Code

Flows:
1. Schedule: Run at 02:00 UTC daily
2. Plan: run `terraform plan -detailed-exitcode -refresh-only` (`exitcode=2` -> "Drift")
3. Reporting: If detect "Drift", auto create GitHub Issue with tag "Drift"
4. Send notification to team through `Slack/Microsoft Chanel`

### Notification

Sample: Sending Notification when Job fail
```yaml
- name: Notify Slack on Failure  
	if: failure() # Only run when previous step failures 
	uses: rtCamp/action-slack-notify@v2  
	env:
		SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}  
		SLACK_CHANNEL: critical-alerts  
		SLACK_COLOR: '#FF0000'  
		SLACK_MESSAGE: 'Warning: Terraform Deploy Fail on Production!'  
		SLACK_TITLE: Deployment Failure
```
