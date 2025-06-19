# Nomad CLI and API - Complete Guide

## Table of Contents
1. [Command Line Interface](#command-line-interface)
2. [Job Management Commands](#job-management-commands)
3. [Allocation Management](#allocation-management)
4. [Node Management](#node-management)
5. [System Commands](#system-commands)
6. [HTTP API](#http-api)
7. [API Authentication](#api-authentication)
8. [API Examples](#api-examples)
9. [Advanced CLI Features](#advanced-cli-features)
10. [Troubleshooting](#troubleshooting)

---

## Command Line Interface

### Basic CLI Structure
```bash
nomad [global options] <command> [command options] [arguments...]
```

### Global Options
```bash
# Common global options
nomad [options] <command>

Options:
  -address=<addr>     # Address of the Nomad server
  -region=<region>    # Region to query
  -namespace=<ns>     # Target namespace for queries and actions
  -token=<token>      # ACL token to use
  -ca-cert=<path>     # Path to CA certificate
  -ca-path=<path>     # Path to CA certificate directory
  -client-cert=<path> # Path to client certificate
  -client-key=<path>  # Path to client key
  -tls-skip-verify    # Skip TLS certificate verification
  -json               # Output in JSON format
  -t                  # Format and display template
  -verbose            # Enable verbose output
```

### Environment Variables
```bash
# Set environment variables for CLI
export NOMAD_ADDR=http://localhost:4646
export NOMAD_REGION=global
export NOMAD_NAMESPACE=default
export NOMAD_TOKEN=your-token-here
export NOMAD_CACERT=/path/to/ca.crt
export NOMAD_CLIENT_CERT=/path/to/client.crt
export NOMAD_CLIENT_KEY=/path/to/client.key
export NOMAD_SKIP_VERIFY=true
```

---

## Job Management Commands

### Job Lifecycle Commands

#### Plan Job
```bash
# Plan a job to see what changes would be made
nomad job plan example.nomad

# Plan with specific variables
nomad job plan -var="count=5" example.nomad

# Plan and output in JSON
nomad job plan -json example.nomad

# Plan with diff
nomad job plan -diff example.nomad
```

#### Run Job
```bash
# Run a job
nomad job run example.nomad

# Run with variables
nomad job run -var="count=3" example.nomad

# Run with variable files
nomad job run -var-file="production.tfvars" example.nomad

# Run and detach
nomad job run -detach example.nomad

# Run with check-index
nomad job run -check-index=123 example.nomad
```

#### Job Status
```bash
# List all jobs
nomad job list

# Get detailed job status
nomad job status example

# Get job status in JSON
nomad job status -json example

# Get job status with template
nomad job status -t '{{range .TaskGroups}}{{.Name}}: {{.Queued}}/{{.Running}}/{{.Complete}}{{end}}' example
```

#### Job Control
```bash
# Stop a job
nomad job stop example

# Stop with purge (remove from state)
nomad job stop -purge example

# Stop with detach
nomad job stop -detach example

# Restart a job
nomad job restart example

# Restart specific task group
nomad job restart -group=web example

# Restart specific task
nomad job restart -group=web -task=server example
```

#### Job Inspection
```bash
# Inspect job definition
nomad job inspect example

# Inspect in JSON format
nomad job inspect -json example

# Inspect specific version
nomad job inspect -version=2 example

# Inspect with template
nomad job inspect -t '{{.Job.Name}}: {{.Job.Type}}' example
```

#### Job History
```bash
# Show job history
nomad job history example

# Show history in JSON
nomad job history -json example

# Show specific version
nomad job history -version=5 example
```

#### Job Scaling
```bash
# Scale a job
nomad job scale example 5

# Scale specific task group
nomad job scale -group=web example 3

# Scale with detach
nomad job scale -detach example 5

# Scale with check-index
nomad job scale -check-index=123 example 5
```

### Job Templates and Variables

#### Variable Files
```hcl
# variables.tfvars
count = 3
image = "nginx:1.19"
cpu = 500
memory = 512
```

```hcl
# job.nomad
job "example" {
  datacenters = ["dc1"]
  type = "service"

  group "web" {
    count = var.count

    task "server" {
      driver = "docker"
      config {
        image = var.image
      }
      resources {
        cpu    = var.cpu
        memory = var.memory
      }
    }
  }
}
```

#### Template Functions
```hcl
# Using template functions
job "example" {
  datacenters = ["dc1"]
  type = "service"

  group "web" {
    count = 3

    task "server" {
      driver = "docker"
      config {
        image = "nginx:${var.version}"
      }
      env {
        ENVIRONMENT = var.environment
        LOG_LEVEL   = var.log_level
      }
    }
  }
}
```

---

## Allocation Management

### Allocation Commands

#### List Allocations
```bash
# List all allocations
nomad alloc list

# List allocations for specific job
nomad alloc list -job=example

# List allocations in JSON
nomad alloc list -json

# List with template
nomad alloc list -t '{{range .}}{{.ID}}: {{.JobID}} - {{.Status}}{{end}}'
```

#### Allocation Status
```bash
# Get allocation status
nomad alloc status <alloc-id>

# Get status in JSON
nomad alloc status -json <alloc-id>

# Get status with template
nomad alloc status -t '{{.ID}}: {{.Status}} - {{.DesiredStatus}}' <alloc-id>
```

#### Allocation Logs
```bash
# Get allocation logs
nomad alloc logs <alloc-id>

# Get logs for specific task
nomad alloc logs -task=server <alloc-id>

# Follow logs
nomad alloc logs -f <alloc-id>

# Get logs with timestamps
nomad alloc logs -t <alloc-id>

# Get logs for specific time range
nomad alloc logs -since=1h <alloc-id>

# Get logs with tail
nomad alloc logs -tail=100 <alloc-id>

# Get logs in JSON
nomad alloc logs -json <alloc-id>
```

#### Allocation Execution
```bash
# Execute command in allocation
nomad alloc exec <alloc-id> /bin/bash

# Execute in specific task
nomad alloc exec -task=server <alloc-id> /bin/bash

# Execute with specific user
nomad alloc exec -user=root <alloc-id> /bin/bash

# Execute with environment
nomad alloc exec -env=DEBUG=1 <alloc-id> /bin/bash

# Execute with working directory
nomad alloc exec -cwd=/app <alloc-id> /bin/bash
```

#### Allocation Restart
```bash
# Restart allocation
nomad alloc restart <alloc-id>

# Restart specific task
nomad alloc restart -task=server <alloc-id>

# Restart with detach
nomad alloc restart -detach <alloc-id>
```

#### Allocation Signal
```bash
# Send signal to allocation
nomad alloc signal <alloc-id> SIGTERM

# Send signal to specific task
nomad alloc signal -task=server <alloc-id> SIGTERM
```

---

## Node Management

### Node Commands

#### List Nodes
```bash
# List all nodes
nomad node status

# List nodes in JSON
nomad node status -json

# List nodes with template
nomad node status -t '{{range .}}{{.ID}}: {{.Status}} - {{.Datacenter}}{{end}}'
```

#### Node Status
```bash
# Get node status
nomad node status <node-id>

# Get status in JSON
nomad node status -json <node-id>

# Get status with template
nomad node status -t '{{.ID}}: {{.Status}} - {{.Address}}' <node-id>
```

#### Node Drain
```bash
# Drain a node
nomad node drain <node-id>

# Drain with detach
nomad node drain -detach <node-id>

# Drain with deadline
nomad node drain -deadline=1h <node-id>

# Drain with ignore system jobs
nomad node drain -ignore-system <node-id>

# Drain with no shutdown
nomad node drain -no-shutdown <node-id>
```

#### Node Eligibility
```bash
# Set node eligibility
nomad node eligibility -enable <node-id>

# Disable node eligibility
nomad node eligibility -disable <node-id>
```

#### Node GC
```bash
# Garbage collect a node
nomad node gc <node-id>
```

---

## System Commands

### Server Commands

#### Server Members
```bash
# List server members
nomad server members

# List members in JSON
nomad server members -json

# List members with template
nomad server members -t '{{range .}}{{.Name}}: {{.Status}} - {{.Address}}{{end}}'
```

#### Server Join
```bash
# Join server to cluster
nomad server join <address>
```

#### Server Leave
```bash
# Leave server from cluster
nomad server leave
```

### Agent Commands

#### Agent Info
```bash
# Get agent information
nomad agent info

# Get info in JSON
nomad agent info -json
```

#### Agent Reload
```bash
# Reload agent configuration
nomad agent reload
```

### Operator Commands

#### Operator Raft
```bash
# Show Raft configuration
nomad operator raft list-peers

# Remove Raft peer
nomad operator raft remove-peer <address>
```

#### Operator Scheduler
```bash
# Get scheduler configuration
nomad operator scheduler get-config

# Set scheduler configuration
nomad operator scheduler set-config <config-file>
```

---

## HTTP API

### API Endpoints

#### Base URL
```
http://localhost:4646/v1/
```

#### Authentication
```bash
# Set token in header
curl -H "X-Nomad-Token: your-token" http://localhost:4646/v1/jobs

# Set token in query parameter
curl "http://localhost:4646/v1/jobs?token=your-token"
```

### Job API Endpoints

#### List Jobs
```bash
# List all jobs
curl http://localhost:4646/v1/jobs

# List jobs with prefix
curl "http://localhost:4646/v1/jobs?prefix=web"

# List jobs in specific namespace
curl "http://localhost:4646/v1/jobs?namespace=production"
```

#### Get Job
```bash
# Get job details
curl http://localhost:4646/v1/job/example

# Get job with query options
curl "http://localhost:4646/v1/job/example?pretty=true"
```

#### Create/Update Job
```bash
# Create job
curl -X POST -d @job.json http://localhost:4646/v1/jobs

# Update job
curl -X PUT -d @job.json http://localhost:4646/v1/job/example
```

#### Delete Job
```bash
# Delete job
curl -X DELETE http://localhost:4646/v1/job/example

# Delete with purge
curl -X DELETE "http://localhost:4646/v1/job/example?purge=true"
```

#### Job Scaling
```bash
# Scale job
curl -X POST -d '{"Count": 5}' http://localhost:4646/v1/job/example/scale
```

### Allocation API Endpoints

#### List Allocations
```bash
# List all allocations
curl http://localhost:4646/v1/allocations

# List allocations for job
curl "http://localhost:4646/v1/job/example/allocations"
```

#### Get Allocation
```bash
# Get allocation details
curl http://localhost:4646/v1/allocation/<alloc-id>
```

#### Allocation Logs
```bash
# Get allocation logs
curl "http://localhost:4646/v1/client/fs/logs/<alloc-id>?task=server"

# Get logs with query parameters
curl "http://localhost:4646/v1/client/fs/logs/<alloc-id>?task=server&origin=start&offset=1000"
```

### Node API Endpoints

#### List Nodes
```bash
# List all nodes
curl http://localhost:4646/v1/nodes
```

#### Get Node
```bash
# Get node details
curl http://localhost:4646/v1/node/<node-id>
```

#### Node Drain
```bash
# Drain node
curl -X POST -d '{"DrainSpec": {"Deadline": "1h"}}' http://localhost:4646/v1/node/<node-id>/drain
```

### System API Endpoints

#### Agent Self
```bash
# Get agent information
curl http://localhost:4646/v1/agent/self
```

#### Agent Members
```bash
# Get server members
curl http://localhost:4646/v1/agent/members
```

#### Agent Join
```bash
# Join server
curl -X POST -d '["192.168.1.10:4648"]' http://localhost:4646/v1/agent/join
```

---

## API Authentication

### ACL Tokens
```bash
# Create ACL token
curl -X POST -d '{
  "Name": "my-token",
  "Type": "client",
  "Policies": ["readonly"]
}' http://localhost:4646/v1/acl/token

# List ACL tokens
curl -H "X-Nomad-Token: management-token" http://localhost:4646/v1/acl/tokens

# Delete ACL token
curl -X DELETE -H "X-Nomad-Token: management-token" http://localhost:4646/v1/acl/token/<token-id>
```

### ACL Policies
```bash
# Create ACL policy
curl -X POST -d '{
  "Name": "readonly",
  "Description": "Read-only access",
  "Rules": "namespace \"default\" { policy = \"read\" }"
}' http://localhost:4646/v1/acl/policy

# List ACL policies
curl -H "X-Nomad-Token: management-token" http://localhost:4646/v1/acl/policies
```

---

## API Examples

### Python Examples
```python
import requests
import json

# Base configuration
NOMAD_ADDR = "http://localhost:4646"
NOMAD_TOKEN = "your-token"

headers = {
    "X-Nomad-Token": NOMAD_TOKEN,
    "Content-Type": "application/json"
}

# List jobs
response = requests.get(f"{NOMAD_ADDR}/v1/jobs", headers=headers)
jobs = response.json()

# Create job
with open("job.json", "r") as f:
    job_data = json.load(f)

response = requests.post(
    f"{NOMAD_ADDR}/v1/jobs",
    headers=headers,
    json=job_data
)

# Get job status
response = requests.get(f"{NOMAD_ADDR}/v1/job/example", headers=headers)
job = response.json()

# Scale job
scale_data = {"Count": 5}
response = requests.post(
    f"{NOMAD_ADDR}/v1/job/example/scale",
    headers=headers,
    json=scale_data
)
```

### Go Examples
```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "net/http"
)

type Job struct {
    ID   string `json:"ID"`
    Name string `json:"Name"`
    Type string `json:"Type"`
}

func main() {
    client := &http.Client{}
    
    // List jobs
    req, _ := http.NewRequest("GET", "http://localhost:4646/v1/jobs", nil)
    req.Header.Set("X-Nomad-Token", "your-token")
    
    resp, err := client.Do(req)
    if err != nil {
        panic(err)
    }
    defer resp.Body.Close()
    
    var jobs []Job
    json.NewDecoder(resp.Body).Decode(&jobs)
    
    // Create job
    job := Job{
        ID:   "example",
        Name: "example",
        Type: "service",
    }
    
    jobData, _ := json.Marshal(job)
    req, _ = http.NewRequest("POST", "http://localhost:4646/v1/jobs", bytes.NewBuffer(jobData))
    req.Header.Set("X-Nomad-Token", "your-token")
    req.Header.Set("Content-Type", "application/json")
    
    resp, err = client.Do(req)
    if err != nil {
        panic(err)
    }
    defer resp.Body.Close()
}
```

### JavaScript Examples
```javascript
const axios = require('axios');

const NOMAD_ADDR = 'http://localhost:4646';
const NOMAD_TOKEN = 'your-token';

const headers = {
    'X-Nomad-Token': NOMAD_TOKEN,
    'Content-Type': 'application/json'
};

// List jobs
async function listJobs() {
    try {
        const response = await axios.get(`${NOMAD_ADDR}/v1/jobs`, { headers });
        return response.data;
    } catch (error) {
        console.error('Error listing jobs:', error);
    }
}

// Create job
async function createJob(jobData) {
    try {
        const response = await axios.post(`${NOMAD_ADDR}/v1/jobs`, jobData, { headers });
        return response.data;
    } catch (error) {
        console.error('Error creating job:', error);
    }
}

// Scale job
async function scaleJob(jobId, count) {
    try {
        const response = await axios.post(
            `${NOMAD_ADDR}/v1/job/${jobId}/scale`,
            { Count: count },
            { headers }
        );
        return response.data;
    } catch (error) {
        console.error('Error scaling job:', error);
    }
}
```

---

## Advanced CLI Features

### Templates
```bash
# Use Go templates for output formatting
nomad job status -t '{{range .TaskGroups}}{{.Name}}: {{.Queued}}/{{.Running}}/{{.Complete}}{{end}}' example

# Template with functions
nomad alloc list -t '{{range .}}{{.ID | truncate 8}}: {{.Status}}{{end}}'

# Template with conditions
nomad node status -t '{{range .}}{{if eq .Status "ready"}}{{.ID}}: {{.Status}}{{end}}{{end}}'
```

### JSON Output
```bash
# Get JSON output for all commands
nomad job status -json example
nomad alloc list -json
nomad node status -json
```

### Watch Mode
```bash
# Watch job status
watch nomad job status example

# Watch allocations
watch nomad alloc list -job=example
```

### Batch Operations
```bash
# Stop multiple jobs
for job in job1 job2 job3; do
    nomad job stop $job
done

# Scale multiple jobs
for job in web api cache; do
    nomad job scale $job 3
done
```

---

## Troubleshooting

### Common Issues

#### Connection Issues
```bash
# Check if Nomad is running
curl http://localhost:4646/v1/agent/self

# Check server members
nomad server members

# Check node status
nomad node status
```

#### Authentication Issues
```bash
# Check token validity
curl -H "X-Nomad-Token: your-token" http://localhost:4646/v1/jobs

# List ACL tokens
nomad acl token list
```

#### Job Issues
```bash
# Check job status
nomad job status example

# Check allocation status
nomad alloc list -job=example

# Check allocation logs
nomad alloc logs <alloc-id>
```

### Debug Commands
```bash
# Enable debug logging
NOMAD_LOG_LEVEL=DEBUG nomad agent -dev

# Get detailed job information
nomad job inspect example

# Get allocation details
nomad alloc status <alloc-id>
```

### Performance Monitoring
```bash
# Get agent metrics
curl http://localhost:4646/v1/agent/self

# Get cluster metrics
curl http://localhost:4646/v1/operator/scheduler/configuration
```

---

## Best Practices

### CLI Best Practices
1. **Use Templates**: Format output for better readability
2. **Use JSON**: For programmatic access and automation
3. **Use Variables**: For dynamic job configurations
4. **Use Namespaces**: For multi-tenant environments
5. **Use ACLs**: For security and access control

### API Best Practices
1. **Use Pagination**: For large datasets
2. **Use Filtering**: To reduce data transfer
3. **Use Caching**: For frequently accessed data
4. **Use Error Handling**: For robust applications
5. **Use Rate Limiting**: To avoid overwhelming the API

### Security Best Practices
1. **Use TLS**: For all communications
2. **Use ACLs**: For access control
3. **Use Tokens**: For authentication
4. **Use Namespaces**: For isolation
5. **Use Audit Logging**: For compliance

---

## Next Steps

After mastering Nomad CLI and API, you'll be ready to:

1. **Explore Job Types**: Understand different job types and their use cases
2. **Advanced Features**: Learn about advanced scheduling and features
3. **Production Setup**: Deploy Nomad in production environments
4. **Integration**: Integrate with Consul and Vault

### Practice Projects

1. **CLI Automation**: Create scripts for common operations
2. **API Integration**: Build applications that use Nomad API
3. **Monitoring**: Create monitoring dashboards
4. **CI/CD**: Integrate Nomad into CI/CD pipelines

### Resources

- [Nomad API Documentation](https://www.nomadproject.io/api-docs/)
- [Nomad CLI Documentation](https://www.nomadproject.io/docs/commands/)
- [Nomad GitHub Repository](https://github.com/hashicorp/nomad)
- [Nomad Community](https://discuss.hashicorp.com/c/nomad) 