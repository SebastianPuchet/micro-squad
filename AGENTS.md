# micro-squad — Skill Index

**Install path:** `~/.agents/skills/` (vendor-neutral — read by both Claude Code and GitHub Copilot). Run `./setup --claude` / `--copilot` to also symlink into vendor-specific paths.

**Artifacts:** `~/.agents/squad-artifacts/<repo-id>/<sprint-id>/` by default. Override with `SQUAD_DIR=/path/to/root` in your shell.

| Command | Purpose |
|---------|---------|
| `/squad` | Full sprint: think > plan > build > verify > ship. Pass `--careful` for high-stakes mode. |
| `/think` | Forcing questions, alternatives, scope mode selection |
| `/plan` | Parallel architect + scout → unified implementation plan |
| `/build` | Atomic commits, auto-revert, 3-strike escalation |
| `/verify` | Judgment day: Eng + DevEx + Design review lanes + QA + fix loop |
| `/secure` | Infrastructure-first security audit |
| `/investigate` | 3-strike root cause debugging |
| `/ship` | PR with full evidence trail |
| `/retro` | Sprint retrospective — git stats, patterns, learnings |
| `/skillify` | Scaffold a new micro-squad skill — generates SKILL.md, registers in setup + AGENTS.md |
| `/explore` | Standalone reconnaissance — Current State / Affected Areas / Approaches for a topic |

## Careful Mode

Pass `--careful` to `/squad` (e.g., `/squad --careful migrate auth`) to enable high-stakes mode for the sprint. Effects:

- Pre-step `squad-checkpoint: <phase>` commits before each phase
- Blast-radius gate at >5 files — every change prompts proceed/split/abort
- No skip shortcuts — every gate requires explicit `yes`
- Fix Agent commits one fix per checkpoint, not bundled

The flag is recorded as `careful: true` in the sprint's `state.md` header. All skills read it at init.

## Shared Files

| File | Purpose |
|------|---------|
| `_shared/orchestrator-contract.md` | Sprint state machine, contract rules, gate formats |
| `_shared/agent-prompts.md` | Prompt templates for all sub-agents (architect, scout, builder, review lanes, QA, fix, security, investigator) |

## Phase Dependencies

```
think → plan → build → verify → ship
                  ↓        ↑
                secure (optional, after build)

investigate (standalone, any time)
retro (standalone, any time)
explore (standalone, any time — feeds /think if exploration.md exists)
skillify (standalone, any time — author new skills)
```

## Sub-Agents

| Agent | Launched by | Runs in parallel | Writes |
|-------|-----------|:----------------:|--------|
| Architect | /plan | yes (with Scout) | architecture.md |
| Scout | /plan | yes (with Architect) | scout-report.md |
| Builder | /build | no | build-log.md, build-summary.md |
| Eng-Review | /verify | yes (with DevEx + Design + QA) | review-eng.md |
| DevEx-Review | /verify | yes (with Eng + Design + QA) | review-devex.md |
| Design-Review | /verify | yes (with Eng + DevEx + QA) | review-design.md |
| QA | /verify | yes (with all lanes) | qa-report.md |
| Fix Agent | /verify | no | (commits directly) |
| Security | /secure | no | security.md |
| Investigator | /investigate | no | investigation.md |
