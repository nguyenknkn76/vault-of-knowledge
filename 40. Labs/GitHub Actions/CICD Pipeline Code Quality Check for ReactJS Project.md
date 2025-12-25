---
tags:
  - devops
  - git
  - cicd/git
  - cicd
---

```yml
name: CI/CD Reactjs Project

# Trigger
on:
	push: 
		branches: ["main", "dev"]
		paths-ignore: # just update documents -> skip pipeline
			- '**.md'  
			- 'docs/**'
	pull_requests:
		branches: ["main"]
	workflow_dispatch:

# assign permission for GITHUB_TOKEN (security best practice)
permissions:
	contents: read

jobs:
	code_quality_check:
		name: Build & Test
		runs-on: ubuntu_latest
		env:
			DOCKERHUB_USERNAME: ${{secrets.DOCKERHUB_USERNAME}}
			DOCKERHUB_PASSWORD: ${{secrets.DOCKERHUB_PASSWORD}}
			
		steps:
			# Clone source code to runner 
			- name: Checkout Repository 
			  uses: actions/checkout@v4
			  with:
				  fetch-depth: 0 # default (1) just get latest commit
			
			# Install runtime
			- name: Setup Nodejs environment
			  uses: actions/setup-node@v4
			  with: 
				  node-version: '22'
				  cache: 'npm' # auto config cache for npm 
			
			# Install dependencies
			- name: Install dependencies
			  run: npm run ci # Clean install
			
			- name: Lint code
			  run: npm run lint
			  
			- name: Run tests
			  run: npm run test
			  
			- name: Build Project
			  run: npm run build
```
