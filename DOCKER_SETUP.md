# 🐳 Docker Setup for VOXSA-RAG

## 📋 Prerequisites

- Docker Desktop installed
- API Keys:
  - Hugging Face API Key: [Get it here](https://huggingface.co/settings/tokens)
  - Groq API Key: [Get it here](https://console.groq.com/keys)

---

## 🚀 Quick Start

### Step 1: Set Up Environment Variables

Create a `.env` file in the project root:

```bash
cp .env.example .env
```

Add your API keys:
```env
HUGGINGFACE_API_KEY=your_huggingface_key
GROQ_API_KEY=your_groq_key
```

**⚠️ IMPORTANT: Never commit .env to version control. Add to .gitignore:**
```bash
echo ".env" >> .gitignore
```

### Step 2: Build & Run

```bash
# Build the image
docker-compose build

# Start the application
docker-compose up -d

# View logs
docker-compose logs -f voxsa-app
```

### Step 3: Access Application

Open your browser: `http://localhost:8501`

---

## 🛑 Stop Application

```bash
docker-compose down          # Keep data
docker-compose down -v       # Remove all volumes and data
```

---

## 🔧 Common Commands

```bash
# View logs
docker-compose logs -f voxsa-app

# Execute command in container
docker-compose exec voxsa-app bash

# Rebuild without cache
docker-compose build --no-cache

# Check container status
docker-compose ps
```

---

## ⚠️ Troubleshooting

| Issue | Solution |
|-------|----------|
| Port 8501 in use | Change to `"9000:8501"` in docker-compose.yml |
| API keys not working | Verify .env file exists with correct keys; restart container |
| Build fails | Run `docker-compose build --no-cache` |
| Out of memory | Increase memory limit in docker-compose.yml |
| Permission denied | Run `sudo chmod -R 777 ./` or add user to docker group |

---

## 📊 Volumes

Data is persisted in these volumes:
- `app-data` - Application data
- `chroma-db` - Vector database
- `app-output` - Generated audio files

---

## 📝 Notes

- First run downloads ML models (~1-2GB) - may take 10-15 minutes
- Models are cached for subsequent runs
- ChromaDB persists indexed documents
- Check logs for any issues: `docker-compose logs voxsa-app`
