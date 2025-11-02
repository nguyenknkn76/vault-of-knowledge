
- Install Server > ssh 
- Install Git: https://packages.gitlab.com/gitlab/gitlab-ee
- Settings: ( on linux server, virtual machine or cloud)
	* add host:
	```bash
	vi /etc/hosts
	# ip ur_domain (every fake domain)
	# 192.168.1.100 domain.tech.com
	```
	* fix gitlab external url
	```bash
	vi /etc/gitlab/gitlab.rb
	# external_url 'http://ur_domain'
	
	gitlab-ctl reconfigure
	```
	- access ur_domain
```bash
# username: root
# password: 

cat /etc/gitlab/initial_root_password
```
- Setup:
	- disable create new user
	- disable `defult to auto devops pipeline`
	- change user root password 
- Gitlab Server General
	- Admin
		- Create user, create user pw