# Docker Setup Guide - Open WebUI & N8N RAG Agent

## Prerequisites

### Docker Installation
- **Download Docker Desktop**: [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
- **If you encounter errors, run**: `wsl --update`
- **Important**: If Docker engine is not running, please restart your computer

### Verify Docker Installation
```bash
docker --version
```

---

## Open WebUI Setup

### About Open WebUI
- **GitHub Repository**: [https://github.com/open-webui/open-webui](https://github.com/open-webui/open-webui)
- **Web Interface**: Accessible at `http://localhost:3000` after setup

### Installation Command
```bash
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

---

## N8N Setup

### Documentation
- **Official N8N Docker Docs**: [https://docs.n8n.io/hosting/installation/docker/#prerequisites](https://docs.n8n.io/hosting/installation/docker/#prerequisites)

### Step 1: Create N8N Data Volume
```bash
docker volume create n8n_data
```

### Step 2: Run N8N Container
```bash
docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

**Access N8N**: `http://localhost:5678`

---

## N8N RAG Agent Setup

### Step 1: Create Local Network
Create a dedicated network for all services to communicate:
```bash
docker network create srm
```

### Step 2: Create PostgreSQL Database with Vector Support
```bash
docker run --name pgvector-container -e POSTGRES_USER=myuser -e POSTGRES_PASSWORD=mypassword -e POSTGRES_DB=mydatabase -p 5432:5432 -d ankane/pgvector
```

### Step 3: Connect Services to Network
Connect both N8N and PostgreSQL containers to the shared network:

```bash
# Connect N8N container to network
docker network connect srm {your_n8n_container_name}

# Connect PostgreSQL container to network
docker network connect srm {your_postgres_container_name}
```

**Note**: Replace `{your_n8n_container_name}` and `{your_postgres_container_name}` with the actual container names from your setup.

---

## Container Management Commands

### Useful Docker Commands
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Stop a container
docker stop {container_name}

# Start a container
docker start {container_name}

# Remove a container
docker rm {container_name}

# View container logs
docker logs {container_name}

# List networks
docker network ls

# Inspect network
docker network inspect srm
```

---

## Troubleshooting

### Common Issues
1. **Docker Engine Not Running**: Restart your computer
2. **WSL Issues**: Run `wsl --update`
3. **Port Conflicts**: Make sure ports 3000, 5678, and 5432 are not in use by other applications
4. **Container Connection Issues**: Verify containers are connected to the `srm` network using `docker network inspect srm`

### Service URLs
- **Open WebUI**: `http://localhost:3000`
- **N8N**: `http://localhost:5678`
- **PostgreSQL**: `localhost:5432`

---

## Network Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────────┐
│   Open WebUI    │    │       N8N        │    │   PostgreSQL +     │
│  (Port 3000)    │    │   (Port 5678)    │    │     pgvector       │
│                 │    │                  │    │   (Port 5432)      │
└─────────────────┘    └──────────────────┘    └─────────────────────┘
                                │                          │
                                └──────────┬───────────────┘
                                          │
                                   ┌──────▼──────┐
                                   │  srm network │
                                   └─────────────┘
```

This setup creates a complete environment for running Open WebUI and N8N with RAG (Retrieval-Augmented Generation) capabilities using PostgreSQL with vector extensions.
