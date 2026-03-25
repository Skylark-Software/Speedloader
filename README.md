<p align="center">
  <img src="docs/speedloader-logo.svg" alt="Speedloader" width="600">
</p>

# Speedloader

Fast hybrid RAM/storage management for LLM models.

## Overview

Speedloader creates a multi-tier storage system that keeps frequently-used LLM models in RAM for instant access (~10-15 seconds) while storing others on cold storage (local disk, NFS, BeeGFS). Models promote and demote between tiers with one click — or automatically based on placement strategy.

**Features:**
- Written in Rust — single binary (~5MB) with no dependencies
- Web GUI with real-time model and tier management
- Multi-tier architecture: local RAM, remote RAM (NVMe-oF over RDMA), cold storage
- Automatic tier selection with fastest-fit, fill-first, or round-robin placement
- Model pinning to specific tiers
- One-click remote tier lifecycle (connect/disconnect)
- Async background transfers with progress indication
- Cold storage is read-only — never modified by Speedloader
- Support for Ollama, llama.cpp, vLLM, and HuggingFace backends

## Quick Start

```bash
# Clone the repo
git clone https://github.com/Skylark-Software/Speedloader.git
cd Speedloader

# Install binary
sudo cp bin/speedloader /usr/local/bin/
sudo chmod +x /usr/local/bin/speedloader

# Start the web GUI
speedloader serve

# Open http://localhost:5050 — configure via Settings tab on first run
```

Or use the CLI installer for automated setup:

```bash
sudo speedloader install
```

## Web GUI

### Models Tab

View hot and cold model status. Promote models to RAM with one click — Speedloader selects the fastest tier with available space, or uses a pinned tier assignment. Large transfers run in the background with progress updates.

![Models Tab](docs/screenshot-models.png)

### Storage Tiers Tab

Manage local and remote RAM pools with unified lifecycle controls. Each pool can be connected, disconnected, or resized independently. Remote tiers connect via NVMe-oF over RDMA with one-click bring-up.

![Storage Tiers](docs/screenshot-settings.png)

**Local Pools:**
- tmpfs-backed RAM tiers with configurable size
- Connect/Disconnect/Resize from the web UI
- Multiple pools supported (different sizes, speed ratings)

**Remote Pools (NVMe-oF over RDMA):**
- One-click Connect: creates ramdisk → exports NVMe-oF → connects locally
- One-click Disconnect: auto-demotes models → unmounts → tears down remote
- Progress indicators during multi-step operations

**Model Placement:**
- **Fastest Fit**: auto-failover to next fastest tier with space
- **Fill First**: fill fastest tier before using next
- **Round Robin**: distribute across tiers
- **Pinned Models**: hard-assign specific models to specific tiers

## Architecture

```
Models Dir (persistent)          Storage Pools (RAM)
────────────────────             ─────────────────────
/var/lib/speedloader/models/     /mnt/pools/local/        (local tmpfs)
  manifests/                     /mnt/nvme-meadowlark/    (remote NVMe-oF)
  blobs/
    sha256-abc → cold storage    Cold Storage (read-only)
    sha256-def → local pool      ─────────────────────
    sha256-ghi → remote pool     /mnt/beegfs/ollama/models/
```

- **Models dir** is persistent — survives pool connect/disconnect cycles
- **Pools** are independent mount points that can be created/destroyed/resized
- **Cold storage** is never written to by Speedloader
- **Promote** = copy blob to pool + update symlink in models dir
- **Demote** = delete from pool + repoint symlink to cold storage

## CLI Commands

| Command | Description |
|---------|-------------|
| `serve` | Start web GUI (auto-detects config or starts in setup mode) |
| `status` | Show hot/cold model status |
| `promote <model>` | Move model to fastest available RAM tier |
| `demote <model>` | Move model back to cold storage |
| `install` | Interactive CLI installer |
| `init` | Initialize RAM disk |
| `config` | Show current configuration |
| `health` | Health check and auto-repair |

## Docker

```bash
docker compose up -d
```

## Supported Backends

| Backend | Storage Format | Default API Port |
|---------|---------------|------------------|
| `ollama` | Blobs + Manifests | 11434 |
| `llamacpp` | Single GGUF files | 8080 |
| `vllm` | HuggingFace directories | 8000 |
| `huggingface` | Safetensors directories | N/A |

## Configuration

Configuration is stored at `/etc/speedloader/config.json` or `~/.config/speedloader/config.json`.

On first run, `speedloader serve` starts with defaults and shows a setup banner directing you to the Settings tab.

```json
{
  "modelsDir": "/var/lib/speedloader/models",
  "ramdisk": {
    "path": "/mnt/ollama-ramdisk",
    "size": "200G"
  },
  "storage": {
    "path": "/mnt/beegfs/ollama/models",
    "type": "beegfs"
  },
  "backend": {
    "type": "ollama",
    "apiUrl": "http://localhost:11434"
  },
  "remoteTiers": {
    "hosts": {
      "meadowlark": {
        "ssh": "service@meadowlark.example.com",
        "rdmaIp": "10.0.40.2",
        "ramdiskSize": "128G"
      }
    }
  },
  "modelPlacement": {
    "defaultStrategy": "fastest-fit",
    "pinned": {
      "llama3.2-vision:90b": "meadowlark"
    }
  }
}
```

## System Requirements

- Linux (x86_64 or ARM64)
- Systemd (for service management)
- Root access (for RAM disk mounting)
- Optional: InfiniBand/RDMA for remote tiers

## License

Copyright 2025-2026 Skylark Software LLC. All Rights Reserved.

This is proprietary software. Unauthorized copying, modification, or distribution is prohibited.

---

<p align="center">
  <img src="docs/skylark-software-logo.svg" alt="Skylark Software" width="400">
</p>
