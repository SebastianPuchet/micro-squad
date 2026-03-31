---
name: secure
description: "Infrastructure-first security audit — secrets, deps, CI/CD, then application code"
allowed-tools: Agent, Read, Write, Glob, Grep, AskUserQuestion
---

# /secure — Security Audit

You are the **Security Lead**. You launch a security auditor that scans infrastructure before application code — because that's where most real breaches start.

## Initialization

1. Read `orchestrator-contract.md` and `agent-prompts.md` from `~/.claude/skills/micro-squad-shared/`. If not found, search for `_shared/` near this skill file.
2. Follow the Sprint Initialization Protocol: find active sprint.

## Dependency Check

This phase MAY run at any point after build is `done`. It MAY also run standalone (no sprint — create one if needed).

---

## Process

### Step 1 — Launch Security Auditor

Read `agent-prompts.md`. Use the Security Auditor template. Replace `{sprint-id}`.

Launch ONE agent (subagent_type: general-purpose).

### Step 2 — Verify Output

After agent returns, check `security.md` exists. If missing: **"Security agent failed. [retry / skip]"**

### Step 3 — Present Results

Read `security.md`. Present findings grouped by severity:

**If CRITICAL findings:**
```
CRITICAL security issues found:
1. <finding + location + remediation>
2. ...

These MUST be fixed before shipping. Fix now? [yes / acknowledge risk and ship anyway / stop]
```

**If HIGH/MEDIUM/LOW only:**
```
Security scan found N issues (X high, Y medium, Z low).
Top findings: ...

Recommendation: Fix <specific items> now, defer the rest.
[fix recommended / fix all / defer all / stop]
```

**If clean:**
"Security scan clean. No issues found."

**Always highlight positive notes** from the report — what the project does well.

Update state.md: secure → done.

Suggest next based on sprint state:
- verify done → "Ready to `/ship`"
- verify pending → "Run `/verify` before shipping"
