# Production Considerations

## Overview
Important considerations when moving from local development to production with Nomad, Consul, and Vault.

## Prerequisites
- Complete [Advanced Features](6_advanced_features.md)

## Security Considerations

### 1. Vault Security
```bash
# Disable dev mode in production
# Instead of: vault server -dev
# Use: vault server -config=/etc/vault.d/vault.hcl

# Production Vault configuration
storage "consul" {
  path = "vault/"
  service = "vault"
}

listener "tcp" {
  address = "0.0.0.0:8200"
  tls_disable = false
  tls_cert_file = "/etc/vault.d/tls/server.crt"
  tls_key_file = "/etc/vault.d/tls/server.key"
}

api_addr = "https://vault.example.com:8200"
cluster_addr = "https://vault.example.com:8201"

ui = true
disable_mlock = true
```

### 2. Consul Security
```hcl
# Enable ACLs
acl {
  enabled = true
  default_policy = "deny"
  enable_token_persistence = true
}

# Enable TLS
tls {
  defaults {
    ca_file = "/etc/consul.d/tls/ca.crt"
    cert_file = "/etc/consul.d/tls/server.crt"
    key_file = "/etc/consul.d/tls/server.key"
    verify_incoming = true
    verify_outgoing = true
  }
}
```

### 3. Nomad Security
```hcl
# Enable TLS
tls {
  http = true
  rpc  = true
  ca_file   = "/etc/nomad.d/tls/nomad-ca.pem"
  cert_file = "/etc/nomad.d/tls/server.pem"
  key_file  = "/etc/nomad.d/tls/server-key.pem"
  verify_server_hostname = true
  verify_https_client = true
}

# Enable ACLs
acl {
  enabled = true
}
```

## High Availability Setup

### 1. Multi-Node Docker Compose
```yaml
version: '3.8'

services:
  # Vault Cluster
  vault-1:
    image: vault:latest
    ports:
      - "8200:8200"
    environment:
      - VAULT_CACERT=/vault/userconfig/vault-ha-tls/vault.ca
    volumes:
      - ./vault-config:/vault/config
    command: vault server -config=/vault/config/vault-1.hcl

  vault-2:
    image: vault:latest
    ports:
      - "8201:8200"
    environment:
      - VAULT_CACERT=/vault/userconfig/vault-ha-tls/vault.ca
    volumes:
      - ./vault-config:/vault/config
    command: vault server -config=/vault/config/vault-2.hcl

  vault-3:
    image: vault:latest
    ports:
      - "8202:8200"
    environment:
      - VAULT_CACERT=/vault/userconfig/vault-ha-tls/vault.ca
    volumes:
      - ./vault-config:/vault/config
    command: vault server -config=/vault/config/vault-3.hcl

  # Consul Cluster
  consul-1:
    image: consul:latest
    ports:
      - "8500:8500"
    command: consul agent -server -bootstrap-expect=3 -ui -client=0.0.0.0

  consul-2:
    image: consul:latest
    ports:
      - "8501:8500"
    command: consul agent -server -join=consul-1 -ui -client=0.0.0.0

  consul-3:
    image: consul:latest
    ports:
      - "8502:8500"
    command: consul agent -server -join=consul-1 -ui -client=0.0.0.0

  # Nomad Cluster
  nomad-1:
    image: nomad:latest
    ports:
      - "4646:4646"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: nomad agent -server -bootstrap-expect=3

  nomad-2:
    image: nomad:latest
    ports:
      - "4647:4646"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: nomad agent -server -join=nomad-1

  nomad-3:
    image: nomad:latest
    ports:
      - "4648:4646"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: nomad agent -server -join=nomad-1
```

## Backup and Recovery

### 1. Vault Backup
```bash
# Backup Vault data
vault operator raft snapshot save backup.snap

# Restore from backup
vault operator raft snapshot restore backup.snap
```

### 2. Consul Backup
```bash
# Backup Consul KV store
consul kv export > consul-backup.json

# Restore Consul KV store
consul kv import @consul-backup.json
```

### 3. Nomad Backup
```bash
# Nomad state is stored in Consul
# Backup Consul to backup Nomad state
consul kv export nomad/ > nomad-backup.json
```

## Monitoring and Alerting

### 1. Health Checks
```bash
# Vault health
curl -s http://localhost:8200/v1/sys/health | jq

# Consul health
consul health check

# Nomad health
nomad server members
nomad node status
```

### 2. Metrics Collection
```yaml
# Prometheus configuration for production
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'vault'
    static_configs:
      - targets: ['vault-1:8200', 'vault-2:8200', 'vault-3:8200']
    metrics_path: /v1/sys/metrics
    params:
      format: ['prometheus']

  - job_name: 'consul'
    static_configs:
      - targets: ['consul-1:8500', 'consul-2:8500', 'consul-3:8500']
    metrics_path: /v1/agent/metrics
    params:
      format: ['prometheus']

  - job_name: 'nomad'
    static_configs:
      - targets: ['nomad-1:4646', 'nomad-2:4646', 'nomad-3:4646']
    metrics_path: /v1/metrics
    params:
      format: ['prometheus']
```

## Performance Tuning

### 1. Resource Limits
```yaml
# Docker Compose with resource limits
services:
  vault:
    image: vault:latest
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M

  consul:
    image: consul:latest
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M

  nomad:
    image: nomad:latest
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
```

### 2. Network Optimization
```yaml
# Use host networking for better performance
services:
  vault:
    network_mode: "host"
    # ... other config

  consul:
    network_mode: "host"
    # ... other config

  nomad:
    network_mode: "host"
    # ... other config
```

## Disaster Recovery

### 1. Backup Strategy
```bash
#!/bin/bash
# backup.sh

# Backup Vault
vault operator raft snapshot save /backups/vault-$(date +%Y%m%d-%H%M%S).snap

# Backup Consul
consul kv export > /backups/consul-$(date +%Y%m%d-%H%M%S).json

# Backup Nomad (via Consul)
consul kv export nomad/ > /backups/nomad-$(date +%Y%m%d-%H%M%S).json
```

### 2. Recovery Procedures
```bash
#!/bin/bash
# restore.sh

# Restore Vault
vault operator raft snapshot restore /backups/vault-20231201-120000.snap

# Restore Consul
consul kv import @/backups/consul-20231201-120000.json

# Restore Nomad
consul kv import @/backups/nomad-20231201-120000.json
```

## Best Practices

### 1. Security
- Use TLS for all communications
- Enable ACLs on all services
- Rotate secrets regularly
- Use least privilege access
- Audit all access

### 2. Monitoring
- Set up comprehensive monitoring
- Configure alerting for critical events
- Monitor resource usage
- Track performance metrics
- Log all activities

### 3. Backup
- Regular automated backups
- Test recovery procedures
- Store backups securely
- Document recovery processes
- Version control configurations

### 4. Documentation
- Document all configurations
- Maintain runbooks
- Keep architecture diagrams updated
- Document troubleshooting procedures
- Maintain change logs

## Next Steps
- [Troubleshooting Guide](8_troubleshooting.md)
- Production deployment guides
- Security hardening guides 