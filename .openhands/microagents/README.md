# OpenHands Microagents

These microagents provide specialized guidance for building Django/React applications using the multi-agent development system.

## Available Microagents

### 0. **startup.md** - Project Initialization
**Triggered by:** bootstrap, initialize, new project, start from scratch, create project
**Role:** Bootstrap new Django + React projects with complete structure and tooling

### 1. **architect.md** - System Design
**Triggered by:** design, architecture, openapi, contract, api spec, data model
**Role:** Transform requirements into implementation-ready design artifacts

### 2. **backend.md** - Django/Python Implementation
**Triggered by:** django, backend, api, service, model, drf, postgres, opensearch
**Role:** Implement Django backend with clean separation of concerns

### 3. **frontend.md** - React/TypeScript Implementation
**Triggered by:** react, frontend, component, tanstack, router, query, form, typescript
**Role:** Build React frontend with OpenAPI-generated client

### 4. **qa.md** - Testing & Quality Assurance
**Triggered by:** test, pytest, qa, fixture, bdd, testing, coverage
**Role:** Write pragmatic tests focused on business logic and critical paths

## How Microagents Work

Microagents are automatically activated by OpenHands when you use trigger keywords in your prompts.

**Example:**
```
You: "Bootstrap a new blog application"
→ Triggers: startup.md (contains "bootstrap")

You: "Design a blog API with posts and comments"
→ Triggers: architect.md (contains "design")

You: "Implement the Django backend for posts"
→ Triggers: backend.md (contains "django", "backend")

You: "Build React components for the post list"
→ Triggers: frontend.md (contains "react", "component")

You: "Write tests for the post service"
→ Triggers: qa.md (contains "test")
```

## Workflow

0. **Project Initialization** → `startup.md`
   - Input: Project name
   - Output: Complete project structure (Django + React + Docker + Git)

1. **Architecture Phase** → `architect.md`
   - Input: User requirements
   - Output: OpenAPI contract, task lists, data models

2. **Backend Implementation** → `backend.md`
   - Input: backend_tasks.md, api_contract.yaml
   - Output: Django services, API views, models

3. **Frontend Implementation** → `frontend.md`
   - Input: frontend_tasks.md, api_contract.yaml
   - Output: React components, API client, routes

4. **Testing Phase** → `qa.md`
   - Input: Completed features
   - Output: Unit tests, integration tests, BDD specs

## Tech Stack

**Backend:** Django 5.0+ • DRF • PostgreSQL • OpenSearch • Redis • uv • ruff
**Frontend:** React 18+ • TypeScript • TanStack Router/Query/Form • shadcn/ui • Vite
**Testing:** pytest • pytest-django • pytest-bdd • Hypothesis • SQLite (tests)

## Key Principles

- **OpenAPI contract is single source of truth**
- **Business logic in services/ (pure Python)**
- **OpenSearch documents fully denormalized**
- **Frontend uses OpenAPI-generated types**
- **Tests use SQLite + real services (no mocking)**

## Installation

These microagents are located at: `/Users/shayan/code/.openhands/microagents/`

When you start OpenHands from `/Users/shayan/code`, they are automatically loaded.

## Usage

Start OpenHands from your code directory:
```bash
cd /Users/shayan/code
uvx --python 3.12 --from openhands-ai openhands serve --mount-cwd
```

Verify microagents loaded (check terminal logs):
```
Loading user workspace microagents: ['startup', 'architect', 'backend', 'frontend', 'qa']
```

Then use trigger keywords in your prompts to activate specific agents.
