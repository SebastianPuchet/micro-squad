---
name: verify
description: "Judgment day — dual blind judges + QA agent in parallel, consensus synthesis, fix loop"
allowed-tools: Agent, Read, Write, Bash, Glob, Grep, AskUserQuestion
---

# /verify — Judgment Day

You are the **Verify Lead**. You run an adversarial review: two independent code reviewers (each unaware the other exists) plus a QA agent, all in parallel. You synthesize a consensus verdict and run a fix loop if needed.

## Initialization

1. Read `orchestrator-contract.md` and `agent-prompts.md` from `~/.claude/skills/micro-squad-shared/`. If not found, search for `_shared/` near this skill file.
2. Follow the Sprint Initialization Protocol: find active sprint, read state.md for base branch and test command.

## Dependency Check

Read state.md. Verify:
- build is `done` (REQUIRED)

Read `.squad/<sprint-id>/plan.md` and `.squad/<sprint-id>/build-log.md`. Both must exist.

If base branch is not in state.md, detect it:
```bash
git branch -r | grep -E 'origin/(main|master|develop|trunk)' | head -1 | sed 's|origin/||' | tr -d ' '
```
If detection fails, ask: **"What's your base branch? [main / master / other]"**

---

## Round 1 — Launch Three Agents in Parallel

Read `agent-prompts.md`. Replace `{sprint-id}`, `{base-branch}`, `{test-command}`.

Construct three prompts from templates:
- **Judge A**: Prepend `"You are Judge A."` to the Judge template. Append `"Write to: .squad/<sprint-id>/review-a.md"`
- **Judge B**: Prepend `"You are Judge B."` to the Judge template. Append `"Write to: .squad/<sprint-id>/review-b.md"`
- **QA**: Append `"Write to: .squad/<sprint-id>/qa-report.md"` to the QA template

Launch all THREE in a single message (three Agent tool calls, all subagent_type: general-purpose).

## Verify Agent Output

After all three return, check that `review-a.md`, `review-b.md`, and `qa-report.md` exist.
- If a report is missing: **"<Agent> failed to produce output. [retry that agent / proceed without it]"**
- You need at least 2 of 3 reports to synthesize a verdict.

## Synthesis — You Do This

Read all available reports. Map findings across agents. Build the consensus table:

```markdown
## Judgment Day — Round 1

| # | Finding | Severity | Judge A | Judge B | QA | Verdict |
|---|---------|----------|:-------:|:-------:|:--:|---------|
| 1 | Null check missing in auth.ts:42 | CRITICAL | YES | YES | — | **FIX** |
| 2 | Race condition in submit handler | WARNING | YES | no | FAIL | **FIX** |
| 3 | Unused import in utils.ts | SUGGESTION | YES | no | — | TRIAGE |
```

**Column values:**
- **YES** = agent found this issue
- **no** = agent reviewed this area and found no issue
- **FAIL** = QA test failed on this
- **—** = agent didn't examine this area

**Verdict rules:**
- **FIX**: Found by 2+ agents (any combination of YES/FAIL)
- **FIX**: ANY single CRITICAL finding — too dangerous to dismiss even with 1 agent
- **TRIAGE**: Found by exactly 1 agent, severity is WARNING or SUGGESTION. Present to user.
- **DISMISS**: Severity SUGGESTION + explicitly contradicted by another agent.

## Fix Loop (Max 2 Rounds)

If FIX items exist:

1. Read Fix Agent template from agent-prompts.md. Replace `{issues}` with the FIX items (copy the finding, file, and fix description). Replace `{test-command}`.
2. Launch Fix Agent (subagent_type: general-purpose).
3. After fixes, re-launch all three agents (Judge A + Judge B + QA) in parallel for round 2.
4. Build round 2 consensus table.
5. If FIX items remain after round 2 → **ESCALATED**.

## Write Verdict

Write `.squad/<sprint-id>/verdict.md`:

```markdown
# Verdict: APPROVED | APPROVED WITH NOTES | ESCALATED

## Round 1
<consensus table>
<summary: N findings, X fixed, Y triaged>

## Round 2 (if applicable)
<consensus table>

## Triage Items
<findings from 1 agent only — not blocking, but worth user awareness>

## Final Status
<APPROVED / APPROVED WITH NOTES / ESCALATED>
<1-2 sentence summary>
```

Update state.md: verify → done.

## Learning Capture

After writing verdict.md, extract 1-3 key findings from FIX and TRIAGE items:

1. Read verdict.md for FIX and TRIAGE items.
2. For each actionable finding, format as: `- YYYY-MM-DD <sprint-slug>: <finding>`
3. Append to `.squad/learnings.md` in the project root.
4. If the file doesn't exist, create it with a header: `# Learnings` followed by a blank line.
5. If the file exceeds 50 entries after appending, trim the oldest 10.

Example entries:
```
- 2026-03-31 dark-mode: Judge prompts need scope-drift detection
- 2026-03-31 dark-mode: Builder missed error path in auth.ts — add edge case checklist
```

**Present verdict:**
- **APPROVED**: "Verdict: APPROVED. Ready to ship? [yes / secure first / stop]"
- **APPROVED WITH NOTES**: "Verdict: APPROVED WITH NOTES. Triage items to review: ... [ship / fix triage items / stop]"
- **ESCALATED**: "Verdict: ESCALATED. Unresolved issues: ... [fix manually / retry / stop]"

Next: `/ship` or `/secure`
