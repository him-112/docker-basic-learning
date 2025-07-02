# Docker Commands Cheat Sheet

## 🐳 Container Operations

### Basic Container Commands
```bash
# Run a container
docker run [OPTIONS] IMAGE [COMMAND]
docker run -d -p 80:80 --name web nginx

# List containers
docker ps                  # Running containers
docker ps -a              # All containers
docker ps -q              # Only container IDs

# Stop/Start containers
docker stop CONTAINER      # Graceful stop
docker kill CONTAINER      # Force stop
docker start CONTAINER     # Start stopped container
docker restart CONTAINER   # Restart container

# Remove containers
docker rm CONTAINER        # Remove stopped container
docker rm -f CONTAINER     # Force remove running container
docker container prune     # Remove all stopped containers
```

### Interactive Containers
```bash
# Run interactive container
docker run -it ubuntu bash
docker run -it --rm alpine sh  # Remove after exit

# Execute commands in running container
docker exec -it CONTAINER bash
docker exec CONTAINER ls /app
```

### Container Information
```bash
# Inspect container
docker inspect CONTAINER
docker logs CONTAINER
docker logs -f CONTAINER      # Follow logs
docker stats CONTAINER        # Resource usage
docker top CONTAINER          # Running processes
```

---

## 📦 Image Operations

### Basic Image Commands
```bash
# List images
docker images
docker images -q              # Only image IDs

# Pull/Push images
docker pull IMAGE[:TAG]
docker push IMAGE[:TAG]

# Remove images
docker rmi IMAGE
docker image prune            # Remove unused images
docker image prune -a         # Remove all unused images
```

### Building Images
```bash
# Build from Dockerfile
docker build -t NAME:TAG .
docker build -t myapp:v1.0 .
docker build -f Dockerfile.prod -t myapp:prod .

# Build with build args
docker build --build-arg VERSION=1.0 -t myapp .

# Multi-platform builds
docker buildx build --platform linux/amd64,linux/arm64 -t myapp .
```

### Image Information
```bash
# Inspect image
docker inspect IMAGE
docker history IMAGE         # Image layers
docker image ls --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

---

## 🔗 Volume Operations

### Volume Commands
```bash
# Create volume
docker volume create VOLUME_NAME

# List volumes
docker volume ls

# Inspect volume
docker volume inspect VOLUME_NAME

# Remove volumes
docker volume rm VOLUME_NAME
docker volume prune           # Remove unused volumes
```

### Using Volumes
```bash
# Named volume
docker run -v VOLUME_NAME:/path/in/container IMAGE

# Bind mount
docker run -v /host/path:/container/path IMAGE
docker run -v $(pwd):/app IMAGE

# Read-only mount
docker run -v /host/path:/container/path:ro IMAGE

# tmpfs mount
docker run --tmpfs /tmp IMAGE
```

---

## 🌐 Network Operations

### Network Commands
```bash
# List networks
docker network ls

# Create network
docker network create NETWORK_NAME
docker network create --driver bridge my-network

# Remove network
docker network rm NETWORK_NAME
docker network prune         # Remove unused networks

# Connect/Disconnect containers
docker network connect NETWORK CONTAINER
docker network disconnect NETWORK CONTAINER
```

### Network Usage
```bash
# Run container on specific network
docker run --network NETWORK_NAME IMAGE

# Run with custom hostname
docker run --hostname myhost --network my-net IMAGE

# Publish ports
docker run -p 8080:80 IMAGE           # Host:Container
docker run -p 127.0.0.1:8080:80 IMAGE # Bind to specific IP
docker run -P IMAGE                    # Publish all exposed ports
```

---

## 🏗️ Docker Compose Commands

### Basic Compose Operations
```bash
# Start services
docker-compose up
docker-compose up -d          # Detached mode
docker-compose up SERVICE     # Start specific service

# Stop services
docker-compose down
docker-compose down -v        # Remove volumes too
docker-compose stop
docker-compose stop SERVICE

# View status
docker-compose ps
docker-compose logs
docker-compose logs -f SERVICE
```

### Compose Management
```bash
# Build services
docker-compose build
docker-compose build SERVICE

# Scale services
docker-compose up -d --scale SERVICE=3

# Execute commands
docker-compose exec SERVICE bash
docker-compose run SERVICE command

# Configuration
docker-compose config         # Validate and view config
docker-compose -f file1.yml -f file2.yml up
```

---

## 🚀 Docker Swarm Commands

### Swarm Initialization
```bash
# Initialize swarm
docker swarm init
docker swarm init --advertise-addr IP

# Join swarm
docker swarm join --token TOKEN MANAGER_IP:2377

# Leave swarm
docker swarm leave
docker swarm leave --force    # Manager node
```

### Node Management
```bash
# List nodes
docker node ls

# Inspect node
docker node inspect NODE

# Update node
docker node update --availability drain NODE
docker node update --label-add storage=ssd NODE
```

### Service Management
```bash
# Create service
docker service create --name web --replicas 3 -p 80:80 nginx

# List services
docker service ls
docker service ps SERVICE

# Scale service
docker service scale SERVICE=5

# Update service
docker service update --image nginx:1.20 SERVICE

# Remove service
docker service rm SERVICE
```

### Stack Management
```bash
# Deploy stack
docker stack deploy -c docker-compose.yml STACK_NAME

# List stacks
docker stack ls
docker stack services STACK_NAME
docker stack ps STACK_NAME

# Remove stack
docker stack rm STACK_NAME
```

---

## 🔧 System Operations

### System Information
```bash
# System info
docker system info
docker version

# Resource usage
docker system df           # Disk usage
docker stats              # Live resource stats

# System events
docker system events
docker system events --filter container=myapp
```

### Cleanup Commands
```bash
# Clean up everything
docker system prune        # Remove unused data
docker system prune -a     # Remove all unused data
docker system prune --volumes  # Include volumes

# Specific cleanup
docker container prune     # Remove stopped containers
docker image prune         # Remove unused images
docker volume prune        # Remove unused volumes
docker network prune       # Remove unused networks
```

---

## 🏷️ Useful Docker Run Options

### Common Options
```bash
-d, --detach              # Run in background
-it                       # Interactive + TTY
--rm                      # Remove container after exit
--name NAME               # Assign container name
-p, --publish HOST:CONTAINER  # Port mapping
-v, --volume HOST:CONTAINER   # Volume mount
-e, --env KEY=VALUE       # Environment variable
--env-file FILE           # Environment file
--network NETWORK         # Connect to network
--restart POLICY          # Restart policy (no|on-failure|always|unless-stopped)
```

### Resource Limits
```bash
--memory 512m             # Memory limit
--cpus 1.5               # CPU limit
--cpu-shares 512         # CPU shares
--oom-kill-disable       # Disable OOM killer
--user 1000:1000         # User ID:Group ID
--workdir /app           # Working directory
```

---

## 🔍 Troubleshooting Commands

### Debugging
```bash
# Enter running container
docker exec -it CONTAINER /bin/bash

# Check container processes
docker top CONTAINER

# View container changes
docker diff CONTAINER

# Export/Import containers
docker export CONTAINER > container.tar
docker import container.tar

# Save/Load images
docker save IMAGE > image.tar
docker load < image.tar

# Copy files
docker cp CONTAINER:/path/file /host/path
docker cp /host/path CONTAINER:/path/
```

### Performance Monitoring
```bash
# Real-time stats
docker stats

# Container resource usage
docker container stats --no-stream

# System resource usage
docker system df
docker system events
```

---

## 💡 Quick Tips

1. **Use specific image tags**: `nginx:1.20` instead of `nginx:latest`
2. **Clean up regularly**: Use `docker system prune` to free space
3. **Use .dockerignore**: Exclude unnecessary files from build context
4. **Multi-stage builds**: Reduce final image size
5. **Health checks**: Add health checks to your containers
6. **Labels**: Use labels for better organization
7. **Environment files**: Use `.env` files for environment variables
8. **Logging**: Configure proper logging drivers for production

---

## 📚 Common Patterns

### Development Setup
```bash
# Hot reload development
docker run -v $(pwd):/app -p 3000:3000 node:18 npm run dev

# Database for development
docker run -d -e POSTGRES_PASSWORD=dev -p 5432:5432 postgres:15
```

### Production Patterns
```bash
# Production with health check
docker run -d --restart unless-stopped \
  --health-cmd="curl -f http://localhost/health" \
  --health-interval=30s \
  -p 80:80 myapp:prod

# Resource-limited container
docker run -d --memory=512m --cpus=1.0 myapp:prod
``` 