# Self-Hosted AI Stack

> A self-hosted AI infrastructure stack combining Ollama, Docker, and Prometheus/Grafana for local LLM inference and observability.

---

## 🎯 Overview

This repository defines the complete AI runtime environment of the sovereign lab. Every component runs locally on bare-metal hardware — no API keys, no cloud billing, no data leaving the perimeter.

**Stack:** Ollama (LLM runtime) + Docker Compose + Prometheus/Grafana + n8n automation + Portainer management

**Host:** Arch-GPU node or Dell-Gateway (flexible deployment)

**Status:** ✅ Operational — All services running and monitored

---

## 🧩 Stack Components

| Component | Role | Technology | Port |
|-----------|------|-----------|------|
| **LLM Runtime** | Local model inference | Ollama | 11434 |
| **AI Model** | 27B parameter LLM | Qwen 3.5:27B | - |
| **GPU Backend** | Multi-GPU VRAM pooling | 4× NVIDIA P106-100 / CUDA | - |
| **Monitoring** | Real-time dashboards | Grafana | 3000 |
| **Metrics** | Telemetry collection | Prometheus | 9090 |
| **Workflows** | Autonomous automation | n8n | 5678 |
| **Container Mgmt** | Docker UI | Portainer | 9443 |
| **Orchestration** | Service management | Docker Compose | - |

---

## 🏗️ Architecture

```
[Bare-Metal Host (Arch Linux / Dell Node)]
         │
    [Docker Engine]
         │
    ┌────┴────────────────────────────┐
    │                                 │
[Ollama Container]            [Monitoring Stack]
  ├── GPU 0: P106-100             ├── Grafana :3000
  ├── GPU 1: P106-100             ├── Prometheus :9090
  ├── GPU 2: P106-100             └── Node Exporter
  └── GPU 3: P106-100
         │
    [n8n Automation]
         │
   [Autonomous Workflows]
```

---

## 📦 Services & Deployment

### Ollama (LLM Runtime)

**Purpose:** Local large language model inference with multi-GPU acceleration

**Configuration:**
```yaml
Container: ollama/ollama:latest
GPUs: all 4× P106-100 cards
Model: Qwen 3.5:27B
VRAM Pool: 24 GB total
Port: 11434
```

**Health Check:**
```bash
curl http://localhost:11434/api/tags
```

### Prometheus (Metrics Collection)

**Purpose:** Collect system and application metrics across all nodes

**Configuration:**
```yaml
Container: prom/prometheus:latest
Port: 9090
Scrape Interval: 15s
Retention: 30d
Targets: node-exporter, ollama, n8n
```

### Grafana (Monitoring Dashboards)

**Purpose:** Real-time visualization of system health, GPU metrics, and inference performance

**Configuration:**
```yaml
Container: grafana/grafana:latest
Port: 3000
Data Source: Prometheus :9090
Dashboards: GPU cluster, inference latency, system health
```

**Default Credentials:**
- Admin: `admin` / `admin` (change on first login)

### n8n (Workflow Automation)

**Purpose:** Orchestrate complex workflows, trigger security scans, manage infrastructure

**Configuration:**
```yaml
Container: n8n:latest
Port: 5678
Workflows: autonomous-executor, security-ops, infrastructure-manager
Execution: background jobs + webhooks
```

### Portainer (Docker Management)

**Purpose:** Visual management of containers, images, volumes

**Configuration:**
```yaml
Container: portainer/portainer-ce:latest
Port: 9443
Features: container management, logs, resource monitoring
```

---

## 🚀 Deployment

### Prerequisites

```bash
# System requirements
- Docker Engine 20.10+
- Docker Compose 2.0+
- 62+ GB RAM (for multi-GPU + monitoring)
- 24 GB GPU VRAM available
```

### Quick Start

```bash
# Clone repository
git clone https://github.com/Dinaverse/local-ai-sovereign-stack.git
cd local-ai-sovereign-stack

# Configure environment
cp .env.example .env
nano .env  # Edit GPU settings, model, etc.

# Deploy stack
docker-compose up -d

# Verify services
docker-compose ps
```

### Environment Configuration (`.env`)

```bash
# Ollama Configuration
OLLAMA_MODEL=qwen2.5:27b
OLLAMA_GPU_COUNT=4
OLLAMA_NUM_GPU=4
CUDA_VISIBLE_DEVICES=0,1,2,3

# Prometheus Configuration
PROMETHEUS_RETENTION=30d
PROMETHEUS_SCRAPE_INTERVAL=15s

# Grafana Configuration
GF_SECURITY_ADMIN_PASSWORD=secure_password
GF_INSTALL_PLUGINS=grafana-piechart-panel

# n8n Configuration
N8N_SECURE_COOKIE=true
N8N_ENCRYPTION_KEY=your-secure-key

# Network Configuration
DOCKER_NETWORK=sovereign-net
HOST_IP=192.168.1.100
```

### Compose File Structure

```yaml
version: '3.8'

networks:
  sovereign-net:
    driver: bridge

volumes:
  prometheus_data:
  grafana_data:
  ollama_models:
  n8n_data:

services:
  ollama:
    image: ollama/ollama:latest
    gpus: all
    ports:
      - "11434:11434"
    environment:
      - OLLAMA_NUM_GPU=${OLLAMA_NUM_GPU}
      - CUDA_VISIBLE_DEVICES=${CUDA_VISIBLE_DEVICES}
    volumes:
      - ollama_models:/root/.ollama
    networks:
      - sovereign-net

  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    networks:
      - sovereign-net

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GF_SECURITY_ADMIN_PASSWORD}
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - sovereign-net

  n8n:
    image: n8n:latest
    ports:
      - "5678:5678"
    environment:
      - N8N_SECURE_COOKIE=${N8N_SECURE_COOKIE}
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
    volumes:
      - n8n_data:/home/node/.n8n
    networks:
      - sovereign-net

  portainer:
    image: portainer/portainer-ce:latest
    ports:
      - "9443:9443"
      - "8000:8000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - sovereign-net
```

---

## 📊 Monitoring & Dashboards

### Access Points

| Service | URL | Purpose |
|---------|-----|---------|
| **Ollama** | `http://localhost:11434` | LLM API endpoint |
| **Prometheus** | `http://localhost:9090` | Metrics database |
| **Grafana** | `http://localhost:3000` | Monitoring dashboards |
| **n8n** | `http://localhost:5678` | Workflow builder |
| **Portainer** | `https://localhost:9443` | Docker management |

### Grafana Dashboards

Pre-configured dashboards:

1. **GPU Cluster Overview** — GPU utilization, VRAM usage, temperature
2. **Inference Performance** — Model latency, throughput, requests/sec
3. **System Health** — CPU, memory, disk, network across all nodes
4. **Container Status** — Running services, resource consumption
5. **n8n Workflows** — Execution status, success rate, errors

---

## 🔧 Common Operations

### Check Ollama Status

```bash
# List loaded models
curl http://localhost:11434/api/tags

# Generate response (test inference)
curl -X POST http://localhost:11434/api/generate \
  -d '{"model": "qwen2.5:27b", "prompt": "What is AI?"}'
```

### View Metrics in Prometheus

```bash
# Access Prometheus UI
# Navigate to: http://localhost:9090

# Query examples:
# GPU utilization: nvidia_smi_utilization_gpu
# Model latency: ollama_request_duration_seconds
# Memory usage: container_memory_usage_bytes
```

### Manage Workflows in n8n

```bash
# Access n8n UI
# Navigate to: http://localhost:5678

# Create workflows for:
# - Daily security scans
# - Infrastructure health checks
# - Model performance logging
# - Incident response automation
```

### Scale GPU Resources

```bash
# Modify .env
OLLAMA_NUM_GPU=4
CUDA_VISIBLE_DEVICES=0,1,2,3

# Restart Ollama service
docker-compose restart ollama
```

---

## 🛡️ Security

- **No Cloud Dependencies** — All data stays local
- **Network Isolation** — Internal Docker network only
- **Encrypted Communication** — TLS/SSL for external access
- **Access Control** — Authentication required for all services
- **Audit Logging** — All operations logged to Prometheus/Grafana

---

## 🔗 Related Repositories

| Repository | Purpose |
|------------|---------|
| [sovereign-ai-infrastructure](https://github.com/Dinaverse/sovereign-ai-infrastructure) | Architecture documentation |
| [arch-linux-multi-gpu-llm](https://github.com/Dinaverse/arch-linux-multi-gpu-llm) | GPU cluster setup guide |
| [n8n-automation-hub](https://github.com/Dinaverse/n8n-automation-hub) | Workflow definitions |
| [sovereign-lab-orchestration](https://github.com/Dinaverse/sovereign-lab-orchestration) | IaC & orchestration |

---

## 📈 Performance Optimization

### GPU Optimization

```bash
# Enable persistence mode (reduces GPU init overhead)
nvidia-smi -pm 1

# Check current GPU allocation
nvidia-smi

# Monitor inference metrics
watch -n 1 nvidia-smi
```

### Memory Tuning

```bash
# Increase Docker memory limits if needed
# In docker-compose.yml:
mem_limit: 32g
memswap_limit: 32g
```

---

## 🚨 Troubleshooting

### Ollama Not Detecting GPUs

```bash
# Check CUDA availability
docker exec ollama nvidia-smi

# Verify GPU runtime
docker run --rm --gpus all nvidia/cuda:11.8.0-runtime-ubuntu22.04 nvidia-smi

# Restart with explicit GPU configuration
docker-compose down
CUDA_VISIBLE_DEVICES=0,1,2,3 docker-compose up ollama -d
```

### High Memory Usage

```bash
# Check container resource usage
docker stats

# Limit Ollama memory
# Update docker-compose.yml with memory limits
# Restart: docker-compose restart ollama
```

### Prometheus Disk Space

```bash
# Clean up old metrics
docker exec prometheus promtool query instant 'up'

# Adjust retention policy in .env
PROMETHEUS_RETENTION=14d  # Reduce if needed
```

---

## ✅ Health Checks

```bash
# All services status
docker-compose ps

# Ollama readiness
curl -s http://localhost:11434/api/tags | jq '.models'

# Prometheus scrape targets
curl -s http://localhost:9090/api/v1/targets | jq '.data.activeTargets'

# Grafana datasources
curl -s http://localhost:3000/api/datasources

# n8n execution status
curl -s http://localhost:5678/api/v1/executions
```

---

*Sovereign AI. Local. Offline. Yours.*
