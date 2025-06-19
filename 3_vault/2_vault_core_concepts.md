# Vault Core Concepts - Complete Guide

## Table of Contents
1. [Policies](#policies)
2. [Authentication Methods](#authentication-methods)
3. [Secrets Engines](#secrets-engines)
4. [Leasing and Renewal](#leasing-and-renewal)
5. [Audit Devices](#audit-devices)
6. [Best Practices](#best-practices)

---

## Policies

### What are Policies?
Policies define what actions users and applications can perform in Vault. They are written in HCL or JSON and attached to tokens.

### Example Policy (HCL)
```hcl
# Allow read/write to secrets in 'secret/'
path "secret/*" {
  capabilities = ["create", "read", "update", "delete", "list"]
}

# Allow read-only access to 'kv/data/app'
path "kv/data/app" {
  capabilities = ["read"]
}
```

### Managing Policies
```bash
# Write a policy
vault policy write my-policy my-policy.hcl

# List policies
vault policy list

# Read a policy
vault policy read my-policy

# Delete a policy
vault policy delete my-policy
```

---

## Authentication Methods

### Overview
Auth methods allow users, applications, and machines to authenticate to Vault and receive a token with associated policies.

### Common Auth Methods
- **AppRole**: For machines and automation
- **Userpass**: Username/password login
- **GitHub**: GitHub user/org authentication
- **JWT/OIDC**: JSON Web Token and OpenID Connect
- **Kubernetes**: Service account-based auth for K8s workloads
- **LDAP**: Enterprise directory integration

### Enabling and Using Auth Methods
```bash
# Enable AppRole
dvault auth enable approle

# Enable GitHub
dvault auth enable github

# Enable Kubernetes
dvault auth enable kubernetes

# List enabled auth methods
vault auth list
```

### Example: AppRole
```bash
# Enable AppRole
dvault auth enable approle

# Create role
vault write auth/approle/role/my-role policies="my-policy"

# Fetch RoleID and SecretID
vault read auth/approle/role/my-role/role-id
vault write -f auth/approle/role/my-role/secret-id

# Login with AppRole
vault write auth/approle/login role_id="<role-id>" secret_id="<secret-id>"
```

---

## Secrets Engines

### What are Secrets Engines?
Secrets engines are plugins that manage, generate, or store secrets. Each engine is enabled at a path (e.g., `secret/`, `kv/`, `database/`).

### Common Secrets Engines
- **KV (Key/Value)**: Store arbitrary secrets
- **Database**: Dynamic DB credentials
- **AWS/GCP/Azure**: Cloud credentials
- **PKI**: Certificate authority and issuance
- **Transit**: Encryption as a service
- **TOTP**: Time-based one-time passwords

### Enabling and Using Secrets Engines
```bash
# Enable KV engine at 'secret/'
vault secrets enable -path=secret kv

# Enable database engine
vault secrets enable database

# Enable AWS engine
vault secrets enable aws

# List enabled engines
vault secrets list
```

### Example: KV Secrets
```bash
vault kv put secret/myapp username=admin password=secret
vault kv get secret/myapp
```

### Example: Database Secrets
```bash
vault write database/config/mydb \
  plugin_name=mysql-database-plugin \
  connection_url="{{username}}:{{password}}@tcp(127.0.0.1:3306)/" \
  allowed_roles="readonly"

vault write database/roles/readonly \
  db_name=mydb \
  creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}'; GRANT SELECT ON *.* TO '{{name}}'@'%';" \
  default_ttl="1h" \
  max_ttl="24h"

vault read database/creds/readonly
```

---

## Leasing and Renewal

### What is Leasing?
Most secrets in Vault are leased, meaning they have a TTL (time-to-live) and can be revoked or renewed.

### Lease Management
```bash
# List leases
vault list sys/leases/lookup

# Renew a lease
vault lease renew <lease_id>

# Revoke a lease
vault lease revoke <lease_id>
```

---

## Audit Devices

### What are Audit Devices?
Audit devices log all requests and responses to Vault for compliance and troubleshooting.

### Enabling Audit Devices
```bash
# Enable file audit device
vault audit enable file file_path=/var/log/vault_audit.log

# List audit devices
vault audit list

# Disable audit device
vault audit disable file
```

---

## Best Practices
- Use least-privilege policies
- Enable audit logging
- Use strong authentication methods
- Rotate secrets and tokens regularly
- Use dynamic secrets where possible
- Monitor and alert on audit logs

---

## Next Steps
- Learn CLI and API usage
- Set up dynamic secrets
- Explore advanced features and production setup 