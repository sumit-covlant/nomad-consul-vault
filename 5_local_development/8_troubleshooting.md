# Troubleshooting Guide

## Overview
Common issues and solutions when working with Nomad, Consul, and Vault in Docker.

## Prerequisites
- Basic understanding of Docker and Docker Compose
- Familiarity with [Quick Start](1_quick_start.md)

## Common Issues

### 1. Service Startup Issues

#### Vault Won't Start
```bash
# Check if port 8200 is in use
lsof -i :8200

# Check Vault logs
docker-compose logs vault

# Common solutions:
# 1. Kill process using port 8200
sudo kill -9 $(lsof -t -i:8200)

# 2. Check if Vault has proper permissions
docker-compose exec vault ls -la /vault

# 3. Restart Vault container
docker-compose restart vault
```

#### Consul Won't Start
```bash
# Check if port 8500 is in use
lsof -i :8500

# Check Consul logs
docker-compose logs consul

# Common solutions:
# 1. Kill process using port 8500
sudo kill -9 $(lsof -t -i:8500)

# 2. Check Consul configuration
docker-compose exec consul consul agent -dev -help

# 3. Restart Consul container
docker-compose restart consul
```

#### Nomad Won't Start
```bash
# Check if port 4646 is in use
lsof -i :4646

# Check Nomad logs
docker-compose logs nomad

# Common solutions:
# 1. Kill process using port 4646
sudo kill -9 $(lsof -t -i:4646)

# 2. Check Docker socket access
docker-compose exec nomad ls -la /var/run/docker.sock

# 3. Restart Nomad container
docker-compose restart nomad
```

### 2. Network Connectivity Issues

#### Services Can't Communicate
```bash
# Check if containers are on the same network
docker network ls
docker network inspect nomad-consul-vault

# Check container connectivity
docker-compose exec vault ping consul
docker-compose exec vault ping nomad

# Common solutions:
# 1. Recreate network
docker-compose down
docker network prune
docker-compose up -d

# 2. Check DNS resolution
docker-compose exec vault nslookup consul
docker-compose exec vault nslookup nomad
```

#### Port Binding Issues
```bash
# Check which ports are bound
netstat -tulpn | grep -E ':(8200|8500|4646)'

# Check Docker port mappings
docker-compose ps

# Common solutions:
# 1. Change ports in docker-compose.yml
# 2. Stop conflicting services
# 3. Use different port ranges
```

### 3. Authentication Issues

#### Vault Authentication
```bash
# Check Vault status
curl http://localhost:8200/v1/sys/health

# Set environment variables
export VAULT_ADDR='http://localhost:8200'
export VAULT_TOKEN='root'

# Test authentication
vault status

# Common solutions:
# 1. Check if Vault is unsealed
vault operator unseal

# 2. Verify token
vault token lookup

# 3. Generate new token
vault token create
```

#### Consul Authentication
```bash
# Check Consul status
curl http://localhost:8500/v1/status/leader

# Set environment variables
export CONSUL_HTTP_ADDR='http://localhost:8500'

# Test connectivity
consul members

# Common solutions:
# 1. Check ACL tokens if enabled
export CONSUL_HTTP_TOKEN='your-token'

# 2. Verify Consul is running
consul info
```

#### Nomad Authentication
```bash
# Check Nomad status
curl http://localhost:4646/v1/status/leader

# Set environment variables
export NOMAD_ADDR='http://localhost:4646'

# Test connectivity
nomad server members

# Common solutions:
# 1. Check ACL tokens if enabled
export NOMAD_TOKEN='your-token'

# 2. Verify Nomad is running
nomad agent info
```

### 4. Job Deployment Issues

#### Job Won't Deploy
```bash
# Check job status
nomad job status <job-name>

# Check allocation status
nomad alloc status <allocation-id>

# Check job logs
nomad alloc logs <allocation-id>

# Common solutions:
# 1. Validate job specification
nomad job validate <job-file>

# 2. Check resource availability
nomad node status

# 3. Check job specification syntax
nomad job plan <job-file>
```

#### Service Discovery Issues
```bash
# Check if service is registered
consul catalog services

# Check service health
consul health service <service-name>

# Check DNS resolution
dig @localhost -p 8600 <service-name>.service.consul

# Common solutions:
# 1. Verify service registration
consul services register <service-definition>

# 2. Check service tags
consul catalog nodes -service=<service-name>

# 3. Restart service
nomad job restart <job-name>
```

### 5. Secret Management Issues

#### Vault Secrets Not Accessible
```bash
# Check if secret exists
vault kv get secret/myapp

# Check policies
vault policy list
vault policy read <policy-name>

# Check token capabilities
vault token capabilities <token> secret/data/myapp

# Common solutions:
# 1. Create missing secrets
vault kv put secret/myapp key=value

# 2. Update policies
vault policy write <policy-name> <policy-file>

# 3. Generate new token
vault token create -policy=<policy-name>
```

#### Template Rendering Issues
```bash
# Check template syntax
nomad job plan <job-file>

# Check template logs
nomad alloc logs <allocation-id> -f

# Common solutions:
# 1. Validate template syntax
# 2. Check secret paths
# 3. Verify template permissions
```

## Debugging Commands

### 1. System Information
```bash
# Check Docker version
docker --version
docker-compose --version

# Check system resources
docker system df
docker stats

# Check container status
docker-compose ps
docker ps -a
```

### 2. Log Analysis
```bash
# View all logs
docker-compose logs

# View specific service logs
docker-compose logs vault
docker-compose logs consul
docker-compose logs nomad

# Follow logs in real-time
docker-compose logs -f vault

# Check container logs directly
docker logs <container-name>
```

### 3. Network Debugging
```bash
# Check network connectivity
docker-compose exec vault ping consul
docker-compose exec vault ping nomad

# Check DNS resolution
docker-compose exec vault nslookup consul
docker-compose exec vault nslookup nomad

# Check port availability
netstat -tulpn | grep -E ':(8200|8500|4646)'
```

### 4. Configuration Validation
```bash
# Validate Vault configuration
docker-compose exec vault vault server -config=/vault/config/vault.hcl -verify-only

# Validate Consul configuration
docker-compose exec consul consul validate /consul/config/consul.hcl

# Validate Nomad configuration
docker-compose exec nomad nomad agent -config=/nomad/config/nomad.hcl -dev
```

## Performance Issues

### 1. High Resource Usage
```bash
# Check resource usage
docker stats

# Check specific container resources
docker stats vault consul nomad

# Common solutions:
# 1. Increase resource limits in docker-compose.yml
# 2. Optimize configurations
# 3. Use resource constraints
```

### 2. Slow Response Times
```bash
# Check service response times
time curl http://localhost:8200/v1/sys/health
time curl http://localhost:8500/v1/status/leader
time curl http://localhost:4646/v1/status/leader

# Common solutions:
# 1. Optimize network configuration
# 2. Use host networking
# 3. Adjust resource allocation
```

## Recovery Procedures

### 1. Complete Reset
```bash
# Stop all services
docker-compose down

# Remove all containers and networks
docker-compose down --volumes --remove-orphans

# Clean up Docker system
docker system prune -a

# Restart services
docker-compose up -d
```

### 2. Data Recovery
```bash
# Backup current state
docker-compose exec vault vault operator raft snapshot save backup.snap
consul kv export > consul-backup.json

# Restore from backup
docker-compose exec vault vault operator raft snapshot restore backup.snap
consul kv import @consul-backup.json
```

### 3. Service Recovery
```bash
# Restart specific service
docker-compose restart vault
docker-compose restart consul
docker-compose restart nomad

# Recreate specific service
docker-compose up -d --force-recreate vault
```

## Getting Help

### 1. Official Documentation
- [Vault Documentation](https://www.vaultproject.io/docs)
- [Consul Documentation](https://www.consul.io/docs)
- [Nomad Documentation](https://www.nomadproject.io/docs)

### 2. Community Resources
- [HashiCorp Community](https://discuss.hashicorp.com/)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/hashicorp)
- [GitHub Issues](https://github.com/hashicorp)

### 3. Debugging Tools
```bash
# Install debugging tools
docker-compose exec vault apk add --no-cache curl jq
docker-compose exec consul apk add --no-cache curl jq
docker-compose exec nomad apk add --no-cache curl jq

# Use debugging tools
docker-compose exec vault jq . /vault/config/vault.hcl
docker-compose exec consul consul debug
docker-compose exec nomad nomad debug
```

## Next Steps
- Review [Production Considerations](7_production_considerations.md)
- Practice with different scenarios
- Build your own test cases 