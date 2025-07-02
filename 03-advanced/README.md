# 🔴 Advanced Level - Production Docker & Orchestration

## Learning Objectives
- Master Docker Swarm for container orchestration
- Implement security best practices for production
- Optimize Docker performance and resource usage
- Deploy containerized applications to production
- Set up monitoring and logging for Docker environments

---

## 1. Docker Swarm - Container Orchestration

Docker Swarm is Docker's native clustering and orchestration solution.

### Key Concepts

- **Swarm**: A cluster of Docker engines
- **Node**: A Docker engine participating in the swarm
- **Manager Node**: Manages swarm state and schedules services
- **Worker Node**: Runs containers assigned by managers
- **Service**: Definition of how containers should run across the swarm
- **Task**: A running container instance of a service

### Setting Up Docker Swarm

```bash
# Initialize swarm on manager node
docker swarm init --advertise-addr <MANAGER-IP>

# Join workers to swarm (run on worker nodes)
docker swarm join --token <WORKER-TOKEN> <MANAGER-IP>:2377

# Add additional managers
docker swarm join --token <MANAGER-TOKEN> <MANAGER-IP>:2377

# View swarm nodes
docker node ls

# View swarm info
docker info
```

### Creating and Managing Services

```bash
# Create a service
docker service create --name web --replicas 3 --publish 80:80 nginx

# List services
docker service ls

# Inspect a service
docker service inspect web

# View service tasks
docker service ps web

# Scale a service
docker service scale web=5

# Update a service
docker service update --image nginx:1.20 web

# Remove a service
docker service rm web
```

### Docker Stack - Multi-Service Applications

**docker-stack.yml**
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      placement:
        constraints:
          - node.role == worker
      resources:
        limits:
          cpus: '0.50'
          memory: 128M
        reservations:
          cpus: '0.25'
          memory: 64M
    networks:
      - webnet

  api:
    image: myapi:latest
    ports:
      - "5000:5000"
    deploy:
      replicas: 2
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
    depends_on:
      - db
    networks:
      - webnet
      - dbnet

  db:
    image: postgres:15
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: secret
    volumes:
      - db_data:/var/lib/postgresql/data
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.storage == ssd
    networks:
      - dbnet

volumes:
  db_data:

networks:
  webnet:
    external: true
  dbnet:
    external: true
```

### Stack Management Commands

```bash
# Deploy a stack
docker stack deploy -c docker-stack.yml myapp

# List stacks
docker stack ls

# List services in a stack
docker stack services myapp

# View stack tasks
docker stack ps myapp

# Remove a stack
docker stack rm myapp
```

---

## 2. Security Best Practices

### Container Security Hardening

#### 1. Use Non-Root Users

```dockerfile
# Create and use non-root user
FROM node:18-alpine

# Create user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Set working directory and ownership
WORKDIR /app
RUN chown nodejs:nodejs /app

# Copy files and set permissions
COPY --chown=nodejs:nodejs . .

# Switch to non-root user
USER nodejs

CMD ["npm", "start"]
```

#### 2. Minimize Attack Surface

```dockerfile
# Use minimal base images
FROM node:18-alpine

# Remove unnecessary packages
RUN apk del \
    curl \
    wget \
    && rm -rf /var/cache/apk/*

# Use specific versions
FROM node:18.17.0-alpine3.18

# Scan for vulnerabilities
# docker scan myimage:latest
```

#### 3. Secret Management

```bash
# Create secrets in swarm
echo "mysecretpassword" | docker secret create db_password -

# Use secrets in services
docker service create \
  --name myapp \
  --secret db_password \
  myimage:latest

# Access secret in container at /run/secrets/db_password
```

**Docker Compose with secrets:**
```yaml
version: '3.8'

services:
  app:
    image: myapp
    secrets:
      - db_password
      - api_key

secrets:
  db_password:
    external: true
  api_key:
    file: ./api_key.txt
```

#### 4. Resource Limits

```yaml
services:
  app:
    image: myapp
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
          pids: 100
        reservations:
          cpus: '0.25'
          memory: 256M
```

### Security Scanning

```bash
# Scan image for vulnerabilities
docker scan myimage:latest

# Use Docker Bench Security
docker run -it --net host --pid host --userns host --cap-add audit_control \
  -e DOCKER_CONTENT_TRUST=$DOCKER_CONTENT_TRUST \
  -v /etc:/etc:ro \
  -v /usr/bin/containerd:/usr/bin/containerd:ro \
  -v /usr/bin/runc:/usr/bin/runc:ro \
  -v /usr/lib/systemd:/usr/lib/systemd:ro \
  -v /var/lib:/var/lib:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  --label docker_bench_security \
  docker/docker-bench-security
```

---

## 3. Performance Optimization

### Image Optimization

#### Multi-Stage Build for Minimal Images

```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
WORKDIR /app
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./package.json
USER nodejs
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

#### .dockerignore for Faster Builds

```dockerignore
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.nyc_output
coverage
.nyc_output
.DS_Store
Dockerfile
.dockerignore
docker-compose*.yml
```

### Runtime Performance

#### Resource Monitoring

```bash
# Monitor container resource usage
docker stats

# Get container resource usage
docker container stats --no-stream

# Limit resources at runtime
docker run --cpus="1.5" --memory="2g" myapp
```

#### Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

```yaml
# Docker Compose health check
services:
  app:
    image: myapp
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

---

## 4. Production Deployment Strategies

### Blue-Green Deployment

```bash
# Current production is "blue"
docker service create --name web-blue --replicas 3 myapp:v1

# Deploy new version as "green"
docker service create --name web-green --replicas 3 myapp:v2

# Test green deployment
# Switch traffic from blue to green
docker service update --publish-rm 80:80 web-blue
docker service update --publish-add 80:80 web-green

# Remove old version
docker service rm web-blue
```

### Rolling Updates

```yaml
services:
  web:
    image: myapp:latest
    deploy:
      replicas: 5
      update_config:
        parallelism: 2
        delay: 10s
        failure_action: rollback
        monitor: 60s
        max_failure_ratio: 0.3
      rollback_config:
        parallelism: 1
        delay: 5s
        failure_action: pause
        monitor: 60s
```

### Canary Deployments

```bash
# Deploy canary version (10% of traffic)
docker service create --name web-canary --replicas 1 myapp:v2

# Monitor metrics and gradually increase replicas
docker service scale web-canary=2
docker service scale web-main=8

# If successful, complete rollout
docker service update --image myapp:v2 web-main
docker service rm web-canary
```

---

## 5. Monitoring and Logging

### Centralized Logging with ELK Stack

**docker-compose.monitoring.yml**
```yaml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.14.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - esdata:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"

  logstash:
    image: docker.elastic.co/logstash/logstash:7.14.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5044:5044"
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:7.14.0
    ports:
      - "5601:5601"
    environment:
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
    depends_on:
      - elasticsearch

  filebeat:
    image: docker.elastic.co/beats/filebeat:7.14.0
    volumes:
      - ./filebeat.yml:/usr/share/filebeat/filebeat.yml
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    depends_on:
      - logstash

volumes:
  esdata:
```

### Monitoring with Prometheus and Grafana

**docker-compose.prometheus.yml**
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
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

  node-exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro

volumes:
  prometheus_data:
  grafana_data:
```

### Application Metrics

```dockerfile
# Add health check endpoint to your app
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install

# Expose metrics endpoint
EXPOSE 3000 9090

CMD ["npm", "start"]
```

---

## 6. CI/CD Integration

### GitLab CI Pipeline

**.gitlab-ci.yml**
```yaml
stages:
  - build
  - test
  - security
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"

before_script:
  - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY

build:
  stage: build
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA

test:
  stage: test
  script:
    - docker run --rm $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA npm test

security_scan:
  stage: security
  script:
    - docker run --rm -v /var/run/docker.sock:/var/run/docker.sock 
      aquasec/trivy image $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  allow_failure: true

deploy_staging:
  stage: deploy
  script:
    - docker service update --image $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA staging_app
  environment:
    name: staging
  only:
    - develop

deploy_production:
  stage: deploy
  script:
    - docker service update --image $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA prod_app
  environment:
    name: production
  only:
    - master
  when: manual
```

---

## 🎯 Advanced Practice Exercises

### Exercise 1: High Availability Web Application
Deploy a web application with:
- 3 manager nodes, 5 worker nodes
- Load balancer (Nginx)
- Database with replication
- Redis cluster
- Monitoring stack

### Exercise 2: Zero-Downtime Deployment
Implement a complete CI/CD pipeline with:
- Automated testing
- Security scanning
- Blue-green deployment
- Rollback strategy

### Exercise 3: Microservices Platform
Build a complete microservices platform with:
- API Gateway
- Service discovery
- Circuit breakers
- Distributed tracing
- Centralized logging

---

## 📋 Production Checklist

### Security
- [ ] No containers running as root
- [ ] Secrets management implemented
- [ ] Regular security scans
- [ ] Resource limits configured
- [ ] Network segmentation

### Performance
- [ ] Multi-stage builds used
- [ ] Images optimized for size
- [ ] Health checks implemented
- [ ] Resource monitoring
- [ ] Caching strategies

### Reliability
- [ ] High availability setup
- [ ] Backup and recovery plan
- [ ] Monitoring and alerting
- [ ] Disaster recovery tested
- [ ] Documentation updated

---

## 🔗 Beyond Docker

### When to Consider Alternatives

- **Kubernetes**: For complex orchestration needs
- **Podman**: For rootless containers
- **containerd**: For lightweight container runtime
- **Cloud Services**: AWS ECS, Google Cloud Run, Azure Container Instances

---

## 💡 Advanced Best Practices

1. **Use multi-stage builds for all production images**
2. **Implement proper secret management**
3. **Monitor everything - logs, metrics, traces**
4. **Automate security scanning in CI/CD**
5. **Plan for disaster recovery**
6. **Document everything**
7. **Regular updates and patches**
8. **Performance testing under load**

---

## 🎉 Congratulations!

You've completed the advanced Docker course! You now have the skills to:
- Deploy production-ready containerized applications
- Implement security best practices
- Set up monitoring and logging
- Orchestrate containers with Docker Swarm
- Optimize performance and resource usage

Keep practicing and stay updated with the latest Docker developments! 