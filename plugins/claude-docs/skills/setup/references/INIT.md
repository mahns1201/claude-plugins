---
title: "init procedure"
description: "Procedure that creates the standard structure in a project that has no Claude Code documents"
created_at: 2026-09-28
updated_at: 2026-09-28
---

# init procedure

Precondition: none of `CLAUDE.md`, `.claude/CLAUDE.md`, `.claude/rules/` exist. If any does, stop and recommend `migrate`. `CLAUDE.local.md` and `.claude/settings.local.json` are personal files, are not part of the precondition, and are left untouched if present.

## 1. Copy common files

`${CLAUDE_PLUGIN_ROOT}` below stands for the plugin directory. Write its absolute path as shown in `SKILL.md`, not the variable. Run each line as its own command. Every command here asks for permission once: `mkdir` and `cp` because they write under `.claude/`, and `cp` also because the plugin directory is outside the working directory. This is Claude Code's protection, not a fault in the command; the user may pick "always allow" for the session.

```bash
mkdir -p .claude/rules
cp ${CLAUDE_PLUGIN_ROOT}/skills/setup/common/.claude/rules/CLAUDE_CODE.md ${CLAUDE_PLUGIN_ROOT}/skills/setup/common/.claude/rules/MARKDOWN.md .claude/rules/
cp ${CLAUDE_PLUGIN_ROOT}/skills/setup/common/.claude/settings.json .claude/
cp ${CLAUDE_PLUGIN_ROOT}/skills/setup/common/.claude/settings.local.json .claude/
cp ${CLAUDE_PLUGIN_ROOT}/skills/setup/common/CLAUDE.local.md .
```

Before the last two lines, check with `ls` whether `.claude/settings.local.json` and `CLAUDE.local.md` already exist. Skip the copy for any that does. They are personal files and are never overwritten. Do not fall back to reading a common file and writing it back if a `cp` is denied; stop and tell the user.

Add `CLAUDE.local.md` and `.claude/settings.local.json` to `.gitignore` if missing. Read `.gitignore` first; if it does not exist, create it with `Write`, otherwise append the missing lines with `Edit`. Do not use shell redirection for this.

## 2. Analyze

Run `SKILL.md` common procedure 1 (codebase analysis).

## 3. Generate

Run `SKILL.md` common procedure 2 (generating project-dependent documents). Order: `ARCHITECTURE.md` → module `CLAUDE.md` → `NAMING.md` → `TEST.md` → `REVIEW.md` → root `CLAUDE.md`. The root `CLAUDE.md` is written last so that its Rules reference guide points only to rule files that were actually created.

## 4. Verify and report

Run `SKILL.md` common procedures 3 (verification) and 4 (report).
