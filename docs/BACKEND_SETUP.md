# Local Backend Environment Setup Guide

This guide walks you through setting up the HELPDESK.AI backend locally for development and testing.

## Prerequisites

- **Python 3.9+** (3.10+ recommended)
- **pip** (Python package manager)
- **Git**
- **Supabase account** (free tier works) — [supabase.com](https://supabase.com)

## Step 1: Clone and Navigate

```bash
git clone https://github.com/ritesh-1918/HELPDESK.AI.git
cd HELPDESK.AI/backend
```

## Step 2: Create Virtual Environment

```bash
# Create virtual environment
python3 -m venv venv

# Activate (macOS/Linux)
source venv/bin/activate

# Activate (Windows)
venv\Scripts\activate
```

## Step 3: Install Dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

> **Note**: `torch` and `transformers` are large packages (2+ GB). Installation may take several minutes.

## Step 4: Configure Environment Variables

```bash
# Copy the example env file
cp .env.example .env

# Edit .env with your values
nano .env  # or use your preferred editor
```

### Required .env Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `SUPABASE_URL` | Your Supabase project URL | `https://abc123.supabase.co` |
| `SUPABASE_SERVICE_KEY` | Supabase service role key | Found in Supabase → Settings → API |

### Optional .env Variables

| Variable | Purpose |
|----------|---------|
| `ALLOW_DEGRADED_STARTUP=1` | Start backend even if ML models fail to load (useful for dev) |
| `SENTENCE_TRANSFORMER_MODEL_PATH` | Local path to sentence-transformer model |
| `REQUIRE_SUPABASE=true` | Fail startup if Supabase is not configured |

## Step 5: Set Up Supabase

1. Go to [supabase.com](https://supabase.com) and create a free project
2. Go to **Settings → API** in your Supabase dashboard
3. Copy the **Project URL** → paste as `SUPABASE_URL` in `.env`
4. Copy the **service_role key** → paste as `SUPABASE_SERVICE_KEY` in `.env`

## Step 6: Run the Backend

```bash
# Development mode (with auto-reload)
uvicorn main:app --reload --host 0.0.0.0 --port 8000

# Or with degraded startup (skip ML model loading):
ALLOW_DEGRADED_STARTUP=1 uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

The API will be available at `http://localhost:8000`.

## Step 7: Verify

### Health Check

```bash
curl http://localhost:8000/health
```

Expected response:
```json
{"status": "healthy"}
```

### API Documentation

Open your browser to:
- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

### Test Ticket Analysis

```bash
curl -X POST http://localhost:8000/ai/analyze_ticket \
  -H "Content-Type: application/json" \
  -d '{"subject": "Login broken", "description": "Cannot login after update"}'
```

## Schema Verification

To verify the database schema is correctly set up:

```bash
# Check Supabase tables exist
python3 -c "
import os
from dotenv import load_dotenv
from supabase import create_client

load_dotenv()
supabase = create_client(os.getenv('SUPABASE_URL'), os.getenv('SUPABASE_SERVICE_KEY'))
tables = supabase.table('tickets').select('*', count='exact').execute()
print(f'Tickets table OK (rows: {tables.count})')
"
```

## Common Issues

| Issue | Solution |
|-------|----------|
| `ModuleNotFoundError: fastapi` | Run `pip install -r requirements.txt` |
| `torch` install fails | Try `pip install torch --index-url https://download.pytorch.org/whl/cpu` |
| Supabase connection error | Verify `SUPABASE_URL` and `SUPABASE_SERVICE_KEY` in `.env` |
| Model download hangs | Set `ALLOW_DEGRADED_STARTUP=1` and provide local model path |
| Port 8000 already in use | Use `--port 8001` or kill existing process |

## Docker (Alternative)

A Dockerfile is provided for containerized setup:

```bash
docker build -t helpdesk-backend .
docker run -p 8000:8000 --env-file .env helpdesk-backend
```
