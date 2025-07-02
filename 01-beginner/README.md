# 🟢 Beginner Level - Docker Basics

## Learning Objectives
- Understand what Docker is and why it's useful
- Learn basic Docker commands
- Create your first Dockerfile
- Run and manage containers
- Work with Docker images

---

## 1. Your First Docker Commands

### Installation Check
```bash
# Check if Docker is installed and running
docker --version
docker info
```

### Basic Container Operations

```bash
# Pull an image from Docker Hub
docker pull hello-world

# Run your first container
docker run hello-world

# Run a container interactively
docker run -it ubuntu bash

# Run a container in the background (detached)
docker run -d nginx

# List running containers
docker ps

# List all containers (including stopped ones)
docker ps -a

# Stop a running container
docker stop <container_id>

# Remove a container
docker rm <container_id>

# Remove all stopped containers
docker container prune
```

### Working with Images

```bash
# List all images
docker images

# Remove an image
docker rmi <image_id>

# Remove all unused images
docker image prune
```

---

## 2. Understanding Ports and Networking

### Port Mapping Examples

```bash
# Run nginx web server on port 8080
docker run -d -p 8080:80 nginx

# Run nginx with a custom name
docker run -d -p 8081:80 --name my-website nginx

# Run multiple containers on different ports
docker run -d -p 8082:80 --name website1 nginx
docker run -d -p 8083:80 --name website2 nginx
```

### Container Inspection

```bash
# Get detailed information about a container
docker inspect <container_name>

# View container logs
docker logs <container_name>

# Follow logs in real-time
docker logs -f <container_name>

# Execute commands in running container
docker exec -it <container_name> bash
```

---

## 3. Environment Variables

```bash
# Run container with environment variables
docker run -e MYSQL_ROOT_PASSWORD=mypassword mysql:8.0

# Run container with multiple environment variables
docker run -d \
  -e POSTGRES_DB=myapp \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=secret123 \
  -p 5432:5432 \
  postgres:15
```

---

## 4. Your First Dockerfile

Create a simple web application to understand how Dockerfiles work.

### Example: Simple HTML Website

**File: `index.html`**
```html
<!DOCTYPE html>
<html>
<head>
    <title>My First Docker Website</title>
    <style>
        body { font-family: Arial, sans-serif; text-align: center; margin-top: 50px; }
        .container { max-width: 600px; margin: 0 auto; }
        h1 { color: #2196F3; }
    </style>
</head>
<body>
    <div class="container">
        <h1>🐳 Welcome to Docker!</h1>
        <p>This website is running inside a Docker container.</p>
        <p>Current time: <span id="time"></span></p>
    </div>
    <script>
        document.getElementById('time').textContent = new Date().toLocaleString();
    </script>
</body>
</html>
```

**File: `Dockerfile`**
```dockerfile
# Use official nginx image as base
FROM nginx:alpine

# Copy our HTML file to nginx default directory
COPY index.html /usr/share/nginx/html/

# Expose port 80 (documentation - nginx already exposes this)
EXPOSE 80

# nginx starts automatically, so no CMD needed
```

### Build and Run Your Image

```bash
# Build the image
docker build -t my-website .

# Run the container
docker run -d -p 8080:80 --name my-site my-website

# Visit http://localhost:8080 in your browser
```

---

## 5. Working with Data

### Temporary Data (Container Filesystem)
```bash
# Create a file inside a container
docker run -it ubuntu bash
# Inside container: echo "Hello Docker" > /tmp/message.txt
# Exit container
# File is lost when container is removed!
```

### Persistent Data with Bind Mounts
```bash
# Create a directory on your host
mkdir ~/my-data

# Run container with bind mount
docker run -it -v ~/my-data:/data ubuntu bash
# Inside container: echo "This persists!" > /data/message.txt
# Exit container - file remains on host!
```

---

## 🎯 Practice Exercises

### Exercise 1: Run Multiple Web Servers
1. Run 3 nginx containers on ports 8001, 8002, 8003
2. Give each container a unique name
3. Verify all are running with `docker ps`

**Solution:**
```bash
docker run -d -p 8001:80 --name web1 nginx
docker run -d -p 8002:80 --name web2 nginx
docker run -d -p 8003:80 --name web3 nginx
```

### Exercise 2: Create a Custom Image
1. Create an HTML file with your name
2. Write a Dockerfile to serve it with nginx
3. Build and run the image

**Solution:** See example above, replace content in `index.html`

### Exercise 3: Database Container
1. Run a MySQL container with environment variables
2. Connect to it and create a simple table
3. Verify data persists after container restart

**Solution:**
```bash
# Run MySQL
docker run -d \
  -e MYSQL_ROOT_PASSWORD=password123 \
  -e MYSQL_DATABASE=testdb \
  -p 3306:3306 \
  --name mysql-test \
  mysql:8.0

# Connect to MySQL
docker exec -it mysql-test mysql -u root -p

# Inside MySQL: CREATE TABLE users (id INT, name VARCHAR(50));
```

---

## 📋 Key Takeaways

1. **Containers are lightweight**: They share the host OS kernel
2. **Images are templates**: Used to create containers
3. **Port mapping**: `-p host_port:container_port`
4. **Environment variables**: Use `-e` flag to set them
5. **Data persistence**: Use volumes or bind mounts for important data
6. **Container lifecycle**: Create → Run → Stop → Remove

---

## 🔗 What's Next?

Move to **02-intermediate** to learn about:
- Docker Compose for multi-container applications
- Advanced Dockerfile techniques
- Volume management
- Container networking

---

## 💡 Common Beginner Mistakes to Avoid

1. **Not using specific image tags**: Use `nginx:1.21` instead of `nginx:latest`
2. **Running containers as root**: Always create non-root users in production
3. **Not cleaning up**: Use `docker system prune` to clean unused resources
4. **Forgetting port mapping**: Your app won't be accessible without `-p`
5. **Not using .dockerignore**: Include unnecessary files in build context 