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

Bootstrap new Django/React projects from scratch. Execute these steps ONE AT A TIME in order.

**CRITICAL:** Execute each command separately. Do NOT try to run multiple steps at once.

## Step 1: Create Project Directory

```bash
cd /workspace/agent-factory
mkdir -p todo-app/backend todo-app/frontend todo-app/docker todo-app/docs
cd todo-app
pwd  # Verify you're in /workspace/agent-factory/todo-app
```

## Step 2: Initialize Backend (Django + uv)

```bash
cd backend

# Create minimal pyproject.toml
cat > pyproject.toml << 'PYPROJECT_EOF'
[project]
name = "backend"
version = "0.1.0"
requires-python = ">=3.12"
dependencies = [
    "django>=5.0",
    "djangorestframework>=3.14",
    "django-cors-headers>=4.3",
]

[project.optional-dependencies]
dev = [
    "ruff>=0.1",
    "pytest>=7.4",
    "pytest-django>=4.5",
]

[tool.ruff]
line-length = 120

[tool.pytest.ini_options]
DJANGO_SETTINGS_MODULE = "config.settings"
PYPROJECT_EOF

# Create venv and install
uv venv --python 3.12
source .venv/bin/activate
uv pip install -e ".[dev]"

# Create Django project
uv run django-admin startproject config .

# Create directory structure
mkdir -p apps services api documents

# Create .env
cat > .env << 'ENV_EOF'
DJANGO_SECRET_KEY=dev-secret-key-change-in-production
DEBUG=True
ENV_EOF

# Create .gitignore
cat > .gitignore << 'GITIGNORE_EOF'
__pycache__/
*.pyc
.venv/
db.sqlite3
.env
.pytest_cache/
GITIGNORE_EOF

pwd  # Verify you're in backend directory
ls -la  # Show created files
```

## Step 3: Configure Django Settings

```bash
# Still in backend directory
cd config

# Update settings.py to use .env
cat >> settings.py << 'SETTINGS_EOF'

# Custom settings
import os
from pathlib import Path

CORS_ALLOWED_ORIGINS = [
    "http://localhost:5173",
]

INSTALLED_APPS += [
    "rest_framework",
    "corsheaders",
]

MIDDLEWARE.insert(1, "corsheaders.middleware.CorsMiddleware")

REST_FRAMEWORK = {
    "DEFAULT_PAGINATION_CLASS": "rest_framework.pagination.PageNumberPagination",
    "PAGE_SIZE": 20,
}
SETTINGS_EOF

cd ..  # Back to backend directory
```

## Step 4: Verify Backend Setup

```bash
# Should still be in backend directory
pwd  # Should show: /workspace/agent-factory/todo-app/backend

# Check Django works
source .venv/bin/activate
python manage.py check

# Show what we created
ls -la
```

## Step 5: Initialize Frontend (Vite + React)

```bash
cd ../frontend  # Go to frontend directory

# Create Vite project
npm create vite@latest . -- --template react-ts

# Install base dependencies
npm install

# Install TanStack
npm install @tanstack/react-router @tanstack/react-query

# Update vite.config.ts for API proxy
cat > vite.config.ts << 'VITE_EOF'
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },
    },
  },
})
VITE_EOF

pwd  # Should show: /workspace/agent-factory/todo-app/frontend
ls -la
```

## Step 6: Create Docker Services

```bash
cd ../docker  # Go to docker directory

cat > docker-compose.yml << 'DOCKER_EOF'
version: '3.8'

services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: postgres
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    ports:
      - "6379:6379"

volumes:
  postgres_data:
DOCKER_EOF

pwd  # Should show: /workspace/agent-factory/todo-app/docker
```

## Step 7: Create Project README

```bash
cd ..  # Back to todo-app root

cat > README.md << 'README_EOF'
# Todo App

## Backend Setup
```bash
cd backend
source .venv/bin/activate
python manage.py migrate
python manage.py runserver
```

## Frontend Setup
```bash
cd frontend
npm run dev
```

## Docker Services
```bash
cd docker
docker-compose up -d
```
README_EOF

pwd  # Should show: /workspace/agent-factory/todo-app
```

## Step 8: Initialize Git

```bash
# Should be in todo-app root
cd /workspace/agent-factory/todo-app

cat > .gitignore << 'GIT_EOF'
__pycache__/
*.pyc
.venv/
node_modules/
dist/
db.sqlite3
.env
GIT_EOF

git init
git add .
git commit -m "chore: initial project setup"

pwd
ls -la
```

## Step 9: Final Verification

```bash
cd /workspace/agent-factory/todo-app

# Check backend
cd backend
ls -la pyproject.toml manage.py .venv/
source .venv/bin/activate
python manage.py check

# Check frontend
cd ../frontend
ls -la package.json vite.config.ts

# Show structure
cd ..
find . -maxdepth 3 -type f -name "*.py" -o -name "*.json" -o -name "*.toml" | head -20
```

## Success Criteria

After completion, you should have:
- ✅ `/workspace/agent-factory/todo-app/` directory
- ✅ Backend: `pyproject.toml`, `manage.py`, `.venv/`, `config/`
- ✅ Frontend: `package.json`, `vite.config.ts`, `src/`
- ✅ Docker: `docker-compose.yml`
- ✅ Git initialized

## Handoff

Tell the user:

> "✅ Project bootstrapped at `/workspace/agent-factory/todo-app/`
>
> Next step: Use the **architect** agent to design the API.
>
> Prompt: `Design a todo app API with create, list, complete, delete operations.`"

## Critical Rules

1. **Execute commands ONE AT A TIME** - Do not batch commands
2. **Always use `cd` to navigate** - Verify location with `pwd`
3. **Use uv for Python** - Never use pip
4. **Use heredocs for files** - One file at a time
5. **Verify each step** - Use `ls -la` and `pwd` after major steps

## Anti-Patterns

❌ Running entire script as one command
❌ Creating files in wrong directory
❌ Using `pip install`
❌ Skipping verification steps
❌ Not activating venv before Python commands
