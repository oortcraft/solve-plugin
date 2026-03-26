---
name: solve-init
description: Initialize a new solve project with MD files and state tracking
level: 1
triggers:
  - "solve init"
  - "solve new"
  - "start solving"
---

<Purpose>
Creates a `.solve/` directory in the current working directory with 6 structured Markdown files and a state.json sidecar. This is the starting point for any new problem-solving project.
</Purpose>

<Use_When>
- User runs `/solve init` or `/solve init <project-name>`
- User describes a new problem and no `.solve/` directory exists
- Redirected from the main `/solve` skill when no existing project is found
</Use_When>

<Steps>

## Initialization

1. **Check for existing project**
   - If `.solve/` already exists in cwd, warn: "A solve project already exists here. Use `/solve` to continue or add `--force` to reinitialize."
   - If `--force` flag present, back up existing `.solve/` to `.solve.bak/` before reinitializing

2. **Get project name**
   - If provided as argument: use it (e.g., `/solve init my-app`)
   - If not provided: ask the user "What problem or project are you working on? Give it a short name."

3. **Create .solve/ directory and files**
   - Create `.solve/` directory
   - Copy all 6 MD templates, replacing `{{PROJECT_NAME}}` with the project name and `{{DATE}}` / `{{TIMESTAMP}}` with current date/time
   - Create `state.json` from template with `current_phase: "problem"`

4. **Announce initialization**
   Show the user:
   ```
   Solve project initialized: "{project_name}"

   Created .solve/ with:
     WHY.md        — Problem background and motivation
     THINK.md      — Brainstorming and structured thoughts
     DIRECTION.md  — Principles and key decisions
     PLAN.md       — Concrete execution plan
     PROGRESS.md   — Current status and session log
     REFERENCE.md  — Research and reference materials
     state.json    — Phase tracking

   Current phase: PROBLEM
   ```

5. **Hand off to Problem Analyst**
   - If the user provided a problem description along with the init command, pass it to the problem-analyst agent
   - Otherwise, prompt: "Describe the problem you want to solve. Don't worry about being organized — just tell me what's on your mind."
   - Spawn the problem-analyst agent (sonnet) with the user's input + WHY.md + THINK.md as context

</Steps>

<Tool_Usage>
- Use `Write` tool to create all template files
- Use `Bash` to create the `.solve/` directory
- Use `AskUserQuestion` if project name is not provided
- Spawn `problem-analyst` agent after initialization
</Tool_Usage>
