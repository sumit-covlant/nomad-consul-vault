# HashiCorp Nomad, Consul, and Vault Learning Path

A comprehensive guide to mastering HashiCorp's core infrastructure tools: **Nomad** (orchestration), **Consul** (service discovery), and **Vault** (secrets management).

## 🎯 Learning Objectives

By the end of this guide, you'll be able to:
- Deploy and manage applications with Nomad
- Implement service discovery and mesh with Consul
- Secure applications with Vault secrets management
- Integrate all three tools for production-ready infrastructure
- Run everything locally using Docker and Docker Compose

## 📚 Learning Path

### 0. Prerequisites
Essential knowledge before diving into HashiCorp tools:
- **[Docker & Docker Compose](0_prerequisites/1_docker.md)** - Container fundamentals
- **[Linux CLI & Shell Scripting](0_prerequisites/2_linux.md)** - Command line mastery
- **[Networking Basics](0_prerequisites/3_networking.md)** - Network concepts
- **[Container Orchestration](0_prerequisites/4_container_orchestration.md)** - Orchestration concepts
- **[Service Discovery & Secrets](0_prerequisites/5_service_discovery_secrets.md)** - Core concepts

### 1. Nomad - Application Orchestration
Learn to deploy and manage applications at scale:
- **[Nomad Basics](1_nomad/1_nomad_basics.md)** - Architecture and core concepts
- **[CLI and API](1_nomad/2_nomad_cli_and_api.md)** - Command line and HTTP API usage
- **[Job Types](1_nomad/3_nomad_job_types.md)** - Service, batch, and system jobs
- **[Advanced Features](1_nomad/4_nomad_advanced_features.md)** - Autoscaling, multi-region, and more
- **[Production Setup](1_nomad/5_nomad_production_setup.md)** - High availability and monitoring

### 2. Consul - Service Discovery & Mesh
Master service discovery and secure service communication:
- **[Consul Basics](2_consul/1_consul_basics.md)** - Architecture and service registration
- **[Service Discovery](2_consul/2_consul_service_discovery.md)** - DNS, API, and health checks
- **[Consul Connect](2_consul/3_consul_connect.md)** - Service mesh with mTLS
- **[Advanced Features](2_consul/4_consul_advanced_features.md)** - ACLs, federation, and monitoring
- **[Production Setup](2_consul/5_consul_production_setup.md)** - Multi-datacenter deployment

### 3. Vault - Secrets Management
Secure your applications with centralized secrets management:
- **[Vault Basics](3_vault/1_vault_basics.md)** - Architecture and core concepts
- **[Core Concepts](3_vault/2_vault_core_concepts.md)** - Policies, authentication, and secrets engines
- **[CLI and API](3_vault/3_vault_cli_and_api.md)** - Command line and HTTP API usage
- **[Dynamic Secrets](3_vault/4_vault_dynamic_secrets.md)** - Database and cloud credentials
- **[Advanced Features](3_vault/5_vault_advanced_features.md)** - Auto-unseal, replication, and Vault Agent
- **[Production Setup](3_vault/6_vault_production_setup.md)** - High availability and security

### 4. Combined Use Cases
Learn to integrate these tools for powerful infrastructure:
- **[Nomad + Consul](4_combined_use_cases/1_nomad_consul.md)** - Service registration, discovery, health checks, and networking
- **[Nomad + Vault](4_combined_use_cases/2_nomad_vault.md)** - Secrets injection, Vault Agent, and secure workloads
- **[Consul + Vault](4_combined_use_cases/3_consul_vault.md)** - Service mesh mTLS, intentions, and secure service communication
- **[Nomad + Consul + Vault](4_combined_use_cases/4_nomad_consul_vault.md)** - End-to-end integration for secure, observable, and dynamic infrastructure

### 5. Local Development with Docker
**NEW!** Learn and experiment locally using Docker and Docker Compose:
- **[Quick Start](5_local_development/1_quick_start.md)** - Get all three tools running in minutes
- **[Vault Basics](5_local_development/2_vault_basics.md)** - Learn secrets management locally
- **[Consul Service Discovery](5_local_development/3_consul_service_discovery.md)** - Understand service mesh
- **[Nomad Jobs](5_local_development/4_nomad_jobs.md)** - Deploy and manage applications
- **[Full Integration](5_local_development/5_full_integration.md)** - Combine all three tools
- **[Advanced Features](5_local_development/6_advanced_features.md)** - Explore advanced capabilities
- **[Production Considerations](5_local_development/7_production_considerations.md)** - Prepare for production
- **[Troubleshooting](5_local_development/8_troubleshooting.md)** - Solve common issues
- **[Sample Applications](5_local_development/9_sample_applications.md)** - Real-world examples

## 🎯 Quick Integration Overview

### Nomad + Consul
- **Service Registration**: Automatic service registration in Consul
- **Health Checks**: Built-in health monitoring and failover
- **Service Discovery**: DNS and API-based service discovery
- **Load Balancing**: Automatic load balancing across healthy instances

### Nomad + Vault
- **Secrets Injection**: Secure secrets delivery to applications
- **Dynamic Credentials**: On-demand database and cloud credentials
- **Vault Agent**: Automated token management and secret renewal
- **Template Rendering**: Dynamic configuration with secrets

### Consul + Vault
- **Service Mesh Security**: mTLS encryption for service communication
- **Intentions**: Fine-grained access control between services
- **Certificate Management**: Automated certificate issuance and rotation
- **Secure Service Discovery**: Encrypted service communication

### Nomad + Consul + Vault
- **End-to-End Security**: Complete security from deployment to runtime
- **Dynamic Infrastructure**: Self-healing, auto-scaling applications
- **Observability**: Comprehensive monitoring and logging
- **Zero Trust**: Security-first architecture with least privilege access

## 🚀 Getting Started

### For Beginners
1. Start with [Prerequisites](0_prerequisites/) to build foundational knowledge
2. Begin with [Local Development](5_local_development/) for hands-on learning
3. Progress through individual tool guides
4. Practice with integration scenarios

### For Experienced Users
1. Jump directly to [Local Development](5_local_development/) for quick setup
2. Focus on [Combined Use Cases](4_combined_use_cases/) for integration patterns
3. Review [Production Setup](1_nomad/5_nomad_production_setup.md) guides for deployment

### For Production Teams
1. Study [Production Considerations](5_local_development/7_production_considerations.md)
2. Implement [Advanced Features](5_local_development/6_advanced_features.md)
3. Follow [Troubleshooting](5_local_development/8_troubleshooting.md) best practices
4. Use [Sample Applications](5_local_development/9_sample_applications.md) as templates

## 🛠️ Tools Covered

| Tool | Purpose | Key Features |
|------|---------|--------------|
| **Nomad** | Application Orchestration | Multi-cloud deployment, resource management, job scheduling |
| **Consul** | Service Discovery & Mesh | Service registration, health checks, mTLS, intentions |
| **Vault** | Secrets Management | Dynamic secrets, PKI, authentication, encryption |

## 📖 Additional Resources

- [HashiCorp Documentation](https://www.hashicorp.com/docs)
- [HashiCorp Learn](https://learn.hashicorp.com/)
- [Community Forums](https://discuss.hashicorp.com/)
- [GitHub Repositories](https://github.com/hashicorp)

## 🤝 Contributing

This guide is designed to be a living document. Feel free to:
- Suggest improvements
- Report issues
- Add new examples
- Share your experiences

---

**Ready to start your HashiCorp journey?** 

Begin with [Local Development](5_local_development/) for hands-on learning, or dive into the [Prerequisites](0_prerequisites/) if you need to build foundational knowledge first.