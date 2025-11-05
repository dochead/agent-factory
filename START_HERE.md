# First Test: Simple CLI Tool

Test the OpenHands + microagent setup by building a simple Python CLI tool.

## How Microagents Work

Microagents are triggered by **keywords** in your prompts:
- **Startup**: "bootstrap", "initialize", "new project"

When you use these keywords, OpenHands loads the relevant microagent into context.

## Goal

Build a working Python CLI tool to verify your OpenHands setup works.

## Prerequisites

1. **Install OpenHands** (one-time):
   ```bash
   uv tool install openhands-ai --python 3.12
   ```

2. **Copy microagents** to home directory (OpenHands looks here):
   ```bash
   mkdir -p ~/.openhands/microagents
   cp .openhands/microagents/*.md ~/.openhands/microagents/
   ```

3. **Start OpenHands**:
   ```bash
   cd /Users/shayan/code/agent-factory
   uvx openhands serve --mount-cwd
   ```

4. **Verify microagents loaded** in terminal output:
   ```
   Loading user workspace microagents: ['startup', 'architect', 'backend', 'frontend', 'qa']
   ```

## Test: Bootstrap a CLI Tool

**Copy this prompt into OpenHands:**

```
Bootstrap a new CLI tool called "task-breaker" that breaks down large tasks into steps.
```

**Expected Results:**

OpenHands should:
1. Ask you to confirm tool name and description (or use defaults)
2. Create `/workspace/agent-factory/task-breaker/` directory
3. Set up:
   - `pyproject.toml` with typer + rich dependencies
   - `task-breaker/cli.py` with working CLI
   - `.venv/` with dependencies installed
   - Git repository initialized
4. Test that `task-breaker --help` works

**Verify It Worked:**

After OpenHands finishes:
```bash
cd task-breaker
source .venv/bin/activate
task-breaker --help
task-breaker "test input"
```

You should see output with colored text (from rich).

## Success Criteria

- ✅ Directory `task-breaker/` exists
- ✅ `pyproject.toml` has correct dependencies
- ✅ `.venv/` created and working
- ✅ CLI command `task-breaker --help` works
- ✅ Git initialized

## If It Works

You now have a working OpenHands + microagent setup!

**Next steps:**
1. Edit `task-breaker/cli.py` to add actual logic
2. Create more CLI tools using the same pattern
3. Add more microagents for different types of projects

## If It Fails

Common issues:

### "Microagents not loading"
Check terminal output when OpenHands starts. If you don't see "Loading user workspace microagents", the files aren't in the right place.

**Fix:**
```bash
ls -la ~/.openhands/microagents/
# Should show: startup.md, architect.md, backend.md, frontend.md, qa.md
```

### "Commands failing"
OpenHands might be batching commands. The microagent instructs it to run commands one at a time, but if it's still batching:
- Report this as feedback to OpenHands team
- Manually run the commands from the agent's output

### "Wrong directory"
If OpenHands creates files in wrong locations, it's not following the `cd` instructions.
- Check the agent output for `pwd` results
- Manually verify you're in `/workspace/agent-factory`

## Expected Timeline

- **Bootstrap:** 5-10 minutes (OpenHands autonomous)
- **Manual verification:** 2 minutes (you)

## What's Next?

Once this works, you can:

1. **Add actual logic** to the CLI tool
2. **Create more tools** using the same bootstrap prompt
3. **Build up complexity** - add other microagents for Django backends, etc.
4. **Iterate** - improve the microagent based on what works/fails

The key is: **Start simple, iterate upward.**
