- Concepts
	- Continuous Integration
	- Continuous deployment
	- Continuous delivery
- Tools:
	- Gitlab CI/CD
	- Jenkins

## Gitlab CI/CD (Continuous Deployment)

### 1. Install automation tools
- Install Gitlab Runner on Gitlab Server (Ubuntu)
	- Install
		- Check version 
		```bash
		gitlab-runner -v
		```
		- On server: user gitlab-runner
		```bash
		vi /etc/passwd
		```
- register
	- run command
		```bash
		gitlab-runner register
		# url: ur_gitlab_server_domain.com
		# token: registration token in gitlabserver.com > settings > CI/CD
		# description: = name_of_ server 
		# tags (name of runner): = name_of_server
		# executor: shell
		
		vi /etc/gitlab-runner/config.toml
		# concurrent = 4 (number of projects can use this runner) 
		``` 
- run
	```bash
	nohup gitlab-runner run --working-directory /home/gitlab-runner/ --config /etc/gitlab-runner/config.toml --service gitlab-runner --user gitlab-runner 2>&1 & [1] 4975
	
	ls
	# nohup.out snap
	
	ps -ef | grep gitlab-runner
 	```
-  setting runner: Gitlab Server > Settings > CI/CD > runner (abcxyz)> edit
	- active
	- protected
### 2. Write config file
* Permission for user gitlab-runner
```bash
mkdir /datas/project_name

visudo
# User priviledge specification
# root ALL=(ALL:ALL) ALL

# add this line:
# gitlab-runner ALL=(ALL) NOPASSWD: /bin/cp*
# gitlab-runner ALL=(ALL) NOPASSWD: /bin/chown*
# gitlab-runner ALL=(ALL) NOPASSWD: /bin/su project_user*
```
* Write `.gitlab-ci.yml`
```yaml
variables:
	projectname: project_name
	projectuser: project_user 
	version: 0.0.1-SNAPSHOT 
	projectpath: datas/$projectuser/
stages:
	- build
	- deploy
	- checklog
build
	stage: build
	variables:
		GIT_STRATEGY: clone
	script:
		- whoami
		- pwd
		- ls
		- mvn install -DskipTests=true 
	tags: 
		- runner_name
	only: 
		- tags
deploy
	stage: deploy
	variables:
		GIT_STRATEGY: none
	script: 
		- sudo cp target/project_name-0.0.1-SNAPSHOT.jar /dats/project_name
		- sudo chown -R project_user. /datas/project_name
		- sudo su project_user -c "kill -9 $(ps -ef| grep project_name-0.0.1-SNAPSHOT.jar | grep -v grep | awk '{print $2}')"
		- sudo su project_user -c "cd /datas/project_name/; nohup java -jar target/project_name-0.0.1-SNAPSHOT.jar 2>&1 &"
		#- java -jar target/project_name-0.0.1-SNAPSHOT.jar
	tags:
		- runner_name
	only:
		- tags  # only rebuild project when create new tags 

deploy
	stage: checklog
	variables:
		GIT_STRATEGY: none
	script: 
		- sudo su project_user -c "cd $projectpath; tail -n 10000 nohup.out"
	tags:
		- runner_name
	only:
		- tags 
```
- commit 
- check pipeline: gitlab server > CI/CD > Pipelines
> [!note] each state gitlab server delete all old src code on server and clone new code 
## Gitlab CI/CD (Continuous Delivery)

 Write `.gitlab-ci.yml`
```yaml
variables:
	projectname: project_name
	projectuser: project_user 
	version: 0.0.1-SNAPSHOT 
	projectpath: datas/$projectuser/
stages:
	- build
	- deploy
	- checklog
build
	stage: build
	variables:
		GIT_STRATEGY: clone
	script:
		- whoami
		- pwd
		- ls
		- mvn install -DskipTests=true 
	tags: 
		- runner_name
	only: 
		- tags
deploy
	stage: deploy
	variables:
		GIT_STRATEGY: none
	when: manual # deploy manual 
	script: 
		- >
		  if ["$GITLAB_USER_LOGIN"=="ur_username"]; then
			sudo cp target/project_name-0.0.1-SNAPSHOT.jar /dats/project_name
			sudo chown -R project_user. /datas/project_name
			sudo su project_user -c "kill -9 $(ps -ef| grep project_name-0.0.1-SNAPSHOT.jar | grep -v grep | awk '{print $2}')"
			sudo su project_user -c "cd /datas/project_name/; nohup java -jar target/project_name-0.0.1-SNAPSHOT.jar 2>&1 &"
			else
				echo "Poermission denied"
				exit 1
			fi
	tags:
		- runner_name
	only:
		- tags  # only rebuild project when create new tags 

deploy
	stage: checklog
	variables:
		GIT_STRATEGY: none
	when: manual # deploy manual
	script: 
		- sudo su project_user -c "cd $projectpath; tail -n 10000 nohup.out"
	tags:
		- runner_name
	only:
		- tags 
# commit msg
# "config(pipeline): ... description ... "

purpose of manual: deploy to higher environment -> your source have be scan by clean code tools, check security tools and someone approve. 
 -> config: who can run this pipeline (using this condition `$GITLAB_USER_LOGIN` and `$GITLAB_USER_ID`)