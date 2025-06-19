# Nomad + Consul Integration - Complete Guide

## Table of Contents
1. [Overview](#overview)
2. [Why Integrate Nomad and Consul?](#why-integrate-nomad-and-consul)
3. [Architecture](#architecture)
4. [Service Registration and Discovery](#service-registration-and-discovery)
5. [Health Checks](#health-checks)
6. [Networking and DNS](#networking-and-dns)
7. [Configuration Examples](#configuration-examples)
8. [Best Practices](#best-practices)

---

## Overview

Nomad and Consul are designed to work together for dynamic service discovery, health checking, and networking in modern infrastructure. Nomad schedules and runs workloads, while Consul provides service discovery and health monitoring.

---

## Why Integrate Nomad and Consul?
- **Automatic service registration**: Nomad jobs are registered in Consul automatically
- **Dynamic service discovery**: Applications can find services via Consul DNS or API
- **Health checks**: Consul monitors the health of Nomad tasks
- **Service mesh**: Enable Consul Connect for secure service-to-service communication

---

## Architecture
```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│  Nomad      │ <--> │  Consul     │ <--> │  Applications│
│  Cluster    │      │  Cluster    │      │  & Services  │
└─────────────┘      └─────────────┘      └─────────────┘
```
- Nomad clients and servers run Consul agents
- Nomad jobs register services in Consul
- Consul provides DNS and API for service discovery

---

## Service Registration and Discovery

### How it Works
- Nomad jobs define `service` blocks
- Nomad registers/deregisters services in Consul automatically
- Consul tracks service health and availability

### Example Nomad Job with Service Registration
```hcl
job "web" {
  group "web" {
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
    }
  }
}
```

### Discovering Services
- **DNS**: `web.service.consul`
- **API**: `GET /v1/catalog/service/web`

---

## Health Checks
- Nomad can define health checks in the job spec
- Consul executes and tracks health checks
- Only healthy services are discoverable

---

## Networking and DNS
- Consul provides DNS for service discovery
- Nomad jobs can use Consul DNS to find other services
- Supports dynamic IPs and ports

---

## Configuration Examples

### Nomad Client Configuration
```hcl
consul {
  address = "127.0.0.1:8500"
  auto_advertise = true
  client_auto_join = true
}
```

### Consul Agent Configuration
```hcl
server = true
ui = true
bind_addr = "0.0.0.0"
data_dir = "/opt/consul/data"
```

---

## Best Practices
- Always enable health checks for services
- Use tags for service versioning and environments
- Secure Consul with ACLs and TLS
- Monitor service health and logs
- Use Consul Connect for service mesh and mTLS

---

## Next Steps
- Integrate Vault for secrets management
- Enable Consul Connect for service mesh
- Automate job and service registration 