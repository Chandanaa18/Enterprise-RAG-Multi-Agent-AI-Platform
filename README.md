# Secure Enterprise RAG + Agent Platform
A production-ready multi-agent RAG platform built with FastAPI, LangGraph, PostgreSQL (pgvector), and MLflow.
# Architecture
<img width="1301" height="731" alt="architecture" src="https://github.com/user-attachments/assets/9421e3c2-829a-4d3e-b793-dd4bc6063c10" />

# Workflow
<img width="1369" height="736" alt="workflow" src="https://github.com/user-attachments/assets/7a620e60-64b9-47c3-b6f6-f66c69f7158d" />

# Quick Start
Prerequisites
Tool	Version	Notes
Docker + Compose	24+	PostgreSQL, Redis, MLflow
Python	3.11+	Backend runtime
Node.js	18+	Frontend only — skip if not using the UI
Google API Key	—	Gemini LLM + embeddings (free tier)
Step 1 — Environment file

cp .env.example .env
Minimum .env for local development:


APP_NAME=Enterprise RAG Platform
APP_VERSION=1.0.0
APP_HOST=localhost
APP_PORT=8000
APP_ENABLE_DEBUG_MODE=true

DB_CONNECTION=postgresql+asyncpg
DB_HOST=localhost
DB_PORT=5433
DB_USERNAME=rag
DB_PASSWORD=rag
DB_DATABASE=rag
DATABASE_URL=postgresql+asyncpg://rag:rag@localhost:5433/rag

REDIS_URL=redis://localhost:6379/0

MLFLOW_TRACKING_URI=http://localhost:5001
MLFLOW_EXPERIMENT_NAME=knowledge_rag

GOOGLE_API_KEY=your_google_api_key_here

LLM_CONFIG_PATH=config/llm.yaml
RAG_CHUNK_STRATEGY=recursive
RAG_CHUNK_SIZE=800
RAG_CHUNK_OVERLAP=120
RAG_TOP_K_DENSE=20
RAG_TOP_K_AFTER_RERANK=6
RAG_ANSWER_PROMPT_VERSION=2

AZURE_AUTH_ENABLED=false

SESSION_INACTIVITY_TIMEOUT_HOURS=8
Step 2 — Start infrastructure

docker compose up -d
Service	URL / port
PostgreSQL	localhost:5433
Redis	localhost:6379
MLflow UI	http://localhost:5001
Adminer	http://localhost:8081
Step 3 — Python environment

python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
Step 4 — Run database migrations

make migrate
Step 5 — Seed a dev user

make seed
# Creates alice@example.com with roles ["employee", "admin"]
Step 6 — Start the backend

make dev
Service	URL
API Docs	http://localhost:8000/docs
Health check	http://localhost:8000/health_check
Step 7 — Start the frontend (optional)

cd frontend && npm install && npm run dev
Frontend runs at http://localhost:3000.

Step 8 — Start the Celery worker

make worker
Makefile Reference
Command	What it does
make dev	Backend with hot-reload on :8000
make worker	Celery worker for ingestion tasks
make chat	Frontend dev server on :3000
make migrate	Apply new Alembic migrations
make migrate-down	Roll back the last migration
make seed	Create alice@example.com dev user
make test	Run all tests
make eval	MLflow evaluation run
make clean	Remove __pycache__ / .pyc / pytest artifacts
API Reference

# Chat
curl -X POST http://localhost:8000/api/v1/chat \
  -H "Content-Type: application/json" \
  -H "X-User-Email: alice@example.com" \
  -d '{"message": "What is the onboarding process?"}'

# List sessions
curl http://localhost:8000/api/v1/sessions \
  -H "X-User-Email: alice@example.com"

# Health
curl http://localhost:8000/health_check
Troubleshooting
Symptom	Fix
User 'x@y.com' not found	Run make seed
ModuleNotFoundError	Activate venv: source .venv/bin/activate
Connection refused	Run docker compose up -d and wait for healthy status
MLflow traces missing	Confirm MLFLOW_TRACKING_URI=http://localhost:5001 and MLflow is running before the backend
Gemini 429 rate limit	Add OPENROUTER_API_KEY as fallback
AZURE_CLIENT_ID must be set	Set AZURE_AUTH_ENABLED=false for local dev
Empty retrieval	Sync documents via Confluence connector
