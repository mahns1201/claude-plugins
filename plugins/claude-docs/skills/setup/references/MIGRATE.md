---
title: "migrate procedure"
description: "Procedure that absorbs existing Claude Code documents as analysis input and replaces them with the standard structure"
created_at: 2026-09-28
updated_at: 2026-09-28
---

# migrate procedure

Precondition: existing Claude documents are present. If none, stop and recommend `init`. Existing documents are not an obstacle; they are the first source of evidence. Read them before the codebase.

Principle: never discard content silently. Every section of the existing documents either moves to a new location or goes on the unplaced list for the user to decide.

## 1. Inventory and classification

Run sections 1 and 3 of `references/AUDIT.md`. The result is the list of existing files and the section-level classification table.

## 2. Present the plan and confirm once

Show the following as a table and get the user's confirmation once. After that, ask no further questions of your own. Claude Code's own permission prompts still appear: for writes under `.claude/`, for `cp` from the plugin directory, and for `rm` and `mv` in section 6. The first two kinds are safe to approve for the session; `rm` and `mv` should be approved one by one.

| Item | Content |
| --- | --- |
| Replace | Existing `CLAUDE.md` and `.claude/rules/*` become the new structure |
| Section moves | Existing section → new file, exactly as in the classification table |
| Unplaced | Sections with no destination. The user decides to drop them or where to put them |
| Out of scope | Multi-step procedures, behavioral rules. Reported only, not placed in the new structure |
| Originals | Git-tracked files are deleted (history keeps them). Untracked files, or no git repository: moved to `.claude/legacy/` |
| `settings.json` | Replaced by the common file. Existing `permissions` and `hooks` entries are listed here and in the report for the user to add back; an `env` block is listed as belonging in `settings.local.json` |
| Kept | `CLAUDE.local.md` and `.claude/settings.local.json` are not touched |

Take the user's answers on the unplaced sections and finalize the classification table.

## 3. Copy common files

Same as section 1 of `references/INIT.md`. The common files overwrite what is there, `.claude/settings.json` included. Read the existing `.claude/settings.json` before the copy so its content is in the plan table of section 2; nothing is merged into the new file. `CLAUDE.local.md` and `.claude/settings.local.json` are skipped when present, as in `init`.

If `.claude/rules/` has a rule with the same role under a different file name (for example `architecture.md`), unify it under the new name. If both remain, both load. When the two names differ only in case, rename the existing file with `mv` before writing the new one: on a case-insensitive file system such as the macOS default, writing `ARCHITECTURE.md` next to an existing `architecture.md` overwrites that file in place and git keeps the old name.

## 4. Analyze

Run `SKILL.md` common procedure 1 (codebase analysis). For facts already established in the existing documents, only re-verify against the codebase; investigate afresh only what the existing documents lack. Where the existing documents and the code disagree, follow the code and record the difference in the report.

## 5. Generate

Run `SKILL.md` common procedure 2 (generating project-dependent documents). Move each section's content from the classification table into its file. Normalize to MARKDOWN.md syntax while moving, without changing meaning.

## 6. Handle originals

Do this only after every new document is written.

```bash
# check tracking: prints the path when tracked, nothing when untracked
git ls-files <file>
```

Keep this command in exactly that form. It matches the skill's `git ls-files` permission; adding `&&`, `||`, `echo`, or a redirection turns it into a compound command that falls outside the allowed pattern and adds a prompt.

- tracked (path printed): delete with `rm`. Do not use `git rm`. Staging is the user's job.
- untracked (nothing printed), or not a git repository: create `.claude/legacy/` with `mkdir -p` and move the file there with `mv`. Claude Code does not read that directory.
- Files that were `@` import targets: confirm nothing else uses them, then apply the same rule.
- `rm` and `mv` are not in the skill's allowed tools, so each one asks the user for permission. This is the only step that removes user files, and the prompt is the user's last chance to stop. Run one file per command so each prompt names the file. Do not batch them into a single command or loop to avoid the prompts.

## 7. Verify and report

Run `SKILL.md` common procedures 3 (verification) and 4 (report). Add the following to the report.

- Section move mapping: old file:section → new file:section
- Entries of the replaced `.claude/settings.json`, if it had any, so the user can add them back
- Final handling of unplaced sections
- Items reported as out of scope (skill candidates, behavioral rules)
- Items where the existing documents disagreed with the code and the code was followed
- Claude-related `.md` files left outside the standard structure. These were not in the classification table and were not touched. Show path and reason as a table and leave the handling to the user
