# Vault Dynamic Secrets - Complete Guide

## Table of Contents
1. [What are Dynamic Secrets?](#what-are-dynamic-secrets)
2. [Database Dynamic Secrets](#database-dynamic-secrets)
3. [Cloud Dynamic Secrets](#cloud-dynamic-secrets)
4. [PKI and Certificate Issuance](#pki-and-certificate-issuance)
5. [Revocation and Renewal](#revocation-and-renewal)
6. [Best Practices](#best-practices)

---

## What are Dynamic Secrets?

Dynamic secrets are generated on-demand when requested, rather than being static and long-lived. They are unique, time-bound, and can be automatically revoked.

### Benefits
- Short-lived, automatically expiring credentials
- Unique per client/request
- Reduced risk of credential leakage
- Easy revocation and rotation

---

## Database Dynamic Secrets

### Supported Databases
- MySQL/MariaDB
- PostgreSQL
- MSSQL
- MongoDB
- Oracle

### Example: MySQL Dynamic Credentials
```bash
# Enable database secrets engine
vault secrets enable database

# Configure database connection
vault write database/config/mydb \
  plugin_name=mysql-database-plugin \
  connection_url="{{username}}:{{password}}@tcp(127.0.0.1:3306)/" \
  allowed_roles="readonly"

# Create a role for dynamic users
vault write database/roles/readonly \
  db_name=mydb \
  creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}'; GRANT SELECT ON *.* TO '{{name}}'@'%';" \
  default_ttl="1h" \
  max_ttl="24h"

# Generate dynamic credentials
vault read database/creds/readonly
```

### Example: PostgreSQL Dynamic Credentials
```bash
vault write database/config/pgdb \
  plugin_name=postgresql-database-plugin \
  allowed_roles="readonly" \
  connection_url="postgresql://{{username}}:{{password}}@localhost:5432/postgres?sslmode=disable"

vault write database/roles/readonly \
  db_name=pgdb \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"

vault read database/creds/readonly
```

---

## Cloud Dynamic Secrets

### AWS Dynamic Credentials
```bash
# Enable AWS secrets engine
vault secrets enable aws

# Configure AWS credentials
vault write aws/config/root \
  access_key=<AWS_ACCESS_KEY> \
  secret_key=<AWS_SECRET_KEY> \
  region=us-east-1

# Create a role for dynamic users
vault write aws/roles/my-role \
  credential_type=iam_user \
  policy_document=-<<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:Describe*",
      "Resource": "*"
    }
  ]
}
EOF

# Generate dynamic AWS credentials
vault read aws/creds/my-role
```

### GCP Dynamic Credentials
```bash
vault secrets enable gcp
vault write gcp/config credentials=@gcp-creds.json
vault write gcp/roleset/my-role \
  project="my-gcp-project" \
  secret_type="access_token" \
  bindings='{"roles/viewer": ["group:devs@example.com"]}'
vault read gcp/key/my-role
```

---

## PKI and Certificate Issuance

### Enable PKI Engine
```bash
vault secrets enable pki
vault secrets tune -max-lease-ttl=87600h pki

# Generate root certificate
vault write pki/root/generate/internal \
  common_name="example.com" \
  ttl=87600h

# Configure URLs
vault write pki/config/urls \
  issuing_certificates="http://127.0.0.1:8200/v1/pki/ca" \
  crl_distribution_points="http://127.0.0.1:8200/v1/pki/crl"

# Create a role
vault write pki/roles/my-role \
  allowed_domains="example.com" \
  allow_subdomains=true \
  max_ttl="72h"

# Issue a certificate
vault write pki/issue/my-role common_name="test.example.com"
```

---

## Revocation and Renewal

### Revoke a Secret
```bash
vault lease revoke <lease_id>
```

### Renew a Secret
```bash
vault lease renew <lease_id>
```

### List Leases
```bash
vault list sys/leases/lookup
```

---

## Best Practices
- Use dynamic secrets wherever possible
- Set appropriate TTLs for all secrets
- Regularly rotate root credentials
- Monitor and alert on lease expirations
- Revoke credentials immediately on compromise

---

## Next Steps
- Explore advanced features (auto-unseal, audit, replication)
- Prepare for production deployment 