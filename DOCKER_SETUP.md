# 🐳 Docker Setup for VOXSA-RAG

Complete guide for dockerizing and running VOXSA-RAG with proper API key management.

---

## 📋 Prerequisites

- Docker Desktop installed ([Download](https://www.docker.com/products/docker-desktop))
- Docker Compose installed (included with Docker Desktop)
- API Keys:
  - Hugging Face API Key: [Get it here](https://huggingface.co/settings/tokens)
  - Groq API Key: [Get it here](https://console.groq.com/keys)

---

## 🚀 Quick Start

### Step 1: Set Up Environment Variables

Create a `.env` file in the project root directory:

```bash
# Copy the example template
cp .env.example .env

# Edit .env with your API keys
nano .env  # or use your preferred editor
```

Add your API keys to the `.env` file:

```env
HUGGINGFACE_API_KEY=your_actual_huggingface_key
GROQ_API_KEY=your_actual_groq_key
```

### Step 2: Build and Run with Docker Compose

```bash
# Build the Docker image
docker-compose build

# Start the application
docker-compose up -d

# View logs
docker-compose logs -f voxsa-app

# Check container status
docker-compose ps
```

### Step 3: Access the Application

Open your browser and navigate to:
```
http://localhost:8501
```

---

## 🛑 Stopping the Application

```bash
# Stop the application (keeps data)
docker-compose down

# Stop and remove all volumes (removes data)
docker-compose down -v

# Stop specific container
docker-compose stop voxsa-app
```

---

## 🏗️ Docker Components

### Dockerfile Features
- ✅ Multi-stage build for optimized image size (~3.5GB)
- ✅ Proper dependency installation
- ✅ FFmpeg and audio processing tools included
- ✅ Health checks
- ✅ Production-ready configuration

### docker-compose.yml Features
- ✅ Environment variable management from `.env` file
- ✅ Named volumes for data persistence
- ✅ Resource limits and reservations
- ✅ Logging configuration
- ✅ Custom network setup
- ✅ Auto-restart policy

### .dockerignore
- Excludes unnecessary files from build context
- Reduces build time and final image size
- Improves security by excluding source control files

---

## 🔐 API Key Management

### Secure Environment Variable Handling

**DO NOT** commit your `.env` file to version control:

```bash
# Add to .gitignore (if not already present)
echo ".env" >> .gitignore
```

**For Production Deployment:**

1. Use Docker Secrets or Swarm Secrets:
```yaml
secrets:
  huggingface_api_key:
    external: true
  groq_api_key:
    external: true
```

2. Or use environment variables from your hosting platform:
   - AWS: Use AWS Secrets Manager
   - GCP: Use Google Secret Manager
   - Azure: Use Azure Key Vault
   - GitHub Actions: Use GitHub Secrets

---

## 📊 Volume Management

| Volume | Purpose | Persistence |
|--------|---------|-------------|
| `app-data` | Application data storage | ✅ Persistent |
| `chroma-db` | Vector database storage | ✅ Persistent |
| `app-output` | Generated audio files | ✅ Persistent |

### Backup Volumes

```bash
# Create backup of volumes
docker run --rm -v app-data:/data -v $(pwd):/backup alpine tar czf /backup/app-data.tar.gz -C /data .

# Restore volumes
docker run --rm -v app-data:/data -v $(pwd):/backup alpine tar xzf /backup/app-data.tar.gz -C /data
```

---

## 🔧 Common Commands

### Building & Running

```bash
# Build without cache
docker-compose build --no-cache

# Run in detached mode
docker-compose up -d

# Run in foreground (useful for debugging)
docker-compose up

# Rebuild and restart
docker-compose up -d --build
```

### Container Management

```bash
# View container logs
docker-compose logs voxsa-app

# Follow logs in real-time
docker-compose logs -f voxsa-app

# Execute command in running container
docker-compose exec voxsa-app bash

# Run one-off command
docker-compose run voxsa-app python script.py

# Check container status
docker-compose ps

# Inspect container details
docker inspect voxsa-rag-app
```

### Volume Management

```bash
# List volumes
docker volume ls

# Inspect a volume
docker volume inspect voxsa-rag_app-data

# Remove unused volumes
docker volume prune

# Remove specific volume
docker volume rm voxsa-rag_app-data
```

### Debugging

```bash
# Enter container shell
docker-compose exec voxsa-app bash

# Run with interactive terminal
docker-compose run -it voxsa-app bash

# View detailed logs with timestamps
docker-compose logs --timestamps -f voxsa-app

# Check resource usage
docker stats voxsa-rag-app
```

---

## ⚠️ Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| **Port 8501 already in use** | Another service using port | Change port in docker-compose.yml: `"9000:8501"` |
| **API Key not recognized** | Missing/incorrect .env file | Verify .env exists and contains correct keys; restart container |
| **Build fails** | Cache issues or missing dependencies | Run `docker-compose build --no-cache` |
| **Out of memory** | Container exceeds memory limit | Increase memory in docker-compose.yml: `memory: 8G` |
| **Volume permission denied** | Linux permission issues | Run `sudo chmod -R 777 ./data` or add user to docker group |
| **Application crashes on startup** | Missing model files or dependencies | Check logs: `docker-compose logs voxsa-app` |
| **Can't access application** | Network issues | Verify port mapping and firewall rules |
| **Slow model loading** | Downloading large models | First run may take 10-15 minutes; be patient |

### Debug Mode

Enable detailed logging:

```bash
# Modify docker-compose.yml
environment:
  - LOG_LEVEL=DEBUG
  - STREAMLIT_LOGGER_LEVEL=debug

# Then restart
docker-compose up -d
```

---

## 🚀 Production Deployment

### Best Practices

1. **Use specific Python version tag** (already done: `python:3.11-slim`)
2. **Never commit .env file** to version control
3. **Use Docker Secrets** for sensitive data
4. **Enable restart policies** (configured: `unless-stopped`)
5. **Set resource limits** (configured in docker-compose.yml)
6. **Use health checks** (configured in Dockerfile)
7. **Enable logging** (configured: JSON file driver)

### Kubernetes Deployment

Example deployment manifest:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: voxsa-api-keys
type: Opaque
stringData:
  HUGGINGFACE_API_KEY: your-key-here
  GROQ_API_KEY: your-key-here
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: voxsa-rag
spec:
  replicas: 1
  selector:
    matchLabels:
      app: voxsa-rag
  template:
    metadata:
      labels:
        app: voxsa-rag
    spec:
      containers:
      - name: voxsa-app
        image: voxsa-rag:latest
        ports:
        - containerPort: 8501
        envFrom:
        - secretRef:
            name: voxsa-api-keys
        resources:
          requests:
            memory: "2Gi"
            cpu: "1"
          limits:
            memory: "4Gi"
            cpu: "2"
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8501
          initialDelaySeconds: 40
          periodSeconds: 30
```

---

## 📈 Performance Optimization

### Image Size Optimization

Current optimization techniques used:
- ✅ Multi-stage build (reduces final size)
- ✅ Alpine-based Python image (slim variant)
- ✅ `--no-cache-dir` for pip
- ✅ Removed build dependencies from final stage

### Runtime Optimization

```yaml
# In docker-compose.yml
deploy:
  resources:
    limits:
      cpus: '2'
      memory: 4G
```

Adjust based on your system:
- **CPU**: More cores = faster processing (but doesn't help much for I/O)
- **Memory**: Needed for model loading (~2-3GB minimum)

---

## 🐛 Logs and Monitoring

### View Application Logs

```bash
# Last 50 lines
docker-compose logs --tail=50 voxsa-app

# Follow logs
docker-compose logs -f voxsa-app

# Logs from last hour
docker-compose logs --since 1h voxsa-app

# Logs until 30 minutes ago
docker-compose logs --until 30m voxsa-app
```

### Container Statistics

```bash
# Real-time resource usage
docker stats voxsa-rag-app

# CPU and memory usage
docker-compose exec voxsa-app free -h
docker-compose exec voxsa-app ps aux
```

---

## 🔄 Updating the Application

```bash
# Pull latest code
git pull origin main

# Rebuild the image
docker-compose build --no-cache

# Restart the application
docker-compose down
docker-compose up -d

# Verify
docker-compose logs -f voxsa-app
```

---

## 📝 Notes

- The first run will download large ML models (~1-2GB). This is normal and may take 10-15 minutes.
- Models are cached in volumes for subsequent runs.
- ChromaDB persists indexed documents across restarts.
- Output audio files are saved to `app-output` volume.

---

## 🆘 Need Help?

1. Check logs: `docker-compose logs voxsa-app`
2. Verify .env file is in the project root
3. Ensure Docker and Docker Compose are installed
4. Check that required ports are not in use
5. Verify API keys are correct and have necessary permissions

---

**Happy Dockerizing! 🚀**
