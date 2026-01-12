---
tags:
  - devops
  - git
  - cicd
---

## Project Structure

- root
	- actions
		- determine-language
			- action.yml
		- build-java
			- action.yml
		- build-dotnet
		- setup-java
		- setup-maven
	- workflows
		- pr-opened-workflows.yml
		- pr-closed-workflows.yml
		- manually-trigger-workflows.yml


> [!note] Read more about:
> - `compose github actions`
> - `github resuable workflows`

input 
project name
service name 
image tag


> [!note] Assign permission for a repository access another repository

Settíng > Actions > General > Actions permissions > enable `Allow <org-name> actiosn and reusable workflows`

```yml
```