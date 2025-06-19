# Vault Production Setup - Complete Guide

## Table of Contents
1. [Production Architecture](#production-architecture)
2. [Storage Backends](#storage-backends)
3. [High Availability (HA) Setup](#high-availability-ha-setup)
4. [TLS and Security](#tls-and-security)
5. [Bootstrap and Initialization](#bootstrap-and-initialization)
6. [Monitoring and Alerting](#monitoring-and-alerting)
7. [Backup and Disaster Recovery](#backup-and-disaster-recovery)
8. [Maintenance and Operations](#maintenance-and-operations)

---

## Production Architecture

### Recommended Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                Production Vault Cluster                    │
├─────────────────────────────────────────────────────────────┤
│                    Load Balancer                           │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │ HAProxy     │ │ NGINX       │ │ AWS ELB     │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
│         │               │               │                 │
│         └───────────────┼───────────────┘                 │
│                         │                                 │
│                    Vault Servers (3-5)                    │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │ Vault 1     │ │ Vault 2     │ │ Vault 3     │           │
│ │ (Leader)    │ │ (Standby)   │ │ (Standby)   │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

### Infrastructure Requirements
- 3-5 Vault servers (odd number for Raft)
- 2+ CPU cores, 4GB+ RAM per node
- SSD storage, 20GB+ per node
- Private network, firewall, VPN
- TLS certificates for all nodes

---

## Storage Backends

### Integrated Storage (Raft)
Recommended for most deployments. Built-in, HA, and easy to manage.

```hcl
storage "raft" {
  path    = "/opt/vault/data"
  node_id = "vault-1"
}
```

### Consul Storage
```hcl
storage "consul" {
  address = "127.0.0.1:8500"
  path    = "vault/"
}
```

### Cloud Storage
- AWS S3
- Google Cloud Storage
- Azure Storage

---

## High Availability (HA) Setup

### Raft HA
- 3+ Vault servers, each with its own data directory
- One active leader, others are standby
- Automatic failover

### Consul HA
- Consul as storage backend
- Multiple Vault servers register with Consul
- Leader election and failover managed by Consul

### Load Balancer
- Use HAProxy, NGINX, or cloud load balancer
- Health checks on `/v1/sys/health`

---

## TLS and Security

### TLS Configuration
```hcl
listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_cert_file = "/etc/vault/tls/server.crt"
  tls_key_file  = "/etc/vault/tls/server.key"
  tls_disable   = 0
}
```

### Certificate Generation
```bash
openssl genrsa -out ca.key 2048
openssl req -new -x509 -key ca.key -out ca.crt -days 365
openssl genrsa -out server.key 2048
openssl req -new -key server.key -out server.csr
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -out server.crt
```

### Security Best Practices
- Use TLS for all communication
- Enable audit logging
- Use least-privilege policies
- Rotate root and unseal keys regularly
- Restrict network access to Vault

---

## Bootstrap and Initialization

### Initialize Vault
```bash
vault operator init -key-shares=5 -key-threshold=3
```

### Unseal Vault
```bash
vault operator unseal <unseal-key-1>
vault operator unseal <unseal-key-2>
vault operator unseal <unseal-key-3>
```

### Store Unseal Keys Securely
- Use a password manager or HSM
- Never store unseal keys in plaintext

---

## Monitoring and Alerting

### Prometheus Metrics
```hcl
telemetry {
  prometheus_retention_time = "24h"
  disable_hostname = true
  statsd_address = "localhost:8125"
}
```

### Grafana Dashboard
- Use official Vault dashboards
- Monitor health, performance, and audit logs

### Alerting
- Alert on leader loss, unseal events, high latency, and errors

---

## Backup and Disaster Recovery

### Backup Strategies
- Regularly backup storage backend (Raft, Consul, etc.)
- Backup configuration and audit logs

### Example: Raft Backup
```bash
vault operator raft snapshot save /backup/vault-snapshot.snap
```

### Restore from Backup
```bash
vault operator raft snapshot restore /backup/vault-snapshot.snap
```

### Disaster Recovery Plan
- Document and test recovery procedures
- Store backups securely and offsite

---

## Maintenance and Operations

### Regular Maintenance Tasks
- Rotate root and unseal keys
- Review and update policies
- Monitor audit logs
- Test failover and recovery
- Update Vault to latest version

### Upgrade Procedure
- Read release notes and upgrade guides
- Backup before upgrade
- Upgrade standby nodes first, then leader
- Verify cluster health after upgrade

---

## Next Steps
- Integrate with Consul and Nomad
- Automate secrets management
- Implement advanced features (auto-unseal, replication)
- Review security and compliance 