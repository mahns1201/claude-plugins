---
name: setup
description: >-
  Brings the current project's Claude Code documents into the standard structure.
  The standard is a thin root CLAUDE.md (overview, commands, rules routing), .claude/rules/, a CLAUDE.md at each module root, and the two settings files.
  Performs no git write operations and never touches personal files. Opus or better and effort xhigh or higher recommended.
  Three modes.
  audit (default): writes nothing; compares the current documents with the standard, reports, and suggests the next mode.
  init: for a project with no documents; copies the common rules and analyzes the codebase to generate the project documents.
  migrate: moves existing documents section by section into the new structure, confirms the plan once, then cleans up the originals.
  Common rules are copied verbatim with cp; project documents are filled only with what the code confirms, leaving TODO where there is no evidence.
argument-hint: "[audit|init|migrate]"
disable-model-invocation: true
effort: xhigh
compatibility: "Opus-class model or better (Opus, Fable) with effort xhigh or higher recommended. The skill analyzes a codebase and writes project documents; smaller models or lower effort produce shallow or invented rules. Requires git for tracking checks."
allowed-tools:
  - Read
  - Glob
  - Grep
  - Write
  - Edit
  - Bash(ls *)
  - Bash(cat *)
  - Bash(head *)
  - Bash(wc *)
  - Bash(find *)
  - Bash(grep *)
  - Bash(git status *)
  - Bash(git ls-files *)
  - Bash(git diff *)
  - Bash(git log *)
  - Bash(git check-ignore *)
disallowed-tools:
  - Bash(git commit *)
  - Bash(git merge *)
  - Bash(git push *)
  - Bash(git rebase *)
  - Bash(git reset *)
  - Bash(git cherry-pick *)
  - Bash(git stash *)
  - Bash(git tag *)
  - Bash(git branch -D *)
  - Bash(git branch -d *)
  - Bash(git rm *)
---

# Claude Code project setup

Runs one of three modes against the current working directory. With no argument, `audit`.

| Mode | Writes | Procedure file | When |
| --- | --- | --- | --- |
| `audit` | No | `references/AUDIT.md` | Default. Compares the current state with the standard, reports, and suggests the next mode |
| `init` | Yes | `references/INIT.md` | A project that has no `CLAUDE.md` and no `.claude/rules/` yet |
| `migrate` | Yes | `references/MIGRATE.md` | A project with existing Claude documents. Absorbs their content and replaces them with the standard structure |

## Before starting

1. Read the mode from `$ARGUMENTS`. Empty means `audit`. Any other value: ask the user to pick one of the three modes and stop.
2. If the session model is a Sonnet or Haiku class model, say so and ask whether to continue. This skill reads a codebase and infers rules, so Opus or better with effort xhigh is recommended. Do not read `settings.json` files to find out the model or effort; state the recommendation once and move on.
3. The target is the current working directory. If it is not a git repository, say so and proceed. This changes how originals are handled in `migrate`.
4. Tell the user what permission prompts to expect in `init` and `migrate`, then proceed. Claude Code protects the `.claude/` directory and blocks shell access to paths outside the working directory. Neither can be pre-approved by this skill, so every write under `.claude/` and every `cp` from the plugin directory asks once. Choosing "always allow" for the session at the first prompt of each kind is safe; `rm` and `mv` in `migrate` are the exception and should be approved one by one.
5. Read the procedure file for the mode and follow it exactly. The common procedures below are referenced from the procedure files.

## File classification

| Kind | Location | Handling |
| --- | --- | --- |
| Common | `common/` | Copied with `cp`, content unchanged. `CLAUDE.local.md`, `.claude/settings.json`, `.claude/settings.local.json`, `.claude/rules/CLAUDE_CODE.md`, `.claude/rules/MARKDOWN.md` |
| Project-dependent | `samples/` | Skeletons. Filled from the analysis and written into the target. `CLAUDE.md`, `src/CLAUDE.md` (one per module), `.claude/rules/ARCHITECTURE.md`, `NAMING.md`, `TEST.md`, `REVIEW.md` |

Never move a `common/` file by reading it and writing it back. Always use `cp`. If a single character changes, it is no longer a common file.

The plugin directory is `${CLAUDE_PLUGIN_ROOT}`. Claude Code substitutes the real path into this file when the skill loads, but not into the procedure files, which show the variable name. In shell commands always write that absolute path, never a shell variable reference; a command containing a variable expansion is never auto-approved and is denied outright in non-interactive runs.

## Common procedures

### 1. Codebase analysis

Establish the following before writing any file. Record the evidence file paths and carry them into the generated documents.

| What to establish | Evidence |
| --- | --- |
| Language, runtime, framework, build tool | Build files such as `package.json`, `build.gradle`, `pyproject.toml`, `go.mod`, `Cargo.toml`; `.tool-versions` |
| Install, run, test, lint, build commands | Scripts in the build file, `Makefile`, `README.md`, CI configuration |
| Module boundaries and roles | Top-level directory layout, multi-module configuration, package roots |
| Dependency direction | Import relationships between modules. Actually read the imports of a few representative files |
| Naming patterns | Rules actually in use in existing file, class, and function names. Lint configuration |
| Test structure | Test framework, test file location and names, structure of existing tests |

Do not guess. Leave anything without evidence as `<TODO: needs confirmation - reason>`. In `migrate`, the existing Claude documents are the first source of evidence.

### 2. Generating project-dependent documents

Read the skeletons in `${CLAUDE_PLUGIN_ROOT}/skills/setup/samples/`, fill them from the analysis, and write them into the target.

| Generated file | Skeleton | What to fill |
| --- | --- | --- |
| `CLAUDE.md` | `samples/CLAUDE.md` | Project overview, commands, rules reference guide. Three sections only, 50 lines or fewer |
| `.claude/rules/ARCHITECTURE.md` | `samples/.claude/rules/ARCHITECTURE.md` | Module table, dependency diagram, layer table. Set `paths` to the real source root |
| `<module root>/CLAUDE.md` | `samples/src/CLAUDE.md` | One per module. For a module without enough evidence, do not create it and leave the ARCHITECTURE.md table cell empty |
| `.claude/rules/NAMING.md` | `samples/.claude/rules/NAMING.md` | Rules observed in real code. Do not add rules that were not observed. Every example must be a name that exists in the codebase. `paths` is `"**/*"`, so keep it short |
| `.claude/rules/TEST.md` | `samples/.claude/rules/TEST.md` | Framework, location, structure. Set `paths` to the real test file pattern |
| `.claude/rules/REVIEW.md` | `samples/.claude/rules/REVIEW.md` | Keep the checklist table as is. Fill only the project-specific checks. `paths` is `"**/*"`, so keep it short |

Generated documents follow `.claude/rules/MARKDOWN.md` and `.claude/rules/CLAUDE_CODE.md` as copied from `common/`. Replace every placeholder `<...>` with a real value or leave it as `<TODO: ...>`. Never leave an empty placeholder in place. Prose in the skeletons is written for the reader of the finished document, such as cross-references to other rule files. Keep it as is; only the placeholders are filled.

Proposing additional rules. When the analysis reveals an area that fits none of the `samples/` files, propose a new rule. Examples: database schema and migrations, API contracts and documentation, authentication and authorization, internationalization, deployment and infrastructure configuration, generated code. Show proposals as a table and create only the ones the user picks.

| Item | Content |
| --- | --- |
| File name | `.claude/rules/<UPPERCASE>.md` |
| `paths` | The real path pattern for the area. If it must always load, the reason why |
| Content | Two or three rules observed in the code |
| Evidence | File paths supporting the judgment |

When creating one, follow the skeleton format of `samples/.claude/rules/` (frontmatter, table-first, placeholders) and add a row to the Rules reference guide in the root `CLAUDE.md` and to the `REVIEW.md` checklist. Proposals that were not created go into the report.

### 3. Verification

- Search the generated `.md` files for `**`, `~~`, `####`, and imports starting with `@`. Exclude frontmatter globs and lines that explain the syntax itself.
- Confirm every `paths` value in `.claude/rules/*.md` is quoted and matches paths that actually exist. Confirm every rule has `paths`; a rule that applies everywhere uses `"**/*"`.
- Confirm the Rules reference guide in `CLAUDE.md` points only to rule files that exist.
- Confirm `.gitignore` contains `CLAUDE.local.md` and `.claude/settings.local.json`.
- Collect the remaining `<TODO>` items.
- Find Claude-related `.md` files outside the standard structure. Candidates: `.md` files under `.claude/` that are not standard files, other agents' instruction files at the root such as `AGENTS.md` or `GEMINI.md`, files the old `CLAUDE.md` pointed to via `@` or by path, other tools' rule files such as `.cursor/rules/`. Do not delete them.

### 4. Report

- Report copied files, generated files, deleted or moved files, and the `<TODO>` list as separate groups.
- Report proposed rules that were not created, with their evidence.
- If Claude-related `.md` files exist outside the standard structure, report their paths and the reasoning as a table. This skill did not touch them, so they remain in place; the user decides whether to move them into the new structure or delete them.
- The first generated result is a draft. Recommend that the user review it once.
- This skill performs no git write operations. The user commits.
