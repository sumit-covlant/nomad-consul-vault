# Consul Service Discovery with Docker

## Overview
Learn Consul service discovery using the local Docker environment.

## Prerequisites
- Complete [Quick Start](1_quick_start.md)

## Getting Started

### 1. Access Consul
```bash
# Set environment variables
export CONSUL_HTTP_ADDR='http://localhost:8500'

# Verify connection
consul members
```

### 2. Register Your First Service
Create `web-service.json`:
```json
{
  "service": {
    "name": "web",
    "port": 8080,
    "address": "127.0.0.1",
    "tags": ["v1", "frontend"],
    "check": {
      "http": "http://localhost:8080/health",
      "interval": "10s",
      "timeout": "5s"
    }
  }
}
```

```bash
# Register the service
consul services register web-service.json

# List services
consul catalog services

# Get service details
consul catalog nodes -service=web
```

### 3. Use DNS for Service Discovery
```bash
# Query service via DNS
dig @localhost -p 8600 web.service.consul

# Query with tags
dig @localhost -p 8600 v1.web.service.consul
```

### 4. Store Configuration in KV Store
```bash
# Store configuration
consul kv put myapp/config/database_url "postgresql://localhost:5432/mydb"
consul kv put myapp/config/redis_url "redis://localhost:6379"

# Retrieve configuration
consul kv get myapp/config/database_url

# List keys
consul kv get -recurse myapp/
```

### 5. Health Checks
```bash
# Check service health
consul health service web

# Check all health checks
consul health check
```

## Docker Compose for Consul Only
If you want to run just Consul:

```yaml
version: '3.8'
services:
  consul:
    image: consul:latest
    container_name: consul-dev
    ports:
      - "8500:8500"
      - "8600:8600/udp"
    environment:
      - CONSUL_BIND_INTERFACE=eth0
    command: consul agent -dev -client=0.0.0.0 -ui
```

## Sample Application with Consul
Create `docker-compose-app.yml`:
```yaml
version: '3.8'
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    environment:
      - CONSUL_HTTP_ADDR=consul:8500
    depends_on:
      - consul
    command: >
      sh -c "
        consul services register -name=web -port=8080 -address=web &&
        nginx -g 'daemon off;'
      "

  api:
    image: nginx:alpine
    ports:
      - "8081:80"
    environment:
      - CONSUL_HTTP_ADDR=consul:8500
    depends_on:
      - consul
    command: >
      sh -c "
        consul services register -name=api -port=8081 -address=api &&
        nginx -g 'daemon off;'
      "

  consul:
    image: consul:latest
    ports:
      - "8500:8500"
      - "8600:8600/udp"
    command: consul agent -dev -client=0.0.0.0 -ui
```

## Exercises
1. Register multiple services with different tags
2. Use DNS to discover services
3. Store and retrieve configuration from KV store
4. Set up health checks for services
5. Use service filtering and metadata

## Next Steps
- [Nomad Jobs](4_nomad_jobs.md)
- [Full Integration](5_full_integration.md) 