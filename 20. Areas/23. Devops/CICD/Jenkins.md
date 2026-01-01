---
tags:
  - devops
  - cicd
  - cicd/jenkins
  - jenkins
---
Host Jenkins with Docker Compose 
Host Jenkins with K8s Cluster

## Host Jenkins with Docker Compose

Host Jenkins server via docker compose 
Access Jenkins UI through exposed port

Manage Jenkins > Install Plugins >
- Pipeline Stage View
- Maven, Java, JDK
- html, css
- Blue Ocean


Jenkins Dashboard > New Item > Pipelines > ... > Configuration
- Pipelines 
	- Pipeline script 
	- Pipeline script in from SCM (recommend)

```Groovy
pipeline {
	agent any
	stages {
		stage('Clone source') {
			steps {
				echo "Setting up the tools"
				git url: 
				
			}
		}
		
		stage('Build') {
			steps {
				echo "Building the application"
			}
		}
		stage('Test') {
			steps {
				echo "Running tests"
			}
		}
		
		stage('Analysis') {
			steps {
				echo "Analysis Application"
			}
		}
		
		stage('Push Docker Image') {
			steps {
				echo "Build & Push Docker Image to Registry"
			}
		}
		
		stage('Deploy') {
			steps {
				echo "Deploying the application"
			}
		}
	}
	post {
		always {
			echo "Pipeline execution completed"
		}
	}
}
```

Step 1: Clone source code

Add credentials:  `username & password`
- username: git_username
- password: git_personal_access_token
-> credentials_Id 

Step 2: Build applications
Install dependencies with `Plugins` > setup in Tools > update Script  

Step 3: Build & Push Docker Image



1. Create EC2 jenkins_worker Agent.
2.  Manage Jenkins > Nodes > New Node > Configure:
	- Remote root directory: `/home/jenkins`
	- Launch method: `Launch agent by connecting it to the controller`
	  -> Connection Scripts
3. On `jenkins_worker` run connection scripts.
4. Choose exact self_hosted_agent_name in groovy code.

## Host
