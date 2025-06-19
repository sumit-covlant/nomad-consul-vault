# Nomad Basics - Complete Guide

## Table of Contents
1. [What is Nomad?](#what-is-nomad)
2. [Nomad Architecture](#nomad-architecture)
3. [Core Concepts](#core-concepts)
4. [Installation and Setup](#installation-and-setup)
5. [First Steps](#first-steps)
6. [Understanding the Scheduler](#understanding-the-scheduler)
7. [Best Practices](#best-practices)

---

## What is Nomad?

### Overview
Nomad is a flexible workload orchestrator that enables you to deploy and manage any containerized or legacy application using a single, unified workflow. It can manage a diverse workload of applications, containers, and microservices across on-premises and cloud environments.

### Key Features
```
┌─────────────────────────────────────────────────────────────┐
│                    Nomad Features                           │
├─────────────────────────────────────────────────────────────┤
│ 🚀 Simple & Powerful: Single binary, easy to deploy        │
│ 🔄 Multi-Datacenter: Native multi-DC and multi-region      │
│ 🐳 Container Support: Docker, containerd, Podman           │
│ 🖥️  VM Support: QEMU, VirtualBox, VMware                   │
│ 📦 Application Support: Java, Python, Go, Node.js          │
│ 🔐 Security: ACLs, mTLS, Vault integration                 │
│ 📊 Observability: Built-in metrics and logging             │
│ 🔧 Extensible: Plugin system for custom drivers            │
└─────────────────────────────────────────────────────────────┘
```

### Use Cases
- **Application Deployment**: Deploy and manage applications across clusters
- **Service Orchestration**: Coordinate microservices and their dependencies
- **Batch Processing**: Run batch jobs and scheduled tasks
- **System Administration**: Manage system-level services across infrastructure
- **Multi-Cloud**: Deploy applications across different cloud providers

### Comparison with Other Orchestrators
```
┌─────────────────────────────────────────────────────────────┐
│                Orchestrator Comparison                      │
├─────────────────────────────────────────────────────────────┤
│ Nomad:      Simple, flexible, multi-workload               │
│ Kubernetes: Complex, container-focused, extensive ecosystem │
│ Docker Swarm: Docker-native, simple, limited features      │
│ Mesos:      Resource management, complex, declining usage  │
└─────────────────────────────────────────────────────────────┘
```

---

## Nomad Architecture

### High-Level Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    Nomad Cluster                           │
├─────────────────────────────────────────────────────────────┤
│                    Server Nodes                            │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │   Server 1  │ │   Server 2  │ │   Server 3  │           │
│ │  (Leader)   │ │  (Follower) │ │  (Follower) │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
│         │               │               │                 │
│         └───────────────┼───────────────┘                 │
│                         │                                 │
│                    Client Nodes                           │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │  Client 1   │ │  Client 2   │ │  Client 3   │           │
│ │ ┌─┐ ┌─┐ ┌─┐ │ │ ┌─┐ ┌─┐ ┌─┐ │ │ ┌─┐ ┌─┐ ┌─┐ │           │
│ │ │A│ │B│ │C│ │ │ │D│ │E│ │F│ │ │ │G│ │H│ │I│ │           │
│ │ └─┘ └─┘ └─┘ │ │ └─┘ └─┘ └─┘ │ │ └─┘ └─┘ └─┘ │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

### Server Nodes
Server nodes are responsible for:
- **Consensus**: Using Raft protocol for leader election and data consistency
- **Scheduling**: Making placement decisions for jobs
- **API**: Providing HTTP API for job management
- **State Management**: Storing job definitions and allocations

### Client Nodes
Client nodes are responsible for:
- **Task Execution**: Running actual workloads
- **Resource Management**: Monitoring and reporting resource usage
- **Health Reporting**: Reporting task health and status
- **Driver Management**: Managing different task drivers (Docker, exec, etc.)

### Communication Flow
```
1. Job Submission
   Client ──▶ Server (API) ──▶ Scheduler ──▶ Placement Decision

2. Task Execution
   Server ──▶ Client ──▶ Driver ──▶ Task Execution

3. Status Updates
   Task ──▶ Driver ──▶ Client ──▶ Server ──▶ API Response
```

---

## Core Concepts

### Jobs
A job is the primary configuration unit in Nomad. It describes an application or service that should be running in the cluster.

```hcl
job "example" {
  datacenters = ["dc1"]
  type = "service"

  group "web" {
    count = 3

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
    }
  }
}
```

### Task Groups
Task groups are collections of tasks that should be co-located on the same node. They share networking and can communicate with each other.

```hcl
group "web" {
  count = 3

  network {
    port "http" {
      static = 8080
    }
  }

  task "web" {
    driver = "docker"
    config {
      image = "nginx:latest"
      ports = ["http"]
    }
  }

  task "cache" {
    driver = "docker"
    config {
      image = "redis:alpine"
    }
  }
}
```

### Tasks
Tasks are the smallest unit of work in Nomad. They represent a single process or container that should be running.

```hcl
task "web" {
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
```

### Allocations
Allocations are the result of the scheduler placing tasks on specific nodes. They represent the actual running instances of your jobs.

```bash
# View allocations
nomad alloc status <allocation-id>

# View allocation logs
nomad alloc logs <allocation-id>

# Execute command in allocation
nomad alloc exec <allocation-id> /bin/bash
```

### Drivers
Drivers are plugins that allow Nomad to run different types of workloads.

#### Supported Drivers
```
┌─────────────────────────────────────────────────────────────┐
│                    Nomad Drivers                           │
├─────────────────────────────────────────────────────────────┤
│ docker:     Run Docker containers                          │
│ exec:       Run binaries directly on the host              │
│ java:       Run Java applications                          │
│ qemu:       Run virtual machines                           │
│ raw_exec:   Run binaries with minimal isolation            │
│ podman:     Run Podman containers                          │
│ rkt:        Run rkt containers (deprecated)                │
└─────────────────────────────────────────────────────────────┘
```

---

## Installation and Setup

### System Requirements
```
┌─────────────────────────────────────────────────────────────┐
│                System Requirements                         │
├─────────────────────────────────────────────────────────────┤
│ OS:         Linux, macOS, Windows                          │
│ CPU:        1+ cores                                       │
│ Memory:     512MB+ (2GB+ recommended)                     │
│ Storage:    1GB+ available space                          │
│ Network:    TCP ports 4646 (HTTP), 4647 (RPC), 4648 (Serf)│
└─────────────────────────────────────────────────────────────┘
```

### Installation Methods

#### Binary Installation
```bash
# Download Nomad
wget https://releases.hashicorp.com/nomad/1.6.0/nomad_1.6.0_linux_amd64.zip
unzip nomad_1.6.0_linux_amd64.zip
sudo mv nomad /usr/local/bin/

# Verify installation
nomad version
```

#### Package Manager Installation
```bash
# Ubuntu/Debian
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install nomad

# CentOS/RHEL
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install nomad

# macOS
brew tap hashicorp/tap
brew install hashicorp/tap/nomad
```

### Development Setup
```bash
# Start Nomad in development mode
nomad agent -dev

# This starts:
# - Single server node
# - Single client node
# - Web UI at http://localhost:4646
# - API at http://localhost:4646
```

### Production Setup
```bash
# Create configuration directory
sudo mkdir -p /etc/nomad.d

# Create server configuration
sudo tee /etc/nomad.d/server.hcl <<EOF
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
  retry_join = ["192.168.1.11:4648", "192.168.1.12:4648"]
}
EOF

# Create client configuration
sudo tee /etc/nomad.d/client.hcl <<EOF
data_dir = "/opt/nomad/data"
bind_addr = "0.0.0.0"

client {
  enabled = true
  servers = ["192.168.1.10:4647", "192.168.1.11:4647", "192.168.1.12:4647"]
}

advertise {
  http = "192.168.1.20"
  rpc  = "192.168.1.20"
  serf = "192.168.1.20"
}
EOF

# Start Nomad
sudo systemctl enable nomad
sudo systemctl start nomad
```

---

## First Steps

### Starting Nomad
```bash
# Development mode (single node)
nomad agent -dev

# Check status
nomad server members
nomad node status
```

### Your First Job
Create a simple job file `example.nomad`:

```hcl
job "example" {
  datacenters = ["dc1"]
  type = "service"

  group "web" {
    count = 1

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

### Running the Job
```bash
# Plan the job
nomad job plan example.nomad

# Run the job
nomad job run example.nomad

# Check job status
nomad job status example

# View job logs
nomad job logs example

# Access the application
curl http://localhost:8080
```

### Basic Commands
```bash
# Job management
nomad job list
nomad job status <job-name>
nomad job stop <job-name>
nomad job restart <job-name>

# Allocation management
nomad alloc list
nomad alloc status <alloc-id>
nomad alloc logs <alloc-id>

# Node management
nomad node status
nomad node status <node-id>
nomad node drain <node-id>

# System information
nomad server members
nomad agent info
```

---

## Understanding the Scheduler

### Scheduling Process
```
1. Job Submission
   └─ Job definition submitted to Nomad server

2. Job Validation
   └─ Validate job syntax and constraints

3. Scheduling
   └─ Scheduler evaluates placement options

4. Placement Decision
   └─ Select optimal node for each task

5. Task Execution
   └─ Client node starts the task

6. Status Updates
   └─ Monitor task health and status
```

### Scheduling Algorithms

#### Bin Packing
Nomad uses bin packing to efficiently utilize cluster resources.

```hcl
job "example" {
  datacenters = ["dc1"]
  type = "service"

  group "web" {
    count = 5

    task "server" {
      driver = "docker"
      config {
        image = "nginx:latest"
      }
      resources {
        cpu    = 200  # 0.2 cores
        memory = 256  # 256MB
      }
    }
  }
}
```

#### Spread Scheduling
Spread tasks across nodes for high availability.

```hcl
job "example" {
  datacenters = ["dc1"]
  type = "service"

  group "web" {
    count = 3

    spread {
      attribute = "${attr.unique.hostname}"
      target "dc1" {
        percent = 100
      }
    }

    task "server" {
      driver = "docker"
      config {
        image = "nginx:latest"
      }
    }
  }
}
```

### Constraints
Constraints limit where tasks can be scheduled.

```hcl
job "example" {
  datacenters = ["dc1"]
  type = "service"

  group "web" {
    count = 3

    constraint {
      attribute = "${attr.kernel.name}"
      value     = "linux"
    }

    constraint {
      attribute = "${attr.cpu.arch}"
      value     = "amd64"
    }

    task "server" {
      driver = "docker"
      config {
        image = "nginx:latest"
      }
    }
  }
}
```

### Affinity
Affinity rules prefer certain placement options.

```hcl
job "example" {
  datacenters = ["dc1"]
  type = "service"

  group "web" {
    count = 3

    affinity "prefer_ssd" {
      attribute = "${attr.storage.ssd}"
      value     = "true"
      weight    = 50
    }

    task "server" {
      driver = "docker"
      config {
        image = "nginx:latest"
      }
    }
  }
}
```

---

## Best Practices

### Job Design
1. **Use Task Groups**: Group related tasks together
2. **Resource Limits**: Always specify resource requirements
3. **Health Checks**: Implement proper health checks
4. **Service Discovery**: Use Nomad's service discovery features
5. **Configuration Management**: Use templates and environment variables

### Security
1. **ACLs**: Enable and configure Access Control Lists
2. **TLS**: Use TLS for all communications
3. **Vault Integration**: Use Vault for secrets management
4. **Network Policies**: Implement network segmentation
5. **Regular Updates**: Keep Nomad updated

### Performance
1. **Resource Planning**: Plan resource requirements carefully
2. **Monitoring**: Monitor cluster performance
3. **Scaling**: Use appropriate scaling strategies
4. **Optimization**: Optimize job configurations
5. **Cleanup**: Regularly clean up old allocations

### Operational
1. **Documentation**: Document job configurations
2. **Testing**: Test jobs in development first
3. **Backup**: Backup job definitions
4. **Monitoring**: Set up comprehensive monitoring
5. **Disaster Recovery**: Plan for failure scenarios

---

## Next Steps

After mastering Nomad basics, you'll be ready to:

1. **Learn CLI and API**: Master Nomad's command-line interface and API
2. **Explore Job Types**: Understand different job types and their use cases
3. **Advanced Features**: Learn about advanced scheduling and features
4. **Production Setup**: Deploy Nomad in production environments

### Practice Projects

1. **Simple Web Application**: Deploy a multi-tier web application
2. **Batch Processing**: Create batch jobs for data processing
3. **System Services**: Deploy system-level services
4. **Microservices**: Build and deploy microservices architecture

### Resources

- [Nomad Documentation](https://www.nomadproject.io/docs/)
- [Nomad GitHub Repository](https://github.com/hashicorp/nomad)
- [Nomad Community](https://discuss.hashicorp.com/c/nomad)
- [Nomad Learn](https://learn.hashicorp.com/nomad) 