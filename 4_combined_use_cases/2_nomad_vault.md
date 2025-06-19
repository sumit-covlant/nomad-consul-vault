# Nomad + Vault Integration - Complete Guide

## Table of Contents
1. [Overview](#overview)
2. [Why Integrate Nomad and Vault?](#why-integrate-nomad-and-vault)
3. [Architecture](#architecture)
4. [Secrets Injection](#secrets-injection)
5. [Vault Agent and Templates](#vault-agent-and-templates)
6. [Configuration Examples](#configuration-examples)
7. [Best Practices](#best-practices)

---

## Overview

Nomad and Vault integration enables secure, dynamic secrets management for workloads. Vault provides secrets, and Nomad injects them into running jobs.

---

## Why Integrate Nomad and Vault?
- **Dynamic secrets**: Inject short-lived credentials into jobs
- **No hardcoded secrets**: Secrets are never stored on disk
- **Automated rotation**: Secrets can be rotated without redeploying jobs
- **Auditability**: All secret access is logged

---

## Architecture
```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│  Nomad      │ <--> │  Vault      │ <--> │  Applications│
│  Cluster    │      │  Cluster    │      │  & Services  │
└─────────────┘      └─────────────┘      └─────────────┘
```
- Nomad clients are configured to talk to Vault
- Vault provides secrets to Nomad jobs
- Vault Agent can be used for advanced injection and renewal

---

## Secrets Injection

### Nomad Template Stanza
Nomad can render secrets from Vault directly into files using the `template` stanza.

```hcl
job "app" {
  group "app" {
    task "web" {
      driver = "docker"
      config {
        image = "myapp:latest"
      }
      template {
        data = <<EOH
{{ with secret "secret/data/myapp" }}
export DB_USER={{ .Data.data.username }}
export DB_PASS={{ .Data.data.password }}
{{ end }}
EOH
        destination = "secrets/env.sh"
        env         = true
      }
      env {
        VAULT_ADDR = "http://vault.service.consul:8200"
      }
    }
  }
}
```

### Vault Policies for Nomad
```hcl
path "secret/data/myapp" {
  capabilities = ["read"]
}
```

---

## Vault Agent and Templates
- Vault Agent can run as a sidecar to authenticate and renew tokens
- Agent can render secrets to files for applications

### Example Vault Agent Configuration
```hcl
auto_auth {
  method "approle" {
    mount_path = "auth/approle"
    config = {
      role_id_file_path = "/etc/vault/role_id"
      secret_id_file_path = "/etc/vault/secret_id"
    }
  }
  sink "file" {
    config = {
      path = "/tmp/vault-token"
    }
  }
}

template {
  source      = "/etc/vault/template.ctmpl"
  destination = "/etc/secrets/config.json"
}
```

---

## Configuration Examples

### Nomad Client Configuration
```hcl
vault {
  enabled = true
  address = "http://127.0.0.1:8200"
  token   = "<nomad-vault-token>"
}
```

---

## Best Practices
- Use short-lived, least-privilege tokens
- Never hardcode secrets in job files
- Enable audit logging in Vault
- Use Vault Agent for advanced scenarios
- Rotate secrets and tokens regularly

---

## Next Steps
- Integrate Consul for service discovery
- Use dynamic secrets for databases and cloud
- Automate secret injection and renewal 