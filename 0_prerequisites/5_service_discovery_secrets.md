# Service Discovery and Secrets Management - Complete Guide

## Table of Contents
1. [Introduction to Service Discovery](#introduction-to-service-discovery)
2. [Service Discovery Patterns](#service-discovery-patterns)
3. [Service Discovery Implementations](#service-discovery-implementations)
4. [Introduction to Secrets Management](#introduction-to-secrets-management)
5. [Secrets Management Challenges](#secrets-management-challenges)
6. [Secrets Management Solutions](#secrets-management-solutions)
7. [Integration Patterns](#integration-patterns)
8. [Security Best Practices](#security-best-practices)
9. [Monitoring and Observability](#monitoring-and-observability)
10. [Troubleshooting](#troubleshooting)

---

## Introduction to Service Discovery

### What is Service Discovery?
Service discovery is the automatic detection of services and their network locations in a distributed system. It enables services to find and communicate with each other without hardcoded addresses.

### Why Service Discovery?
```
Without Service Discovery:
┌─────────────────────────────────────────────────────────────┐
│ Service A ──▶ Hardcoded IP: 192.168.1.10:8080              │
│ Service B ──▶ Hardcoded IP: 192.168.1.11:8080              │
│ Service C ──▶ Hardcoded IP: 192.168.1.12:8080              │
│                                                             │
│ Problems:                                                   │
│ - Manual configuration updates                              │
│ - No automatic failover                                     │
│ - Difficult scaling                                         │
│ - Network changes break services                            │
└─────────────────────────────────────────────────────────────┘

With Service Discovery:
┌─────────────────────────────────────────────────────────────┐
│ Service Registry                                            │
│ ├─ Service A: 192.168.1.10:8080 (healthy)                  │
│ ├─ Service B: 192.168.1.11:8080 (healthy)                  │
│ └─ Service C: 192.168.1.12:8080 (unhealthy)                │
│                                                             │
│ Service A ──▶ Query Registry ──▶ Service B                  │
│ Service B ──▶ Query Registry ──▶ Service A                  │
│                                                             │
│ Benefits:                                                   │
│ - Automatic service detection                               │
│ - Health-based routing                                      │
│ - Dynamic scaling                                           │
│ - Fault tolerance                                           │
└─────────────────────────────────────────────────────────────┘
```

### Service Discovery Components
```
┌─────────────────────────────────────────────────────────────┐
│                    Service Discovery                        │
├─────────────────────────────────────────────────────────────┤
│ Service Registry: Central database of service information   │
│ Health Checks: Monitor service availability                 │
│ Load Balancing: Distribute requests across instances        │
│ DNS Integration: Provide DNS-based service resolution       │
│ API Gateway: Route external requests to internal services   │
└─────────────────────────────────────────────────────────────┘
```

---

## Service Discovery Patterns

### Client-Side Discovery
In client-side discovery, the client is responsible for determining the network locations of available service instances and load balancing requests across them.

```
Client ──▶ Service Registry ──▶ Service Instance
   │              │
   └─▶ Load Balancer
```

#### Advantages
- Reduced network hops
- Better performance
- More control over load balancing

#### Disadvantages
- Client complexity
- Language/framework specific
- Harder to implement

#### Example Implementation
```python
import consul
import random

class ServiceDiscovery:
    def __init__(self):
        self.consul = consul.Consul()
    
    def get_service(self, service_name):
        # Query Consul for service instances
        _, services = self.consul.health.service(service_name, passing=True)
        
        if not services:
            raise Exception(f"No healthy instances of {service_name}")
        
        # Simple round-robin load balancing
        service = random.choice(services)
        return f"{service['Service']['Address']}:{service['Service']['Port']}"
    
    def call_service(self, service_name, endpoint):
        service_url = self.get_service(service_name)
        return requests.get(f"http://{service_url}{endpoint}")
```

### Server-Side Discovery
In server-side discovery, the client makes requests to a load balancer that queries the service registry and forwards the request to an available service instance.

```
Client ──▶ Load Balancer ──▶ Service Registry
   │              │
   └─▶ Service Instance
```

#### Advantages
- Client simplicity
- Centralized load balancing
- Language agnostic

#### Disadvantages
- Additional network hop
- Load balancer as single point of failure
- Less control over routing

#### Example Implementation
```nginx
# Nginx configuration with Consul template
upstream backend {
    # Consul template generates this dynamically
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Service Mesh
A service mesh is a dedicated infrastructure layer for handling service-to-service communication.

```
┌─────────────────────────────────────────────────────────────┐
│                    Service Mesh                             │
├─────────────────────────────────────────────────────────────┤
│ Control Plane: Service discovery, policy, telemetry        │
│ Data Plane: Sidecar proxies for each service               │
│                                                             │
│ Service A ──▶ Sidecar A ──▶ Sidecar B ──▶ Service B        │
│     │            │            │            │                │
│     └─▶ Control Plane ◀───────┴────────────┘                │
└─────────────────────────────────────────────────────────────┘
```

#### Example: Istio Service Mesh
```yaml
# Virtual Service
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: web-vs
spec:
  hosts:
  - web.example.com
  http:
  - route:
    - destination:
        host: web-service
        port:
          number: 80
      weight: 100
```

---

## Service Discovery Implementations

### Consul
Consul is a service mesh solution that provides service discovery, configuration, and segmentation functionality.

#### Consul Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    Consul Cluster                          │
├─────────────────────────────────────────────────────────────┤
│ Server 1 (Leader) │ Server 2 │ Server 3                    │
│                   │          │                              │
│ ┌─┐ ┌─┐ ┌─┐      │ ┌─┐ ┌─┐  │ ┌─┐ ┌─┐                     │
│ │A│ │B│ │C│      │ │D│ │E│  │ │F│ │G│                     │
│ └─┘ └─┘ └─┘      │ └─┘ └─┘  │ └─┘ └─┘                     │
└─────────────────────────────────────────────────────────────┘
```

#### Consul Installation
```bash
# Download Consul
wget https://releases.hashicorp.com/consul/1.15.0/consul_1.15.0_linux_amd64.zip
unzip consul_1.15.0_linux_amd64.zip
sudo mv consul /usr/local/bin/

# Start Consul server
consul agent -server -bootstrap-expect=3 -data-dir=/tmp/consul -node=server-1 -bind=192.168.1.10

# Start Consul client
consul agent -data-dir=/tmp/consul -node=client-1 -bind=192.168.1.20
```

#### Service Registration
```json
{
  "service": {
    "name": "web",
    "port": 8080,
    "address": "192.168.1.10",
    "tags": ["v1", "production"],
    "check": {
      "http": "http://192.168.1.10:8080/health",
      "interval": "10s",
      "timeout": "5s"
    }
  }
}
```

```bash
# Register service
consul services register web.json

# List services
consul catalog services

# Get service details
consul catalog nodes -service=web

# Health check
curl http://localhost:8500/v1/health/service/web
```

#### Consul DNS
```bash
# Query service via DNS
dig @127.0.0.1 -p 8600 web.service.consul

# Query specific datacenter
dig @127.0.0.1 -p 8600 web.service.dc1.consul

# Query with tags
dig @127.0.0.1 -p 8600 v1.web.service.consul
```

### etcd
etcd is a distributed key-value store that provides a reliable way to store data across a cluster of machines.

#### etcd Installation
```bash
# Download etcd
wget https://github.com/etcd-io/etcd/releases/download/v3.5.0/etcd-v3.5.0-linux-amd64.tar.gz
tar xzf etcd-v3.5.0-linux-amd64.tar.gz
sudo mv etcd-v3.5.0-linux-amd64/etcd* /usr/local/bin/

# Start etcd
etcd --listen-client-urls http://0.0.0.0:2379 --advertise-client-urls http://192.168.1.10:2379
```

#### Service Registration with etcd
```bash
# Store service information
etcdctl put /services/web/instance1 '{"host":"192.168.1.10","port":8080,"version":"v1"}'
etcdctl put /services/web/instance2 '{"host":"192.168.1.11","port":8080,"version":"v1"}'

# List services
etcdctl get /services/web/ --prefix

# Watch for changes
etcdctl watch /services/web/ --prefix
```

### Kubernetes Service Discovery
Kubernetes provides built-in service discovery through Services and DNS.

#### Service Definition
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
  labels:
    app: web
spec:
  selector:
    app: web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP
```

#### DNS Resolution
```bash
# Service DNS format
# <service-name>.<namespace>.svc.<cluster-domain>

# Example
nslookup web-service.default.svc.cluster.local

# Environment variables
echo $WEB_SERVICE_SERVICE_HOST
echo $WEB_SERVICE_SERVICE_PORT
```

---

## Introduction to Secrets Management

### What is Secrets Management?
Secrets management is the practice of securely storing, accessing, and managing sensitive information such as passwords, API keys, certificates, and tokens.

### Why Secrets Management?
```
Without Secrets Management:
┌─────────────────────────────────────────────────────────────┐
│ Problems:                                                   │
│ - Hardcoded secrets in code                                │
│ - Secrets in configuration files                           │
│ - No rotation of secrets                                   │
│ - Difficult audit trail                                    │
│ - Security vulnerabilities                                 │
│ - Compliance violations                                    │
└─────────────────────────────────────────────────────────────┘

With Secrets Management:
┌─────────────────────────────────────────────────────────────┐
│ Benefits:                                                   │
│ - Centralized secret storage                               │
│ - Automatic secret rotation                                │
│ - Access control and audit                                 │
│ - Encryption at rest and in transit                        │
│ - Integration with identity providers                      │
│ - Compliance and governance                                │
└─────────────────────────────────────────────────────────────┘
```

### Types of Secrets
```
┌─────────────────────────────────────────────────────────────┐
│                    Secret Types                             │
├─────────────────────────────────────────────────────────────┤
│ Passwords: User and service account passwords              │
│ API Keys: External service API keys                        │
│ Certificates: SSL/TLS certificates and keys                │
│ Tokens: JWT, OAuth, and session tokens                     │
│ Database Credentials: Connection strings and passwords     │
│ Encryption Keys: Data encryption keys                      │
│ SSH Keys: SSH private keys                                 │
│ OAuth Secrets: OAuth client secrets                        │
└─────────────────────────────────────────────────────────────┘
```

---

## Secrets Management Challenges

### Security Challenges
```
┌─────────────────────────────────────────────────────────────┐
│                    Security Challenges                      │
├─────────────────────────────────────────────────────────────┤
│ Storage Security: Encrypting secrets at rest               │
│ Transmission Security: Securing secrets in transit         │
│ Access Control: Managing who can access secrets            │
│ Audit Trail: Tracking secret access and changes            │
│ Key Rotation: Automatically rotating secrets               │
│ Zero Trust: Verifying identity before access               │
└─────────────────────────────────────────────────────────────┘
```

### Operational Challenges
```
┌─────────────────────────────────────────────────────────────┐
│                  Operational Challenges                     │
├─────────────────────────────────────────────────────────────┤
│ Integration: Integrating with existing systems             │
│ Performance: Minimizing latency for secret access          │
│ Availability: Ensuring high availability                   │
│ Backup and Recovery: Securely backing up secrets           │
│ Compliance: Meeting regulatory requirements                 │
│ Cost: Managing costs of secret management                  │
└─────────────────────────────────────────────────────────────┘
```

### Common Anti-patterns
```bash
# ❌ Hardcoded secrets in code
DATABASE_PASSWORD = "mypassword123"

# ❌ Secrets in environment variables
export DB_PASSWORD="mypassword123"

# ❌ Secrets in configuration files
{
  "database": {
    "password": "mypassword123"
  }
}

# ❌ Secrets in version control
# .env file committed to git

# ❌ Plain text secrets in logs
logger.info(f"Connecting to database with password: {password}")
```

---

## Secrets Management Solutions

### HashiCorp Vault
Vault is a secrets management and encryption platform that provides secure storage and access to secrets.

#### Vault Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    Vault Cluster                           │
├─────────────────────────────────────────────────────────────┤
│ Vault Server 1 │ Vault Server 2 │ Vault Server 3           │
│ (Active)       │ (Standby)      │ (Standby)                │
│                │                │                           │
│ ┌─┐ ┌─┐ ┌─┐   │ ┌─┐ ┌─┐ ┌─┐   │ ┌─┐ ┌─┐ ┌─┐              │
│ │A│ │B│ │C│   │ │D│ │E│ │F│   │ │G│ │H│ │I│              │
│ └─┘ └─┘ └─┘   │ └─┘ └─┘ └─┘   │ └─┘ └─┘ └─┘              │
└─────────────────────────────────────────────────────────────┘
```

#### Vault Installation
```bash
# Download Vault
wget https://releases.hashicorp.com/vault/1.13.0/vault_1.13.0_linux_amd64.zip
unzip vault_1.13.0_linux_amd64.zip
sudo mv vault /usr/local/bin/

# Start Vault server
vault server -config=vault.hcl
```

#### Vault Configuration
```hcl
# vault.hcl
storage "file" {
  path = "/opt/vault/data"
}

listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = 1
}

api_addr = "http://0.0.0.0:8200"
cluster_addr = "https://0.0.0.0:8201"

ui = true
```

#### Vault Initialization
```bash
# Initialize Vault
vault operator init

# Output:
# Unseal Key 1: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
# Unseal Key 2: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
# Unseal Key 3: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
# Unseal Key 4: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
# Unseal Key 5: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
# Initial Root Token: hvs.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# Unseal Vault (need 3 keys)
vault operator unseal
vault operator unseal
vault operator unseal

# Login with root token
vault login hvs.xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

#### Secrets Engines
```bash
# Enable key-value secrets engine
vault secrets enable -path=secret kv

# Enable database secrets engine
vault secrets enable database

# Enable AWS secrets engine
vault secrets enable aws

# Enable PKI secrets engine
vault secrets enable pki
```

#### Storing and Retrieving Secrets
```bash
# Store secret
vault kv put secret/myapp/database \
  username=myuser \
  password=mypassword

# Retrieve secret
vault kv get secret/myapp/database

# Retrieve specific field
vault kv get -field=password secret/myapp/database
```

#### Dynamic Secrets
```bash
# Configure database connection
vault write database/config/my-mysql-database \
  plugin_name=mysql-database-plugin \
  connection_url="{{username}}:{{password}}@tcp(127.0.0.1:3306)/" \
  allowed_roles="my-role"

# Create database role
vault write database/roles/my-role \
  db_name=my-mysql-database \
  creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}';GRANT SELECT ON *.* TO '{{name}}'@'%';" \
  default_ttl="1h" \
  max_ttl="24h"

# Generate dynamic credentials
vault read database/creds/my-role
```

### AWS Secrets Manager
AWS Secrets Manager is a secrets management service that helps protect access to applications, services, and IT resources.

#### AWS CLI Examples
```bash
# Create secret
aws secretsmanager create-secret \
  --name "myapp/database" \
  --description "Database credentials for myapp" \
  --secret-string '{"username":"myuser","password":"mypassword"}'

# Retrieve secret
aws secretsmanager get-secret-value --secret-id "myapp/database"

# Update secret
aws secretsmanager update-secret \
  --secret-id "myapp/database" \
  --secret-string '{"username":"myuser","password":"newpassword"}'

# Delete secret
aws secretsmanager delete-secret --secret-id "myapp/database"
```

### Azure Key Vault
Azure Key Vault is a cloud service for securely storing and accessing secrets, keys, and certificates.

#### Azure CLI Examples
```bash
# Create key vault
az keyvault create --name mykeyvault --resource-group mygroup --location eastus

# Store secret
az keyvault secret set --vault-name mykeyvault --name "database-password" --value "mypassword"

# Retrieve secret
az keyvault secret show --vault-name mykeyvault --name "database-password"

# Update secret
az keyvault secret set --vault-name mykeyvault --name "database-password" --value "newpassword"
```

### Google Cloud Secret Manager
Google Cloud Secret Manager is a secure and convenient storage system for API keys, passwords, certificates, and other sensitive data.

#### gcloud CLI Examples
```bash
# Create secret
echo -n "mypassword" | gcloud secrets create database-password --data-file=-

# Access secret
gcloud secrets versions access latest --secret="database-password"

# Update secret
echo -n "newpassword" | gcloud secrets versions add database-password --data-file=-

# Delete secret
gcloud secrets delete database-password
```

---

## Integration Patterns

### Application Integration

#### Environment Variables
```bash
# Retrieve secret and set as environment variable
export DB_PASSWORD=$(vault kv get -field=password secret/myapp/database)
export API_KEY=$(vault kv get -field=api_key secret/myapp/external-api)
```

#### Configuration Files
```python
import hvac
import json

# Initialize Vault client
client = hvac.Client(url='http://localhost:8200')
client.token = 'your-token'

# Retrieve secrets
secrets = client.secrets.kv.v2.read_secret_version(
    path='myapp/config',
    mount_point='secret'
)

# Use secrets in configuration
config = {
    'database': {
        'host': secrets['data']['data']['db_host'],
        'password': secrets['data']['data']['db_password']
    },
    'api': {
        'key': secrets['data']['data']['api_key']
    }
}
```

#### Vault Agent
```hcl
# vault-agent.hcl
pid_file = "/tmp/vault-agent.pid"

auto_auth {
  method "approle" {
    mount_path = "auth/approle"
    config = {
      role_id_file_path = "/etc/vault/role-id"
      secret_id_file_path = "/etc/vault/secret-id"
    }
  }
}

template {
  source      = "/etc/vault/templates/app.conf.tpl"
  destination = "/etc/app/app.conf"
}

vault {
  address = "http://localhost:8200"
}
```

```bash
# Template file: app.conf.tpl
database {
  host = "{{ with secret "secret/myapp/database" }}{{ .Data.data.host }}{{ end }}"
  password = "{{ with secret "secret/myapp/database" }}{{ .Data.data.password }}{{ end }}"
}
```

### Container Integration

#### Kubernetes Integration
```yaml
# Vault injector annotation
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    metadata:
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/agent-inject-secret-database: "secret/myapp/database"
        vault.hashicorp.com/role: "myapp"
    spec:
      containers:
      - name: myapp
        image: myapp:latest
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: vault-agent-injector
              key: database
```

#### Docker Integration
```dockerfile
# Dockerfile with Vault integration
FROM alpine:latest

# Install Vault CLI
RUN apk add --no-cache vault

# Copy application
COPY myapp /usr/local/bin/

# Copy Vault configuration
COPY vault-config.hcl /etc/vault/

# Start Vault agent and application
CMD ["sh", "-c", "vault agent -config=/etc/vault/vault-config.hcl & sleep 5 && myapp"]
```

### CI/CD Integration

#### GitHub Actions
```yaml
# .github/workflows/deploy.yml
name: Deploy with Secrets

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Get secrets from Vault
      run: |
        # Install Vault CLI
        wget https://releases.hashicorp.com/vault/1.13.0/vault_1.13.0_linux_amd64.zip
        unzip vault_1.13.0_linux_amd64.zip
        
        # Login to Vault
        ./vault login -method=token ${{ secrets.VAULT_TOKEN }}
        
        # Get secrets
        DB_PASSWORD=$(./vault kv get -field=password secret/myapp/database)
        echo "DB_PASSWORD=$DB_PASSWORD" >> $GITHUB_ENV
    
    - name: Deploy application
      run: |
        # Use secrets in deployment
        echo "Deploying with database password: $DB_PASSWORD"
```

#### Jenkins Pipeline
```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        VAULT_ADDR = 'http://vault:8200'
    }
    
    stages {
        stage('Get Secrets') {
            steps {
                script {
                    // Login to Vault
                    sh 'vault login -method=token ${VAULT_TOKEN}'
                    
                    // Get secrets
                    def dbPassword = sh(
                        script: 'vault kv get -field=password secret/myapp/database',
                        returnStdout: true
                    ).trim()
                    
                    env.DB_PASSWORD = dbPassword
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo "Deploying with database password: ${env.DB_PASSWORD}"
                // Deployment steps
            }
        }
    }
}
```

---

## Security Best Practices

### Access Control
```hcl
# Vault policy for application
path "secret/myapp/*" {
  capabilities = ["read"]
}

path "auth/token/create" {
  capabilities = ["update"]
}
```

### Authentication Methods
```bash
# Enable AppRole authentication
vault auth enable approle

# Create AppRole
vault write auth/approle/role/myapp \
  token_policies="myapp-policy" \
  token_ttl=1h \
  token_max_ttl=4h

# Get role ID
vault read auth/approle/role/myapp/role-id

# Generate secret ID
vault write -f auth/approle/role/myapp/secret-id
```

### Encryption
```bash
# Enable transit secrets engine
vault secrets enable transit

# Create encryption key
vault write -f transit/keys/myapp-key

# Encrypt data
vault write transit/encrypt/myapp-key plaintext=$(base64 <<< "mysecret")

# Decrypt data
vault write transit/decrypt/myapp-key ciphertext=vault:v1:...
```

### Audit Logging
```hcl
# Enable audit logging
vault audit enable file file_path=/var/log/vault/audit.log
```

### Key Rotation
```bash
# Rotate encryption key
vault write -f transit/keys/myapp-key/rotate

# Rotate database credentials
vault write database/rotate-root/my-mysql-database
```

---

## Monitoring and Observability

### Metrics Collection
```bash
# Enable Vault metrics
vault audit enable file file_path=/var/log/vault/audit.log

# Prometheus metrics endpoint
curl http://localhost:8200/v1/sys/metrics?format=prometheus
```

### Health Checks
```bash
# Vault health check
curl http://localhost:8200/v1/sys/health

# Consul health check
curl http://localhost:8500/v1/status/leader
```

### Logging
```hcl
# Vault logging configuration
log_level = "info"
log_format = "json"
```

### Dashboards
```yaml
# Grafana dashboard for Vault
apiVersion: v1
kind: ConfigMap
metadata:
  name: vault-dashboard
data:
  dashboard.json: |
    {
      "dashboard": {
        "title": "Vault Metrics",
        "panels": [
          {
            "title": "Vault Requests",
            "type": "graph",
            "targets": [
              {
                "expr": "rate(vault_core_handle_request_total[5m])",
                "legendFormat": "{{method}}"
              }
            ]
          }
        ]
      }
    }
```

---

## Troubleshooting

### Common Issues

#### Service Discovery Issues
```bash
# Check Consul health
consul members
consul catalog services

# Check DNS resolution
dig @127.0.0.1 -p 8600 web.service.consul

# Check service registration
consul catalog nodes -service=web
```

#### Vault Issues
```bash
# Check Vault status
vault status

# Check Vault logs
tail -f /var/log/vault/vault.log

# Check authentication
vault auth list

# Check secrets engines
vault secrets list
```

#### Network Issues
```bash
# Check connectivity
telnet vault 8200
telnet consul 8500

# Check firewall rules
iptables -L

# Check DNS resolution
nslookup vault
nslookup consul
```

### Debugging Commands
```bash
# Vault debugging
vault debug

# Consul debugging
consul monitor

# Network debugging
tcpdump -i any port 8200
tcpdump -i any port 8500
```

### Performance Tuning
```hcl
# Vault performance tuning
storage "consul" {
  path = "vault/"
  service = "vault"
  service_tags = "vault"
  service_address = "192.168.1.10"
  check_timeout = "5s"
  disable_registration = "false"
  max_parallel = "128"
}

listener "tcp" {
  address = "0.0.0.0:8200"
  tls_disable = 1
  tls_client_ca_file = "/etc/vault/ca.pem"
  tls_cert_file = "/etc/vault/cert.pem"
  tls_key_file = "/etc/vault/key.pem"
}
```

---

## Best Practices

### Service Discovery Best Practices
1. **Health Checks**: Implement comprehensive health checks
2. **Load Balancing**: Use appropriate load balancing strategies
3. **Caching**: Cache service information to reduce latency
4. **Monitoring**: Monitor service discovery metrics
5. **Documentation**: Document service interfaces and dependencies

### Secrets Management Best Practices
1. **Principle of Least Privilege**: Grant minimal necessary access
2. **Secret Rotation**: Regularly rotate secrets
3. **Encryption**: Encrypt secrets at rest and in transit
4. **Audit Logging**: Log all secret access
5. **Backup**: Securely backup secret management systems

### Integration Best Practices
1. **Zero Trust**: Verify identity before granting access
2. **Secure Communication**: Use TLS for all communications
3. **Error Handling**: Handle failures gracefully
4. **Monitoring**: Monitor integration health
5. **Testing**: Test integration thoroughly

---

## Next Steps

After mastering service discovery and secrets management, you'll be ready to:

1. **Learn Consul**: Deep dive into Consul's service mesh capabilities
2. **Explore Vault**: Master Vault's advanced features
3. **Integrate with Nomad**: Use Consul and Vault with Nomad
4. **Build Production Systems**: Deploy secure, observable applications

### Practice Projects

1. **Service Mesh**: Implement a service mesh with Consul
2. **Secrets Management**: Set up Vault for application secrets
3. **Integration**: Integrate Consul and Vault with applications
4. **Monitoring**: Set up monitoring for service discovery and secrets

### Resources

- [Consul Documentation](https://www.consul.io/docs/)
- [Vault Documentation](https://www.vaultproject.io/docs/)
- [Service Mesh Patterns](https://www.servicemesh.io/)
- [Secrets Management Best Practices](https://www.hashicorp.com/resources/secrets-management-best-practices) 