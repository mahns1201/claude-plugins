---
title: "Claude Code memory and settings rules"
description: "Single source of truth for what goes where in CLAUDE.md files and .claude/settings files"
created_at: 2026-09-27
updated_at: 2026-09-28
paths:
  - "**/CLAUDE.md"
  - "**/CLAUDE.local.md"
  - ".claude/settings.json"
  - ".claude/settings.local.json"
  - ".claude/rules/**"
---

# Claude Code memory and settings rules

## Placement principles

| Content | Location | Loaded |
| --- | --- | --- |
| Project overview and rule routing | `CLAUDE.md` | Session start |
| Personal notes, machine-specific paths | `CLAUDE.local.md` | Session start |
| Guidance that applies only to certain paths | `.claude/rules/*.md` + `paths` | When a matching file is read |
| Multi-step procedures, long reference material | `.claude/skills/<name>/SKILL.md` | When invoked |
| Tool permissions, hooks, environment variables | `.claude/settings*.json` | Session start |

- Natural-language instructions go in `CLAUDE.md` files. Machine-read configuration goes in `settings*.json`. Do not mix them.
- Every always-loaded file costs context. Move procedures and long material into skills.

## CLAUDE.md

Inside a project, `CLAUDE.md` exists in three forms.

| File | Location | Role | Loaded |
| --- | --- | --- | --- |
| `CLAUDE.md` | Project root | Team-shared guidance. Overview, commands, rule routing | Session start |
| `CLAUDE.local.md` | Project root | Personal guidance. Listed in `.gitignore` | Session start, after `CLAUDE.md` |
| `<directory>/CLAUDE.md` | Working subdirectory | Conventions valid only in that directory | When a file in that directory is read |

### Root CLAUDE.md

- Lives at `./CLAUDE.md`.
- No frontmatter.
- The heading names the role, not the file: `# Project CLAUDE.md` at the root, `# Local CLAUDE.md` for the personal file, `# <module> module CLAUDE.md` at a module root. Several of these load together, and the heading is how Claude tells them apart.
- Keep it thin. Every detailed rule lives in `.claude/rules/`. `CLAUDE.md` only carries the overview and the routing.
- Exactly three sections.
  - Project overview
    - One-line introduction (domain, language and runtime, framework, build tool)
    - Modules or top-level directories and their roles
    - Dependency direction
  - Commands: install, run, test, lint/format, build. One line each. Commands only, no explanation.
  - Rules reference guide: a two-column table mapping task types to rule files. This is where Claude finds which rule to read. Keep the task-type column to short noun phrases.
- When a rule is added or removed, update the Rules reference guide in the same change.
- Do not use `@path` imports. Point to other files by writing the path as text. Reason: files pulled in with `@` are loaded into context in full at session start, which defeats keeping the file thin. A text path is read only when Claude needs it.
- Loaded concatenated with the user-global `~/.claude/CLAUDE.md`.

Skeleton:

```markdown
# Project CLAUDE.md

## Project overview

- <one-line domain description> <language/runtime> / <framework> / <build tool>
- `<module or directory>` — <role>
- Dependency direction: `<A> → <B> → <C>` (one-way)

## Commands

- Install: `<command>`
- Test: `<command>`
- Build: `<command>`

## Rules reference guide

Always consult the matching `.claude/rules` file for the task type.

| Task type | Rule file |
| --- | --- |
| <task type> | `.claude/rules/<RULE>.md` |
```

### CLAUDE.local.md

- Lives at `./CLAUDE.local.md`.
- For personal guidance and notes: machine-specific paths, personal preferences, work-in-progress notes.
- Never committed.
- Loaded concatenated after `CLAUDE.md`.

### Working-directory CLAUDE.md

- A `CLAUDE.md` in a subdirectory is loaded when Claude reads a file in that directory.
- It is a companion to `.claude/rules/ARCHITECTURE.md`. ARCHITECTURE.md covers the whole structure such as the module list and dependency direction. How code is written inside each module belongs to that module's root `CLAUDE.md`.
- Place one at each module root only. Do not add one per subdirectory. Register the path in the module table of ARCHITECTURE.md so Claude can find it before reading any file there.
- Hold only conventions and pitfalls that matter inside that module. Do not repeat anything from the root `CLAUDE.md` or ARCHITECTURE.md.
- When a session starts in a subdirectory, the `CLAUDE.md` files of parent directories are also loaded at start.
- Do not create the file if there is nothing to put in it.

## .claude/rules

- The only frontmatter key Claude Code reads is `paths`. Everything else is metadata for humans.
- Always quote `paths` values. A value starting with `*` fails YAML parsing when unquoted.
- A rule without `paths` is loaded at every session start, like `CLAUDE.md`. Do not rely on that. A rule that applies everywhere declares `paths` as `"**/*"`, which loads it at the first file read and marks the choice as deliberate.
- A rule with `paths` loads when a matching file is read, not when a new file is only written. Rules about creating new files, such as naming, use `"**/*"` so they are in place once any file has been read.
- Rules are guidance Claude reads, not enforcement. Block actions that must never happen with `permissions.deny` in `settings.json` or with hooks.

## settings.json

### .claude/settings.json

- Team-shared settings. Committed.
- Holds `permissions.allow`, `permissions.deny`, `hooks`.
- Does not hold `env`. Environment variables go only in `settings.local.json`.
- No natural-language instructions. Those go in `CLAUDE.md`.
- Hooks live in `settings*.json`, in a plugin's `hooks/hooks.json`, or in the frontmatter of a skill or subagent. `CLAUDE.md` files cannot carry hooks.
- Array values such as `permissions.allow` are merged with other settings files, not overridden.

### .claude/settings.local.json

- Personal settings. Listed in `.gitignore`.
- Takes precedence over `settings.json` for the same key.
- Holds `env`, personal tool permissions, hooks used only on this machine.
- When you choose "always allow" in a permission prompt, Claude Code writes the rule here automatically.

### Precedence

Higher wins.

1. Managed (organization) settings
2. Command-line arguments
3. `.claude/settings.local.json`
4. `.claude/settings.json`
5. `~/.claude/settings.json`
