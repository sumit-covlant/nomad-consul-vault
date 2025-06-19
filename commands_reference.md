# Nomad, Consul, and Vault Commands Reference

This file contains all the essential commands for Nomad, Consul, and Vault with detailed comments and examples.

## Table of Contents
1. [Nomad Commands](#nomad-commands)
2. [Consul Commands](#consul-commands)
3. [Vault Commands](#vault-commands)
4. [Environment Variables](#environment-variables)
5. [Common Use Cases](#common-use-cases)

---

## Nomad Commands

### Environment Setup
```bash
# Set Nomad server address
export NOMAD_ADDR='http://localhost:4646'

# Set region
export NOMAD_REGION='global'

# Set namespace
export NOMAD_NAMESPACE='default'

# Set ACL token
export NOMAD_TOKEN='your-token-here'

# Set TLS certificates
export NOMAD_CACERT='/path/to/ca.crt'
export NOMAD_CLIENT_CERT='/path/to/client.crt'
export NOMAD_CLIENT_KEY='/path/to/client.key'
```

### Agent Management
```bash
# Start Nomad in development mode
nomad agent -dev

# Start Nomad with config file
nomad agent -config=/etc/nomad.d/

# Get agent information
nomad agent info

# Reload agent configuration
nomad agent reload

# Check agent status
nomad agent info -json
```

### Server Management
```bash
# List server members
nomad server members

# Join server to cluster
nomad server join <address>

# Leave server from cluster
nomad server leave

# Show Raft configuration
nomad operator raft list-peers

# Remove Raft peer
nomad operator raft remove-peer <address>

# Get scheduler configuration
nomad operator scheduler get-config

# Set scheduler configuration
nomad operator scheduler set-config <config-file>
```

### Job Management
```bash
# Plan a job (see what changes would be made)
nomad job plan example.nomad

# Plan with variables
nomad job plan -var="count=5" example.nomad

# Plan with JSON output
nomad job plan -json example.nomad

# Plan with diff
nomad job plan -diff example.nomad

# Run a job
nomad job run example.nomad

# Run with variables
nomad job run -var="count=3" example.nomad

# Run with variable files
nomad job run -var-file="production.tfvars" example.nomad

# Run and detach
nomad job run -detach example.nomad

# List all jobs
nomad job list

# Get job status
nomad job status example

# Get job status in JSON
nomad job status -json example

# Get job status with template
nomad job status -t '{{range .TaskGroups}}{{.Name}}: {{.Queued}}/{{.Running}}/{{.Complete}}{{end}}' example

# Stop a job
nomad job stop example

# Stop with purge (remove from state)
nomad job stop -purge example

# Stop with detach
nomad job stop -detach example

# Restart a job
nomad job restart example

# Restart specific task group
nomad job restart -group=web example

# Restart specific task
nomad job restart -group=web -task=server example

# Inspect job definition
nomad job inspect example

# Inspect in JSON format
nomad job inspect -json example

# Inspect specific version
nomad job inspect -version=2 example

# Show job history
nomad job history example

# Show history in JSON
nomad job history -json example

# Show specific version
nomad job history -version=5 example

# Scale a job
nomad job scale example 5

# Scale specific task group
nomad job scale -group=web example 3

# Scale with detach
nomad job scale -detach example 5

# Validate job specification
nomad job validate example.nomad
```

### Allocation Management
```bash
# List all allocations
nomad alloc list

# List allocations for specific job
nomad alloc list -job=example

# List allocations in JSON
nomad alloc list -json

# List with template
nomad alloc list -t '{{range .}}{{.ID}}: {{.JobID}} - {{.Status}}{{end}}'

# Get allocation status
nomad alloc status <alloc-id>

# Get status in JSON
nomad alloc status -json <alloc-id>

# Get allocation logs
nomad alloc logs <alloc-id>

# Get logs for specific task
nomad alloc logs -task=server <alloc-id>

# Follow logs
nomad alloc logs -f <alloc-id>

# Get logs with timestamps
nomad alloc logs -t <alloc-id>

# Get logs for specific time range
nomad alloc logs -since=1h <alloc-id>

# Get logs with tail
nomad alloc logs -tail=100 <alloc-id>

# Execute command in allocation
nomad alloc exec <alloc-id> /bin/bash

# Execute in specific task
nomad alloc exec -task=server <alloc-id> /bin/bash

# Execute with specific user
nomad alloc exec -user=root <alloc-id> /bin/bash

# Execute with environment
nomad alloc exec -env=DEBUG=1 <alloc-id> /bin/bash

# Execute with working directory
nomad alloc exec -cwd=/app <alloc-id> /bin/bash

# Restart allocation
nomad alloc restart <alloc-id>

# Restart specific task
nomad alloc restart -task=server <alloc-id>

# Send signal to allocation
nomad alloc signal <alloc-id> SIGTERM

# Send signal to specific task
nomad alloc signal -task=server <alloc-id> SIGTERM
```

### Node Management
```bash
# List all nodes
nomad node status

# List nodes in JSON
nomad node status -json

# List nodes with template
nomad node status -t '{{range .}}{{.ID}}: {{.Status}} - {{.Datacenter}}{{end}}'

# Get node status
nomad node status <node-id>

# Get status in JSON
nomad node status -json <node-id>

# Drain a node
nomad node drain <node-id>

# Drain with detach
nomad node drain -detach <node-id>

# Drain with deadline
nomad node drain -deadline=1h <node-id>

# Drain with ignore system jobs
nomad node drain -ignore-system <node-id>

# Drain with no shutdown
nomad node drain -no-shutdown <node-id>

# Set node eligibility
nomad node eligibility -enable <node-id>

# Disable node eligibility
nomad node eligibility -disable <node-id>

# Garbage collect a node
nomad node gc <node-id>
```

### ACL Management
```bash
# Bootstrap ACL system
nomad acl bootstrap

# Create management token
nomad acl token create -name="management" -type=management

# Create policy
nomad acl policy apply -name="readonly" -description="Read-only access" readonly.policy

# Create token with policy
nomad acl token create -name="readonly-token" -policy=readonly

# List tokens
nomad acl token list

# Get token details
nomad acl token info <token-id>

# Update token
nomad acl token update -id=<token-id> -name="new-name"

# Delete token
nomad acl token delete <token-id>

# List policies
nomad acl policy list

# Get policy details
nomad acl policy info <policy-name>

# Delete policy
nomad acl policy delete <policy-name>
```

### System Commands
```bash
# Get system status
nomad system gc

# Get system status with detach
nomad system gc -detach

# Get system status with verbose output
nomad system gc -verbose
```

---

## Consul Commands

### Environment Setup
```bash
# Set Consul HTTP address
export CONSUL_HTTP_ADDR='http://localhost:8500'

# Set Consul token
export CONSUL_HTTP_TOKEN='your-token-here'

# Set Consul datacenter
export CONSUL_DATACENTER='dc1'

# Set Consul namespace
export CONSUL_NAMESPACE='default'
```

### Agent Management
```bash
# Start Consul in development mode
consul agent -dev

# Start Consul with config file
consul agent -config=/etc/consul.d/

# Start Consul server
consul agent -server -bootstrap-expect=3 -ui -client=0.0.0.0

# Start Consul client
consul agent -client=0.0.0.0 -join=192.168.1.10

# Get agent information
consul info

# Get agent members
consul members

# Get agent members in JSON
consul members -json

# Get agent members with template
consul members -format=json | jq '.[] | {name: .Name, status: .Status}'

# Reload agent configuration
consul reload
```

### Service Management
```bash
# Register a service
consul services register web-service.json

# Deregister a service
consul services deregister <service-id>

# List all services
consul catalog services

# List services in JSON
consul catalog services -json

# Get service details
consul catalog nodes -service=web

# Get service details in JSON
consul catalog nodes -service=web -json

# Get service health
consul health service web

# Get passing services only
consul health service -passing web

# Get service health in JSON
consul health service -format=json web
```

### Node Management
```bash
# List all nodes
consul catalog nodes

# List nodes in JSON
consul catalog nodes -json

# Get node details
consul catalog nodes -node=web-server-1

# Get node health
consul health node web-server-1

# List datacenters
consul catalog datacenters

# Get datacenter info
consul catalog datacenters -json
```

### Key-Value Store
```bash
# Put a key-value pair
consul kv put myapp/config/database_url "postgresql://localhost:5432/mydb"

# Put with flags
consul kv put -flags=42 myapp/config/feature_flag "enabled"

# Get a value
consul kv get myapp/config/database_url

# Get with detailed info
consul kv get -detailed myapp/config/database_url

# List keys recursively
consul kv get -recurse myapp/

# Delete a key
consul kv delete myapp/config/database_url

# Delete recursively
consul kv delete -recurse myapp/

# Export KV store
consul kv export > consul-backup.json

# Import KV store
consul kv import < consul-backup.json

# Atomic operations
consul kv put -cas -modify-index=123 myapp/config/version "2.0.0"
```

### Health Checks
```bash
# List all health checks
consul health check

# List health checks in JSON
consul health check -json

# Get health checks for a service
consul health service web

# Get health checks for a node
consul health node web-server-1

# Check specific health check
consul health check web-health

# Get passing health checks only
consul health check -passing

# Get critical health checks only
consul health check -critical
```

### DNS Queries
```bash
# Query service via DNS
dig @localhost -p 8600 web.service.consul

# Query service with tag
dig @localhost -p 8600 production.web.service.consul

# Query specific node
dig @localhost -p 8600 web-server-1.node.consul

# Query service in specific datacenter
dig @localhost -p 8600 web.service.dc1.consul

# Query with SRV records
dig @localhost -p 8600 web.service.consul SRV

# Query with A records
dig @localhost -p 8600 web.service.consul A
```

### ACL Management
```bash
# Bootstrap ACL system
consul acl bootstrap

# Create policy
consul acl policy create -name="web-service-policy" -rules=@web-policy.hcl

# Create token
consul acl token create -description="Web service token" -policy-name="web-service-policy"

# List tokens
consul acl token list

# Read token
consul acl token read <token-id>

# Update token
consul acl token update -id=<token-id> -policy-name="new-policy"

# Delete token
consul acl token delete <token-id>

# List policies
consul acl policy list

# Read policy
consul acl policy read <policy-name>

# Delete policy
consul acl policy delete <policy-name>
```

### Connect Service Mesh
```bash
# List intentions
consul intention list

# Create intention (allow)
consul intention create web api

# Create intention (deny)
consul intention create -deny web admin

# Check intention
consul intention check web api

# Delete intention
consul intention delete web api

# List sidecar proxies
consul connect proxy -list

# Get sidecar configuration
consul connect proxy -sidecar-for web-1 -http-addr=localhost:8500

# Stop sidecar proxy
consul connect proxy -sidecar-for web-1 -http-addr=localhost:8500 -stop
```

### Operator Commands
```bash
# List Raft peers
consul operator raft list-peers

# Remove Raft peer
consul operator raft remove-peer <address>

# Get Raft configuration
consul operator raft configuration

# Get Raft stats
consul operator raft stats

# Get cluster leader
consul operator raft list-peers | grep leader

# Get cluster health
consul operator raft list-peers | grep -c "follower\|leader"
```

### Monitoring and Debugging
```bash
# Monitor Consul logs
consul monitor

# Monitor with log level
consul monitor -log-level=debug

# Get metrics
curl http://localhost:8500/v1/agent/metrics?format=prometheus

# Get status
curl http://localhost:8500/v1/status/leader

# Get health
curl http://localhost:8500/v1/health/service/web

# Get catalog
curl http://localhost:8500/v1/catalog/services
```

---

## Vault Commands

### Environment Setup
```bash
# Set Vault address
export VAULT_ADDR='http://localhost:8200'

# Set Vault token
export VAULT_TOKEN='your-token-here'

# Set Vault namespace
export VAULT_NAMESPACE='admin'

# Set Vault CA certificate
export VAULT_CACERT='/path/to/ca.crt'

# Set Vault client certificate
export VAULT_CLIENT_CERT='/path/to/client.crt'

# Set Vault client key
export VAULT_CLIENT_KEY='/path/to/client.key'

# Skip TLS verification (development only)
export VAULT_SKIP_VERIFY='true'
```

### Server Management
```bash
# Start Vault in development mode
vault server -dev

# Start Vault with config file
vault server -config=vault.hcl

# Check Vault status
vault status

# Check Vault status in JSON
vault status -format=json

# Initialize Vault
vault operator init

# Initialize with specific key shares
vault operator init -key-shares=5 -key-threshold=3

# Unseal Vault
vault operator unseal <unseal-key>

# Seal Vault
vault operator seal

# Get Vault version
vault version

# Get Vault help
vault -help
```

### Authentication
```bash
# Login with token
vault login <token>

# Login with username/password
vault login -method=userpass username=admin

# Login with AppRole
vault write auth/approle/login role_id=<role-id> secret_id=<secret-id>

# Login with GitHub
vault login -method=github token=<github-token>

# List auth methods
vault auth list

# Enable auth method
vault auth enable approle

# Disable auth method
vault auth disable approle

# Get token info
vault token lookup

# Create token
vault token create

# Create token with policy
vault token create -policy=my-policy

# Create token with TTL
vault token create -ttl=1h

# Revoke token
vault token revoke <token>

# Renew token
vault token renew <token>
```

### Secrets Management (KV)
```bash
# Enable KV secrets engine
vault secrets enable -path=secret kv

# Enable KV v2
vault secrets enable -path=secret kv-v2

# List secrets engines
vault secrets list

# Put a secret
vault kv put secret/myapp username=admin password=secret123

# Put with metadata
vault kv put -metadata=description="Database credentials" secret/myapp/db username=admin password=secret123

# Get a secret
vault kv get secret/myapp

# Get specific field
vault kv get -field=password secret/myapp

# Get with metadata
vault kv get -metadata secret/myapp

# List secrets
vault kv list secret/

# Delete a secret
vault kv delete secret/myapp

# Undelete a secret (v2 only)
vault kv undelete -versions=1 secret/myapp

# Destroy a secret (v2 only)
vault kv destroy -versions=1 secret/myapp

# Patch a secret (v2 only)
vault kv patch secret/myapp new_field=new_value

# Rollback to previous version (v2 only)
vault kv rollback -version=1 secret/myapp
```

### Dynamic Secrets (Database)
```bash
# Enable database secrets engine
vault secrets enable database

# Configure database connection
vault write database/config/mydb \
  plugin_name=mysql-database-plugin \
  connection_url="{{username}}:{{password}}@tcp(127.0.0.1:3306)/" \
  allowed_roles="readonly,admin"

# Create database role
vault write database/roles/readonly \
  db_name=mydb \
  creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}'; GRANT SELECT ON *.* TO '{{name}}'@'%';" \
  default_ttl="1h" \
  max_ttl="24h"

# Generate database credentials
vault read database/creds/readonly

# Revoke database credentials
vault lease revoke <lease-id>

# Renew database credentials
vault lease renew <lease-id>

# Rotate database root credentials
vault write database/rotate-root/mydb
```

### Dynamic Secrets (AWS)
```bash
# Enable AWS secrets engine
vault secrets enable aws

# Configure AWS credentials
vault write aws/config/root \
  access_key=<AWS_ACCESS_KEY> \
  secret_key=<AWS_SECRET_KEY> \
  region=us-east-1

# Create AWS role
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

# Generate AWS credentials
vault read aws/creds/my-role

# Revoke AWS credentials
vault lease revoke <lease-id>
```

### PKI and Certificates
```bash
# Enable PKI secrets engine
vault secrets enable pki

# Tune PKI max lease TTL
vault secrets tune -max-lease-ttl=87600h pki

# Generate root certificate
vault write pki/root/generate/internal \
  common_name="example.com" \
  ttl=87600h

# Configure URLs
vault write pki/config/urls \
  issuing_certificates="http://127.0.0.1:8200/v1/pki/ca" \
  crl_distribution_points="http://127.0.0.1:8200/v1/pki/crl"

# Create PKI role
vault write pki/roles/my-role \
  allowed_domains="example.com" \
  allow_subdomains=true \
  max_ttl="72h"

# Issue certificate
vault write pki/issue/my-role common_name="test.example.com"

# Revoke certificate
vault write pki/revoke serial_number=<serial>

# Get CA certificate
vault read pki/cert/ca

# Get CRL
vault read pki/crl
```

### Transit (Encryption as a Service)
```bash
# Enable transit secrets engine
vault secrets enable transit

# Create encryption key
vault write -f transit/keys/myapp-key

# Create key with specific algorithm
vault write transit/keys/myapp-key type=aes256-gcm96

# Encrypt data
vault write transit/encrypt/myapp-key plaintext=$(base64 <<< "mysecret")

# Decrypt data
vault write transit/decrypt/myapp-key ciphertext=vault:v1:...

# Rotate key
vault write -f transit/keys/myapp-key/rotate

# Get key info
vault read transit/keys/myapp-key

# Backup key
vault write transit/backup/myapp-key

# Restore key
vault write transit/restore backup=<backup-data>
```

### Policy Management
```bash
# Write policy
vault policy write my-policy my-policy.hcl

# List policies
vault policy list

# Read policy
vault policy read my-policy

# Delete policy
vault policy delete my-policy

# Format policy
vault policy fmt my-policy.hcl
```

### Audit Devices
```bash
# Enable file audit device
vault audit enable file file_path=/var/log/vault_audit.log

# Enable syslog audit device
vault audit enable syslog tag="vault"

# Enable socket audit device
vault audit enable socket address=127.0.0.1:5000

# List audit devices
vault audit list

# Disable audit device
vault audit disable file
```

### Lease Management
```bash
# List leases
vault list sys/leases/lookup

# Lookup lease
vault write sys/leases/lookup lease_id=<lease-id>

# Renew lease
vault lease renew <lease-id>

# Revoke lease
vault lease revoke <lease-id>

# Revoke lease prefix
vault lease revoke -prefix aws/creds/my-role
```

### Monitoring and Debugging
```bash
# Get metrics
curl http://localhost:8200/v1/sys/metrics?format=prometheus

# Get health
curl http://localhost:8200/v1/sys/health

# Get leader
curl http://localhost:8200/v1/sys/leader

# Get HA status
curl http://localhost:8200/v1/sys/ha-status

# Get replication status
curl http://localhost:8200/v1/sys/replication/status

# Debug Vault
vault debug

# Debug with specific duration
vault debug -duration=30s

# Debug with specific interval
vault debug -interval=5s
```

### Backup and Recovery
```bash
# Backup Raft snapshot
vault operator raft snapshot save backup.snap

# Restore from Raft snapshot
vault operator raft snapshot restore backup.snap

# Backup with metadata
vault operator raft snapshot save -metadata=description="Daily backup" backup.snap

# List snapshots
vault operator raft snapshot list
```

---

## Environment Variables

### Nomad Environment Variables
```bash
# Core settings
export NOMAD_ADDR='http://localhost:4646'
export NOMAD_REGION='global'
export NOMAD_NAMESPACE='default'
export NOMAD_TOKEN='your-token-here'

# TLS settings
export NOMAD_CACERT='/path/to/ca.crt'
export NOMAD_CAPATH='/path/to/ca/directory'
export NOMAD_CLIENT_CERT='/path/to/client.crt'
export NOMAD_CLIENT_KEY='/path/to/client.key'
export NOMAD_SKIP_VERIFY='true'

# Logging
export NOMAD_LOG_LEVEL='INFO'
export NOMAD_LOG_JSON='true'
```

### Consul Environment Variables
```bash
# Core settings
export CONSUL_HTTP_ADDR='http://localhost:8500'
export CONSUL_HTTP_TOKEN='your-token-here'
export CONSUL_DATACENTER='dc1'
export CONSUL_NAMESPACE='default'

# TLS settings
export CONSUL_HTTP_SSL='true'
export CONSUL_HTTP_SSL_VERIFY='true'
export CONSUL_CACERT='/path/to/ca.crt'
export CONSUL_CLIENT_CERT='/path/to/client.crt'
export CONSUL_CLIENT_KEY='/path/to/client.key'

# Logging
export CONSUL_LOG_LEVEL='INFO'
```

### Vault Environment Variables
```bash
# Core settings
export VAULT_ADDR='http://localhost:8200'
export VAULT_TOKEN='your-token-here'
export VAULT_NAMESPACE='admin'

# TLS settings
export VAULT_CACERT='/path/to/ca.crt'
export VAULT_CAPATH='/path/to/ca/directory'
export VAULT_CLIENT_CERT='/path/to/client.crt'
export VAULT_CLIENT_KEY='/path/to/client.key'
export VAULT_SKIP_VERIFY='true'

# Logging
export VAULT_LOG_LEVEL='INFO'
export VAULT_LOG_FORMAT='json'
```

---

## Common Use Cases

### 1. Deploy a Web Application with Nomad
```bash
# Create job file
cat > web-app.nomad <<EOF
job "web-app" {
  datacenters = ["dc1"]
  type = "service"

  group "web" {
    count = 3

    task "nginx" {
      driver = "docker"
      config {
        image = "nginx:alpine"
        ports = ["http"]
      }
      resources {
        cpu    = 500
        memory = 512
        network {
          port "http" {}
        }
      }
      service {
        name = "web-app"
        port = "http"
        check {
          type     = "http"
          path     = "/"
          interval = "10s"
          timeout  = "2s"
        }
      }
    }
  }
}
EOF

# Deploy the job
nomad job run web-app.nomad

# Check status
nomad job status web-app

# Scale the job
nomad job scale web-app 5
```

### 2. Register Services with Consul
```bash
# Create service definition
cat > web-service.json <<EOF
{
  "service": {
    "name": "web-app",
    "port": 8080,
    "address": "192.168.1.10",
    "tags": ["v1", "production"],
    "check": {
      "http": "http://localhost:8080/health",
      "interval": "10s",
      "timeout": "5s"
    }
  }
}
EOF

# Register service
consul services register web-service.json

# Check service health
consul health service web-app

# Query via DNS
dig @localhost -p 8600 web-app.service.consul
```

### 3. Store Secrets in Vault
```bash
# Store application secrets
vault kv put secret/web-app/database \
  username=myuser \
  password=mypassword \
  host=localhost \
  port=5432

# Store API keys
vault kv put secret/web-app/external-api \
  api_key=sk-1234567890abcdef \
  endpoint=https://api.example.com

# Retrieve secrets
vault kv get secret/web-app/database
vault kv get -field=password secret/web-app/database
```

### 4. Generate Dynamic Database Credentials
```bash
# Configure database connection
vault write database/config/webapp-db \
  plugin_name=postgresql-database-plugin \
  connection_url="postgresql://{{username}}:{{password}}@localhost:5432/webapp?sslmode=disable" \
  allowed_roles="webapp-readonly,webapp-admin"

# Create readonly role
vault write database/roles/webapp-readonly \
  db_name=webapp-db \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT SELECT ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"

# Generate credentials
vault read database/creds/webapp-readonly
```

### 5. Set up Service Mesh with Consul Connect
```bash
# Create intention to allow web service to access API
consul intention create web-app api-service

# Check intention
consul intention check web-app api-service

# List all intentions
consul intention list
```

### 6. Monitor and Debug
```bash
# Check Nomad cluster health
nomad server members
nomad node status

# Check Consul cluster health
consul members
consul operator raft list-peers

# Check Vault status
vault status
vault operator raft list-peers

# Get metrics from all services
curl http://localhost:4646/v1/metrics?format=prometheus
curl http://localhost:8500/v1/agent/metrics?format=prometheus
curl http://localhost:8200/v1/sys/metrics?format=prometheus
```

### 7. Backup and Recovery
```bash
# Backup Nomad jobs
nomad job list -json | jq -r '.[].ID' | while read job; do
  nomad job inspect $job > backup/jobs_${job}.json
done

# Backup Consul KV store
consul kv export > backup/consul-kv.json

# Backup Vault data
vault operator raft snapshot save backup/vault.snap

# Restore procedures
# Nomad: nomad job run backup/jobs_*.json
# Consul: consul kv import < backup/consul-kv.json
# Vault: vault operator raft snapshot restore backup/vault.snap
```

---

## Troubleshooting Commands

### Nomad Troubleshooting
```bash
# Check server status
nomad server members

# Check node status
nomad node status

# Check job status
nomad job status <job-name>

# Check allocation logs
nomad alloc logs <alloc-id>

# Check agent info
nomad agent info

# Debug job deployment
nomad job plan <job-file>
```

### Consul Troubleshooting
```bash
# Check cluster health
consul members

# Check service health
consul health service <service-name>

# Check Raft status
consul operator raft list-peers

# Check logs
consul monitor

# Debug DNS
dig @localhost -p 8600 <service>.service.consul
```

### Vault Troubleshooting
```bash
# Check Vault status
vault status

# Check authentication
vault auth list

# Check secrets engines
vault secrets list

# Check policies
vault policy list

# Debug Vault
vault debug

# Check audit logs
tail -f /var/log/vault/audit.log
```

---

## Best Practices

### Security
- Always use TLS in production
- Use least-privilege policies
- Rotate secrets and tokens regularly
- Enable audit logging
- Use strong authentication methods

### Performance
- Monitor resource usage
- Use appropriate TTLs for dynamic secrets
- Implement caching where appropriate
- Use connection pooling for databases

### Reliability
- Set up high availability clusters
- Implement proper backup strategies
- Monitor service health
- Use health checks for all services

### Operations
- Document all configurations
- Use version control for job specifications
- Implement proper logging and monitoring
- Test disaster recovery procedures regularly 