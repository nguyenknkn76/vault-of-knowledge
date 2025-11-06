## Overview

- **Infra:** 3 EC2 (Amazon Linux), 1 manager + 2 workers
- **Ports:** 80 open to internet; Swarm ports open **between nodes**
- **Registry:** Docker Hub (public repo recommended)
- **Result:** Access the app at `http://<ANY_NODE_PUBLIC_IP>`
---
## 0. Security Group

Open:
- **Internet → nodes:** `80/tcp` (later 443 if you add TLS)
- **Nodes ↔ nodes (internal only):** `2377/tcp`, `7946/tcp`, `7946/udp`, `4789/udp`
- **SSH:** `22/tcp` from your IP

---
## 1. Install Docker on each EC2

```bash
# ssh into each instances
# Detect OS
cat /etc/os-release

# Amazon Linux 2023
if grep -q "Amazon Linux 2023" /etc/os-release; then
  sudo dnf update -y
  sudo dnf install -y docker
fi

# Amazon Linux 2
if grep -q "Amazon Linux 2" /etc/os-release; then
  sudo yum update -y
  sudo amazon-linux-extras enable docker
  sudo yum install -y docker
fi

# Enable/start Docker and grant ec2-user group access
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ec2-user
newgrp docker

# (Optional) set friendly hostnames
# manager: sudo hostnamectl set-hostname manager-1
# worker1: sudo hostnamectl set-hostname worker-1
# worker2: sudo hostnamectl set-hostname worker-2
```

## 2.  Init the Swarm (master)
```bash
# on master node
# Get manager private IP
MANAGER_IP=$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)
echo $MANAGER_IP

# Initialize swarm using private IP
docker swarm init --advertise-addr $MANAGER_IP

# If you need tokens later
docker swarm join-token worker
docker swarm join-token manager
```

## 3. Join the workers
```bash
# on each worker 
docker swarm join --token <WORKER_TOKEN> <MANAGER_IP>:2377

# on manager -> check -> all nodes should be `Ready`
docker node ls
```

## 4. Create overlay network (master)
```bash
docker network create --driver overlay --attachable app-net
```

## 5. Prepare the repository (local or manager)

```txt
# /ops/nginx.conf
server {
  listen 80;
  server_name _;
  root /usr/share/nginx/html;
  index index.html;

  location / {
    try_files $uri /index.html;
  }
}
```

```Dockerfile
# dockerfile at repo root
# --- build stage ---
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
# If your build needs an API URL at build time, keep ARG/ENV. Else remove them.
ARG VITE_API_URL
ENV VITE_API_URL=$VITE_API_URL
RUN npm run build

# --- runtime stage ---
FROM nginx:1.27-alpine
RUN rm -f /etc/nginx/conf.d/default.conf
COPY ./ops/nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
```
## 6. Build & push image to Docker Hub
```bash
# run from local device
docker login  # enter Docker Hub creds or a token

DOCKERHUB_USER=<your_dockerhub_username>
REPO=quickbite-frontend
TAG=$(date +%Y%m%d-%H%M)             # e.g., 20251016-1420
IMAGE=$DOCKERHUB_USER/$REPO:$TAG

# If you don't need API URL at build time, remove the --build-arg line
docker build \
  --build-arg VITE_API_URL=https://example.invalid \
  -t $IMAGE .

# (Optional) also tag latest
docker tag $IMAGE $DOCKERHUB_USER/$REPO:latest

docker push $IMAGE
# (Optional)
docker push $DOCKERHUB_USER/$REPO:latest

echo "IMAGE=$IMAGE"   # keep this value for the stack.yml
```

> [!tip] : Public Docker Hub repositories
> Make the Docker Hub repo public so workers can pull without extra auth.
> If it’s **private**, you can deploy with `--with-registry-auth` (below) or `docker login` on each node.

## 7. Create the Swarm stack (master)
```yml
# stack.yml
networks:
  app-net:
    external: true

services:
  frontend:
    image: <DOCKERHUB_USER>/quickbite-frontend:<TAG>  # replace with your image
    networks: [app-net]
    ports:
      - "80:80"                 # Swarm routing mesh; any node's :80 works
    deploy:
      replicas: 3               # one per node
      update_config:
        parallelism: 1
        delay: 5s
        order: start-first
      restart_policy:
        condition: on-failure
        env:
          api_gateway_url: ga
```
## 8. Deploy
```bash
# public docker hub repo
docker stack deploy -c stack.yml quickbite

# private docker hub repo
docker stack deploy -c stack.yml quickbite --with-registry-auth
```

## 9. Verify &  access
```bash
docker stack services quickbite
docker service ps quickbite_frontend
docker service logs -f quickbite_frontend

# open in browser: http:<any_node_public_ip> -> can see the application.
(Routing mesh with load-balance to replicas across worker)
```

## 10. Update / Rollback / Scale (frontend only)
```bash
# After pushing a new tag (repeat Step 6):
docker service update --image $DOCKERHUB_USER/$REPO:<NEW_TAG> quickbite_frontend

# Roll back if needed
docker service rollback quickbite_frontend

# Scale (if you add more workers)
docker service scale quickbite_frontend=5
```

## 11. Quick troubleshooting 
- **Security Group:** `80/tcp` open to internet; Swarm ports open between nodes.
- **Nodes ready:** `docker node ls` → all `Ready`.
- **Tasks running:** `docker service ps quickbite_frontend` → `Running` (not `Rejected/Pending`).
- **Private repos:** use `-with-registry-auth` or run `docker login` on all nodes.
- **Nginx 404 on deep links:** ensure `try_files $uri /index.html;` in `ops/nginx.conf`.

## 12. Clean up (done or want to reset)
### 12.1. App level cleanup (keep the swarm)
```bash
# on manager
# Remove the frontend stack
docker stack rm quickbite

# Watch until all services/tasks disappear
watch -n 1 'docker stack ls; echo; docker service ls; echo; docker ps'

# (Optional) remove the overlay network if you created it manually
docker network rm app-net 2>/dev/null || true

# (Optional) prune unused containers, networks, images (be careful)
docker system prune -f
```

### 12.2. Cluster reset (keep Docker, keep EC2s)
```bash
# drain worker: moves task off before leaving
docker node ls
docker node update --availability drain <worker-1-name>
docker node update --availability drain <worker-2-name>

# each worker leave the swarm 
# run on each worker
docker swarm leave

# remove drained workers from the master 
# run on master
docker node rm <worker-1-name> <worker-2-name>

# remove master from swarm
docker swarm leave --force

# (optional) remove the app network
docker network rm app-net 2>/dev/null || true

# (optional) prune local artifacts
# on each node
docker system prune -af   # removes stopped containers, unused images/networks/build cache
```