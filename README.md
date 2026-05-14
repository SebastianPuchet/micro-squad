# micro-squad

**Drop-in multi-agent sprint for Claude Code.**

No binaries. No servers. No external dependencies. Just Markdown skill files.

micro-squad turns Claude Code into a coordinated engineering team. An orchestrator delegates to parallel sub-agents that think, plan, build, review, and ship — using Claude Code's native Agent tool.

## What happens when you run `/squad`

```
You: /squad add dark mode support

THINK ─── Forcing questions, explore alternatives
          You pick scope: EXPAND / SELECTIVE / HOLD / REDUCE
              │
PLAN ─────── Architect ──┐ parallel    Unified plan with
              Scout ─────┘              effort estimates
              │
BUILD ─────── Builder implements with atomic commits
              Auto-reverts on regression, 3-strike escalation
              │
VERIFY ───── Eng ───────┐
              DevEx ─────┤ parallel    Consensus table:
              Design ────┤              FIX / TRIAGE / DISMISS
              QA Agent ──┘
              │
              Fix Agent → Re-review (max 2 rounds)
              │
SHIP ──────── Commit + PR with full evidence trail
```

## Why

AI makes completeness cheap. The last 10% that teams used to skip costs seconds now. micro-squad is built on three principles:

- **Boil the Lake** — When 100% costs minutes more than 90%, do 100%.
- **Search Before Building** — Check what exists before designing from scratch.
- **User Sovereignty** — AI recommends, users decide. Always.

Read [ETHOS.md](ETHOS.md) for the full philosophy.

## Key Features

- **Three review lanes** — Engineering, DevEx, Design — plus QA, all four parallel and blind
- **CRITICAL override** — A single CRITICAL finding is never dismissed, even by one lane
- **Real parallelism** — Architect + Scout, all four review agents launch simultaneously
- **Context isolation** — Each agent gets fresh context, reads only what it needs
- **Self-regulating builder** — Atomic commits, auto-revert on regression, 3-strike rule
- **Decision Point Format** — Every gate presents context + recommendation + options with effort
- **Careful mode** — `/squad --careful` enables checkpoints, blast-radius gates, no skip shortcuts
- **State machine** — Enforced phase dependencies, resumable sprints, skip/stop support
- **Artifact trail** — Everything in `~/.agents/squad-artifacts/<repo-id>/`, inspectable, diffable, outside your working repo by default
- **Artifact size budgets** — Agents produce concise output, no token bloat downstream
- **Learning loop** — `/verify` captures findings, future agents start smarter
- **Retrospectives** — `/retro` analyzes git stats, hotspots, and patterns
- **Standalone recon** — `/explore <topic>` maps a topic without committing to a sprint
- **Skill scaffolding** — `/skillify` generates a new SKILL.md and registers it
- **Agent failure handling** — Missing output detected, retry/skip options
- **User stays in control** — Confirmation between every phase, explicit scope modes

## Install

Vendor-neutral default — both Claude Code and GitHub Copilot read from `~/.agents/skills/`.

What `./setup` does by default:

1. Symlinks every skill's `SKILL.md` into `~/.agents/skills/` (read by both Claude Code and GitHub Copilot).
2. Creates `~/.agents/squad-artifacts/<repo-id>/` as the sprint artifact root (outside your working repo — nothing to `.gitignore`).
3. If an in-repo `.squad/` directory from a previous version exists, prompts you interactively to migrate it.

```bash
git clone https://github.com/SebastianPuchet/micro-squad.git ~/.agents/skills/micro-squad
cd ~/.agents/skills/micro-squad && ./setup
```

Run `./setup --help` to see all flags.

Flags:

- `./setup --claude` — also symlink into `~/.claude/skills/` (for Claude Code users who haven't migrated yet)
- `./setup --copilot` — also symlink into `~/.copilot/skills/`
- `./setup --no-migrate` — skip the in-repo `.squad/` migration prompt
- `./setup --yes` — assume yes to all prompts (migration: move; foreign-skill collision: overwrite)

### Uninstalling

```bash
cd ~/.agents/skills/micro-squad && ./uninstall
```

Removes all skill symlinks from `~/.agents/skills/`, `~/.claude/skills/`, and `~/.copilot/skills/` — only symlinks that point into this source directory. Sprint artifacts in `~/.agents/squad-artifacts/` are left untouched.

- `./uninstall --yes` — skip the confirmation prompt

### GitHub Copilot support

The `~/.agents/skills/` path is the vendor-neutral standard. GitHub Copilot's Agent Skills reads SKILL.md files from this directory with the same frontmatter Claude uses — no separate install needed. If you want belt-and-suspenders, run `./setup --copilot` to also symlink to `~/.copilot/skills/`.

### Where sprint artifacts live

By default: `~/.agents/squad-artifacts/<repo-id>/<sprint-id>/`. Outside your working repo — nothing to add to `.gitignore`.

Override with `SQUAD_DIR`:

```bash
export SQUAD_DIR=/path/to/your/artifact/root
```

When set, micro-squad writes all sprint artifacts under `$SQUAD_DIR/<sprint-id>/` instead of the per-repo default.

### How `<repo-id>` is derived

`<repo-id>` is the `basename` of `git rev-parse --show-toplevel` (or `basename $PWD` outside a git repo). On first sprint creation, the resolved git origin URL is written to `~/.agents/squad-artifacts/<repo-id>/.repo-origin`. On later runs, if a different repo with the same basename shows up (e.g. two repos both called `app`), the new repo gets a `-<sha7>` suffix derived from its origin URL so the two artifact roots don't collide.

## Commands

| Command | What it does |
|---------|-------------|
| `/squad <task>` | Full sprint — think, plan, build, verify, ship. Add `--careful` for high-stakes mode. |
| `/think <topic>` | Challenge assumptions with forcing questions |
| `/plan` | Parallel architect + scout → unified plan |
| `/build` | Implementation with guardrails |
| `/verify` | Judgment day: Eng + DevEx + Design lanes + QA + fix loop |
| `/secure` | Infrastructure-first security audit |
| `/investigate <bug>` | 3-strike root cause debugging |
| `/ship` | PR with full evidence trail |
| `/retro` | Sprint retrospective — git stats, patterns, learnings |
| `/explore <topic>` | Standalone reconnaissance — Current State / Affected Areas / Approaches |
| `/skillify <name>` | Scaffold a new micro-squad skill (SKILL.md + setup + AGENTS.md) |

Each command works standalone or as part of a full `/squad` sprint.

### Careful Mode

`/squad --careful <task>` flips the sprint into high-stakes mode:

- Pre-step `squad-checkpoint: <phase>` commits before each phase
- Blast-radius gate at >5 files — every change prompts proceed/split/abort
- No skip shortcuts — every gate requires explicit `yes`
- Fix Agent commits one fix per checkpoint, not bundled

The flag is recorded in `state.md`. All skills read it at init.

## Project Structure

```
micro-squad/
├── squad/SKILL.md              # Full sprint orchestrator
├── think/SKILL.md              # Forcing questions + scope selection
├── plan/SKILL.md               # Parallel architect + scout
├── build/SKILL.md              # Builder with guardrails
├── verify/SKILL.md             # Judgment day (dual judges + QA)
├── secure/SKILL.md             # Security audit
├── investigate/SKILL.md        # Root cause debugging
├── ship/SKILL.md               # PR with evidence
├── retro/SKILL.md              # Sprint retrospective
├── explore/SKILL.md            # Standalone reconnaissance
├── skillify/SKILL.md           # New skill scaffolder
├── _shared/
│   ├── orchestrator-contract.md  # State machine, artifact contract, rules
│   └── agent-prompts.md          # All sub-agent prompt templates
├── ETHOS.md                    # Builder philosophy
├── CLAUDE.md                   # Project context for Claude Code
├── AGENTS.md                   # Skill index + dependency graph
└── setup                       # Installs all skills
```

## Artifacts

Each sprint creates `~/.agents/squad-artifacts/<repo-id>/<sprint-id>/` (or `$SQUAD_DIR/<sprint-id>/` if overridden):

```
~/.agents/squad-artifacts/myrepo/2025-03-27-dark-mode/
├── state.md           # Phase tracking + project context (squad_dir + repo_id cached here)
├── think.md           # Problem, alternatives, scope decision
├── architecture.md    # Data flows, file changes, edge cases
├── scout-report.md    # Existing patterns, reusable code, risks
├── plan.md            # Unified plan with effort estimates
├── build-log.md       # Commit-by-commit progress
├── build-summary.md   # Final build status + blockers
├── review-eng.md      # Engineering lane findings
├── review-devex.md    # DevEx lane findings
├── review-design.md   # Design lane findings
├── qa-report.md       # Test results
├── verdict.md         # Consensus table + final verdict
├── security.md        # Security audit (optional)
├── investigation.md   # Root cause analysis (if debugging)
├── exploration.md     # /explore output (if run inside sprint)
└── retro.md           # Sprint retrospective (if run)
```

`learnings.md` lives one level up at `~/.agents/squad-artifacts/<repo-id>/learnings.md` — per-repo, shared across all sprints for that repo.

## The Judgment Day Pattern

Three specialized review lanes — Engineering, DevEx, Design — plus a QA agent run in parallel. None of the lanes knows the others exist. The orchestrator builds a consensus table:

| Finding | Severity | Eng | DevEx | Design | QA | Verdict |
|---------|----------|:---:|:-----:|:------:|:--:|---------|
| Null check missing | CRITICAL | YES | — | — | — | **FIX** |
| Race condition | WARNING | YES | no | — | FAIL | **FIX** |
| Confusing flag name | WARNING | — | YES | YES | — | **FIX** |
| Output table not aligned | SUGGESTION | — | — | YES | — | TRIAGE |

- **FIX**: Found by 2+ lanes, or ANY single CRITICAL finding
- **TRIAGE**: Found by 1 lane (non-critical) — user decides
- **DISMISS**: SUGGESTION explicitly contradicted by another lane

The Design lane never returns CLEAN by default — when the diff has no UI surface, it reviews API shape, output structure, error message clarity, and doc structure.

If FIX items exist, a Fix Agent addresses confirmed issues, then all four agents re-run (max 2 rounds). If still broken, escalated to user with full history.

## Ethos

micro-squad is opinionated about how to build with AI. Three principles guide every agent:

1. **Boil the Lake** — Completeness is cheap. Do 100% when it costs minutes more than 90%.
2. **Search Before Building** — Three layers: tried-and-true, new-and-popular, first-principles.
3. **User Sovereignty** — AI recommends, users decide. No exceptions.

See [ETHOS.md](ETHOS.md) for details.

## License

MIT
