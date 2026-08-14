# Homelab Workstation

A self-hosted AI and development workstation built around Docker, GPU-accelerated workloads, remote access, and infrastructure monitoring.

This project serves as both my personal homelab and a hands-on infrastructure/DevOps project. It hosts LLM inference, image generation, development environments, and monitoring services while keeping persistent application data and AI models on dedicated storage.

## Overview

The workstation is designed to provide a centralized environment for:

- Local LLM inference with `llama.cpp`
- Image generation and editing with ComfyUI
- Web-based AI access through Open WebUI
- Remote development through code-server
- Jupyter-based Python/ML development
- GPU-accelerated machine learning with PyTorch
- Infrastructure monitoring with Prometheus and Grafana
- Docker container monitoring with cAdvisor
- NVIDIA GPU monitoring with DCGM Exporter
- Secure remote access through Cloudflare Tunnel
- Persistent storage on a dedicated storage drive

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
                 ┌────────────────┴────────────────┐
                 │      Docker Network             │
                 │                                 │
                 │  ┌─────────────┐                │
                 │  │  Open WebUI │                │
                 │  └──────┬──────┘                │
                 │         │                       │
                 │  ┌──────▼──────┐                │
                 │  │   llama.cpp │                │
                 │  └─────────────┘                │
                 │                                 │
                 │  ┌─────────────┐                │
                 │  │   ComfyUI   │                │
                 │  └─────────────┘                │
                 │                                 │
                 │  ┌─────────────┐                │
                 │  │ code-server │                │
                 │  └─────────────┘                │
                 │                                 │
                 │  ┌─────────────┐                │
                 │  │   Grafana   │◄──────┐       │
                 │  └─────────────┘       │       │
                 │                        │       │
                 │  ┌─────────────┐       │       │
                 │  │ Prometheus  │───────┘       │
                 │  └──────┬──────┘               │
                 │         │                       │
                 │    ┌────┴─────┐                 │
                 │    │          │                 │
                 │    ▼          ▼                 │
                 │ Node       cAdvisor              │
                 │ Exporter                         │
                 │                                 │
                 │         DCGM Exporter           │
                 │              │                  │
                 └──────────────┼──────────────────┘
                                │
                                ▼
                           NVIDIA GPU
