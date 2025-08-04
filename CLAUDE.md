# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a homelab infrastructure repository that uses Docker Compose to orchestrate various self-hosted services for media management, file sharing, and local AI/LLM capabilities.

## Key Services and Architecture

### Media Management Stack (*arr Suite)
- **Prowlarr** (port 9696): Indexer manager that integrates with other *arr apps
- **Sonarr** (port 8989): TV show management and download automation
- **Radarr** (port 7878): Movie management and download automation
- **Readarr** (port 8787): eBook management (using develop branch)
- **Lidarr** (port 8686): Music management (currently commented out)

### Media Streaming
- **Plex** (port 32400): Media server with NVIDIA GPU hardware acceleration support for transcoding

### Download Management
- **Transmission-VPN**: BitTorrent client with OpenVPN integration using Surfshark VPN
- **FlareSolverr** (port 8191): Cloudflare bypass solution for indexers

### AI/LLM Services (in llm/ subdirectory)
- **Ollama** (port 11434): Local LLM runtime with NVIDIA GPU support
- **OpenWebUI** (port 11435): Web interface for Ollama

### Search Services (in searxng/ subdirectory)
- **SearxNG** (port 9001): Privacy-respecting metasearch engine
- **Redis**: Cache backend for SearxNG

## Common Development Commands

### Docker Compose Operations

The repository now uses a modular structure with grouped services:

```bash
# Start ALL services at once (from root directory)
docker-compose up -d

# Stop ALL services
docker-compose down

# Start specific service groups
cd media && docker-compose up -d     # All media services (Plex, *arr, transmission)
cd llm && docker-compose up -d       # AI/LLM services (Ollama, OpenWebUI)
cd searxng && docker-compose up -d   # Search services (SearxNG, Redis)

# View logs for all services
docker-compose logs -f

# View logs for specific service group
cd media && docker-compose logs -f
cd llm && docker-compose logs -f

# Restart a specific service (from root)
docker-compose restart [service_name]

# Pull latest images and recreate all services
docker-compose pull && docker-compose up -d
```

### Service Groups Structure
- **media/**: Plex, Transmission-VPN, Prowlarr, Sonarr, Radarr, Readarr, Lidarr, FlareSolverr
- **llm/**: Ollama, OpenWebUI  
- **searxng/**: SearxNG, Redis

### Service Health Checks
```bash
# Check if all containers are running
docker ps

# Check Plex health
curl -f http://localhost:32400/web

# Check *arr services
curl http://localhost:9696  # Prowlarr
curl http://localhost:8989  # Sonarr
curl http://localhost:7878  # Radarr
curl http://localhost:8787  # Readarr
```

## Directory Structure and Volume Mappings

### Media Storage
- `/media/downloads/completed`: Completed downloads from Transmission
- `/media/downloads/incomplete`: In-progress downloads
- `/media/tv`: TV shows library (mounted to Sonarr and Plex)
- `/media/movies`: Movies library (mounted to Radarr and Plex)
- `/media/books`: eBooks library (mounted to Readarr)
- `/media/music`: Music library (for future Lidarr use)

### Configuration Storage
- `./[service]/config`: Persistent configuration for each service
- `./transmission/config`: VPN and Transmission configuration
- `./plex/config`: Plex server configuration and metadata

## Environment Variables

The main docker-compose.yml expects these environment variables (stored in .env):
- `SURFSHARK_UNAME`: Surfshark VPN username
- `SURFSHARK_PASSWORD`: Surfshark VPN password

## GPU Configuration

Both Plex and Ollama are configured to use NVIDIA GPU:
- Plex: Hardware-accelerated transcoding
- Ollama: GPU-accelerated LLM inference

Ensure NVIDIA Container Toolkit is installed and configured on the host system.

## Network Configuration

- Most services use bridge networking with exposed ports
- Plex uses host networking for better performance and discovery
- Transmission-VPN has NET_ADMIN capability for VPN tunnel creation
- Local network is configured as 192.168.0.0/16 for VPN bypass

## Important Notes

- All services use PUID=1000 and PGID=1000 for file permissions consistency
- Services are configured with `restart: unless-stopped` for automatic recovery
- Docker Compose files do NOT use version tags (deprecated in modern Compose)
- The maybe-finance service appears to be added but not yet configured
- Monitoring stack is planned but not yet implemented