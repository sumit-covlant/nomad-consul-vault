# Vault CLI and API - Complete Guide

## Table of Contents
1. [Vault CLI Overview](#vault-cli-overview)
2. [Common CLI Commands](#common-cli-commands)
3. [Using the HTTP API](#using-the-http-api)
4. [API Authentication](#api-authentication)
5. [API Examples](#api-examples)
6. [Best Practices](#best-practices)

---

## Vault CLI Overview

The Vault CLI (`vault`) is the primary tool for interacting with Vault from the command line. It supports all administrative and operational tasks.

### CLI Authentication
```bash
export VAULT_ADDR='http://127.0.0.1:8200'
vault login <token>
```

---

## Common CLI Commands

### Initialization and Unseal
```bash
vault operator init
vault operator unseal <unseal-key>
```

### Authentication
```bash
vault login <token>
vault auth list
```

### Secrets Management
```bash
vault kv put secret/myapp username=admin password=secret
vault kv get secret/myapp
vault kv delete secret/myapp
```

### Policy Management
```bash
vault policy write my-policy my-policy.hcl
vault policy read my-policy
vault policy delete my-policy
```

### Enable/Disable Secrets Engines
```bash
vault secrets enable -path=secret kv
vault secrets disable secret/
```

### Lease Management
```bash
vault lease renew <lease_id>
vault lease revoke <lease_id>
```

### Audit Devices
```bash
vault audit enable file file_path=/var/log/vault_audit.log
vault audit list
vault audit disable file
```

---

## Using the HTTP API

Vault exposes a RESTful HTTP API for all operations. The API is useful for automation, integration, and programmatic access.

### API Endpoints
- `/v1/sys/init` — Initialize Vault
- `/v1/sys/seal` — Seal Vault
- `/v1/sys/unseal` — Unseal Vault
- `/v1/auth/<method>/login` — Authenticate
- `/v1/secret/data/<path>` — Read/write secrets (KV v2)
- `/v1/policy/<name>` — Manage policies

### Example: cURL Usage
```bash
# Write a secret
curl --header "X-Vault-Token: <token>" \
     --request POST \
     --data '{"data": {"username": "admin", "password": "secret"}}' \
     http://127.0.0.1:8200/v1/secret/data/myapp

# Read a secret
curl --header "X-Vault-Token: <token>" \
     http://127.0.0.1:8200/v1/secret/data/myapp
```

### Example: Python (hvac)
```python
import hvac

client = hvac.Client(url='http://127.0.0.1:8200', token='<token>')

# Write a secret
client.secrets.kv.v2.create_or_update_secret(
    path='myapp',
    secret={'username': 'admin', 'password': 'secret'},
    mount_point='secret',
)

# Read a secret
read_response = client.secrets.kv.v2.read_secret_version(
    path='myapp',
    mount_point='secret',
)
print(read_response['data']['data'])
```

---

## API Authentication

### Token Authentication
- Pass the token in the `X-Vault-Token` header

### AppRole Authentication
```bash
# Login with AppRole
curl --request POST \
     --data '{"role_id": "<role-id>", "secret_id": "<secret-id>"}' \
     http://127.0.0.1:8200/v1/auth/approle/login
```

### Kubernetes Authentication
```bash
# Login with Kubernetes service account JWT
curl --request POST \
     --data '{"role": "my-role", "jwt": "<jwt-token>"}' \
     http://127.0.0.1:8200/v1/auth/kubernetes/login
```

---

## API Examples

### List Secrets
```bash
curl --header "X-Vault-Token: <token>" \
     http://127.0.0.1:8200/v1/secret/metadata/?list=true
```

### Create Policy
```bash
curl --header "X-Vault-Token: <token>" \
     --request PUT \
     --data @my-policy.hcl \
     http://127.0.0.1:8200/v1/sys/policy/my-policy
```

### Enable Secrets Engine
```bash
curl --header "X-Vault-Token: <token>" \
     --request POST \
     --data '{"type": "kv", "options": {"version": "2"}}' \
     http://127.0.0.1:8200/v1/sys/mounts/secret
```

---

## Best Practices
- Use environment variables for tokens and addresses
- Prefer short-lived tokens and dynamic secrets
- Use least-privilege policies for API tokens
- Enable audit logging for API access
- Use HTTPS in production

---

## Next Steps
- Set up dynamic secrets
- Explore advanced features
- Prepare for production deployment 