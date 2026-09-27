---
title: "Test rules"
description: "Single source of truth for this project's test framework, file locations, and structure"
created_at: <YYYY-MM-DD>
updated_at: <YYYY-MM-DD>
paths:
  - "<test-file-pattern>"
---

# Test rules

Run commands follow the Commands section of the root `CLAUDE.md`. Test naming follows `.claude/rules/NAMING.md`.

## Tooling

| Item | Value |
| --- | --- |
| Framework | `<framework>` |
| Runner, run command | `<command>` |
| Assertion and mocking libraries | `<library>` |

## Location and naming

- <where test files live, for example next to the unit or in a separate directory>
- <test file name pattern>

## Structure

- <structure observed in existing tests, for example how arrange/act/assert is separated or how deep describe blocks nest>
- <how test doubles are handled>

## Scope

- <which changes must come with which tests>
- <what is not tested and why>
