# Full Integration: Nomad + Consul + Vault with Docker

## Overview
Complete integration of all three tools for a production-like local development environment.

## Prerequisites
- Complete [Quick Start](1_quick_start.md)
- Basic understanding of [Vault](2_vault_basics.md), [Consul](3_consul_service_discovery.md), and [Nomad](4_nomad_jobs.md)

## Enhanced Docker Compose Setup

### 1. Production-like Setup
Create `docker-compose-production.yml`:

```yaml
version: '3.8'

services:
  # Vault Server with Consul Storage
  vault:
    image: vault:latest
    container_name: vault
    ports:
      - "8200:8200"
    environment:
      - VAULT_DEV_ROOT_TOKEN_ID=root
      - VAULT_DEV_LISTEN_ADDRESS=0.0.0.0:8200
    cap_add:
      - IPC_LOCK
    command: vault server -dev
    depends_on:
      - consul
    networks:
      - nomad-consul-vault

  # Consul Server with Connect
  consul:
    image: consul:latest
    container_name: consul
    ports:
      - "8500:8500"
      - "8600:8600/udp"
    environment:
      - CONSUL_BIND_INTERFACE=eth0
    command: >
      consul agent -dev -client=0.0.0.0 -ui
      -connect-enabled=true
      -enable-script-checks=true
    networks:
      - nomad-consul-vault

  # Nomad Server with Consul and Vault Integration
  nomad:
    image: nomad:latest
    container_name: nomad
    ports:
      - "4646:4646"
      - "4647:4647"
    environment:
      - NOMAD_ADDR=http://0.0.0.0:4646
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./nomad-config:/etc/nomad.d
    command: nomad agent -dev -bind=0.0.0.0 -client
    depends_on:
      - consul
      - vault
    networks:
      - nomad-consul-vault

networks:
  nomad-consul-vault:
    driver: bridge
```

### 2. Nomad Configuration
Create `nomad-config/client.hcl`:
```hcl
consul {
  address = "consul:8500"
  auto_advertise = true
  client_auto_join = true
}

vault {
  enabled = true
  address = "http://vault:8200"
  token   = "root"
}
```

## Integration Examples

### 1. Job with Vault Secrets and Consul Service Discovery
Create `full-stack-job.nomad`:

```hcl
job "fullstack" {
  datacenters = ["dc1"]
  type = "service"

  group "app" {
    count = 2

    task "web" {
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
{{ with secret "secret/data/myapp" }}
export DB_USER={{ .Data.data.username }}
export DB_PASS={{ .Data.data.password }}
{{ end }}
EOH
        destination = "secrets/env.sh"
        env         = true
      }
    }

    task "api" {
      driver = "docker"

      config {
        image = "hashicorp/http-echo"
        args  = ["-text", "Hello from API"]
        port_map {
          http = 5678
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
{{ with secret "secret/data/myapp" }}
export API_KEY={{ .Data.data.api_key }}
{{ end }}
EOH
        destination = "secrets/api.env"
        env         = true
      }
    }
  }
}
```

### 2. Setup Vault Secrets
```bash
# Set environment variables
export VAULT_ADDR='http://localhost:8200'
export VAULT_TOKEN='root'

# Store application secrets
vault kv put secret/myapp username=admin password=secret123 api_key=abc123

# Create policy for Nomad
vault policy write nomad-policy -<<EOF
path "secret/data/myapp" {
  capabilities = ["read"]
}
EOF
```

### 3. Deploy the Job
```bash
# Set Nomad address
export NOMAD_ADDR='http://localhost:4646'

# Deploy the job
nomad job run full-stack-job.nomad

# Check status
nomad job status fullstack
```

### 4. Verify Integration
```bash
# Check services in Consul
consul catalog services

# Check service health
consul health service web
consul health service api

# Use DNS discovery
dig @localhost -p 8600 web.service.consul
dig @localhost -p 8600 api.service.consul

# Check Vault secrets
vault kv get secret/myapp
```

## Advanced Integration

### 1. Consul Connect with Vault CA
```hcl
# In Consul configuration
connect {
  enabled = true
  ca_provider = "vault"
  ca_config = {
    address = "http://vault:8200"
    token   = "root"
    root_pki_path = "connect-root"
    intermediate_pki_path = "connect-intermediate"
  }
}
```

### 2. Job with Consul Connect
```hcl
job "secure-app" {
  group "app" {
    task "web" {
      # ... existing config ...

      service {
        name = "web"
        port = "http"
        connect {
          sidecar_service {
            proxy {
              upstreams {
                destination_name = "api"
                local_bind_port  = 9090
              }
            }
          }
        }
      }
    }

    task "api" {
      # ... existing config ...

      service {
        name = "api"
        port = "http"
        connect {
          sidecar_service {}
        }
      }
    }
  }
}
```

## Monitoring and Debugging

### 1. Check All Services
```bash
# Nomad
nomad job list
nomad node status

# Consul
consul members
consul catalog services

# Vault
vault status
vault secrets list
```

### 2. View Logs
```bash
# Docker logs
docker-compose logs vault
docker-compose logs consul
docker-compose logs nomad

# Nomad logs
nomad alloc logs <allocation-id>
```

## Exercises
1. Deploy a multi-service application with secrets injection
2. Set up Consul Connect for secure service communication
3. Use Vault for dynamic database credentials
4. Implement service discovery between applications
5. Monitor and troubleshoot the integrated stack

## Next Steps
- [Advanced Features](6_advanced_features.md)
- [Production Considerations](7_production_considerations.md)
- [Troubleshooting Guide](8_troubleshooting.md) 