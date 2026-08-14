# Homelab Workstation

A self-hosted AI and development workstation built around **Docker, GPU-accelerated workloads, remote access, and infrastructure monitoring**.

This project serves as both my personal homelab and a hands-on infrastructure/DevOps project. It hosts local LLM inference, image generation, development environments, machine learning workloads, and observability tools while keeping persistent application data and AI models on dedicated storage.

---

## Overview

The workstation provides a centralized environment for:

* Local LLM inference with `llama.cpp`
* Image generation and editing with ComfyUI
* Web-based AI access through Open WebUI
* Remote development through code-server
* JupyterLab-based Python and ML development
* GPU-accelerated machine learning with PyTorch
* Infrastructure monitoring with Prometheus and Grafana
* Docker container monitoring with cAdvisor
* NVIDIA GPU monitoring with DCGM Exporter
* Secure remote access through Cloudflare Tunnel
* Persistent storage on a dedicated storage drive

---

## Architecture

```text
                                  Internet
                                     │
                                     ▼
                                Cloudflare
                                     │
                                     ▼
                            Cloudflare Tunnel
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │     Docker Network    │
                         │                       │
                         │  ┌───────────────┐    │
                         │  │   Open WebUI  │    │
                         │  └───────┬───────┘    │
                         │          │            │
                         │  ┌───────▼───────┐    │
                         │  │    llama.cpp  │    │
                         │  └───────────────┘    │
                         │                       │
                         │  ┌───────────────┐    │
                         │  │    ComfyUI    │    │
                         │  └───────────────┘    │
                         │                       │
                         │  ┌───────────────┐    │
                         │  │  code-server  │    │
                         │  └───────────────┘    │
                         │                       │
                         │  ┌───────────────┐    │
                         │  │    Grafana    │◄───┼───┐
                         │  └───────────────┘    │   │
                         │                       │   │
                         │  ┌───────────────┐    │   │
                         │  │  Prometheus   │────┼───┘
                         │  └───────┬───────┘    │
                         │          │            │
                         │     ┌────┴─────┐      │
                         │     │          │      │
                         │     ▼          ▼      │
                         │ Node       cAdvisor  │
                         │ Exporter             │
                         │                      │
                         │  DCGM Exporter       │
                         │        │             │
                         └────────┼─────────────┘
                                  │
                                  ▼
                             NVIDIA GPU
```

---

## Hardware

| Component             | Specification                  |
| --------------------- | ------------------------------ |
| **CPU**               | AMD Ryzen 9 3900x              |
| **GPU**               | NVIDIA RTX 3070                |
| **RAM**               | 64 GB DDR4                     |
| **OS**                | Ubuntu 24.04 LTS               |
| **Storage**           | NVMe + dedicated storage drive |
| **Container Runtime** | Docker                         |

The dedicated storage drive is mounted at `/mnt/s` and is used for large AI models, caches, generated files, and monitoring data rather than filling the primary NVMe drive.

---

# Services

## AI

### llama.cpp

GPU-accelerated local LLM inference using GGUF models.

Models are stored separately from the container filesystem:

```text
/mnt/s/raj-storage/models/gguf/
```

This allows models to persist independently of the container lifecycle and makes it easy to swap models without rebuilding the stack.

### Open WebUI

Provides a browser-based interface for interacting with locally hosted models.

Open WebUI connects to the local `llama.cpp` server and supports multimodal model usage where supported.

### ComfyUI

GPU-accelerated image generation and image editing.

The environment supports FLUX.2 Klein models, including GGUF-based components, and can be integrated with Open WebUI for image-generation workflows.

---

## Development

### code-server

Provides a browser-accessible VS Code environment for development directly on the workstation.

This makes it possible to develop remotely without requiring access to the workstation's physical desktop environment.

### JupyterLab

Provides a browser-based Python environment for machine learning, experimentation, and data analysis.

Python virtual environments are used for individual projects, including GPU-enabled PyTorch environments.

---

# Monitoring & Observability

The workstation includes a complete Prometheus/Grafana monitoring stack capable of monitoring the underlying Linux host, Docker containers, and NVIDIA GPU.

## Prometheus

Prometheus collects time-series metrics from the workstation and containerized services.

## Grafana

Grafana provides dashboards for system, storage, network, container, and GPU performance.

Current dashboard metrics include:

* CPU utilization
* CPU frequency
* Memory utilization
* System load
* GPU utilization
* GPU temperature
* GPU VRAM usage
* GPU power consumption
* Disk utilization
* Disk read/write throughput
* Disk I/O operations
* Network receive/transmit traffic
* Network errors
* Server uptime

## Node Exporter

Collects Linux host-level metrics including:

* CPU
* Memory
* Filesystems
* Disk activity
* Network statistics
* System load

## cAdvisor

Collects Docker container resource usage including:

* CPU utilization
* Memory usage
* Network traffic
* Container activity

## NVIDIA DCGM Exporter

Exposes NVIDIA GPU telemetry to Prometheus, including:

* GPU utilization
* VRAM utilization
* GPU temperature
* Power consumption
* GPU-specific hardware metrics

---

# Monitoring Architecture

```text
                         Linux Host
                             │
                ┌────────────┼────────────┐
                │            │            │
                ▼            ▼            ▼
          Node Exporter   cAdvisor    DCGM Exporter
                │            │            │
                └────────────┼────────────┘
                             │
                             ▼
                         Prometheus
                             │
                             ▼
                          Grafana
```

This provides a centralized view of both the underlying host and the containerized workloads running on it.

---

# Remote Access

Remote access is provided through **Cloudflare Tunnel** rather than directly forwarding ports from the router.

```text
                         Internet
                            │
                            ▼
                        Cloudflare
                            │
                            ▼
                    Cloudflare Tunnel
                            │
                            ▼
                        cloudflared
                            │
                            ▼
                     Docker Service
```

For example:

```text
grafana.example.com
        │
        ▼
    Cloudflare
        │
        ▼
   Cloudflare Tunnel
        │
        ▼
     Grafana
```

This allows services to remain behind the local network while still being accessible remotely without exposing individual service ports directly to the public internet.

---

# Storage

Persistent application data is stored outside the containers on a dedicated storage drive.

```text
/mnt/s/raj-storage/
├── datasets/
│
├── models/
│   └── gguf/
│
├── cache/
│
├── outputs/
│
└── monitoring/
    ├── grafana/
    └── prometheus-data/
```

Separating persistent data from container filesystems provides:

* Persistent model storage
* Easier container replacement
* Reduced NVMe usage
* Centralized management of large files
* Easier backups
* Independent data lifecycle from containers

---

# Repository Structure

```text
homelab-workstation/
│
├── docker-compose/
│   ├── ...
│   └── ...
│
├── grafana-dashboard/
│   └── ...
│
└── README.md
```

The Compose files in this repository are sanitized copies of the stacks running on the workstation.

**Secrets, credentials, API tokens, and machine-specific configuration are intentionally excluded.**

---

# Goals

The primary goals of this project are to:

1. Run AI workloads locally using consumer GPU hardware.
2. Build a reproducible containerized environment.
3. Separate persistent data from container lifecycles.
4. Access development and AI services remotely.
5. Monitor system, container, and GPU performance.
6. Gain practical experience with Linux, Docker, networking, GPU infrastructure, and observability.

---

# Future Improvements

Planned improvements include:

* Grafana alerting
* AI workload-specific dashboards
* llama.cpp inference metrics
* Container health monitoring
* Automated backups
* Improved service health checks
* Additional GPU and inference performance metrics
* Infrastructure documentation and diagrams
* Automated deployment and update workflows

---

# Technologies

### Infrastructure

* Docker
* Docker Compose
* Linux / Ubuntu
* Cloudflare Tunnel

### AI / Machine Learning

* llama.cpp
* GGUF
* PyTorch
* ComfyUI
* FLUX.2
* Open WebUI

### Monitoring & Observability

* Grafana
* Prometheus
* Node Exporter
* cAdvisor
* NVIDIA DCGM Exporter

### Development

* VS Code / code-server
* JupyterLab
* Python
* Git

---

# Why I Built This

Rather than relying entirely on cloud services for AI experimentation and development, I wanted to build and operate my own GPU-accelerated workstation.

The project has evolved into a practical environment for experimenting with:

* Local LLM inference
* Image generation
* Machine learning
* Containerization
* Linux infrastructure
* Networking
* Remote development
* Infrastructure monitoring

It also provides a way to observe the performance characteristics of different AI workloads and understand how **CPU, GPU, memory, storage, and networking interact under load**.

Ultimately, this project is less about running individual applications and more about building and operating a small-scale production-style infrastructure environment from the ground up.
