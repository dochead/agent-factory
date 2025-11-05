# Project Structure Guidelines

## Virtual Environment Separation

**CRITICAL: Keep OpenHands venv and project venvs completely separate.**

### OpenHands Environment (Tool Level)
```bash
# OpenHands runs in its own isolated uv tool environment
uv tool install openhands-ai --python 3.12

# Location: ~/.local/share/uv/tools/openhands-ai/
```

### Project Environments (Project Level)
```bash
# Each project gets its own venv
cd /Users/shayan/code/agent-factory/todo-app/backend
uv venv --python 3.12
source .venv/bin/activate
uv pip install -r requirements.txt
```

## Directory Structure

```
agent-factory/
├── .openhands/                  # OpenHands microagents (shared)
├── README.md
├── START_HERE.md
│
├── todo-app/                    # First project
│   ├── backend/
│   │   ├── .venv/              # Backend Python venv
│   │   ├── pyproject.toml
│   │   ├── manage.py
│   │   └── ...
│   ├── frontend/
│   │   ├── node_modules/       # Frontend dependencies
│   │   ├── package.json
│   │   └── ...
│   └── docs/
│       ├── api_contract.yaml
│       ├── architecture.md
│       └── ...
│
└── blog-app/                    # Future project
    ├── backend/
    │   ├── .venv/              # Separate backend venv
    │   └── ...
    └── frontend/
        └── ...
```

## Important Rules

### 1. Never Mix Environments
❌ **WRONG:**
```bash
# Don't install project deps in OpenHands environment
uv tool install openhands-ai --python 3.12
uv pip install django  # BAD - pollutes OpenHands env
```

✅ **CORRECT:**
```bash
# OpenHands in tool env
uv tool install openhands-ai --python 3.12

# Project in separate venv
cd agent-factory/todo-app/backend
uv venv --python 3.12
source .venv/bin/activate
uv pip install django
```

### 2. OpenHands Should Create Project Venvs
When OpenHands bootstraps a project, it should:
```bash
# In the project directory
uv venv --python 3.12
echo ".venv/" >> .gitignore
uv pip install django djangorestframework
```

### 3. Use pyproject.toml for Projects
Each project should have:
```toml
# backend/pyproject.toml
[project]
name = "todo-app-backend"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "django>=5.0",
    "djangorestframework>=3.14",
    "django-cors-headers>=4.3",
]

[tool.uv]
dev-dependencies = [
    "pytest>=7.4",
    "pytest-django>=4.5",
    "ruff>=0.1",
]
```

### 4. Activate Project Venv for Development
```bash
# When working on a project
cd agent-factory/todo-app/backend
source .venv/bin/activate
python manage.py runserver

# When running OpenHands
cd agent-factory
uvx openhands serve --mount-cwd  # Uses tool env
```

## Why This Matters

**Problem if mixed:**
- OpenHands updates break your projects
- Project dependencies conflict with OpenHands
- Hard to reproduce project environments
- Can't have multiple projects with different Python versions

**Solution with separation:**
- OpenHands upgrades don't affect projects
- Each project has isolated dependencies
- Clean, reproducible environments
- Multiple projects can use different stacks

## For Microagents

When the **startup** microagent bootstraps a new project, it should:

1. Create project directory structure
2. Create separate backend venv: `uv venv --python 3.12`
3. Create `pyproject.toml` with dependencies
4. Install deps: `uv pip install -e .`
5. Add `.venv/` to `.gitignore`
6. Create frontend with `npm create vite@latest`

**The project should never touch OpenHands' tool environment.**
