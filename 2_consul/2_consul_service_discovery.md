# Consul Service Discovery - Complete Guide

## Table of Contents
1. [Service Discovery Overview](#service-discovery-overview)
2. [Service Registration](#service-registration)
3. [DNS-Based Discovery](#dns-based-discovery)
4. [HTTP API Discovery](#http-api-discovery)
5. [Service Health and Filtering](#service-health-and-filtering)
6. [Load Balancing](#load-balancing)
7. [Service Mesh Integration](#service-mesh-integration)
8. [Best Practices](#best-practices)

---

## Service Discovery Overview

### What is Service Discovery?
Service discovery is the process of automatically detecting and registering services in a distributed system, allowing services to find and communicate with each other without hardcoded addresses.

### Service Discovery Flow
```
┌─────────────────────────────────────────────────────────────┐
│                Service Discovery Flow                       │
├─────────────────────────────────────────────────────────────┤
│ 1. Service starts and registers with Consul                │
│ 2. Consul stores service metadata and health status        │
│ 3. Other services query Consul for service locations       │
│ 4. Consul returns healthy service instances                │
│ 5. Services communicate directly using discovered info     │
└─────────────────────────────────────────────────────────────┘
```

### Service Discovery Benefits
- **Dynamic Registration**: Services register themselves automatically
- **Health Awareness**: Only healthy services are returned
- **Load Balancing**: Distribute traffic across multiple instances
- **Fault Tolerance**: Automatic failover to healthy instances
- **Configuration Management**: Centralized service configuration

---

## Service Registration

### Manual Registration
```bash
# Register a service manually
consul services register -name=web -port=8080 -address=192.168.1.10

# Register with health check
consul services register -name=web -port=8080 -check-http=http://localhost:8080/health

# Register with tags
consul services register -name=web -port=8080 -tags=v1,production
```

### Configuration File Registration
Create `web-service.json`:
```json
{
  "service": {
    "name": "web",
    "id": "web-1",
    "port": 8080,
    "address": "192.168.1.10",
    "tags": ["v1", "production", "frontend"],
    "meta": {
      "version": "1.0.0",
      "environment": "production",
      "team": "frontend"
    },
    "check": {
      "id": "web-health",
      "name": "Web Health Check",
      "http": "http://localhost:8080/health",
      "interval": "10s",
      "timeout": "5s",
      "method": "GET"
    }
  }
}
```

### Application Integration
```python
import consul
import requests

# Connect to Consul
c = consul.Consul()

# Register service
c.agent.service.register(
    name='web',
    service_id='web-1',
    port=8080,
    address='192.168.1.10',
    tags=['v1', 'production'],
    check=consul.Check.http(
        url='http://localhost:8080/health',
        interval='10s',
        timeout='5s'
    )
)

# Deregister service
c.agent.service.deregister('web-1')
```

### Docker Integration
```yaml
# docker-compose.yml
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

  consul:
    image: consul:latest
    ports:
      - "8500:8500"
      - "8600:8600/udp"
    command: consul agent -dev -client=0.0.0.0
```

---

## DNS-Based Discovery

### DNS Query Format
```
# Service discovery
<service-name>.service.<datacenter>.consul

# Service discovery with tags
<tag>.<service-name>.service.<datacenter>.consul

# Node discovery
<node-name>.node.<datacenter>.consul
```

### Basic DNS Queries
```bash
# Query service
dig @localhost -p 8600 web.service.consul

# Query service with tag
dig @localhost -p 8600 production.web.service.consul

# Query specific node
dig @localhost -p 8600 web-server-1.node.consul

# Query service in specific datacenter
dig @localhost -p 8600 web.service.dc1.consul
```

### DNS Response Examples
```bash
# Service query response
$ dig @localhost -p 8600 web.service.consul

; <<>> DiG 9.16.1 <<>> @localhost -p 8600 web.service.consul
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12345
;; flags: qr aa rd; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 0

;; ANSWER SECTION:
web.service.consul.    0 IN A 192.168.1.10
web.service.consul.    0 IN A 192.168.1.11

# Tagged service query
$ dig @localhost -p 8600 production.web.service.consul

;; ANSWER SECTION:
production.web.service.consul. 0 IN A 192.168.1.10
```

### DNS Configuration
```bash
# Configure system DNS
echo "nameserver 127.0.0.1" | sudo tee -a /etc/resolv.conf

# Test DNS resolution
nslookup web.service.consul 127.0.0.1:8600

# Use in applications
curl http://web.service.consul:8080/
```

### DNS with Load Balancing
```bash
# Round-robin load balancing
for i in {1..5}; do
  dig @localhost -p 8600 web.service.consul +short
done

# Output shows different IPs
192.168.1.10
192.168.1.11
192.168.1.10
192.168.1.11
192.168.1.10
```

---

## HTTP API Discovery

### Service Catalog API
```bash
# List all services
curl http://localhost:8500/v1/catalog/services

# Get service details
curl http://localhost:8500/v1/catalog/service/web

# Get healthy services only
curl "http://localhost:8500/v1/health/service/web?passing=true"

# Get services by tag
curl "http://localhost:8500/v1/catalog/services?tag=production"
```

### Health API
```bash
# Get service health
curl http://localhost:8500/v1/health/service/web

# Get passing services only
curl "http://localhost:8500/v1/health/service/web?passing=true"

# Get service health with details
curl "http://localhost:8500/v1/health/service/web?format=json"
```

### Agent API
```bash
# List local services
curl http://localhost:8500/v1/agent/services

# Get local service
curl http://localhost:8500/v1/agent/service/web

# Register service via API
curl -X PUT http://localhost:8500/v1/agent/service/register \
  -d '{
    "name": "web",
    "port": 8080,
    "check": {
      "http": "http://localhost:8080/health",
      "interval": "10s"
    }
  }'
```

### API Response Examples
```json
// GET /v1/catalog/service/web
[
  {
    "ID": "web-1",
    "Node": "web-server-1",
    "Address": "192.168.1.10",
    "Datacenter": "dc1",
    "TaggedAddresses": {
      "lan": "192.168.1.10",
      "wan": "10.0.0.10"
    },
    "NodeMeta": {
      "rack": "rack-1"
    },
    "ServiceID": "web-1",
    "ServiceName": "web",
    "ServiceAddress": "192.168.1.10",
    "ServicePort": 8080,
    "ServiceTags": ["v1", "production"],
    "ServiceMeta": {
      "version": "1.0.0"
    }
  }
]

// GET /v1/health/service/web?passing=true
[
  {
    "Node": {
      "ID": "web-server-1",
      "Node": "web-server-1",
      "Address": "192.168.1.10",
      "Datacenter": "dc1"
    },
    "Service": {
      "ID": "web-1",
      "Service": "web",
      "Tags": ["v1", "production"],
      "Port": 8080,
      "Address": "192.168.1.10"
    },
    "Checks": [
      {
        "Node": "web-server-1",
        "CheckID": "service:web-1",
        "Name": "Service 'web' check",
        "Status": "passing",
        "Notes": "",
        "Output": "HTTP GET http://localhost:8080/health: 200 OK",
        "ServiceID": "web-1",
        "ServiceName": "web"
      }
    ]
  }
]
```

---

## Service Health and Filtering

### Health Check States
```bash
# Get all service instances (including unhealthy)
curl http://localhost:8500/v1/health/service/web

# Get only passing services
curl "http://localhost:8500/v1/health/service/web?passing=true"

# Get services by status
curl "http://localhost:8500/v1/health/service/web?filter=Checks.Status==passing"
```

### Service Filtering
```bash
# Filter by tag
curl "http://localhost:8500/v1/catalog/service/web?tag=production"

# Filter by metadata
curl "http://localhost:8500/v1/catalog/service/web?filter=Service.Meta.version==1.0.0"

# Filter by node
curl "http://localhost:8500/v1/catalog/service/web?filter=Node==web-server-1"
```

### Health Check Configuration
```json
{
  "service": {
    "name": "web",
    "port": 8080,
    "check": {
      "id": "web-http",
      "name": "Web HTTP Check",
      "http": "http://localhost:8080/health",
      "interval": "10s",
      "timeout": "5s",
      "method": "GET",
      "header": {
        "Authorization": ["Bearer token123"]
      }
    },
    "checks": [
      {
        "id": "web-tcp",
        "name": "Web TCP Check",
        "tcp": "localhost:8080",
        "interval": "30s",
        "timeout": "10s"
      },
      {
        "id": "web-ttl",
        "name": "Web TTL Check",
        "ttl": "30s"
      }
    ]
  }
}
```

---

## Load Balancing

### DNS Load Balancing
```bash
# Round-robin DNS
dig @localhost -p 8600 web.service.consul

# Weighted round-robin (using tags)
dig @localhost -p 8600 primary.web.service.consul
dig @localhost -p 8600 secondary.web.service.consul
```

### Application-Level Load Balancing
```python
import consul
import random

def get_healthy_services(service_name):
    c = consul.Consul()
    _, services = c.health.service(service_name, passing=True)
    return services

def load_balance_request(service_name):
    services = get_healthy_services(service_name)
    if not services:
        raise Exception("No healthy services available")
    
    # Simple round-robin
    service = random.choice(services)
    return f"http://{service['Service']['Address']}:{service['Service']['Port']}"

# Usage
url = load_balance_request('web')
response = requests.get(f"{url}/api/data")
```

### Advanced Load Balancing
```python
import consul
from collections import defaultdict

class LoadBalancer:
    def __init__(self):
        self.c = consul.Consul()
        self.service_counters = defaultdict(int)
    
    def get_service(self, service_name, strategy='round_robin'):
        services = self.get_healthy_services(service_name)
        
        if strategy == 'round_robin':
            return self._round_robin(services)
        elif strategy == 'least_connections':
            return self._least_connections(services)
        elif strategy == 'random':
            return self._random(services)
    
    def _round_robin(self, services):
        if not services:
            return None
        
        service = services[self.service_counters[service_name] % len(services)]
        self.service_counters[service_name] += 1
        return service
    
    def _least_connections(self, services):
        # Implementation would track active connections
        return random.choice(services) if services else None
    
    def _random(self, services):
        return random.choice(services) if services else None
```

---

## Service Mesh Integration

### Consul Connect Overview
Consul Connect provides service-to-service communication with automatic mTLS encryption and authorization.

### Service Registration with Connect
```json
{
  "service": {
    "name": "web",
    "port": 8080,
    "connect": {
      "sidecar_service": {
        "port": 20000,
        "check": {
          "name": "Connect Sidecar Health",
          "http": "http://localhost:20000/health",
          "interval": "10s"
        }
      }
    }
  }
}
```

### Service Discovery with Connect
```bash
# Discover services via Connect
dig @localhost -p 8600 web.service.consul

# Connect to service via sidecar
curl http://localhost:20000/web/api/data
```

### Intentions (Authorization)
```bash
# Allow web service to connect to database
consul intention create web database

# Deny web service from connecting to admin
consul intention create -deny web admin

# List intentions
consul intention list

# Check intention
consul intention check web database
```

---

## Best Practices

### Service Naming
1. **Use consistent naming**: Follow a consistent naming convention
2. **Include version information**: Use tags for versioning
3. **Use descriptive names**: Make service names self-explanatory
4. **Avoid special characters**: Use only alphanumeric characters and hyphens

### Health Checks
1. **Implement comprehensive checks**: Use multiple check types
2. **Set appropriate intervals**: Balance responsiveness with overhead
3. **Handle check failures gracefully**: Implement proper error handling
4. **Monitor check performance**: Watch for check timeouts and failures

### Service Registration
1. **Register on startup**: Register services when they start
2. **Deregister on shutdown**: Clean up registrations when services stop
3. **Use unique IDs**: Ensure service IDs are unique
4. **Include metadata**: Add relevant metadata for filtering

### Load Balancing
1. **Use health-aware load balancing**: Only route to healthy services
2. **Implement circuit breakers**: Handle service failures gracefully
3. **Monitor load balancer performance**: Track metrics and adjust
4. **Use appropriate strategies**: Choose the right load balancing strategy

### Security
1. **Use ACLs**: Implement access control lists
2. **Enable TLS**: Encrypt all communications
3. **Validate service identity**: Verify service authenticity
4. **Monitor access patterns**: Audit service discovery usage

### Performance
1. **Cache service data**: Cache frequently accessed service information
2. **Optimize queries**: Use efficient API calls
3. **Monitor latency**: Track service discovery performance
4. **Scale appropriately**: Add more Consul servers for large clusters

---

## Next Steps

After mastering service discovery, you'll be ready to:

1. **Explore Consul Connect**: Learn about service mesh capabilities
2. **Advanced Features**: Master ACLs, federation, and monitoring
3. **Production Setup**: Deploy Consul in production environments
4. **Integration**: Integrate with other HashiCorp tools

### Practice Projects

1. **Multi-Service Application**: Build a microservices application with service discovery
2. **Load Balancer**: Implement a custom load balancer using Consul
3. **Health Monitoring**: Create comprehensive health monitoring
4. **Service Mesh**: Set up secure service communication with Connect

### Resources

- [Consul Service Discovery Documentation](https://www.consul.io/docs/discovery)
- [Consul API Reference](https://www.consul.io/api-docs)
- [Consul DNS Interface](https://www.consul.io/docs/discovery/dns)
- [Consul Connect Documentation](https://www.consul.io/docs/connect) 