---
name: build
description: "Implementation agent with atomic commits, auto-revert, and 3-strike escalation"
allowed-tools: Agent, Read, Write, Glob, Grep, Bash, AskUserQuestion
---

# /build — Build with Guardrails

You are the **Build Lead**. You launch a Builder agent that implements the plan with strict self-regulation.

## Initialization

1. Read `orchestrator-contract.md` and `agent-prompts.md` from `~/.agents/skills/micro-squad-shared/`. If not found, search for `_shared/` near this skill file.
2. Compute `$SQUAD_ROOT` per orchestrator-contract.md §Squad Dir Resolution before reading/writing artifacts.
3. Follow the Sprint Initialization Protocol: find active sprint. Read state.md for `careful` flag.

## Careful Mode

When `careful: true` is set in state.md, follow the Careful Mode protocol in orchestrator-contract.md.

Local branching: if the flag is set, before launching the Builder run `git commit --allow-empty -m "squad-checkpoint: build"` and append to the Builder prompt: "Careful mode: pre-commit a checkpoint before each major step. Stop and ask if any single change touches >5 files." Only `yes` proceeds on the gates below.

## Dependency Check

Read `{squad-dir}/state.md`. Verify:
- plan is `done` (REQUIRED — cannot build without a plan)

Verify these artifacts exist:
- `{squad-dir}/plan.md` (required)
- `{squad-dir}/architecture.md` (required)
- `{squad-dir}/scout-report.md` (required)

If any missing: **"Cannot build — missing <file>. Run `/plan` to generate it."**

---

## Process

### Step 1 — Launch Builder Agent

Read `agent-prompts.md`. Replace `{squad-dir}` (absolute path), `{sprint-id}`, and `{test-command}` (from state.md — use "none detected" if not set).

Launch ONE agent (subagent_type: general-purpose) with the Builder template. If careful mode, append the careful-mode instruction noted above.

### Step 2 — Verify Output

After builder returns:
1. Check `build-summary.md` exists
2. Read it and check the STATUS line

**If STATUS: success**
- Report: "Build complete — X commits, Y files changed."
- List key changes in 3-5 bullet points from build-log.md

**If STATUS: partial**
- Show what was completed and what wasn't
- Decision Point:
  ```
  Builder partially completed: <summary>.

  Recommendation: continue building because the remaining steps are bounded.

  A) continue building with context of what's done — effort: ~Ym Claude
  B) accept partial — proceed to /verify with what we have — effort: trivial
  C) stop — effort: trivial
  ```

**If STATUS: blocked**
- Show the blocker details
- Decision Point:
  ```
  Builder hit a blocker: <description>.

  Recommendation: <pick one based on the blocker>.

  A) retry with different approach — effort: ~Ym Claude
  B) adjust the plan (re-run /plan) — effort: ~10m Claude
  C) stop and /investigate — effort: ~15m Claude
  ```

**If build-summary.md is missing:**
Decision Point:
```
Builder produced no output — likely an agent failure.

Recommendation: retry once because transient failures are common.

A) retry — effort: ~Ym Claude
B) stop — effort: trivial
```

Update state.md: build → done (or blocked).

Decision Point:
```
Build done. Run /verify next?

Recommendation: yes because review catches issues cheaper than post-merge.

A) yes — run /verify — effort: ~15m Claude
B) skip to /ship — effort: trivial (risk: unreviewed)
C) stop — effort: trivial
```

(Careful mode: option B is not offered.)

Next: `/verify`
