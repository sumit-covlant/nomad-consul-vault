# Advanced Features with Docker

## Overview
Explore advanced features of Nomad, Consul, and Vault in the local environment.

## Prerequisites
- Complete [Full Integration](5_full_integration.md)

## Advanced Vault Features

### 1. Dynamic Database Credentials
```bash
# Enable database secrets engine
vault secrets enable database

# Configure MySQL connection
vault write database/config/mysql \
  plugin_name=mysql-database-plugin \
  connection_url="{{username}}:{{password}}@tcp(mysql:3306)/" \
  allowed_roles="readonly"

# Create a role
vault write database/roles/readonly \
  db_name=mysql \
  creation_statements="CREATE USER '{{name}}'@'%' IDENTIFIED BY '{{password}}'; GRANT SELECT ON *.* TO '{{name}}'@'%';" \
  default_ttl="1h" \
  max_ttl="24h"

# Generate credentials
vault read database/creds/readonly
```

### 2. PKI Certificate Management
```bash
# Enable PKI
vault secrets enable pki
vault secrets tune -max-lease-ttl=87600h pki

# Generate root CA
vault write pki/root/generate/internal \
  common_name="example.com" \
  ttl=87600h

# Create a role
vault write pki/roles/example-dot-com \
  allowed_domains="example.com" \
  allow_subdomains=true \
  max_ttl="72h"

# Issue a certificate
vault write pki/issue/example-dot-com common_name="test.example.com"
```

## Advanced Consul Features

### 1. Consul Connect Service Mesh
```hcl
# Job with Consul Connect
job "service-mesh" {
  group "app" {
    task "frontend" {
      driver = "docker"
      config {
        image = "nginx:alpine"
      }
      service {
        name = "frontend"
        port = "80"
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
      }
    }

    task "backend" {
      driver = "docker"
      config {
        image = "hashicorp/http-echo"
        args  = ["-text", "Backend Service"]
      }
      service {
        name = "backend"
        port = "5678"
        connect {
          sidecar_service {}
        }
      }
    }
  }
}
```

### 2. Intentions for Access Control
```bash
# Allow frontend to access backend
consul intention create frontend backend

# Deny access to admin service
consul intention create -deny frontend admin

# List intentions
consul intention list
```

## Advanced Nomad Features

### 1. Multi-Region Jobs
```hcl
job "multi-region" {
  datacenters = ["dc1", "dc2"]
  type = "service"

  group "app" {
    count = 1

    task "app" {
      driver = "docker"
      config {
        image = "nginx:alpine"
      }
    }
  }
}
```

### 2. Advanced Scheduling
```hcl
job "scheduled" {
  type = "batch"
  periodic {
    cron = "*/5 * * * * * *"
  }

  group "batch" {
    count = 1

    task "batch" {
      driver = "docker"
      config {
        image = "alpine:latest"
        command = "sh"
        args = ["-c", "echo 'Scheduled job at $(date)'"]
      }
    }
  }
}
```

### 3. Resource Constraints and Placement
```hcl
job "constrained" {
  group "app" {
    count = 2

    constraint {
      attribute = "${attr.kernel.name}"
      value     = "linux"
    }

    constraint {
      attribute = "${attr.cpu.arch}"
      value     = "amd64"
    }

    task "app" {
      driver = "docker"
      config {
        image = "nginx:alpine"
      }

      resources {
        cpu    = 500
        memory = 512
        network {
          port "http" {}
        }
      }
    }
  }
}
```

## Integration Examples

### 1. Vault Agent with Nomad
```hcl
job "vault-agent" {
  group "app" {
    task "vault-agent" {
      driver = "docker"
      config {
        image = "vault:latest"
        args = [
          "agent",
          "-config=/local/agent.hcl"
        ]
        mount {
          type = "bind"
          target = "/local"
          source = "local"
          readonly = true
        }
      }

      template {
        data = <<EOH
auto_auth {
  method "approle" {
    mount_path = "auth/approle"
    config = {
      role_id_file_path = "/local/role_id"
      secret_id_file_path = "/local/secret_id"
    }
  }
}

template {
  source      = "/local/template.ctmpl"
  destination = "/local/config.json"
}
EOH
        destination = "local/agent.hcl"
      }
    }
  }
}
```

### 2. Service Mesh with Dynamic Secrets
```hcl
job "secure-mesh" {
  group "app" {
    task "web" {
      driver = "docker"
      config {
        image = "nginx:alpine"
      }

      service {
        name = "web"
        port = "80"
        connect {
          sidecar_service {
            proxy {
              upstreams {
                destination_name = "api"
                local_bind_port  = 8080
              }
            }
          }
        }
      }

      template {
        data = <<EOH
{{ with secret "database/creds/readonly" }}
export DB_USER={{ .Data.username }}
export DB_PASS={{ .Data.password }}
{{ end }}
EOH
        destination = "secrets/db.env"
        env         = true
      }
    }

    task "api" {
      driver = "docker"
      config {
        image = "hashicorp/http-echo"
        args  = ["-text", "Secure API"]
      }

      service {
        name = "api"
        port = "5678"
        connect {
          sidecar_service {}
        }
      }
    }
  }
}
```

## Monitoring and Observability

### 1. Enable Metrics
```bash
# Vault metrics
curl http://localhost:8200/v1/sys/metrics?format=prometheus

# Consul metrics
curl http://localhost:8500/v1/agent/metrics?format=prometheus

# Nomad metrics
curl http://localhost:4646/v1/metrics?format=prometheus
```

### 2. Prometheus Configuration
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'vault'
    static_configs:
      - targets: ['localhost:8200']
    metrics_path: /v1/sys/metrics
    params:
      format: ['prometheus']

  - job_name: 'consul'
    static_configs:
      - targets: ['localhost:8500']
    metrics_path: /v1/agent/metrics
    params:
      format: ['prometheus']

  - job_name: 'nomad'
    static_configs:
      - targets: ['localhost:4646']
    metrics_path: /v1/metrics
    params:
      format: ['prometheus']
```

## Exercises
1. Set up dynamic database credentials with Vault
2. Implement Consul Connect service mesh
3. Create multi-region Nomad jobs
4. Use Vault Agent for secret injection
5. Set up monitoring and alerting
6. Implement advanced scheduling and constraints

## Next Steps
- [Production Considerations](7_production_considerations.md)
- [Troubleshooting Guide](8_troubleshooting.md) 