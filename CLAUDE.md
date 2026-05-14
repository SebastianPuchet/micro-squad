# CLAUDE.md — micro-squad

## What This Is

micro-squad is a drop-in multi-agent sprint system for AI coding assistants (Claude Code, GitHub Copilot). It coordinates sub-agents through think, plan, build, verify, and ship phases using only Markdown skill files and native agent delegation.

No binaries. No servers. No external dependencies.

## Structure

```
micro-squad/                # Sources (clone anywhere; default ~/.agents/skills/micro-squad)
├── squad/SKILL.md          # Full sprint orchestrator
├── think/SKILL.md          # Forcing questions + scope
├── plan/SKILL.md           # Parallel architect + scout
├── build/SKILL.md          # Builder with guardrails
├── verify/SKILL.md         # Eng + DevEx + Design lanes + QA
├── secure/SKILL.md         # Security audit
├── investigate/SKILL.md    # Root cause debugging
├── explore/SKILL.md        # Standalone reconnaissance
├── ship/SKILL.md           # PR with evidence trail
├── retro/SKILL.md          # Sprint retrospective
├── skillify/SKILL.md       # Scaffold a new skill
├── _shared/
│   ├── orchestrator-contract.md  # Rules all skills follow
│   └── agent-prompts.md         # Sub-agent prompt templates
├── ETHOS.md                # Builder philosophy
├── AGENTS.md               # Skill index + dependency graph
└── setup                   # Installs all skills

# Sprint artifacts live OUTSIDE this repo by default:
~/.agents/squad-artifacts/<repo-id>/<sprint-id>/
~/.agents/squad-artifacts/<repo-id>/learnings.md    # per-repo
```

## Conventions

- **Markdown only.** No binaries, no compiled assets, no runtime dependencies.
- **Vendor-neutral install.** Skills install to `~/.agents/skills/` — read by both Claude Code and GitHub Copilot. Use `./setup --claude` / `--copilot` to also symlink into vendor-specific paths.
- **Artifacts external by default.** Sprints land in `~/.agents/squad-artifacts/<repo-id>/`. Override with `SQUAD_DIR=/path/to/root`. Never write inside the user's working repo.
- **Artifact budgets.** Each phase has a word limit (see orchestrator-contract.md). Bloated output wastes tokens downstream.
- **Atomic commits.** One logical change per commit. Message format: `squad: <what>`.
- **Frontmatter format.** Every SKILL.md starts with YAML frontmatter: `name`, `description`, `allowed-tools`.
- **Voice.** Direct, concrete, builder-oriented. Name the file, the function, the line. No hedging.

## Adding a New Skill

Recommended path: run `/skillify` — it scaffolds the SKILL.md and registers the skill in `setup`, `AGENTS.md`, and this file's Structure tree automatically. Manual steps below are the fallback.

1. Create `<skill-name>/SKILL.md` with frontmatter matching existing skills.
2. Follow the pattern: Initialization (read contract) > Dependency Check > Process (numbered steps).
3. Add agent prompt templates to `_shared/agent-prompts.md` if the skill launches sub-agents.
4. Add the skill to `setup` (SKILLS array) and `AGENTS.md` (command table + dependency graph).
5. If standalone (no phase dependencies), note that in AGENTS.md like `/investigate`.

## Testing

No test suite. This project is Markdown files only. Validation is manual: install with `./setup`, run a skill, verify it works.
