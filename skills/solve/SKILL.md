---
name: solve
description: MD-centric problem-solving framework — guides you from problem analysis to execution
level: 2
triggers:
  - "solve"
  - "solve continue"
  - "solve status"
  - "solve next"
  - "solve back"
---

<Purpose>
Main orchestrator for the solve framework. Routes users to the correct phase agent based on project state, handles session resumption, and manages phase transitions. This is the single entry point for all ongoing solve work.
</Purpose>

<Use_When>
- User runs `/solve` to continue an existing project
- User runs `/solve status` to check progress
- User runs `/solve next` to advance phases
- User runs `/solve back` to return to a previous phase
- User says "solve", "continue solving", "where was I"
</Use_When>

<Do_Not_Use_When>
- User wants to start a brand new project — use `solve-init` instead
- User has a clear, specific coding task — use executor directly
</Do_Not_Use_When>

<Steps>

## Command Dispatch

Parse the user's command and route:

| Input | Action |
|-------|--------|
| `/solve` (no args) | Run Session Resumption Algorithm |
| `/solve` + problem description | Run Session Resumption; if no project exists, init with description |
| `/solve status` | Run Status Display |
| `/solve next` | Run Phase Advance |
| `/solve back` | Run Phase Regression |

---

## Session Resumption Algorithm

This is the core entry point. Execute these steps in order:

### Step 1: Detect Project

Check if `.solve/state.json` exists in the current working directory.

**If state.json exists:**
- Read and parse state.json
- Extract `current_phase` field
- Go to Step 2 (Route to Agent)

**If .solve/ directory exists but no state.json:**
- Warn: ".solve/ directory exists but state.json is missing or corrupted."
- Attempt to reconstruct: Read PROGRESS.md, look for "## Current Phase" section, extract phase name
- If reconstructible: Create new state.json with extracted phase, go to Step 2
- If not reconstructible: Ask user to select phase manually:
  "Which phase are you in? [problem / direction / planning / execution]"
  Rebuild state.json with selection, go to Step 2

**If no .solve/ directory:**
- If user provided a problem description with the command:
  Invoke `solve-init` skill with the description, then start Phase 1
- If no description:
  "No solve project found here. Describe what you want to work on and I'll set one up, or run `/solve init`."

### Step 2: Route to Agent

Based on `current_phase` from state.json:

| Phase | Agent | Model | Context Files |
|-------|-------|-------|---------------|
| `problem` | problem-analyst | sonnet | WHY.md, THINK.md, PROGRESS.md, REFERENCE.md |
| `direction` | direction-setter | sonnet | WHY.md, THINK.md, DIRECTION.md, PROGRESS.md, REFERENCE.md |
| `planning` | solve-planner | opus | All .solve/ MD files |
| `execution` | solve-executor | opus | PLAN.md, PROGRESS.md, REFERENCE.md + project source |
| `complete` | (none) | — | Display completion summary, offer: "Start a new problem?" or "Revisit any phase?" |

**On agent spawn:**
1. Read all relevant MD files listed above
2. Read PROGRESS.md session log to understand history
3. Pass all context to the agent
4. Agent resumes conversation based on file content
5. After agent completes, append session entry to PROGRESS.md

---

## Status Display (`/solve status`)

Read `.solve/state.json` and `.solve/PROGRESS.md`. Display:

```
Project: {project_name}
Phase: {current_phase} ({phase_number}/4)
Started: {created_at}
Last updated: {updated_at}

Recent activity:
{last 3 entries from PROGRESS.md session log}

Transitions: {count from transition_log}
```

---

## Phase Advance (`/solve next`)

1. Read state.json for current_phase
2. Check transition preconditions for current phase:

**problem → direction:**
- WHY.md has non-empty content under "## Problem Statement" AND "## Background"
- THINK.md has at least 2 items under "## Structured Thoughts"

**direction → planning:**
- DIRECTION.md has non-empty "## Vision"
- DIRECTION.md has at least 2 items under "## Principles"
- DIRECTION.md has at least 1 entry in "## Key Decisions" table
- DIRECTION.md has content in both "### In Scope" and "### Out of Scope"

**planning → execution:**
- PLAN.md has at least 2 task items (`- [ ]`) each with an "**Acceptance:**" field

**execution → complete:**
- All tasks in PLAN.md are checked (`- [x]`)

3. **If preconditions met:** Advance phase, update state.json, update PROGRESS.md, spawn next agent
4. **If preconditions NOT met:**
   - List which conditions are not met
   - Ask: "Some conditions aren't met yet. Force advance anyway? This will be logged as an override."
   - If user confirms: advance with `trigger: "user_override"` in transition_log
   - If user declines: stay in current phase

---

## Phase Regression (`/solve back`)

1. Read state.json for current_phase
2. If current_phase is "problem": "Already at the first phase. Nothing to go back to."
3. Otherwise:
   - Ask: "Why are you going back? (This helps track the project history)"
   - Record reason in transition_log
   - Move to previous phase
   - Update state.json and PROGRESS.md
   - Spawn the previous phase's agent

Phase order: problem → direction → planning → execution

---

## State Update Protocol

Every phase transition (forward or backward) must:
1. Update `state.json`:
   - Set `current_phase` to new phase
   - Update `updated_at` to current timestamp
   - Close previous phase in `phase_history` (set `exited_at`)
   - Add new phase to `phase_history`
   - Add entry to `transition_log` with: from, to, trigger, reason, at
2. Update `PROGRESS.md`:
   - Set "## Current Phase" to new phase name
   - Add entry to session log table

</Steps>

<Tool_Usage>
- Use `Read` to check for .solve/state.json and parse it
- Use `Read` to load MD files for agent context
- Use `Write` / `Edit` to update state.json and PROGRESS.md on transitions
- Use `AskUserQuestion` for phase override confirmations and regression reasons
- Use `Agent` tool to spawn phase-specific agents with appropriate model tier
</Tool_Usage>
