# Boomerang — Smart Campus Lost & Found System

## Project Overview

**Boomerang** is a privacy-first, automated lost-and-found web platform tailored for university and college campuses. Designed to solve the chaos of misplaced personal belongings, Boomerang provides an intelligent, secure, and respectful mechanism for students and staff to report lost items, report found items, match candidate reports using machine learning, verify claims through privacy-preserving verification questions, and securely exchange contact information upon mutual agreement.

---

## Central Repository & Live Deployment

- **Main Project Repository:** [https://github.com/HNAnirudh06/Boomerang](https://github.com/HNAnirudh06/Boomerang)
- **Live Production URL:** [https://boomerang-9gwe.onrender.com](https://boomerang-9gwe.onrender.com)
- **Upstream Verified Commit:** `0e94b4315d05b51a552d3433d2700905350dacbe`
- **Application Framework:** Flask 3.0 / Python 3.11 / PostgreSQL 16 (Render & Neon) / Gunicorn WSGI (`gunicorn.conf.py`)

---

## Academic & Team Context

- **Institution:** The National Institute of Engineering (NIE), Mysuru
- **Department:** Department of Computer Science & Engineering (CSE)
- **Course:** DevOps Laboratory / CDD Minor Project
- **Contributor:** **Harsh Swami**
- **USN:** `4NI25CS081`
- **Module Ownership:** **System Architecture, Containerization & Docker DevOps Infrastructure**

---

## Technical Stack

| Component | Technology | Description |
| :--- | :--- | :--- |
| **Backend Framework** | Flask 3.0.x (Python 3.11) | Modular blueprints architecture (`auth`, `items`, `claims`, `notifications`, `exchanges`) |
| **Database ORM** | SQLAlchemy 2.x & Flask-SQLAlchemy | Robust data models with PostgreSQL backend and SQLite test fallback |
| **Database Driver** | `psycopg[binary]>=3.1` (psycopg3) | Modern PostgreSQL driver with native binary protocol |
| **Database Server** | PostgreSQL 16 / Neon Serverless | Production database with transactional advisory locking (`pg_advisory_xact_lock`) |
| **WSGI Server** | Gunicorn | Production WSGI HTTP Server configured via `gunicorn.conf.py` with dynamic port binding |
| **Matching Engine** | scikit-learn & PyTorch/ResNet-50 | Random Forest Classifier trained on 6 engineered text, image, category, location, and date features |
| **Storage Engine** | Local / Cloudinary / Cloudflare R2 | Pluggable image storage backend with signed/authenticated access |
| **Containerization** | Docker & Docker Compose (`compose.yaml`) | Multi-stage build (`python:3.11-slim`), non-root execution (`appuser:1000`), dual-container stack |
| **CI/CD** | GitHub Actions (`.github/workflows/ci.yml`) | Multi-job workflow: SQLite unit tests, PostgreSQL 16 integration tests, and Docker runtime validation |

---

## Architecture & Containerization Highlights

As part of this integration, comprehensive documentation has been created:

1. [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
   - Component topology, blueprint boundaries (`auth`, `items`, `claims`, `notifications`, `exchanges`), and request lifecycle.
   - Privacy-first claim verification state machine (`submitted` → `finder_approved` → `mutual_consent_granted` → `completed`).
   - Schema relationships, immutability triggers, and data access security.

2. [docs/CONTAINERIZATION.md](docs/CONTAINERIZATION.md)
   - Multi-stage Docker build specification (`base`, `test`, `runtime`).
   - Dual-container orchestration via `compose.yaml` (Flask app + PostgreSQL 16).
   - Dynamic port binding (`$PORT`, default 8000) and container healthcheck (`/health`).
   - Storage backends and persistent volume management.

3. [.github/workflows/docs-lint.yml](.github/workflows/docs-lint.yml)
   - Automated workflow validating markdown formatting and preventing sensitive file inclusion.

---

## Quality Assurance & Verification

The core application maintains strict code hygiene and comprehensive automated test coverage:
- **Test Suite:** 181 passed, 3 deselected (PostgreSQL-specific integration tests) out of 184 total collected tests (`pytest -m "not postgres"`).
- **Code Style:** Flake8 compliant with 0 violations across all modules.
- **Security:** CSRF protection on all forms, secure HTTP-only cookies, parameter normalization, and role-based access control.
