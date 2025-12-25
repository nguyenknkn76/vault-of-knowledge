---
tags:
  - devops
  - cicd
  - git
  - cicd/git
---
![[assets/GitHub Actions - 01. Application CICD/file-20251219114712870.png]]

```yml
 steps:
  # Clone source code to Runner
  - name: Checkout Repository  
	uses: actions/checkout@v4  
	with:  
	  fetch-depth: 0 # just get 
```

> [!tip] Security Best Practise
```yml
# Just assign read permission for github_token -> clone, fetch
# SUS: `default = write` -> push, commit, tag, ... directly on git repo
permissions: 
	contents: read
```

```yml
# /.github/workflows/github-actions-demo.yml
name: 
run-name:
on: [push]
jobs: 
	...
		runs-on:
		steps:
			- run: echo "The job was automatically triggered by ... event"
			- name: Check out repository code
			  uses: actions/checkout@v5
			- run: echo "..."
```

pre-define actions: ... (e.g. `actions/checkout@v5`, `actions/setup-java@v5`)

> [!note] pre-define actions:
> - Side Project: using pre define action ( that be provided by Community )
> - Enterprise: using Actions ( that be defined by themselves )

run-on
- GitHub Shared Runner ->infra deployed by GitHub + limited + CAN'T ssh 
- Self hosted Runner -> host in EC2 instances

How to setup `Self-hosted Runner` ?

Setup Self-host Runner for Repository (or Organization) ? 
- GitHub: Settings > Actions > Runners 
- AWS: Login to AWS: Create EC2 Instance > ...
- Running bash scripts like Guidance of GitHub in EC2 Instances to Install GitHub Actions in EC2 Instances 

Install necessary dependencies:
- Install manually: ssh to Agent (EC2 Instances)
- Using pipeline file to install automatic

```bash

```

> Quick Start Tutorial: [Github Actions Quickstart](https://docs.github.com/en/actions/get-started/quickstart)

**When does GitHub trigger a workflow to be started?**
- An event on GitHub (push a commit, pull request or issues are created)
- A scheduled event
- An external event 

> Read more here: [Trigger GitHub Actions Workflow Events](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows)

## Git Workflows 

Protected Branch:
- CAN'T remove
- CAN'T push directly
- Change source code by merge event
- All action refer to **manually approve**
- Code Long Time Support (LTS) / Stable

3 Pipelines: 
- Pull Request Opened Pipeline
- Pull Request Closed / Merged Pipeline
- Pull Request Manual Trigger Pipeline

```yml
# conditions: when ci/cd pipeline run
on:
	pull_request:
		branches: 
			- main
			- master
			- feature
		# types: [closed]
		
# conditions: when pull requests is closed 
on:
	pull_request:
		types: [closed]
		
# manual trigger pipeline
on:
	workflow_dispatch:
		inputs: 
			environment:
				type: ...
			...
```

Environment: j4f in GitHub + Types of Input

Manage Secrets: 
- Settings > `Secrets and variables`

> Read more here: [Best Practices for GitHub Actions Secret Management](https://www.graphite.com/guides/best-practices-for-github-action-secrets-management)

## EXAMPLES: CI/CD Pipeline for ReactJS Project

![[../../../40. Labs/GitHub Actions/CICD Pipeline Code Quality Check for ReactJS Project|CICD Pipeline for ReactJS Project]]


> [! note] GitHub Actions
> - `fetch-depth: 0`
> 	- default value (1): just clone latest commit -> save time + resource 
> 	- custom value (0): full clone commit history -> necessary for SonarQube
> - `paths-ignore` & `paths`


> [! tip]
> - `run: npm run ci`  **Clean Install**: Del current node_modules folder -> read package.json file and install dependencies base on package.json file -> Ensure: consistency 

## 2. Build, Test & Caching Strategy

### 2.1. Matrix Strategy: Multi Environments Testing 

> [!tip] Insight 
> - `fail-fast: false` if a jobs fail, Another jobs still run to get results 
> 	- default: true to save resource
> 	- custom: false to see overall picture

### 2.2. Caching 

Saving time and resource of re-install dependencies ‌/ each build

**Mechanism**: key -> Cache Hit or Cache Miss

2 caching methods
- **Built in Action Caching** (**Recommend**): 
```yml
- uses: actions/setup-node@v4  
  with:  
    cache: 'npm' # auto hash package-lock.json as key
```

- **`actions/cache`** manually + flexible 
```yaml
- name: Cache Custom Build Artifacts  
  uses: actions/cache@v4  
  with:  
    path: |  
      ~/.npm  
    ./.next/cache  
    # Key cache includes: OS + hash of dependencies file  
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}  
    # Restore keys: if key isn't match, try to find older cache (partial match)
    restore-keys: |  
      ${{ runner.os }}-node-
```

> Cache is **immutable** -> It CAN'T be overwrite. Just delete old cache and create new key + new cache

## 3. Docker & Containerization

Require of `build docker image`: **Speed & Security**

### 3.1. Docker Build & Layer Caching

![[../../../40. Labs/GitHub Actions/CICD Pipeline Docker Build and Push Nodejs Image to Docker Hub|CICD Pipeline Docker Build and Push Nodejs Image to Docker Hub]]

> [!note] Caching Strategy
> - `type = gha`
> - `mode = max`

## 3.2. Container Security: **Vulnerability Scanning**

Integrate **Trivy** into pipelines to prevent unsafe images

```yml
- name: Run Trivy Scanner
  uses: aquasecurity/trivy-action@0.33.1
  env:
	  # in case, dockerhub repo is private
      DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
      DOCKERHUB_PASSWORD: ${{ secrets.DOCKERHUB_PASSWORD }}
  with:
	  image-ref: '${{ secrets.DOCKERHUB_USERNAME }}/${{ github.event.repository.name }}:${{ github.sha }}'
	  format: 'table'  
	  exit-code: '1' # Fail pipeline if an error is detected 
	  ignore-unfixed: true # Only report error if a fix is available  
	  severity: 'CRITICAL,HIGH'
```

> [!note] Insight 
> - `exit-code: '1'` immediately block Pipeline (Pull Request)
> - `continue-on-error: true` only block or warn with  CRITICAL errors

## 4. **Continuous Deployment**

2 Popular Solutions: SSH and Cloud Native

### 4.1. Traditional Deploy: SSH & VPS

> [!warning] **Risk**: 
>  - Save Private SSH Key in GitHub Secrets -> Risk of **Key Leak**
>  - Key Rotation is hard to manage manually

```yaml
jobs:
	deploy-vps: 
		runs-on: ubuntu-latest
		steps: 
			- name: Deploy via SSH
			  uses: appleboy/ssh-action@v1.0.3  
				  with:  
					  host: ${{ secrets.HOST }}  
					  username: ${{ secrets.USER }}  
					  key: ${{ secrets.SSH_PRIVATE_KEY }}  
					  port: 22  
					  script: |  
						cd /var/www/myapp  
						git pull origin main  
						docker compose up -d --build --remove-orphans  
						docker system prune -f 
```

> [!note] Best Practice
> In spite of the fact that `appleboy/ssh-action` is very popular but **Best Practice**
> is using `rsync` or `ssh native` with strong config `StrictHostKeyChecking`
> -> limit permission of `ssh user` ( just have permission to run docker, not sudo root)

### 4.2. Modern Deploy: AWS & OIDC (without secrets)

OpenID Connect (OIDC) > save AWS_ACCESS_KEY in GitHub Secrets

**Flows**
1. GitHub Actions requests short term token (JWT)
2. Send token to AWS STS (Secrets Token Service)
3. AWS validate token (`from repo X branch Y`)
4. If true, AWS returns  temp session token for deploying

```yml
permissions:
	id-token: write 
	contents: read
	
	steps:
		- name: Config AWS Credentials
		  uses: aws-actions/configure-aws-credentials@v4
		  with:
			   role-to-assume: arn:aws:iam::123456789:role/GitHubDeployRole
			   aws-region: ap-northeast-2
			   
		- name: Update ECS Service / Lambda / S3
		  run: |
			  aws ecs update-service --cluster my-cluster --service my-service --force-new-deployment	  
```

> [!note] Insight
> - **Least Privilege Principle** and manage dynamic secrets 
> - Config IAM role -> only have permission to deploy from production in GitHub

## 4.3. Environments & Manual Approvals 

Prevent wrong deploy to production -> Using Environments Feature of GitHub
1.  Settings > Environments > create `Production`
2.  Enable Required reviewers: Tech Lead or DevOps Lead

```yml
jobs:
	deploy-production:
		runs-on: ubuntu-latest
		needs: [build, test]
		environment:
			name: Production
			url: https://application-domain.com
		steps:
			- run: ./deploy.sh
```

Pipeline run and meet this job -> Suspend + send notification to Approval 
Approval approve -> Job continues to run

## Microservices &  Monorepo

### Conditional Execution & Path Filters

Using `dorny/paths-filter` to detect changes and redirect pipeline logic
### Reusable Workflows

```yml
# **template**: .github/workflows/maven-build.yml
name: Maven Build Template  
on:  
  workflow_call:  
    inputs:  
      service-name:  
        required: true  
        type: string  
    secrets:  
      SONAR_TOKEN:  
        required: true  
jobs:  
  build:  
    runs-on: ubuntu-latest  
    steps:  
      - uses: actions/checkout@v4  
      - name: Build ${{ inputs.service-name }}  
        run: mvn clean package

# ---

# **caller workflow**: .github/workflows/caller-workflow.yml
jobs:  
  build-order:  
    uses:./.github/workflows/maven-build.yml  
    with:  
      service-name: order-service  
    secrets:  
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

## Enterprise: Security and Management

- Manage Secrets & Gitleaks: NOT hardcode secret + using `gitleaks` to auto scan in pipelines
- Pinning Actions: 
	- Solutions: Pin the version using an immutable SHA hash code.
		- unsafe: `uses: third-party/action@v1`
		- **safe**: `uses: third-party/action@immutable-sha-hash-code`
- Self-Hosted Runner: **NOT** using with Public Git Repository

## Cost Optimize and Observability

| Issues              | Solutions               | Config                                                                              |
| ------------------- | ----------------------- | ----------------------------------------------------------------------------------- |
| Hanging Jobs        | Set Timeouts            | `timeout-mitutes: 10`                                                               |
| Excessive Artifacts | Reduce Retention Period | Settings: 90 days -> 3 - 7 days                                                     |
| Commit Spam         | Cancel Obsolete Jobs    | ```yml<br>concurrency: group: ${{ github.ref }} <br>cancel-in-progress: true<br>``` |
| Idle Runner         | Optimize Runner Size    | choose Linux Runner (cheapest)                                                      |
```yml
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 10 # Highly recommended to avoid high costs
```

```yml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

## References

https://viblo.asia/p/toi-uu-hoa-workflows-github-actions-GAWVpqWP405
https://viblo.asia/p/reusable-workflows-tai-su-dung-workflows-trong-github-actions-zOQJwowbJMP