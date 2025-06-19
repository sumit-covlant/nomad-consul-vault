# Consul + Vault Integration - Complete Guide

## Table of Contents
1. [Overview](#overview)
2. [Why Integrate Consul and Vault?](#why-integrate-consul-and-vault)
3. [Architecture](#architecture)
4. [Consul Connect mTLS with Vault CA](#consul-connect-mtls-with-vault-ca)
5. [Intentions and Access Control](#intentions-and-access-control)
6. [Configuration Examples](#configuration-examples)
7. [Best Practices](#best-practices)

---

## Overview

Consul and Vault integration enables secure service-to-service communication using Consul Connect's service mesh and Vault as the certificate authority (CA).

---

## Why Integrate Consul and Vault?
- **Automatic mTLS**: Vault issues certificates for Consul Connect
- **Centralized PKI**: Vault manages CA lifecycle and rotation
- **Fine-grained access control**: Use Consul intentions and Vault policies
- **Auditability**: All certificate issuance and access is logged

---

## Architecture
```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│  Consul     │ <--> │  Vault      │ <--> │  Applications│
│  Cluster    │      │  Cluster    │      │  & Services  │
└─────────────┘      └─────────────┘      └─────────────┘
```
- Consul Connect uses Vault as its CA provider
- Vault issues and rotates certificates for service mesh
- Consul manages service mesh and intentions

---

## Consul Connect mTLS with Vault CA

### How it Works
- Consul Connect is configured to use Vault as its CA
- Vault issues leaf and intermediate certificates
- All service-to-service traffic is encrypted with mTLS

### Example Consul CA Configuration
```hcl
connect {
  enabled = true
  ca_provider = "vault"
  ca_config = {
    address = "https://vault.service.consul:8200"
    token   = "<vault-token>"
    root_pki_path = "connect-root"
    intermediate_pki_path = "connect-intermediate"
  }
}
```

---

## Intentions and Access Control
- Consul intentions control which services can talk to each other
- Vault policies control access to CA endpoints

### Example Intention
```bash
consul intention create web api
consul intention create -deny web admin
```

### Example Vault Policy
```hcl
path "connect-intermediate/issue/*" {
  capabilities = ["update"]
}
```

---

## Configuration Examples

### Vault PKI Setup for Consul
```bash
vault secrets enable -path=connect-root pki
vault secrets enable -path=connect-intermediate pki
vault write connect-root/root/generate/internal common_name="consul.service.consul" ttl=87600h
vault write connect-intermediate/intermediate/generate/internal common_name="consul.service.consul" ttl=43800h
vault write connect-root/root/sign-intermediate csr=@csr.pem format=pem_bundle ttl=43800h > signed.pem
vault write connect-intermediate/intermediate/set-signed certificate=@signed.pem
```

---

## Best Practices
- Use Vault as the CA for Consul Connect
- Rotate certificates regularly
- Use intentions for service mesh access control
- Enable audit logging in both Consul and Vault
- Monitor certificate issuance and expiration

---

## Next Steps
- Integrate Nomad for workload orchestration
- Use Vault for dynamic secrets in services
- Automate certificate management 