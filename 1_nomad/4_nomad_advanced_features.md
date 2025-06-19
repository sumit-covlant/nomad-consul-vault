# Nomad Advanced Features - Complete Guide

## Table of Contents
1. [Autoscaling](#autoscaling)
2. [Multi-Region and Federation](#multi-region-and-federation)
3. [ACLs and Authentication](#acls-and-authentication)
4. [Advanced Scheduling](#advanced-scheduling)
5. [Service Mesh Integration](#service-mesh-integration)
6. [Vault Integration](#vault-integration)
7. [Consul Integration](#consul-integration)
8. [Advanced Networking](#advanced-networking)
9. [Monitoring and Observability](#monitoring-and-observability)
10. [Performance Tuning](#performance-tuning)

---

## Autoscaling

### What is Autoscaling?
Autoscaling in Nomad allows you to automatically adjust the number of running instances based on metrics, policies, or external triggers. This ensures optimal resource utilization and application performance.

### Autoscaling Components
```
┌─────────────────────────────────────────────────────────────┐
│                    Autoscaling Architecture                 │
├─────────────────────────────────────────────────────────────┤
│ External Autoscaler │ Nomad API │ Scheduler │ Jobs         │
│                     │           │           │              │
│ ┌─┐ ┌─┐ ┌─┐        │ ┌─┐       │ ┌─┐       │ ┌─┐ ┌─┐ ┌─┐  │
│ │M│ │P│ │T│ ──▶ API │ │ │ ──▶   │ │ │ ──▶   │ │J│ │J│ │J│  │
│ └─┘ └─┘ └─┘        │ └─┘       │ └─┘       │ └─┘ └─┘ └─┘  │
│ Metrics│Policy│Trig│           │           │              │
└─────────────────────────────────────────────────────────────┘
```

### Horizontal Pod Autoscaler (HPA) Equivalent
```hcl
job "autoscaled-app" {
  datacenters = ["dc1"]
  type = "service"

  group "web" {
    count = 3

    scaling {
      min = 1
      max = 10
      enabled = true
    }

    task "web" {
      driver = "docker"
      config {
        image = "nginx:latest"
      }

      resources {
        cpu    = 500
        memory = 512
      }

      service {
        name = "web"
        port = "http"
      }
    }
  }
}
```

### External Autoscaler Configuration
```yaml
# autoscaler-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nomad-autoscaler-config
data:
  config.hcl: |
    nomad {
      address = "http://nomad:4646"
    }

    apiserver {
      bind_port = 8080
    }

    scaling "web" {
      enabled = true
      min = 1
      max = 10

      policy "cpu" {
        cooldown = "5m"
        evaluation_interval = "30s"

        check "cpu_utilization" {
          source = "nomad_alloc_cpu_user"
          query = "avg(rate(nomad_alloc_cpu_user{job=\"web\"}[5m]))"
          strategy "target-value" {
            target = 70
          }
        }
      }
    }
```

### Custom Autoscaler Implementation
```python
import requests
import time
import json

class NomadAutoscaler:
    def __init__(self, nomad_addr, job_name):
        self.nomad_addr = nomad_addr
        self.job_name = job_name
        self.headers = {"Content-Type": "application/json"}

    def get_job_status(self):
        response = requests.get(f"{self.nomad_addr}/v1/job/{self.job_name}")
        return response.json()

    def get_metrics(self):
        # Get metrics from Prometheus or other monitoring system
        response = requests.get("http://prometheus:9090/api/v1/query", params={
            "query": f'avg(rate(nomad_alloc_cpu_user{{job="{self.job_name}"}}[5m]))'
        })
        return response.json()

    def scale_job(self, count):
        response = requests.post(
            f"{self.nomad_addr}/v1/job/{self.job_name}/scale",
            headers=self.headers,
            json={"Count": count}
        )
        return response.json()

    def autoscale(self):
        while True:
            try:
                # Get current job status
                job_status = self.get_job_status()
                current_count = job_status["TaskGroups"][0]["Count"]

                # Get metrics
                metrics = self.get_metrics()
                cpu_utilization = float(metrics["data"]["result"][0]["value"][1])

                # Scaling logic
                if cpu_utilization > 80 and current_count < 10:
                    new_count = min(current_count + 1, 10)
                    self.scale_job(new_count)
                    print(f"Scaling up to {new_count}")
                elif cpu_utilization < 30 and current_count > 1:
                    new_count = max(current_count - 1, 1)
                    self.scale_job(new_count)
                    print(f"Scaling down to {new_count}")

                time.sleep(30)
            except Exception as e:
                print(f"Error in autoscaling: {e}")
                time.sleep(60)

# Usage
autoscaler = NomadAutoscaler("http://nomad:4646", "web-app")
autoscaler.autoscale()
```

---

## Multi-Region and Federation

### What is Multi-Region?
Multi-region in Nomad allows you to deploy and manage jobs across multiple datacenters or regions. This provides high availability, disaster recovery, and geographic distribution.

### Multi-Region Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    Multi-Region Setup                      │
├─────────────────────────────────────────────────────────────┤
│ Region: us-west-1          │ Region: us-east-1             │
│ ┌─────────────┐            │ ┌─────────────┐               │
│ │ Nomad Server│ ◄─────────►│ │ Nomad Server│               │
│ └─────────────┘            │ └─────────────┘               │
│ ┌─────────────┐            │ ┌─────────────┐               │
│ │ Nomad Client│            │ │ Nomad Client│               │
│ └─────────────┘            │ └─────────────┘               │
│ ┌─────────────┐            │ ┌─────────────┐               │
│ │ Nomad Client│            │ │ Nomad Client│               │
│ └─────────────┘            │ └─────────────┘               │
└─────────────────────────────────────────────────────────────┘
```

### Multi-Region Job Configuration
```hcl
job "multi-region-app" {
  datacenters = ["us-west-1", "us-east-1"]
  type = "service"

  group "web" {
    count = 3

    spread {
      attribute = "${attr.datacenter}"
      target "us-west-1" {
        percent = 50
      }
      target "us-east-1" {
        percent = 50
      }
    }

    task "web" {
      driver = "docker"
      config {
        image = "nginx:latest"
      }

      service {
        name = "web"
        port = "http"
      }
    }
  }
}
```

### Region-Specific Configuration
```hcl
job "region-specific" {
  datacenters = ["us-west-1", "us-east-1"]
  type = "service"

  group "app" {
    count = 2

    task "app" {
      driver = "docker"
      config {
        image = "myapp:latest"
      }

      template {
        data = <<EOH
region: ${NOMAD_DC}
environment: {{ with secret "app/config" }}{{ .Data.data.environment }}{{ end }}
EOH
        destination = "local/config.yaml"
      }

      service {
        name = "app"
        port = "8080"
      }
    }
  }
}
```

### Federation Configuration
```hcl
# Server configuration for federation
data_dir = "/opt/nomad/data"
bind_addr = "0.0.0.0"

server {
  enabled = true
  bootstrap_expect = 3
}

advertise {
  http = "192.168.1.10"
  rpc  = "192.168.1.10"
  serf = "192.168.1.10"
}

server_join {
  retry_join = [
    "192.168.1.11:4648",
    "192.168.1.12:4648",
    "192.168.2.10:4648"  # Different region
  ]
  retry_max = 3
  retry_interval = "15s"
}
```

### Cross-Region Service Discovery
```hcl
job "cross-region-service" {
  datacenters = ["us-west-1", "us-east-1"]
  type = "service"

  group "api" {
    count = 2

    task "api" {
      driver = "docker"
      config {
        image = "myapp/api:latest"
      }

      service {
        name = "api"
        port = "8080"
        tags = ["v1", "api"]

        check {
          type     = "http"
          path     = "/health"
          interval = "10s"
          timeout  = "2s"
        }
      }
    }
  }
}
```

---

## ACLs and Authentication

### What are ACLs?
Access Control Lists (ACLs) in Nomad provide fine-grained access control to resources. They allow you to define who can perform what actions on which resources.

### ACL Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    ACL Architecture                        │
├─────────────────────────────────────────────────────────────┤
│ Client Request │ ACL Check │ Policy Evaluation │ Response  │
│                │           │                   │           │
│ ┌─┐ ┌─┐ ┌─┐   │ ┌─┐       │ ┌─┐               │ ┌─┐       │
│ │T│ │R│ │A│ ──▶│ │ │ ──▶   │ │ │ ──▶           │ │ │       │
│ └─┘ └─┘ └─┘   │ └─┘       │ └─┘               │ └─┘       │
│ Token│Req│API │           │                   │           │
└─────────────────────────────────────────────────────────────┘
```

### Enabling ACLs
```hcl
# Server configuration with ACLs
acl {
  enabled = true
}
```

### ACL Policies
```hcl
# Read-only policy
namespace "default" {
  policy = "read"
}

# Write access to specific jobs
namespace "default" {
  policy = "write"
  job "web" {
    policy = "write"
  }
}

# Admin access
namespace "default" {
  policy = "write"
}

agent {
  policy = "write"
}

operator {
  policy = "write"
}

quota {
  policy = "write"
}

node {
  policy = "write"
}
```

### Creating ACL Tokens
```bash
# Bootstrap ACL system
nomad acl bootstrap

# Create management token
nomad acl token create -name="management" -type=management

# Create policy
nomad acl policy apply -name="readonly" -description="Read-only access" readonly.policy

# Create token with policy
nomad acl token create -name="readonly-token" -policy=readonly
```

### ACL Token Management
```bash
# List tokens
nomad acl token list

# Get token details
nomad acl token info <token-id>

# Update token
nomad acl token update -id=<token-id> -name="new-name"

# Delete token
nomad acl token delete <token-id>
```

### Using ACLs with API
```bash
# Set token in environment
export NOMAD_TOKEN="your-token"

# Use token in API calls
curl -H "X-Nomad-Token: $NOMAD_TOKEN" http://localhost:4646/v1/jobs

# Use token in CLI
nomad job list
```

### ACL Best Practices
```hcl
# Principle of least privilege
namespace "production" {
  policy = "read"
  job "web" {
    policy = "write"
  }
}

namespace "development" {
  policy = "write"
}

# Separate tokens for different purposes
# - Management token for administration
# - Application token for deployments
# - Read-only token for monitoring
```

---

## Advanced Scheduling

### Advanced Scheduling Features
Nomad provides advanced scheduling capabilities including constraints, affinities, spread, and custom scheduling algorithms.

### Complex Constraints
```hcl
job "constrained-app" {
  datacenters = ["dc1"]
  type = "service"

  group "app" {
    count = 3

    constraint {
      attribute = "${attr.kernel.name}"
      value     = "linux"
    }

    constraint {
      attribute = "${attr.cpu.arch}"
      value     = "amd64"
    }

    constraint {
      attribute = "${attr.driver.docker}"
      value     = "1"
    }

    constraint {
      attribute = "${attr.meta.environment}"
      value     = "production"
    }

    constraint {
      attribute = "${attr.unique.hostname}"
      regexp    = "web-.*"
    }

    task "app" {
      driver = "docker"
      config {
        image = "myapp:latest"
      }
    }
  }
}
```

### Affinity Rules
```hcl
job "affinity-app" {
  datacenters = ["dc1"]
  type = "service"

  group "app" {
    count = 3

    affinity "prefer_ssd" {
      attribute = "${attr.storage.ssd}"
      value     = "true"
      weight    = 50
    }

    affinity "prefer_same_rack" {
      attribute = "${attr.meta.rack}"
      value     = "${attr.meta.rack}"
      weight    = 100
    }

    affinity "avoid_old_hardware" {
      attribute = "${attr.cpu.modelname}"
      value     = "Intel(R) Core(TM) i5-2500"
      weight    = -50
    }

    task "app" {
      driver = "docker"
      config {
        image = "myapp:latest"
      }
    }
  }
}
```

### Spread Scheduling
```hcl
job "spread-app" {
  datacenters = ["dc1"]
  type = "service"

  group "app" {
    count = 6

    spread {
      attribute = "${attr.unique.hostname}"
      target "dc1" {
        percent = 100
      }
    }

    spread {
      attribute = "${attr.meta.rack}"
      target "rack1" {
        percent = 50
      }
      target "rack2" {
        percent = 50
      }
    }

    task "app" {
      driver = "docker"
      config {
        image = "myapp:latest"
      }
    }
  }
}
```

### Custom Scheduler
```hcl
job "custom-scheduler" {
  datacenters = ["dc1"]
  type = "service"

  group "app" {
    count = 3

    task "app" {
      driver = "docker"
      config {
        image = "myapp:latest"
      }

      resources {
        cpu    = 500
        memory = 512
      }
    }
  }
}
```

---

## Service Mesh Integration

### What is Service Mesh?
Service mesh provides a dedicated infrastructure layer for handling service-to-service communication. It provides features like service discovery, load balancing, failure recovery, metrics, and monitoring.

### Consul Connect Integration
```hcl
job "service-mesh" {
  datacenters = ["dc1"]
  type = "service"

  group "app" {
    count = 3

    network {
      port "http" {
        static = 8080
      }
    }

    task "web" {
      driver = "docker"
      config {
        image = "myapp/web:latest"
        ports = ["http"]
      }

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
      driver = "docker"
      config {
        image = "myapp/api:latest"
      }

      service {
        name = "api"
        port = "8080"
        connect {
          sidecar_service {}
        }
      }
    }
  }
}
```

### Service Mesh with Intentions
```hcl
job "secure-services" {
  datacenters = ["dc1"]
  type = "service"

  group "frontend" {
    count = 2

    task "frontend" {
      driver = "docker"
      config {
        image = "myapp/frontend:latest"
      }

      service {
        name = "frontend"
        port = "8080"
        connect {
          sidecar_service {
            proxy {
              upstreams {
                destination_name = "backend"
                local_bind_port  = 9090
              }
            }
          }
        }
      }
    }
  }

  group "backend" {
    count = 2

    task "backend" {
      driver = "docker"
      config {
        image = "myapp/backend:latest"
      }

      service {
        name = "backend"
        port = "8080"
        connect {
          sidecar_service {}
        }
      }
    }
  }
}
```

### Service Mesh Monitoring
```hcl
job "monitored-services" {
  datacenters = ["dc1"]
  type = "service"

  group "app" {
    count = 3

    task "app" {
      driver = "docker"
      config {
        image = "myapp:latest"
      }

      service {
        name = "app"
        port = "8080"
        connect {
          sidecar_service {
            proxy {
              config {
                envoy_prometheus_bind_addr = "0.0.0.0:9102"
              }
            }
          }
        }
      }
    }
  }
}
```

---

## Vault Integration

### What is Vault Integration?
Vault integration allows Nomad to securely manage and inject secrets into jobs. This provides a secure way to handle sensitive information like passwords, API keys, and certificates.

### Vault Configuration
```hcl
# Nomad server configuration with Vault
vault {
  enabled = true
  address = "http://vault:8200"
  create_from_role = "nomad-cluster"
  task_token_ttl = "1h"
  task_token_no_default_policy = true
}
```

### Job with Vault Integration
```hcl
job "vault-integrated" {
  datacenters = ["dc1"]
  type = "service"

  group "app" {
    count = 3

    task "app" {
      driver = "docker"
      config {
        image = "myapp:latest"
      }

      vault {
        policies = ["app-policy"]
        change_mode = "restart"
        change_signal = "SIGTERM"
      }

      template {
        data = <<EOH
DATABASE_URL={{ with secret "database/creds/app" }}{{ .Data.data.connection_string }}{{ end }}
API_KEY={{ with secret "secret/app" }}{{ .Data.data.api_key }}{{ end }}
EOH
        destination = "local/secrets.env"
        env = true
      }
    }
  }
}
```

### Dynamic Secrets
```hcl
job "dynamic-secrets" {
  datacenters = ["dc1"]
  type = "service"

  group "app" {
    count = 2

    task "app" {
      driver = "docker"
      config {
        image = "myapp:latest"
      }

      vault {
        policies = ["database-policy"]
        change_mode = "restart"
      }

      template {
        data = <<EOH
DB_USERNAME={{ with secret "database/creds/app" }}{{ .Data.data.username }}{{ end }}
DB_PASSWORD={{ with secret "database/creds/app" }}{{ .Data.data.password }}{{ end }}
DB_HOST={{ with secret "database/creds/app" }}{{ .Data.data.host }}{{ end }}
EOH
        destination = "local/db.env"
        env = true
      }
    }
  }
}
```

### Vault Policies
```hcl
# Vault policy for Nomad
path "secret/app/*" {
  capabilities = ["read"]
}

path "database/creds/app" {
  capabilities = ["read"]
}

path "auth/token/create" {
  capabilities = ["update"]
}
```

---

## Consul Integration

### What is Consul Integration?
Consul integration provides service discovery, health checking, and key-value storage for Nomad jobs. It enables services to find and communicate with each other.

### Consul Configuration
```hcl
# Nomad server configuration with Consul
consul {
  address = "127.0.0.1:8500"
  server_service_name = "nomad"
  client_service_name = "nomad-client"
  auto_advertise = true
  server_auto_join = true
  client_auto_join = true
}
```

### Job with Consul Integration
```hcl
job "consul-integrated" {
  datacenters = ["dc1"]
  type = "service"

  group "app" {
    count = 3

    network {
      port "http" {
        static = 8080
      }
    }

    task "app" {
      driver = "docker"
      config {
        image = "myapp:latest"
        ports = ["http"]
      }

      service {
        name = "app"
        port = "http"
        tags = ["v1", "api"]

        check {
          type     = "http"
          path     = "/health"
          interval = "10s"
          timeout  = "2s"
        }

        check {
          type     = "tcp"
          interval = "30s"
          timeout  = "5s"
        }
      }
    }
  }
}
```

### Consul KV Integration
```hcl
job "consul-kv" {
  datacenters = ["dc1"]
  type = "service"

  group "app" {
    count = 2

    task "app" {
      driver = "docker"
      config {
        image = "myapp:latest"
      }

      template {
        data = <<EOH
{{ key "app/config" }}
EOH
        destination = "local/config.json"
      }
    }
  }
}
```

---

## Advanced Networking

### What is Advanced Networking?
Advanced networking in Nomad includes features like dynamic ports, service mesh networking, and custom network configurations.

### Dynamic Ports
```hcl
job "dynamic-ports" {
  datacenters = ["dc1"]
  type = "service"

  group "app" {
    count = 3

    network {
      port "http" {
        static = 0  # Dynamic port allocation
      }
      port "metrics" {
        static = 0
      }
    }

    task "app" {
      driver = "docker"
      config {
        image = "myapp:latest"
        ports = ["http", "metrics"]
      }

      service {
        name = "app"
        port = "http"
      }

      service {
        name = "app-metrics"
        port = "metrics"
      }
    }
  }
}
```

### Custom Network Configuration
```hcl
job "custom-network" {
  datacenters = ["dc1"]
  type = "service"

  group "app" {
    count = 2

    network {
      mode = "bridge"
      port "http" {
        static = 8080
        to     = 80
      }
    }

    task "app" {
      driver = "docker"
      config {
        image = "myapp:latest"
        ports = ["http"]
        network_mode = "bridge"
      }

      service {
        name = "app"
        port = "http"
      }
    }
  }
}
```

### Network Policies
```hcl
job "network-policies" {
  datacenters = ["dc1"]
  type = "service"

  group "app" {
    count = 3

    network {
      port "http" {
        static = 8080
      }
    }

    task "app" {
      driver = "docker"
      config {
        image = "myapp:latest"
        ports = ["http"]
      }

      service {
        name = "app"
        port = "http"
        tags = ["internal"]
      }
    }
  }
}
```

---

## Monitoring and Observability

### Built-in Metrics
Nomad provides built-in metrics for monitoring cluster health and performance.

### Metrics Configuration
```hcl
# Nomad server configuration with metrics
telemetry {
  collection_interval = "1s"
  disable_hostname = true
  prometheus_metrics = true
  publish_allocation_metrics = true
  publish_node_metrics = true
}
```

### Prometheus Integration
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'nomad'
    static_configs:
      - targets: ['nomad:4646']
    metrics_path: '/v1/metrics'
    params:
      format: ['prometheus']
```

### Grafana Dashboard
```json
{
  "dashboard": {
    "title": "Nomad Cluster",
    "panels": [
      {
        "title": "Job Status",
        "type": "stat",
        "targets": [
          {
            "expr": "nomad_client_allocs_running",
            "legendFormat": "Running Allocations"
          }
        ]
      },
      {
        "title": "CPU Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(nomad_client_host_cpu_user[5m])",
            "legendFormat": "CPU Usage"
          }
        ]
      }
    ]
  }
}
```

### Logging Configuration
```hcl
# Nomad logging configuration
log_level = "INFO"
log_json = true
log_file = "/var/log/nomad.log"
log_rotate_bytes = 100000000
log_rotate_max_files = 5
```

---

## Performance Tuning

### Server Performance
```hcl
# Server performance tuning
server {
  enabled = true
  bootstrap_expect = 3
  retry_join = ["192.168.1.11:4648", "192.168.1.12:4648"]
  retry_max = 3
  retry_interval = "15s"
  start_join = ["192.168.1.11:4648", "192.168.1.12:4648"]
  rejoin_after_leave = true
  non_voting_server = false
  redundancy_zone = "zone1"
  upgrade_version = "1.6.0"
  enable_debug = false
}
```

### Client Performance
```hcl
# Client performance tuning
client {
  enabled = true
  servers = ["192.168.1.10:4647", "192.168.1.11:4647", "192.168.1.12:4647"]
  node_class = "web"
  node_pool = "default"
  servers_join = ["192.168.1.10:4648", "192.168.1.11:4648", "192.168.1.12:4648"]
  meta {
    "environment" = "production"
    "rack" = "rack1"
  }
  options {
    "driver.raw_exec.enable" = "1"
    "driver.docker.volumes.enabled" = "true"
  }
  chroot_env {
    "/bin" = "/bin"
    "/lib" = "/lib"
    "/lib64" = "/lib64"
    "/usr/bin" = "/usr/bin"
  }
}
```

### Resource Limits
```hcl
# Resource limits configuration
limits {
  http_max_conns_per_client = 100
  rpc_max_conns_per_client = 100
}
```

---

## Best Practices

### Security Best Practices
1. **Enable ACLs**: Use ACLs for access control
2. **Use TLS**: Encrypt all communications
3. **Vault Integration**: Use Vault for secrets management
4. **Network Policies**: Implement network segmentation
5. **Regular Updates**: Keep Nomad updated

### Performance Best Practices
1. **Resource Planning**: Plan resource requirements carefully
2. **Monitoring**: Monitor cluster performance
3. **Scaling**: Use appropriate scaling strategies
4. **Optimization**: Optimize job configurations
5. **Cleanup**: Regularly clean up old allocations

### Operational Best Practices
1. **Documentation**: Document configurations and procedures
2. **Testing**: Test in development first
3. **Backup**: Backup configurations and data
4. **Monitoring**: Set up comprehensive monitoring
5. **Disaster Recovery**: Plan for failure scenarios

---

## Next Steps

After mastering advanced features, you'll be ready to:

1. **Production Setup**: Deploy Nomad in production environments
2. **Integration**: Integrate with other HashiCorp tools
3. **Customization**: Build custom solutions
4. **Optimization**: Optimize for specific use cases

### Practice Projects

1. **Multi-Region Deployment**: Deploy across multiple regions
2. **Service Mesh**: Implement service mesh architecture
3. **Security Hardening**: Implement comprehensive security
4. **Performance Optimization**: Optimize cluster performance

### Resources

- [Nomad Advanced Features](https://www.nomadproject.io/docs/internals/)
- [Nomad Security](https://www.nomadproject.io/docs/security/)
- [Nomad Performance](https://www.nomadproject.io/docs/internals/performance/)
- [Nomad Integration](https://www.nomadproject.io/docs/integrations/) 