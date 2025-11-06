# OpenHands Microagents

Specialized guidance for building CLI tools and Python applications using OpenHands.

## Available Microagents

### **startup.md** - Project Initialization
**Triggered by:** bootstrap, initialize, new project, start from scratch, create project
**Role:** Bootstrap new Python CLI projects with complete structure and tooling

## How It Works

Microagents are automatically activated by OpenHands when you use trigger keywords in your prompts.

**Example:**
```
You: "Bootstrap a new CLI quotation tool"
→ Triggers: startup.md (contains "bootstrap")
```

## Workflow

**Project Initialization** → `startup.md`
- Input: Project name and description
- Output: Complete Python project structure (pyproject.toml + CLI + tooling)

## Tech Stack

**Python:** uv • ruff • rich/typer for CLI
**Testing:** pytest • hypothesis

## Installation

Install OpenHands (one-time):
```bash
uv tool install openhands-ai --python 3.12
```

## Usage

Start OpenHands from your code directory:
```bash
cd /Users/shayan/code
uvx openhands serve --mount-cwd
```

Verify microagents loaded (check terminal logs):
```
Loading user workspace microagents: ['startup']
```

Then use trigger keywords in your prompts to activate the agent.
