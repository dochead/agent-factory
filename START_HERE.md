# First Test: Todo App

This is your first test of the multi-agent system.

## How Microagents Work

Microagents are triggered by **keywords** in your prompts:
- **Startup**: "bootstrap", "initialize", "new project"
- **Architect**: "design", "architecture", "openapi"
- **Backend**: "django", "backend", "implement"
- **Frontend**: "react", "frontend", "component"
- **QA**: "test", "pytest", "qa"

When you use these keywords, OpenHands loads the relevant microagent into context.

## Goal

Build a complete todo app following the strict Backend → Binding (OpenAPI) → Frontend pattern.

## Phase 1: Architecture & Design

**Copy this prompt into OpenHands:**

**Note:** The word "design" triggers the architect microagent.

```
I want to design a todo application.

REQUIREMENTS:
- Users can create todos (title, description, due date, status)
- Users can list all todos (with filtering by status: pending/completed)
- Users can mark todos as complete
- Users can delete todos
- Authentication: Simple user authentication (username/password)

TECHNICAL CONSTRAINTS:
- Backend: Django 5.0+ with DRF
- Database: SQLite (for simplicity)
- Frontend: React 18+ with TypeScript, Vite, TanStack Router/Query/Form, shadcn/ui
- OpenAPI contract is the single source of truth

DELIVERABLES:
1. api_contract.yaml (OpenAPI 3.0 specification)
2. architecture.md (system overview, component diagram)
3. data_model.md (Django models)
4. backend_tasks.md (ordered task list for backend agent)
5. frontend_tasks.md (ordered task list for frontend agent)

IMPORTANT PATTERNS:
- Business logic must be in services/ (pure Python, no Django web server specifc imports. Django utility, ORM and standalone libraries are allowed and encouraged)
- List view must use denormalized data (even if it's simple for this app)
- Frontend must generate API client from OpenAPI contract (never hand-write API calls)

Do NOT implement yet, just create the design artifacts.
```

## Phase 2: Review & Approve

Once the architect agent completes:
1. Review `api_contract.yaml` - is it complete?
2. Review `architecture.md` - does it make sense?
3. Review `backend_tasks.md` and `frontend_tasks.md` - are they specific?

**If good:** Proceed to Phase 3
**If needs changes:** Give feedback, iterate

## Phase 3: Backend Implementation

**Prompt:**

**Note:** The words "Django" and "backend" trigger the backend microagent.

```
Implement the Django backend for the todo app using backend_tasks.md and api_contract.yaml.

Follow these patterns strictly:
- Business logic in services/todo_service.py (pure Python)
- Thin API views (just delegate to services)
- Django models in todos/models.py
- DRF serializers and views in api/

Create these commits as you go:
1. "feat(todos): add todo model"
2. "feat(todos): add todo service layer"
3. "feat(todos): add DRF API views"

Run ruff format and ruff check before each commit.
```

## Phase 4: Frontend Implementation

**Prompt:**

**Note:** The words "React" and "frontend" trigger the frontend microagent.

```
Implement the React frontend for the todo app using frontend_tasks.md and api_contract.yaml.

Follow these patterns strictly:
1. Generate API client from api_contract.yaml (use openapi-typescript or similar)
2. Use TanStack Router for routing (file-based)
3. Use TanStack Query for data fetching
4. Use TanStack Form + Zod for form validation
5. Use shadcn/ui components (Button, Card, Dialog, etc.)

Create these commits as you go:
1. "feat(frontend): setup Vite + TanStack + shadcn"
2. "feat(frontend): generate API client from OpenAPI"
3. "feat(frontend): add todo list view"
4. "feat(frontend): add todo creation form"
5. "feat(frontend): add todo actions (complete, delete)"
```

## Phase 5: Testing

**Prompt:**

**Note:** The word "tests" triggers the qa microagent.

```
Write tests for the todo app.

Focus on:
1. Service layer tests (business logic)
2. API endpoint tests (integration tests)
3. Happy paths only for this MVP

Use SQLite for tests (no mocking).
```

## Success Criteria

After all phases:
- [ ] OpenAPI contract exists and is valid
- [ ] Django backend runs (`python manage.py runserver`)
- [ ] React frontend runs (`npm run dev`)
- [ ] API calls work end-to-end
- [ ] Tests pass (`pytest`)
- [ ] Code follows patterns (ruff check passes)
- [ ] Commits are clean and incremental

## Expected Timeline

- **Phase 1 (Design):** 15-30 minutes (OpenHands autonomous)
- **Phase 2 (Review):** 5-10 minutes (you)
- **Phase 3 (Backend):** 30-60 minutes (OpenHands autonomous)
- **Phase 4 (Frontend):** 45-90 minutes (OpenHands autonomous)
- **Phase 5 (Testing):** 20-40 minutes (OpenHands autonomous)

**Total:** 2-3 hours of OpenHands runtime, with your review/approval between phases.

## Next Steps

1. Install OpenHands (one-time): `uv tool install openhands-ai --python 3.12`
2. Start OpenHands: `cd /Users/shayan/code/agent-factory && uvx openhands serve --mount-cwd`
3. Use Phase 1 prompt above
4. Report back any issues or questions
