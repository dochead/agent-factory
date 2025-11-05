# Agent Factory

Multi-agent development system for building Django + React applications.

## Quick Start

### 1. Install OpenHands (one-time)

```bash
uv tool install openhands-ai --python 3.12
```

### 2. Start OpenHands

```bash
cd /Users/shayan/code/agent-factory
uvx openhands serve --mount-cwd
```

**IMPORTANT:** OpenHands runs in its own isolated uv tool environment. Each project you build will have its own separate venv. See `PROJECT_STRUCTURE.md` for details.

### 3. Verify Microagents Loaded

Check terminal output for:
```
Loading user workspace microagents: ['startup', 'architect', 'backend', 'frontend', 'qa']
```

### 3. Configure OpenAI

In OpenHands UI (http://localhost:3000):
- Settings → LLM Provider → OpenAI
- Enter API key
- Model: `gpt-4o` or `gpt-4o-mini`

### 4. Run First Test

Use the prompt from `START_HERE.md`

## Tech Stack

**Backend:** Django 5.0+ • DRF • SQLite • uv • ruff
**Frontend:** React 18+ • TypeScript • Vite • TanStack (Router/Query/Form) • shadcn/ui
**Testing:** pytest • pytest-django • SQLite (tests)

## Microagents

- **startup.md** - Bootstrap new projects
- **architect.md** - Design OpenAPI contracts + architecture
- **backend.md** - Implement Django backend (service layer pattern)
- **frontend.md** - Build React frontend (OpenAPI-generated client)
- **qa.md** - Write tests

## Workflow

```
Requirements → Architect → [Backend + Frontend parallel] → QA
```

The OpenAPI contract is the single source of truth between backend and frontend.

## Key Patterns

- Business logic in `services/` (pure Python, no Django imports)
- OpenSearch documents fully denormalized
- Frontend uses OpenAPI-generated types (never hand-written)
- Tests use SQLite + real services (no mocking internal Django services)

## Projects

Each project lives in its own directory with separate venvs:
```
agent-factory/
├── .openhands/          # OpenHands microagents (shared)
├── todo-app/
│   ├── backend/.venv/   # Backend Python venv (isolated)
│   └── frontend/        # Frontend Node deps (isolated)
├── blog-app/            # Future project
│   ├── backend/.venv/   # Separate venv
│   └── frontend/
└── ...
```

**See `PROJECT_STRUCTURE.md` for critical venv separation guidelines.**

## Documentation

Full documentation at: `/Users/shayan/code/multi-agent-system-docs/`
