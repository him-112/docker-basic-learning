# 🎯 Docker Practice Exercises

Complete these hands-on exercises to master Docker concepts. Start with beginner level and progress through intermediate to advanced.

---

## 🟢 Beginner Exercises

### Exercise 1: Your First Container
**Objective**: Get familiar with basic Docker commands

**Tasks**:
1. Pull the `hello-world` image and run it
2. Pull the `nginx` image and run it on port 8080
3. List running containers and stop the nginx container
4. Clean up by removing all stopped containers

**Expected Output**: Successfully run containers and understand basic lifecycle

**Solution**:
```bash
# Pull and run hello-world
docker pull hello-world
docker run hello-world

# Run nginx on port 8080
docker run -d -p 8080:80 --name my-nginx nginx

# List containers
docker ps

# Stop container
docker stop my-nginx

# Clean up
docker rm my-nginx
docker container prune
```

---

### Exercise 2: Environment Variables and Data
**Objective**: Learn to work with environment variables and persistent data

**Tasks**:
1. Run a MySQL container with custom database name and password
2. Connect to the database and create a table
3. Stop and remove the container
4. Run MySQL again with a volume for data persistence
5. Verify your data persists

**Expected Output**: Database data survives container restarts

**Solution**:
```bash
# Run MySQL with environment variables
docker run -d \
  -e MYSQL_ROOT_PASSWORD=secret123 \
  -e MYSQL_DATABASE=testdb \
  -p 3306:3306 \
  --name mysql-test \
  mysql:8.0

# Connect and create table
docker exec -it mysql-test mysql -u root -p
# In MySQL: USE testdb; CREATE TABLE users (id INT, name VARCHAR(50));

# Run with volume
docker run -d \
  -e MYSQL_ROOT_PASSWORD=secret123 \
  -e MYSQL_DATABASE=testdb \
  -v mysql_data:/var/lib/mysql \
  -p 3306:3306 \
  --name mysql-persistent \
  mysql:8.0
```

---

### Exercise 3: Build Your First Image
**Objective**: Create and build a custom Docker image

**Tasks**:
1. Create a simple HTML page with your name and favorite color
2. Write a Dockerfile to serve it with nginx
3. Build the image with tag `my-website:v1.0`
4. Run a container from your image
5. Visit the website in your browser

**Expected Output**: Custom website running in a container

**Files Needed**:
```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head><title>My Website</title></head>
<body>
    <h1>Hello, I'm [Your Name]!</h1>
    <p>My favorite color is [Your Color]</p>
</body>
</html>
```

```dockerfile
# Dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
EXPOSE 80
```

**Solution**:
```bash
# Build image
docker build -t my-website:v1.0 .

# Run container
docker run -d -p 8080:80 --name my-site my-website:v1.0

# Visit http://localhost:8080
```

---

## 🟡 Intermediate Exercises

### Exercise 4: Multi-Container Blog Application
**Objective**: Create a complete blog application using Docker Compose

**Tasks**:
1. Set up WordPress with MySQL database
2. Add phpMyAdmin for database management
3. Use named volumes for data persistence
4. Configure proper networking between services
5. Set up environment variables properly

**Expected Output**: Working WordPress blog with database admin panel

**docker-compose.yml**:
```yaml
version: '3.8'

services:
  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: secret123
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress_data:/var/www/html
    depends_on:
      - db

  db:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: secret123
      MYSQL_ROOT_PASSWORD: rootsecret123
    volumes:
      - db_data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin:latest
    ports:
      - "8081:80"
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: rootsecret123
    depends_on:
      - db

volumes:
  wordpress_data:
  db_data:
```

---

### Exercise 5: Node.js API with Redis Cache
**Objective**: Build a multi-tier application with caching

**Tasks**:
1. Create a simple Node.js API that stores/retrieves data
2. Add Redis for caching
3. Create a multi-stage Dockerfile for the API
4. Use Docker Compose to orchestrate services
5. Implement health checks for all services

**Expected Output**: Scalable API with caching layer

**API Structure**:
```javascript
// app.js
const express = require('express');
const redis = require('redis');
const app = express();
const client = redis.createClient({ host: 'redis' });

app.get('/api/data/:key', async (req, res) => {
  const { key } = req.params;
  const cachedData = await client.get(key);
  
  if (cachedData) {
    return res.json({ data: cachedData, source: 'cache' });
  }
  
  // Simulate database query
  const data = `Data for ${key} - ${new Date().toISOString()}`;
  await client.setex(key, 60, data);
  
  res.json({ data, source: 'database' });
});

app.listen(3000, () => {
  console.log('API running on port 3000');
});
```

---

### Exercise 6: Development vs Production Environments
**Objective**: Configure different environments using Docker Compose

**Tasks**:
1. Create base docker-compose.yml for shared services
2. Create docker-compose.override.yml for development
3. Create docker-compose.prod.yml for production
4. Configure different settings for each environment
5. Test both environments

**Expected Output**: Environment-specific configurations working correctly

---

## 🔴 Advanced Exercises

### Exercise 7: Docker Swarm Cluster
**Objective**: Set up and manage a Docker Swarm cluster

**Tasks**:
1. Initialize a 3-node swarm (1 manager, 2 workers)
2. Deploy a web application stack across the swarm
3. Scale services and test load distribution
4. Implement rolling updates
5. Test node failure scenarios

**Expected Output**: Highly available application across multiple nodes

**Commands**:
```bash
# Initialize swarm
docker swarm init --advertise-addr <MANAGER-IP>

# Deploy stack
docker stack deploy -c docker-compose.yml webapp

# Scale service
docker service scale webapp_web=5

# Rolling update
docker service update --image nginx:1.20 webapp_web
```

---

### Exercise 8: Secure Production Deployment
**Objective**: Implement security best practices

**Tasks**:
1. Create non-root user in Dockerfile
2. Implement Docker secrets for sensitive data
3. Configure resource limits
4. Set up SSL/TLS termination
5. Implement vulnerability scanning in CI/CD

**Expected Output**: Production-ready secure deployment

**Secure Dockerfile**:
```dockerfile
FROM node:18-alpine

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy app files
COPY --chown=nodejs:nodejs . .

# Switch to non-root user
USER nodejs

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1

EXPOSE 3000
CMD ["npm", "start"]
```

---

### Exercise 9: Monitoring and Logging Stack
**Objective**: Set up comprehensive monitoring

**Tasks**:
1. Deploy Prometheus for metrics collection
2. Set up Grafana for visualization
3. Configure Loki for log aggregation
4. Add alerting rules
5. Create custom dashboards

**Expected Output**: Complete observability stack

**Monitoring Stack**:
```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana

  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    volumes:
      - loki_data:/loki

volumes:
  prometheus_data:
  grafana_data:
  loki_data:
```

---

### Exercise 10: CI/CD Pipeline
**Objective**: Create automated deployment pipeline

**Tasks**:
1. Set up GitLab CI or GitHub Actions
2. Build and test Docker images
3. Scan for security vulnerabilities
4. Deploy to staging automatically
5. Deploy to production with manual approval

**Expected Output**: Fully automated CI/CD pipeline

**.github/workflows/docker.yml**:
```yaml
name: Docker CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Build Docker image
      run: docker build -t myapp:${{ github.sha }} .
    
    - name: Run tests
      run: docker run --rm myapp:${{ github.sha }} npm test
    
    - name: Security scan
      run: |
        docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
          aquasec/trivy image myapp:${{ github.sha }}
    
    - name: Deploy to staging
      if: github.ref == 'refs/heads/develop'
      run: |
        # Deploy to staging environment
        echo "Deploying to staging"
```

---

## 🏆 Challenge Projects

### Challenge 1: Microservices E-commerce Platform
Build a complete e-commerce platform with:
- User service (authentication)
- Product catalog service
- Order management service
- Payment processing service
- API Gateway
- Message queue (RabbitMQ/Redis)
- Load balancers
- Monitoring stack

### Challenge 2: High-Performance Web Application
Create a high-performance web application with:
- CDN integration
- Database sharding
- Read replicas
- Caching layers (Redis, Memcached)
- Auto-scaling
- Performance monitoring

### Challenge 3: Multi-Cloud Deployment
Deploy applications across multiple cloud providers:
- AWS ECS deployment
- Google Cloud Run deployment
- Azure Container Instances
- Cross-cloud load balancing
- Disaster recovery setup

---

## 📝 Exercise Completion Checklist

For each exercise, ensure you can:
- [ ] Complete all tasks successfully
- [ ] Explain what each command does
- [ ] Troubleshoot common issues
- [ ] Optimize for performance and security
- [ ] Document your solution

---

## 🎯 Assessment Criteria

**Beginner Level**: 
- Basic Docker commands
- Simple Dockerfile creation
- Container lifecycle management

**Intermediate Level**:
- Docker Compose usage
- Multi-container applications
- Volume and network management

**Advanced Level**:
- Production deployment
- Security implementation
- Monitoring and scaling

---

## 💡 Tips for Success

1. **Read error messages carefully** - Docker provides detailed error information
2. **Use official documentation** - Docker docs are comprehensive and up-to-date
3. **Practice regularly** - Hands-on experience is crucial
4. **Join communities** - Docker forums, Reddit, Stack Overflow
5. **Keep learning** - Docker ecosystem evolves rapidly
6. **Document your work** - Good documentation helps with debugging
7. **Test thoroughly** - Always test your containers in different scenarios

---

## 🔗 Additional Resources

- [Docker Official Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Play with Docker](https://labs.play-with-docker.com/)
- [Docker Curriculum](https://docker-curriculum.com/)
- [Awesome Docker](https://github.com/veggiemonk/awesome-docker)

Happy containerizing! 🐳 