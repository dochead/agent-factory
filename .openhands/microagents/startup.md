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

Bootstrap new Python CLI tools from scratch using uv. Execute these steps ONE AT A TIME.

**CRITICAL:** Execute each command separately. Verify with `pwd` and `ls` frequently.

## Instructions

When the user asks to bootstrap/create a new tool, ask them:
1. **Tool name** (e.g., "task-breaker", "code-analyzer")
2. **One-line description** (e.g., "Breaks down tasks into steps")

Then execute these steps:

## Step 1: Create Project Directory

```bash
cd /workspace/agent-factory
mkdir -p {TOOL_NAME}
cd {TOOL_NAME}
pwd  # Verify location
```

## Step 2: Create pyproject.toml

```bash
cat > pyproject.toml << 'EOF'
[project]
name = "{TOOL_NAME}"
version = "0.1.0"
description = "{DESCRIPTION}"
requires-python = ">=3.12"
dependencies = [
    "typer>=0.9.0",
    "rich>=13.0.0",
]

[project.optional-dependencies]
dev = [
    "ruff>=0.1",
    "pytest>=7.4",
]

[project.scripts]
{TOOL_NAME} = "{TOOL_NAME}.cli:app"

[tool.ruff]
line-length = 120

[tool.ruff.lint]
select = ["E", "F", "I"]
EOF

ls -la
```

## Step 3: Create Source Files

```bash
mkdir -p {TOOL_NAME}

# Create __init__.py
cat > {TOOL_NAME}/__init__.py << 'EOF'
"""
{DESCRIPTION}
"""
__version__ = "0.1.0"
EOF

# Create cli.py
cat > {TOOL_NAME}/cli.py << 'EOF'
import typer
from rich.console import Console

app = typer.Typer()
console = Console()

@app.command()
def main(
    input_text: str = typer.Argument(..., help="Input text to process"),
):
    """
    {DESCRIPTION}
    """
    console.print(f"[green]Processing:[/green] {input_text}")
    # TODO: Add actual logic here
    console.print("[blue]Done![/blue]")

if __name__ == "__main__":
    app()
EOF

ls -la {TOOL_NAME}/
```

## Step 4: Create README

```bash
cat > README.md << 'EOF'
# {TOOL_NAME}

{DESCRIPTION}

## Installation

```bash
uv venv --python 3.12
source .venv/bin/activate
uv pip install -e ".[dev]"
```

## Usage

```bash
{TOOL_NAME} "your input here"
```

## Development

```bash
# Format code
ruff format .

# Lint
ruff check .

# Run tests
pytest
```
EOF

ls -la
```

## Step 5: Initialize with uv

```bash
# Create venv
uv venv --python 3.12

# Activate and install
source .venv/bin/activate
uv pip install -e ".[dev]"

# Verify installation
which {TOOL_NAME}
{TOOL_NAME} --help
```

## Step 6: Create .gitignore

```bash
cat > .gitignore << 'EOF'
__pycache__/
*.pyc
.venv/
.pytest_cache/
*.egg-info/
dist/
build/
EOF
```

## Step 7: Initialize Git

```bash
git init
git add .
git commit -m "feat: initial {TOOL_NAME} setup"

ls -la
```

## Step 8: Test It Works

```bash
# Verify you're in the right place
pwd

# Test the CLI
{TOOL_NAME} "test input"

# Show structure
find . -type f -name "*.py" -o -name "*.toml" -o -name "*.md" | grep -v .venv
```

## Success Criteria

After completion, verify:
- ✅ Directory `/workspace/agent-factory/{TOOL_NAME}/` exists
- ✅ `pyproject.toml` with correct dependencies
- ✅ `{TOOL_NAME}/cli.py` with working CLI
- ✅ `.venv/` created and activated
- ✅ Command `{TOOL_NAME} --help` works
- ✅ Git initialized

## Handoff

Tell the user:

> "✅ CLI tool '{TOOL_NAME}' bootstrapped at `/workspace/agent-factory/{TOOL_NAME}/`
>
> You can now:
> 1. Edit `{TOOL_NAME}/cli.py` to add your logic
> 2. Test with: `{TOOL_NAME} "your input"`
> 3. Run `ruff format .` before committing
>
> The tool is installed in development mode - changes to .py files take effect immediately."

## Critical Rules

1. **Execute commands ONE AT A TIME** - Wait for each to complete
2. **Replace {TOOL_NAME} and {DESCRIPTION}** with actual values
3. **Use `pwd` frequently** - Verify you're in the right directory
4. **Use uv for everything** - Never use pip or pip3
5. **Activate venv** - `source .venv/bin/activate` before installing

## Anti-Patterns

❌ Running multiple commands in one go
❌ Creating files in wrong directory
❌ Using `pip install` instead of `uv pip install`
❌ Forgetting to replace {TOOL_NAME} placeholders
❌ Not activating venv before testing CLI
