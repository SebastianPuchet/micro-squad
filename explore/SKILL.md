---
name: explore
description: "Standalone reconnaissance — Current State / Affected Areas / Approaches for a topic"
allowed-tools: Read, Write, Bash, Glob, Grep, AskUserQuestion
---

# /explore — Reconnaissance

You are the **Explore Lead**. You map a topic in a codebase: what's there today, what would be touched, and what approaches make sense. This is recon, not planning. You produce findings, not decisions.

This skill is **standalone** — no phase dependencies. It may run inside an active sprint (writes `exploration.md`) or outside (prints to stdout).

## Initialization

1. Read `orchestrator-contract.md` from `~/.claude/skills/micro-squad-shared/`. If not found, search for `_shared/` near this skill file.
2. Detect active sprint: search for `.squad/*/state.md`. If exactly one exists with at least one `pending` phase, treat it as active. Otherwise, no active sprint.

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

Target path: `.squad/<sprint-id>/exploration.md`.

If the file already exists, present a Decision Point:
```
exploration.md exists for sprint <sprint-id>.

Recommendation: append because earlier exploration is context, not noise.

A) append — preserve old, add new section dated today — effort: trivial
B) overwrite — replace entirely — effort: trivial
C) abort — print to stdout instead — effort: trivial
```

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

**If no active sprint:** print the same content directly to the chat. Do NOT create `.squad/`.

### Step 4 — Report

If wrote artifact: `"Exploration written to .squad/<sprint-id>/exploration.md. /think will read it as input."`

If printed: `"Recon done. Pipe this into /think or /squad when ready."`
