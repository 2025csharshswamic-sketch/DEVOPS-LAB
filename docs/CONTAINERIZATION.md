# Boomerang — Containerization & Docker DevOps Infrastructure

## 1. Containerization Strategy Overview

Boomerang utilizes Docker containerization to achieve environment parity across developer machines, continuous integration (CI) runners, and production cloud platforms (such as Render).

The containerized topology operates as a dual-container architecture:
1. **Web Application Container:** Built from `python:3.11-slim` executing Flask 3.0 via Gunicorn WSGI (`gunicorn.conf.py`).
2. **Database Container:** PostgreSQL 16 container providing relational ACID storage with transaction-scoped advisory locks.

```mermaid
flowchart LR
    subgraph Host ["Host / Docker Engine"]
        subgraph BridgeNet ["Default Compose Bridge Network"]
            AppCont["boomerang-web\n(Flask 3.0 / Gunicorn)\nPort: 8000 / ${PORT}"]
            DbCont["boomerang-db\n(PostgreSQL 16)\nPort: 5432"]
        end
        
        VolUploads[("uploads\nVolume (/app/static/uploads)")]
        VolDb[("postgres_data\nVolume (/var/lib/postgresql/data)")]
    end

    AppCont -->|SQLAlchemy connection (psycopg3) on 5432| DbCont
    AppCont --- VolUploads
    DbCont --- VolDb
```

---

## 2. Dockerfile Specification

Boomerang's Dockerfile uses an optimized multi-stage build structure based on `python:3.11-slim`:

```dockerfile
FROM python:3.11-slim AS base

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Stage 2: Testing stage for CI test runners
FROM base AS test
COPY requirements-dev.txt .
RUN pip install --no-cache-dir -r requirements-dev.txt
COPY . .

# Stage 3: Lean production runtime
FROM base AS runtime
RUN groupadd --gid 1000 appuser \
    && useradd --uid 1000 --gid 1000 --create-home --shell /bin/sh appuser

COPY --chown=appuser:appuser . .
RUN mkdir -p /app/static/uploads \
    && chown -R appuser:appuser /app/static/uploads

USER appuser

ENV PORT=8000
EXPOSE 8000

HEALTHCHECK --interval=15s --timeout=5s --start-period=15s --retries=3 \
    CMD ["python", "-c", "import os, urllib.request; port = os.environ.get('PORT', '8000'); urllib.request.urlopen(f'http://127.0.0.1:{port}/health', timeout=3)"]

CMD ["gunicorn", "--config", "gunicorn.conf.py", "app:app"]
```

### Key Dockerfile Features
- **Multi-Stage Design:** The `base` stage installs shared runtime dependencies (`psycopg`, `Pillow`, `scikit-learn`, `boto3`, `cloudinary`). The `test` stage adds development dependencies (`pytest`, `flake8`) for CI testing, while `runtime` packages only the production artifacts.
- **Non-Root Execution:** The application executes under `appuser` (UID 1000, GID 1000) rather than `root`, adhering to container security best practices.
- **Dynamic Port Binding:** The health check probes `http://127.0.0.1:${PORT}/health`, allowing dynamic port assignment on cloud platforms like Render where `$PORT` is assigned at runtime.
- **Gunicorn Entrypoint:** Configured via `gunicorn.conf.py` pointing to `app:app`, supporting dynamic worker concurrency (`WEB_CONCURRENCY=2` by default).

---

## 3. Docker Compose Orchestration (`compose.yaml`)

For local multi-container development and testing, `compose.yaml` orchestrates the complete stack:

```yaml
services:
  web:
    build: .
    restart: unless-stopped
    ports:
      - "8000:8000"
    environment:
      SECRET_KEY: ${SECRET_KEY:?Copy .env.example to .env and set SECRET_KEY}
      DATABASE_URL: postgresql+psycopg://${POSTGRES_USER:?Set POSTGRES_USER in .env}:${POSTGRES_PASSWORD:?Set POSTGRES_PASSWORD in .env}@db:5432/${POSTGRES_DB:?Set POSTGRES_DB in .env}
      UPLOAD_FOLDER: static/uploads
      IMAGE_STORAGE_BACKEND: ${IMAGE_STORAGE_BACKEND:-local}
      BOOMERANG_ENV: ${BOOMERANG_ENV:-}
      SESSION_COOKIE_SECURE: ${SESSION_COOKIE_SECURE:-}
      TRUSTED_HOSTS: ${TRUSTED_HOSTS:-}
      TRUSTED_PROXY_COUNT: ${TRUSTED_PROXY_COUNT:-}
      RATE_LIMIT_ENABLED: ${RATE_LIMIT_ENABLED:-true}
      CLOUDINARY_CLOUD_NAME: ${CLOUDINARY_CLOUD_NAME:-}
      CLOUDINARY_API_KEY: ${CLOUDINARY_API_KEY:-}
      CLOUDINARY_API_SECRET: ${CLOUDINARY_API_SECRET:-}
      CLOUDINARY_FOLDER: ${CLOUDINARY_FOLDER:-boomerang}
      GEMINI_API_KEY: ${GEMINI_API_KEY:-}
      GEMINI_MODEL: ${GEMINI_MODEL:-gemini-3.8-flash}
    volumes:
      - uploads:/app/static/uploads
    healthcheck:
      test: ["CMD", "python", "-c", "import os, urllib.request; port = os.environ.get('PORT', '8000'); urllib.request.urlopen(f'http://127.0.0.1:{port}/health', timeout=3)"]
      interval: 15s
      timeout: 5s
      retries: 3
      start_period: 15s
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB:?Set POSTGRES_DB in .env}
      POSTGRES_USER: ${POSTGRES_USER:?Set POSTGRES_USER in .env}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?Set POSTGRES_PASSWORD in .env}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 5
      start_period: 10s

volumes:
  postgres_data:
  uploads:
```

---

## 4. Volume Lifecycle & Image Storage Backends

1. **`postgres_data` Volume:** Mounts to `/var/lib/postgresql/data`, persisting database schemas, user accounts, items, and verification claims.
2. **`uploads` Volume:** Mounts to `/app/static/uploads`, persisting uploaded photographs in local development.
3. **Storage Backends (`IMAGE_STORAGE_BACKEND`):**
   - `local` (default): Stores uploaded files in `static/uploads`.
   - `cloudinary`: Zero-cost persistent image storage for ephemeral container environments (e.g., Render Free Tier).
   - `r2`: Cloudflare R2 S3-compatible private object storage for large-scale production.

---

## 5. Health Probing & Operational Commands

Boomerang exposes an unauthenticated health check endpoint at `/health`. It issues an active database ping (`SELECT 1`), returning `{"status": "ok"}, 200` on success or `503` if the database is unreachable.

### Operational Commands
```powershell
# 1. Copy environment template and configure secrets
Copy-Item .env.example .env

# 2. Build and launch services in background
docker compose up -d --build

# 3. Verify health status of running containers
docker compose ps

# 4. Probe application health endpoint
Invoke-RestMethod http://localhost:8000/health

# 5. Execute test suite inside running web container
docker compose exec web python -m pytest -m "not postgres"

# 6. Check Flake8 compliance inside container
docker compose exec web python -m flake8 .

# 7. Stop containers preserving persistent volumes
docker compose down
```
