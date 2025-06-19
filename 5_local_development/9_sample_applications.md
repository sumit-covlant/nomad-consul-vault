# Sample Applications

## Overview
Complete sample applications demonstrating various use cases with Nomad, Consul, and Vault.

## Prerequisites
- Complete [Full Integration](5_full_integration.md)

## Sample 1: Simple Web Application

### Application Structure
```
sample-web/
├── docker-compose.yml
├── web-job.nomad
├── vault-secrets.json
└── README.md
```

### Docker Compose Setup
```yaml
version: '3.8'

services:
  vault:
    image: vault:latest
    ports:
      - "8200:8200"
    environment:
      - VAULT_DEV_ROOT_TOKEN_ID=root
    command: vault server -dev
    cap_add:
      - IPC_LOCK

  consul:
    image: consul:latest
    ports:
      - "8500:8500"
      - "8600:8600/udp"
    command: consul agent -dev -client=0.0.0.0 -ui

  nomad:
    image: nomad:latest
    ports:
      - "4646:4646"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: nomad agent -dev -bind=0.0.0.0 -client
    depends_on:
      - consul
      - vault
```

### Nomad Job
```hcl
job "sample-web" {
  datacenters = ["dc1"]
  type = "service"

  group "web" {
    count = 2

    task "nginx" {
      driver = "docker"

      config {
        image = "nginx:alpine"
        port_map {
          http = 80
        }
      }

      resources {
        cpu    = 100
        memory = 128
        network {
          port "http" {}
        }
      }

      service {
        name = "web"
        port = "http"
        tags = ["nginx", "frontend"]

        check {
          type     = "http"
          path     = "/"
          interval = "10s"
          timeout  = "2s"
        }
      }

      template {
        data = <<EOH
server {
    listen 80;
    server_name localhost;
    
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
EOH
        destination = "local/nginx.conf"
      }
    }
  }
}
```

### Setup Instructions
```bash
# 1. Start the infrastructure
docker-compose up -d

# 2. Wait for services to be ready
sleep 30

# 3. Deploy the job
nomad job run web-job.nomad

# 4. Check status
nomad job status sample-web

# 5. Access the application
curl http://localhost:8080
```

## Sample 2: Multi-Service Application

### Application Structure
```
multi-service/
├── docker-compose.yml
├── app-job.nomad
├── vault-setup.sh
└── README.md
```

### Nomad Job with Multiple Services
```hcl
job "multi-service" {
  datacenters = ["dc1"]
  type = "service"

  group "app" {
    count = 1

    task "frontend" {
      driver = "docker"

      config {
        image = "nginx:alpine"
        port_map {
          http = 80
        }
      }

      resources {
        cpu    = 100
        memory = 128
        network {
          port "http" {}
        }
      }

      service {
        name = "frontend"
        port = "http"
        tags = ["web", "frontend"]

        check {
          type     = "http"
          path     = "/"
          interval = "10s"
          timeout  = "2s"
        }
      }
    }

    task "api" {
      driver = "docker"

      config {
        image = "hashicorp/http-echo"
        args  = ["-text", "Hello from API", "-listen", ":8080"]
        port_map {
          http = 8080
        }
      }

      resources {
        cpu    = 100
        memory = 128
        network {
          port "http" {}
        }
      }

      service {
        name = "api"
        port = "http"
        tags = ["api", "backend"]

        check {
          type     = "http"
          path     = "/"
          interval = "10s"
          timeout  = "2s"
        }
      }

      template {
        data = <<EOH
{{ with secret "secret/data/api" }}
export API_KEY={{ .Data.data.api_key }}
export DB_URL={{ .Data.data.db_url }}
{{ end }}
EOH
        destination = "secrets/api.env"
        env         = true
      }
    }

    task "database" {
      driver = "docker"

      config {
        image = "postgres:13-alpine"
        port_map {
          postgres = 5432
        }
        env {
          POSTGRES_DB = "myapp"
          POSTGRES_USER = "postgres"
          POSTGRES_PASSWORD = "password"
        }
      }

      resources {
        cpu    = 200
        memory = 256
        network {
          port "postgres" {}
        }
      }

      service {
        name = "database"
        port = "postgres"
        tags = ["database", "postgres"]

        check {
          type     = "tcp"
          port     = "postgres"
          interval = "10s"
          timeout  = "2s"
        }
      }
    }
  }
}
```

### Vault Setup Script
```bash
#!/bin/bash
# vault-setup.sh

export VAULT_ADDR='http://localhost:8200'
export VAULT_TOKEN='root'

# Store API secrets
vault kv put secret/api api_key="abc123" db_url="postgresql://postgres:password@database:5432/myapp"

# Create policy for API
vault policy write api-policy -<<EOF
path "secret/data/api" {
  capabilities = ["read"]
}
EOF

# Enable AppRole
vault auth enable approle

# Create role for API
vault write auth/approle/role/api policies="api-policy"

echo "Vault setup complete!"
```

### Setup Instructions
```bash
# 1. Start infrastructure
docker-compose up -d

# 2. Setup Vault
chmod +x vault-setup.sh
./vault-setup.sh

# 3. Deploy application
nomad job run app-job.nomad

# 4. Check services
consul catalog services

# 5. Test service discovery
dig @localhost -p 8600 frontend.service.consul
dig @localhost -p 8600 api.service.consul
dig @localhost -p 8600 database.service.consul
```

## Sample 3: Service Mesh with Consul Connect

### Application Structure
```
service-mesh/
├── docker-compose.yml
├── mesh-job.nomad
├── intentions.sh
└── README.md
```

### Nomad Job with Service Mesh
```hcl
job "service-mesh" {
  datacenters = ["dc1"]
  type = "service"

  group "app" {
    count = 1

    task "frontend" {
      driver = "docker"

      config {
        image = "nginx:alpine"
        port_map {
          http = 80
        }
      }

      resources {
        cpu    = 100
        memory = 128
        network {
          port "http" {}
        }
      }

      service {
        name = "frontend"
        port = "http"
        tags = ["web", "frontend"]

        connect {
          sidecar_service {
            proxy {
              upstreams {
                destination_name = "backend"
                local_bind_port  = 8080
              }
            }
          }
        }

        check {
          type     = "http"
          path     = "/"
          interval = "10s"
          timeout  = "2s"
        }
      }
    }

    task "backend" {
      driver = "docker"

      config {
        image = "hashicorp/http-echo"
        args  = ["-text", "Secure Backend Service", "-listen", ":8080"]
        port_map {
          http = 8080
        }
      }

      resources {
        cpu    = 100
        memory = 128
        network {
          port "http" {}
        }
      }

      service {
        name = "backend"
        port = "http"
        tags = ["api", "backend"]

        connect {
          sidecar_service {}
        }

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
```

### Intentions Setup
```bash
#!/bin/bash
# intentions.sh

export CONSUL_HTTP_ADDR='http://localhost:8500'

# Allow frontend to access backend
consul intention create frontend backend

# Deny access to admin service (if it exists)
consul intention create -deny frontend admin

echo "Intentions configured!"
```

### Setup Instructions
```bash
# 1. Start infrastructure with Connect enabled
docker-compose up -d

# 2. Deploy service mesh
nomad job run mesh-job.nomad

# 3. Configure intentions
chmod +x intentions.sh
./intentions.sh

# 4. Test service mesh
# Frontend can access backend via localhost:8080
# Backend is only accessible through the service mesh
```

## Sample 4: Batch Processing with Vault Secrets

### Application Structure
```
batch-processing/
├── docker-compose.yml
├── batch-job.nomad
├── vault-batch-setup.sh
└── README.md
```

### Nomad Batch Job
```hcl
job "batch-processing" {
  type = "batch"

  group "batch" {
    count = 1

    task "processor" {
      driver = "docker"

      config {
        image = "alpine:latest"
        command = "sh"
        args = [
          "-c",
          "echo 'Processing with API key: $API_KEY' && echo 'Database URL: $DB_URL' && sleep 10 && echo 'Batch processing complete!'"
        ]
      }

      resources {
        cpu    = 100
        memory = 128
      }

      template {
        data = <<EOH
{{ with secret "secret/data/batch" }}
export API_KEY={{ .Data.data.api_key }}
export DB_URL={{ .Data.data.db_url }}
{{ end }}
EOH
        destination = "secrets/batch.env"
        env         = true
      }
    }
  }
}
```

### Vault Setup for Batch
```bash
#!/bin/bash
# vault-batch-setup.sh

export VAULT_ADDR='http://localhost:8200'
export VAULT_TOKEN='root'

# Store batch processing secrets
vault kv put secret/batch api_key="batch123" db_url="postgresql://batch:pass@db:5432/batch"

# Create policy for batch processing
vault policy write batch-policy -<<EOF
path "secret/data/batch" {
  capabilities = ["read"]
}
EOF

echo "Batch Vault setup complete!"
```

### Setup Instructions
```bash
# 1. Start infrastructure
docker-compose up -d

# 2. Setup Vault for batch
chmod +x vault-batch-setup.sh
./vault-batch-setup.sh

# 3. Run batch job
nomad job run batch-job.nomad

# 4. Monitor job
nomad job status batch-processing

# 5. Check logs
nomad alloc logs <allocation-id>
```

## Sample 5: Complete Microservices Application

### Application Structure
```
microservices/
├── docker-compose.yml
├── microservices-job.nomad
├── vault-microservices-setup.sh
├── intentions-microservices.sh
└── README.md
```

### Complete Microservices Job
```hcl
job "microservices" {
  datacenters = ["dc1"]
  type = "service"

  group "services" {
    count = 1

    task "gateway" {
      driver = "docker"

      config {
        image = "nginx:alpine"
        port_map {
          http = 80
        }
      }

      resources {
        cpu    = 100
        memory = 128
        network {
          port "http" {}
        }
      }

      service {
        name = "gateway"
        port = "http"
        tags = ["gateway", "frontend"]

        connect {
          sidecar_service {
            proxy {
              upstreams {
                destination_name = "users"
                local_bind_port  = 8081
              }
              upstreams {
                destination_name = "orders"
                local_bind_port  = 8082
              }
            }
          }
        }
      }
    }

    task "users" {
      driver = "docker"

      config {
        image = "hashicorp/http-echo"
        args  = ["-text", "Users Service", "-listen", ":8080"]
        port_map {
          http = 8080
        }
      }

      resources {
        cpu    = 100
        memory = 128
        network {
          port "http" {}
        }
      }

      service {
        name = "users"
        port = "http"
        tags = ["api", "users"]

        connect {
          sidecar_service {}
        }
      }

      template {
        data = <<EOH
{{ with secret "secret/data/users" }}
export DB_URL={{ .Data.data.db_url }}
export API_KEY={{ .Data.data.api_key }}
{{ end }}
EOH
        destination = "secrets/users.env"
        env         = true
      }
    }

    task "orders" {
      driver = "docker"

      config {
        image = "hashicorp/http-echo"
        args  = ["-text", "Orders Service", "-listen", ":8080"]
        port_map {
          http = 8080
        }
      }

      resources {
        cpu    = 100
        memory = 128
        network {
          port "http" {}
        }
      }

      service {
        name = "orders"
        port = "http"
        tags = ["api", "orders"]

        connect {
          sidecar_service {}
        }
      }

      template {
        data = <<EOH
{{ with secret "secret/data/orders" }}
export DB_URL={{ .Data.data.db_url }}
export API_KEY={{ .Data.data.api_key }}
{{ end }}
EOH
        destination = "secrets/orders.env"
        env         = true
      }
    }

    task "database" {
      driver = "docker"

      config {
        image = "postgres:13-alpine"
        port_map {
          postgres = 5432
        }
        env {
          POSTGRES_DB = "microservices"
          POSTGRES_USER = "postgres"
          POSTGRES_PASSWORD = "password"
        }
      }

      resources {
        cpu    = 200
        memory = 256
        network {
          port "postgres" {}
        }
      }

      service {
        name = "database"
        port = "postgres"
        tags = ["database", "postgres"]
      }
    }
  }
}
```

### Setup Instructions
```bash
# 1. Start infrastructure
docker-compose up -d

# 2. Setup Vault secrets
chmod +x vault-microservices-setup.sh
./vault-microservices-setup.sh

# 3. Deploy microservices
nomad job run microservices-job.nomad

# 4. Configure service mesh intentions
chmod +x intentions-microservices.sh
./intentions-microservices.sh

# 5. Test the application
curl http://localhost:8080
```

## Running All Samples

### Quick Start Script
```bash
#!/bin/bash
# run-all-samples.sh

echo "Starting all sample applications..."

# Start infrastructure
docker-compose up -d

# Wait for services
sleep 30

# Run each sample
echo "Running Sample 1: Simple Web Application"
cd sample-web && nomad job run web-job.nomad

echo "Running Sample 2: Multi-Service Application"
cd ../multi-service && nomad job run app-job.nomad

echo "Running Sample 3: Service Mesh"
cd ../service-mesh && nomad job run mesh-job.nomad

echo "Running Sample 4: Batch Processing"
cd ../batch-processing && nomad job run batch-job.nomad

echo "Running Sample 5: Microservices"
cd ../microservices && nomad job run microservices-job.nomad

echo "All samples deployed!"
echo "Check the UIs:"
echo "- Nomad: http://localhost:4646"
echo "- Consul: http://localhost:8500"
echo "- Vault: http://localhost:8200"
```

## Next Steps
- Customize the samples for your use case
- Add monitoring and logging
- Implement CI/CD pipelines
- Scale the applications
- Add security hardening 