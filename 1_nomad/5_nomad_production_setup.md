# Nomad Production Setup - Complete Guide

## Table of Contents
1. [Production Architecture](#production-architecture)
2. [High Availability Setup](#high-availability-setup)
3. [Security Configuration](#security-configuration)
4. [Monitoring and Observability](#monitoring-and-observability)
5. [Backup and Disaster Recovery](#backup-and-disaster-recovery)
6. [Performance Tuning](#performance-tuning)
7. [Deployment Strategies](#deployment-strategies)
8. [Troubleshooting](#troubleshooting)
9. [Maintenance Procedures](#maintenance-procedures)
10. [Best Practices](#best-practices)

---

## Production Architecture

### Production Architecture Overview
```
┌─────────────────────────────────────────────────────────────┐
│                    Production Architecture                  │
├─────────────────────────────────────────────────────────────┤
│ Load Balancer (HAProxy/Nginx)                              │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │   LB 1      │ │   LB 2      │ │   LB 3      │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
│         │               │               │                 │
│                    Server Nodes                            │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │ Server 1    │ │ Server 2    │ │ Server 3    │           │
│ │ (Leader)    │ │ (Follower)  │ │ (Follower)  │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
│         │               │               │                 │
│                    Client Nodes                            │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │ Client 1    │ │ Client 2    │ │ Client 3    │           │
│ │ ┌─┐ ┌─┐ ┌─┐ │ │ ┌─┐ ┌─┐ ┌─┐ │ │ ┌─┐ ┌─┐ ┌─┐ │           │
│ │ │A│ │B│ │C│ │ │ │D│ │E│ │F│ │ │ │G│ │H│ │I│ │           │
│ │ └─┘ └─┘ └─┘ │ │ └─┘ └─┘ └─┘ │ │ └─┘ └─┘ └─┘ │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

### Infrastructure Requirements
```
┌─────────────────────────────────────────────────────────────┐
│                Infrastructure Requirements                  │
├─────────────────────────────────────────────────────────────┤
│ Servers:     3+ nodes for HA                               │
│ Clients:     2+ nodes for redundancy                       │
│ CPU:         4+ cores per node                             │
│ Memory:      8GB+ per node                                 │
│ Storage:     100GB+ SSD per node                           │
│ Network:     1Gbps+ connectivity                           │
│ Load Balancer: HAProxy or Nginx                            │
│ Monitoring:  Prometheus, Grafana, AlertManager             │
│ Logging:     ELK stack or similar                          │
└─────────────────────────────────────────────────────────────┘
```

### Network Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    Network Architecture                     │
├─────────────────────────────────────────────────────────────┤
│ Internet                                                    │
│     │                                                       │
│ Load Balancer (80/443)                                      │
│     │                                                       │
│ Firewall (4646, 4647, 4648)                                │
│     │                                                       │
│ Nomad Servers (4646, 4647, 4648)                           │
│     │                                                       │
│ Nomad Clients (4646, 4647, 4648)                           │
│     │                                                       │
│ Applications (Dynamic ports)                                │
└─────────────────────────────────────────────────────────────┘
```

---

## High Availability Setup

### Server Configuration
```hcl
# /etc/nomad.d/server.hcl
data_dir = "/opt/nomad/data"
bind_addr = "0.0.0.0"

# Server configuration
server {
  enabled = true
  bootstrap_expect = 3
  retry_join = [
    "192.168.1.10:4648",
    "192.168.1.11:4648",
    "192.168.1.12:4648"
  ]
  retry_max = 3
  retry_interval = "15s"
  start_join = [
    "192.168.1.10:4648",
    "192.168.1.11:4648",
    "192.168.1.12:4648"
  ]
  rejoin_after_leave = true
  non_voting_server = false
  redundancy_zone = "zone1"
  upgrade_version = "1.6.0"
  enable_debug = false
}

# Advertise addresses
advertise {
  http = "192.168.1.10"
  rpc  = "192.168.1.10"
  serf = "192.168.1.10"
}

# TLS configuration
tls {
  http = true
  rpc  = true
  ca_file   = "/etc/nomad.d/tls/nomad-ca.pem"
  cert_file = "/etc/nomad.d/tls/server.pem"
  key_file  = "/etc/nomad.d/tls/server-key.pem"
  verify_server_hostname = true
  verify_https_client = true
}

# ACL configuration
acl {
  enabled = true
}

# Vault integration
vault {
  enabled = true
  address = "http://vault:8200"
  create_from_role = "nomad-cluster"
  task_token_ttl = "1h"
  task_token_no_default_policy = true
}

# Consul integration
consul {
  address = "127.0.0.1:8500"
  server_service_name = "nomad"
  client_service_name = "nomad-client"
  auto_advertise = true
  server_auto_join = true
  client_auto_join = true
}

# Telemetry
telemetry {
  collection_interval = "1s"
  disable_hostname = true
  prometheus_metrics = true
  publish_allocation_metrics = true
  publish_node_metrics = true
}

# Logging
log_level = "INFO"
log_json = true
log_file = "/var/log/nomad.log"
log_rotate_bytes = 100000000
log_rotate_max_files = 5
```

### Client Configuration
```hcl
# /etc/nomad.d/client.hcl
data_dir = "/opt/nomad/data"
bind_addr = "0.0.0.0"

# Client configuration
client {
  enabled = true
  servers = [
    "192.168.1.10:4647",
    "192.168.1.11:4647",
    "192.168.1.12:4647"
  ]
  servers_join = [
    "192.168.1.10:4648",
    "192.168.1.11:4648",
    "192.168.1.12:4648"
  ]
  node_class = "web"
  node_pool = "default"
  meta {
    "environment" = "production"
    "rack" = "rack1"
    "zone" = "zone1"
  }
  options {
    "driver.raw_exec.enable" = "1"
    "driver.docker.volumes.enabled" = "true"
    "driver.docker.privileged.enabled" = "false"
  }
  chroot_env {
    "/bin" = "/bin"
    "/lib" = "/lib"
    "/lib64" = "/lib64"
    "/usr/bin" = "/usr/bin"
  }
  reserved {
    cpu    = 500
    memory = 1024
    disk   = 10240
    ports  = ["22", "80", "443"]
  }
}

# Advertise addresses
advertise {
  http = "192.168.1.20"
  rpc  = "192.168.1.20"
  serf = "192.168.1.20"
}

# TLS configuration
tls {
  http = true
  rpc  = true
  ca_file   = "/etc/nomad.d/tls/nomad-ca.pem"
  cert_file = "/etc/nomad.d/tls/client.pem"
  key_file  = "/etc/nomad.d/tls/client-key.pem"
  verify_server_hostname = true
  verify_https_client = false
}

# Telemetry
telemetry {
  collection_interval = "1s"
  disable_hostname = true
  prometheus_metrics = true
  publish_allocation_metrics = true
  publish_node_metrics = true
}

# Logging
log_level = "INFO"
log_json = true
log_file = "/var/log/nomad.log"
log_rotate_bytes = 100000000
log_rotate_max_files = 5
```

### Load Balancer Configuration (HAProxy)
```haproxy
# /etc/haproxy/haproxy.cfg
global
    log /dev/log local0
    log /dev/log local1 notice
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

frontend nomad_http
    bind *:4646
    mode http
    default_backend nomad_servers_http

frontend nomad_rpc
    bind *:4647
    mode tcp
    default_backend nomad_servers_rpc

frontend nomad_serf
    bind *:4648
    mode tcp
    default_backend nomad_servers_serf

backend nomad_servers_http
    mode http
    balance roundrobin
    option httpchk GET /v1/agent/self
    server nomad1 192.168.1.10:4646 check
    server nomad2 192.168.1.11:4646 check
    server nomad3 192.168.1.12:4646 check

backend nomad_servers_rpc
    mode tcp
    balance roundrobin
    server nomad1 192.168.1.10:4647 check
    server nomad2 192.168.1.11:4647 check
    server nomad3 192.168.1.12:4647 check

backend nomad_servers_serf
    mode tcp
    balance roundrobin
    server nomad1 192.168.1.10:4648 check
    server nomad2 192.168.1.11:4648 check
    server nomad3 192.168.1.12:4648 check
```

### Systemd Service Configuration
```ini
# /etc/systemd/system/nomad.service
[Unit]
Description=Nomad
Documentation=https://www.nomadproject.io/docs/
Requires=network-online.target
After=network-online.target
ConditionFileNotEmpty=/etc/nomad.d/server.hcl

[Service]
User=nomad
Group=nomad
ExecStart=/usr/local/bin/nomad agent -config=/etc/nomad.d/
ExecReload=/bin/kill --signal HUP $MAINPID
KillMode=process
KillSignal=SIGINT
LimitNOFILE=65536
LimitNPROC=infinity
Restart=on-failure
RestartSec=2
StartLimitBurst=3
StartLimitIntervalSec=10
TasksMax=infinity
OOMScoreAdjust=-1000

[Install]
WantedBy=multi-user.target
```

---

## Security Configuration

### TLS Certificate Generation
```bash
#!/bin/bash
# generate-tls.sh

# Create directory
mkdir -p /etc/nomad.d/tls
cd /etc/nomad.d/tls

# Generate CA
nomad tls ca create

# Generate server certificates
nomad tls cert create -server -dc=dc1

# Generate client certificates
nomad tls cert create -client -dc=dc1

# Set permissions
chown -R nomad:nomad /etc/nomad.d/tls
chmod 600 /etc/nomad.d/tls/*
```

### ACL Configuration
```hcl
# ACL policies
# management.policy
namespace "default" {
  policy = "write"
}

agent {
  policy = "write"
}

operator {
  policy = "write"
}

quota {
  policy = "write"
}

node {
  policy = "write"
}

# readonly.policy
namespace "default" {
  policy = "read"
}

# app.policy
namespace "default" {
  policy = "read"
  job "web" {
    policy = "write"
  }
  job "api" {
    policy = "write"
  }
}
```

### ACL Token Management
```bash
#!/bin/bash
# setup-acls.sh

# Bootstrap ACL system
nomad acl bootstrap

# Create management token
nomad acl token create -name="management" -type=management

# Create policies
nomad acl policy apply -name="readonly" -description="Read-only access" readonly.policy
nomad acl policy apply -name="app" -description="Application access" app.policy

# Create tokens
nomad acl token create -name="readonly-token" -policy=readonly
nomad acl token create -name="app-token" -policy=app
```

### Network Security
```bash
# Firewall configuration (iptables)
#!/bin/bash

# Allow Nomad ports
iptables -A INPUT -p tcp --dport 4646 -j ACCEPT  # HTTP API
iptables -A INPUT -p tcp --dport 4647 -j ACCEPT  # RPC
iptables -A INPUT -p tcp --dport 4648 -j ACCEPT  # Serf

# Allow Consul ports
iptables -A INPUT -p tcp --dport 8500 -j ACCEPT  # HTTP API
iptables -A INPUT -p tcp --dport 8300 -j ACCEPT  # Server RPC
iptables -A INPUT -p tcp --dport 8301 -j ACCEPT  # LAN Serf
iptables -A INPUT -p tcp --dport 8302 -j ACCEPT  # WAN Serf

# Allow Vault ports
iptables -A INPUT -p tcp --dport 8200 -j ACCEPT  # HTTP API

# Save rules
iptables-save > /etc/iptables/rules.v4
```

---

## Monitoring and Observability

### Prometheus Configuration
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "nomad.rules"

scrape_configs:
  - job_name: 'nomad'
    static_configs:
      - targets: ['nomad1:4646', 'nomad2:4646', 'nomad3:4646']
    metrics_path: '/v1/metrics'
    params:
      format: ['prometheus']
    scrape_interval: 10s

  - job_name: 'consul'
    static_configs:
      - targets: ['consul1:8500', 'consul2:8500', 'consul3:8500']
    metrics_path: '/v1/agent/metrics'
    params:
      format: ['prometheus']

  - job_name: 'vault'
    static_configs:
      - targets: ['vault1:8200', 'vault2:8200', 'vault3:8200']
    metrics_path: '/v1/sys/metrics'
    params:
      format: ['prometheus']
```

### Alerting Rules
```yaml
# nomad.rules
groups:
  - name: nomad
    rules:
      - alert: NomadServerDown
        expr: nomad_server_heartbeat == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Nomad server is down"
          description: "Nomad server has been down for more than 1 minute"

      - alert: NomadClientDown
        expr: nomad_client_heartbeat == 0
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "Nomad client is down"
          description: "Nomad client has been down for more than 2 minutes"

      - alert: HighCPUUsage
        expr: nomad_client_host_cpu_user > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on Nomad client"
          description: "CPU usage is above 80% for more than 5 minutes"

      - alert: HighMemoryUsage
        expr: nomad_client_host_memory_used / nomad_client_host_memory_total > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on Nomad client"
          description: "Memory usage is above 90% for more than 5 minutes"
```

### Grafana Dashboard
```json
{
  "dashboard": {
    "title": "Nomad Production Cluster",
    "panels": [
      {
        "title": "Server Status",
        "type": "stat",
        "targets": [
          {
            "expr": "nomad_server_heartbeat",
            "legendFormat": "{{instance}}"
          }
        ]
      },
      {
        "title": "Client Status",
        "type": "stat",
        "targets": [
          {
            "expr": "nomad_client_heartbeat",
            "legendFormat": "{{instance}}"
          }
        ]
      },
      {
        "title": "Running Allocations",
        "type": "graph",
        "targets": [
          {
            "expr": "nomad_client_allocs_running",
            "legendFormat": "{{instance}}"
          }
        ]
      },
      {
        "title": "CPU Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(nomad_client_host_cpu_user[5m])",
            "legendFormat": "{{instance}}"
          }
        ]
      },
      {
        "title": "Memory Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "nomad_client_host_memory_used / nomad_client_host_memory_total",
            "legendFormat": "{{instance}}"
          }
        ]
      }
    ]
  }
}
```

### Logging Configuration
```hcl
# Logging configuration
log_level = "INFO"
log_json = true
log_file = "/var/log/nomad.log"
log_rotate_bytes = 100000000
log_rotate_max_files = 5

# Log forwarding to centralized logging
template {
  data = <<EOH
<source>
  @type tail
  path /var/log/nomad.log
  pos_file /var/log/fluentd-nomad.log.pos
  tag nomad
  <parse>
    @type json
  </parse>
</source>

<match nomad>
  @type elasticsearch
  host elasticsearch
  port 9200
  logstash_format true
  logstash_prefix nomad
</match>
EOH
  destination = "local/fluent.conf"
}
```

---

## Backup and Disaster Recovery

### Backup Strategy
```bash
#!/bin/bash
# backup-nomad.sh

# Configuration
BACKUP_DIR="/backup/nomad"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup job definitions
nomad job list -json | jq -r '.[].ID' | while read job; do
  nomad job inspect $job > $BACKUP_DIR/jobs_${DATE}_${job}.json
done

# Backup ACL policies
nomad acl policy list -json | jq -r '.[].Name' | while read policy; do
  nomad acl policy info $policy > $BACKUP_DIR/policies_${DATE}_${policy}.json
done

# Backup ACL tokens
nomad acl token list -json > $BACKUP_DIR/tokens_${DATE}.json

# Backup configuration files
cp /etc/nomad.d/*.hcl $BACKUP_DIR/config_${DATE}/

# Compress backup
tar -czf $BACKUP_DIR/nomad_backup_${DATE}.tar.gz -C $BACKUP_DIR .

# Cleanup old backups
find $BACKUP_DIR -name "nomad_backup_*.tar.gz" -mtime +$RETENTION_DAYS -delete
```

### Disaster Recovery Plan
```bash
#!/bin/bash
# restore-nomad.sh

# Configuration
BACKUP_FILE=$1
RESTORE_DIR="/tmp/nomad_restore"

# Extract backup
mkdir -p $RESTORE_DIR
tar -xzf $BACKUP_FILE -C $RESTORE_DIR

# Restore job definitions
find $RESTORE_DIR -name "jobs_*.json" | while read job_file; do
  nomad job run $job_file
done

# Restore ACL policies
find $RESTORE_DIR -name "policies_*.json" | while read policy_file; do
  nomad acl policy apply -file $policy_file
done

# Restore configuration
cp $RESTORE_DIR/config_*/* /etc/nomad.d/

# Restart Nomad
systemctl restart nomad

# Cleanup
rm -rf $RESTORE_DIR
```

### High Availability Testing
```bash
#!/bin/bash
# ha-test.sh

# Test server failover
echo "Testing server failover..."

# Stop leader
LEADER=$(nomad server members | grep leader | awk '{print $1}')
echo "Stopping leader: $LEADER"
ssh $LEADER "systemctl stop nomad"

# Wait for failover
sleep 30

# Check new leader
NEW_LEADER=$(nomad server members | grep leader | awk '{print $1}')
echo "New leader: $NEW_LEADER"

# Restart old leader
echo "Restarting old leader..."
ssh $LEADER "systemctl start nomad"

# Test client failover
echo "Testing client failover..."

# Stop a client
CLIENT=$(nomad node status | grep ready | head -1 | awk '{print $1}')
echo "Stopping client: $CLIENT"
ssh $CLIENT "systemctl stop nomad"

# Check job rescheduling
sleep 30
nomad job status

# Restart client
echo "Restarting client..."
ssh $CLIENT "systemctl start nomad"
```

---

## Performance Tuning

### Server Performance Tuning
```hcl
# Server performance configuration
server {
  enabled = true
  bootstrap_expect = 3
  retry_join = ["192.168.1.10:4648", "192.168.1.11:4648", "192.168.1.12:4648"]
  retry_max = 3
  retry_interval = "15s"
  start_join = ["192.168.1.10:4648", "192.168.1.11:4648", "192.168.1.12:4648"]
  rejoin_after_leave = true
  non_voting_server = false
  redundancy_zone = "zone1"
  upgrade_version = "1.6.0"
  enable_debug = false
}

# Performance limits
limits {
  http_max_conns_per_client = 100
  rpc_max_conns_per_client = 100
}

# Telemetry optimization
telemetry {
  collection_interval = "1s"
  disable_hostname = true
  prometheus_metrics = true
  publish_allocation_metrics = true
  publish_node_metrics = true
  statsite_address = ""
  statsd_address = ""
  disable_hostname = true
  enable_hostname_label = false
}
```

### Client Performance Tuning
```hcl
# Client performance configuration
client {
  enabled = true
  servers = ["192.168.1.10:4647", "192.168.1.11:4647", "192.168.1.12:4647"]
  node_class = "web"
  node_pool = "default"
  meta {
    "environment" = "production"
    "rack" = "rack1"
    "zone" = "zone1"
  }
  options {
    "driver.raw_exec.enable" = "1"
    "driver.docker.volumes.enabled" = "true"
    "driver.docker.privileged.enabled" = "false"
    "driver.docker.allow_runtimes" = "runc"
    "driver.docker.cleanup_image" = "true"
  }
  chroot_env {
    "/bin" = "/bin"
    "/lib" = "/lib"
    "/lib64" = "/lib64"
    "/usr/bin" = "/usr/bin"
  }
  reserved {
    cpu    = 500
    memory = 1024
    disk   = 10240
    ports  = ["22", "80", "443"]
  }
  gc_interval = "1m"
  gc_disk_usage_threshold = 80
  gc_inode_usage_threshold = 70
  gc_max_allocs = 50
  no_host_uuid = false
}
```

### System Performance Tuning
```bash
#!/bin/bash
# system-tuning.sh

# Increase file descriptors
echo "* soft nofile 65536" >> /etc/security/limits.conf
echo "* hard nofile 65536" >> /etc/security/limits.conf

# Increase process limits
echo "* soft nproc 32768" >> /etc/security/limits.conf
echo "* hard nproc 32768" >> /etc/security/limits.conf

# Kernel tuning
cat >> /etc/sysctl.conf << EOF
# Network tuning
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 5000
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.tcp_max_tw_buckets = 2000000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.ip_local_port_range = 1024 65535

# Memory tuning
vm.swappiness = 10
vm.dirty_ratio = 15
vm.dirty_background_ratio = 5
EOF

# Apply sysctl changes
sysctl -p
```

---

## Deployment Strategies

### Blue-Green Deployment
```hcl
job "blue-green" {
  datacenters = ["dc1"]
  type = "service"

  update {
    stagger      = "10s"
    max_parallel = 1
    health_check = "checks"
    auto_revert  = true
    auto_promote = false
  }

  group "app" {
    count = 3

    task "app" {
      driver = "docker"
      config {
        image = "myapp:blue"
      }

      service {
        name = "app-blue"
        port = "8080"
        tags = ["blue"]

        check {
          type     = "http"
          path     = "/health"
          interval = "10s"
          timeout  = "2s"
        }
      }
    }
  }
}
```

### Canary Deployment
```hcl
job "canary" {
  datacenters = ["dc1"]
  type = "service"

  group "stable" {
    count = 4

    task "app" {
      driver = "docker"
      config {
        image = "myapp:stable"
      }

      service {
        name = "app"
        port = "8080"
        tags = ["stable"]

        check {
          type     = "http"
          path     = "/health"
          interval = "10s"
          timeout  = "2s"
        }
      }
    }
  }

  group "canary" {
    count = 1

    task "app" {
      driver = "docker"
      config {
        image = "myapp:canary"
      }

      service {
        name = "app-canary"
        port = "8080"
        tags = ["canary"]

        check {
          type     = "http"
          path     = "/health"
          interval = "10s"
          timeout  = "2s"
        }
      }
    }
  }
}
```

### Rolling Updates
```hcl
job "rolling-update" {
  datacenters = ["dc1"]
  type = "service"

  update {
    stagger      = "30s"
    max_parallel = 1
    health_check = "checks"
    auto_revert  = true
    min_healthy_time = "10s"
    healthy_deadline = "5m"
    progress_deadline = "10m"
  }

  group "app" {
    count = 5

    task "app" {
      driver = "docker"
      config {
        image = "myapp:latest"
      }

      service {
        name = "app"
        port = "8080"

        check {
          type     = "http"
          path     = "/health"
          interval = "10s"
          timeout  = "2s"
        }
      }
    }
  }
}
```

---

## Troubleshooting

### Common Issues and Solutions

#### Server Issues
```bash
# Check server status
nomad server members

# Check server logs
journalctl -u nomad -f

# Check server configuration
nomad agent info

# Restart server
systemctl restart nomad
```

#### Client Issues
```bash
# Check client status
nomad node status

# Check client logs
journalctl -u nomad -f

# Check client configuration
nomad agent info

# Restart client
systemctl restart nomad
```

#### Job Issues
```bash
# Check job status
nomad job status <job-name>

# Check allocation status
nomad alloc list -job=<job-name>

# Check allocation logs
nomad alloc logs <alloc-id>

# Restart job
nomad job restart <job-name>
```

#### Network Issues
```bash
# Check connectivity
telnet nomad-server 4646
telnet nomad-server 4647
telnet nomad-server 4648

# Check firewall
iptables -L

# Check DNS
nslookup nomad-server
```

### Debug Commands
```bash
# Enable debug logging
NOMAD_LOG_LEVEL=DEBUG nomad agent -config=/etc/nomad.d/

# Get detailed information
nomad job inspect <job-name>
nomad alloc status <alloc-id>
nomad node status <node-id>

# Check metrics
curl http://localhost:4646/v1/metrics?format=prometheus
```

---

## Maintenance Procedures

### Regular Maintenance Tasks
```bash
#!/bin/bash
# maintenance.sh

# Daily tasks
echo "Running daily maintenance..."

# Check cluster health
nomad server members
nomad node status

# Check job status
nomad job list

# Check resource usage
nomad node status -json | jq '.[] | {node: .Name, cpu: .Resources.CPU, memory: .Resources.Memory}'

# Weekly tasks
if [ $(date +%u) -eq 1 ]; then
    echo "Running weekly maintenance..."
    
    # Backup cluster state
    ./backup-nomad.sh
    
    # Clean up old allocations
    nomad system gc
    
    # Update job definitions
    find /etc/nomad/jobs -name "*.nomad" -exec nomad job run {} \;
fi

# Monthly tasks
if [ $(date +%d) -eq 1 ]; then
    echo "Running monthly maintenance..."
    
    # Update Nomad
    wget https://releases.hashicorp.com/nomad/1.6.0/nomad_1.6.0_linux_amd64.zip
    unzip nomad_1.6.0_linux_amd64.zip
    sudo mv nomad /usr/local/bin/
    
    # Restart services
    systemctl restart nomad
fi
```

### Upgrade Procedures
```bash
#!/bin/bash
# upgrade-nomad.sh

VERSION=$1

if [ -z "$VERSION" ]; then
    echo "Usage: $0 <version>"
    exit 1
fi

echo "Upgrading Nomad to version $VERSION"

# Download new version
wget https://releases.hashicorp.com/nomad/${VERSION}/nomad_${VERSION}_linux_amd64.zip
unzip nomad_${VERSION}_linux_amd64.zip

# Backup current version
cp /usr/local/bin/nomad /usr/local/bin/nomad.backup

# Install new version
sudo mv nomad /usr/local/bin/

# Restart Nomad
systemctl restart nomad

# Verify upgrade
nomad version

echo "Upgrade completed successfully"
```

---

## Best Practices

### Security Best Practices
1. **Enable TLS**: Use TLS for all communications
2. **Enable ACLs**: Use ACLs for access control
3. **Use Vault**: Integrate with Vault for secrets management
4. **Network Security**: Implement network segmentation
5. **Regular Updates**: Keep Nomad updated

### Performance Best Practices
1. **Resource Planning**: Plan resource requirements carefully
2. **Monitoring**: Monitor cluster performance
3. **Scaling**: Use appropriate scaling strategies
4. **Optimization**: Optimize job configurations
5. **Cleanup**: Regularly clean up old allocations

### Operational Best Practices
1. **Documentation**: Document configurations and procedures
2. **Testing**: Test in development first
3. **Backup**: Backup configurations and data
4. **Monitoring**: Set up comprehensive monitoring
5. **Disaster Recovery**: Plan for failure scenarios

### Deployment Best Practices
1. **Blue-Green**: Use blue-green deployments for zero downtime
2. **Canary**: Use canary deployments for risk mitigation
3. **Rolling Updates**: Use rolling updates for gradual deployment
4. **Health Checks**: Implement proper health checks
5. **Rollback**: Plan for rollback procedures

---

## Next Steps

After setting up production Nomad, you'll be ready to:

1. **Scale Operations**: Scale to larger clusters
2. **Advanced Integration**: Integrate with more tools
3. **Customization**: Build custom solutions
4. **Optimization**: Optimize for specific use cases

### Practice Projects

1. **Multi-Region Deployment**: Deploy across multiple regions
2. **Service Mesh**: Implement service mesh architecture
3. **Security Hardening**: Implement comprehensive security
4. **Performance Optimization**: Optimize cluster performance

### Resources

- [Nomad Production Guide](https://www.nomadproject.io/docs/guides/production/)
- [Nomad Security](https://www.nomadproject.io/docs/security/)
- [Nomad Performance](https://www.nomadproject.io/docs/internals/performance/)
- [Nomad Best Practices](https://www.nomadproject.io/docs/guides/best-practices/) 