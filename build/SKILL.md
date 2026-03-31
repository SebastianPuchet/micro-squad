---
name: build
description: "Implementation agent with atomic commits, auto-revert, and 3-strike escalation"
allowed-tools: Agent, Read, Write, Glob, Grep, AskUserQuestion
---

# /build — Build with Guardrails

You are the **Build Lead**. You launch a Builder agent that implements the plan with strict self-regulation.

## Initialization

1. Read `orchestrator-contract.md` and `agent-prompts.md` from `~/.claude/skills/micro-squad-shared/`. If not found, search for `_shared/` near this skill file.
2. Follow the Sprint Initialization Protocol: find active sprint.

## Dependency Check

Read `.squad/<sprint-id>/state.md`. Verify:
- plan is `done` (REQUIRED — cannot build without a plan)

Verify these artifacts exist:
- `.squad/<sprint-id>/plan.md` (required)
- `.squad/<sprint-id>/architecture.md` (required)
- `.squad/<sprint-id>/scout-report.md` (required)

If any missing: **"Cannot build — missing <file>. Run `/plan` to generate it."**

---

## Process

### Step 1 — Launch Builder Agent

Read `agent-prompts.md`. Replace `{sprint-id}` and `{test-command}` (from state.md — use "none detected" if not set).

Launch ONE agent (subagent_type: general-purpose) with the Builder template.

### Step 2 — Verify Output

After builder returns:
1. Check `build-summary.md` exists
2. Read it and check the STATUS line

**If STATUS: success**
- Report: "Build complete — X commits, Y files changed."
- List key changes in 3-5 bullet points from build-log.md

**If STATUS: partial**
- Show what was completed and what wasn't
- Ask: **"Builder partially completed. [continue building / accept partial / stop]"**
- If continue: re-launch builder with context about what's already done

**If STATUS: blocked**
- Show the blocker details
- Ask: **"Builder hit a blocker: <description>. Options:"**
  - **A)** Retry with different approach
  - **B)** Adjust the plan (re-run `/plan`)
  - **C)** Stop and investigate (run `/investigate`)

**If build-summary.md is missing:**
- Report: **"Builder agent failed to produce output. [retry / stop]"**

Update state.md: build → done (or blocked).

Ask: **"Build done. Run judgment day? [yes / skip to ship / stop]"**

Next: `/verify`
