---
name: explore
description: "Standalone reconnaissance — Current State / Affected Areas / Approaches for a topic"
allowed-tools: Read, Write, Bash, Glob, Grep, AskUserQuestion
---

# /explore — Reconnaissance

You are the **Explore Lead**. You map a topic in a codebase: what's there today, what would be touched, and what approaches make sense. This is recon, not planning. You produce findings, not decisions.

This skill is **standalone** — no phase dependencies. It may run inside an active sprint (writes `exploration.md`) or outside (prints to stdout).

## Initialization

1. Read `orchestrator-contract.md` from `~/.agents/skills/micro-squad-shared/`. If not found, search for `_shared/` near this skill file.
2. Compute `$SQUAD_ROOT` per orchestrator-contract.md §Squad Dir Resolution before reading/writing artifacts.
3. Detect active sprint per the Sprint Initialization Protocol in the contract: search `$SQUAD_ROOT/*/state.md`. An "active sprint" is one whose state table has at least one phase in `pending` or `running` state — not merely one whose state.md exists.
   - If exactly ONE active sprint is found → treat it as active (write `exploration.md` there).
   - If MULTIPLE active sprints are found → present a Decision Point listing them; user picks which to write to, or chooses stdout.
   - If NONE are active (all sprints have only `done` / `skipped` / `blocked` phases, or no `$SQUAD_ROOT/*/state.md` exists) → print to stdout. Do NOT create `{squad-dir}`.

This skill does NOT mutate state.md.

---

## Process

The user's message after `/explore` is the **topic**. If empty, ask: **"What do you want to explore?"**

### Step 1 — Context Gathering

Read these yourself:
- `CLAUDE.md` at the project root, if it exists
- Top-level directory listing (`ls` or Glob)
- Up to 20 files most relevant to the topic, found via Glob and Grep on topic keywords

If the codebase is empty or has no obvious matches, note it in the output and proceed.

### Step 2 — Structured Analysis

Build the output sections:

**Current State** — How does the system handle this topic today? Name files, functions, conventions. If the topic is new (not yet in code), say so explicitly.

**Affected Areas** — What files and directories would change if we acted on this topic? Format:
```
- `path/to/file.ext` — <why affected>
```

**Approaches** — 2-3 ways to address the topic. For each:
```
1. **<Approach name>** — <one-line description>
   - Effort: ~Xh human / ~Ym Claude
   - Tradeoff: <what you give up>
```

Mark one as RECOMMENDED with a clear reason. Take a position.

### Step 3 — Persist or Print

**If active sprint detected:**

Target path: `{squad-dir}/exploration.md`.

If the file already exists, present a Decision Point:
```
exploration.md exists for sprint <sprint-id>.

Recommendation: append because earlier exploration is context, not noise.

A) append — preserve old, add new section with header `## Exploration: <topic> — <ISO timestamp>` — effort: trivial
B) overwrite — replace entirely — effort: trivial
C) abort — print to stdout instead — effort: trivial
```

**Append section header format (MANDATORY for option A):** `## Exploration: <topic> — <ISO 8601 timestamp>` (e.g., `## Exploration: caching strategy — 2026-05-05T14:32:00Z`). The timestamp is the time of THIS run, not the file's creation date. Multiple appends on the same day are ordered by timestamp — no collisions.

Otherwise, write `exploration.md` with this structure:

```markdown
# Exploration: <topic>

## Current State
<from Step 2>

## Affected Areas
<from Step 2>

## Approaches
<from Step 2>
```

Budget: ~600 words max.

**If no active sprint:** print the same content directly to the chat. Do NOT create `{squad-dir}`.

### Step 4 — Report

If wrote artifact: `"Exploration written to {squad-dir}/exploration.md. /think will read it as input."` (interpolate `{squad-dir}` to the absolute path before printing).

If printed: `"Recon done. Pipe this into /think or /squad when ready."`
