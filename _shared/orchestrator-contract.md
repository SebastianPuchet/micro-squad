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
careful: true  # optional — present only when /squad was invoked with --careful

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
| exploration.md | ~500 words | Current State + Affected Areas + Approaches |
| plan.md | ~600 words | Merged architecture + scout + steps |
| build-log.md | 1 row per commit | Table only |
| build-summary.md | ~300 words | Stats + blockers + deviations |
| review-eng.md | ~500 words | Findings list (Engineering lane) |
| review-devex.md | ~500 words | Findings list (DevEx lane) |
| review-design.md | ~500 words | Findings list (Design lane) |
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

### Decision Point Format (MANDATORY for all gates)
Every choice presented to the user — between phases, within phases, on retry/skip/abort prompts, on uncommitted changes, branch checks, push confirmation, scope selection — MUST use this structure:

```
<1-sentence context: where we are and what needs deciding>

Recommendation: <your pick> because <reason>.

A) <option> — effort: ~Xh human / ~Ym Claude
B) <option> — effort: ~Xh human / ~Ym Claude
C) <option> — effort: ~Xh human / ~Ym Claude
```

Rules:
- The 1-line context is required. Do not skip it.
- The Recommendation line is required. Take a position. Don't be neutral.
- Effort estimates use the dual scale: `~Xh human / ~Ym Claude`. If irrelevant (e.g., a yes/no with negligible cost), write `effort: trivial`.
- 2 options are acceptable when only two paths exist. Otherwise prefer 3.
- Bare prompts like "Continue? [yes/no]" are NOT acceptable inside a phase. Always supply context + recommendation.
- Use 2-4 options. The recommendation always names a specific letter/option, never just "A is best".

**Worked example:**
```
Build complete — 4 commits, 7 files changed, all tests green.

Recommendation: A (continue to /verify) because review catches issues cheaper than post-merge.

A) yes — run /verify — effort: ~15m Claude
B) inspect — review with git log/diff first — effort: ~10m human
C) skip to /ship — effort: trivial (risk: unreviewed)
```

---

## Flag Parsing

Skills that accept flags MUST follow this convention:

- **Boolean flag:** `--<name>` — present means `true`, absent means `false`. Example: `--careful`.
- **Assignment flag:** `--<name>=<value>` — value is everything after the first `=`. Example: `--scope=hold`.
- **Stripping rule:** strip ALL recognized flags from the user's message BEFORE treating the remainder as the task description. Whitespace collapses; quoting is preserved as-is.
- **Unknown flags:** if a token starts with `--` but is not recognized, do NOT pass it through as part of the task description. Surface it to the user via Decision Point: ignore / treat as task text / abort.
- **Position-independent:** flags may appear anywhere in the argument string. Order does not matter.

**Recognized flags (current):**

| Flag | Type | Owner | Effect |
|------|------|-------|--------|
| `--careful` | boolean | `/squad` | Sets `careful: true` in state.md; enables Careful Mode Protocol |

Adding a new flag: append a row above and document parsing in the owner skill's Initialization section.

---

## Careful Mode Protocol

State.md may include `careful: true` in its header. Set by `/squad --careful` at sprint start, or written manually before resuming.

When `careful: true`, every skill MUST:

1. **Pre-step checkpoint commits.** Before each major step (phase entry, agent launch, fix application), create a checkpoint commit:
   ```bash
   git commit --allow-empty -m "squad-checkpoint: <phase>"
   ```
   Use `--allow-empty` so checkpoints land even when nothing has changed yet.

2. **Blast-radius gate.** Any single change touching >5 files MUST stop and present a Decision Point: proceed / split into smaller commits / abort. No automatic continuation.

3. **No skip shortcuts.** Ignore any "[skip]" / `--skip` / "just do it" shortcut. Every gate requires an explicit `yes`. Treat any non-`yes` answer as stop.

4. **Fix Agent constraint.** One fix per checkpoint commit, not bundled.

5. **Checkpoint ownership.** Each phase skill (`/build`, `/verify`, `/ship`) owns its own phase-entry checkpoint. The `/squad` orchestrator NEVER duplicates them — it delegates to the sub-skill, which writes the single `squad-checkpoint: <phase>` commit at its init.

Skills detect careful mode by reading state.md header at initialization. If absent, default behavior applies.

### Careful Mode — Worked Example

**Blast-radius gate trigger (AskUserQuestion-style):**
```
Blast-radius gate: this change touches 8 files (>5 threshold).

Files:
  src/auth/session.ts
  src/auth/middleware.ts
  src/api/login.ts
  src/api/logout.ts
  src/db/schema.sql
  tests/auth.test.ts
  tests/api.test.ts
  docs/auth.md

Recommendation: B (split) because 8 files crosses the careful-mode threshold and a single revert would lose unrelated work.

A) proceed — apply all 8 in one commit — effort: trivial (risk: hard revert)
B) split — break into 3 smaller commits by area (auth / api / docs+tests) — effort: ~5m Claude
C) abort — drop the change, replan — effort: trivial
```

**Resulting checkpoint commit log:**
```
$ git log --oneline
a1b2c3d squad-checkpoint: ship
9e8f7a6 squad: update auth docs
5d4c3b2 squad: wire login/logout endpoints
1a2b3c4 squad: refactor session middleware
f0e1d2c squad-checkpoint: build
b9a8c7d squad-checkpoint: plan
e6f5d4c squad-checkpoint: think
```
Note: each phase entry has its own checkpoint, and each fix lands as its own commit (no bundling).

---

## Verify Verdict Matrix

`/verify` runs three review lanes (Engineering, DevEx, Design) plus QA — 4 agents in parallel. Synthesis builds a consensus table with columns: Eng | DevEx | Design | QA | Verdict.

**Verdict rules:**
- **FIX** — Found by 2+ lanes (any combination of YES/FAIL across Eng, DevEx, Design, QA).
- **FIX** — ANY single CRITICAL finding from any lane. CRITICAL is never dismissed.
- **TRIAGE** — Found by exactly 1 lane, severity WARNING or SUGGESTION. Present to user.
- **DISMISS** — SUGGESTION explicitly contradicted by another lane.

**Minimum to synthesize:** 3 of 4 reports. If two or more lanes fail to write, abort synthesis and report agent failures.

**Design lane fallback:** When a diff has no UI/UX surface, the Design lane reviews API shape, output structure, error message clarity, and doc structure. Design NEVER returns CLEAN by default; it must explicitly note "no UI surface in diff" if it makes that call.

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
