# Project CLAUDE.md

## Project overview

- <one-line domain description> <language/runtime> / <framework> / <build tool>
- `<module or directory>` — <role>
- `<module or directory>` — <role>
- Dependency direction: `<A> → <B> → <C>` (one-way). Details in `.claude/rules/ARCHITECTURE.md`

## Commands

- Install: `<command>`
- Run: `<command>`
- Test: `<command>`
- Lint/format: `<command>`
- Build: `<command>`

## Rules reference guide

Always consult the matching `.claude/rules` file for the task type.

| Task type | Rule file |
| --- | --- |
| Modules, layers, dependencies | `.claude/rules/ARCHITECTURE.md` |
| Naming | `.claude/rules/NAMING.md` |
| Writing or changing tests | `.claude/rules/TEST.md` |
| Code review, final check | `.claude/rules/REVIEW.md` |
| Writing or editing `.md` documents | `.claude/rules/MARKDOWN.md` |
| Writing or editing `CLAUDE.md` or `.claude/` settings | `.claude/rules/CLAUDE_CODE.md` |
