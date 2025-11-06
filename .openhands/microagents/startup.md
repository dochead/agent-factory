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

Bootstrap new Python CLI tools from scratch using uv and modern Python practices.

## Instructions

When the user asks to bootstrap/create a new tool:

1. **Ask for clarification** if needed:
   - Tool name (e.g., "task-breaker")
   - Brief description (e.g., "Breaks down tasks into steps")

2. **Create the tool** in `/workspace/{TOOL_NAME}/`

## Requirements

### Technology Stack
- **Package manager**: uv (NEVER pip)
- **Python version**: 3.12+
- **CLI framework**: typer
- **Output**: rich (for colored terminal output)
- **Dev tools**: ruff (formatting + linting), pytest

### Implementation Steps

1. **Initialize the project:**
   ```bash
   uv init {TOOL_NAME} --python 3.12
   cd {TOOL_NAME}
   ```
   This creates: `pyproject.toml`, `.python-version`, `README.md`, `.gitignore`, `hello.py`

2. **Add dependencies:**
   ```bash
   uv add typer rich
   uv add --dev ruff pytest
   ```
   This updates `pyproject.toml`, creates `uv.lock`, and syncs `.venv`

3. **Create the CLI structure:**
   - Create `{TOOL_NAME}/` package directory
   - Create `{TOOL_NAME}/__init__.py` with `__version__ = "0.1.0"`
   - Create `{TOOL_NAME}/cli.py` with the main CLI code

4. **Add the CLI entry point to pyproject.toml:**
   ```toml
   [project.scripts]
   {TOOL_NAME} = "{TOOL_NAME}.cli:app"
   ```

5. **Configure ruff in pyproject.toml:**
   ```toml
   [tool.ruff]
   line-length = 120
   ```

### CLI Template Requirements
- Use `typer.Typer()` app
- Use `rich.Console()` for output
- Include an about command with app name, description and version information
- The command should be encapsulated in a separate function for separation of concerns
- Create a pytest unit test for the command functionality, asserting the correct app name (not the version)
- Include helpful --help text

### Final Project Structure
```
{TOOL_NAME}/
├── pyproject.toml          # Created by uv init, modified to add scripts
├── uv.lock                 # Created by uv add
├── .python-version         # Created by uv init
├── README.md               # Created by uv init
├── .gitignore              # Created by uv init
├── .venv/                  # Created by uv add (don't commit)
├── {TOOL_NAME}/
│   ├── __init__.py         # You create this
│   └── cli.py              # You create this
└── tests/
    └── test_cli.py         # You create this
```

### uv Project Management Best Practices

**Initialize a new project:**
```bash
# Create new project with scaffolding
uv init my-tool

# Or initialize in existing directory
cd my-tool && uv init
```

This creates:
- `pyproject.toml` - Project config and dependencies
- `.python-version` - Python version specification
- `uv.lock` - Cross-platform lockfile (commit to git)
- `.venv/` - Virtual environment (auto-managed)
- `main.py` - Entry point

**Managing dependencies:**
```bash
# Add runtime dependencies (updates pyproject.toml, uv.lock, and .venv)
uv add typer rich

# Add with version constraints
uv add 'ruff>=0.3.0'

# Add development dependencies
uv add --dev pytest ruff

# Add optional dependency groups
uv add --group docs sphinx

# Remove dependencies
uv remove package-name

# Sync environment to lockfile (after pulling changes)
uv sync

# Sync with all optional dependencies
uv sync --all-extras

# Sync frozen (fail if lockfile out of date)
uv sync --locked
```

**Running code:**
```bash
# Run commands in project environment (auto-syncs first)
uv run python script.py
uv run my-tool --help

# Run Python module
uv run -m pytest

# Run without syncing (uses current env state)
uv run --frozen my-tool
```

**Key files uv manages:**
- `pyproject.toml` - Created by `uv init`, updated by `uv add`
- `uv.lock` - Lockfile ensuring reproducible installs (commit to git)
- `.python-version` - Pins Python version for the project
- `.venv/` - Auto-created virtual environment (don't commit)

### Setup Steps
1. Initialize project: `uv init {TOOL_NAME} --python 3.12`
2. Change to project directory: `cd {TOOL_NAME}`
3. Add dependencies: `uv add typer rich`
4. Add dev dependencies: `uv add --dev ruff pytest`
5. Create CLI package structure and code
6. Update pyproject.toml with `[project.scripts]` entry point
7. Initialize git repo and commit
8. Verify the CLI works: `uv run {TOOL_NAME} --help`

## Critical Rules

1. **Use uv project commands** - `uv init`, `uv add`, `uv run`, `uv sync` (NOT `pip` or `uv pip`)
2. **Let uv manage the environment** - It auto-creates and syncs `.venv`
3. **Commit the lockfile** - Always commit `uv.lock` to git
4. **Verify it works** - Run `uv run {TOOL_NAME} --help` before finishing
5. **One command at a time** - Don't batch complex operations
6. **Git init** - Initialize git and make initial commit

## Success Criteria

The user should be able to:
```bash
cd {TOOL_NAME}
uv run {TOOL_NAME} --help    # Shows help (uv auto-syncs environment)
uv run {TOOL_NAME} "test"    # Runs successfully
```

## Handoff Message

After completion, tell the user:

```
✅ CLI tool '{TOOL_NAME}' ready at /workspace/{TOOL_NAME}/

Test it:
  cd {TOOL_NAME}
  uv run {TOOL_NAME} --help

Changes to {TOOL_NAME}/cli.py take effect immediately with `uv run`.
To add dependencies: `uv add package-name`
```

## Anti-Patterns

❌ Using `pip` or `uv pip` instead of `uv add`/`uv sync`
❌ Manually managing virtual environments
❌ Not committing `uv.lock` to git
❌ Creating files outside /workspace/{TOOL_NAME}/
❌ Not verifying the CLI works before finishing
❌ Forgetting to initialize git
