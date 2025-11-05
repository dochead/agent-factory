# Agent Factory

Automated development system using OpenHands + microagents to build Python CLI tools (and eventually full-stack apps).

## Current Status

**Phase 1: CLI Tool Builder**
- Simple Python CLI tool bootstrapping via OpenHands
- Start small, iterate upward

## Quick Start

### 1. Install OpenHands (one-time)

```bash
uv tool install openhands-ai --python 3.12
```

### 2. Copy microagents to home directory

```bash
mkdir -p ~/.openhands/microagents
cp .openhands/microagents/*.md ~/.openhands/microagents/
```

### 3. Start OpenHands

```bash
cd /Users/shayan/code/agent-factory
uvx openhands serve --mount-cwd
```

Verify microagents loaded in terminal:
```
Loading user workspace microagents: ['startup', ...]
```

### 4. Configure LLM

In OpenHands UI (http://localhost:3000):
- Settings → LLM Provider → OpenAI (or Claude)
- Enter API key
- Model: `gpt-4o` or `claude-sonnet-4`

### 5. Run First Test

See `START_HERE.md` for detailed test instructions.

**Quick test prompt:**
```
Bootstrap a new CLI tool called "task-breaker" that breaks down large tasks into steps.
```

## Tech Stack

**Current:**
- Python 3.12+
- uv (package management)
- typer (CLI framework)
- rich (terminal output)
- ruff (formatting/linting)

**Future:**
- Django + DRF (backend)
- React + TypeScript (frontend)
- OpenAPI contracts
- pytest

## Microagents

Active:
- **startup.md** - Bootstrap Python CLI tools

Future:
- **architect.md** - Design OpenAPI contracts
- **backend.md** - Implement Django backends
- **frontend.md** - Build React frontends
- **qa.md** - Write tests

## Philosophy

1. **Start simple** - Get ONE thing working
2. **Iterate upward** - Add complexity incrementally
3. **Verify constantly** - Test at each step
4. **No grand visions** - Build what works today

## Project Structure

```
agent-factory/
├── .openhands/
│   └── microagents/       # OpenHands agents (shared across all tools)
├── tool-name-1/           # Each tool is independent
│   ├── .venv/             # Isolated Python environment
│   ├── pyproject.toml
│   └── tool-name-1/
│       └── cli.py
├── tool-name-2/           # Another independent tool
│   ├── .venv/
│   └── ...
└── ...
```

Each tool has its own venv and dependencies. No shared environment.

## Workflow

1. Use OpenHands to bootstrap new tool
2. Edit the generated code manually
3. Test it works
4. Commit to git
5. Repeat

## Next Steps

Once CLI builder works:
1. Add logic to existing tools
2. Create more specialized microagents
3. Add Django backend builder
4. Add React frontend builder
5. Build full-stack orchestration

## Documentation

- `START_HERE.md` - First test instructions
- `.openhands/microagents/` - Microagent prompts
- `PROJECT_STRUCTURE.md` - Detailed architecture notes

## Why OpenHands?

- Autonomous coding agent
- Microagent system for specialized tasks
- Can execute commands, edit files, commit to git
- Works with any LLM (OpenAI, Anthropic, etc.)
- Open source, extensible
