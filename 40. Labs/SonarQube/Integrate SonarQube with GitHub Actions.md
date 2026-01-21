---
tags:
  - devops
  - cicd
  - cicd/git
  - git
  - sonarqube
---

| No. | Resource | Name               | IP  | Note       |     |
| --- | -------- | ------------------ | --- | ---------- | --- |
| 01  | EC2      | dop10-sonar-server |     | `t3.micro` |     |
|     |          | dop10-gh-agent     |     | `t3.small` |     |

## Setup SonarQube

## Setup GitHub Actions

Create GitHub Secrets:
- SONAR_TOKEN
- SONAR_HOST_URL

Configure GitHub Actions:
```yaml
name: ...
on:
	...
env:
	SONAR_TOKEN: ${{secrets.SONAR_TOKEN}}
	SONAR_HOST_URL: ${{secrets.SONAR_HOST_URL}}

jobs:
	sonarqube_integration:  
		name: SonarQube Integration  
		runs-on: ubuntu-latest  
		steps:
			- name: Checkout Repo
			  uses: actions/checkout@v6
			  with:
			  # Disabling shallow clone is recommended for improving relevancy of reporting
				  fetch-depth: 0
				   
			- name: Analyze Source Code with SonarQube
			  uses: sonarsource/sonarqube-scan-action@master
			  env:
				  SONAR_TOKEN: ${{ env.SONAR_TOKEN }}
				  SONAR_HOST_URL: ${{ env.SONAR_HOST_URL }}
			  with:
				  args: >
					  -Dsonar.projectKey=shopsquare-frontend
					  -Dsonar.projectName=shopsquare-frontend
					  -Dsonar.sources=.
```

> [!bug] java.io.IOException: No space left on device
> - Reason: SonarScanner try to write report file (`activerules.pb`) to temp folder `.scannerwork` but **hard disk of ubuntu server is full**.
> 
```bash
# in self hosted runner 
df -h
rm -rf /home/ubuntu/actions-runner/_work/*
docker system prune -a --volumes -f
sudo journalctl --vacuum-time=1d 
```

> [!note] Config Auto Clean in Workflows
```yml
- name: Clean up previous builds 
  run: | 
	  echo "Cleaning up workspace..." 
	  rm -rf ${{ github.workspace }}/*


- name: Free Disk Space (Ubuntu)
  uses: jlumbroso/free-disk-space@main
  with:
    # Optimize: delete unused components to free ~20-30GB memory
    tool-cache: true
    android: true
    dotnet: true
    haskell: true
    large-packages: true
    docker-images: true
    swap-storage: true

- name: Free Disk Space 
  run: | 
	  sudo rm -rf /usr/share/dotnet 
	  sudo rm -rf /usr/local/lib/android 
	  sudo rm -rf /opt/ghc 
	  sudo docker image prune -af 
	  df -h
```
## References

[Integrate SonarQube in GitHub Actions](https://medium.com/@s.mehrotrasahil/integrate-sonarqube-in-github-actions-d89eafc7fd69)
