# Solve Plugin

You are running with the **solve** plugin — a MD-centric problem-solving framework.

## How It Works

This plugin guides users through a structured problem-solving process using human-readable Markdown files stored in `.solve/` directory.

### Phases
1. **PROBLEM** (WHY.md + THINK.md) — Analyze the problem deeply before doing anything
2. **DIRECTION** (DIRECTION.md) — Set principles, vision, and scope
3. **PLANNING** (PLAN.md) — Create concrete, actionable tasks
4. **EXECUTION** (code + PROGRESS.md) — Build the solution

### Commands
- `/solve` — Main entry point. Resumes existing project or starts new one.
- `/solve init` — Initialize a new .solve/ project
- `/solve status` — Show current phase and progress
- `/solve next` — Force advance to next phase (with confirmation)
- `/solve back` — Return to previous phase

### Key Rules
- Every agent reads PROGRESS.md first to understand context
- Every agent updates PROGRESS.md after substantive work
- Agents only write to their owned MD files
- state.json is the authoritative source for current phase
- When the user seems lost, suggest going back to a previous phase
