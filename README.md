# Speedloader

Fast hybrid RAM/storage management for LLM models.

## Overview

Speedloader creates a hybrid storage tier that keeps frequently-used LLM models in RAM for instant access (~10-15 seconds) while storing others on cold storage.

**Features:**
- Single statically-linked binary (~10MB) with no dependencies
- Web GUI for easy model management
- Multi-tier RAM storage with speed-based placement
- Remote RAM pooling via NVMe-oF over RDMA
- Support for Ollama, llama.cpp, vLLM, and HuggingFace backends

## Download

Download the latest binary from [Releases](https://github.com/Skylark-Software/Speedloader/releases).

## Installation

```bash
# Download and install binary
sudo cp speedloader /usr/local/bin/
sudo chmod +x /usr/local/bin/speedloader

# Run interactive installer
sudo speedloader install
```

The installer will:
1. Detect your system (RAM, storage mounts, Ollama)
2. Prompt for configuration
3. Generate `/etc/speedloader/config.json`
4. Install systemd services
5. Start the services

## Usage

```bash
# Show model status
speedloader status

# Promote a model to RAM
sudo speedloader promote mixtral:8x7b

# Demote a model from RAM
sudo speedloader demote llama3:70b

# Start web GUI
speedloader serve --port 5050

# Show configuration
speedloader config

# Run health check
speedloader health
```

## Web GUI

Access the web dashboard at **http://localhost:5050** after installation.

### Models Tab
View hot/cold model status with RAM tier usage. Promote models to RAM or demote them to storage with one click.

![Models Tab](docs/screenshot-models.png)

### Storage Tiers Tab
Manage multiple RAM storage tiers - local DDR5, remote NVMe-oF over RDMA, and more. Configure model placement strategies and pin specific models to tiers.

![Storage Tiers Tab](docs/screenshot-tiers.png)

**Features:**
- **Multiple RAM Tiers**: Local tmpfs + remote NVMe-oF RAM pools
- **Speed Ratings**: DDR5-5600, DDR4-3200, NVMe-oF, etc.
- **Placement Strategies**: Fastest-fit (auto-failover), fill-first, round-robin
- **Pinned Models**: Hard-assign models to specific tiers

### Settings Tab
Configure backend, storage paths, and RAM disk settings.

![Settings Tab](docs/screenshot-settings.png)

## Commands

| Command | Description |
|---------|-------------|
| `status` | Show hot/cold model status |
| `promote <model>` | Move model to RAM |
| `demote <model>` | Move model to storage |
| `serve` | Start web GUI server |
| `install` | Interactive installation |
| `init` | Initialize RAM disk |
| `config` | Show configuration |
| `health` | Health check and auto-repair |

## Supported Backends

| Backend | Storage Format | Default API Port |
|---------|---------------|------------------|
| `ollama` | Blobs + Manifests | 11434 |
| `llamacpp` | Single GGUF files | 8080 |
| `vllm` | HuggingFace directories | 8000 |
| `huggingface` | Safetensors directories | N/A |

## Configuration

Configuration is stored at `/etc/speedloader/config.json`. See the [documentation](docs/) for full configuration options.

## System Requirements

- Linux (x86_64 or ARM64)
- Systemd (for service management)
- Root access (for RAM disk mounting)

## License

Copyright 2025-2026 Skylark Software LLC. All Rights Reserved.

This is proprietary software. Unauthorized copying, modification, or distribution is prohibited.
