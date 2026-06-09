# Docker Setup for VOXSA-RAG

## Quick Start

### Using Docker Compose

```bash
# Start the application
docker-compose up -d

# View logs
docker-compose logs -f voxsa-app

# Stop the application
docker-compose down
```

### Using Docker directly

```bash
# Build the image
docker build -t voxsa-rag .

# Run the container
docker run -p 8000:8000 -v $(pwd):/app voxsa-rag
```

## Components

- **Dockerfile** - Builds the container with Python 3.11, ffmpeg, and your app
- **docker-compose.yml** - Manages the container with port and volume mappings
- **.dockerignore** - Excludes unnecessary files from the Docker build

## Key Configuration

| Setting | Value | Purpose |
|---------|-------|---------|
| Base Image | python:3.11-slim | Lightweight Python environment |
| Port | 8000 | Application port |
| Volumes | ./:/app | Live code updates in development |
| Restart Policy | on-failure | Auto-restart if app crashes |

## Common Commands

```bash
# Rebuild without cache
docker-compose build --no-cache

# Run a command in the container
docker-compose exec voxsa-app python script.py

# Check container status
docker-compose ps

# View real-time logs
docker-compose logs -f voxsa-app

# Clean up containers and volumes
docker-compose down -v
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Port 8000 in use | Change to `"9000:8000"` in docker-compose.yml |
| Build fails | Run `docker-compose build --no-cache` |
| No logs showing | Check `docker-compose logs voxsa-app` |
| Permission issues | Add your user to docker group or use sudo |
