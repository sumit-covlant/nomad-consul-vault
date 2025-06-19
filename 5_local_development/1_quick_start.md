# Quick Start: Nomad + Consul + Vault with Docker Compose

## Overview
This guide will help you quickly set up a local development environment with Nomad, Consul, and Vault using Docker Compose.

## Prerequisites
- Docker and Docker Compose installed
- At least 4GB RAM available
- Ports 8200, 8500, 4646, 8600 available

## Quick Setup

### 1. Create the Docker Compose file
Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  # Vault Server
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
    networks:
      - nomad-consul-vault

  # Consul Server
  consul:
    image: consul:latest
    container_name: consul
    ports:
      - "8500:8500"
      - "8600:8600/udp"
    environment:
      - CONSUL_BIND_INTERFACE=eth0
    command: consul agent -dev -client=0.0.0.0 -ui
    networks:
      - nomad-consul-vault

  # Nomad Server
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

### 2. Start the services
```bash
docker-compose up -d
```

### 3. Verify the services
```bash
# Check Vault
curl http://localhost:8200/v1/sys/health

# Check Consul
curl http://localhost:8500/v1/status/leader

# Check Nomad
curl http://localhost:4646/v1/status/leader
```

### 4. Access the UIs
- **Vault UI**: http://localhost:8200 (Token: `root`)
- **Consul UI**: http://localhost:8500
- **Nomad UI**: http://localhost:4646

## Next Steps
- Explore individual tools: [Vault Basics](2_vault_basics.md)
- Learn service discovery: [Consul Service Discovery](3_consul_service_discovery.md)
- Deploy your first job: [Nomad Jobs](4_nomad_jobs.md)
- Integrate all three: [Full Integration](5_full_integration.md)

## Troubleshooting
- If services fail to start, check port availability
- Ensure Docker has enough resources allocated
- Check logs: `docker-compose logs <service-name>` 