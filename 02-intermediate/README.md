# 🟡 Intermediate Level - Docker Compose & Advanced Concepts

## Learning Objectives
- Master Docker Compose for multi-container applications
- Understand Docker volumes and data persistence
- Learn container networking concepts
- Write optimized Dockerfiles with best practices
- Implement multi-stage builds

---

## 1. Docker Compose Fundamentals

Docker Compose lets you define and run multi-container applications using a YAML file.

### Why Docker Compose?
- **Simplicity**: Define complex applications in one file
- **Consistency**: Same environment across development, testing, and production
- **Service dependencies**: Automatically handle container startup order
- **Scaling**: Easily scale services up or down

### Basic docker-compose.yml Structure

```yaml
version: '3.8'

services:
  service-name:
    image: image-name
    # OR
    build: .
    ports:
      - "host-port:container-port"
    environment:
      - KEY=value
    volumes:
      - host-path:container-path
    depends_on:
      - other-service

volumes:
  volume-name:

networks:
  network-name:
```

---

## 2. Full-Stack Web Application Example

Let's build a complete web application with:
- **Frontend**: React application
- **Backend**: Node.js API
- **Database**: PostgreSQL
- **Cache**: Redis

### Project Structure
```
web-app/
├── docker-compose.yml
├── frontend/
│   ├── Dockerfile
│   └── (React app files)
├── backend/
│   ├── Dockerfile
│   └── (Node.js API files)
└── database/
    └── init.sql
```

### docker-compose.yml
```yaml
version: '3.8'

services:
  # Frontend React Application
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:5000
    depends_on:
      - backend
    volumes:
      # Enable hot reload in development
      - ./frontend/src:/app/src

  # Backend Node.js API
  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgresql://admin:secret@database:5432/myapp
      - REDIS_URL=redis://cache:6379
      - NODE_ENV=development
    depends_on:
      - database
      - cache
    volumes:
      # Mount source code for development
      - ./backend:/app
      - /app/node_modules

  # PostgreSQL Database
  database:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
    volumes:
      # Persist data
      - postgres_data:/var/lib/postgresql/data
      # Initialize database
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"

  # Redis Cache
  cache:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

# Named volumes for data persistence
volumes:
  postgres_data:
  redis_data:
```

### Essential Compose Commands

```bash
# Start all services in background
docker-compose up -d

# View logs from all services
docker-compose logs

# View logs from specific service
docker-compose logs -f backend

# Scale a service
docker-compose up -d --scale backend=3

# Execute command in running service
docker-compose exec backend bash

# Stop all services
docker-compose down

# Stop and remove volumes (DATA LOSS!)
docker-compose down -v

# Rebuild images
docker-compose build

# Pull latest images
docker-compose pull
```

---

## 3. Docker Volumes Deep Dive

### Volume Types Comparison

| Type | Use Case | Performance | Portability |
|------|----------|-------------|-------------|
| Named Volumes | Production data storage | High | High |
| Bind Mounts | Development, config files | Medium | Low |
| tmpfs Mounts | Temporary data, secrets | Highest | N/A |

### Named Volumes (Recommended for Production)

```yaml
services:
  app:
    image: myapp
    volumes:
      - app_data:/data
      - app_logs:/var/log

volumes:
  app_data:
    driver: local
  app_logs:
    driver: local
```

### Bind Mounts (Good for Development)

```yaml
services:
  app:
    image: myapp
    volumes:
      # Absolute path
      - /host/path:/container/path
      # Relative path
      - ./src:/app/src
      # Read-only mount
      - ./config:/app/config:ro
```

### Volume Management Commands

```bash
# List volumes
docker volume ls

# Create volume
docker volume create my-volume

# Inspect volume
docker volume inspect my-volume

# Remove volume
docker volume rm my-volume

# Remove unused volumes
docker volume prune
```

---

## 4. Container Networking

### Network Types

#### Bridge Network (Default)
```bash
# Create custom bridge network
docker network create my-network

# Run containers on same network
docker run --network my-network --name app1 nginx
docker run --network my-network --name app2 nginx

# Containers can communicate using container names
# app1 can reach app2 at http://app2:80
```

#### Host Network
```bash
# Container uses host's network stack
docker run --network host nginx
# Container is accessible at host's IP directly
```

#### Docker Compose Networking

```yaml
version: '3.8'

services:
  web:
    image: nginx
    networks:
      - frontend
      - backend

  api:
    image: myapi
    networks:
      - backend

  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access
```

---

## 5. Advanced Dockerfile Techniques

### Multi-stage Build Example

```dockerfile
# Stage 1: Build stage
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install all dependencies (including dev dependencies)
RUN npm install

# Copy source code
COPY . .

# Build the application
RUN npm run build

# Stage 2: Production stage
FROM node:18-alpine AS production

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install only production dependencies
RUN npm install --only=production && npm cache clean --force

# Copy built application from builder stage
COPY --from=builder /app/dist ./dist

# Change ownership to nodejs user
RUN chown -R nodejs:nodejs /app
USER nodejs

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

EXPOSE 3000

CMD ["npm", "start"]
```

### Dockerfile Best Practices

```dockerfile
# Use specific version tags
FROM node:18.17-alpine

# Combine RUN commands to reduce layers
RUN apk add --no-cache \
    curl \
    git \
    && rm -rf /var/cache/apk/*

# Use .dockerignore to exclude unnecessary files
# Create .dockerignore file

# Copy package files first for better caching
COPY package*.json ./
RUN npm install

# Copy application code last
COPY . .

# Use LABEL for metadata
LABEL version="1.0.0" \
      description="My Node.js application" \
      maintainer="your-email@example.com"

# Don't run as root
USER nodejs

# Use WORKDIR instead of cd
WORKDIR /app

# Prefer COPY over ADD
COPY src/ ./src/

# Clean up in same layer
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

---

## 6. Environment-specific Configurations

### Development Override

Create `docker-compose.override.yml`:

```yaml
version: '3.8'

services:
  backend:
    volumes:
      - ./backend:/app
    environment:
      - NODE_ENV=development
      - DEBUG=*
    command: npm run dev

  frontend:
    volumes:
      - ./frontend/src:/app/src
    environment:
      - CHOKIDAR_USEPOLLING=true
```

### Production Configuration

Create `docker-compose.prod.yml`:

```yaml
version: '3.8'

services:
  backend:
    environment:
      - NODE_ENV=production
    restart: unless-stopped
    deploy:
      replicas: 3
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M

  frontend:
    restart: unless-stopped

  database:
    restart: unless-stopped
    volumes:
      - postgres_data:/var/lib/postgresql/data
```

Run with specific configuration:
```bash
# Development (uses override automatically)
docker-compose up

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

---

## 🎯 Practice Exercises

### Exercise 1: Blog Application
Create a Docker Compose setup for a blog with:
- WordPress (frontend)
- MySQL (database)
- phpMyAdmin (database management)

**Hints:**
- Use official WordPress and MySQL images
- Set up proper environment variables
- Use named volumes for data persistence

### Exercise 2: Microservices Architecture
Build a simple e-commerce system:
- User service (Node.js)
- Product service (Python Flask)
- Order service (Node.js)
- PostgreSQL database
- Redis cache
- Nginx reverse proxy

### Exercise 3: Multi-stage Build
Create a multi-stage Dockerfile for a React application that:
1. Builds the React app in one stage
2. Serves it with Nginx in the final stage
3. Results in an image under 50MB

---

## 📋 Key Takeaways

1. **Docker Compose simplifies multi-container applications**
2. **Named volumes provide persistent, portable data storage**
3. **Multi-stage builds optimize image size and security**
4. **Custom networks enable secure service communication**
5. **Environment-specific configurations support different deployment stages**

---

## 🔗 What's Next?

Move to **03-advanced** to learn about:
- Docker Swarm orchestration
- Security hardening
- Performance optimization
- Production deployment strategies
- Monitoring and logging

---

## 💡 Common Intermediate Mistakes

1. **Not using named volumes**: Bind mounts in production can cause issues
2. **Ignoring build context**: Large build contexts slow down builds
3. **Running containers as root**: Security vulnerability
4. **Not using multi-stage builds**: Results in bloated images
5. **Hardcoding environment variables**: Use environment files instead 