---
name: plan
description: "Parallel architect + scout agents produce a unified implementation plan"
allowed-tools: Agent, Read, Write, Glob, Grep, AskUserQuestion
---

# /plan — Parallel Architecture & Scouting

You are the **Plan Lead**. You launch an Architect and a Scout in parallel, then synthesize their outputs into a unified plan.

## Initialization

1. Read `orchestrator-contract.md` and `agent-prompts.md` from `~/.claude/skills/micro-squad-shared/`. If not found, search for `_shared/` near this skill file.
2. Follow the Sprint Initialization Protocol: find active sprint.

## Dependency Check

Read `.squad/<sprint-id>/state.md`. Verify:
- think is `done` or `skipped`

If think is `pending`, stop: **"Run `/think` first, or `/think` then type 'skip' to bypass."**

If think is `skipped`, read the task description from state.md header and proceed without think.md.

---

## Process

### Step 1 — Launch Two Agents in Parallel

Read `agent-prompts.md`. Replace `{sprint-id}` with the actual sprint ID.

Launch both in a **single message** (two Agent tool calls):

1. **Architect** (subagent_type: general-purpose) — uses Architect template → writes `architecture.md`
2. **Scout** (subagent_type: Explore) — uses Scout template → writes `scout-report.md`

### Step 2 — Verify Agent Output

After both return:
1. Check `architecture.md` exists and has content. If missing: **"Architect agent failed. [retry / stop]"**
2. Check `scout-report.md` exists and has content. If missing: **"Scout agent failed. [retry / stop]"**

### Step 3 — Synthesize

Read both artifacts. Write `.squad/<sprint-id>/plan.md` (~600 words max):

```markdown
# Plan: <title>

## Architecture
<merged from architect — data flow, diagrams, key decisions>

## Codebase Context
<from scout — patterns to follow, code to reuse, risks>

## Conflicts
<where architect proposed X but scout found Y already exists — and your resolution>

## Implementation Steps
| # | Step | Files | Depends On | Notes |
|---|------|-------|------------|-------|

## Effort Estimate
- Human: ~Xh | Claude: ~Ym

## Open Questions
<anything unresolved that needs user input>
```

### Step 4 — User Confirmation

Present the plan summary. If there are open questions, ask them now.

Ask: **"Plan ready. Proceed to build? [yes / adjust / stop]"**
- **yes** → update state, suggest `/build`
- **adjust** → ask what to change, re-run the relevant agent (architect or scout) with modified context
- **stop** → save state, exit

Do NOT proceed without explicit confirmation.

Update state.md: plan → done.

Next: `/build`
