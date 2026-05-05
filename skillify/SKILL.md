---
name: skillify
description: "Scaffold a new micro-squad skill — generates SKILL.md, registers in setup and AGENTS.md"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
---

# /skillify — Scaffold a New Skill

You are the **Skillify Lead**. You author a new SKILL.md from a minimal template, register it in the setup script and AGENTS.md, and tell the user what to do next.

This skill is **standalone** — no phase dependencies, no sprint required.

## Initialization

1. Read `orchestrator-contract.md` from `~/.claude/skills/micro-squad-shared/`. If not found, search for `_shared/` near this skill file.
2. Find the micro-squad repo root: the directory that contains `setup`, `AGENTS.md`, and skill subdirectories. Walk up from the current working directory if needed.

This skill does NOT call the Sprint Initialization Protocol. It does not touch state.md.

---

## Process

The user's message after `/skillify` may be the desired skill name. If empty, ask in Step 1.

### Step 1 — Collect Inputs (Decision Point Format on every choice)

Ask in order. Validate each before continuing.

1. **Name.** Lowercase, kebab-case, 1-2 words. Example: `freeze`, `pair-debug`.
   Collision check: list `~/.claude/skills/*/SKILL.md` and the local skill directories. If the name already exists, abort:
   ```
   Skill `<name>` already exists at <path>. Pick a new name.
   ```

2. **Description.** One line, concrete. Voice: direct, builder-oriented.

3. **Allowed-tools list.** Comma-separated. Recommend defaults:
   ```
   You're scaffolding `<name>`. What tools does it need?

   Recommendation: Read, Write, Bash, Glob, Grep, AskUserQuestion because most skills do file I/O + ask user.

   A) minimal — Read, AskUserQuestion — effort: trivial
   B) standard — Read, Write, Bash, Glob, Grep, AskUserQuestion — effort: trivial
   C) with agents — Agent, Read, Write, Bash, Glob, Grep, AskUserQuestion — effort: trivial
   ```

4. **Dependency.** Where in the phase graph?
   ```
   When can this skill run?

   Recommendation: standalone unless it consumes another phase's artifact.

   A) standalone — runs any time, no dependency
   B) after build — needs build artifacts (like /verify, /secure)
   C) after a specific phase — name it
   ```

### Step 2 — Pre-Write Validation

Build the frontmatter. Validate:
- `name` matches the directory name you'll create (kebab-case)
- `description` is one line, no newlines
- `allowed-tools` is a non-empty comma-separated list
- If any invalid: show the user what's wrong, return to Step 1 for that field.

### Step 3 — Generate SKILL.md

Create `<repo-root>/<name>/SKILL.md` from this template (substitute fields):

```markdown
---
name: <name>
description: "<description>"
allowed-tools: <allowed-tools>
---

# /<name> — <Title Case>

You are the **<Name> Lead**. <One-line role.>

## Initialization

1. Read `orchestrator-contract.md` from `~/.claude/skills/micro-squad-shared/`. If not found, search for `_shared/` near this skill file.
2. <Sprint Initialization Protocol if needed, otherwise note "standalone — no sprint required".>

## Dependency Check

<Specify required phases or "none — standalone".>

---

## Process

### Step 1 — <First step>

<What this step does.>

### Step 2 — <Next step>

<...>

### Step N — Report

<What the user sees at the end.>
```

If the user picked a dependency in Step 1, fill the Dependency Check section accordingly. Otherwise write `none — standalone`.

### Step 4 — Register in `setup`

Edit `<repo-root>/setup`. Find the `SKILLS=(...)` line. Append `<name>` to the array, preserving alphabetical-or-existing order. Use the Edit tool, not sed.

### Step 5 — Register in `AGENTS.md`

Edit `<repo-root>/AGENTS.md`:
1. Add a row to the command table: `| \`/<name>\` | <description> |`
2. If standalone, add to the dep graph under the standalone line. If phase-dependent, add to the appropriate phase position.

### Step 6 — Report

```
Created /<name>:
  - <repo-root>/<name>/SKILL.md
  - registered in setup
  - registered in AGENTS.md

Next: run `./setup` from the repo root to symlink the new skill.
Then `/<name>` will be available in Claude Code.
```

Do NOT run `./setup` automatically. The user runs it.
