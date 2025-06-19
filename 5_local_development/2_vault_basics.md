# Vault Basics with Docker

## Overview
Learn Vault fundamentals using the local Docker environment.

## Prerequisites
- Complete [Quick Start](1_quick_start.md)

## Getting Started

### 1. Access Vault
```bash
# Set environment variables
export VAULT_ADDR='http://localhost:8200'
export VAULT_TOKEN='root'

# Verify connection
vault status
```

### 2. Store Your First Secret
```bash
# Store a secret
vault kv put secret/myapp username=admin password=secret123

# Retrieve the secret
vault kv get secret/myapp

# List secrets
vault kv list secret/
```

### 3. Enable Secrets Engines
```bash
# Enable database secrets engine
vault secrets enable database

# Enable AWS secrets engine
vault secrets enable aws

# List enabled engines
vault secrets list
```

### 4. Create Policies
Create `my-policy.hcl`:
```hcl
path "secret/data/myapp" {
  capabilities = ["read"]
}

path "secret/data/*" {
  capabilities = ["list"]
}
```

```bash
# Create the policy
vault policy write my-policy my-policy.hcl

# List policies
vault policy list
```

### 5. Enable Authentication Methods
```bash
# Enable AppRole
vault auth enable approle

# Create a role
vault write auth/approle/role/my-role policies="my-policy"

# Get role ID and secret ID
vault read auth/approle/role/my-role/role-id
vault write -f auth/approle/role/my-role/secret-id
```

## Docker Compose for Vault Only
If you want to run just Vault:

```yaml
version: '3.8'
services:
  vault:
    image: vault:latest
    container_name: vault-dev
    ports:
      - "8200:8200"
    environment:
      - VAULT_DEV_ROOT_TOKEN_ID=root
      - VAULT_DEV_LISTEN_ADDRESS=0.0.0.0:8200
    cap_add:
      - IPC_LOCK
    command: vault server -dev
```

## Exercises
1. Store application configuration in Vault
2. Create a policy for read-only access
3. Enable and configure the database secrets engine
4. Use AppRole for application authentication

## Next Steps
- [Consul Service Discovery](3_consul_service_discovery.md)
- [Nomad Jobs](4_nomad_jobs.md)
- [Full Integration](5_full_integration.md) 