# CLAUDE.md — micro-squad

## What This Is

micro-squad is a drop-in multi-agent sprint system for Claude Code. It coordinates sub-agents through think, plan, build, verify, and ship phases using only Markdown skill files and Claude Code's native Agent tool.

No binaries. No servers. No external dependencies.

## Structure

```
micro-squad/
├── squad/SKILL.md          # Full sprint orchestrator
├── think/SKILL.md          # Forcing questions + scope
├── plan/SKILL.md           # Parallel architect + scout
├── build/SKILL.md          # Builder with guardrails
├── verify/SKILL.md         # Dual blind judges + QA
├── secure/SKILL.md         # Security audit
├── investigate/SKILL.md    # Root cause debugging
├── ship/SKILL.md           # PR with evidence trail
├── retro/SKILL.md          # Sprint retrospective
├── _shared/
│   ├── orchestrator-contract.md  # Rules all skills follow
│   └── agent-prompts.md         # Sub-agent prompt templates
├── ETHOS.md                # Builder philosophy
├── AGENTS.md               # Skill index + dependency graph
└── setup                   # Installs all skills
```

## Conventions

- **Markdown only.** No binaries, no compiled assets, no runtime dependencies.
- **Artifact budgets.** Each phase has a word limit (see orchestrator-contract.md). Bloated output wastes tokens downstream.
- **Atomic commits.** One logical change per commit. Message format: `squad: <what>`.
- **Frontmatter format.** Every SKILL.md starts with YAML frontmatter: `name`, `description`, `allowed-tools`.
- **Voice.** Direct, concrete, builder-oriented. Name the file, the function, the line. No hedging.

## Adding a New Skill

1. Create `<skill-name>/SKILL.md` with frontmatter matching existing skills.
2. Follow the pattern: Initialization (read contract) > Dependency Check > Process (numbered steps).
3. Add agent prompt templates to `_shared/agent-prompts.md` if the skill launches sub-agents.
4. Add the skill to `setup` (SKILLS array) and `AGENTS.md` (command table + dependency graph).
5. If standalone (no phase dependencies), note that in AGENTS.md like `/investigate`.

## Testing

No test suite. This project is Markdown files only. Validation is manual: install with `./setup`, run a skill, verify it works.
