---
name: verify
description: "Judgment day — Eng + DevEx + Design review lanes + QA in parallel, consensus synthesis, fix loop"
allowed-tools: Agent, Read, Write, Bash, Glob, Grep, AskUserQuestion
---

# /verify — Judgment Day

You are the **Verify Lead**. You run an adversarial review in three specialized lanes (Engineering, DevEx, Design) plus QA — four agents in parallel, none aware of the others. You synthesize a consensus verdict and run a fix loop if needed.

## Initialization

1. Read `orchestrator-contract.md` and `agent-prompts.md` from `~/.agents/skills/micro-squad-shared/`. If not found, search for `_shared/` near this skill file.
2. Compute `$SQUAD_ROOT` per orchestrator-contract.md §Squad Dir Resolution before reading/writing artifacts.
3. Follow the Sprint Initialization Protocol: find active sprint, read state.md for base branch, test command, and `careful` flag.

## Dependency Check

Read state.md. Verify:
- build is `done` (REQUIRED)

Read `{squad-dir}/plan.md` and `{squad-dir}/build-log.md`. Both must exist.

If base branch is not in state.md, detect it:
```bash
git branch -r | grep -E 'origin/(main|master|develop|trunk)' | head -1 | sed 's|origin/||' | tr -d ' '
```
If detection fails, ask via Decision Point Format.

If `careful: true`, create a checkpoint commit before launching agents:
```bash
git commit --allow-empty -m "squad-checkpoint: verify"
```

---

## Round 1 — Launch Four Agents in Parallel

Read `agent-prompts.md` (the `## Review Lanes` section + the `## QA` section). Replace `{squad-dir}` (absolute path), `{sprint-id}`, `{base-branch}`, `{test-command}`.

Construct four prompts:
- **Eng-Review**: Lane: Engineering template. Append `"Write to: {squad-dir}/review-eng.md"`
- **DevEx-Review**: Lane: DevEx template. Append `"Write to: {squad-dir}/review-devex.md"`
- **Design-Review**: Lane: Design template. Append `"Write to: {squad-dir}/review-design.md"`
- **QA**: QA template. Append `"Write to: {squad-dir}/qa-report.md"`

Launch all FOUR in a single message (four Agent tool calls, all `subagent_type: general-purpose`).

## Verify Agent Output

After all four return, check that `review-eng.md`, `review-devex.md`, `review-design.md`, and `qa-report.md` exist.

You need at least 3 of 4 reports to synthesize. If two or more are missing, present a Decision Point:
```
<N> of 4 review reports missing: <list>.

Recommendation: retry the missing lanes because consensus needs at least 3.

A) retry missing lanes — effort: ~5m Claude
B) proceed with what we have — risk: weaker verdict — effort: trivial
C) stop — effort: trivial
```

If exactly one is missing, present a Decision Point:
```
1 of 4 review reports missing: <lane>. Consensus needs 2+ lanes for FIX, so a missing lane may weaken the verdict.

Recommendation: B (retry the missing lane) because consensus is weakened without it and a single retry is cheap.

A) proceed with 3 reports — note the gap in verdict.md — effort: trivial (risk: weaker verdict)
B) retry the missing lane — effort: ~3m Claude
C) stop — effort: trivial
```
Do not silently proceed.

## Synthesis — You Do This

Read all available reports. Map findings across lanes. Build the consensus table:

```markdown
## Judgment Day — Round 1

| # | Finding | Severity | Eng | DevEx | Design | QA | Verdict |
|---|---------|----------|:---:|:-----:|:------:|:--:|---------|
| 1 | Null check missing in auth.ts:42 | CRITICAL | YES | — | — | — | **FIX** |
| 2 | Race condition in submit handler | WARNING | YES | no | — | FAIL | **FIX** |
| 3 | Confusing flag name `--no-skip` | WARNING | — | YES | YES | — | **FIX** |
| 4 | Output table not aligned | SUGGESTION | — | — | YES | — | TRIAGE |
```

**Column values:**
- **YES** = lane found this issue
- **no** = lane reviewed this area and found no issue
- **FAIL** = QA test failed on this
- **—** = lane didn't examine this area

**Verdict rules** (from orchestrator-contract.md Verify Verdict Matrix):
- **FIX**: Found by 2+ lanes (any combination of YES/FAIL)
- **FIX**: ANY single CRITICAL finding — too dangerous to dismiss even with 1 lane
- **TRIAGE**: Found by exactly 1 lane, severity WARNING or SUGGESTION. Present to user.
- **DISMISS**: SUGGESTION explicitly contradicted by another lane.

## Fix Loop (Max 2 Rounds)

If FIX items exist:

1. Read Fix Agent template from agent-prompts.md. Replace `{issues}` with FIX items (finding, file, fix description). Replace `{test-command}`.
2. If `careful: true`: instruct Fix Agent to commit one fix per checkpoint, not bundled.
3. Launch Fix Agent (subagent_type: general-purpose).
4. After fixes, re-launch all four review agents in parallel for round 2.
5. Build round 2 consensus table.
6. If FIX items remain after round 2 → **ESCALATED**.

## Write Verdict

Write `{squad-dir}/verdict.md`:

```markdown
# Verdict: APPROVED | APPROVED WITH NOTES | ESCALATED

## Round 1
<consensus table with Eng | DevEx | Design | QA columns>
<summary: N findings, X fixed, Y triaged>

## Round 2 (if applicable)
<consensus table>

## Triage Items
<findings from 1 lane only — not blocking, but worth user awareness>

## Final Status
<APPROVED / APPROVED WITH NOTES / ESCALATED>
<1-2 sentence summary>
```

Update state.md: verify → done.

## Learning Capture

After writing verdict.md, extract 1-3 key findings from FIX and TRIAGE items:

1. Read verdict.md for FIX and TRIAGE items.
2. For each actionable finding, format as: `- YYYY-MM-DD <sprint-slug>: <finding>`
3. Append to `{squad-dir}/../learnings.md` (the per-repo learnings file, one level above the sprint dir).
4. If the file doesn't exist, create it with `# Learnings` followed by a blank line.
5. If the file exceeds 50 entries after appending, trim the oldest 10.

**Present verdict** using Decision Point Format:

- **APPROVED**:
  ```
  Verdict: APPROVED — all four lanes clean.

  Recommendation: ship because review found no blockers.

  A) ship now — effort: trivial
  B) run /secure first — effort: ~3m Claude
  C) stop — effort: trivial
  ```

- **APPROVED WITH NOTES**:
  ```
  Verdict: APPROVED WITH NOTES. Triage items: <list>.

  Recommendation: ship and address triage in a follow-up because nothing is blocking.

  A) ship — effort: trivial
  B) fix triage items now — effort: ~Ym Claude
  C) stop — effort: trivial
  ```

- **ESCALATED**:
  ```
  Verdict: ESCALATED. Unresolved: <list>.

  Recommendation: fix manually because two fix-agent rounds failed.

  A) fix manually — effort: ~Xh human
  B) retry verify with new context — effort: ~10m Claude
  C) stop — effort: trivial
  ```

Next: `/ship` or `/secure`
