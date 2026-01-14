# AGENTS.md

This document is for automated coding agents (and humans) working in this repository. It focuses on **how to build, run, lint, and test** the project without guessing.

## Repo overview

Second Me is a local-first system with:

- **Backend**: Python/Flask code under `lpm_kernel/`
- **Frontend**: Next.js app under `lpm_frontend/`
- **Ops scripts**: `scripts/*.sh` (local start/stop/status/setup)
- **Docker**: `docker-compose.yml`, `docker-compose-gpu.yml`, `Dockerfile.*`
- **Docs**: `docs/` (see `TECHNICAL_DESIGN*.md`)

Default ports used by local/dev scripts:

- **Backend**: `8002`
- **Frontend**: `3000`

## Quick start (recommended paths)

### Option A: Docker (fastest to get running)

```bash
make docker-up
# UI: http://localhost:3000
```

Stop containers:

```bash
make docker-down
```

### Option B: Integrated local (non-Docker)

Prereqs (as enforced by `scripts/setup.sh`):

- Python **3.12+**
- Poetry
- Node.js + npm
- cmake
- sqlite3

Install everything:

```bash
make setup
```

Start services (backend + frontend):

```bash
make start
```

Check status / stop:

```bash
make status
make stop
```

Backend-only mode:

```bash
./scripts/start.sh --backend-only
```

## Environment configuration

- The local scripts expect a root `.env` file.
- `scripts/start.sh` reads `LOCAL_APP_PORT` and `LOCAL_FRONTEND_PORT` from `.env` (defaults to `8002`/`3000` if missing).
- `scripts/start_local.sh` sources `.env` and uses values such as `LOCAL_BASE_DIR`, `LOCAL_LOG_DIR`, `LOCAL_APP_PORT`.

Agent guidance:

- **Do not commit secrets** into `.env` or other config files.
- If you need new env vars, prefer documenting them and providing safe defaults.

## Development commands (backend)

Python dependencies are managed via **Poetry** (`pyproject.toml`).

Install:

```bash
poetry install
```

Lint/format:

```bash
make format   # ruff format lpm_kernel/
make lint     # ruff check lpm_kernel/
```

Tests:

```bash
make test     # poetry run pytest tests
```

Database migrations:

```bash
python scripts/run_migrations.py
```

## Development commands (frontend)

Located in `lpm_frontend/` (Next.js).

```bash
cd lpm_frontend
npm install
npm run dev
```

Lint:

```bash
cd lpm_frontend
npm run eslint
npm run stylelint
```

## Common logs & runtime files

Local scripts write:

- `logs/start.log` (backend startup wrapper output)
- `logs/frontend.log` (frontend dev server output)
- `run/.backend.pid`, `run/.frontend.pid` (PID files)

Backend also appends runtime logs to `${LOCAL_LOG_DIR}/backend.log` (from `.env`) when started via `scripts/start_local.sh`.

## Project structure notes

- **Backend entry**: local start is orchestrated by `scripts/start.sh` → `scripts/start_local.sh`.
- **Docker**:
  - Backend container: `second-me-backend`
  - Frontend container: `second-me-frontend`
  - Compose file selects CPU vs GPU via `.gpu_selected` (see `Makefile` targets `docker-use-gpu` / `docker-use-cpu`).

## Contribution workflow for agents

- Keep changes **small and focused** (prefer multiple commits).
- Run the most relevant checks for the area you touched:
  - Backend-only change: `make lint` (and `make test` when feasible)
  - Frontend-only change: `npm run eslint` / `npm run stylelint` (in `lpm_frontend/`)
  - Docs-only change: no tests required
- Avoid reformatting unrelated files.

