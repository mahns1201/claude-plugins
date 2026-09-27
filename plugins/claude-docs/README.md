English | [한국어](README.ko.md)

# claude-docs

A skill that brings a project's Claude Code documents into a standard structure: a thin `CLAUDE.md` plus `.claude/rules/`. For a project with no documents it creates them; for a project with an already heavy `CLAUDE.md` it carries the content over into the new structure.

## Install

```bash
claude plugin marketplace add mahns1201/claude-plugins
claude plugin install claude-docs@mahns
```

## Use

Open a Claude Code session at the project root and run it.

```text
/claude-docs:setup
```

With no argument this is `audit` mode. It only reads and reports how the current documents differ from the standard: how long the root `CLAUDE.md` is, how many lines of rules load at every session because they have no `paths`, whether there are `@` imports or files in the wrong place, whether environment variables have crept into `settings.json`, and so on. It ends by suggesting the next mode. Run it once before changing anything.

```text
/claude-docs:setup init
```

For a project with neither `CLAUDE.md` nor `.claude/rules/`. It copies the common rule files, then reads the stack and commands from the build files, the modules and dependency direction from the directory layout and imports, and the naming and test structure from existing code, and writes the project documents. It writes only what the code confirms and leaves `TODO` where it could not, so when it finishes you skim the result and fill in the `TODO`s.

```text
/claude-docs:setup migrate
```

For a project that already has Claude documents. It reads the existing `CLAUDE.md` and rule files section by section, decides which file in the new structure each section belongs to, and collects anything with no destination to ask you about. It shows the plan as a table and writes nothing until you confirm. Originals are deleted if git tracks them (history keeps them) and moved to `.claude/legacy/` otherwise. Each delete or move asks for permission; that is the one step that removes your files, so it is not pre-approved.

In `init` and `migrate`, expect permission prompts. Claude Code protects the `.claude/` directory and blocks shell access outside the working directory, and a plugin cannot pre-approve either, so each write under `.claude/` and each copy from the plugin directory asks once. Choosing "always allow" for the session at the first prompt of each kind is safe. Only the `rm` and `mv` of your original files in `migrate` are worth approving one at a time.

An Opus-class model or better with effort xhigh or higher is recommended. The work is reading a codebase and inferring rules, and smaller models tend to produce shallow or invented ones.

## What you get

The root `CLAUDE.md` holds only the project overview, the commands, and a table that tells Claude which rule file to read for which kind of task. 50 lines or fewer.

Two kinds of files appear under `.claude/rules/`. `CLAUDE_CODE.md` and `MARKDOWN.md` are common rules that are the same in every project and are copied unchanged. `ARCHITECTURE.md`, `NAMING.md`, `TEST.md`, and `REVIEW.md` differ per project and are filled from the codebase analysis. Each rule file carries `paths`, so it loads only when a related file is read. `NAMING.md` and `REVIEW.md` apply everywhere and use `**/*`.

Each module gets a small `CLAUDE.md` at its root. `ARCHITECTURE.md` covers the whole structure; the module `CLAUDE.md` covers the conventions inside it. It loads automatically when Claude reads a file in that module.

`.claude/settings.json` is the team-shared settings file and `.claude/settings.local.json` the personal one. Environment variables go only in the local file, which is added to `.gitignore` together with `CLAUDE.local.md`.

If the analysis finds an area the standard structure does not cover, such as a database schema or an API contract, the skill proposes an additional rule file. It only proposes; you pick what gets created.

## What it does not do

It does not write to git. Commit, merge, and push are blocked inside the skill; when it finishes, you review and commit.

It does not touch personal files. An existing `CLAUDE.local.md` or `.claude/settings.local.json` is left alone.

It does not invent rules. Only what the code confirms is written; the rest is `TODO`.

It does not drop existing content silently. In `migrate`, anything with no destination is listed for you to decide.
