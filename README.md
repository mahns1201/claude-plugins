English | [한국어](README.ko.md)

# claude-plugins

Use Claude Code for a while and `CLAUDE.md` grows. Build commands get an architecture section next to them, then naming rules, then how to write tests, then a review checklist. That whole file is loaded into context at the start of every session, and the longer it gets, the less of it Claude follows. Start another project and you write the same document again from scratch.

This repository packages an answer to that problem as a single plugin. The root `CLAUDE.md` holds only the project overview, the commands, and a table that says which rule file to read for which kind of task. The rules themselves are split into files under `.claude/rules/`, each with a `paths` setting so it loads only when a related file is read. Each module gets a small `CLAUDE.md` at its root for conventions that matter only inside it. Rules that are the same in every project are copied as they are; content that differs per project is filled in by reading the codebase.

## How it works

This repository is a Claude Code plugin marketplace. Once registered, you can install the `claude-docs` plugin, which contains one skill, `setup`. The skill runs in one of three modes.

`audit` writes nothing. It reads the project's current Claude documents and reports how they differ from the standard: how many lines load at every session, whether there are `@` imports or files in the wrong place, and which mode to run next. Running the skill with no argument gives you this mode.

`init` is for a project with no Claude documents at all. It copies the common rule files, then reads the build files, directory layout, and existing code to produce the project documents. It writes only what the code confirms and leaves a `TODO` wherever it could not.

`migrate` is for a project that already has a `CLAUDE.md`. It reads the existing documents section by section, classifies where each section belongs in the new structure, and collects anything with no destination to ask you about. After you confirm the plan once, it writes the new documents and cleans up the originals. Nothing is dropped silently.

Usage details are in `plugins/claude-docs/README.md`.

## Repository layout

```text
claude-plugins/                        # marketplace root
├── .claude-plugin/marketplace.json        # plugin catalog (name: mahns)
├── plugins/claude-docs/                   # the only unit that is distributed
│   ├── .claude-plugin/plugin.json
│   ├── README.md, README.ko.md
│   └── skills/setup/
│       ├── SKILL.md                       # mode selection and common procedures
│       ├── references/                    # per-mode procedures: AUDIT.md, INIT.md, MIGRATE.md
│       ├── common/                        # files copied verbatim
│       └── samples/                       # skeletons filled from analysis
└── README.md, README.ko.md
```

Only the `plugins/claude-docs/` directory is installed on a user's machine. The root READMEs are for people developing or visiting this repository.

## Design decisions

A plugin cannot ship `CLAUDE.md` or `rules/` directly into a user's project, because Claude Code does not read a `CLAUDE.md` at a plugin root. So the skill copies and generates files instead.

The skill contains no shell scripts. Copying is `cp`, and deciding what to write where is Claude's job. At half a dozen files, a script only added maintenance cost.

The skill does not write to git. Commit, merge, and push are blocked in its frontmatter; when it finishes, you review the result and commit.

Documents are written in English. READMEs have a Korean version alongside.

## Development and release

Validate both manifests after any change.

```bash
claude plugin validate .
claude plugin validate --strict ./plugins/claude-docs
```

To try it in a real session while editing, load the plugin directory directly. Files are read in place, so after an edit a `/reload-plugins` is enough.

```bash
claude --plugin-dir ./plugins/claude-docs
```

To test the install path itself, register this directory as a local marketplace. This copies the plugin into `~/.claude/plugins/cache/`, so edits are not picked up until you uninstall and install again.

```bash
claude plugin marketplace add ./
claude plugin install claude-docs@mahns
```

Releasing is a push to GitHub. The plugin name `claude-docs` and the marketplace name `mahns` must never change after publishing; a rename breaks every existing install. Bump `version` in `plugins/claude-docs/.claude-plugin/plugin.json` on every release, or `claude plugin update` will not pick it up.

## Deferred from the first release

Sample skill, sample subagent, PRD rules, MCP configuration, and a code-change policy were left out of the first release. For a later release, add the file to `common/` or `samples/` and add one row to the file classification table in `SKILL.md`.
