# Docker and Docker Compose - Complete Guide

## Table of Contents
1. [Introduction to Containers](#introduction-to-containers)
2. [Docker Fundamentals](#docker-fundamentals)
3. [Docker Images](#docker-images)
4. [Docker Containers](#docker-containers)
5. [Docker Networking](#docker-networking)
6. [Docker Volumes](#docker-volumes)
7. [Docker Compose](#docker-compose)
8. [Multi-Stage Builds](#multi-stage-builds)
9. [Docker Security](#docker-security)
10. [Best Practices](#best-practices)
11. [Troubleshooting](#troubleshooting)

---

## Introduction to Containers

### What are Containers?
Containers are lightweight, isolated environments that package applications and their dependencies. Unlike virtual machines, containers share the host OS kernel but run in isolated user spaces.

**Key Benefits:**
- **Portability**: Run anywhere Docker is installed
- **Consistency**: Same environment across development, testing, and production
- **Isolation**: Applications don't interfere with each other
- **Efficiency**: Lower resource overhead compared to VMs
- **Scalability**: Easy to scale horizontally

### Container vs Virtual Machine
```
┌─────────────────────────────────────────────────────────────┐
│                    Virtual Machine                          │
├─────────────────────────────────────────────────────────────┤
│  App A  │  App B  │  App C  │  Guest OS  │  Guest OS      │
├─────────────────────────────────────────────────────────────┤
│                    Hypervisor                               │
├─────────────────────────────────────────────────────────────┤
│                    Host Operating System                    │
├─────────────────────────────────────────────────────────────┤
│                    Hardware                                 │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                      Containers                            │
├─────────────────────────────────────────────────────────────┤
│  App A  │  App B  │  App C  │  App D  │  App E            │
├─────────────────────────────────────────────────────────────┤
│                    Docker Engine                            │
├─────────────────────────────────────────────────────────────┤
│                    Host Operating System                    │
├─────────────────────────────────────────────────────────────┤
│                    Hardware                                 │
└─────────────────────────────────────────────────────────────┘
```

---

## Docker Fundamentals

### Docker Architecture
Docker uses a client-server architecture:

- **Docker Daemon (dockerd)**: Background service that manages containers
- **Docker Client (docker)**: CLI tool to interact with the daemon
- **Docker Registry**: Repository for storing and distributing images
- **Docker Hub**: Default public registry

### Installation

#### macOS
```bash
# Install Docker Desktop
brew install --cask docker

# Or download from https://www.docker.com/products/docker-desktop
```

#### Linux (Ubuntu/Debian)
```bash
# Update package index
sudo apt-get update

# Install prerequisites
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker's official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Set up stable repository
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# Add user to docker group
sudo usermod -aG docker $USER
```

#### Windows
Download Docker Desktop from https://www.docker.com/products/docker-desktop

### Basic Commands
```bash
# Check Docker version
docker --version

# Check Docker system info
docker system info

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# List images
docker images

# Check Docker daemon status
docker system df
```

---

## Docker Images

### What are Docker Images?
Docker images are read-only templates used to create containers. They contain the application code, runtime, libraries, and dependencies.

### Image Layers
Images are built in layers, with each instruction in a Dockerfile creating a new layer:

```dockerfile
FROM ubuntu:20.04          # Base layer
RUN apt-get update         # Layer 1
RUN apt-get install nginx  # Layer 2
COPY app/ /var/www/html/   # Layer 3
EXPOSE 80                  # Layer 4
CMD ["nginx", "-g", "daemon off;"]  # Layer 5
```

### Working with Images

#### Pulling Images
```bash
# Pull latest version
docker pull nginx

# Pull specific version
docker pull nginx:1.21.0

# Pull from specific registry
docker pull myregistry.com/myapp:v1.0
```

#### Building Images
```bash
# Build from current directory
docker build -t myapp:v1.0 .

# Build with specific Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Build with build arguments
docker build --build-arg VERSION=1.0 -t myapp:v1.0 .
```

#### Managing Images
```bash
# List images
docker images

# Remove image
docker rmi nginx:latest

# Remove all unused images
docker image prune -a

# Save image to tar file
docker save -o nginx.tar nginx:latest

# Load image from tar file
docker load -i nginx.tar
```

### Dockerfile Best Practices

#### Basic Dockerfile Example
```dockerfile
# Use specific base image version
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files first (for better caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# Start application
CMD ["npm", "start"]
```

#### Multi-Stage Build Example
```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

---

## Docker Containers

### Container Lifecycle
```
Created → Running → Paused → Running → Stopped → Deleted
    ↑         ↓        ↓        ↓        ↓
    └─────────┴────────┴────────┴────────┴─────── Restart
```

### Container Commands

#### Creating and Running Containers
```bash
# Run container in foreground
docker run nginx

# Run container in background
docker run -d nginx

# Run with custom name
docker run -d --name my-nginx nginx

# Run with port mapping
docker run -d -p 8080:80 nginx

# Run with environment variables
docker run -d -e DB_HOST=localhost -e DB_PORT=5432 myapp

# Run with volume mount
docker run -d -v /host/path:/container/path nginx

# Run with network
docker run -d --network my-network nginx
```

#### Managing Running Containers
```bash
# List running containers
docker ps

# List all containers
docker ps -a

# Stop container
docker stop container_name

# Start stopped container
docker start container_name

# Restart container
docker restart container_name

# Pause container
docker pause container_name

# Unpause container
docker unpause container_name

# Remove container
docker rm container_name

# Remove running container (force)
docker rm -f container_name
```

#### Container Inspection
```bash
# View container logs
docker logs container_name

# Follow logs
docker logs -f container_name

# View last N lines
docker logs --tail 100 container_name

# Execute command in running container
docker exec -it container_name /bin/bash

# View container details
docker inspect container_name

# View container stats
docker stats

# View container processes
docker top container_name
```

### Container Resource Limits
```bash
# Limit memory
docker run -d --memory=512m nginx

# Limit CPU
docker run -d --cpus=1.0 nginx

# Limit both
docker run -d --memory=512m --cpus=1.0 nginx

# Set memory reservation
docker run -d --memory-reservation=256m nginx
```

---

## Docker Networking

### Network Types

#### Bridge Network (Default)
```bash
# Create custom bridge network
docker network create my-network

# Run container on specific network
docker run -d --network my-network --name web nginx

# Connect existing container to network
docker network connect my-network container_name

# Inspect network
docker network inspect my-network
```

#### Host Network
```bash
# Use host networking
docker run -d --network host nginx
```

#### None Network
```bash
# Disable networking
docker run -d --network none nginx
```

### Network Commands
```bash
# List networks
docker network ls

# Create network with custom subnet
docker network create --subnet=172.18.0.0/16 my-network

# Remove network
docker network rm my-network

# Remove unused networks
docker network prune
```

### Container Communication
```bash
# Run containers on same network
docker run -d --network my-network --name db postgres
docker run -d --network my-network --name app myapp

# Containers can communicate using container names
# app can connect to db using hostname "db"
```

---

## Docker Volumes

### Volume Types

#### Named Volumes
```bash
# Create named volume
docker volume create my-data

# Run container with named volume
docker run -d -v my-data:/app/data nginx

# List volumes
docker volume ls

# Inspect volume
docker volume inspect my-data

# Remove volume
docker volume rm my-data
```

#### Bind Mounts
```bash
# Mount host directory
docker run -d -v /host/path:/container/path nginx

# Mount with read-only
docker run -d -v /host/path:/container/path:ro nginx
```

#### tmpfs Mounts
```bash
# Mount temporary filesystem
docker run -d --tmpfs /app/temp nginx
```

### Volume Commands
```bash
# List all volumes
docker volume ls

# Remove unused volumes
docker volume prune

# Backup volume
docker run --rm -v my-data:/data -v $(pwd):/backup alpine tar czf /backup/my-data.tar.gz -C /data .

# Restore volume
docker run --rm -v my-data:/data -v $(pwd):/backup alpine tar xzf /backup/my-data.tar.gz -C /data
```

---

## Docker Compose

### What is Docker Compose?
Docker Compose is a tool for defining and running multi-container Docker applications using YAML files.

### Basic Compose File
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    depends_on:
      - db
    environment:
      - DB_HOST=db
      - DB_PORT=5432
    networks:
      - app-network

  db:
    image: postgres:13
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network

volumes:
  postgres_data:

networks:
  app-network:
    driver: bridge
```

### Compose Commands
```bash
# Start services
docker-compose up

# Start in background
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs

# View logs for specific service
docker-compose logs web

# Follow logs
docker-compose logs -f

# Execute command in service
docker-compose exec web /bin/bash

# Scale service
docker-compose up -d --scale web=3

# Build and start
docker-compose up --build

# Stop and remove volumes
docker-compose down -v
```

### Advanced Compose Features

#### Health Checks
```yaml
services:
  web:
    image: nginx
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

#### Resource Limits
```yaml
services:
  web:
    image: nginx
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
```

#### Secrets Management
```yaml
services:
  db:
    image: postgres
    secrets:
      - db_password
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password

secrets:
  db_password:
    file: ./db_password.txt
```

---

## Multi-Stage Builds

### Purpose
Multi-stage builds allow you to use multiple FROM statements in your Dockerfile to create smaller, more secure final images.

### Example: Node.js Application
```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Example: Go Application
```dockerfile
# Build stage
FROM golang:1.19-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Production stage
FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
CMD ["./main"]
```

---

## Docker Security

### Security Best Practices

#### 1. Use Non-Root Users
```dockerfile
FROM node:18-alpine
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001
USER nodejs
```

#### 2. Scan Images for Vulnerabilities
```bash
# Using Docker Scout (formerly Snyk)
docker scout cves nginx:latest

# Using Trivy
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image nginx:latest
```

#### 3. Use Specific Image Tags
```dockerfile
# Good
FROM nginx:1.21.0-alpine

# Bad
FROM nginx:latest
```

#### 4. Minimize Attack Surface
```dockerfile
# Use minimal base images
FROM alpine:latest

# Remove unnecessary packages
RUN apk add --no-cache nginx && \
    rm -rf /var/cache/apk/*
```

#### 5. Secrets Management
```yaml
# docker-compose.yml
services:
  app:
    image: myapp
    secrets:
      - db_password
    environment:
      DB_PASSWORD_FILE: /run/secrets/db_password

secrets:
  db_password:
    external: true
```

### Security Scanning
```bash
# Install Trivy
brew install trivy

# Scan local image
trivy image myapp:latest

# Scan remote image
trivy image nginx:latest

# Generate report
trivy image --format json --output report.json nginx:latest
```

---

## Best Practices

### 1. Image Optimization
- Use multi-stage builds
- Minimize layers
- Use .dockerignore files
- Remove unnecessary files

### 2. Container Design
- One process per container
- Use health checks
- Implement graceful shutdown
- Log to stdout/stderr

### 3. Resource Management
- Set resource limits
- Monitor container usage
- Use appropriate base images
- Clean up unused resources

### 4. Networking
- Use custom networks
- Avoid host networking
- Implement proper service discovery
- Use reverse proxies

### 5. Data Management
- Use named volumes for persistence
- Backup important data
- Implement proper backup strategies
- Use read-only mounts where possible

---

## Troubleshooting

### Common Issues

#### 1. Container Won't Start
```bash
# Check container logs
docker logs container_name

# Check container details
docker inspect container_name

# Run container interactively
docker run -it image_name /bin/bash
```

#### 2. Port Already in Use
```bash
# Check what's using the port
lsof -i :8080

# Use different port
docker run -p 8081:80 nginx
```

#### 3. Permission Issues
```bash
# Fix volume permissions
docker run -v /host/path:/container/path:rw nginx

# Use proper user in container
USER 1000:1000
```

#### 4. Out of Disk Space
```bash
# Clean up unused resources
docker system prune -a

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune
```

### Debugging Commands
```bash
# View container processes
docker top container_name

# View container stats
docker stats

# Execute command in container
docker exec -it container_name /bin/bash

# View container filesystem
docker diff container_name

# Copy files from/to container
docker cp container_name:/path /host/path
docker cp /host/path container_name:/path
```

### Performance Monitoring
```bash
# Monitor resource usage
docker stats

# View system-wide information
docker system df

# Analyze image layers
docker history image_name

# Profile container performance
docker run --rm -it --privileged -v /var/run/docker.sock:/var/run/docker.sock \
  nicolaka/netshoot
```

---

## Next Steps

After mastering Docker and Docker Compose, you'll be ready to:

1. **Learn Nomad**: Understand how Nomad uses containers for workload orchestration
2. **Explore Consul**: See how service discovery works with containerized applications
3. **Integrate Vault**: Learn how to securely manage secrets in containerized environments
4. **Build Production Systems**: Deploy scalable, secure containerized applications

### Practice Projects

1. **Multi-tier Application**: Build a web app with frontend, backend, and database
2. **Microservices**: Create multiple services that communicate via APIs
3. **CI/CD Pipeline**: Set up automated builds and deployments
4. **Monitoring Stack**: Deploy Prometheus, Grafana, and other monitoring tools

### Resources

- [Docker Official Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Docker Hub](https://hub.docker.com/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Docker Security](https://docs.docker.com/engine/security/) 