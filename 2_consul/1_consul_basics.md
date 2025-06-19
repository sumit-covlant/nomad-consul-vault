# Consul Basics - Complete Guide

## Table of Contents
1. [What is Consul?](#what-is-consul)
2. [Consul Architecture](#consul-architecture)
3. [Core Concepts](#core-concepts)
4. [Installation and Setup](#installation-and-setup)
5. [First Steps](#first-steps)
6. [Key-Value Store](#key-value-store)
7. [Health Checks](#health-checks)
8. [Best Practices](#best-practices)

---

## What is Consul?

### Overview
Consul is a service mesh solution that provides service discovery, configuration, and segmentation functionality. It enables you to discover and configure services in your infrastructure, providing a complete solution for service networking.

### Key Features
```
┌─────────────────────────────────────────────────────────────┐
│                    Consul Features                         │
├─────────────────────────────────────────────────────────────┤
│ 🔍 Service Discovery: Automatic service registration       │
│ 🔧 Configuration: Distributed key-value store              │
│ 🛡️  Service Mesh: Secure service-to-service communication  │
│ 🔐 Security: ACLs, mTLS, and encryption                    │
│ 📊 Health Checking: Built-in health monitoring             │
│ 🌐 Multi-Datacenter: Native multi-DC support               │
│ 🔄 Federation: Connect multiple Consul clusters            │
│ 📈 Observability: Built-in metrics and logging             │
└─────────────────────────────────────────────────────────────┘
```

### Use Cases
- **Service Discovery**: Automatically discover services in your infrastructure
- **Configuration Management**: Store and distribute configuration data
- **Service Mesh**: Secure communication between services
- **Health Monitoring**: Monitor service health and availability
- **Multi-Datacenter**: Connect services across multiple datacenters
- **Load Balancing**: Intelligent load balancing with health awareness

### Comparison with Other Solutions
```
┌─────────────────────────────────────────────────────────────┐
│                Service Discovery Comparison                │
├─────────────────────────────────────────────────────────────┤
│ Consul:      Service mesh, KV store, multi-DC, ACLs       │
│ etcd:        Distributed key-value store, simple           │
│ ZooKeeper:   Coordination service, complex setup           │
│ Eureka:      Netflix service discovery, Java-focused       │
│ Istio:       Service mesh, Kubernetes-native               │
└─────────────────────────────────────────────────────────────┘
```

---

## Consul Architecture

### High-Level Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    Consul Cluster                          │
├─────────────────────────────────────────────────────────────┤
│                    Server Nodes                            │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │ Server 1    │ │ Server 2    │ │ Server 3    │           │
│ │ (Leader)    │ │ (Follower)  │ │ (Follower)  │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
│         │               │               │                 │
│         └───────────────┼───────────────┘                 │
│                         │                                 │
│                    Client Nodes                           │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │ Client 1    │ │ Client 2    │ │ Client 3    │           │
│ │ ┌─┐ ┌─┐ ┌─┐ │ │ ┌─┐ ┌─┐ ┌─┐ │ │ ┌─┐ ┌─┐ ┌─┐ │           │
│ │ │A│ │B│ │C│ │ │ │D│ │E│ │F│ │ │ │G│ │H│ │I│ │           │
│ │ └─┘ └─┘ └─┘ │ │ └─┘ └─┘ └─┘ │ │ └─┘ └─┘ └─┘ │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

### Server Nodes
Server nodes are responsible for:
- **Consensus**: Using Raft protocol for leader election and data consistency
- **Data Storage**: Storing service registrations, KV data, and ACLs
- **API**: Providing HTTP API for service management
- **Gossip**: Participating in gossip protocol for failure detection

### Client Nodes
Client nodes are responsible for:
- **Service Registration**: Registering local services
- **Health Checks**: Running health checks on local services
- **API Proxy**: Proxying API requests to servers
- **Gossip**: Participating in gossip protocol

### Communication Protocols
```
┌─────────────────────────────────────────────────────────────┐
│                    Communication Protocols                 │
├─────────────────────────────────────────────────────────────┤
│ Raft:        Consensus protocol for server coordination    │
│ Gossip:      Failure detection and membership              │
│ HTTP API:    Service registration and querying             │
│ DNS:         Service discovery via DNS                     │
│ gRPC:        Service mesh communication                     │
└─────────────────────────────────────────────────────────────┘
```

---

## Core Concepts

### Services
A service is a logical grouping of tasks that provide the same functionality. Services can have multiple instances running on different nodes.

```json
{
  "service": {
    "name": "web",
    "id": "web-1",
    "port": 8080,
    "address": "192.168.1.10",
    "tags": ["v1", "production"],
    "meta": {
      "version": "1.0.0",
      "environment": "production"
    }
  }
}
```

### Nodes
A node represents a physical or virtual machine running Consul. Each node can host multiple services.

```json
{
  "node": {
    "id": "node-1",
    "node": "web-server-1",
    "address": "192.168.1.10",
    "datacenter": "dc1",
    "meta": {
      "rack": "rack-1",
      "zone": "zone-a"
    }
  }
}
```

### Datacenters
A datacenter is a logical grouping of nodes that are geographically close and have low-latency connectivity.

```hcl
# Consul configuration
datacenter = "dc1"
primary_datacenter = "dc1"
```

### Checks
Health checks monitor the health of services and nodes. They can be HTTP, TCP, or custom scripts.

```json
{
  "check": {
    "id": "web-health",
    "name": "Web Health Check",
    "http": "http://localhost:8080/health",
    "interval": "10s",
    "timeout": "5s"
  }
}
```

---

## Installation and Setup

### System Requirements
```
┌─────────────────────────────────────────────────────────────┐
│                System Requirements                         │
├─────────────────────────────────────────────────────────────┤
│ OS:         Linux, macOS, Windows                          │
│ CPU:        1+ cores                                       │
│ Memory:     512MB+ (2GB+ recommended)                     │
│ Storage:    1GB+ available space                          │
│ Network:    TCP ports 8300, 8301, 8302, 8500, 8600        │
└─────────────────────────────────────────────────────────────┘
```

### Installation Methods

#### Binary Installation
```bash
# Download Consul
wget https://releases.hashicorp.com/consul/1.15.0/consul_1.15.0_linux_amd64.zip
unzip consul_1.15.0_linux_amd64.zip
sudo mv consul /usr/local/bin/

# Verify installation
consul version
```

#### Package Manager Installation
```bash
# Ubuntu/Debian
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install consul

# CentOS/RHEL
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install consul

# macOS
brew tap hashicorp/tap
brew install hashicorp/tap/consul
```

### Development Setup
```bash
# Start Consul in development mode
consul agent -dev

# This starts:
# - Single server node
# - Web UI at http://localhost:8500
# - API at http://localhost:8500
# - DNS at localhost:8600
```

### Production Setup
```bash
# Create configuration directory
sudo mkdir -p /etc/consul.d

# Create server configuration
sudo tee /etc/consul.d/server.hcl <<EOF
datacenter = "dc1"
data_dir = "/opt/consul/data"
log_level = "INFO"
node_name = "server-1"
server = true
bootstrap_expect = 3
client_addr = "0.0.0.0"
bind_addr = "0.0.0.0"
advertise_addr = "192.168.1.10"
retry_join = ["192.168.1.11", "192.168.1.12"]
ui_config {
  enabled = true
}
EOF

# Create client configuration
sudo tee /etc/consul.d/client.hcl <<EOF
datacenter = "dc1"
data_dir = "/opt/consul/data"
log_level = "INFO"
node_name = "client-1"
server = false
client_addr = "0.0.0.0"
bind_addr = "0.0.0.0"
advertise_addr = "192.168.1.20"
retry_join = ["192.168.1.10", "192.168.1.11", "192.168.1.12"]
EOF

# Start Consul
sudo systemctl enable consul
sudo systemctl start consul
```

---

## First Steps

### Starting Consul
```bash
# Development mode (single node)
consul agent -dev

# Check status
consul members
consul info
```

### Your First Service
Create a service definition file `web-service.json`:

```json
{
  "service": {
    "name": "web",
    "port": 8080,
    "address": "192.168.1.10",
    "tags": ["v1", "production"],
    "check": {
      "http": "http://localhost:8080/health",
      "interval": "10s",
      "timeout": "5s"
    }
  }
}
```

### Registering Services
```bash
# Register service
consul services register web-service.json

# List services
consul catalog services

# Get service details
consul catalog nodes -service=web

# Health check
curl http://localhost:8500/v1/health/service/web
```

### Basic Commands
```bash
# Service management
consul services register <service-file>
consul services deregister <service-id>
consul catalog services
consul catalog nodes -service=<service-name>

# Node management
consul members
consul catalog nodes
consul catalog datacenters

# Health checks
consul health check
consul health service <service-name>
consul health node <node-name>

# KV store
consul kv put key value
consul kv get key
consul kv delete key
consul kv export
consul kv import

# System information
consul info
consul operator raft list-peers
consul operator raft remove-peer <address>
```

---

## Key-Value Store

### What is the KV Store?
The Consul Key-Value (KV) store is a distributed, hierarchical key-value store that can be used for configuration management, feature flags, and coordination.

### Basic KV Operations
```bash
# Put a key-value pair
consul kv put myapp/config/database_url "postgresql://localhost:5432/mydb"

# Get a value
consul kv get myapp/config/database_url

# List keys
consul kv get -recurse myapp/

# Delete a key
consul kv delete myapp/config/database_url

# Check if key exists
consul kv get -detailed myapp/config/database_url
```

### Advanced KV Operations
```bash
# Put with flags
consul kv put -flags=42 myapp/config/feature_flag "enabled"

# Get with flags
consul kv get -detailed myapp/config/feature_flag

# Atomic operations
consul kv put -cas -modify-index=123 myapp/config/version "2.0.0"

# Lock and unlock
consul kv put -acquire -session=<session-id> myapp/lock "locked"
consul kv put -release -session=<session-id> myapp/lock "unlocked"
```

### KV Store Structure
```
myapp/
├── config/
│   ├── database_url
│   ├── redis_url
│   └── api_key
├── feature_flags/
│   ├── new_ui
│   └── beta_features
└── locks/
    └── maintenance
```

### KV Store with Applications
```python
import consul

# Connect to Consul
c = consul.Consul()

# Store configuration
c.kv.put('myapp/config/database_url', 'postgresql://localhost:5432/mydb')
c.kv.put('myapp/config/redis_url', 'redis://localhost:6379')
c.kv.put('myapp/config/api_key', 'secret-key-123')

# Retrieve configuration
_, data = c.kv.get('myapp/config/database_url')
database_url = data['Value'].decode('utf-8')

# Watch for changes
index = None
while True:
    index, data = c.kv.get('myapp/config/database_url', index=index)
    if data is not None:
        print(f"Database URL changed: {data['Value'].decode('utf-8')}")
```

---

## Health Checks

### What are Health Checks?
Health checks monitor the health and availability of services and nodes. They help Consul determine which services are healthy and available for routing traffic.

### Health Check Types

#### HTTP Health Checks
```json
{
  "check": {
    "id": "web-health",
    "name": "Web Health Check",
    "http": "http://localhost:8080/health",
    "interval": "10s",
    "timeout": "5s",
    "method": "GET",
    "header": {
      "Authorization": ["Bearer token123"]
    }
  }
}
```

#### TCP Health Checks
```json
{
  "check": {
    "id": "database-health",
    "name": "Database Health Check",
    "tcp": "localhost:5432",
    "interval": "30s",
    "timeout": "10s"
  }
}
```

#### Script Health Checks
```json
{
  "check": {
    "id": "disk-health",
    "name": "Disk Space Check",
    "script": "/usr/local/bin/check-disk.sh",
    "interval": "60s",
    "timeout": "30s"
  }
}
```

#### TTL Health Checks
```json
{
  "check": {
    "id": "app-health",
    "name": "Application Health",
    "ttl": "30s"
  }
}
```

### Health Check States
```
┌─────────────────────────────────────────────────────────────┐
│                    Health Check States                     │
├─────────────────────────────────────────────────────────────┤
│ passing:    Service is healthy and available               │
│ warning:    Service has issues but is still functional     │
│ critical:   Service is unhealthy and should not receive    │
│             traffic                                        │
│ maintenance: Service is under maintenance                  │
└─────────────────────────────────────────────────────────────┘
```

### Health Check Management
```bash
# List all health checks
consul health check

# Get health checks for a service
consul health service web

# Get health checks for a node
consul health node web-server-1

# Check specific health check
consul health check web-health
```

### Custom Health Check Script
```bash
#!/bin/bash
# /usr/local/bin/check-disk.sh

# Check disk usage
DISK_USAGE=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')

if [ $DISK_USAGE -gt 90 ]; then
    echo "Disk usage is critical: ${DISK_USAGE}%"
    exit 2
elif [ $DISK_USAGE -gt 80 ]; then
    echo "Disk usage is high: ${DISK_USAGE}%"
    exit 1
else
    echo "Disk usage is normal: ${DISK_USAGE}%"
    exit 0
fi
```

---

## Best Practices

### Service Registration
1. **Use meaningful names**: Choose descriptive service names
2. **Add metadata**: Include version, environment, and other metadata
3. **Implement health checks**: Always include health checks
4. **Use tags**: Tag services for filtering and routing
5. **Deregister properly**: Clean up services when they stop

### Health Checks
1. **Choose appropriate intervals**: Balance responsiveness with overhead
2. **Set reasonable timeouts**: Don't set timeouts too high
3. **Use multiple check types**: Combine different check types
4. **Handle failures gracefully**: Implement proper error handling
5. **Monitor check performance**: Watch for check failures

### Key-Value Store
1. **Use hierarchical structure**: Organize keys logically
2. **Version control**: Use atomic operations for updates
3. **Encrypt sensitive data**: Don't store plain text secrets
4. **Use locks for coordination**: Implement distributed locking
5. **Backup regularly**: Backup KV store data

### Security
1. **Enable ACLs**: Use Access Control Lists
2. **Use TLS**: Encrypt all communications
3. **Limit access**: Follow principle of least privilege
4. **Monitor access**: Audit ACL usage
5. **Regular updates**: Keep Consul updated

### Performance
1. **Optimize health checks**: Don't overload with too many checks
2. **Use caching**: Cache frequently accessed data
3. **Monitor metrics**: Watch Consul performance metrics
4. **Scale appropriately**: Add more servers for large clusters
5. **Network optimization**: Optimize network configuration

---

## Next Steps

After mastering Consul basics, you'll be ready to:

1. **Learn Service Discovery**: Deep dive into service discovery patterns
2. **Explore Consul Connect**: Understand service mesh capabilities
3. **Advanced Features**: Master ACLs, federation, and monitoring
4. **Production Setup**: Deploy Consul in production environments

### Practice Projects

1. **Service Registration**: Register and discover services
2. **Configuration Management**: Use KV store for configuration
3. **Health Monitoring**: Implement comprehensive health checks
4. **Service Mesh**: Set up secure service communication

### Resources

- [Consul Documentation](https://www.consul.io/docs/)
- [Consul GitHub Repository](https://github.com/hashicorp/consul)
- [Consul Community](https://discuss.hashicorp.com/c/consul)
- [Consul Learn](https://learn.hashicorp.com/consul) 