# Consul Advanced Features - Complete Guide

## Table of Contents
1. [Access Control Lists (ACLs)](#access-control-lists-acls)
2. [Namespaces](#namespaces)
3. [Federation](#federation)
4. [Monitoring and Observability](#monitoring-and-observability)
5. [Performance Tuning](#performance-tuning)
6. [Backup and Recovery](#backup-and-recovery)
7. [Integrations](#integrations)
8. [Best Practices](#best-practices)

---

## Access Control Lists (ACLs)

### What are ACLs?
Access Control Lists (ACLs) provide fine-grained access control for Consul operations, allowing you to control who can read, write, and manage different parts of your Consul cluster.

### ACL Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    ACL Architecture                        │
├─────────────────────────────────────────────────────────────┤
│                    ACL Tokens                              │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │ Management  │ │ Agent       │ │ Service     │           │
│ │ Token       │ │ Token       │ │ Token       │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
│         │               │               │                 │
│         └───────────────┼───────────────┘                 │
│                         │                                 │
│                    ACL Policies                           │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │ Global      │ │ Service     │ │ Node        │           │
│ │ Management  │ │ Management  │ │ Management  │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

### ACL Token Types
- **Management Token**: Full access to all Consul operations
- **Agent Token**: Used by Consul agents for service registration
- **Service Token**: Used by services for specific operations
- **Client Token**: Used by applications for read/write operations

### ACL Policy Management
```bash
# Enable ACLs
consul acl bootstrap

# Create policy
consul acl policy create -name="web-service-policy" -rules=@web-policy.hcl

# Create token
consul acl token create -description="Web service token" -policy-name="web-service-policy"

# List tokens
consul acl token list

# Read token
consul acl token read <token-id>

# Update token
consul acl token update -id=<token-id> -policy-name="new-policy"

# Delete token
consul acl token delete <token-id>
```

### ACL Policy Examples
```hcl
# web-policy.hcl - Allow web service operations
service "web" {
  policy = "write"
}

service_prefix "" {
  policy = "read"
}

key_prefix "web/" {
  policy = "write"
}

node_prefix "" {
  policy = "read"
}

agent_prefix "" {
  policy = "read"
}
```

```hcl
# admin-policy.hcl - Full administrative access
agent "consul" {
  policy = "write"
}

node_prefix "" {
  policy = "write"
}

service_prefix "" {
  policy = "write"
}

key_prefix "" {
  policy = "write"
}

query_prefix "" {
  policy = "write"
}

session_prefix "" {
  policy = "write"
}
```

```hcl
# read-only-policy.hcl - Read-only access
agent_prefix "" {
  policy = "read"
}

node_prefix "" {
  policy = "read"
}

service_prefix "" {
  policy = "read"
}

key_prefix "" {
  policy = "read"
}

query_prefix "" {
  policy = "read"
}
```

### ACL Configuration
```hcl
# consul-server.hcl
acl {
  enabled = true
  default_policy = "deny"
  enable_token_persistence = true
  tokens {
    master = "your-master-token"
    agent = "your-agent-token"
  }
}
```

### ACL with Applications
```python
import consul

# Connect with ACL token
c = consul.Consul(token='your-acl-token')

# Register service (requires write permission)
c.agent.service.register(
    name='web',
    port=8080,
    check=consul.Check.http('http://localhost:8080/health', interval='10s')
)

# Read service (requires read permission)
_, services = c.health.service('web', passing=True)

# Write to KV store (requires write permission)
c.kv.put('web/config/database_url', 'postgresql://localhost:5432/web')
```

---

## Namespaces

### What are Namespaces?
Namespaces provide logical isolation within a Consul datacenter, allowing you to separate services, policies, and tokens by team, environment, or application.

### Namespace Benefits
- **Logical Isolation**: Separate services by team or environment
- **Access Control**: Apply different ACL policies per namespace
- **Resource Management**: Limit resource usage per namespace
- **Multi-tenancy**: Support multiple tenants in a single cluster

### Namespace Management
```bash
# Create namespace
consul namespace create -name="frontend" -description="Frontend services"

# List namespaces
consul namespace list

# Read namespace
consul namespace read frontend

# Update namespace
consul namespace update -name="frontend" -description="Updated description"

# Delete namespace
consul namespace delete frontend
```

### Namespace Configuration
```hcl
# frontend-namespace.hcl
Name = "frontend"
Description = "Frontend services namespace"
Meta = {
  team = "frontend"
  environment = "production"
}
```

### Service Registration with Namespaces
```bash
# Register service in specific namespace
consul services register -namespace=frontend web-service.json

# List services in namespace
consul catalog services -namespace=frontend

# Get service health in namespace
consul health service -namespace=frontend web
```

### ACL Policies with Namespaces
```hcl
# frontend-policy.hcl
namespace "frontend" {
  service_prefix "" {
    policy = "write"
  }
  
  key_prefix "frontend/" {
    policy = "write"
  }
  
  node_prefix "" {
    policy = "read"
  }
}
```

### Namespace with Applications
```python
import consul

# Connect to specific namespace
c = consul.Consul(namespace='frontend')

# Register service in namespace
c.agent.service.register(
    name='web',
    port=8080,
    namespace='frontend'
)

# Read from KV store in namespace
_, data = c.kv.get('web/config', namespace='frontend')
```

---

## Federation

### What is Federation?
Federation allows you to connect multiple Consul datacenters, enabling service discovery and communication across geographically distributed clusters.

### Federation Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    Federation Architecture                 │
├─────────────────────────────────────────────────────────────┤
│                    Datacenter 1 (dc1)                     │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │ Server 1    │ │ Server 2    │ │ Server 3    │           │
│ │ (Leader)    │ │ (Follower)  │ │ (Follower)  │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
│         │               │               │                 │
│         └───────────────┼───────────────┘                 │
│                         │                                 │
│                    WAN Gossip                            │
│                         │                                 │
│                    Datacenter 2 (dc2)                     │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │ Server 1    │ │ Server 2    │ │ Server 3    │           │
│ │ (Leader)    │ │ (Follower)  │ │ (Follower)  │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

### Federation Setup
```hcl
# dc1-server.hcl
datacenter = "dc1"
primary_datacenter = "dc1"
server = true
bootstrap_expect = 3
retry_join_wan = ["dc2-server-1", "dc2-server-2", "dc2-server-3"]
```

```hcl
# dc2-server.hcl
datacenter = "dc2"
primary_datacenter = "dc1"
server = true
bootstrap_expect = 3
retry_join_wan = ["dc1-server-1", "dc1-server-2", "dc1-server-3"]
```

### Federation Management
```bash
# Join datacenters
consul join -wan dc2-server-1 dc2-server-2 dc2-server-3

# List WAN members
consul members -wan

# Check federation status
consul operator area members -wan

# Leave federation
consul leave -wan
```

### Cross-Datacenter Service Discovery
```bash
# Query service in specific datacenter
consul catalog nodes -service=web -datacenter=dc2

# Query service health across datacenters
consul health service -datacenter=dc2 web

# DNS query for cross-DC service
dig @localhost -p 8600 web.service.dc2.consul
```

### Federation with Applications
```python
import consul

# Connect to specific datacenter
c = consul.Consul(dc='dc2')

# Query services in remote datacenter
_, services = c.health.service('web', dc='dc2')

# Register service in local datacenter
c.agent.service.register(
    name='web',
    port=8080,
    tags=['dc1']
)
```

---

## Monitoring and Observability

### Metrics Collection
Consul provides comprehensive metrics for monitoring cluster health and performance.

### Prometheus Metrics
```bash
# Enable Prometheus metrics
consul agent -dev -telemetry-statsite-address=localhost:8125

# Configure metrics endpoint
curl http://localhost:8500/v1/agent/metrics?format=prometheus
```

### Key Metrics
```
┌─────────────────────────────────────────────────────────────┐
│                    Key Metrics                             │
├─────────────────────────────────────────────────────────────┤
│ consul.raft.apply: Raft log applications per second       │
│ consul.raft.commitTime: Raft commit time                   │
│ consul.raft.leader: Current Raft leader                    │
│ consul.raft.peers: Number of Raft peers                    │
│ consul.raft.state: Current Raft state                      │
│ consul.runtime.alloc_bytes: Memory allocation              │
│ consul.runtime.sys_bytes: System memory usage              │
│ consul.runtime.num_goroutines: Number of goroutines       │
│ consul.runtime.num_threads: Number of threads              │
│ consul.agent.memberlist.msg.suspect: Suspect messages      │
│ consul.agent.memberlist.msg.alive: Alive messages          │
│ consul.agent.memberlist.msg.dead: Dead messages            │
└─────────────────────────────────────────────────────────────┘
```

### Monitoring Configuration
```hcl
# consul-server.hcl
telemetry {
  prometheus_retention_time = "24h"
  disable_hostname = true
  statsd_address = "localhost:8125"
  statsite_address = "localhost:8125"
  dogstatsd_addr = "localhost:8125"
  dogstatsd_tags = ["tag1:value1", "tag2:value2"]
}
```

### Health Monitoring
```bash
# Check cluster health
consul operator raft list-peers

# Check server health
consul operator raft list-peers -server-id=server-1

# Check client health
consul members

# Check service health
consul health service web
```

### Logging Configuration
```hcl
# consul-server.hcl
log_level = "INFO"
log_file = "/var/log/consul/consul.log"
log_rotate_max_files = 5
log_rotate_max_size = "10MB"
log_rotate_duration = "24h"
```

### Alerting Rules
```yaml
# prometheus-rules.yml
groups:
  - name: consul
    rules:
      - alert: ConsulRaftNoLeader
        expr: consul_raft_leader == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Consul cluster has no leader"
          
      - alert: ConsulRaftPeers
        expr: consul_raft_peers < 3
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "Consul cluster has fewer than 3 peers"
          
      - alert: ConsulHighMemoryUsage
        expr: consul_runtime_alloc_bytes > 1e9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Consul is using more than 1GB of memory"
```

---

## Performance Tuning

### Server Performance
```hcl
# consul-server.hcl
performance {
  raft_multiplier = 1
  leave_drain_time = "5s"
  server_health_verification = true
  grpc_keepalive_time = "30s"
  grpc_keepalive_interval = "30s"
  grpc_keepalive_timeout = "5s"
}
```

### Client Performance
```hcl
# consul-client.hcl
performance {
  raft_multiplier = 1
  leave_drain_time = "5s"
  server_health_verification = true
}
```

### Network Tuning
```hcl
# consul-server.hcl
ports {
  dns = 8600
  http = 8500
  https = 8501
  grpc = 8502
  serf_lan = 8301
  serf_wan = 8302
  server = 8300
}

addresses {
  dns = "0.0.0.0"
  http = "0.0.0.0"
  https = "0.0.0.0"
  grpc = "0.0.0.0"
}
```

### Memory Tuning
```hcl
# consul-server.hcl
limits {
  http_max_conns_per_client = 200
  rpc_hold_timeout = "7s"
  rpc_rate = 100
  rpc_burst = 100
}
```

### Storage Tuning
```hcl
# consul-server.hcl
data_dir = "/opt/consul/data"
disable_update_check = true
disable_anonymous_signature = true
```

---

## Backup and Recovery

### Backup Strategies
```bash
# Backup KV store
consul kv export > consul-backup.json

# Backup ACLs
consul acl policy list > acl-policies.json
consul acl token list > acl-tokens.json

# Backup configuration
cp /etc/consul.d/* /backup/consul-config/

# Backup data directory
tar -czf consul-data-backup.tar.gz /opt/consul/data/
```

### Recovery Procedures
```bash
# Restore KV store
consul kv import < consul-backup.json

# Restore ACLs
consul acl policy create -name="policy-name" -rules=@policy.hcl
consul acl token create -description="token" -policy-name="policy-name"

# Restore configuration
cp /backup/consul-config/* /etc/consul.d/

# Restore data directory
tar -xzf consul-data-backup.tar.gz -C /
```

### Automated Backup Script
```bash
#!/bin/bash
# backup-consul.sh

BACKUP_DIR="/backup/consul"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR/$DATE

# Backup KV store
consul kv export > $BACKUP_DIR/$DATE/kv-backup.json

# Backup ACLs
consul acl policy list > $BACKUP_DIR/$DATE/acl-policies.json
consul acl token list > $BACKUP_DIR/$DATE/acl-tokens.json

# Backup configuration
cp -r /etc/consul.d/* $BACKUP_DIR/$DATE/config/

# Backup data directory
tar -czf $BACKUP_DIR/$DATE/data-backup.tar.gz /opt/consul/data/

# Clean up old backups (keep last 7 days)
find $BACKUP_DIR -type d -mtime +7 -exec rm -rf {} \;

echo "Backup completed: $BACKUP_DIR/$DATE"
```

---

## Integrations

### Nomad Integration
```hcl
# nomad-server.hcl
consul {
  address = "127.0.0.1:8500"
  server_service_name = "nomad"
  client_service_name = "nomad-client"
  auto_advertise = true
  server_auto_join = true
  client_auto_join = true
}
```

### Vault Integration
```hcl
# vault-server.hcl
storage "consul" {
  path = "vault/"
  service = "vault"
  service_tags = "vault"
  service_address = "127.0.0.1:8200"
  check_timeout = "5s"
  disable_registration = "false"
}

ha_storage "consul" {
  path = "vault/"
  service = "vault"
  service_tags = "vault"
  service_address = "127.0.0.1:8200"
  check_timeout = "5s"
  disable_registration = "false"
}
```

### Kubernetes Integration
```yaml
# consul-k8s.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: consul-config
data:
  consul.json: |
    {
      "datacenter": "dc1",
      "server": true,
      "bootstrap_expect": 3,
      "ui_config": {
        "enabled": true
      }
    }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: consul
spec:
  replicas: 3
  selector:
    matchLabels:
      app: consul
  template:
    metadata:
      labels:
        app: consul
    spec:
      containers:
      - name: consul
        image: consul:latest
        ports:
        - containerPort: 8500
        - containerPort: 8600
        volumeMounts:
        - name: consul-config
          mountPath: /consul/config
      volumes:
      - name: consul-config
        configMap:
          name: consul-config
```

### Terraform Integration
```hcl
# main.tf
resource "consul_key_prefix" "app_config" {
  path_prefix = "app/"

  subkeys = {
    "database_url" = "postgresql://localhost:5432/app"
    "redis_url"    = "redis://localhost:6379"
    "api_key"      = "secret-key-123"
  }
}

resource "consul_service" "web" {
  name = "web"
  port = 8080
  tags = ["v1", "production"]
  
  check {
    check_id = "web-health"
    name     = "Web Health Check"
    http     = "http://localhost:8080/health"
    interval = "10s"
    timeout  = "5s"
  }
}
```

---

## Best Practices

### Security Best Practices
1. **Enable ACLs**: Always use ACLs in production
2. **Use Namespaces**: Separate services by team/environment
3. **Rotate Tokens**: Regularly rotate ACL tokens
4. **Monitor Access**: Audit ACL usage and access patterns
5. **Secure Communication**: Use TLS for all communications

### Performance Best Practices
1. **Optimize Network**: Configure appropriate network settings
2. **Monitor Resources**: Track memory and CPU usage
3. **Scale Appropriately**: Add more servers for large clusters
4. **Use Caching**: Cache frequently accessed data
5. **Optimize Health Checks**: Don't overload with too many checks

### Operational Best Practices
1. **Backup Regularly**: Implement automated backup procedures
2. **Monitor Health**: Set up comprehensive monitoring
3. **Document Configuration**: Document all configuration changes
4. **Test Recovery**: Regularly test backup and recovery procedures
5. **Update Regularly**: Keep Consul updated with latest releases

### Federation Best Practices
1. **Plan Network**: Ensure proper network connectivity between DCs
2. **Monitor WAN**: Monitor WAN gossip performance
3. **Limit Cross-DC Traffic**: Minimize cross-datacenter communication
4. **Use Appropriate DCs**: Route traffic to appropriate datacenters
5. **Test Failover**: Test cross-datacenter failover procedures

---

## Next Steps

After mastering advanced features, you'll be ready to:

1. **Production Setup**: Deploy Consul in production environments
2. **Customization**: Customize Consul for specific use cases
3. **Integration**: Integrate with other tools and platforms
4. **Troubleshooting**: Master troubleshooting and debugging

### Practice Projects

1. **Multi-DC Setup**: Set up federation between datacenters
2. **ACL Implementation**: Implement comprehensive access control
3. **Monitoring Dashboard**: Build monitoring and alerting
4. **Integration Testing**: Test integrations with other tools

### Resources

- [Consul Advanced Features Documentation](https://www.consul.io/docs/advanced)
- [Consul Federation Guide](https://www.consul.io/docs/connect/federation)
- [Consul ACL Documentation](https://www.consul.io/docs/security/acl)
- [Consul Monitoring Guide](https://www.consul.io/docs/agent/telemetry) 