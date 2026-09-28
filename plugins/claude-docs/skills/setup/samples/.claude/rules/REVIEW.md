---
title: "Review rules"
description: "Single source of truth for the checklist used before reporting work as done and when a review is requested"
created_at: <YYYY-MM-DD>
updated_at: <YYYY-MM-DD>
paths:
  - "**/*"
---

# Review rules

`paths` matches every file, so this rule is loaded as soon as any file is read.

Run this before reporting work as done and whenever a review is requested. Each item points to the rule file that owns the criterion. The criteria themselves are not repeated here.

## Checklist

| Check | Criterion |
| --- | --- |
| Dependency direction and layer boundaries are respected | `.claude/rules/ARCHITECTURE.md` |
| New code sits in the right module and layer | `.claude/rules/ARCHITECTURE.md`, code placement guide |
| New names follow the rules | `.claude/rules/NAMING.md` |
| Changed code has tests and they pass | `.claude/rules/TEST.md` |
| If documents changed, syntax and frontmatter are correct | `.claude/rules/MARKDOWN.md` |
| If modules or rules changed, the root `CLAUDE.md` and the ARCHITECTURE.md table are updated | `.claude/rules/CLAUDE_CODE.md` |

## Project-specific checks

Things this project tends to miss.

- [ ] <example: a migration file accompanies every schema change>
- [ ] <example: API changes update the docs or contract file>
- [ ] <example: a new config key is also added to the example file>
