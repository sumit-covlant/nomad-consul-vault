# Vault Advanced Features - Complete Guide

## Table of Contents
1. [Auto-Unseal](#auto-unseal)
2. [Audit Devices and Logging](#audit-devices-and-logging)
3. [Performance Replication](#performance-replication)
4. [Disaster Recovery Replication](#disaster-recovery-replication)
5. [Vault Agent and Templates](#vault-agent-and-templates)
6. [Sentinel Policy Integration](#sentinel-policy-integration)
7. [Best Practices](#best-practices)

---

## Auto-Unseal

### What is Auto-Unseal?
Auto-unseal allows Vault to be unsealed automatically using a trusted external key management service (KMS), removing the need for manual unseal key entry.

### Supported KMS Providers
- AWS KMS
- Google Cloud KMS
- Azure Key Vault
- Alibaba Cloud KMS
- HSM (Hardware Security Module)

### Example: AWS KMS Auto-Unseal
```hcl
seal "awskms" {
  region     = "us-east-1"
  kms_key_id = "<kms-key-id>"
  access_key = "<aws-access-key>"
  secret_key = "<aws-secret-key>"
}
```

### Example: GCP KMS Auto-Unseal
```hcl
seal "gcpckms" {
  project     = "my-gcp-project"
  region      = "global"
  key_ring    = "vault-keys"
  crypto_key  = "vault-unseal"
  credentials = "<base64-encoded-service-account-json>"
}
```

---

## Audit Devices and Logging

### What are Audit Devices?
Audit devices log all requests and responses to Vault for compliance and troubleshooting.

### Enabling Audit Devices
```bash
vault audit enable file file_path=/var/log/vault_audit.log
vault audit enable syslog tag="vault"
vault audit list
vault audit disable file
```

### Audit Log Best Practices
- Store logs securely and with restricted access
- Forward logs to SIEM or log management systems
- Monitor for suspicious activity

---

## Performance Replication

### What is Performance Replication?
Performance Replication allows Vault to replicate data from a primary cluster to one or more secondary clusters for scalability and high availability.

### Enabling Performance Replication
```bash
# Enable replication (Enterprise only)
vault write -f sys/replication/performance/primary/enable
vault write sys/replication/performance/secondary/enable \
  token=<primary-replication-token> \
  primary_api_addr="https://primary.vault:8200"
```

### Monitoring Replication
```bash
vault read sys/replication/status
```

---

## Disaster Recovery Replication

### What is DR Replication?
Disaster Recovery (DR) Replication allows Vault to replicate data to a secondary cluster for failover in case of disaster.

### Enabling DR Replication
```bash
# Enable DR replication (Enterprise only)
vault write -f sys/replication/dr/primary/enable
vault write sys/replication/dr/secondary/enable \
  token=<primary-dr-token> \
  primary_api_addr="https://primary.vault:8200"
```

---

## Vault Agent and Templates

### What is Vault Agent?
Vault Agent is a client-side daemon that automates authentication, secret retrieval, and template rendering for applications.

### Example: Agent Auto-Auth
```hcl
# agent.hcl
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
```

### Example: Template Rendering
```hcl
template {
  source      = "/etc/vault/template.ctmpl"
  destination = "/etc/secrets/config.json"
}
```

---

## Sentinel Policy Integration

### What is Sentinel?
Sentinel is a policy-as-code framework for fine-grained, logic-based policy decisions in Vault Enterprise.

### Example: Sentinel Policy
```hcl
import "vault"

main = rule {
  vault.auth.method("approle").allowed
}
```

### Enabling Sentinel Policies
```bash
vault write sys/policies/egp/my-policy \
  policy=@my-policy.sentinel \
  enforcement_level="hard-mandatory"
```

---

## Best Practices
- Use auto-unseal for production clusters
- Enable and monitor audit logging
- Use replication for HA and DR (Enterprise)
- Use Vault Agent for secret injection
- Write and enforce Sentinel policies (Enterprise)
- Regularly test failover and recovery

---

## Next Steps
- Prepare for production deployment
- Explore production setup and security 