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

Bootstrap new Django/React projects from scratch with complete structure, tooling, and dependencies.

**CRITICAL:** You MUST run these exact commands in order. Do NOT skip steps. Do NOT make assumptions.

## Step 1: Create Project Directory

```bash
# Determine project name from user (e.g., "todo-app")
PROJECT_NAME="todo-app"  # Replace with actual name

# Create project structure
cd /workspace/agent-factory
mkdir -p ${PROJECT_NAME}/{backend,frontend,docker,docs}
cd ${PROJECT_NAME}
```

**Working directory is now:** `/workspace/agent-factory/${PROJECT_NAME}/`

## Step 2: Backend Setup (Django + uv)

```bash
cd backend

# Create pyproject.toml
cat > pyproject.toml <<'EOF'
[project]
name = "backend"
version = "0.1.0"
description = "Django backend"
requires-python = ">=3.12"
dependencies = [
    "django>=5.0",
    "djangorestframework>=3.14",
    "django-cors-headers>=4.3",
    "psycopg2-binary>=2.9",
    "redis>=5.0",
    "opensearch-py>=2.4",
    "python-dotenv>=1.0",
]

[project.optional-dependencies]
dev = [
    "ruff>=0.1",
    "pytest>=7.4",
    "pytest-django>=4.5",
    "pytest-cov>=4.1",
]

[tool.ruff]
line-length = 120
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I"]
ignore = []

[tool.pytest.ini_options]
DJANGO_SETTINGS_MODULE = "config.settings.test"
python_files = ["test_*.py", "*_test.py"]
markers = [
    "quick: Quick tests (unit tests)",
    "slow: Slow tests (integration tests)",
]
EOF

# Create venv and install dependencies
uv venv --python 3.12
source .venv/bin/activate
uv pip install -e ".[dev]"

# Create Django project
django-admin startproject config .

# Create directory structure
mkdir -p apps services api documents
mkdir -p config/settings

# Split settings
cat > config/settings/__init__.py <<'EOF'
EOF

cat > config/settings/base.py <<'EOF'
from pathlib import Path
import os
from dotenv import load_dotenv

load_dotenv()

BASE_DIR = Path(__file__).resolve().parent.parent.parent

SECRET_KEY = os.getenv("DJANGO_SECRET_KEY", "change-me-in-production")

DEBUG = True

ALLOWED_HOSTS = []

INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "rest_framework",
    "corsheaders",
]

MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "corsheaders.middleware.CorsMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.common.CommonMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    "django.contrib.messages.middleware.MessageMiddleware",
    "django.middleware.clickjacking.XFrameOptionsMiddleware",
]

ROOT_URLCONF = "config.urls"

TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [],
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.debug",
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
            ],
        },
    },
]

WSGI_APPLICATION = "config.wsgi.application"

DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": os.getenv("DB_NAME", "postgres"),
        "USER": os.getenv("DB_USER", "postgres"),
        "PASSWORD": os.getenv("DB_PASSWORD", "postgres"),
        "HOST": os.getenv("DB_HOST", "localhost"),
        "PORT": os.getenv("DB_PORT", "5432"),
    }
}

AUTH_PASSWORD_VALIDATORS = [
    {"NAME": "django.contrib.auth.password_validation.UserAttributeSimilarityValidator"},
    {"NAME": "django.contrib.auth.password_validation.MinimumLengthValidator"},
    {"NAME": "django.contrib.auth.password_validation.CommonPasswordValidator"},
    {"NAME": "django.contrib.auth.password_validation.NumericPasswordValidator"},
]

LANGUAGE_CODE = "en-us"
TIME_ZONE = "UTC"
USE_I18N = True
USE_TZ = True

STATIC_URL = "static/"
DEFAULT_AUTO_FIELD = "django.db.models.BigAutoField"

REST_FRAMEWORK = {
    "DEFAULT_PAGINATION_CLASS": "rest_framework.pagination.PageNumberPagination",
    "PAGE_SIZE": 20,
}

CORS_ALLOWED_ORIGINS = [
    "http://localhost:5173",  # Vite dev server
]
EOF

cat > config/settings/development.py <<'EOF'
from .base import *

DEBUG = True
EOF

cat > config/settings/test.py <<'EOF'
from .base import *

DEBUG = False

DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": ":memory:",
    }
}
EOF

cat > config/settings/production.py <<'EOF'
from .base import *

DEBUG = False
ALLOWED_HOSTS = os.getenv("ALLOWED_HOSTS", "").split(",")
EOF

# Create .env.example
cat > .env.example <<'EOF'
DJANGO_SETTINGS_MODULE=config.settings.development
DJANGO_SECRET_KEY=change-me-in-production
DB_NAME=postgres
DB_USER=postgres
DB_PASSWORD=postgres
DB_HOST=localhost
DB_PORT=5432
EOF

cp .env.example .env

# Create .gitignore
cat > .gitignore <<'EOF'
__pycache__/
*.py[cod]
*$py.class
.venv/
*.egg-info/
db.sqlite3
.env
.coverage
htmlcov/
.pytest_cache/
EOF
```

## Step 3: Frontend Setup (Vite + React + TanStack)

```bash
cd ../frontend

# Create Vite project with TypeScript
npm create vite@latest . -- --template react-ts

# Install dependencies
npm install
npm install @tanstack/react-router @tanstack/react-query @tanstack/react-form
npm install @tanstack/zod-form-adapter zod
npm install -D @tanstack/router-vite-plugin

# Initialize shadcn/ui
npx shadcn-ui@latest init -y
npx shadcn-ui@latest add button input card form dialog

# Configure vite.config.ts
cat > vite.config.ts <<'EOF'
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import { TanStackRouterVite } from '@tanstack/router-vite-plugin'
import path from 'path'

export default defineConfig({
  plugins: [react(), TanStackRouterVite()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },
    },
  },
})
EOF

# Create directory structure
mkdir -p src/routes
mkdir -p src/api
mkdir -p src/features

# Create root route
cat > src/routes/__root.tsx <<'EOF'
import { createRootRoute, Outlet } from '@tanstack/react-router'

export const Route = createRootRoute({
  component: () => (
    <div>
      <nav>
        <a href="/">Home</a>
      </nav>
      <hr />
      <Outlet />
    </div>
  ),
})
EOF

# Create index route
cat > src/routes/index.tsx <<'EOF'
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/')({
  component: Index,
})

function Index() {
  return (
    <div>
      <h1>Welcome</h1>
    </div>
  )
}
EOF

# Update main.tsx with QueryClient
cat > src/main.tsx <<'EOF'
import React from 'react'
import ReactDOM from 'react-dom/client'
import { RouterProvider, createRouter } from '@tanstack/react-router'
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { routeTree } from './routeTree.gen'
import './index.css'

const queryClient = new QueryClient()

const router = createRouter({ routeTree })

declare module '@tanstack/react-router' {
  interface Register {
    router: typeof router
  }
}

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <QueryClientProvider client={queryClient}>
      <RouterProvider router={router} />
    </QueryClientProvider>
  </React.StrictMode>,
)
EOF
```

## Step 4: Docker Services

```bash
cd ../docker

cat > docker-compose.yml <<'EOF'
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
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5

  opensearch:
    image: opensearchproject/opensearch:2.11.0
    environment:
      - discovery.type=single-node
      - OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m
      - DISABLE_SECURITY_PLUGIN=true
    ports:
      - "9200:9200"
    volumes:
      - opensearch_data:/usr/share/opensearch/data
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
  opensearch_data:
EOF
```

## Step 5: Project-Level Files

```bash
cd ..  # Back to /workspace/agent-factory/${PROJECT_NAME}/

# Create README
cat > README.md <<EOF
# ${PROJECT_NAME}

## Setup

### Backend
\`\`\`bash
cd backend
source .venv/bin/activate
python manage.py migrate
python manage.py runserver
\`\`\`

### Frontend
\`\`\`bash
cd frontend
npm run dev
\`\`\`

### Docker Services
\`\`\`bash
cd docker
docker-compose up -d
\`\`\`

## Development

- Backend: http://localhost:8000
- Frontend: http://localhost:5173
- OpenSearch: http://localhost:9200
EOF

# Create .gitignore
cat > .gitignore <<'EOF'
# Python
__pycache__/
*.py[cod]
.venv/
*.egg-info/
db.sqlite3

# Node
node_modules/
dist/

# Environment
.env
.env.local

# IDEs
.vscode/
.idea/
.DS_Store
EOF

# Initialize git
git init
git add .
git commit -m "chore: initial project setup"
```

## Step 6: Verify Setup

```bash
# Check backend
cd backend
source .venv/bin/activate
python manage.py check  # Should pass

# Check frontend
cd ../frontend
npm run build  # Should succeed
```

## Handoff to Next Agent

After bootstrap completes, tell the user:

> "Project structure created at `/workspace/agent-factory/${PROJECT_NAME}/`
>
> Ready for architecture phase. Use the **architect** agent to design the OpenAPI contract."

## Critical Rules

1. **Always create project subdirectory** - Never create files directly in `/workspace/agent-factory/`
2. **Use uv for Python** - Never use pip or poetry
3. **Create venv before installing** - Always run `uv venv` first
4. **Split Django settings** - Use base/development/test/production pattern
5. **Initialize git in project directory** - Not in agent-factory root
6. **Set working directory** - All subsequent commands should run in `/workspace/agent-factory/${PROJECT_NAME}/`

## Anti-Patterns

❌ Creating files in `/workspace/agent-factory/` directly
❌ Using `pip install` instead of `uv`
❌ Single `settings.py` file
❌ No venv
❌ No `.env` file
❌ Missing pyproject.toml
