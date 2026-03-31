# Orchestrator Contract

Rules that ALL micro-squad skills MUST follow. Read this before doing anything else.

---

## Sprint Initialization Protocol

Every skill MUST run these steps at the start:

### 1. Find Active Sprint
```
Search for .squad/*/state.md files.
- If ONE exists with pending phases → resume it (read state.md for context)
- If MULTIPLE exist → show list, ask user which to resume or start new
- If NONE exist → create new sprint (only if task description provided)
```

### 2. Create Sprint Directory (new sprints only)
- Format: `.squad/YYYY-MM-DD-<slug>/` where slug is 2-4 words from the task, kebab-case
- Create `state.md` with all phases set to `pending`
- If a sprint with the same slug exists today, append `-2`, `-3`, etc.

### 3. Detect Project Context (first phase only)
Run this once per sprint and cache results in `state.md`:
```bash
# Base branch detection
git branch -r | grep -E 'origin/(main|master|develop|trunk)' | head -1 | sed 's|origin/||' | tr -d ' '
# Fallback: ask user
```
```bash
# Test command detection (check in order)
grep -q '"test"' package.json 2>/dev/null && echo "npm test"
[ -f pytest.ini ] || [ -f pyproject.toml ] && echo "pytest"
[ -f Makefile ] && grep -q '^test:' Makefile && echo "make test"
# Fallback: ask user, or mark as "no test command detected"
```

Store in state.md header:
```markdown
base_branch: main
test_command: npm test  # or "none detected"
```

---

## State Machine

### Valid Phase Transitions

```
pending → running → done
pending → skipped (user explicitly chose to skip)
running → blocked (agent reported blocker)
blocked → running (user provided resolution)
```

A phase MAY only start if its dependencies are `done` or `skipped`:

| Phase | Dependencies |
|-------|-------------|
| think | none |
| plan | think |
| build | plan |
| verify | build |
| secure | build |
| ship | build + (verify OR verify:skipped) |
| investigate | none (standalone) |
| retro | none (standalone) |

If a dependency is not met, tell the user: **"Cannot run /X — /Y must complete first. Run `/Y` or `/squad` to continue."**

### State File Format

`.squad/<sprint-id>/state.md`:
```markdown
# Sprint: <description>
Started: <ISO date>
Base branch: <detected>
Test command: <detected or "none">

| Phase | Status | Artifact | Updated |
|-------|--------|----------|---------|
| think | done | think.md | 2026-03-27T10:00 |
| plan | done | plan.md | 2026-03-27T10:15 |
| build | running | build-log.md | 2026-03-27T10:30 |
| verify | pending | — | — |
| secure | pending | — | — |
| ship | pending | — | — |
```

---

## Artifact Contract

### Naming
All artifacts live in `.squad/<sprint-id>/`. Names are fixed — never rename or move them.

### Ownership
Each phase writes ONLY its own artifacts. NEVER modify another phase's output.

### Size Budgets
Keep artifacts concise. Bloated artifacts waste tokens for downstream phases.

| Artifact | Max Size | Format |
|----------|----------|--------|
| think.md | ~400 words | Problem + Q&A + approaches + decision |
| architecture.md | ~800 words | Data flow + file changes + edge cases |
| scout-report.md | ~600 words | Patterns + reusable code + risks |
| plan.md | ~600 words | Merged architecture + scout + steps |
| build-log.md | 1 row per commit | Table only |
| build-summary.md | ~300 words | Stats + blockers + deviations |
| review-a.md / review-b.md | ~500 words each | Findings list |
| qa-report.md | ~400 words | Test table + summary |
| verdict.md | ~400 words | Consensus table + status |
| security.md | ~600 words | Findings table + summary |
| investigation.md | ~500 words | Hypotheses + root cause + fix |
| retro.md | ~400 words | Stats + hotspots + patterns + improvements |

### Missing Artifact Handling
If a required artifact doesn't exist:
1. Check if the producing phase is `done` in state.md — if yes, the file was lost. Re-run that phase.
2. If the phase is `pending` or `skipped`, tell the user which phase to run.
3. NEVER proceed with missing required artifacts. NEVER fabricate content.

---

## Learnings

If `.squad/learnings.md` exists, read it for past findings. This file captures key findings from past sprints — things that went wrong, patterns discovered, edge cases hit.

- `/verify` appends 1-3 findings after each sprint review.
- Format: `- YYYY-MM-DD <sprint-slug>: <finding>`
- Cap: 50 entries max. When appending would exceed 50, trim the oldest 10.
- If the file doesn't exist, the first `/verify` run creates it.

Agents are not required to act on every learning — they provide context, not mandates.

---

## Agent Delegation Rules

### Launch Protocol
1. Read `agent-prompts.md` for the template
2. Replace ALL placeholders: `{sprint-id}`, `{base-branch}`, `{issues}`
3. Launch via the Agent tool with the fully interpolated prompt
4. For parallel agents: launch ALL in a single message (multiple Agent tool calls)

### Agent Failure Handling
If an agent returns without writing its expected artifact:
1. Check if the artifact file exists
2. If missing: report to user — **"Agent failed to produce <artifact>. Options: [retry / skip / stop]"**
3. If file exists but empty or malformed: same as above
4. NEVER silently continue with missing output

### Return Expectations
Agents SHOULD end their output with a status line:
```
STATUS: success | partial | blocked
```
- **success** — all work complete, artifact written
- **partial** — some work done, blockers noted in artifact
- **blocked** — could not proceed, reason in artifact

---

## User Control Protocol

### Between Phases
After every phase completion, ask the user using this format:

```
<phase> complete. <1-sentence summary of what was produced>

Continue to <next phase>? [yes / adjust / skip / stop]
```

- **yes** → proceed to next phase
- **adjust** → ask what to change, re-run current phase with modifications
- **skip** → mark next phase as `skipped`, move to the one after
- **stop** → save state, exit. User can resume later with `/squad` or individual phase commands

### Decision Points (AskUserQuestion Format)
When presenting choices within a phase, use this structure:

```
<1-sentence context: where we are and what needs deciding>

Recommendation: <your pick> because <reason>.

A) <option> — effort: ~Xh human / ~Ym Claude
B) <option> — effort: ~Xh human / ~Ym Claude
C) <option> — effort: ~Xh human / ~Ym Claude
```

Take a position. Recommend the best option. Don't be neutral.

---

## Ethos

- **Boil the Lake** — When 100% costs minutes more than 90%, do 100%. Completeness is cheap with AI.
- **Search Before Building** — Check what exists (tried-and-true, new-and-popular, first-principles) before designing from scratch.
- **User Sovereignty** — AI recommends, users decide. Always. Even when two models agree.

Read `ETHOS.md` for full context.

---

## Principles

1. **Anti-sycophancy** — Take positions. Say "this is wrong" not "this could be improved."
2. **Evidence-backed** — Every claim in the PR body traces to an artifact.
3. **Atomic commits** — One logical change per commit. Message: `squad: <what>`.
4. **No guessing** — If you don't know, investigate or ask. Never assume.
5. **Fail loud** — Surface problems immediately. Never bury errors.

---

## Voice

Direct, concrete, builder-oriented. Name the file, the function, the line number.
No corporate language. No hedging. No "it might be worth considering."
Say what it does, why it matters, what to do next.
