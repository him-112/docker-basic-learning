# Docker Learning Guide - Interview Preparation

Welcome to your comprehensive Docker learning journey! This guide covers everything you need to know for Docker interviews, from beginner to advanced concepts.

## 📚 Learning Path

### 🟢 Beginner Level
1. [What is Docker?](#what-is-docker)
2. [Key Concepts](#key-concepts)
3. [Basic Commands](#basic-commands)
4. [Your First Container](#your-first-container)
5. [Working with Images](#working-with-images)

### 🟡 Intermediate Level
1. [Dockerfile Best Practices](#dockerfile-best-practices)
2. [Docker Compose](#docker-compose)
3. [Volumes and Data Management](#volumes-and-data-management)
4. [Networking](#networking)
5. [Multi-stage Builds](#multi-stage-builds)

### 🔴 Advanced Level
1. [Docker Swarm](#docker-swarm)
2. [Security Best Practices](#security-best-practices)
3. [Performance Optimization](#performance-optimization)
4. [Production Deployment](#production-deployment)
5. [Monitoring and Logging](#monitoring-and-logging)

---

## 🟢 Beginner Level

### What is Docker?

Docker is a **containerization platform** that packages applications and their dependencies into lightweight, portable containers.

**Why Docker?**
- **Consistency**: "It works on my machine" → "It works everywhere"
- **Isolation**: Applications run in separate environments
- **Efficiency**: Lightweight compared to virtual machines
- **Scalability**: Easy to scale up/down
- **DevOps**: Streamlines development to production workflow

### Key Concepts

#### 1. **Container**
- A running instance of an image
- Isolated process with its own filesystem, network, and resources
- Think of it as a "lightweight VM"

#### 2. **Image**
- A read-only template used to create containers
- Contains application code, runtime, libraries, and dependencies
- Built from Dockerfile instructions

#### 3. **Dockerfile**
- A text file with instructions to build an image
- Each instruction creates a new layer in the image

#### 4. **Registry**
- A service for storing and distributing images
- Docker Hub is the default public registry

### Basic Commands

```bash
# Check Docker version
docker --version

# Pull an image from registry
docker pull nginx

# List all images
docker images

# Run a container
docker run nginx

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop a container
docker stop <container_id>

# Remove a container
docker rm <container_id>

# Remove an image
docker rmi <image_id>
```

### Your First Container

Let's run a simple web server:

```bash
# Run nginx web server
# -d: run in background (detached mode)
# -p: map port 8080 on host to port 80 in container
# --name: give container a friendly name
docker run -d -p 8080:80 --name my-nginx nginx

# Visit http://localhost:8080 in your browser
```

---

## 🟡 Intermediate Level

### Dockerfile Best Practices

#### Basic Dockerfile Structure

```dockerfile
# Use official base image (always specify version)
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files first (for better caching)
COPY package*.json ./

# Install dependencies
RUN npm install --only=production

# Copy application code
COPY . .

# Create non-root user for security
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001
USER nodejs

# Expose port (documentation purpose)
EXPOSE 3000

# Start the application
CMD ["npm", "start"]
```

### Docker Compose

Docker Compose manages multi-container applications using a YAML file.

#### Basic docker-compose.yml

```yaml
version: '3.8'

services:
  # Web application
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=database
    depends_on:
      - database
    volumes:
      - ./logs:/app/logs

  # Database
  database:
    image: postgres:15
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

#### Common Compose Commands

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f web

# Scale a service
docker-compose up -d --scale web=3

# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

### Volumes and Data Management

#### Types of Volumes

1. **Named Volumes** (Recommended for persistent data)
```bash
docker volume create my-data
docker run -v my-data:/data nginx
```

2. **Bind Mounts** (Good for development)
```bash
docker run -v /host/path:/container/path nginx
```

3. **tmpfs Mounts** (Temporary data in memory)
```bash
docker run --tmpfs /tmp nginx
```

### Networking

#### Network Types

```bash
# List networks
docker network ls

# Create custom network
docker network create my-network

# Run container on custom network
docker run --network my-network nginx

# Connect running container to network
docker network connect my-network container_name
```

---

## 🔴 Advanced Level

### Multi-stage Builds

Optimize image size by using multi-stage builds:

```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm install --only=production
COPY --from=builder /app/dist ./dist
USER node
CMD ["npm", "start"]
```

### Security Best Practices

1. **Use official base images**
2. **Don't run as root**
3. **Scan images for vulnerabilities**
4. **Use specific image tags**
5. **Minimize image layers**
6. **Use .dockerignore**

### Performance Optimization

#### Image Size Optimization
- Use Alpine Linux base images
- Multi-stage builds
- Remove unnecessary packages
- Combine RUN commands

#### Runtime Optimization
- Set appropriate resource limits
- Use health checks
- Optimize application startup time

---

## 🎯 Common Interview Questions

### Beginner Questions
1. **What is Docker and how does it differ from VMs?**
2. **Explain the difference between an image and a container**
3. **What is a Dockerfile?**
4. **How do you share data between containers?**

### Intermediate Questions
1. **Explain Docker Compose and its benefits**
2. **What are Docker volumes and their types?**
3. **How does Docker networking work?**
4. **What are multi-stage builds and why use them?**

### Advanced Questions
1. **How do you secure Docker containers in production?**
2. **Explain Docker Swarm vs Kubernetes**
3. **How do you monitor Docker containers?**
4. **What are the best practices for Docker in CI/CD?**

---

## 🚀 Quick Start Guide

1. **Install Docker**: Visit [docker.com](https://docker.com) and install Docker Desktop
2. **Verify Installation**: `docker --version`
3. **Run First Container**: `docker run hello-world`
4. **Practice Examples**: Work through the examples in each folder
5. **Build Your Own**: Create a simple application and containerize it

---

## 📁 Repository Structure

```
├── 01-beginner/          # Basic examples and exercises
├── 02-intermediate/      # Docker Compose and advanced concepts
├── 03-advanced/          # Production-ready examples
├── exercises/            # Hands-on practice problems
└── cheat-sheets/         # Quick reference guides
```

Let's start your Docker journey! 🐳 