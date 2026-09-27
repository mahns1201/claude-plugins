---
title: "audit procedure"
description: "Procedure that reads the project's Claude Code documents without writing and reports how they differ from the standard"
created_at: 2026-09-28
updated_at: 2026-09-28
---

# audit procedure

Writes no files. The output is one report and a suggested next mode.

## 1. Inventory

Build a table of whether each of the following exists and its line count.

- `CLAUDE.md`, `.claude/CLAUDE.md`, `CLAUDE.local.md`, `.claude/CLAUDE.local.md`
- `.claude/rules/*.md` (whether each has `paths`)
- `.claude/settings.json`, `.claude/settings.local.json`
- Files imported by `CLAUDE.md` with `@` (recursively)
- `CLAUDE.md` files in subdirectories
- `CLAUDE.local.md` and `.claude/settings.local.json` entries in `.gitignore`
- Claude-related `.md` files outside the standard structure: non-standard files under `.claude/`, `AGENTS.md`, `GEMINI.md`, `.cursor/rules/*.md`, external files pointed to by existing documents

## 2. Checks

The criteria are `${CLAUDE_PLUGIN_ROOT}/skills/setup/common/.claude/rules/CLAUDE_CODE.md` and `MARKDOWN.md`. Mark each item pass, warning, or violation.

| Item | Criterion | Effect when violated |
| --- | --- | --- |
| Root CLAUDE.md size | 50 lines or fewer. Three sections: overview, commands, rules reference guide | Context cost every session, lower instruction adherence |
| Duplicate CLAUDE.md | Only one of root and `.claude/CLAUDE.md` | Same content loaded twice |
| `@` imports | None | Imported files load in full at start |
| CLAUDE.local.md location | Root only. Not loaded from under `.claude/` | File is ignored |
| Always-loaded rules | Every rule has `paths`; a rule that applies everywhere uses `"**/*"`. Report total lines of rules without `paths` | Context cost at every session, and a forgotten `paths` looks the same as a deliberate one |
| `paths` quoting | Every value quoted | YAML parse failure, whole rule ignored |
| Rule frontmatter | `title`, `description`, `created_at`, `updated_at` | Human metadata missing |
| `env` in `settings.json` | None | Environment variables committed |
| Natural language in `settings.json` | None | Ignored |
| gitignore | Both entries present | Personal files committed |
| Forbidden syntax | No `**`, `~~`, `####`, HTML tags | MARKDOWN.md violation |
| Rules reference guide | Points only to rule files that exist | Broken references |

## 3. Content distribution

If existing documents are present, classify each section by kind of content. `migrate` uses this table as is.

| Kind | Examples | Place in the standard structure |
| --- | --- | --- |
| Overview, commands | Stack, module list, build commands | Root `CLAUDE.md` |
| Structure | Layers, dependency direction, package layout | `ARCHITECTURE.md`, module `CLAUDE.md` |
| Naming | Suffixes, casing | `NAMING.md` |
| Tests | Framework, structure | `TEST.md` |
| Review, done criteria | Checklists | `REVIEW.md` |
| Multi-step procedures | Deployment order, release steps | Out of scope. Report as skill candidates |
| Behavioral rules | Always ask, minimal change | Out of scope. Report only |
| Other | | Unplaced |

## 4. Report

- Show the inventory table, the checks table, and the content distribution table in order.
- If `.md` files exist outside the standard structure, show a table with path, size, which tool it appears to belong to, and kind of content. Mark whether each is a candidate for absorption in `migrate`.
- Suggest the next mode. No documents at all: `init`. Documents present: `migrate`. No violations and already standard: "no change needed".
- Write nothing until the user picks the next mode.
