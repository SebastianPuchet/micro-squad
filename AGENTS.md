# micro-squad — Skill Index

| Command | Purpose |
|---------|---------|
| `/squad` | Full sprint: think > plan > build > verify > ship |
| `/think` | Forcing questions, alternatives, scope mode selection |
| `/plan` | Parallel architect + scout → unified implementation plan |
| `/build` | Atomic commits, auto-revert, 3-strike escalation |
| `/verify` | Judgment day: dual blind judges + QA + fix loop |
| `/secure` | Infrastructure-first security audit |
| `/investigate` | 3-strike root cause debugging |
| `/ship` | PR with full evidence trail |
| `/retro` | Sprint retrospective — git stats, patterns, learnings |

## Shared Files

| File | Purpose |
|------|---------|
| `_shared/orchestrator-contract.md` | Sprint init, state machine, artifact contract, delegation rules, user control |
| `_shared/agent-prompts.md` | Prompt templates for all sub-agents (architect, scout, builder, judges, QA, fix, security, investigator) |

## Phase Dependencies

```
think → plan → build → verify → ship
                  ↓        ↑
                secure (optional, after build)

investigate (standalone, any time)
retro (standalone, any time)
```

## Sub-Agents

| Agent | Launched by | Runs in parallel | Writes |
|-------|-----------|:----------------:|--------|
| Architect | /plan | yes (with Scout) | architecture.md |
| Scout | /plan | yes (with Architect) | scout-report.md |
| Builder | /build | no | build-log.md, build-summary.md |
| Judge A | /verify | yes (with B + QA) | review-a.md |
| Judge B | /verify | yes (with A + QA) | review-b.md |
| QA | /verify | yes (with A + B) | qa-report.md |
| Fix Agent | /verify | no | (commits directly) |
| Security | /secure | no | security.md |
| Investigator | /investigate | no | investigation.md |
