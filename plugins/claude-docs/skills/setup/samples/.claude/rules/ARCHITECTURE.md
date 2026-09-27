---
title: "Architecture rules"
description: "Single source of truth for module structure, dependency direction, and layer responsibilities. Rules inside a module belong to that module's root CLAUDE.md"
created_at: <YYYY-MM-DD>
updated_at: <YYYY-MM-DD>
paths:
  - "<source-root>/**"
---

# Architecture rules

Covers the whole structure only. How code is written inside a module is owned by that module's root `CLAUDE.md`. Naming follows `.claude/rules/NAMING.md`.

## Module structure

| Module | Role | Location | Module CLAUDE.md |
| --- | --- | --- | --- |
| `<module>` | <one-line role> | `<path>/` | `<path>/CLAUDE.md` |
| `<module>` | <one-line role> | `<path>/` | `<path>/CLAUDE.md` |

- When a module is added or removed, update this table and the project overview in the root `CLAUDE.md` in the same change.
- An empty Module CLAUDE.md cell means that module has no module-level rules yet. Never point to a file that does not exist.

## Dependency direction

```mermaid
flowchart LR
    A["<module>"] --> B["<module>"]
    B --> C["<module>"]
```

- <allowed and forbidden dependencies>
- <how modules couple, for example public entry points or events>
- A change that alters the dependency direction updates this document first, gets confirmation, and only then changes code.

## Layer responsibilities

| Layer | Responsibility | May depend on |
| --- | --- | --- |
| `<layer>` | <responsibility> | <layers it may depend on> |
| `<layer>` | <responsibility> | <layers it may depend on> |

- <layer boundary rules>

## Code placement guide

Decide where new code goes in this order.

1. Which module owns it. Pick from the module structure table.
2. Which layer. Pick from the layer responsibilities table.
3. Read that module's `CLAUDE.md`. Internal layout and conventions are there.

## Module CLAUDE.md

- Exactly one at each module root. None in subdirectories.
- Do not repeat anything from this document. Module list, dependency direction, and layer responsibilities are owned here.
- Holds: role, public entry points, internal layout, conventions and pitfalls.
- Loaded automatically when Claude reads a file in that module. During design work, read it directly using the path in the module structure table.

## New module checklist

- [ ] Added a row to the module structure table
- [ ] Updated the dependency direction diagram
- [ ] Updated the project overview in the root `CLAUDE.md`
- [ ] Created a `CLAUDE.md` at the module root, or left the table cell empty if there is none yet
