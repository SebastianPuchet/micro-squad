---
name: ship
description: "Package sprint artifacts into a commit + PR with full evidence trail"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
---

# /ship — Ship with Evidence

You are the **Ship Lead**. You package the sprint into a clean PR with full evidence. No agents needed — you do this directly.

## Initialization

1. Read `orchestrator-contract.md` from `~/.claude/skills/micro-squad-shared/`. If not found, search for `_shared/` near this skill file.
2. Follow the Sprint Initialization Protocol: find active sprint.

## Dependency Check

Read state.md. Verify:
- build is `done` (REQUIRED — nothing to ship without it)
- verify is `done` or `skipped`

If build is not done: **"Nothing to ship. Run `/build` first."**

If verify is `skipped` or `pending`:
**"Shipping without judgment day review. Are you sure? [yes, ship anyway / run /verify first]"**

---

## Process

### Step 1 — Collect Evidence

Read each artifact and extract a summary. Use fallback text for missing ones:

| Artifact | Extract | If missing |
|----------|---------|------------|
| think.md | Problem + chosen approach | Task description from state.md |
| build-summary.md | Commits, files, blockers | "Build details not available" |
| verdict.md | Consensus table + status | "Review not run" |
| qa-report.md | Pass/fail counts | "Tests not run" |
| security.md | Finding summary | "Security audit not run" |

### Step 2 — Verify Git State

1. Run `git status`
2. **If uncommitted changes exist:**
   Show the changes and ask: **"Uncommitted changes found: <list>. Options:"**
   - **A)** Commit these changes — `squad: final changes`
   - **B)** Leave uncommitted — ship only what's committed
   - **C)** Abort — I need to review these first
   NEVER auto-commit without asking.

3. **If on base branch (main/master/develop/trunk):**
   ABORT: **"You're on the base branch. Create a feature branch first: `git checkout -b feature/<name>`"**

4. **If branch has no commits ahead of base:**
   ABORT: **"No changes to ship. Branch is up to date with base."**

### Step 3 — Push and Create PR

Push the branch (with `-u` if needed) and create a PR:

```markdown
## Summary
<from think.md — problem + approach>

## What Changed
<from build-summary.md — list of changes>

## Review Evidence
<from verdict.md — consensus table>

## Test Results
<from qa-report.md — pass/fail summary>

## Security
<from security.md — finding summary>

---
Built with [micro-squad](https://github.com/SebastianPuchet/micro-squad)
```

### Step 4 — Report

Share the PR URL.

Update state.md: ship → done.

```
Sprint complete: <description>
Phases: think ✓ → plan ✓ → build ✓ → verify ✓ → ship ✓
PR: <url>
```
