---
tags:
  - devops
  - cicd
  - cicd/git
  - docker
---
## 01. Composite Actions

`/docker-build-and-push/action.yml`

```yml
## /cicd-pipeline-centralized/.gihub/actions/docker-build-and-push/action.yml
name: 'Docker Build and Push'
description: 'A composite action to build and push Docker images to various container registries.'

inputs:
  REGISTRY_TYPE:
    description: 'The type of container registry (dockerhub or ecr).'
    required: true
    default: 'dockerhub'
  # Docker Hub Inputs
  DOCKER_USERNAME:
    description: 'The username for Docker Hub.'
    required: false
  DOCKER_PASSWORD:
    description: 'The password or token for Docker Hub.'
    required: false
  # ECR Inputs
  AWS_ACCESS_KEY_ID:
    description: 'AWS access key ID for ECR.'
    required: false
  AWS_SECRET_ACCESS_KEY:
    description: 'AWS secret access key for ECR.'
    required: false
  AWS_REGION:
    description: 'AWS region for ECR.'
    required: false
  # Common Inputs
  IMAGE_NAME:
    description: 'The name of the Docker image.'
    required: true
  IMAGE_TAG:
    description: 'The tag for the Docker image.'
    required: false
    default: 'latest'
  DOCKERFILE_PATH:
    description: 'The path to the Dockerfile.'
    required: false
    default: 'Dockerfile'
  BUILD_CONTEXT:
    description: 'The build context for the Docker build.'
    required: false
    default: '.'

runs:
  using: 'composite'
  steps:
    - name: Configure AWS credentials
      if: ${{ inputs.REGISTRY_TYPE == 'ecr' }}
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ inputs.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ inputs.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ inputs.AWS_REGION }}

    - name: Login to Amazon ECR
      id: login-ecr
      if: ${{ inputs.REGISTRY_TYPE == 'ecr' }}
      uses: aws-actions/amazon-ecr-login@v2

    - name: Login to Docker Hub
      if: ${{ inputs.REGISTRY_TYPE == 'dockerhub' }}
      uses: docker/login-action@v3
      with:
        username: ${{ inputs.DOCKER_USERNAME }}
        password: ${{ inputs.DOCKER_PASSWORD }}

    - name: Docker meta
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: |
          ${{ inputs.REGISTRY_TYPE == 'ecr' && steps.login-ecr.outputs.registry || inputs.DOCKER_USERNAME }}/${{ inputs.IMAGE_NAME }}
        tags: |
          type=raw,value=${{ inputs.IMAGE_TAG }}

    - name: Set up QEMU
      uses: docker/setup-qemu-action@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Build and push Docker image
      uses: docker/build-push-action@v6
      with:
        context: ${{ inputs.BUILD_CONTEXT }}
        file: ${{ inputs.DOCKERFILE_PATH }}
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
```

## 02. Reusable Workflows

`/workflows/demo-workflow.yml`

```yml
# /cicd-pipeline-centralized/.gihub/workflows/demo-workflow.yml
name: '...'

on:
  workflow_call:
    inputs:
      REGISTRY_TYPE:
        description: 'The type of container registry (dockerhub or ecr).'
        required: false
        type: string
        default: 'dockerhub'
      IMAGE_NAME:
        description: 'The name of the Docker image.'
        required: true
        type: string
      IMAGE_TAG:
        description: 'The tag for the Docker image.'
        required: false
        type: string
        default: 'latest'
      DOCKERFILE_PATH:
        description: 'The path to the Dockerfile.'
        required: false
        type: string
        default: 'Dockerfile'
      BUILD_CONTEXT:
        description: 'The build context for the Docker build.'
        required: false
        type: string
        default: '.'
      # Docker Hub specific
      DOCKER_USERNAME:
        description: 'Docker Hub username.'
        required: false
        type: string
      # ECR specific
      AWS_REGION:
        description: 'AWS region for ECR.'
        required: false
        type: string
        default: 'ap-northeast-2'
    secrets:
      # Docker Hub specific
      DOCKER_PASSWORD:
        required: false
      # ECR specific
      AWS_ACCESS_KEY_ID:
        required: false
      AWS_SECRET_ACCESS_KEY:
        required: false

jobs:
  build_and_push:
    runs-on: ubuntu-latest
    steps:
      - name: 'Checkout code'
        uses: actions/checkout@v4

      - name: 'Build and Push Docker Image'
        uses: ./.github/actions/docker-build-and-push
        with:
          REGISTRY_TYPE: ${{ inputs.REGISTRY_TYPE }}
          IMAGE_NAME: ${{ inputs.IMAGE_NAME }}
          IMAGE_TAG: ${{ inputs.IMAGE_TAG }}
          DOCKERFILE_PATH: ${{ inputs.DOCKERFILE_PATH }}
          BUILD_CONTEXT: ${{ inputs.BUILD_CONTEXT }}
          # Docker Hub
          DOCKER_USERNAME: ${{ inputs.DOCKER_USERNAME }}
          DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
          # ECR
          AWS_REGION: ${{ inputs.AWS_REGION }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## 03. Caller Workflows

### 3.1.  Build and Push Project to Docker Hub

```yml
name: '...'

on:
	push:
		branches:
			- main

jobs:
	build:
		uses: DOP2025/centralized-cicd-pipeline/.github/workflows/demo-workflow.yml@main
		with:
			REGISTRY_TYPE: 'dockerhub'
			IMAGE_NAME: 'demo'
			IMAGE_TAG: ${{ github.sha }}
			DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
		secrets:
			# These secrets must be set in the `demo` repository settings
			DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
```

### 3.2. Build and Push Project to ECR

```yml
name: '...'

on:
	push:
		branches:
			- main

jobs:
	build:
		uses: DOP2025/centralized-cicd-pipeline/.github/workflows/demo-workflow.yml@main
		with:
			REGISTRY_TYPE:'ecr'
			IMAGE_NAME: 'demo'
			IMAGE_TAG: ${{github.sha}}
			AWS_REGION: 'ap-northeast-2' # seoul
		secrets:
			AWS_ACCESS_KEY_ID: ${{secrets.AWS_ACCESS_KEY_ID}}
			AWS_SECRETS_ACCESS_KEY: ${{secrets.AWS_SECRETS_ACCESS_KEY}}
```