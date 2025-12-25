---
tags:
  - devops
  - cicd
  - cicd/git
  - git
  - aws
  - terraform
---

This GitHub Actions workflow handles the destruction of infrastructure provisioned via Terraform. It includes safety mechanisms such as manual approval inputs and environment protection rules.

## Workflow Details

- **Name:** IAC Destroy Infrastructure
- **Trigger:** Manual (`workflow_dispatch`)
- **Safety Check:** Requires typing "DESTROY" to confirm.
- **Runner:** `dop10_self_hosted_runner_01`
- **Environment:** `dev`

## Workflow Code

```yaml
name: IAC Destroy Infrastructure

on:
  workflow_dispatch:
    inputs:
      confirmation:
        description: 'Type "DESTROY" to confirm infrastructure destruction'
        required: true
        default: ''

permissions:
  id-token: write
  contents: read

env:
  TF_VERSION: latest
  AWS_REGION: ap-northeast-2
  AWS_IAM_ID: ${{secrets.AWS_IAM_ID}}

jobs:
  destroy_infrastructure:
    name: Destroy Infrastructure
    if: ${{ github.event.inputs.confirmation == 'DESTROY' }}
    runs-on: dop10_self_hosted_runner_01
    environment: dev
    steps:
      - name: Update OS
        run: |
          sudo apt-get update
          sudo apt-get install -y libatomic1 unzip

      - name: Checkout Repository
        uses: actions/checkout@v6

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::${{env.AWS_IAM_ID}}:role/GithubActionsDeployRole
          aws-region: ${{env.AWS_REGION}}

      - name: Install AWS CLI v2
        run: |
          curl "[https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip](https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip)" -o "awscliv2.zip"
          unzip awscliv2.zip
          sudo ./aws/install --update

      - name: AWS Connection Check
        run: |
          aws --version
          aws s3 ls
          # aws sts get-caller-identity

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{env.TF_VERSION}}

      - name: Terraform Version Check
        run: terraform version

      - name: Terraform Init
        run: |
          pwd
          cd .infra
          terraform init

      - name: Terraform Format
        run: |
          terraform fmt -check
          terraform validate

      - name: Terraform Plan
        working-directory: .infra
        run: terraform plan -no-color

      - name: Terraform Destroy
        working-directory: .infra
        run: terraform destroy -auto-approve
```

> [!note]
> Have to run `terraform init` and `terraform plan` before run `terraform destroy`

