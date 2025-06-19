# Consul Production Setup - Complete Guide

## Table of Contents
1. [Production Architecture](#production-architecture)
2. [Security Configuration](#security-configuration)
3. [High Availability Setup](#high-availability-setup)
4. [Monitoring and Alerting](#monitoring-and-alerting)
5. [Backup and Disaster Recovery](#backup-and-disaster-recovery)
6. [Performance Tuning](#performance-tuning)
7. [Deployment Strategies](#deployment-strategies)
8. [Maintenance and Operations](#maintenance-and-operations)

---

## Production Architecture

### Recommended Production Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                Production Consul Cluster                   │
├─────────────────────────────────────────────────────────────┤
│                    Load Balancer                           │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │ HAProxy     │ │ HAProxy     │ │ HAProxy     │           │
│ │ (Active)    │ │ (Passive)   │ │ (Passive)   │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
│         │               │               │                 │
│         └───────────────┼───────────────┘                 │
│                         │                                 │
│                    Server Nodes                           │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │ Server 1    │ │ Server 2    │ │ Server 3    │           │
│ │ (Leader)    │ │ (Follower)  │ │ (Follower)  │           │
│ │ DC: dc1     │ │ DC: dc1     │ │ DC: dc1     │           │
│ │ Zone: A     │ │ Zone: B     │ │ Zone: C     │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
│         │               │               │                 │
│         └───────────────┼───────────────┘                 │
│                         │                                 │
│                    Client Nodes                           │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │ Client 1    │ │ Client 2    │ │ Client 3    │           │
│ │ Services:   │ │ Services:   │ │ Services:   │           │
│ │ web, api    │ │ db, cache   │ │ monitoring  │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

### Infrastructure Requirements
```
┌─────────────────────────────────────────────────────────────┐
│                Infrastructure Requirements                 │
├─────────────────────────────────────────────────────────────┤
│ Servers:      3-5 server nodes (odd number)               │
│ Clients:      As needed for services                      │
│ CPU:          2+ cores per node                           │
│ Memory:       4GB+ per node                               │
│ Storage:      20GB+ SSD per node                          │
│ Network:      Low latency, high bandwidth                 │
│ Security:     Firewall, VPN, TLS                          │
└─────────────────────────────────────────────────────────────┘
```

### Network Architecture
```hcl
# Network segmentation
# Public Network: Load balancers, external access
# Private Network: Consul servers and clients
# Management Network: SSH, monitoring, backup

# Firewall rules
# - Allow 8300 (server RPC)
# - Allow 8301 (LAN gossip)
# - Allow 8302 (WAN gossip)
# - Allow 8500 (HTTP API)
# - Allow 8600 (DNS)
# - Allow 8600/udp (DNS)
```

---

## Security Configuration

### TLS Configuration
```hcl
# consul-server.hcl
tls {
  defaults {
    ca_file = "/etc/consul/tls/ca.crt"
    cert_file = "/etc/consul/tls/server.crt"
    key_file = "/etc/consul/tls/server.key"
    verify_incoming = true
    verify_outgoing = true
    verify_server_hostname = true
  }
  
  internal_rpc {
    verify_server_hostname = true
  }
}
```

### Gossip Encryption
```hcl
# consul-server.hcl
encrypt = "your-gossip-encryption-key"
encrypt_verify_incoming = true
encrypt_verify_outgoing = true
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
    default = "your-default-token"
  }
}
```

### Security Best Practices
```bash
# Generate TLS certificates
openssl genrsa -out ca.key 2048
openssl req -new -x509 -key ca.key -out ca.crt -days 365

# Generate server certificate
openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -out server.crt

# Generate gossip encryption key
consul keygen

# Set proper file permissions
chmod 600 /etc/consul/tls/*
chown consul:consul /etc/consul/tls/*
```

---

## High Availability Setup

### Server Configuration
```hcl
# consul-server-1.hcl
datacenter = "dc1"
primary_datacenter = "dc1"
server = true
bootstrap_expect = 3
retry_join = ["192.168.1.11", "192.168.1.12"]
bind_addr = "192.168.1.10"
advertise_addr = "192.168.1.10"
client_addr = "0.0.0.0"
ui_config {
  enabled = true
}

# TLS configuration
tls {
  defaults {
    ca_file = "/etc/consul/tls/ca.crt"
    cert_file = "/etc/consul/tls/server.crt"
    key_file = "/etc/consul/tls/server.key"
    verify_incoming = true
    verify_outgoing = true
  }
}

# ACL configuration
acl {
  enabled = true
  default_policy = "deny"
  enable_token_persistence = true
}

# Performance tuning
performance {
  raft_multiplier = 1
  leave_drain_time = "5s"
}

# Logging
log_level = "INFO"
log_file = "/var/log/consul/consul.log"
log_rotate_max_files = 5
log_rotate_max_size = "10MB"
```

### Client Configuration
```hcl
# consul-client.hcl
datacenter = "dc1"
server = false
retry_join = ["192.168.1.10", "192.168.1.11", "192.168.1.12"]
bind_addr = "192.168.1.20"
advertise_addr = "192.168.1.20"
client_addr = "0.0.0.0"

# TLS configuration
tls {
  defaults {
    ca_file = "/etc/consul/tls/ca.crt"
    cert_file = "/etc/consul/tls/client.crt"
    key_file = "/etc/consul/tls/client.key"
    verify_incoming = false
    verify_outgoing = true
  }
}

# ACL configuration
acl {
  enabled = true
  default_policy = "deny"
  tokens {
    default = "your-client-token"
  }
}
```

### Load Balancer Configuration
```haproxy
# haproxy.cfg
global
    log /dev/log local0
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log global
    mode http
    option httplog
    option dontlognull
    timeout connect 5000
    timeout client 50000
    timeout server 50000

frontend consul_http
    bind *:8500
    default_backend consul_servers

backend consul_servers
    balance roundrobin
    option httpchk GET /v1/status/leader
    server server1 192.168.1.10:8500 check
    server server2 192.168.1.11:8500 check
    server server3 192.168.1.12:8500 check
```

---

## Monitoring and Alerting

### Prometheus Configuration
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'consul'
    static_configs:
      - targets: ['192.168.1.10:8500', '192.168.1.11:8500', '192.168.1.12:8500']
    metrics_path: /v1/agent/metrics
    params:
      format: ['prometheus']
    scrape_interval: 15s
```

### Grafana Dashboard
```json
{
  "dashboard": {
    "title": "Consul Cluster Dashboard",
    "panels": [
      {
        "title": "Raft Leader",
        "type": "stat",
        "targets": [
          {
            "expr": "consul_raft_leader",
            "legendFormat": "Leader"
          }
        ]
      },
      {
        "title": "Raft Peers",
        "type": "stat",
        "targets": [
          {
            "expr": "consul_raft_peers",
            "legendFormat": "Peers"
          }
        ]
      },
      {
        "title": "Memory Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "consul_runtime_alloc_bytes",
            "legendFormat": "Memory"
          }
        ]
      }
    ]
  }
}
```

### Alerting Rules
```yaml
# alertmanager.yml
groups:
  - name: consul
    rules:
      - alert: ConsulNoLeader
        expr: consul_raft_leader == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Consul cluster has no leader"
          
      - alert: ConsulHighMemory
        expr: consul_runtime_alloc_bytes > 2e9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Consul using more than 2GB memory"
          
      - alert: ConsulServerDown
        expr: up{job="consul"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Consul server is down"
```

### Health Check Scripts
```bash
#!/bin/bash
# check-consul-health.sh

# Check if Consul is running
if ! systemctl is-active --quiet consul; then
    echo "CRITICAL: Consul service is not running"
    exit 2
fi

# Check if Consul is responding
if ! curl -s http://localhost:8500/v1/status/leader > /dev/null; then
    echo "CRITICAL: Consul API is not responding"
    exit 2
fi

# Check if there's a leader
LEADER=$(curl -s http://localhost:8500/v1/status/leader)
if [ "$LEADER" = "null" ]; then
    echo "CRITICAL: No Consul leader"
    exit 2
fi

# Check number of peers
PEERS=$(consul operator raft list-peers | grep -c "follower\|leader")
if [ $PEERS -lt 3 ]; then
    echo "WARNING: Only $PEERS peers in cluster"
    exit 1
fi

echo "OK: Consul cluster is healthy"
exit 0
```

---

## Backup and Disaster Recovery

### Automated Backup Script
```bash
#!/bin/bash
# backup-consul.sh

set -e

BACKUP_DIR="/backup/consul"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_PATH="$BACKUP_DIR/$DATE"

# Create backup directory
mkdir -p $BACKUP_PATH

# Set Consul token if using ACLs
export CONSUL_HTTP_TOKEN="your-backup-token"

# Backup KV store
echo "Backing up KV store..."
consul kv export > $BACKUP_PATH/kv-backup.json

# Backup ACLs
echo "Backing up ACLs..."
consul acl policy list > $BACKUP_PATH/acl-policies.json
consul acl token list > $BACKUP_PATH/acl-tokens.json

# Backup configuration
echo "Backing up configuration..."
cp -r /etc/consul.d/* $BACKUP_PATH/config/

# Backup data directory
echo "Backing up data directory..."
tar -czf $BACKUP_PATH/data-backup.tar.gz /opt/consul/data/

# Create backup manifest
cat > $BACKUP_PATH/manifest.json <<EOF
{
  "backup_date": "$(date -Iseconds)",
  "consul_version": "$(consul version | head -1)",
  "datacenter": "$(consul info | grep datacenter | awk '{print $2}')",
  "files": [
    "kv-backup.json",
    "acl-policies.json",
    "acl-tokens.json",
    "config/",
    "data-backup.tar.gz"
  ]
}
EOF

# Clean up old backups (keep last 7 days)
find $BACKUP_DIR -type d -mtime +7 -exec rm -rf {} \;

echo "Backup completed: $BACKUP_PATH"
```

### Disaster Recovery Plan
```bash
#!/bin/bash
# restore-consul.sh

set -e

BACKUP_PATH="$1"
if [ -z "$BACKUP_PATH" ]; then
    echo "Usage: $0 <backup-path>"
    exit 1
fi

# Set Consul token if using ACLs
export CONSUL_HTTP_TOKEN="your-restore-token"

# Stop Consul
systemctl stop consul

# Restore data directory
echo "Restoring data directory..."
tar -xzf $BACKUP_PATH/data-backup.tar.gz -C /

# Restore configuration
echo "Restoring configuration..."
cp -r $BACKUP_PATH/config/* /etc/consul.d/

# Start Consul
systemctl start consul

# Wait for Consul to be ready
sleep 30

# Restore KV store
echo "Restoring KV store..."
consul kv import < $BACKUP_PATH/kv-backup.json

# Restore ACLs
echo "Restoring ACLs..."
# Note: ACL restoration requires manual intervention
echo "Please manually restore ACLs from: $BACKUP_PATH/acl-policies.json"

echo "Restore completed"
```

### Backup Monitoring
```bash
#!/bin/bash
# monitor-backups.sh

BACKUP_DIR="/backup/consul"
ALERT_EMAIL="admin@example.com"

# Check if backup directory exists
if [ ! -d "$BACKUP_DIR" ]; then
    echo "CRITICAL: Backup directory does not exist"
    exit 2
fi

# Find latest backup
LATEST_BACKUP=$(find $BACKUP_DIR -type d -name "20*" | sort | tail -1)

if [ -z "$LATEST_BACKUP" ]; then
    echo "CRITICAL: No backups found"
    exit 2
fi

# Check backup age (in hours)
BACKUP_AGE=$(( ($(date +%s) - $(stat -c %Y $LATEST_BACKUP)) / 3600 ))

if [ $BACKUP_AGE -gt 24 ]; then
    echo "WARNING: Latest backup is $BACKUP_AGE hours old"
    exit 1
fi

# Check backup size
BACKUP_SIZE=$(du -sm $LATEST_BACKUP | awk '{print $1}')

if [ $BACKUP_SIZE -lt 10 ]; then
    echo "WARNING: Backup size is only ${BACKUP_SIZE}MB"
    exit 1
fi

echo "OK: Backup is recent and complete"
exit 0
```

---

## Performance Tuning

### Server Performance Tuning
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

limits {
  http_max_conns_per_client = 200
  rpc_hold_timeout = "7s"
  rpc_rate = 100
  rpc_burst = 100
}

telemetry {
  prometheus_retention_time = "24h"
  disable_hostname = true
  statsd_address = "localhost:8125"
}
```

### Client Performance Tuning
```hcl
# consul-client.hcl
performance {
  raft_multiplier = 1
  leave_drain_time = "5s"
  server_health_verification = true
}

limits {
  http_max_conns_per_client = 100
  rpc_hold_timeout = "7s"
  rpc_rate = 50
  rpc_burst = 50
}
```

### System Tuning
```bash
#!/bin/bash
# tune-system.sh

# Increase file descriptor limits
echo "consul soft nofile 65536" >> /etc/security/limits.conf
echo "consul hard nofile 65536" >> /etc/security/limits.conf

# Optimize network settings
cat >> /etc/sysctl.conf <<EOF
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 5000
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 300
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl = 15
EOF

sysctl -p

# Optimize disk I/O
echo 'ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/scheduler}="deadline"' >> /etc/udev/rules.d/60-scheduler.rules
```

---

## Deployment Strategies

### Blue-Green Deployment
```bash
#!/bin/bash
# blue-green-deploy.sh

# Deploy new Consul version to green environment
echo "Deploying to green environment..."

# Update configuration
cp consul-new.hcl /etc/consul.d/consul.hcl

# Restart Consul
systemctl restart consul

# Wait for health checks
sleep 30

# Verify cluster health
if consul operator raft list-peers | grep -q "leader"; then
    echo "Green deployment successful"
    
    # Update load balancer to point to green
    # (implementation depends on your load balancer)
    
    echo "Traffic switched to green environment"
else
    echo "Green deployment failed, rolling back..."
    systemctl stop consul
    cp consul-old.hcl /etc/consul.d/consul.hcl
    systemctl start consul
    exit 1
fi
```

### Rolling Update
```bash
#!/bin/bash
# rolling-update.sh

SERVERS=("192.168.1.10" "192.168.1.11" "192.168.1.12")

for server in "${SERVERS[@]}"; do
    echo "Updating server: $server"
    
    # Update server
    ssh consul@$server "sudo systemctl stop consul"
    ssh consul@$server "sudo cp consul-new.hcl /etc/consul.d/consul.hcl"
    ssh consul@$server "sudo systemctl start consul"
    
    # Wait for server to join cluster
    sleep 30
    
    # Verify server health
    if ! consul members | grep -q "$server"; then
        echo "Server $server failed to join cluster"
        exit 1
    fi
    
    echo "Server $server updated successfully"
done

echo "Rolling update completed"
```

---

## Maintenance and Operations

### Regular Maintenance Tasks
```bash
#!/bin/bash
# maintenance-tasks.sh

# Daily tasks
echo "Running daily maintenance tasks..."

# Check cluster health
consul operator raft list-peers

# Check service health
consul health check

# Check ACL token expiration
consul acl token list | grep -E "expired|expiring"

# Weekly tasks
if [ $(date +%u) -eq 1 ]; then
    echo "Running weekly maintenance tasks..."
    
    # Rotate logs
    logrotate /etc/logrotate.d/consul
    
    # Clean up old snapshots
    find /opt/consul/data/snapshots -name "*.snap" -mtime +7 -delete
    
    # Update system packages
    yum update -y
fi

# Monthly tasks
if [ $(date +%d) -eq 1 ]; then
    echo "Running monthly maintenance tasks..."
    
    # Rotate CA certificates
    consul connect ca rotate
    
    # Review and update ACL policies
    echo "Please review ACL policies"
    
    # Performance review
    echo "Please review performance metrics"
fi
```

### Monitoring Scripts
```bash
#!/bin/bash
# monitor-consul.sh

# Check cluster health
check_cluster_health() {
    if ! consul operator raft list-peers | grep -q "leader"; then
        echo "CRITICAL: No leader in cluster"
        return 2
    fi
    
    PEER_COUNT=$(consul operator raft list-peers | grep -c "follower\|leader")
    if [ $PEER_COUNT -lt 3 ]; then
        echo "WARNING: Only $PEER_COUNT peers in cluster"
        return 1
    fi
    
    echo "OK: Cluster is healthy"
    return 0
}

# Check service health
check_service_health() {
    FAILING_SERVICES=$(consul health check | grep -c "critical")
    if [ $FAILING_SERVICES -gt 0 ]; then
        echo "WARNING: $FAILING_SERVICES services are critical"
        return 1
    fi
    
    echo "OK: All services are healthy"
    return 0
}

# Check memory usage
check_memory_usage() {
    MEMORY_USAGE=$(consul info | grep "runtime.alloc_bytes" | awk '{print $2}')
    MEMORY_MB=$((MEMORY_USAGE / 1024 / 1024))
    
    if [ $MEMORY_MB -gt 2048 ]; then
        echo "WARNING: High memory usage: ${MEMORY_MB}MB"
        return 1
    fi
    
    echo "OK: Memory usage is normal: ${MEMORY_MB}MB"
    return 0
}

# Run all checks
check_cluster_health
check_service_health
check_memory_usage
```

### Troubleshooting Guide
```bash
#!/bin/bash
# troubleshoot-consul.sh

echo "Consul Troubleshooting Guide"
echo "============================"

# Check Consul status
echo "1. Checking Consul service status..."
systemctl status consul

# Check logs
echo "2. Checking recent logs..."
journalctl -u consul --since "1 hour ago" | tail -20

# Check cluster members
echo "3. Checking cluster members..."
consul members

# Check Raft status
echo "4. Checking Raft status..."
consul operator raft list-peers

# Check service health
echo "5. Checking service health..."
consul health check

# Check API connectivity
echo "6. Checking API connectivity..."
curl -s http://localhost:8500/v1/status/leader

# Check DNS
echo "7. Checking DNS..."
dig @localhost -p 8600 consul.service.consul

# Check network connectivity
echo "8. Checking network connectivity..."
netstat -tlnp | grep consul

echo "Troubleshooting complete"
```

---

## Next Steps

After setting up production Consul, you'll be ready to:

1. **Scale Operations**: Handle larger clusters and workloads
2. **Advanced Monitoring**: Implement comprehensive observability
3. **Automation**: Automate deployment and maintenance tasks
4. **Integration**: Integrate with other production tools

### Practice Projects

1. **Multi-DC Setup**: Set up federation between datacenters
2. **Automated Deployment**: Implement CI/CD for Consul
3. **Advanced Monitoring**: Build comprehensive dashboards
4. **Disaster Recovery**: Test backup and recovery procedures

### Resources

- [Consul Production Deployment](https://www.consul.io/docs/install/production)
- [Consul Security Best Practices](https://www.consul.io/docs/security)
- [Consul Performance Tuning](https://www.consul.io/docs/agent/options#performance)
- [Consul Troubleshooting](https://www.consul.io/docs/troubleshooting) 