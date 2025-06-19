# Local Development with Docker

## Overview
This directory contains comprehensive guides for learning and experimenting with HashiCorp's Nomad, Consul, and Vault using Docker and Docker Compose in a local development environment.

## 🚀 Quick Start

### Prerequisites
- Docker and Docker Compose installed
- At least 4GB RAM available
- Basic familiarity with command line tools

### Getting Started
1. **Start with [Quick Start](1_quick_start.md)** - Get all three tools running in minutes
2. **Explore individual tools** - Learn each tool's basics
3. **Build integrated solutions** - Combine all three tools
4. **Practice with samples** - Work through real-world examples

## 📚 Learning Path

### Phase 1: Foundation
- **[1. Quick Start](1_quick_start.md)** - Get everything running quickly
- **[2. Vault Basics](2_vault_basics.md)** - Learn secrets management
- **[3. Consul Service Discovery](3_consul_service_discovery.md)** - Understand service mesh
- **[4. Nomad Jobs](4_nomad_jobs.md)** - Deploy and manage applications

### Phase 2: Integration
- **[5. Full Integration](5_full_integration.md)** - Combine all three tools
- **[6. Advanced Features](6_advanced_features.md)** - Explore advanced capabilities
- **[7. Production Considerations](7_production_considerations.md)** - Prepare for production

### Phase 3: Practice
- **[8. Troubleshooting](8_troubleshooting.md)** - Solve common issues
- **[9. Sample Applications](9_sample_applications.md)** - Real-world examples

## 🛠️ What You'll Learn

### Vault
- Secrets storage and retrieval
- Dynamic credentials
- PKI certificate management
- Authentication methods
- Policy-based access control

### Consul
- Service registration and discovery
- Health checking
- Key-value storage
- Service mesh (Consul Connect)
- DNS-based service discovery

### Nomad
- Job scheduling and deployment
- Resource management
- Service and batch jobs
- Multi-region deployments
- Integration with Consul and Vault

### Integration Patterns
- Secure secrets injection
- Service mesh with mTLS
- Dynamic configuration
- Health monitoring
- Automated deployments

## 🎯 Use Cases Covered

### Simple Applications
- Basic web services
- Static content delivery
- Simple API endpoints

### Complex Systems
- Microservices architectures
- Multi-tier applications
- Service mesh implementations
- Batch processing workflows

### Production Scenarios
- High availability setups
- Security hardening
- Monitoring and alerting
- Backup and recovery

## 🏗️ Architecture Examples

### Basic Setup
```
┌─────────┐    ┌─────────┐    ┌─────────┐
│  Vault  │    │ Consul  │    │  Nomad  │
│  8200   │    │  8500   │    │  4646   │
└─────────┘    └─────────┘    └─────────┘
     │              │              │
     └──────────────┼──────────────┘
                    │
              ┌─────────┐
              │ Your    │
              │ Apps    │
              └─────────┘
```

### Service Mesh
```
┌─────────┐    ┌─────────┐    ┌─────────┐
│ Frontend│◄──►│ Backend │◄──►│Database │
│  Proxy  │    │  Proxy  │    │  Proxy  │
└─────────┘    └─────────┘    └─────────┘
     │              │              │
     └──────────────┼──────────────┘
                    │
              ┌─────────┐
              │ Consul  │
              │ Connect │
              └─────────┘
```

## 🚦 Getting Help

### Common Issues
- **Port conflicts**: Check if ports 8200, 8500, 4646 are available
- **Memory issues**: Ensure Docker has enough resources allocated
- **Network problems**: Verify containers can communicate

### Debugging
- Check service logs: `docker-compose logs <service>`
- Verify connectivity: `docker-compose exec <service> ping <other-service>`
- Test endpoints: `curl http://localhost:<port>/v1/sys/health`

### Resources
- [Official Documentation](https://www.hashicorp.com/docs)
- [Community Forums](https://discuss.hashicorp.com/)
- [GitHub Issues](https://github.com/hashicorp)

## 🎓 Next Steps

After completing this local development guide:

1. **Explore the main guides** in the parent directory
2. **Practice with your own applications**
3. **Set up a production environment**
4. **Join the HashiCorp community**
5. **Contribute to open source projects**

## 📝 Notes

- All examples use development mode for simplicity
- Production deployments require additional security considerations
- This environment is for learning and development only
- Always follow security best practices in production

---

**Happy Learning! 🎉**

Start with [Quick Start](1_quick_start.md) to get everything running in minutes. 