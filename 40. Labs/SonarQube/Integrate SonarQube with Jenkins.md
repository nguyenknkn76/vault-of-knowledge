---
tags:
  - jenkins
  - cicd
  - sonarqube
  - devops
---
## 00. Lab

| No. | Resource | Name               | IP  | Note        |     |
| --- | -------- | ------------------ | --- | ----------- | --- |
| 01  | EC2      | dop10-jk-master    |     | `t3.medium` |     |
|     |          | dop10-jk-agent     |     | `t3.medium` |     |
|     |          | dop10-sonar-server |     | `t3.micro`  |     |

## 01. Setup Jenkins Server

[[../Jenkins/Integration Jenkins Pipeline with Git Workflows|Integration Jenkins Pipeline with Git Workflows]]

## 02. Setup SonarQube Server via Docker Compose

```yml
sevices:
  sonarqube:
    image: sonarqube:community
    container_name: sonarqube
    depends_on:
      db:
        condition: service_healthy
    ports:
      - "9000:9000"
    networks:
      - sonarnet
    environment:
      - SONAR_JDBC_URL=jdbc:postgresql://db:5432/sonar
      - SONAR_JDBC_USERNAME=sonar
      - SONAR_JDBC_PASSWORD=sonar
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
    restart: always

  db:
    image: postgres:15
    container_name: postgres
    networks:
      - sonarnet
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
      - POSTGRES_DB=sonar
    volumes:
      - postgresql_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U sonar"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: always

networks:
  sonarnet:
    driver: bridge

volumes:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  postgresql_data:
```

```bash
docker compose up -d
# Access this url: http://<ec2_public_ip>:9000
```

## 03. SonarQube Project Configuration

1. Project > Create Project
	- Project Name: `shopsquare-frontend`
	- Project Key: `shopsquare-frontend`
	- Branch: `main` ( Just main scan 1 branch for `SonarQube Community Version`)
2. Setup Quality Assurance Rules: Issues, Rules, Quality Profiles, Quality Gates.
3. Create Token: Administrator icon  > My Account > Security > Generate Token.

## 04. Jenkins Setup 

1. Install `SonarQube Scanner` Plugin.
2. Create Credentials: 
	- Secret: SonarQube Token
	- ID: `sonar-credentials`
	- Kind: `Secret Text`
3. Setup **SonarQube Installer**: Dashboard > Manage Jenkins > **Tools** > 
	- `SonarQube Scanner installation`
		- Name: `SonarQubeScanner`
		- Install from Maven > Version > ...
4. Setup **SonarQube Server**: Dashboard > Manage Jenkins > **Settings** > 
	- `SonarQube Server`
		- Name: `SonarQubeServer`
		- Server URL: `http://<sonarqube-server-ec2-public-ip>:9000`
		- Server authentication token: `sonar-credentials`

## 05. Jenkins Pipeline 

```Groovy
pipeline {  
    agent any  
    
    environment {
	    SONAR_HOST_URL = 'http://<sonarqube-server-ec2-public-ip>:9000'
		SONAR_AUTH_TOKEN = credentials('sonar-credentials')
		scannerHome = tool 'SonarQubeScanner'
    }
    stages {  
        stage('Clone Source Code') {  
            steps {  
                checkout scm
            }  
        }  
        stage('Code Analysis') {  
            steps {  
				sh "${scannerHome}/bin/sonar-scanner \
					-Dsonar.projectKey=shopsquare-frontend \
					-Dsonar.projectName=shopsquare-frontend \
					-Dsonar.sources=. \
					-Dsonar.host.url=$SONAR_HOST_URL \
					-Dsonar.login=$SONAR_AUTH_TOKEN"
            }  
        }  
    }  
}
```

## Troubleshoot

> [!bug] WARNING: Failed to send back a reply to the request UserRequest:hudson.FilePath$IsDirectory@182e04a1 java.io.IOException: java.lang.InterruptedException
> - Java Version Conflict: `java --version`
> - Lack of Resources: Assign more resources for Jenkins Agent (e.g. `t3.medium`) 

## References

[How to Integrate SonarQube with Jenkins](https://www.geeksforgeeks.org/devops/how-to-integrate-sonarqube-with-jenkins/)
