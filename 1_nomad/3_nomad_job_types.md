# Nomad Job Types - Complete Guide

## Table of Contents
1. [Job Type Overview](#job-type-overview)
2. [Service Jobs](#service-jobs)
3. [Batch Jobs](#batch-jobs)
4. [System Jobs](#system-jobs)
5. [Parameterized Jobs](#parameterized-jobs)
6. [Periodic Jobs](#periodic-jobs)
7. [Job Type Comparison](#job-type-comparison)
8. [Advanced Job Patterns](#advanced-job-patterns)
9. [Best Practices](#best-practices)

---

## Job Type Overview

### What are Job Types?
Job types in Nomad define how the scheduler should handle the job and what behavior to expect. Each job type has different characteristics and use cases.

### Available Job Types
```
┌─────────────────────────────────────────────────────────────┐
│                    Nomad Job Types                          │
├─────────────────────────────────────────────────────────────┤
│ service:    Long-running services (web servers, APIs)      │
│ batch:      Short-lived tasks (data processing, backups)   │
│ system:     Node-level services (monitoring, logging)      │
│ parameterized: Dynamic jobs with variables                 │
│ periodic:   Time-based scheduled jobs                      │
└─────────────────────────────────────────────────────────────┘
```

### Job Type Characteristics
```
┌─────────────────────────────────────────────────────────────┐
│                Job Type Characteristics                     │
├─────────────────────────────────────────────────────────────┤
│ Type        │ Restart │ Reschedule │ Auto-Recovery │ Scale │
├─────────────┼─────────┼────────────┼───────────────┼───────┤
│ service     │ Yes     │ Yes        │ Yes           │ Yes   │
│ batch       │ No      │ Yes        │ Limited       │ No    │
│ system      │ Yes     │ No         │ Yes           │ Auto  │
│ parameterized│ Depends │ Depends    │ Depends       │ Depends│
│ periodic    │ Depends │ Depends    │ Depends       │ Depends│
└─────────────────────────────────────────────────────────────┘
```

---

## Service Jobs

### What are Service Jobs?
Service jobs are designed for long-running applications that should be continuously available. They are the most common job type and are used for web servers, APIs, databases, and other persistent services.

### Service Job Characteristics
- **Long-running**: Designed to run indefinitely
- **Auto-restart**: Automatically restarts on failure
- **Auto-reschedule**: Moves to healthy nodes if current node fails
- **Scalable**: Can be scaled up or down
- **Health checks**: Supports health checks and service discovery

### Basic Service Job Example
```hcl
job "web-service" {
  datacenters = ["dc1"]
  type = "service"

  group "web" {
    count = 3

    network {
      port "http" {
        static = 8080
      }
    }

    task "server" {
      driver = "docker"

      config {
        image = "nginx:latest"
        ports = ["http"]
      }

      resources {
        cpu    = 500
        memory = 512
      }

      service {
        name = "web"
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
```

### Advanced Service Job Example
```hcl
job "api-service" {
  datacenters = ["dc1"]
  type = "service"

  update {
    stagger      = "10s"
    max_parallel = 2
    health_check = "checks"
    auto_revert  = true
    auto_promote = true
  }

  group "api" {
    count = 5

    spread {
      attribute = "${attr.unique.hostname}"
      target "dc1" {
        percent = 100
      }
    }

    network {
      port "http" {
        static = 8080
      }
      port "metrics" {
        static = 9090
      }
    }

    task "api" {
      driver = "docker"

      config {
        image = "myapp/api:v1.0"
        ports = ["http", "metrics"]
      }

      resources {
        cpu    = 1000
        memory = 1024
      }

      service {
        name = "api"
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
          port     = "metrics"
          interval = "30s"
          timeout  = "5s"
        }
      }

      service {
        name = "api-metrics"
        port = "metrics"
        tags = ["metrics", "prometheus"]
      }
    }
  }
}
```

### Service Job with Rolling Updates
```hcl
job "rolling-update" {
  datacenters = ["dc1"]
  type = "service"

  update {
    stagger      = "30s"
    max_parallel = 1
    health_check = "checks"
    auto_revert  = true
    min_healthy_time = "10s"
    healthy_deadline = "5m"
    progress_deadline = "10m"
  }

  group "app" {
    count = 5

    task "app" {
      driver = "docker"

      config {
        image = "myapp:latest"
      }

      service {
        name = "app"
        port = "http"

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

## Batch Jobs

### What are Batch Jobs?
Batch jobs are designed for short-lived tasks that run to completion and then exit. They are perfect for data processing, backups, maintenance tasks, and other one-time operations.

### Batch Job Characteristics
- **Short-lived**: Run to completion and exit
- **No auto-restart**: Don't restart on failure (by default)
- **Reschedulable**: Can be rescheduled to other nodes
- **Not scalable**: Count is fixed
- **Resource efficient**: Don't consume resources after completion

### Basic Batch Job Example
```hcl
job "data-processing" {
  datacenters = ["dc1"]
  type = "batch"

  group "processors" {
    count = 3

    task "processor" {
      driver = "docker"

      config {
        image = "data-processor:latest"
        args = ["--input", "/data/input", "--output", "/data/output"]
      }

      resources {
        cpu    = 2000
        memory = 4096
      }

      restart {
        attempts = 0
        mode     = "fail"
      }
    }
  }
}
```

### Advanced Batch Job Example
```hcl
job "backup-job" {
  datacenters = ["dc1"]
  type = "batch"

  parameterized {
    payload = "required"
  }

  group "backup" {
    count = 1

    task "backup" {
      driver = "docker"

      config {
        image = "backup-tool:latest"
        args = [
          "--source", "${NOMAD_META_source}",
          "--destination", "${NOMAD_META_destination}",
          "--compression", "${NOMAD_META_compression}"
        ]
      }

      resources {
        cpu    = 1000
        memory = 2048
      }

      restart {
        attempts = 2
        delay    = "15s"
        interval = "1m"
        mode     = "fail"
      }

      template {
        data = <<EOH
#!/bin/bash
echo "Starting backup at $(date)"
echo "Source: ${NOMAD_META_source}"
echo "Destination: ${NOMAD_META_destination}"
EOH
        destination = "local/scripts/backup.sh"
        perms        = "0755"
      }
    }
  }
}
```

### Batch Job with Dependencies
```hcl
job "etl-pipeline" {
  datacenters = ["dc1"]
  type = "batch"

  group "pipeline" {
    count = 1

    task "extract" {
      driver = "docker"

      config {
        image = "etl/extract:latest"
      }

      resources {
        cpu    = 500
        memory = 1024
      }
    }

    task "transform" {
      driver = "docker"

      config {
        image = "etl/transform:latest"
      }

      resources {
        cpu    = 1000
        memory = 2048
      }

      depends_on = ["extract"]
    }

    task "load" {
      driver = "docker"

      config {
        image = "etl/load:latest"
      }

      resources {
        cpu    = 500
        memory = 1024
      }

      depends_on = ["transform"]
    }
  }
}
```

---

## System Jobs

### What are System Jobs?
System jobs are designed to run on every node in the cluster. They are perfect for monitoring agents, log collectors, and other node-level services that need to run on every machine.

### System Job Characteristics
- **Node-level**: Runs on every node
- **Auto-restart**: Automatically restarts on failure
- **No rescheduling**: Stays on the same node
- **Auto-scaling**: Automatically scales with cluster size
- **Node constraints**: Can be constrained to specific nodes

### Basic System Job Example
```hcl
job "monitoring" {
  datacenters = ["dc1"]
  type = "system"

  group "monitoring" {
    task "node-exporter" {
      driver = "docker"

      config {
        image = "prom/node-exporter:latest"
        args = [
          "--path.procfs=/host/proc",
          "--path.sysfs=/host/sys",
          "--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)"
        ]
        volumes = [
          "/proc:/host/proc:ro",
          "/sys:/host/sys:ro"
        ]
      }

      resources {
        cpu    = 100
        memory = 128
      }

      service {
        name = "node-exporter"
        port = "9100"

        check {
          type     = "http"
          path     = "/metrics"
          interval = "30s"
          timeout  = "5s"
        }
      }
    }
  }
}
```

### Advanced System Job Example
```hcl
job "logging" {
  datacenters = ["dc1"]
  type = "system"

  constraint {
    attribute = "${attr.kernel.name}"
    value     = "linux"
  }

  group "logging" {
    task "fluentd" {
      driver = "docker"

      config {
        image = "fluent/fluentd:latest"
        volumes = [
          "/var/log:/var/log:ro",
          "/var/lib/docker/containers:/var/lib/docker/containers:ro"
        ]
      }

      resources {
        cpu    = 200
        memory = 256
      }

      service {
        name = "fluentd"
        port = "24224"

        check {
          type     = "tcp"
          interval = "30s"
          timeout  = "5s"
        }
      }

      template {
        data = <<EOF
<source>
  @type tail
  path /var/log/containers/*.log
  pos_file /var/log/fluentd-containers.log.pos
  tag kubernetes.*
  read_from_head true
  <parse>
    @type json
    time_format %Y-%m-%dT%H:%M:%S.%NZ
  </parse>
</source>

<match kubernetes.**>
  @type elasticsearch
  host elasticsearch
  port 9200
  logstash_format true
  logstash_prefix k8s
</match>
EOF
        destination = "local/fluent.conf"
      }
    }
  }
}
```

### System Job with Node Constraints
```hcl
job "storage-monitor" {
  datacenters = ["dc1"]
  type = "system"

  constraint {
    attribute = "${attr.storage.ssd}"
    value     = "true"
  }

  group "monitoring" {
    task "storage-monitor" {
      driver = "docker"

      config {
        image = "storage-monitor:latest"
      }

      resources {
        cpu    = 50
        memory = 64
      }

      service {
        name = "storage-monitor"
        port = "9090"
      }
    }
  }
}
```

---

## Parameterized Jobs

### What are Parameterized Jobs?
Parameterized jobs are templates that can be instantiated with different parameters. They are useful for creating dynamic jobs based on input data or for running the same job with different configurations.

### Parameterized Job Characteristics
- **Template-based**: Job definition with variables
- **Dynamic instantiation**: Created with specific parameters
- **Reusable**: Same template, different parameters
- **Flexible**: Can accept various parameter types

### Basic Parameterized Job Example
```hcl
job "parameterized-example" {
  datacenters = ["dc1"]
  type = "batch"

  parameterized {
    payload = "required"
    meta_required = ["source", "destination"]
    meta_optional = ["compression"]
  }

  group "batch" {
    count = 1

    task "task" {
      driver = "docker"

      config {
        image = "batch-processor:latest"
        args = [
          "--source", "${NOMAD_META_source}",
          "--destination", "${NOMAD_META_destination}",
          "--compression", "${NOMAD_META_compression}"
        ]
      }

      resources {
        cpu    = 500
        memory = 1024
      }
    }
  }
}
```

### Creating Parameterized Job Instances
```bash
# Create job instance with parameters
nomad job dispatch -meta source=/data/input -meta destination=/data/output parameterized-example

# Create with optional parameters
nomad job dispatch -meta source=/data/input -meta destination=/data/output -meta compression=gzip parameterized-example

# Create with payload
echo '{"key": "value"}' | nomad job dispatch -payload - parameterized-example
```

### Advanced Parameterized Job Example
```hcl
job "data-pipeline" {
  datacenters = ["dc1"]
  type = "batch"

  parameterized {
    payload = "required"
    meta_required = ["pipeline_type", "data_source"]
    meta_optional = ["batch_size", "timeout"]
  }

  group "pipeline" {
    count = 1

    task "processor" {
      driver = "docker"

      config {
        image = "data-pipeline:latest"
        args = [
          "--type", "${NOMAD_META_pipeline_type}",
          "--source", "${NOMAD_META_data_source}",
          "--batch-size", "${NOMAD_META_batch_size}",
          "--timeout", "${NOMAD_META_timeout}"
        ]
      }

      resources {
        cpu    = 1000
        memory = 2048
      }

      template {
        data = <<EOH
pipeline_type: ${NOMAD_META_pipeline_type}
data_source: ${NOMAD_META_data_source}
batch_size: ${NOMAD_META_batch_size}
timeout: ${NOMAD_META_timeout}
payload: {{ with secret "data/pipeline" }}{{ .Data.data.payload }}{{ end }}
EOH
        destination = "local/config.yaml"
      }
    }
  }
}
```

---

## Periodic Jobs

### What are Periodic Jobs?
Periodic jobs are scheduled to run at specific intervals. They are useful for maintenance tasks, backups, reports, and other time-based operations.

### Periodic Job Characteristics
- **Time-based**: Scheduled to run at specific intervals
- **Cron-like**: Uses cron syntax for scheduling
- **One-time execution**: Each instance runs to completion
- **Historical tracking**: Maintains history of executions

### Basic Periodic Job Example
```hcl
job "daily-backup" {
  datacenters = ["dc1"]
  type = "batch"

  periodic {
    cron = "0 2 * * *"  # Daily at 2 AM
  }

  group "backup" {
    count = 1

    task "backup" {
      driver = "docker"

      config {
        image = "backup-tool:latest"
        args = ["--source", "/data", "--destination", "/backup"]
      }

      resources {
        cpu    = 500
        memory = 1024
      }
    }
  }
}
```

### Advanced Periodic Job Example
```hcl
job "weekly-report" {
  datacenters = ["dc1"]
  type = "batch"

  periodic {
    cron = "0 6 * * 1"  # Every Monday at 6 AM
    time_zone = "UTC"
  }

  group "report" {
    count = 1

    task "generator" {
      driver = "docker"

      config {
        image = "report-generator:latest"
        args = [
          "--period", "weekly",
          "--output", "/reports",
          "--format", "pdf"
        ]
      }

      resources {
        cpu    = 1000
        memory = 2048
      }

      service {
        name = "report-generator"
        port = "8080"
      }
    }
  }
}
```

### Periodic Job with Constraints
```hcl
job "maintenance" {
  datacenters = ["dc1"]
  type = "batch"

  periodic {
    cron = "0 3 * * 0"  # Every Sunday at 3 AM
    prohibit_overlap = true
  }

  constraint {
    attribute = "${attr.kernel.name}"
    value     = "linux"
  }

  group "maintenance" {
    count = 1

    task "cleanup" {
      driver = "docker"

      config {
        image = "maintenance-tool:latest"
        args = ["--cleanup", "--optimize"]
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

## Job Type Comparison

### Detailed Comparison Table
```
┌─────────────────────────────────────────────────────────────┐
│                Job Type Comparison                          │
├─────────────────────────────────────────────────────────────┤
│ Feature           │ Service │ Batch │ System │ Param │ Per  │
├───────────────────┼─────────┼───────┼────────┼───────┼──────┤
│ Long-running      │ Yes     │ No    │ Yes    │ Dep   │ No   │
│ Auto-restart      │ Yes     │ No    │ Yes    │ Dep   │ Dep  │
│ Auto-reschedule   │ Yes     │ Yes   │ No     │ Dep   │ Dep  │
│ Scalable          │ Yes     │ No    │ Auto   │ Dep   │ Dep  │
│ Health checks     │ Yes     │ No    │ Yes    │ Dep   │ Dep  │
│ Service discovery │ Yes     │ No    │ Yes    │ Dep   │ Dep  │
│ Rolling updates   │ Yes     │ No    │ Yes    │ Dep   │ Dep  │
│ Canary deployments│ Yes     │ No    │ Yes    │ Dep   │ Dep  │
│ Resource limits   │ Yes     │ Yes   │ Yes    │ Yes   │ Yes  │
│ Constraints       │ Yes     │ Yes   │ Yes    │ Yes   │ Yes  │
│ Affinity          │ Yes     │ Yes   │ Yes    │ Yes   │ Yes  │
│ Templates         │ Yes     │ Yes   │ Yes    │ Yes   │ Yes  │
└─────────────────────────────────────────────────────────────┘
```

### Use Case Recommendations

#### Service Jobs
- Web servers and APIs
- Databases and message queues
- Microservices
- Long-running applications
- Applications requiring high availability

#### Batch Jobs
- Data processing pipelines
- Backup and maintenance tasks
- One-time operations
- ETL processes
- Report generation

#### System Jobs
- Monitoring agents
- Log collectors
- Security scanners
- Node-level services
- Infrastructure components

#### Parameterized Jobs
- Dynamic data processing
- Template-based deployments
- User-initiated tasks
- Configurable workflows
- Multi-tenant applications

#### Periodic Jobs
- Scheduled backups
- Regular maintenance
- Report generation
- Data cleanup
- Time-based operations

---

## Advanced Job Patterns

### Multi-Tier Applications
```hcl
job "web-application" {
  datacenters = ["dc1"]
  type = "service"

  group "frontend" {
    count = 3

    network {
      port "http" {
        static = 8080
      }
    }

    task "web" {
      driver = "docker"
      config {
        image = "myapp/frontend:latest"
        ports = ["http"]
      }

      service {
        name = "frontend"
        port = "http"
      }
    }
  }

  group "backend" {
    count = 2

    network {
      port "api" {
        static = 9090
      }
    }

    task "api" {
      driver = "docker"
      config {
        image = "myapp/backend:latest"
        ports = ["api"]
      }

      service {
        name = "backend"
        port = "api"
      }
    }
  }

  group "database" {
    count = 1

    task "db" {
      driver = "docker"
      config {
        image = "postgres:13"
      }

      service {
        name = "database"
        port = "5432"
      }
    }
  }
}
```

### Blue-Green Deployment
```hcl
job "blue-green" {
  datacenters = ["dc1"]
  type = "service"

  update {
    stagger      = "10s"
    max_parallel = 1
    health_check = "checks"
    auto_revert  = true
    auto_promote = false
  }

  group "app" {
    count = 3

    task "app" {
      driver = "docker"
      config {
        image = "myapp:blue"
      }

      service {
        name = "app-blue"
        port = "8080"
      }
    }
  }
}
```

### Canary Deployment
```hcl
job "canary" {
  datacenters = ["dc1"]
  type = "service"

  group "stable" {
    count = 4

    task "app" {
      driver = "docker"
      config {
        image = "myapp:stable"
      }

      service {
        name = "app"
        port = "8080"
      }
    }
  }

  group "canary" {
    count = 1

    task "app" {
      driver = "docker"
      config {
        image = "myapp:canary"
      }

      service {
        name = "app-canary"
        port = "8080"
      }
    }
  }
}
```

---

## Best Practices

### Job Design
1. **Choose the right type**: Select job type based on requirements
2. **Use appropriate resources**: Set realistic CPU and memory limits
3. **Implement health checks**: For service jobs
4. **Use constraints**: To control placement
5. **Use affinity**: To optimize placement

### Service Jobs
1. **Use rolling updates**: For zero-downtime deployments
2. **Implement health checks**: For proper monitoring
3. **Use service discovery**: For inter-service communication
4. **Set appropriate timeouts**: For health checks and updates
5. **Use auto-revert**: For failed deployments

### Batch Jobs
1. **Set restart policy**: Based on failure requirements
2. **Use resource limits**: To prevent resource exhaustion
3. **Implement proper logging**: For debugging
4. **Use parameterized jobs**: For reusable templates
5. **Handle failures gracefully**: With proper error handling

### System Jobs
1. **Use node constraints**: To target specific nodes
2. **Implement health checks**: For monitoring
3. **Use resource limits**: To prevent resource conflicts
4. **Handle node failures**: With proper restart policies
5. **Use templates**: For dynamic configuration

### Security
1. **Use ACLs**: For access control
2. **Use Vault integration**: For secrets management
3. **Use network policies**: For traffic control
4. **Use secure drivers**: When possible
5. **Regular updates**: Keep jobs updated

---

## Next Steps

After mastering job types, you'll be ready to:

1. **Advanced Features**: Learn about advanced scheduling and features
2. **Production Setup**: Deploy Nomad in production environments
3. **Integration**: Integrate with Consul and Vault
4. **Monitoring**: Set up comprehensive monitoring

### Practice Projects

1. **Multi-tier Application**: Deploy a complete web application
2. **Data Pipeline**: Create ETL jobs for data processing
3. **Monitoring Stack**: Deploy monitoring and logging
4. **CI/CD Integration**: Integrate with CI/CD pipelines

### Resources

- [Nomad Job Specification](https://www.nomadproject.io/docs/job-specification/)
- [Nomad Job Types](https://www.nomadproject.io/docs/job-specification/job/)
- [Nomad Scheduling](https://www.nomadproject.io/docs/internals/scheduling/)
- [Nomad Examples](https://github.com/hashicorp/nomad/tree/main/examples) 