---
name: ship
description: "Package sprint artifacts into a commit + PR with full evidence trail"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
---

# /ship — Ship with Evidence

You are the **Ship Lead**. You package the sprint into a clean PR with full evidence. No agents needed — you do this directly.

## Initialization

1. Read `orchestrator-contract.md` from `~/.agents/skills/micro-squad-shared/`. If not found, search for `_shared/` near this skill file.
2. Follow the Sprint Initialization Protocol: find active sprint. Read state.md for `careful` flag.

## Careful Mode

If state.md has `careful: true`:
- Create a checkpoint commit before any state-changing step: `git commit --allow-empty -m "squad-checkpoint: ship"`
- Reject any "skip" / "just do it" shortcut. Every gate requires explicit `yes`.

## Dependency Check

Read state.md. Verify:
- build is `done` (REQUIRED — nothing to ship without it)
- verify is `done` or `skipped`

If build is not done: **"Nothing to ship. Run `/build` first."**

If verify is `skipped` or `pending`, present a Decision Point:
```
Shipping without /verify — review was skipped.

Recommendation: run /verify first because post-build review catches the cheap-to-fix issues.

A) run /verify now — effort: ~15m Claude
B) ship anyway — effort: trivial (risk: unreviewed)
C) stop — effort: trivial
```

(Careful mode: option B is not offered.)

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

2. **If uncommitted changes exist**, present a Decision Point:
   ```
   Uncommitted changes found: <list>.

   Recommendation: commit these as `squad: final changes` because shipping with a dirty tree leaves them stranded.

   A) commit and ship — effort: trivial
   B) leave uncommitted — ship only what's committed — effort: trivial
   C) abort — review them first — effort: ~5m human
   ```
   NEVER auto-commit without asking.

3. **If on base branch (main/master/develop/trunk)**, ABORT:
   **"You're on the base branch. Create a feature branch first: `git checkout -b feature/<name>`"**

4. **If branch has no commits ahead of base**, ABORT:
   **"No changes to ship. Branch is up to date with base."**

### Step 3 — Push and Create PR

Decision Point before pushing:
```
Ready to push <branch> and open the PR?

Recommendation: yes because the evidence trail is complete.

A) yes — push and create PR — effort: ~2m Claude
B) review first — let me inspect, ask again — effort: ~5m human
C) abort — effort: trivial
```

Then push the branch (with `-u` if needed) and create a PR:

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
