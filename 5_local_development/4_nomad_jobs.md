# Nomad Jobs with Docker

## Overview
Learn Nomad job scheduling using the local Docker environment.

## Prerequisites
- Complete [Quick Start](1_quick_start.md)

## Getting Started

### 1. Access Nomad
```bash
# Set environment variables
export NOMAD_ADDR='http://localhost:4646'

# Verify connection
nomad server members
nomad node status
```

### 2. Your First Job
Create `web-job.nomad`:
```hcl
job "web" {
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
          port "http" {
            static = 8080
          }
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
    }
  }
}
```

```bash
# Run the job
nomad job run web-job.nomad

# Check job status
nomad job status web

# List all jobs
nomad job list
```

### 3. Job Types

#### Service Job
```hcl
job "api" {
  type = "service"
  
  group "api" {
    count = 1
    
    task "api" {
      driver = "docker"
      
      config {
        image = "hashicorp/http-echo"
        args  = ["-text", "Hello from API"]
      }
      
      resources {
        cpu    = 100
        memory = 128
        network {
          port "http" {}
        }
      }
    }
  }
}
```

#### Batch Job
```hcl
job "batch-job" {
  type = "batch"
  
  group "batch" {
    count = 1
    
    task "batch" {
      driver = "docker"
      
      config {
        image = "alpine:latest"
        command = "sh"
        args = ["-c", "echo 'Batch job completed' && sleep 5"]
      }
      
      resources {
        cpu    = 50
        memory = 64
      }
    }
  }
}
```

### 4. Job Management
```bash
# Stop a job
nomad job stop web

# Restart a job
nomad job restart web

# Scale a job
nomad job scale web 3

# View job logs
nomad alloc logs <allocation-id>

# View job specification
nomad job inspect web
```

### 5. Service Discovery Integration
Create `web-with-consul.nomad`:
```hcl
job "web-consul" {
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
        name = "web-consul"
        port = "http"
        tags = ["nginx", "frontend"]

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

## Docker Compose for Nomad Only
If you want to run just Nomad:

```yaml
version: '3.8'
services:
  nomad:
    image: nomad:latest
    container_name: nomad-dev
    ports:
      - "4646:4646"
      - "4647:4647"
    environment:
      - NOMAD_ADDR=http://0.0.0.0:4646
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: nomad agent -dev -bind=0.0.0.0 -client
```

## Exercises
1. Deploy a simple web application
2. Create a batch job that runs periodically
3. Scale jobs up and down
4. Use different job types (service, batch, system)
5. Integrate with Consul for service discovery
6. Use resource constraints and placement

## Next Steps
- [Full Integration](5_full_integration.md)
- [Advanced Job Features](6_advanced_nomad.md) 