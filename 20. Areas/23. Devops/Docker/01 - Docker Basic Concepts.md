---
tags:
  - devops
  - "#docker"
---
## 1. What is Docker? Why use Docker?

**Use case:**

![[assets/01 - Docker Basic Concepts/file-20251117164732522.png]]

## 2. Docker Architecture

![[assets/01 - Docker Basic Concepts/file-20251117165042394.png]]
## 3. How to use Docker?
### 3.1. Install Docker

### 3.2. Use Docker

```bash
docker pull ubuntu:22.04
docker images
docker run --name ubuntu -it ubuntu:22.04
exit

docker ps -a
docker start ubuntu # start with name or container id
docker exec -it ubuntu bash

docker stop ubuntu
docker rm ubuntu 
docker rm ubuntu -f # remove docker container without stoping docker container
docker rm -f ${docker ps -a}
docker rmi ubuntu:22.04
```

```bash
docker run --name nginx -d -p 9999:80 nginx:latest
# -d: run in background
# -p: --public
docker ps 
```

## 4. Dockerfile
## 4.1. What is Dockerfile

```Dockerfile
# pull base image as server 
FROM node:20

# work direction
WORKDIR /app

# copy source from server to container 
# `.` position of Dockerfile
COPY . .

# run bash command
RUN pwd

# env
ENV

# assign container's port
EXPOSE 

# command to run application
CMD 
ENTRYPOINT
```
### 4.2. Optimize Dockerfile mindset?

- non root user
- base image
	- choose right image version of project
	- choose image from official, verified or sponsored source
	- choose base image is `alpine`
- tools to scan image
- multi stages

### Example:  Dockerfile for Backend Java Project

**multi stage**
```Dockerfile
## build stage
FROM maven:3.5.3-jdk-8-alpine as build

WORKDIR /app
COPY . .
RUN mvn install DskipTests=true

## run stage (size of Docker image based on result of last stage)
FROM amazoncorretto:8u402-alpine-jre

WORKDIR /run
COPY --from==build /app/target/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar /run/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar

EXPOSE 8080

ENTRYPOINT java -jar /run/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar
```

```bash
# build image from Dockerfile
docker build -t shoeshop:v1 .
docker images
# run docker from images
docker run --name shoeshop -dp 8888:8080 shoeshop:v1
docker ps -a

# browser: http://localhost:8888
# push images to registry

docker logs shoeshop -f
docker exec -it shoeshop sh 
```

**using alpine server and install needed dependencies**
```Dockerfile
## build stage
FROM maven:3.5.3-jdk-8-alpine as build

WORKDIR /app
COPY . .
RUN mvn install DskipTests=true

## run stage (size of Docker image based on result of last stage)
FROM alpine:3.19

# how to install java 8 in alpine 
RUN apk add openjdk 8

WORKDIR /run
COPY --from==build /app/target/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar /run/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar

EXPOSE 8080

ENTRYPOINT java -jar /run/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar
```

**using non root user**
```Dockerfile
## build stage
FROM maven:3.5.3-jdk-8-alpine as build

WORKDIR /app
COPY . .
RUN mvn install DskipTests=true

## run stage (size of Docker image based on result of last stage)
FROM alpine:3.19

# add user
RUN adduser -D shoeshop
# how to install java 8 in alpine 
RUN apk add openjdk 8

WORKDIR /run
COPY --from==build /app/target/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar /run/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar

# change onwer
RUN chown -R shoeshop:shoeshop /run
USER shoeshop

EXPOSE 8080

ENTRYPOINT java -jar /run/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar
```

### Dockerfile for Frontend Vue Project

```Dockerfile
# build stage
FROM node:18.18-alpine as build

WORKDIR /app
COPY . .
RUN npm install
RUN npm run build

# run stage
FROM nginx:alpine

COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

```bash
docker build -t todolist:v1
docker run --name todolist-v1 -dp 8080:80 todolist:v1
# browser: http://localhost:8080
```

### 5. Install Docker Registry (free :>)

3 methods to deploy registry:
- Docker Hub
- Self-certified (private registry - run with https - free)
- Private registry with harbor (using domain or VPS )

#### Deploy Docker Registry by using Docker Hub

```bash
# How to push images to Dockerhub registry

# login to docker hub
docker login
# enter username, password
# check: .docker/config.json

# format image = `domain/project/repo:tag`
docker tag todolist:v1 nguyentdk02/todolist:v1
# push image to docker hub
docker push nguyentdk02/todolist:v1

# logout
docker logout

# pull image
docker pull nguyentdk02/todolist:v1
```

####  Deploy Docker Registry by using  Private Registry

1 server for registry
```bash
# in `docker registry` server
mkdir registry

# self cert
apt install openssl
mkdir certs data
openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -subj "/CN=192.168.1.100" -addext "subjectAltName = DNS:192.168.1.100,IP:192.168.1.100" -x509 -days 365 -out certs/domain.crt

# run docker compose
docker compose up -d  
```

```yml
# docker-compose.yml
services:
	registry:
		image: registry:2
		restart: always
		container_name: registry-server
		ports:
			- "5000:5000"
		volumes: 
			- ./data:/var/lib/registry
			- ./certs:/certs
		environment:
			REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
			REGISTRY_HTTP_TLS_KEY: /certs/domain.key
			
# access browser: https://<server-ip>:5000/v2/_catelog
```

```bash
# push image to registry
# create certs.d folder(ipof server)
mkdir -p /etc/docker/certs.d/192.168.1.100:5000
cp certs/domain.crt /etc/docker/certs.d/192.168.1.100:5000/ca.crt
systemctl restart docker

# docker login <registry-server-ip:<port>  (basic auth)
docker login 192.168.1.100:5000

# push image to ur registry server
docker tags todolist:v1 192.168.1.100:5000/nguyentdk02/todolist:v1
docker push 192.168.1.100:5000/nguyentdk02/todolist:v1

# pull docker image in registry server from another system
```

####  Deploy Docker Registry by using  Harbor Registry

- Condition: VPS + DNS or AWS EC2
- Create A Records and domain point to registry server IP

```bash
apt update -y
# install certbot
apt install certbot -y
# install harbor
mkdir -p /tools/harbor && cd /tools/harbor

curl -s https://api.github.com/repos/goharbor/harbor/releases/latest | grep browser_download_url | cut -d '"' -f 4 | grep '.tgz$' | wget -i -
# unzip harbor.tgz
tar xvzf harbor-offline-installer*.tgz

cp harbor.yml.tmpl harbor.yml
export DOMAIN="ti3ulonggi4.click" 
export EMAIL="nguyentdk02@sample.com"

# create certs
sudo certbot certonly --standalone -d $DOMAIN --preferred-challenges http --agree-tos -m $EMAIL --keep-until-expiring
```

```harbor.yml
hostname: ti3ulonggi4.click
...
https:
	port: 443
	certificate: /.../fullchain.p 
	private_key: /.../privatekey.pem
```

```bash
# in /tools/harbor/harbor
./prepare
./install.sh
docker compose ps
```

```bash
# push image to harbor registry
# login registry
docker login ti3ulonggi4 
# enter usr, pwd
docker images
docker tag todolist:v1 ti3ulonggi4/nguyentdk02/todolist:v1
```

 > [! note] White list IP for `registry server`, `gitlab server`, etc
 

## 6. Another Components of Docker

### 6.1. Docker volume

```bash
docker run --rm -v `pwd`:/app --workdir="app" maven:3.5.3-jdk-8-alpine mvn install -DskipTests=true
```

```bash
# example with mariadb
mkdir -p /db/mariadb-1

# run container mariadb 
docker run -v /db/mariadb-1:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -p 3307:3306 --name mariadb-1 -d mariadb:v10.6

# check db service
docker ps
netstat -tlpn

# connect to db
mysql -h 192.168.1.110 -P 3307 -u root -p

show databases
create database demo
```

### 6.2. Docker compose

```yml
services:
	db: 
		image: mariadb:10.6
		volumes:
			- /db/mariadb-1:/var/lib/mysql
		environment:
			MYSQL_ROOT_PASSWORD: root
		ports:
			- "3307:3306"
		container_name: mariadb-1
		restart: always
	
	backend:
		image: shoeshop:v1
		ports: 
			- "8081:8080"
		container_name: shoeshop-1
		restart: always
```

 ```bash
 docker ps
 docker compose up -d 
 ```

### 6.3. Docker network

```bash
docker inspect shoeshop-1

# exec to mariadb and ping shoeshop
docker inspect mariadb-1
docker exec -it mariadb-1 sh
whoami

# install ping
apk update
apt install iputils-ping -y
ping <container-shoeshop-1-ip>
```
 