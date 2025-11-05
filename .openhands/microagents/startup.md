---
name: Startup Agent
trigger_type: keyword
keywords:
  - bootstrap
  - initialize
  - start from scratch
  - new project
  - create project
  - setup project
  - init
---

# Role

Bootstrap new Django/React projects from scratch with correct structure, tooling, and configuration.

## When to Use

- Starting a brand new project
- User says: "create a new project", "start from scratch", "bootstrap", "initialize"

## Tech Stack

**Backend:** Django 5.0+ • uv • ruff • PostgreSQL • OpenSearch • Redis
**Frontend:** Vite • React 18+ • TypeScript • TanStack Router/Query/Form • shadcn/ui
**DevOps:** Docker Compose • Git • pre-commit hooks

## Project Structure

```
project-name/
├── backend/                    # Django
│   ├── config/settings/       # Split settings (dev/prod/test)
│   ├── apps/                  # Django apps
│   ├── services/              # Business logic
│   ├── api/                   # DRF views/serializers
│   ├── documents/             # OpenSearch documents
│   ├── manage.py
│   └── pyproject.toml         # uv configuration
├── frontend/                   # React
│   ├── src/
│   │   ├── api/               # API client
│   │   ├── features/          # Feature modules
│   │   ├── components/ui/     # shadcn/ui
│   │   ├── routes/            # TanStack Router
│   │   └── main.tsx
│   ├── package.json
│   └── vite.config.ts
├── docker/
│   └── docker-compose.yml     # Postgres, Redis, OpenSearch
├── .gitignore
├── .pre-commit-config.yaml
└── README.md
```

## Backend Setup (Django + uv)

**Key Steps:**

1. **Initialize:**
   ```bash
   cd backend
   echo "3.12" > .python-version
   uv init --python 3.12
   ```

2. **Add dependencies:**
   ```bash
   uv add django djangorestframework django-cors-headers
   uv add psycopg2-binary redis opensearch-py python-dotenv
   uv add --dev ruff pytest pytest-django pytest-cov
   ```

3. **Create Django project:**
   ```bash
   django-admin startproject config .
   mkdir -p apps services api documents
   ```

4. **Split settings:** Create `config/settings/{base,development,test,production}.py`

5. **Configure pyproject.toml:**
   - Line length: 120
   - Ruff linting with isort
   - Pytest with SQLite for tests

**Critical:** Use `uv` not pip/poetry

## Frontend Setup (Vite + TanStack)

**Key Steps:**

1. **Create Vite project:**
   ```bash
   cd frontend
   npm create vite@latest . -- --template react-ts
   ```

2. **Install TanStack:**
   ```bash
   npm install @tanstack/react-router @tanstack/react-query @tanstack/react-form
   npm install @tanstack/zod-form-adapter zod
   npm install -D @tanstack/router-vite-plugin
   ```

3. **Install shadcn/ui:**
   ```bash
   npx shadcn-ui@latest init
   npx shadcn-ui@latest add button input card form
   ```

4. **Configure vite.config.ts:**
   - Add TanStackRouterVite plugin
   - Add `@/` alias
   - Proxy `/api` to Django backend

5. **Setup routing:**
   - Create `src/routes/__root.tsx`
   - Create `src/routes/index.tsx`
   - Configure QueryClient in `main.tsx`

## Docker Services

**docker/docker-compose.yml:**
- PostgreSQL 16
- Redis 7
- OpenSearch 2.11

All services with healthchecks.

## Configuration Files

**Backend:**
- `.env.example` with DB_NAME, DJANGO_SECRET_KEY, etc.
- Split settings for dev/test/prod
- Ruff: 120 char lines, isort integration
- Pytest: SQLite for tests, markers for quick/integration

**Frontend:**
- TypeScript strict mode
- Prettier: 120 char lines
- TanStack Router file-based routing
- Proxy `/api` to Django

## Handoff

After creating structure:
1. **Architect Agent** → Design OpenAPI contract
2. **Backend Agent** → Implement Django apps/services
3. **Frontend Agent** → Build React features
4. **QA Agent** → Write tests

## Quality Checklist

- [ ] Backend uses uv (not pip)
- [ ] Python 3.12+ specified
- [ ] Django split settings configured
- [ ] Ruff configured (120 chars)
- [ ] Frontend uses Vite + TanStack
- [ ] shadcn/ui initialized
- [ ] TypeScript strict mode
- [ ] Docker Compose for services
- [ ] Git initialized
- [ ] .env.example created

## Reference

See `reference/01_project_initialization.md` for detailed walkthrough.

## Common Issues

- **uv not found:** Install with `curl -LsSf https://astral.sh/uv/install.sh | sh`
- **CORS errors:** Add frontend URL to `CORS_ALLOWED_ORIGINS`
- **Docker services fail:** Check Docker Desktop running, ports available
