# Boomerang — Smart Campus Lost & Found System

## Project Overview

**Boomerang** is a privacy-first, automated lost-and-found web platform tailored for university and college campuses. Designed to solve the chaos of misplaced personal belongings, Boomerang provides an intelligent, secure, and respectful mechanism for students and staff to report lost items, report found items, match candidate reports, verify claims through privacy-preserving verification questions, and securely exchange contact information upon mutual agreement.

---

## Central Repository & Live Deployment

- **Main Project Repository:** [https://github.com/HNAnirudh06/Boomerang](https://github.com/HNAnirudh06/Boomerang)
- **Live Production URL:** [https://boomerang-9gwe.onrender.com](https://boomerang-9gwe.onrender.com)
- **Upstream Verified Commit:** `0e94b4315d05b51a552d3433d2700905350dacbe`
- **Application Framework:** Flask 3.0 / Python 3.11 / PostgreSQL 16 (Render & Neon) / Gunicorn WSGI

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
| **Backend Framework** | Flask 3.0.x (Python 3.11) | Modular blueprints architecture (`auth`, `items`, `claims`, `notifications`, `admin`, `api`, `health`) |
| **Database ORM** | SQLAlchemy 2.x & Flask-SQLAlchemy | Robust data models with PostgreSQL backend |
| **Database Server** | PostgreSQL 16 Alpine / Neon Serverless | Production database with SSL pooling and advisory locking |
| **WSGI Server** | Gunicorn | Production-grade WSGI HTTP Server with dynamic worker allocation |
| **Containerization** | Docker & Docker Compose | Multi-container setup (Flask app + PostgreSQL 16) |
| **CI/CD** | GitHub Actions | Automated linting, test suites, and security scans |
| **Hosting Platform** | Render (PaaS) | Web service with continuous deployment and health checks |

---

## Architecture & Containerization Highlights

As part of this integration, comprehensive documentation has been created:

1. [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
   - Component topology, blueprint boundaries, request lifecycle.
   - Privacy-first claim verification state machine.
   - Database schema relationships and data access security.

2. [docs/CONTAINERIZATION.md](docs/CONTAINERIZATION.md)
   - Production multi-stage Docker build specification.
   - Dual-container local orchestration via Docker Compose.
   - Dynamic port binding (`$PORT`) and container healthcheck endpoints (`/health/live`, `/health/ready`).
   - Persistent file storage and upload lifecycle.

3. [.github/workflows/docs-lint.yml](.github/workflows/docs-lint.yml)
   - Automated workflow validating markdown formatting and preventing sensitive file inclusion.

---

## Quality Assurance & Verification

The core application maintains strict code hygiene and comprehensive automated test coverage:
- **Test Suite:** 180 passing tests (`pytest -m "not postgres"`), covering matching algorithms, claims workflow, mutual consent, and contact deduplication.
- **Linting:** Flake8 compliant with 0 violations across all modules.
- **Security:** CSRF protection, secure cookies, parameter normalization, and role-based access control (Finder, Claimant, Admin).
