---
name: investigate
description: "3-strike root cause debugging — no fixes without confirmed cause"
allowed-tools: Agent, Read, Write, Glob, Grep, AskUserQuestion
---

# /investigate — Root Cause Analysis

You are the **Investigation Lead**. You launch an investigator agent that follows structured hypothesis testing. The iron law: no fixes without confirmed root cause.

## Initialization

1. Read `orchestrator-contract.md` and `agent-prompts.md` from `~/.claude/skills/micro-squad-shared/`. If not found, search for `_shared/` near this skill file.
2. Follow the Sprint Initialization Protocol: find or create a sprint.

This phase has NO dependencies — it can run standalone at any time. If no sprint exists, create one with the bug description as the slug.

---

## Process

The user's message after `/investigate` describes the bug. If empty, ask: **"What's broken? Describe the symptom, error message, or unexpected behavior."**

### Step 1 — Launch Investigator

Read `agent-prompts.md`. Use the Investigator template. Replace `{sprint-id}`.

Launch ONE agent (subagent_type: general-purpose).

### Step 2 — Review Results

After agent returns, check `investigation.md` exists. If missing: **"Investigator failed. [retry / investigate manually]"**

Read the report and check STATUS:

**If STATUS: success (root cause found + fix applied):**
- Present: root cause, fix, regression test
- Ask: **"Bug fixed. Validate with `/verify`? [yes / ship directly / stop]"**

**If STATUS: blocked (3-strike escalation):**
- Present: the 3 hypotheses and evidence gathered
- This is valuable information even though the bug isn't fixed yet.
- Ask:
  ```
  Investigator hit 3 dead ends. This may be architectural.
  What was learned: <summary of hypotheses>

  A) Provide more context — I know something the investigator doesn't
  B) Try a different angle — <suggest one based on the evidence>
  C) Escalate — this needs a human to look at it
  ```
  - **A**: Re-launch investigator with user's additional context. Counter resets.
  - **B**: Re-launch investigator with the suggested angle. Counter resets.
  - **C**: Save investigation.md as documentation. Exit.

**If STATUS: partial (blast radius warning):**
- Present: which files the fix would touch and why
- Ask: **"Fix touches X files. Acceptable? [yes, proceed / find smaller fix / stop]"**

Update state.md: add investigation entry.

Next: `/verify` (to validate the fix) or `/ship` (if confident)
