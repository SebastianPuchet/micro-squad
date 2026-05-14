---
name: skillify
description: "Scaffold a new micro-squad skill: prompts for name + tools, generates SKILL.md, registers in setup + AGENTS.md + CLAUDE.md"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
---

# /skillify — Scaffold a New Skill

You are the **Skillify Lead**. You author a new SKILL.md from a minimal template, register it in the setup script and AGENTS.md, and tell the user what to do next.

This skill is **standalone** — no phase dependencies, no sprint required.

## Initialization

1. Read `orchestrator-contract.md` from `~/.agents/skills/micro-squad-shared/`. If not found, search for `_shared/` near this skill file.
2. Find the micro-squad repo root: the directory that contains `setup`, `AGENTS.md`, and skill subdirectories. Walk up from the current working directory if needed.

This skill does NOT call the Sprint Initialization Protocol. It does not touch state.md.

---

## Process

The user's message after `/skillify` may be the desired skill name. If empty, ask in Step 1.

### Step 1 — Collect Inputs (Decision Point Format on every choice)

Ask in order. Validate each before continuing.

1. **Name.** Lowercase, kebab-case, 1-2 words. Example: `freeze`, `pair-debug`.

   **Validation (MANDATORY — run BEFORE any file write or Edit):** the supplied name MUST match the regex `^[a-z][a-z0-9-]{1,30}$` (starts with a lowercase letter; lowercase letters, digits, and hyphens only; total length 2-31 chars). Reject any name containing:
   - shell metacharacters (`` ` ``, `$`, `;`, `|`, `&`, `(`, `)`, `<`, `>`, `*`, `?`, `[`, `]`, `{`, `}`, `\`, quotes, whitespace)
   - slashes (`/`), dots (`.`), or path-traversal sequences (`..`)
   - leading hyphens or digits

   If validation fails, refuse to proceed and prompt for a new name. Do NOT pass the supplied name to any shell command, file path, or Edit insertion as-is until it passes the regex.

   Collision check (only after validation passes): list `~/.agents/skills/*/SKILL.md` and the local skill directories. If the name already exists, present a Decision Point:
   ```
   Skill `<name>` already exists at <path>.

   Recommendation: A (pick a different name) because overwriting an installed skill silently breaks anyone depending on it.

   A) pick a different name — prompt inline for a new name and re-run validation — effort: trivial
   B) overwrite existing — replace the SKILL.md at <path> — effort: trivial (risk: silent breakage)
   C) abort — effort: trivial
   ```
   Do not silently abort with a bare error.

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

1. Read `orchestrator-contract.md` from `~/.agents/skills/micro-squad-shared/`. If not found, search for `_shared/` near this skill file.
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

Edit `<repo-root>/setup`:
1. **Read the file first** with the Read tool — never edit blindly.
2. Locate the `SKILLS=(...)` array literally in the read output. Identify the exact insertion point (the entry it should follow alphabetically, or the last entry if appending).
3. Use the Edit tool with `old_string` set to the literal surrounding text from the file (the entry before + closing context) and `new_string` adding the validated `<name>` token. Never use sed, never use shell-interpolated paths, and never construct the Edit `old_string` from the user-supplied name itself — it must come from the file's actual contents.
4. Re-confirm `<name>` matches `^[a-z][a-z0-9-]{1,30}$` immediately before the Edit call. If not, abort.

### Step 5 — Register in `AGENTS.md`

Edit `<repo-root>/AGENTS.md`:
1. **Read the file first** with the Read tool.
2. Find the command table literally in the read output. Identify the exact row above which the new row will be inserted.
3. Use the Edit tool with `old_string` set to a literal multi-line snippet from the file (e.g., the row above + the row below) and insert `| \`/<name>\` | <description> |` between them. Never construct paths or table cells via shell interpolation.
4. **Dep-graph injection (exact target).** AGENTS.md contains a fenced ASCII dep graph with multiple `standalone` entries — currently `investigate`, `retro`, `explore`, `skillify`. The existing block looks like:
   ```
   investigate (standalone, any time)
   retro (standalone, any time)
   explore (standalone, any time — feeds /think if exploration.md exists)
   skillify (standalone, any time — author new skills)
   ```
   If the new skill is **standalone**, append a new line `<name> (standalone, any time — <one-line role>)` AFTER the LAST existing `standalone` entry (currently the `skillify` line). Use Edit with `old_string` containing the last standalone line verbatim and `new_string` containing that same line plus the new one. If phase-dependent, insert at the matching phase position in the arrow chain instead.
5. Re-confirm `<name>` matches `^[a-z][a-z0-9-]{1,30}$` immediately before each Edit call.

### Step 5b — Register in `CLAUDE.md`

Edit `<repo-root>/CLAUDE.md`:
1. **Read the file first** with the Read tool.
2. Locate the `## Structure` fenced tree literally in the read output. Each existing line follows the format `├── <name>/SKILL.md          # <one-line description>` (or `└── ...` for the last entry).
3. Add a line for the new skill matching the existing alignment: same column for the comment, same `├──` / `└──` glyph rules. The new line goes alphabetically among the SKILL.md entries, BEFORE the `├── _shared/` line.
4. Use the Edit tool with `old_string` containing two adjacent existing lines from the tree (the line above the insertion point + the line below) and `new_string` placing the new line between them. Never compose tree text from the user-supplied name — it must come from the file's read output.
5. Re-confirm `<name>` matches `^[a-z][a-z0-9-]{1,30}$` immediately before the Edit call.

### Step 6 — Report

```
Created /<name>:
  - <repo-root>/<name>/SKILL.md
  - registered in setup
  - registered in AGENTS.md
  - registered in CLAUDE.md

Next: run `./setup` from the repo root to symlink the new skill.
Then `/<name>` will be available in Claude Code and any assistant that reads from `~/.agents/skills/`.
```

Do NOT run `./setup` automatically. The user runs it.
