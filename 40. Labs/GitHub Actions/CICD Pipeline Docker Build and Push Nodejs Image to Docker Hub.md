---
tags:
  - devops
  - git
  - cicd/git
  - docker
---
```yml
jobs:
	docker-build:
		runs-on: ubuntu-latest
		permissions: 
			contents: read
	
	steps:
		- name: Checkout Repository
		  uses: actions/checkout@v4
		  
		# use to build muti platforms (AMD64, ARM64)
		- name: Setup QEMU
		  uses: docker/setup-qemu-action@v3
		
		- name: Setup Docker Buildx
		  uses: docker/setup-buildx-action@v3
		  
		- name: Login to DockerHub
		  uses: docker/login-action@v3
		  with:
			  username: ${{secret.DOCKERHUB_USERNAME}}
			  password: ${{secret.DOCKERHUB_PASSWORD}} # or `DOCKERHUB_TOKEN`
		
		- name: Build and Push
		  uses: docker/build-push-action@v6
		  with:
			  context: .
			  push: true
			  # <username>/<repository_name>:<tag>
			  tags: | 
				  ${{ secrets.DOCKERHUB_USERNAME }}/${{ github.event.repository.name }}:latest 
				  ${{ secrets.DOCKERHUB_USERNAME }}/${{ github.event.repository.name }}:${{ github.sha }}
			  # Caching Strategy: GitHub Actions Cache API (GHA)
			  cache-from: type=gha 
			  cache-to: type=gha,mode=max
```