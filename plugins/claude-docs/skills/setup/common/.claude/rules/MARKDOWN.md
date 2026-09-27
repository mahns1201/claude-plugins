---
title: "Markdown authoring rules"
description: "Single source of truth for the frontmatter, allowed syntax, and notation Claude follows when writing or editing any .md file"
created_at: 2026-09-27
updated_at: 2026-09-28
paths:
  - "**/*.md"
---

# Markdown authoring rules

## Principles

- One topic per file. Splitting is recommended past 200 lines.
- Lead with the conclusion. Details and rationale follow.
- Prefer lists and tables over long paragraphs.
- Emphasize through sentence structure and position, not syntax. Put the important thing first.
- When explaining a flow or a structure, prefer a mermaid diagram over prose.

## Frontmatter

Every `.md` file starts with YAML frontmatter. Exceptions are listed below.

```yaml
---
title: "<short title | Required>"
description: "<one-sentence description | Required>"
created_at: <YYYY-MM-DD | Required>
updated_at: <YYYY-MM-DD | Required>
---
```

- Always wrap `title` and `description` in double quotes. This removes any guessing about YAML special characters such as `:` or `#`. Dates are not quoted.
- `title` is the same string as the body's `#` heading.
- `created_at` is the file's creation date. Once set, it never changes.
- `updated_at` is set to today's date only when the body changes. Typo fixes or frontmatter-only edits do not update it.

`.claude/rules/*.md` may add `paths`. Always quote the values. A value starting with `*` is read as a YAML alias when unquoted and fails to parse.

```yaml
paths:
  - "<glob pattern | Optional>"
```

Every rule declares `paths`. A rule that applies everywhere uses `"**/*"`, which loads it as soon as any file is read. Do not omit the key: a rule without `paths` loads at every session start and cannot be told apart from one where `paths` was forgotten.

The only key Claude Code reads from a rule file is `paths`. The other keys are metadata for humans and are stripped before the rule is loaded.

### Exceptions

The following files do not use this frontmatter.

| File | Reason |
| --- | --- |
| `CLAUDE.md`, `CLAUDE.local.md` | Claude Code reads the whole body without frontmatter |
| `README.md`, `README.<lang>.md` | Rendered by GitHub and package registries, which show frontmatter as a table at the top |
| `.claude/skills/*/SKILL.md` | Follows the Claude Code skill spec (`name`, `description`, and so on). Do not add fields outside the spec |
| `.claude/agents/*.md` | Follows the Claude Code subagent spec (`name`, `description`, `tools`, and so on). Do not add fields outside the spec |

## Allowed syntax

Do not use any syntax that is not listed here.

| Syntax | Notation | Use |
| --- | --- | --- |
| Heading | `#`, `##`, `###` | Document title and sections. Never go below level 3 |
| Unordered list | `- ` | Parallel items |
| Ordered list | `1. `, `2. `, `3. ` | Procedures where order matters |
| Task list | `- [ ]`, `- [x]` | Acceptance criteria, checklists |
| Inline code | `` `code` `` | File names, identifiers, command names, key names |
| Code block | ```` ``` ```` + language tag | Commands, code, configuration, sample output |
| Blockquote | `> ` | Verbatim quotes from external documents or other files |
| Link | `[text](url or path)` | Cross-references, external material |
| Image | `![alt](src)` | Screenshots, visual material |
| Horizontal rule | `---` | Frontmatter boundary, major topic shift |
| Table | `\| a \| b \|` | Parallel data, comparisons |
| Diagram | ```` ```mermaid ```` | Flows, structures, relationships |

## Forbidden syntax

- Bold (`**`, `__`), italic (`*`, `_`), strikethrough (`~~`)
- HTML tags
- Footnotes, definition lists
- Setext headings (`===` or `---` underlines)
- Status shown with emoji or symbols. Write status as text such as `(done 2026-09-28)` or `(in progress)`

## Notation rules

- Do not skip heading levels. Never put `###` directly under `#`.
- One `#` per file.
- One blank line before and after headings, lists, code blocks, tables, and blockquotes.
- Every code block has a language tag. Use `text` when there is no language.
- Commands go in code blocks, never inline in a sentence.
- A list item is one or two sentences. If a paragraph is needed, do not use a list.
- Table cells do not end with a period. If a cell holds two or more sentences, put periods only between them.
- Paths inside the repository are always written from the project root, for example `.claude/rules/TEST.md`. Do not use paths relative to the document (`../rules/`). This keeps paths valid when a document is moved or copied into another document.
- Dates are `YYYY-MM-DD`.
- Wrap file names, directory names, identifiers, and key names in inline code.
- Write slots to be filled as placeholders of the form `<description>`. They are not HTML tags and are replaced with real values when the document is completed.

## Self-check

Before saving, search for the following and confirm zero hits. Exclude frontmatter globs, inline code that explains the syntax itself, and the inside of code blocks.

- `**`, `~~`, `####`
- HTML tags
- Emoji
- Document-relative paths (`../`)
- Placeholders `<...>` that were not replaced with real values (skeleton files excepted)
