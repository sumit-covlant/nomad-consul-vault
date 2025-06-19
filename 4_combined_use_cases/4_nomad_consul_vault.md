# Nomad + Consul + Vault: End-to-End Integration Guide

## Table of Contents
1. [Overview](#overview)
2. [Why Integrate All Three?](#why-integrate-all-three)
3. [Reference Architecture](#reference-architecture)
4. [Cluster Bootstrapping](#cluster-bootstrapping)
5. [Secure Service Delivery](#secure-service-delivery)
6. [Dynamic Secrets and Service Mesh](#dynamic-secrets-and-service-mesh)
7. [Observability and Monitoring](#observability-and-monitoring)
8. [Configuration Examples](#configuration-examples)
9. [Best Practices](#best-practices)

---

## Overview

Combining Nomad, Consul, and Vault creates a powerful, secure, and highly available platform for running modern workloads. This stack enables dynamic scheduling, service discovery, secure communication, and secrets management.

---

## Why Integrate All Three?
- **Unified platform**: Orchestrate, discover, and secure workloads
- **Zero-trust security**: mTLS, intentions, and dynamic secrets
- **Observability**: Centralized health, metrics, and audit logs
- **Automation**: GitOps, dynamic config, and self-healing

---

## Reference Architecture
```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│  Nomad      │ <--> │  Consul     │ <--> │  Vault      │
│  Cluster    │      │  Cluster    │      │  Cluster    │
└─────────────┘      └─────────────┘      └─────────────┘
```
- Nomad schedules workloads and registers services in Consul
- Consul provides service discovery, health, and service mesh
- Vault manages secrets, PKI, and mTLS for Consul Connect

---

## Cluster Bootstrapping
- Deploy and initialize Vault (auto-unseal, HA)
- Deploy Consul (HA, Connect enabled, Vault CA)
- Deploy Nomad (HA, Consul and Vault integration)
- Configure ACLs and policies for all components

---

## Secure Service Delivery
- Nomad jobs register services in Consul
- Consul Connect provides mTLS and service mesh
- Vault injects secrets into Nomad jobs
- All access and operations are audited

---

## Dynamic Secrets and Service Mesh
- Use Vault for dynamic DB/cloud credentials
- Use Consul Connect for secure service-to-service traffic
- Automate certificate issuance and rotation
- Use Vault Agent for token and secret management

---

## Observability and Monitoring
- Enable audit logging in all components
- Use Prometheus and Grafana for metrics
- Monitor health endpoints and logs
- Set up alerting for failures and anomalies

---

## Configuration Examples

### Nomad Client
```hcl
consul {
  address = "127.0.0.1:8500"
  auto_advertise = true
  client_auto_join = true
}
vault {
  enabled = true
  address = "http://127.0.0.1:8200"
  token   = "<nomad-vault-token>"
}
```

### Consul Connect with Vault CA
```hcl
connect {
  enabled = true
  ca_provider = "vault"
  ca_config = {
    address = "https://vault.service.consul:8200"
    token   = "<vault-token>"
    root_pki_path = "connect-root"
    intermediate_pki_path = "connect-intermediate"
  }
}
```

### Vault Policies
```hcl
path "secret/data/*" {
  capabilities = ["read"]
}
path "connect-intermediate/issue/*" {
  capabilities = ["update"]
}
```

---

## Best Practices
- Use ACLs and least-privilege policies everywhere
- Enable mTLS and intentions in Consul Connect
- Use Vault Agent for secret injection and renewal
- Monitor and alert on all critical events
- Regularly test failover and recovery

---

## Next Steps
- Automate cluster deployment (Terraform, Ansible)
- Implement GitOps for job and config management
- Scale clusters and test disaster recovery 