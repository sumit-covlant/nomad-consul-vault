# Vault Basics - Complete Guide

## Table of Contents
1. [What is Vault?](#what-is-vault)
2. [Vault Architecture](#vault-architecture)
3. [Key Features](#key-features)
4. [Use Cases](#use-cases)
5. [Installation and Setup](#installation-and-setup)
6. [First Steps](#first-steps)
7. [Best Practices](#best-practices)

---

## What is Vault?

Vault is an open-source tool by HashiCorp for securely managing secrets, encryption keys, and access to sensitive data. It provides a unified interface to any secret, tightly controls access, and records a detailed audit log.

### Why Use Vault?
- Centralized secrets management
- Dynamic secrets and credentials
- Encryption as a service
- Fine-grained access control
- Audit logging and compliance

---

## Vault Architecture

### High-Level Components
- **Vault Server**: The core service that manages secrets and handles requests
- **Storage Backend**: Where Vault persists data (e.g., integrated storage, Consul, S3, etc.)
- **Seal/Unseal**: Vault starts in a sealed state; unseal keys are required to decrypt the master key
- **API/UI/CLI**: Interfaces for interacting with Vault

### Architecture Diagram
```
┌─────────────────────────────────────────────────────────────┐
│                        Vault Cluster                       │
├─────────────────────────────────────────────────────────────┤
│                    +-------------------+                   │
│                    |   Vault Server    |                   │
│                    +-------------------+                   │
│                    |  Storage Backend  |                   │
│                    +-------------------+                   │
│                    |   Audit Devices   |                   │
│                    +-------------------+                   │
│                    |  Auth Methods     |                   │
│                    +-------------------+                   │
│                    | Secrets Engines   |                   │
│                    +-------------------+                   │
└─────────────────────────────────────────────────────────────┘
```

### Storage Backends
- Integrated Storage (Raft)
- Consul
- AWS S3
- Google Cloud Storage
- Azure Storage

### Seal/Unseal Process
- Vault is sealed on startup; master key is split using Shamir's Secret Sharing
- Unseal keys are required to reconstruct the master key and unseal Vault

---

## Key Features
- **Secrets Management**: Store, access, and control sensitive data
- **Dynamic Secrets**: Generate secrets on-demand (e.g., DB credentials)
- **Encryption as a Service**: Encrypt/decrypt data without storing it
- **Leasing and Renewal**: Secrets have TTLs and can be revoked
- **Audit Logging**: Track all access and operations
- **Access Control**: Policies define who can access what

---

## Use Cases
- Centralized secrets for applications and services
- Dynamic database credentials
- Cloud credentials (AWS, GCP, Azure)
- PKI and certificate management
- Encryption for application data
- Secure introduction and identity brokering

---

## Installation and Setup

### System Requirements
- OS: Linux, macOS, Windows
- CPU: 1+ cores
- Memory: 512MB+ (2GB+ recommended)
- Storage: 1GB+ free space

### Installation Methods
#### Binary
```bash
wget https://releases.hashicorp.com/vault/1.15.0/vault_1.15.0_linux_amd64.zip
unzip vault_1.15.0_linux_amd64.zip
sudo mv vault /usr/local/bin/
vault --version
```
#### Homebrew (macOS)
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/vault
```
#### Docker
```bash
docker run --cap-add=IPC_LOCK -e 'VAULT_DEV_ROOT_TOKEN_ID=root' -p 8200:8200 vault:latest
```

### Development Server
```bash
vault server -dev
# Exposes Vault at http://127.0.0.1:8200 with a root token
```

### Production Server (Raft Storage)
```hcl
# config.hcl
storage "raft" {
  path    = "/opt/vault/data"
  node_id = "vault-1"
}
listener "tcp" {
  address     = "0.0.0.0:8200"
  tls_disable = 1
}
ui = true
```

---

## First Steps

### Initialize and Unseal
```bash
vault operator init
vault operator unseal <unseal-key-1>
vault operator unseal <unseal-key-2>
vault operator unseal <unseal-key-3>
```

### Login
```bash
export VAULT_ADDR='http://127.0.0.1:8200'
vault login <root-token>
```

### Store and Retrieve a Secret
```bash
vault kv put secret/hello value=world
vault kv get secret/hello
```

---

## Best Practices
- Never store unseal keys or root tokens in plaintext
- Use integrated storage for simplicity and HA
- Enable audit logging
- Use policies for least-privilege access
- Regularly rotate secrets and tokens
- Use TLS in production
- Backup storage backend regularly

---

## Next Steps
- Learn core concepts: policies, auth methods, secrets engines
- Explore CLI and API usage
- Set up dynamic secrets and advanced features
- Prepare for production deployment 