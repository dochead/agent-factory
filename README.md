# Agent Factory

Multi-agent development system for building Django + React applications.

## Quick Start

### 1. Start OpenHands

```bash
cd /Users/shayan/code/agent-factory
uvx --python 3.13 --from openhands-ai openhands serve --mount-cwd
```

### 2. Verify Microagents Loaded

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

Each project lives in its own directory:
```
agent-factory/
├── .openhands/          # Microagents
├── todo-app/            # First test project
├── blog-app/            # Future project
└── ...
```

## Documentation

Full documentation at: `/Users/shayan/code/multi-agent-system-docs/`
