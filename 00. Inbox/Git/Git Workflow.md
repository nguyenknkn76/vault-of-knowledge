![[file-20251102113018961.png]]

- setup
	- create repo
	- create branch: main > develop > feature/frontend/login > stagging
	- create user > add user to project 
		- role 
			- devops: maintainer
			- dev: developer 
- settings: 
	- default branch
	- protected branch: 
		- prevent dev can create merge request, and allow that request.
		- main: 
			- merge: maintainer
			- push: no one
*  dev
	*  login > clone src > switch to feature branch > develop
	*  create merge request -> maintainer: approve and merge 
* devops
	* ex update config
* tag