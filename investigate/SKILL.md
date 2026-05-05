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

After agent returns, check `investigation.md` exists. If missing, present a Decision Point:
```
Investigator produced no output — likely an agent failure.

Recommendation: retry because the agent works on hypothesis testing and rarely fails twice.

A) retry — effort: ~5m Claude
B) investigate manually — effort: ~30m human
C) stop — effort: trivial
```

Read the report and check STATUS:

**If STATUS: success (root cause found + fix applied):**
Present root cause, fix, regression test. Decision Point:
```
Bug fixed: <one-line root cause>.

Recommendation: validate with /verify because a fix without review is a fresh bug in disguise.

A) yes — run /verify — effort: ~15m Claude
B) ship directly — effort: trivial (risk: unreviewed fix)
C) stop — effort: trivial
```

**If STATUS: blocked (3-strike escalation):**
Present the 3 hypotheses and evidence gathered. Decision Point:
```
Investigator hit 3 dead ends — may be architectural.
Learned: <summary of hypotheses>.

Recommendation: provide more context if you know something the investigator missed; otherwise escalate.

A) more context — re-launch with my additional info, counter resets — effort: ~5m human + ~10m Claude
B) try a different angle — <suggest one> — effort: ~10m Claude
C) escalate — save investigation.md as documentation, exit — effort: trivial
```

**If STATUS: partial (blast radius warning):**
Present which files the fix would touch and why. Decision Point:
```
Fix touches X files: <list>.

Recommendation: <proceed or find smaller — based on whether files are tightly coupled to the bug>.

A) yes, proceed — effort: as proposed
B) find smaller fix — effort: ~10m Claude
C) stop — effort: trivial
```

Update state.md: add investigation entry.

Next: `/verify` (to validate the fix) or `/ship` (if confident)
