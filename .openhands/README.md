# OpenHands Microagents (Source)

**IMPORTANT:** These microagents are the SOURCE. OpenHands reads from `~/.openhands/microagents/` (your home directory).

## Setup

Copy these microagents to your home directory:

```bash
mkdir -p ~/.openhands/microagents
cp .openhands/microagents/*.md ~/.openhands/microagents/
```

## Why Home Directory?

OpenHands mounts `~/.openhands` into the Docker container, not the project's `.openhands` directory.

This means:
- ✅ Microagents are available to ALL projects
- ✅ One-time setup across projects
- ❌ Each project can't have custom microagents (global only)

## Updating Microagents

If you modify microagents in this directory, re-copy them:

```bash
cp .openhands/microagents/*.md ~/.openhands/microagents/
```

## Verification

After copying, restart OpenHands and check logs for:
```
Loading user workspace microagents: ['startup', 'architect', 'backend', 'frontend', 'qa']
```
