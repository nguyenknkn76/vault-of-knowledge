---
tags:
  - devops
  - git
  - cicd
---
## 1. Overview

**Core Architecture Rule**
- `composite action` -> **How** a step runs
- `reusable workflow` -> **How** a pipeline runs
- `microservice repo` -> **When** a pipeline runs

![[assets/GitHub Actions - 03. Centralized Reusable Pipelines/Drawing 2026-01-20 11.52.18.excalidraw.png]]

Both of Application pipelines and IaC (Terraform) pipelines must use this design. 

## 2. Repository Types

### 2.1. Central CI/CD Platform Repository

- Repository name: `centralized-cicd-pipelines` (shared repository)
- Contains:
	- reusable workflows (pipeline orchestration)
	- composite actions (step logic)

- Expected structure:
```txt
pipeline-cicd-centralized/
└── .github/
    ├── actions/
    │   ├── java-maven-build/
    │   ├── dotnet-build/
    │   ├── docker-build-push/
    │   ├── terraform-plan/
    │   └── terraform-apply/
    └── workflows/
        ├── pr-open-workflows.yml
        ├── pr-merged-workflows.yml
        ├── manually-workflows.yml
        ├── iac-pr-open-workflows.yml
        ├── iac-pr-merged-workflows.yml
        └── iac-manually-workflows.yml
```

### 2.2. Microservice Repositories

```txt
.github/workflows/
├── pr-opened.yml
├── pr-merged.yml
├── manually.yml
├── iac-pr-opened.yml
├── iac-pr-merged.yml
└── iac-manually.yml
```

> [!note] Assign permission for a repository access another repository
> Settings > Actions > General > Actions permissions > enable `Allow <org-name> actiosn and reusable workflows`

## 3. 

### 3.1. Composite Actions

```yml
name: 'Docker Build and Push'
description: '...'

inputs: 
	DOCKER_USERNAME: 
		description: 'The username for Docker Hub'
		required: true
	DOCKER_PASSWORD: # ...
	IMAGE_NAME: # ...
	IMAGE_TAG: # ...

runs:
	using: 'composite'
	steps:
		- name: Login to Docker Hub
		  uses: docker/login-action@v3
		  with:
			  username: ${{inputs.DOCKER_USERNAME}}
			  password: ${{inputs.DOCKER_PASSWORD}}
		
		- name: Docker meta
		  # ...
		- name: Set up QEMU
		  # ...
		- name: Set up Docker Buildx
		- name: Build and Push Docker Image
```

> [!note] Only `steps` & `inputs`

### 3.2. Reusable  Workflows

```yml
name: 'Reusable Pull Request Opened'

on:
	workflow_call:
		inputs: 
			DOCKER_USERNAME: 
				description: 'Docker Hub username'
				required: true
				type: string
			IMAGE_NAME: # ...
			IMAGE_TAG: # ...
		secrets: 
			DOCKER_PASSWORD:
				required: true

jobs: 
	demo_pipeline: 
		runs-on: ubuntu
		steps:
			# check out 2 repositories: application + cicd platform
			- name: 'Checkout Microservice Repository'
			  uses: actions/checkout@v4
			- name: 'Checkout Centrailized CI/CD Pipelines'
			  uses: actions/checkout@v4
			  with: 
				  repository: <org-name>/centralized-cicd-pipeline
				  path: centrailized-cicd-pipeline
				  token: ${{secrets.GITHUB_TOKEN}} # incase repository is private
			
			# pipeline structure + call composite actions
			- name: Detect Programing Language
			  uses: ./centralized-cicd-pipeline/.github/actions/detect-language
			  
			- name: Analyze Source Code with SonarQube
			  uses: ./centralized-cicd-pipeline/.github/actions/analyze-code
			  with:
				# ...
				
			- name: Build and Push Docker Image to DockerHub
			  uses: ./centralized-cicd-pipeline/.github/actions/docker-build-and-push
			  with:
				IMAGE_NAME: ${{ inputs.IMAGE_NAME }}
				IMAGE_TAG: ${{ inputs.IMAGE_TAG }}
				DOCKER_USERNAME: ${{ inputs.DOCKER_USERNAME }}
				DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
				
			# ...
```

> [! note] Using `on: workflow_call`, `runs-on` and **call** `composite-actions`

> [! note] Checkout 2 Repositories
> - Repository of the application that need to be built.
> - Repository of the pipeline (`centralized-cicd-pipeline`) to get access to its local actions (`composite actions`).

### 3.3. Microservice Workflows

```yml
name: 'PUll REQUEST OPENED'

on:
	pull_request:
		branches: [main, dev]
		types: [opened, synchronize]

jobs:
	code_quality_check:
		uses: <org-name>/centrailized-cicd-pipeline/.github/workflows/reusable-pr-opened.yml@main
		with:
			IMAGE_NAME: 'dop2025.shopsquare.frontend'
			IMAGE_TAG: ${{github.sha}}
			DOCKER_USERNAME: ${{secrets.DOCKERHUB_USERNAME}}
		secrets: 
			DOCKER_PASSWORD: ${{secrets.DOCKER_PASSWORD}}
```

>[! note] only `triggers` & `inputs`

## 4. 

### 4.1. Versioning

In `centrailized-cicd-pipeline`
```bash
git tag v1
git push origin v1
```

In `microservices-repo`: use `@v1`

**Why This Is Mandatory?**
- `@main` changes without warning
- Reusable workflows behave like APIs
- Versioning prevents breaking all services

>[! warning] Common Mistakes
>-  Mixing app and IaC logic in one pipeline
>-  Duplicating pipelines across services
>-  Using `@main`
>-  Hardcoding paths or service names
