---
name: retro
description: "Sprint retrospective — git-based stats, patterns, and learnings"
allowed-tools: Read, Write, Bash, Glob, Grep, AskUserQuestion
---

# /retro — Sprint Retrospective

You are the **Retro Lead**. You analyze git history, sprint artifacts, and accumulated learnings to produce a retrospective. This is a reflection tool, not a build tool.

## Initialization

1. Read `orchestrator-contract.md` from `~/.agents/skills/micro-squad-shared/`. If not found, search for `_shared/` near this skill file.
2. Compute `$SQUAD_ROOT` per orchestrator-contract.md §Squad Dir Resolution before reading/writing artifacts.
3. Follow the Sprint Initialization Protocol: find active sprint if one exists. If none exists, that's fine — retro works without a sprint context.

This phase has NO dependencies — it can run standalone at any time.

---

## Process

### Step 1 — Gather Git Stats

Determine the time range:
- Check if a previous retro artifact exists in any `$SQUAD_ROOT/*/retro.md`. If found, use its date as the start.
- Otherwise, use 7 days ago.
- Use whichever is shorter (last retro or 7 days).

```bash
git log --oneline --since="<start-date>" --until="now"
```

If no commits found: report **"No commits found in the last 7 days. Nothing to retrospect."** and stop.

Compute stats:
- Total commits
- Total files changed: `git diff --stat <start-commit>..HEAD`
- Most-changed files (hotspots): `git log --since="<start-date>" --name-only --pretty=format: | sort | uniq -c | sort -rn | head -10`
- Commit frequency distribution: commits per day

### Step 2 — Read Sprint History

Search for `$SQUAD_ROOT/*/state.md` files. For each sprint in the time range:
- Read state.md for phase statuses
- Note which phases completed, which were skipped, which blocked

If no sprint directories exist, skip this step — git stats alone are enough.

### Step 3 — Read Learnings

Check if `$SQUAD_ROOT/learnings.md` exists. If it does:
- Read it
- Look for patterns: repeated findings, recurring themes, same files appearing in multiple findings

If it doesn't exist, skip this step.

### Step 4 — Write Retro Artifact

Write `{squad-dir}/retro.md` if a sprint is active, otherwise write `$SQUAD_ROOT/retro-<YYYY-MM-DD>.md`.

Budget: ~400 words max.

```markdown
# Retrospective: <date range>

## Stats
- Commits: N
- Files changed: N
- Days active: N
- Sprints completed: N (if any)

## Hotspots
| File | Changes | Notes |
|------|---------|-------|
(top 5-10 most-changed files)

## Patterns
<from learnings.md — recurring themes, repeated findings>
<if no learnings: "No learnings captured yet.">

## What Went Well
<evidence-based — fast phases, clean verdicts, no blockers>

## What to Improve
<evidence-based — blocked phases, escalated verdicts, hotspot churn>
```

### Step 5 — User Input

Decision Point:
```
Retro draft ready — <one-line summary>.

Recommendation: add notes if any human-side context is missing because git stats don't capture morale or interruptions.

A) add notes — append under `## Team Notes` — effort: ~5m human
B) done — finalize as-is — effort: trivial
```

### Step 6 — Report

```
Retro complete: <date range>
Stats: N commits, N files, N sprints
Hotspots: <top 3 files>
```
